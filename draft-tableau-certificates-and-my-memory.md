---
description: How to use POSH-ACME certs with Tableau
---

# Notes: Server & SAML Certificates and my memory

The instructions on certificate usage for Tableau server and SAML are not memorable to me. Not much is, so I need to write this stuff down.

## Tableau Official Guidance

The two docs that give the full details are here:

{% embed url="https://help.tableau.com/current/server/en-us/ssl\_config.htm" %}

{% embed url="https://help.tableau.com/current/server/en-us/saml\_requ.htm\#Cert\_Name" %}

## My Lab requirements

For my lab I want the simplest configuration so want to be able to use the same certificate for both the server **and** SAML. The relevant requirements for me are:

1. ...if you want to use the same certificate for SSL and SAML, you must use a key file that is _not_ passphrase protected.
2. ...Confirm that the certificate includes only the certificate that applies to Tableau Server and not any other certificates or keys.
3. SSL certificate chain file: A certificate chain file is required for Tableau Desktop on the Mac and for Tableau Prep Builder on the Mac and Tableau Prep Builder on Windows. The chain file is also required for the Tableau Mobile app if the certificate chain for Tableau Server is not trusted by the iOS or Android operating system on the mobile device.
4. All certificate files must be valid PEM-encoded X509 certificates with the extension `.crt`.

If you are running Windows follow the Tutorial linked below to get your certificates:

{% embed url="https://github.com/rmbolger/Posh-ACME/blob/main/Tutorial.md" %}

{% hint style="info" %}
Remember to delete your TXT records after you have created them
{% endhint %}

Posh-ACME provides you with everything you need and stores the certificates in `%LOCALAPPDATA%\Posh-ACME` on Windows or `~/.config/Posh-ACME` on Linux

* **cert.cer \(Base64 encoded PEM certificate\)** 
* **cert.key \(Base64 encoded PEM private key\)** 
* cert.pfx \(PKCS12 container with cert+key\) 
* **chain.cer \(Base64 encoded PEM with the issuing CA chain\)** 
* chainX.cer \(Base64 encoded PEM with alternate issuing CA chains\) 
* fullchain.cer \(Base64 encoded PEM with cert+chain\) 
* fullchain.pfx \(PKCS12 container with cert+key+chain\)

If you are using Linux then knock yourself out and try [certbot](https://certbot.eff.org/).

{% hint style="info" %}
You can just rename the `.cer` file to be `.crt` 
{% endhint %}

## SiteSAML requirements

I attempted to use the same Posh-ACME certificates for both the Server TLS and SAML. This worked without issue for Tableau Server-Wide SAML configuration. It did not work for Site SAML. I received the following error:

> Expected private key stored in C:/ProgramData/Tableau/Tableau Server/data/tabsvc/config/vizportal\_0.20201.20.0427.1803/files/samlkeyfile.key to be a PEMKeyPair \(unencrypted PEM\), but got PrivateKeyInfo instead

The confusing thing was that I knew the **cert.key** did not have an associated passphrase.

However the **cert.key** provided is PKCS\#8. In our [SAML requirements](https://help.tableau.com/current/server/en-us/saml_requ.htm) we state that:

> To use a password-protected key file, you must configure SAML with a RSA PKCS\#8 file. Note that a PKCS\#8 file with a null password is not supported.

To resolve this I needed to convert the PKCS\#8 formatted private key to PKCS\#1. There are a number of ways to do this:

```text
openssl rsa -in cert.key -out nopasscert.key 
openssl pkcs8 -in cert.key -traditional -nocrypt -out pkcsfingerscrossed.key  
```

Useful docs reference docs I found were:

[https://www.misterpki.com/pkcs8/ ](%20https://www.misterpki.com/pkcs8/%20
)

[https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key  
](%20https://www.misterpki.com/pkcs8/%20
)

### Certificate file not uploading

I attempted to update the cert.key a few times using the SAML configuration tsm command

`tsm authentication saml configure --idp-entity-id  https://tabwin-tfvm.developatribe.com --idp-return-url https://tabwin-tfvm.developatribe.com --cert-file cert.crt --key-file nopasscert.key` 

However, when I checked the key file I found it was updating even though TSM said the file upload was successful.

![](.gitbook/assets/image%20%28129%29.png)

![](.gitbook/assets/image%20%28128%29.png)

A quick workaround was to just copy over the file to the location shown above and restart services. Not sure how supported that is but worked for me!

## Updating your certificate

Click Reset and go through the standard steps of uploading your certificate files. Yes, it needs a restart.

{% embed url="https://help.tableau.com/current/server/en-us/ssl\_config.htm\#change-or-update-ssl-certificate" %}



