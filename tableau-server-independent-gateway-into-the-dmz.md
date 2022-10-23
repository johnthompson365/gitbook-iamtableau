---
description: Proxy me!
---

# Tableau Server Independent Gateway into the DMZ

## Introduction

This is a brief introduction to some of the important technical points regarding the new Tableau Server Independent Gateway. It is not intended to be exhaustive but focuses on the information needed to build this within a PoC environment and prove the networking and end-to-end TLS configurations. Always refer back to the Help documentation for latest updates.

## Benefits and Limitations

The Tableau Server Independent Gateway (TSIG) is a reverse proxy server and simple load balancer based on Apache _HTTPd_. The benefits of it are:

* Can be deployed in DMZ or separate network segment to your Tableau Servers
* It is managed by TSM so is Tableau Cluster aware
* Supports multiple instances for HA
* Supports Mutual TLS&#x20;

The guidance for Enterprise customers is that as TSIG only supports Round Robin load balancing it is not designed as an Enterprise load balancing service so you should front the gateway with one if you require.&#x20;

## Design Choices

Focusing on the purpose of this article is describe the need or nuances for TLS and Certificates with Tableau Server Independent Gateway (TSIG).&#x20;

With TSIG there is a decision to make about whether to enforce end-to-end encryption or terminate TLS at the gateway. The considerations are listed [here](https://help.tableau.com/current/guides/enterprise-deployment/en-us/edg\_part5.htm#independent-gateway-direct-vs-relay-connection):

* **Direct Connection:** The traffic flow must have the TLS terminated at the TSIG direct and then http sent to the Tableau Server background processes. It has less hops but is not encrypted end-to-end and requires a number of ports being opened.&#x20;
* **Relay Connection:** The traffic flow goes from TSIG to the Server Gateway process and then to the backend processes. This is an additional but only requires a single port for operations to be opened and you get end-to-end encryption.

## Certificate Requirements

Please refer to the official specific guidance for TSIG Certificate Requirements article is [here](https://help.tableau.com/current/server/en-us/server\_tsig\_configure\_tls.htm#certificate-requirements-and-considerations)

But the main thing to remember is...

_**The certificate requirements for Independent Gateway are the same as those specified for Tableau Server "external SSL."**_

So you just need to follow the normal articles [above](tableau-server-independent-gateway-into-the-dmz.md#tableau-official-guidance).

* Chain certificate for 'thick clients' etc... which is explained in this [KB](https://kb.tableau.com/articles/HowTo/configure-tls-on-independent-gateway-when-using-intermediate-certificate) for TSIG

## Configuration

There is an extensive article that walks through the TLS Requirements and configuration for TLS to each part of the topology ultimately providing end-end encryption if you need it. [Configure TLS on Independent Gateway](https://help.tableau.com/current/server/en-us/server\_tsig\_configure\_tls.htm)

There are three points of TLS configuration:

* From the external network (internet or front-end load balancer) to Independent Gateway
* From Independent Gateway to Tableau Server
* For housekeeping (HK) process from Tableau Server to Independent Gateway

Here is a detailed example of [how to configure TLS for TSIG in the Tableau Enterprise Deployment Guide in AWS](https://help.tableau.com/current/guides/enterprise-deployment/en-us/edg\_part6.htm#configure-ssltls-from-load-balancer-to-tableau-server)

## Considerations and Actions

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

&#x20;

Everything should pretty much go perfectly well first time; but just in case it doesn't... use this handy article - [Troubleshooting TSIG](https://help.tableau.com/current/guides/enterprise-deployment/en-us/edg\_part7.htm#troubleshooting-tableau-server-independent-gateway).
