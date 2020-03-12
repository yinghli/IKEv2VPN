IKEv2 VPN
=====
Cisco Cloud Service Router as IKEv2 VPN server, Windows 2016 as Certification Authority and Network Policy Server, Windows 10 and iOS device as client. Test end to end IKEv2 VPN with pre-share key, EAP and Certification as authentication method. 

Baisc knowledge of IKEv2
-----
![](https://github.com/yinghli/IKEv2VPN/blob/master/IKEv2Basic.jpg)


+ IKE_SA_INIT: initiator and the responder will exchange the cryptographic algorithms and Diffie-Hellman group (DH). <br>
On CSR1000v, this information is defined by `crypto ikev2 proposal` command. 
```
crypto ikev2 proposal IKEv2-prop1 
 encryption aes-cbc-256
 integrity sha256
 group 14

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac 
 mode tunnel
```
On Windows 10, default profile only support DH group 2, this information can be modified via PowerShell. 
```
Set-VpnConnectionIPsecConfiguration -ConnectionName "test" \
  -CipherTransformConstants AES256 \
  -EncryptionMethod AES256 \
  -IntegrityCheckMethod SHA256 \
  -DHGroup Group14 \
  -PfsGroup None \
  -AuthenticationTransformConstants AES256128
```
![](https://github.com/yinghli/IKEv2VPN/blob/master/IKEAUTH.jpg)

+ IKE_AUTH: initiator and the responder will reveal their identify. Then each device must authenticate their peer. Three methods of authentication are used in IKEv2: signature, pre-shared key, and EAP. <br>
On CSR1000v, this information is defined by `crypto ikev2 profile` command. Here is an example of pre-share key authentication.
```
crypto ikev2 profile AnyConnect-EAP
 match identity remote email router@cisco.com
 identity local fqdn ikev2.cisco.com
 authentication remote pre-share key cisco123
 authentication local pre-share key cisco123
 ```
Windows 10 don't support pre-share key authenticatioin, here is iOS device IKEv2 VPN profile example. 
![](https://github.com/yinghli/IKEv2VPN/blob/master/iOS.jpg)

iOS device native IKEv2 with pre-share key
-----

Windows 10 Anyconnect client with EAP 
-----

Windows 10 native IKEv2 with Certification
-----
