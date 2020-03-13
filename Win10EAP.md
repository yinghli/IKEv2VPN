Windows 10 native IKEv2 with Certificate and EAP authentication
======

This blog is to describe how to setup windows 10 as client to connect IKEv2 VPN. 
Because windows 10 don't support pre-share key, we will use EAP and Certification to demo this. 

Setup Windows 2016 as CA server
-----
+ Follow this [link](https://docs.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/install-the-certification-authority) to setup Windows 2016 as Certificate Authority and with “Network Device Enrollment Service”.

+ Clone certificate template. Windows CA default certificate template is "IPSECIntemediateOffline". Based on this [link](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/115907-config-flexvpn-wcca-00.html), certificate must have the EKU fields set to 'Server Authentication' for Cisco IOS and 'Client Authentication' for the client. We clone a template that support both EKU. Name it as "RASandIASServer".<br>
> please note that "user" should have permission to "enroll" the template.

![](https://github.com/yinghli/IKEv2VPN/blob/master/CAtemplate.jpg)

+ Change MSCEP default template and disable SCEP enrollment password.<br>
Open `Regedit`. <br>
Browse to the path `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP`, change all default template name to previous setup we cloned. <br>
Browse to the path `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP\EnforcePassword`, change EnforcePassword REG_DWORD value to 0. <br>

![](https://github.com/yinghli/IKEv2VPN/blob/master/regedit.jpg)

CSR1000v Certification enrollment
------

+ Use this command to generate key pair. `crypto key generate rsa label ikev2rsa modulus 2048`

+ Define a PKI Trustpoint. In my case, 10.1.1.5 is CA server setup in previous step. VPN server FQDN is ikev2.yinghli.cn.
```
crypto pki trustpoint msca
 enrollment mode ra
 enrollment url http://10.1.1.5:80/certsrv/mscep/mscep.dll
 serial-number none
 fqdn ikev2.yinghli.cn
 subject-name OU=IT,O=yinghli
 revocation-check none
 rsakeypair ikev2rsa
```
+ Download the CA’s root certificate. `crypto pki authenticate msca`

+ Enroll the certificate. `crypto pki enroll msca`

+ Verify the certificate. `show crypto pki certificates verbose msca`


Windows 10 client setup
------
+ Client generate a certificate request.

![](https://github.com/yinghli/IKEv2VPN/blob/master/CSR.jpg)

+ Client open browser and access https://sever ip addr/certsrv to enroll a new certificate.

![](https://github.com/yinghli/IKEv2VPN/blob/master/Enroll.jpg)

Follow this [link](https://www.altaro.com/hyper-v/request-ssl-windows-certificate-server/) to get client certificate. 


CSR1000v IKEv2 setup and AAA setup
------
1. Setup IKEv2 proposal and policy. Windows 10 should setup with powershell to meet same parameters.
```
crypto ikev2 proposal IKEv2-prop1
 encryption aes-cbc-256
 integrity sha256
 group 14
!
crypto ikev2 policy IKEv2-pol
 proposal IKEv2-prop1
```

2. Setup IKEv2 authorization policy. 
```
aaa new-model
!
aaa authorization network local-group-author-list local
!
ip local pool YHLPOOL 10.6.1.10 10.6.1.200
crypto ikev2 authorization policy ikev2-auth-policy
 pool YHLPOOL
 dns 8.8.8.8
```

3. Setup IKEv2 profile.
```
crypto pki certificate map win10 10
 issuer-name co yinghli
!
crypto ikev2 profile Win10
 match certificate win10
 identity local fqdn ikev2.yinghli.cn
 authentication remote rsa-sig
 authentication local rsa-sig
 pki trustpoint msca
 aaa authorization group cert list local-group-author-list ikev2-auth-policy
 virtual-template 400
```

4. Setup IPSec profile.
```
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel
!
crypto ipsec profile win10
 set transform-set TS
 set ikev2-profile Win10
```

5. Setup tunnel template and apply IPSec profile.
```
interface Virtual-Template400 type tunnel
 ip unnumbered Loopback2
 ip mtu 1400
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile win10

```
