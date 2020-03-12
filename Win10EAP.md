Windows 10 native IKEv2 with EAP authentication
======

This blog is to describe how to setup windows 10 as client to connect IKEv2 VPN. 
Because windows 10 don't support pre-share key, we will use EAP and Certification to demo this. 

Setup Windows 2016 as CA server
-----
+ Follow this [link](https://docs.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/install-the-certification-authority) to setup Windows 2016 as Certificate Authority and with “Network Device Enrollment Service”.

+ Clone certificate template. Windows CA default certificate template is "IPSECIntemediateOffline". Based on this [link](https://www.cisco.com/c/en/us/support/docs/security/flexvpn/115907-config-flexvpn-wcca-00.html), certificate must have the EKU fields set to 'Server Authentication' for Cisco IOS and 'Client Authentication' for the client. We clone a template that support both EKU. 

+ Change MSCEP default template and disable SCEP enrollment password.



CSR1000v Certification enrollment
------

CSR1000v IKEv2 setup and AAA setup
------

Windows 10 client setup
------

