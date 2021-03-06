# Windows 10 native IKEv2 with Certificate and EAP authentication

This blog is to describe how to setup windows 10 connect to an IKEv2 VPN Server using EAP-TLS authentication. We are using Windows 2016 as Certificate Authority, Windows Network Policy Server as Radius server and Windows 10 native IKEv2 client.

- [Setup Windows 2016 as CA server](#setup-windows-2016-as-ca-server)
- [Setup Windows 2016 NPS Radius service](#setup-windows-2016-nps-radius-service)
- [CSR1000v Certification enrollment and IKEv2 configuration](#csr1000v-certification-enrollment-and-ikev2-configuration)
  * [CSR1000v Certification enrollment.](#csr1000v-certification-enrollment)
  * [CSR1000v IKEv2 setup and AAA setup](#csr1000v-ikev2-setup-and-aaa-setup)
- [Windows 10 client setup](#windows-10-client-setup)
  * [Client get certificate](#client-get-certificate)
  * [Setup VPN connection](#setup-vpn-connection)

## Setup Windows 2016 as CA server

1. Follow this [link](https://docs.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/install-the-certification-authority) to setup Windows 2016 as Certificate Authority and with "Network Device Enrollment Service(NDES)".

2. Clone certificate template. Windows CA default certificate template is "IPSECIntemediateOffline". Based on this [link](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/115907-config-flexvpn-wcca-00.html), certificate must have the EKU fields set to `Server Authentication` for Cisco IOS and `Client Authentication` for the client. We clone a template that support both EKU. Name it as "RASandIASServer".<br>
> please note that NDES service account should have permission to "enroll" this template.

![](https://github.com/yinghli/IKEv2VPN/blob/master/CAtemplate.jpg)

3. Change MSCEP default template and disable SCEP enrollment password.<br>
Open `Regedit`. <br>
Browse to the path `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP`, change all default template name to previous setup we cloned. <br>
Browse to the path `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP\EnforcePassword`, change EnforcePassword REG_DWORD value to 0. <br>

![](https://github.com/yinghli/IKEv2VPN/blob/master/regedit.jpg)

## Setup Windows 2016 NPS Radius service

1. Setup Radius Client IP and shared secret.
![](https://github.com/yinghli/IKEv2VPN/blob/master/NPS1.jpg)

2. Setup network policy and enable EAP-TLS.
![](https://github.com/yinghli/IKEv2VPN/blob/master/NPS2.jpg)

3. Setup Radius authorization attribuits. `Framed-Pool` defined VPN client IP pool and `ipsec:dns-servers=8.8.8.8` define the DNS server.
![](https://github.com/yinghli/IKEv2VPN/blob/master/NPS3.jpg)

## CSR1000v Certification enrollment and IKEv2 configuration

### CSR1000v Certification enrollment.

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

5. Verify the certificate. `show crypto pki certificates verbose msca`. Make sure this certificate have `Server Authentication` and `Client Authentication` EKU. 
```
    Authority Info Access:
    Extended Key Usage:
        Server Auth
        Client Auth
```

### CSR1000v IKEv2 setup and AAA setup

1. Setup IKEv2 proposal and policy. Windows 10 should setup with PowerShell to meet same parameters.
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
3. Setup AAA and Radius.
```
radius server windowsRadius
 address ipv4 10.1.1.5 auth-port 1645 acct-port 1646
 key 7 031352050200365F 
!
aaa group server radius FreeW
 server name windowsRadius
!
aaa authentication login ikev2win group FreeW
```
4. Setup IKEv2 profile.
```
crypto ikev2 profile Win10
 match identity remote address 0.0.0.0 
 authentication local rsa-sig
 authentication remote eap query-identity
 pki trustpoint msca
 aaa authentication eap ikev2win
 aaa authorization group eap list local-group-author-list ikev2-auth-policy
 aaa authorization user eap cached
 virtual-template 200
```
5. Setup IPSec profile.
```
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel
!
crypto ipsec profile win10
 set transform-set TS
 set ikev2-profile Win10
```
6. Setup tunnel template and apply IPSec profile.
```
interface Virtual-Template200 type tunnel
 ip unnumbered Loopback2
 ip mtu 1400
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile win10
```

## Windows 10 client setup

### Client get certificate

1. Client generate a certificate request, follow the setup and save as file.

![](https://github.com/yinghli/IKEv2VPN/blob/master/CSR.jpg)

2. Client open browser and access https://sever_ip_addr/certsrv and login with your credential. 
3. Download CA certificate and install in your local `Trusted Root Certification Authorities`.
4. Click request a certificate and select advance. 
5. Open previous step file via Notepad and copy into the website, choose "User" template and submit certificate request. 
6. Download the certificate and install into `Current User/Personal Certificates`.

![](https://github.com/yinghli/IKEv2VPN/blob/master/Enroll.jpg)

> Reference this [link](https://www.altaro.com/hyper-v/request-ssl-windows-certificate-server/) to get client certificate. 

### Setup VPN connection

1. Create VPN profile 
![](https://github.com/yinghli/IKEv2VPN/blob/master/Client1.jpg)

2. Setup EAP authentication
![](https://github.com/yinghli/IKEv2VPN/blob/master/Client2.jpg)

3. Change default IPSec profile to meet CSR1000v configuration.
```
Set-VpnConnectionIPsecConfiguration -ConnectionName "test" \
  -CipherTransformConstants AES256 \
  -EncryptionMethod AES256 \
  -IntegrityCheckMethod SHA256 \
  -DHGroup Group14 \
  -PfsGroup None \
  -AuthenticationTransformConstants AES256128
```
