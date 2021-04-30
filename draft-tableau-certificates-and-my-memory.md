---
description: How to use POSH-ACME certs with Tableau
---

# Draft: Server & SAML Certificates and my memory

The instructions on certificate usage for Tableau server and SAML are not memorable to me. Not much is, so I need to write this stuff down.

### Tableau Official Guidance

The two docs that give the full details are here:

{% embed url="https://help.tableau.com/current/server/en-us/ssl\_config.htm" %}

{% embed url="https://help.tableau.com/current/server/en-us/saml\_requ.htm\#Cert\_Name" %}

### My Lab requirements

For my lab I want the simplest configuration so want to be able to use the same certificate for both the server **and** SAML. The relevant requirements for me are:

1. ...if you want to use the same certificate for SSL and SAML, you must use a key file that is _not_ passphrase protected.
2. ...Confirm that the certificate includes only the certificate that applies to Tableau Server and not any other certificates or keys.
3. SSL certificate chain file: A certificate chain file is required for Tableau Desktop on the Mac and for Tableau Prep Builder on the Mac and Tableau Prep Builder on Windows. The chain file is also required for the Tableau Mobile app if the certificate chain for Tableau Server is not trusted by the iOS or Android operating system on the mobile device.



If you are running Windows follow the Tutorial linked below to get your certificates:

{% embed url="https://github.com/rmbolger/Posh-ACME/blob/main/Tutorial.md" %}

Posh-ACME provides you with everything you need:

* **cert.cer \(Base64 encoded PEM certificate\)** 
* **cert.key \(Base64 encoded PEM private key\)** 
* cert.pfx \(PKCS12 container with cert+key\) 
* **chain.cer \(Base64 encoded PEM with the issuing CA chain\)** 
* chainX.cer \(Base64 encoded PEM with alternate issuing CA chains\) 
* fullchain.cer \(Base64 encoded PEM with cert+chain\) 
* fullchain.pfx \(PKCS12 container with cert+key+chain\)

If you are using Linux then knock yourself out with then try [certbot](https://certbot.eff.org/).

### Updating your certificate

Click Reset and go through the standard steps of uploading your certificate files. Yes, it needs a restart.

{% embed url="https://help.tableau.com/current/server/en-us/ssl\_config.htm\#change-or-update-ssl-certificate" %}



