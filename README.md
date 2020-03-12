IKEv2 VPN
=====
Cisco Cloud Service Router as IKEv2 VPN server, Windows 2016 as Certification Authority and Network Policy Server, Windows 10 and iOS device as client. Test end to end IKEv2 VPN with pre-share key, EAP and Certification as authentication method. 

Baisc knowledge of IKEv2
-----
![](https://github.com/yinghli/IKEv2VPN/blob/master/IKEv2Basic.jpg)


..* IKE_SA_INIT: initiator and the responder will exchange the cryptographic algorithms and Diffie-Hellman group (DH). <br>
On CSR1000v, this information is defined by `crypto ikev2 proposal` command. 
```
crypto ikev2 proposal IKEv2-prop1 
 encryption aes-cbc-256
 integrity sha256
 group 14
```
On Windows 10, this information can be modified via PowerShell. 
```
Set-VpnConnectionIPsecConfiguration -ConnectionName "test" \
  -CipherTransformConstants AES256 \
  -EncryptionMethod AES256 \
  -IntegrityCheckMethod SHA256 \
  -DHGroup Group14 \
  -PfsGroup None \
  -AuthenticationTransformConstants None
```

..* IKE_AUTH

iOS device native IKEv2 with pre-share key
-----

Windows 10 Anyconnect client with EAP 
-----

Windows 10 native IKEv2 with Certification
-----
