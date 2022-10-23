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

For my lab I want the simplest configuration so want to be able to use the same certificate for both the server **and** SAML. The relevant requirements for me are:

1. ...Confirm that the certificate includes only the certificate that applies to Tableau Server and not any other certificates or keys.
2. SSL certificate chain file: A certificate chain file is required for Tableau Desktop on the Mac and for Tableau Prep Builder on the Mac and Tableau Prep Builder on Windows. The chain file is also required for the Tableau Mobile app if the certificate chain for Tableau Server is not trusted by the iOS or Android operating system on the mobile device.
   * **How to check:** open the certificate and check the certificates are in order or... `openssl crl2pkcs7 -nocrl -certfile $CERT_FILE | openssl pkcs7 -print_certs -noout3`
3. You must either use a PCKS#1 (no password) or a PCKS#8 (with password) as described [here](https://help.tableau.com/current/server/en-us/saml\_requ.htm#certificate-and-identity-provider-idp-requirements).&#x20;
   * **How to check:** You can check the format of the key simply by opening it shown [here](https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key)
4. All certificate files must be valid PEM-encoded X509 certificates with the extension `.crt`.

If you are running Windows follow the Tutorial linked below to get your certificates:

{% embed url="https://poshac.me/docs/v4/Tutorial#tutorial" %}
Great bit of software
{% endembed %}

Posh-ACME provides you with everything you need and stores the certificates in `%LOCALAPPDATA%\Posh-ACME` on Windows or `~/.config/Posh-ACME` on Linux

* **cert.cer (Base64 encoded PEM certificate)**&#x20;
* **cert.key (Base64 encoded PEM private key)**&#x20;
* **chain.cer (Base64 encoded PEM with the issuing CA chain)**&#x20;
* cert.pfx (PKCS12 container with cert+key)
* chainX.cer (Base64 encoded PEM with alternate issuing CA chains)&#x20;
* fullchain.cer (Base64 encoded PEM with cert+chain)&#x20;
* fullchain.pfx (PKCS12 container with cert+key+chain)

If you are using Linux then knock yourself out and try [certbot](https://certbot.eff.org/).

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

[https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key](https://www.misterpki.com/pkcs8/)



## Independent Gateway Requirements

### Benefits and Limitations

The Tableau Server Independent Gateway (TSIG) is a reverse proxy server and simple load balancer based on Apache _HTTPd_. The benefits of it are:

* Can be deployed in DMZ or separate network segment to your Tableau Servers
* It is managed by TSM so is Tableau Cluster aware
* Supports multiple instances for HA

The guidance for Enterprise customers is that as TSIG only supports Round Robin load balancing it is not designed as an Enterprise load balancing service so you should front the gateway with one if you require.&#x20;

### Design Choices

Focusing on the purpose of this article is describe the need or nuances for TLS and Certificates with Tableau Server Independent Gateway (TSIG).&#x20;

With TSIG there is a decision to make about whether to enforce end-to-end encryption or terminate TLS at the gateway. The considerations are listed [here](https://help.tableau.com/current/guides/enterprise-deployment/en-us/edg\_part5.htm#independent-gateway-direct-vs-relay-connection):

* **Direct Connection:** The traffic flow must have the TLS terminated at the TSIG direct and then http sent to the Tableau Server background processes. It has less hops but is not encrypted end-to-end and requires a number of ports being opened.&#x20;
* **Relay Connection:** The traffic flow goes from TSIG to the Server Gateway process and then to the backend processes. This is an additional but only requires a single port for operations to be opened and you get end-to-end encryption.

### Certificate Requirements

Please refer to the official specific guidance for TSIG Certificate Requirements article is [here](https://help.tableau.com/current/server/en-us/server\_tsig\_configure\_tls.htm#certificate-requirements-and-considerations)

But the main thing to remember is...

_**The certificate requirements for Independent Gateway are the same as those specified for Tableau Server "external SSL."**_

So you just need to follow the normal articles [above](draft-tableau-certificates-and-my-memory.md#tableau-official-guidance).

* Chain certificate for 'thick clients' etc... which is explained in this [KB](https://kb.tableau.com/articles/HowTo/configure-tls-on-independent-gateway-when-using-intermediate-certificate) for TSIG

### Configuration

There is an extensive article that walks through the TLS Requirements and configuration for TLS to each part of the topology ultimately providing end-end encryption if you need it. [Configure TLS on Independent Gateway](https://help.tableau.com/current/server/en-us/server\_tsig\_configure\_tls.htm)

There are three points of TLS configuration:

* From the external network (internet or front-end load balancer) to Independent Gateway
* From Independent Gateway to Tableau Server
* For housekeeping (HK) process from Tableau Server to Independent Gateway

Here is a detailed example of [how to configure TLS for TSIG in the Tableau Enterprise Deployment Guide in AWS](https://help.tableau.com/current/guides/enterprise-deployment/en-us/edg\_part6.htm#configure-ssltls-from-load-balancer-to-tableau-server)

### Considerations and Actions

| Consideration                                                                                                                                                                                                                                                   | Actions                                                                                                                                                                                                                                                                                 |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| By default, the Independent Gateway must be able to communicate with the backend Tableau Server deployment on ports 80 and 21319 during installation.                                                                                                           | To change the HK and TSIG ports after installation, you can re-run the post-install script to provide a different value for `TSIG_HK_PORT`. By default, the script is at `C:\Program Files\Tableau\Tableau Server\independentgateway\scripts\initialize-tsig.bat`.                      |
| TSM does not automatically distribute certificate and key material to Independent Gateway nodes. (as opposed to Tableau Server where TSM automatically distributes them)                                                                                        | Manually copy the certificate and separate key files to each relevant gateway server. Secure the permissions to the key files so that only _tsig-http_ service has read-only access. Ensure it is saved outside of _TSIG\_INSTALL_ and _TSIG\_DATA_ paths to avoid overwrite on upgrade |
| As with all TLS-related files on Independent Gateway computers, you must put the files in the same paths on each computer. All file names for TLS shared files must also be the same.                                                                           | Define a consistent path and file names for each TSIG                                                                                                                                                                                                                                   |
| If you use a non-Public CA and want end-to-end encryption then the Root CA is needed on Tableau Server                                                                                                                                                          | Copy Private PKI Root CA file to each Tableau Server                                                                                                                                                                                                                                    |
| When using a non-Public CA, you may only upload one root CA certificate to Tableau Server. Therefore, if you have already uploaded a root CA certificate, then the same root CA certificate must sign the certificate that you will be using for HK connection. | In a non-public CA scenario, the certificate that you use to secure the  TSIG <-> TS connection must also come from the same Root CA as the one you use to secure the HK <-> TSIG connections.                                                                                          |
| If you have enabled TLS with external network and Independent Gateway, you may use the same certificate and key files for the HK connection                                                                                                                     | Single Certificate, Keys and Root CA can be used for External to TSIG and HK to TSIG connections.                                                                                                                                                                                       |
| You can configure Mutual TLS between the TSIG and Tableau Servers, and the House Keeping Services and the TSIG.                                                                                                                                                 | Enable the configuration on both TSIG (`gateway.tsig.ssl.client_certificate_login.required`) and Tableau Server (g`ateway.tsig.ssl.proxy.machinecertificatefile`) and the HK (`gateway.tsig.hk.ssl.client_certificate_login.required`)                                                  |
| TSIG does not automatically provide a way to supply the optional TLS key passphrase on startup.                                                                                                                                                                 | Use the TSM config key  _`gateway.tsig.ssl.key.passphrase.dialog` to_ specify the passphrase for the key                                                                                                                                                                                |
| HK port by default is 21319                                                                                                                                                                                                                                     | Can be left, or configured using a batch file (old skool) `C:\Program Files\Tableau\Tableau Server\independentgateway\scripts\initialize-tsig.bat`                                                                                                                                      |

&#x20;

Everything should pretty much go perfectly well first time; but just in case it doesn't... use this handy article - [Troubleshooting TSIG](https://help.tableau.com/current/guides/enterprise-deployment/en-us/edg\_part7.htm#troubleshooting-tableau-server-independent-gateway).

## Updating your certificate

Click Reset and go through the standard steps of uploading your certificate files (cert, key and chain) to TSM. Yes, it needs a restart.

![](<.gitbook/assets/image (130).png>)

## Namespace Considerations

{% embed url="https://help.tableau.com/current/server/en-us/ssl_config.htm#change-or-update-ssl-certificate" %}

Tableau clients that need to access the server can use [subject alternative names](https://help.tableau.com/current/server/en-us/ssl\_config.htm#ssl-certificate-requirements) defined in the certificate. So as long as you manage the DNS you can have different names for clients to initially connect to ([internal.example.com](http://internal.example.com/), [external.example.com](http://external.example.com/), [desktop.example.com](http://desktop.example.com/)) the server.&#x20;

However, if you have configured SAML then whatever you define as the SAML URLs (return URL and Entity ID) are what becomes the Server URL once the client has logged on, for both internal and external users and… _if you plan to enable site-specific SAML later, this URL also serves as the base for each site’s unique ID._

![Desktop me.](<.gitbook/assets/image (131) (1).png>)

__

![SAML Uber Alles](.gitbook/assets/2021-11-12\_16-35-02.png)

## ISSUE: SSLHandshakeException

I have seen this error a few times.&#x20;

<figure><img src=".gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

Remember the guidance from :point\_up:

If you are using the Tableau clients you need a Certificate Chain file. That means that your certificate file should not only include the server certificate but also the intermediate certificate.&#x20;

It can have a different root causes. We have a KB article to follow [here](https://kb.tableau.com/articles/en\_US/Issue/ssl-handshake-exception-or-pkix-path-building-failed-upon-signing-in-tableau-server-from-tableau-prep-or-tableau-desktop) which solves if the problem is from converting .pem to .crt.&#x20;

However I have seen some customers not use a chain file at all. It is easy to do a basic check of this by just opening the file and checking the 2 certs match the distinct server and intermediate files. Or you can use a slightly more scientific method that I list above as well:

`openssl crl2pkcs7 -nocrl -certfile $CERT_FILE | openssl pkcs7 -print_certs -noout3`

Your Certificate Authority should have provided a chain certificate so if you only have the server certificate either reach back out to your CA or whoever manages it.



## ISSUE: Certificate file not uploading

I attempted to update the cert.key a few times using the SAML configuration tsm command

`tsm authentication saml configure --idp-entity-id  https://tabwin-tfvm.developatribe.com --idp-return-url https://tabwin-tfvm.developatribe.com --cert-file cert.crt --key-file nopasscert.key`&#x20;

However, when I checked the key file I found it was updating even though TSM said the file upload was successful.

![](<.gitbook/assets/image (129).png>)

![](<.gitbook/assets/image (128).png>)

A quick workaround was to just copy over the file to the location shown above and restart services. Not sure how supported that is but worked for me!
