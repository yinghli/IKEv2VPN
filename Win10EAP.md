Windows 10 native IKEv2 with Certificate and EAP authentication
======

This blog is to describe how to setup windows 10 as client to connect IKEv2 VPN. 
Because windows 10 don't support pre-share key, we will use EAP and Certification to demo this. 

Setup Windows 2016 as CA server
-----
+ Follow this [link](https://docs.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/install-the-certification-authority) to setup Windows 2016 as Certificate Authority and with “Network Device Enrollment Service”.

+ Clone certificate template. Windows CA default certificate template is "IPSECIntemediateOffline". Based on this [link](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/115907-config-flexvpn-wcca-00.html), certificate must have the EKU fields set to 'Server Authentication' for Cisco IOS and 'Client Authentication' for the client. We clone a template that support both EKU. Name it as "RASandIASServer".

![](https://github.com/yinghli/IKEv2VPN/blob/master/CAtemplate.jpg)

+ Change MSCEP default template and disable SCEP enrollment password.<br>
Open `Regedit`. <br>
Browse to the path `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP`, change all default template name to previous setup we cloned. <br>
Browse to the path `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP\EnforcePassword`, change EnforcePassword REG_DWORD value to 0. <br>

![](https://github.com/yinghli/IKEv2VPN/blob/master/regedit.jpg)

CSR1000v Certification enrollment
------

1. Use this command to generate key pair. `crypto key generate rsa label ikev2rsa modulus 2048`

2. Define a PKI Trustpoint. In my case, 10.1.1.5 is CA server setup in previous step. VPN server FQDN is ikev2.yinghli.cn.
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
3. Download the CA’s root certificate. `crypto pki authenticate msca`

4. Enroll the certificate. `crypto pki enroll msca`

5. Verify the certificate. `show crypto pki certificates verbose msca`


CSR1000v IKEv2 setup and AAA setup
------

Windows 10 client setup
------
Follow this [link](https://www.altaro.com/hyper-v/request-ssl-windows-certificate-server/) to get client certificate. 
