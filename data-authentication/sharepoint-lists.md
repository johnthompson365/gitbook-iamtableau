---
description: How to integrate Tableau Desktop with ADFS
---

# Recipe: SharePoint Lists and ADFS

### Overview

Whereas connectivity to Sharepoint Lists using Third-party SSO is currently supported with OneLogin and Okta, you might be wondering where that leaves your organization if you are using ADFS? Well, there is a workable solution as the underlying CData driver has support for ADFS builtin. 

You need to test this in your environment to ensure it works and be confident in maintaining it; as it is not supported by Tableau. 

### The Docs

Tableau provides [step-by-step guidance ](https://help.tableau.com/current/pro/desktop/en-us/examples_sharepoint_lists.htm)on how to use the SharePoint Lists connector with Tableau Desktop, Online and Server.

### Desktop Setup

The instructions are basically the same as the Third Party SSO option in the link above. The connector will guide you to install the driver. Once you have that installed then you have to configure the connection properties. Here you put in the URL of the site \(not the list\), and the username and password that you will be connecting with. 

In my testing I did not require the SSO domain despite using a username from a domain that was different to the ADFS service's domain name.

![Connector properties](../.gitbook/assets/image%20%2864%29.png)

### Under the hood

Using Fiddler I can see that an initial GET is made to the Site URL 

![](../.gitbook/assets/image%20%2863%29.png)

Which is rejected with a 401:

`WWW-Authenticate: IDCRL Type="BPOSIDCRL", EndPoint="/sites/tableau/_vti_bin/idcrl.svc/", RootDomain="sharepoint.com", Policy="MBI"`

The connector then POST's the username to the 'Office 365 Front Door' `login.microsoftonline.com` to do home realm discovery:

![](../.gitbook/assets/image%20%2858%29.png)

And receives an XML response that confirms the domain is federated and the endpoint on the ADFS server \(STSAuthURL\) that needs to be used, in this instance `usernamemixed`.

The `usernamemixed` endpoint is used by **WS-Trust** and traditionally was in use by Exchange Online Office clients in the Active federated authentication flow. Now all clients have moved to passive ADFS flows.

![](../.gitbook/assets/image%20%2866%29.png)

The connector authenticates against the `usernamemixed` endpoint by sending transport encrypted SOAP envelope, passing the Username Token \(username and password\) to request the SAML security token. The response then includes the SAML attributes and claims:

![Claims...](../.gitbook/assets/image%20%2879%29.png)

The connector then makes a request to Azure AD for an access token using the SAML token from ADFS

![](../.gitbook/assets/image%20%2877%29.png)

The response includes the Azure AD authentication token.

![](../.gitbook/assets/image%20%2876%29.png)

The token is then sent as part of the headers to access SharePoint Online

You will receive a 200 \(OK\) response with the authentication cookie **SPOIDCRL** in the header.

![](../.gitbook/assets/image%20%2878%29.png)

And importantly your desktop client finally accessing Sharepoint Lists.

![BOOM!](../.gitbook/assets/image%20%2873%29.png)

### Server Setup

Install the driver \(see above\).

* [ ] _Add user to OneLogin_
* [ ] _Add user to Tableau_
* [ ] _Test sign in to server and license_
* [ ] _Create a data source and publish it_



### Useful reference

[https://sharepoint.stackexchange.com/questions/222917/rest-authentication-to-online-sharepoint](https://sharepoint.stackexchange.com/questions/222917/rest-authentication-to-online-sharepoint)

[https://help.tableau.com/v2020.4/server/en-us/creator\_connect.htm](https://help.tableau.com/v2020.4/server/en-us/creator_connect.htm) - SharePoint Lists are not a supported connector on Server. "If the connector you need doesn't appear in the Connectors tab, you can connect to data through Tableau Desktop and publish your data source to Tableau Online or Tableau Server for web authoring. Learn more about how to [Publish a Data Source in Tableau Desktop](https://help.tableau.com/current/pro/desktop/en-us/publish_datasources.htm)."



