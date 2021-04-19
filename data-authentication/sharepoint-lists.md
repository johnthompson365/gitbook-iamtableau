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

![Connector properties](../.gitbook/assets/image%20%2853%29.png)

### Under the hood



