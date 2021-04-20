---
description: How to integrate Tableau Desktop with ADFS
---

# Recipe: SharePoint Lists and ADFS

### Overview

Currently the supported Identity Providers that you can use with Third Party SSO are Okta and OneLogin. If you are an organization with ADFS and federated domains then you may wonder where that leaves you. 

Well, there is a workable solution as the underlying CData driver has support for ADFS builtin. You would need to test this in your environment to ensure it works in your scenario.

### The Docs

Tableau provides [step-by-step guidance ](https://help.tableau.com/current/pro/desktop/en-us/examples_sharepoint_lists.htm)on how to use the SharePoint Lists connector with Tableau Desktop, Online and Server.

### Desktop Setup

The instructions are basically the same as the Third Party SSO option in the link above. The connector will guide you to install the driver. Once you have that installed then you have to configure the connection properties. Here you put in the URL of the site \(not the list\), and the username and password that you will be connecting with. 

In my testing I did not require the SSO domain despite using a username from a domain that was different to the ADFS service's domain name.

![Connector properties](../.gitbook/assets/image%20%2864%29.png)

### Under the hood

Using Fiddler I can see that an initial GET is made to the Site URL 

![](../.gitbook/assets/image%20%2863%29.png)

Which is rejected with a 401.

`WWW-Authenticate: IDCRL Type="BPOSIDCRL", EndPoint="/sites/tableau/_vti_bin/idcrl.svc/", RootDomain="sharepoint.com", Policy="MBI"`

The connector then POST's the username to do home realm discovery

![](../.gitbook/assets/image%20%2858%29.png)

And receives an XML response that confirms the domain is federated and the endpoint on the ADFS server that needs to be used, in this instance `usernamemixed`:

![](../.gitbook/assets/image%20%2866%29.png)

The `usernamemixed` endpoint is used by **WS-Trust** and traditionally was in use by Exchange Online Office clients in the legacy Active federated authentication flow. Now all clients have moved to passive ADFS flows.

The connector authenticates against the `usernamemixed` endpoint by sending transport encrypted SOAP envelope, passing the Username Token \(username and password\) to request the SAML security token.

The response then includes the SAML attributes and claims:

![](../.gitbook/assets/image%20%2870%29.png)

 These are passed back to Azure AD to gain access to Sharepoint Online.

![BOOM!](../.gitbook/assets/image%20%2873%29.png)

