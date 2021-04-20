---
description: How to integrate with ADFS
---

# SharePoint Lists

## The Docs

Tableau provides [step-by-step guidance ](https://help.tableau.com/current/pro/desktop/en-us/examples_sharepoint_lists.htm)on how to use the SharePoint Lists connector with Tableau Desktop, Online and Server.

### Support

Currently the supported Identity Providers that you can use with Third Party SSO are Okta and OneLogin. If you are an organization with ADFS then you may wonder where that leaves you. 

Well, there is a workable solution you would need to test in your environment; as the underlying CData driver has support for ADFS builtin.

### Desktop Setup

The instructions are basically the same as the Third Party SSO option in the link above. The connector will guide you to install the driver. Once you have that installed then you have to configure the connection properties. Here you put in the URL of the site \(not the list\), and the username and password that you will be connecting with.

![Connector properties](../.gitbook/assets/image%20%2864%29.png)

### Under the hood

Using Fiddler I can see that an initial GET is made to the Site URL 

![](../.gitbook/assets/image%20%2863%29.png)

Which is rejected with a 401.

`WWW-Authenticate: IDCRL Type="BPOSIDCRL", EndPoint="/sites/tableau/_vti_bin/idcrl.svc/", RootDomain="sharepoint.com", Policy="MBI"`

[https://sharepoint.stackexchange.com/questions/222917/rest-authentication-to-online-sharepoint](https://sharepoint.stackexchange.com/questions/222917/rest-authentication-to-online-sharepoint)

The connector then POST's the username to do home realm discovery

![](../.gitbook/assets/image%20%2858%29.png)

And receives an XML response that confirms the domain is federated and the endpoint on the ADFS server that needs to be used, in this instance `usernamemixed`:

![](../.gitbook/assets/image%20%2866%29.png)

The `usernamemixed` endpoint is used by **WS-Federation** and traditionally was in use by Exchange Online Office clients \(older than Office 2013 May 2015 update\), in the legacy federated authentication flow. NOw all clients have moved to passive ADFS flows.

The connector authenticates against the `usernamemixed` endpoint by passing the Username Token \(username and password\) as part of the encrypted SOAP envelope, and requests the SAML security token.

The response then includes the SAML attributes and claims to pass back to Azure AD and gain access to Sharepoint Online. 

![](../.gitbook/assets/image%20%2865%29.png)

