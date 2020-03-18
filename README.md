# IKEv2 VPN

Cisco Cloud Service Router as IKEv2 VPN server, Windows 2016 as Certification Authority and Network Policy Server, Windows 10 and iOS device as client. Test end to end IKEv2 VPN with pre-share key, EAP and Certification as authentication method. 

## Baisc knowledge of IKEv2

![](https://github.com/yinghli/IKEv2VPN/blob/master/IKEv2Basic.jpg)


### IKE_SA_INIT

Initiator and the responder will exchange the cryptographic algorithms and Diffie-Hellman group (DH). <br>
On CSR1000v, this information is defined by `crypto ikev2 proposal` command. 
```
crypto ikev2 proposal IKEv2-prop1 
 encryption aes-cbc-256
 integrity sha256
 group 14

crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac 
 mode tunnel
```
On Windows 10, default profile may not work, please follow [this](https://docs.microsoft.com/en-us/powershell/module/vpnclient/set-vpnconnectionipsecconfiguration?view=win10-ps) to modify via PowerShell. <br>
Below is an example that match CSR1000v IKEv2 and IPSec setup.
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

### IKE_AUTH

Initiator and the responder will reveal their identify. Then each device must authenticate their peer. Three methods of authentication are used in IKEv2: signature, pre-shared key, and EAP. <br>
On CSR1000v, this information is defined by `crypto ikev2 profile` command. Here is an example of pre-share key authentication.
```
crypto ikev2 profile AnyConnect-EAP
 match identity remote email router@cisco.com
 identity local fqdn ikev2.cisco.com
 authentication remote pre-share key cisco123
 authentication local pre-share key cisco123
 ```
> Windows 10 don't support pre-share key authenticatioin.
Below is iOS device IKEv2 VPN profile example. 

![](https://github.com/yinghli/IKEv2VPN/blob/master/iOS.jpg)

## Windows 10 Native IKEv2 Setup

[Example for Windows 10](https://github.com/yinghli/IKEv2VPN/blob/master/Win10EAP.md)

Reference Link:
+ [Cisco: IKEv2 with Windows 7 Certificate](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/115907-config-flexvpn-wcca-00.html) <br>
+ [Cisco: IKEv2 with strongSwan EAP](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/116837-config-strongswan-ios-00.html) <br>
+ [Cisco: AnyConnect IKEv2 with EAP-MD5](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/115755-flexvpn-ike-eap-00.html) <br>
+ [Cisco: VPN Radius Attribute](https://www.cisco.com/en/US/docs/ios-xml/ios/sec_conn_ike2vpn/configuration/15-2mt/sec-apx-flex-rad.html) <br>
+ [Cisco: IOS SSLVPN](https://community.cisco.com/t5/security-documents/configure-sslvpn-on-cisco-cloud-services-router-1000v-csr1000v/ta-p/3156679) <br>
+ [Cisco: WLC EAP-TLS](https://www.cisco.com/c/en/us/support/docs/wireless-mobility/wireless-lan-wlan/213543-configure-eap-tls-flow-with-ise.html) <br>
+ [FreeRadius EAP-TLS](https://documentation.meraki.com/MR/Encryption_and_Authentication/Freeradius%3A_Configure_freeradius_to_work_with_EAP-TLS_authentication) <br>
+ [Cisco IOS Certificate Enrollment](https://integratingit.wordpress.com/2017/08/26/cisco-ios-certificate-enrollment-via-scep/) <br>
