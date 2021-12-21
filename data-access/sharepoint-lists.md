---
description: How it works under the hood.
---

# Recipe: SharePoint Lists and ADFS

### Overview

Whereas connectivity to Sharepoint Lists using Third-party SSO is currently supported with OneLogin and Okta, you might be wondering where that leaves your organization if you are using ADFS? Well, there is a workable solution as the underlying CData driver has support for ADFS builtin.&#x20;

You need to test this in your environment to ensure it works and be confident in maintaining it; as it is not supported by Tableau.&#x20;

### The Docs

Tableau provides [step-by-step guidance ](https://help.tableau.com/current/pro/desktop/en-us/examples\_sharepoint\_lists.htm)on how to use the SharePoint Lists connector with Tableau Desktop, Online and Server.

### Desktop Setup

The instructions are basically the same as the Third Party SSO option in the link above. The connector will guide you to install the driver. Once you have that installed then you have to configure the connection properties. Here you put in the URL of the site (not the list), and the username and password that you will be connecting with.&#x20;

In my testing I did not require the SSO domain despite using a username from a domain that was different to the ADFS service's domain name.

![Connector properties](<../.gitbook/assets/image (64).png>)

### Server Setup

* [x] Install the driver (see above).
* [x] Publish your data source to server
* [x] Embed credentials if you want to configure a refresh
* [x] Connect to data source

![](<../.gitbook/assets/image (86).png>)

There is a Test Connection option to validate everything:

![](<../.gitbook/assets/image (85).png>)

### Under the hood

Using Fiddler I can see that an initial GET is made to the Site URL&#x20;

![](<../.gitbook/assets/image (63).png>)

Which is rejected with a 401:

`WWW-Authenticate: IDCRL Type="BPOSIDCRL", EndPoint="/sites/tableau/_vti_bin/idcrl.svc/", RootDomain="sharepoint.com", Policy="MBI"`

The connector then POST's the username to the 'Office 365 Front Door' `login.microsoftonline.com` to do home realm discovery:

![](<../.gitbook/assets/image (58).png>)

And receives an XML response that confirms the domain is federated and the endpoint on the ADFS server (STSAuthURL) that needs to be used, in this instance `usernamemixed`.

The `usernamemixed` endpoint is used by **WS-Trust** and traditionally was in use by Exchange Online Office clients in the Active federated authentication flow. Now all clients have moved to passive ADFS flows.

![](<../.gitbook/assets/image (66).png>)

The connector authenticates against the `usernamemixed` endpoint by sending transport encrypted SOAP envelope, passing the Username Token (username and password) to request the SAML security token. The response then includes the SAML attributes and claims:

![Claims...](<../.gitbook/assets/image (79).png>)

The connector then makes a request to Azure AD for an access token using the SAML token from ADFS

![](<../.gitbook/assets/image (77).png>)

The response includes the Azure AD authentication token.

![](<../.gitbook/assets/image (76).png>)

The token is then sent as part of the headers to access SharePoint Online

You will receive a 200 (OK) response with the authentication cookie **SPOIDCRL** in the header.

![](<../.gitbook/assets/image (78).png>)

And importantly your desktop client finally accessing Sharepoint Lists.

![BOOM!](<../.gitbook/assets/image (73).png>)



![](<../.gitbook/assets/image (85).png>)

### Limitations

SharePoint Online has the ability to turn off protocols that are viewed as Legacy by Azure AD. When contacting SharePoint Online The response status code is 'Unauthorized'.", and the underlying error is **"Access denied. Before opening files in this location, you must first browse to the web site and select the option to login automatically."**

![](<../.gitbook/assets/image (87).png>)

My current setting was for LegacyAuthProtocolsEnabled was set to True and I am able to access the data.

#### Testing

![](<../.gitbook/assets/image (84).png>)

![You need those Legacy protocols](<../.gitbook/assets/image (88).png>)

In Fiddler I receive the 403 and see a **X-Forms\_Based\_Auth\_Required** and the **X-MSDAVEXT\_Error** in the headers. So ensure that `SetSPOTenant -LegacyAuthProtocolsEnabled $True` for this connector to work.

Another limitation of this connector is the lack of support for MFA. This is listed prominently on our help pages.

> **Note:** Multi-factor authentication (MFA) is not supported by the drivers currently available for Sharepoint Lists.\
>

### Useful reference

[https://sharepoint.stackexchange.com/questions/222917/rest-authentication-to-online-sharepoint](https://sharepoint.stackexchange.com/questions/222917/rest-authentication-to-online-sharepoint)

[https://help.tableau.com/v2020.4/server/en-us/creator\_connect.htm](https://help.tableau.com/v2020.4/server/en-us/creator\_connect.htm) - SharePoint Lists are not a supported connector on Server. "If the connector you need doesn't appear in the Connectors tab, you can connect to data through Tableau Desktop and publish your data source to Tableau Online or Tableau Server for web authoring. Learn more about how to [Publish a Data Source in Tableau Desktop](https://help.tableau.com/current/pro/desktop/en-us/publish\_datasources.htm)."

[https://blog.areflyen.no/2017/06/18/problem-with-connecting-to-sharepoint-online-in-office-365-with-powershell-sharepoint-designer-and-other-3-party-tools/](https://blog.areflyen.no/2017/06/18/problem-with-connecting-to-sharepoint-online-in-office-365-with-powershell-sharepoint-designer-and-other-3-party-tools/)

`X-MSDAVEXT_Error: 917656; Access+denied.+Before+opening+files+in+this+location%2c+you+must+first+browse+to+the+web+site+and+select+the+option+to+login+automatically.`

