---
description: How to use POSH-ACME certs with Tableau and other tall tales.
---

# Tableau Server, SAML Certificates, namespaces and my memory...

The instructions on certificate usage for Tableau server and SAML are not memorable to me. Not much is, so I need to write this stuff down.

## Tableau Official Guidance

The two docs that give the full details are here:

{% embed url="https://help.tableau.com/current/server/en-us/ssl_config.htm" %}

{% embed url="https://help.tableau.com/current/server/en-us/saml_requ.htm#Cert_Name" %}

## My Lab requirements

For my lab I want the simplest configuration so want to be able to use the same certificate for both the server **and **SAML. The relevant requirements for me are:

1. ...Confirm that the certificate includes only the certificate that applies to Tableau Server and not any other certificates or keys.
2. SSL certificate chain file: A certificate chain file is required for Tableau Desktop on the Mac and for Tableau Prep Builder on the Mac and Tableau Prep Builder on Windows. The chain file is also required for the Tableau Mobile app if the certificate chain for Tableau Server is not trusted by the iOS or Android operating system on the mobile device.
   * **How to check:** open the certificate and check the certificates are in order or... `openssl crl2pkcs7 -nocrl -certfile $CERT_FILE | openssl pkcs7 -print_certs -noout3`
3. You must either use a PCKS#1 (no password) or a PCKS#8 (with password) as described [here](https://help.tableau.com/current/server/en-us/saml\_requ.htm#certificate-and-identity-provider-idp-requirements).&#x20;
   * **How to check: **You can check the format of the key simply by opening it shown [here](https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key)
4. All certificate files must be valid PEM-encoded X509 certificates with the extension `.crt`.

If you are running Windows follow the Tutorial linked below to get your certificates:

{% embed url="https://poshac.me/docs/v4/Tutorial#tutorial" %}
Great bit of software
{% endembed %}

Posh-ACME provides you with everything you need and stores the certificates in `%LOCALAPPDATA%\Posh-ACME` on Windows or `~/.config/Posh-ACME` on Linux

* **cert.cer (Base64 encoded PEM certificate) **
* **cert.key (Base64 encoded PEM private key) **&#x20;
* **chain.cer (Base64 encoded PEM with the issuing CA chain) **
* cert.pfx (PKCS12 container with cert+key)
* chainX.cer (Base64 encoded PEM with alternate issuing CA chains)&#x20;
* fullchain.cer (Base64 encoded PEM with cert+chain)&#x20;
* fullchain.pfx (PKCS12 container with cert+key+chain)

If you are using Linux then knock yourself out and try [certbot](https://certbot.eff.org).

{% hint style="info" %}
You can just rename the `.cer` file to be `.crt`&#x20;
{% endhint %}

####

## SiteSAML requirements

I attempted to use the same Posh-ACME certificates for both the Server TLS and SAML. This worked without issue for Tableau Server-Wide SAML configuration. It did not work for Site SAML. I received the following error:

> Expected private key stored in C:/ProgramData/Tableau/Tableau Server/data/tabsvc/config/vizportal\_0.20201.20.0427.1803/files/samlkeyfile.key to be a PEMKeyPair (unencrypted PEM), but got PrivateKeyInfo instead

The confusing thing was that I knew the **cert.key** did not have an associated passphrase.

However the **cert.key** provided is PKCS#8. In our [SAML requirements](https://help.tableau.com/current/server/en-us/saml\_requ.htm) we state that:

> To use a password-protected key file, you must configure SAML with a RSA PKCS#8 file. Note that a PKCS#8 file with a null password is not supported.

To resolve this I needed to convert the PKCS#8 formatted private key to PKCS#1. There are a number of ways to do this:

```
openssl rsa -in cert.key -out nopasscert.key 
openssl pkcs8 -in cert.key -traditional -nocrypt -out pkcsfingerscrossed.key  
```

Useful docs reference docs I found were PKCS formats are:

[https://www.misterpki.com/pkcs8/ ](https://www.misterpki.com/pkcs8/)

[https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key\
](https://www.misterpki.com/pkcs8/)

### ISSUE: Certificate file not uploading

I attempted to update the cert.key a few times using the SAML configuration tsm command

`tsm authentication saml configure --idp-entity-id  https://tabwin-tfvm.developatribe.com --idp-return-url https://tabwin-tfvm.developatribe.com --cert-file cert.crt --key-file nopasscert.key `

However, when I checked the key file I found it was updating even though TSM said the file upload was successful.

![](<.gitbook/assets/image (129).png>)

![](<.gitbook/assets/image (128).png>)

A quick workaround was to just copy over the file to the location shown above and restart services. Not sure how supported that is but worked for me!

## Updating your certificate

Click Reset and go through the standard steps of uploading your certificate files (cert, key and chain) to TSM. Yes, it needs a restart.

![](<.gitbook/assets/image (130).png>)

{% embed url="https://help.tableau.com/current/server/en-us/ssl_config.htm#change-or-update-ssl-certificate" %}

## Namespace Considerations

Tableau clients that need to access the server can use [subject alternative names](https://help.tableau.com/current/server/en-us/ssl\_config.htm#ssl-certificate-requirements) defined in the certificate. So as long as you manage the DNS you can have different names for clients to initially connect to ([internal.example.com](http://internal.example.com), [external.example.com](http://external.example.com), [desktop.example.com](http://desktop.example.com)) the server.&#x20;

However, if you have configured SAML then whatever you define as the SAML URLs (return URL and Entity ID) are what becomes the Server URL once the client has logged on, for both internal and external users and… _if you plan to enable site-specific SAML later, this URL also serves as the base for each site’s unique ID._

![Desktop me.](<.gitbook/assets/image (131).png>)

__

![SAML Uber Alles](.gitbook/assets/2021-11-12\_16-35-02.png)

__
