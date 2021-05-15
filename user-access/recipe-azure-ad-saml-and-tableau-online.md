---
description: '1kg of Azure AD, 500g of SAML, and 100g of TOL and 100g of TS'
---

# Recipe: Azure AD and Tableau

## Scope

Microsoft provides Azure AD apps that can be used to simplify the integration between Tableau Server and Tableau Online and Azure AD. The goal is to create a good onboarding and user experience to the Tableau services. It  explains the attributes required by the service to authenticate and plays around with the configuration to demonstrate how it works. 

## Features

The two apps have a different feature sets. The Tableau Online application supports the following two features, whereas Tableau Server only supports SSO.

1. SP-initiated SSO
2. _REST API user provisioning_

Neither apps support [IdP-initiated sign-on](https://duo.com/blog/the-beer-drinkers-guide-to-saml). This means that if you publish the app in the Azure MyApps portal it will still do an SP-initiated Authentication request and therefore have the usual browser redirections for that flow. Also, SP-Initiated Single Logout \(SLO\) is not possible due to our use of HTTP-Post bindings.

#### M365 Developer Program

{% embed url="https://developer.microsoft.com/en-us/microsoft-365/dev-program" %}

This gives you a free renewable 90 day M365 subscription, and M365 gives you Azure AD! Nice.

## Documentation

There are articles to configure the authentication steps. The Microsoft ones are little simpler to follow as they have screenshots.

**Microsoft Docs:**  
[Tutorial: Azure Active Directory single sign-on \(SSO\) integration with Tableau Online](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauonline-tutorial)  
[Tutorial: Azure Active Directory single sign-on \(SSO\) integration with Tableau Server](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauserver-tutorial)  
**Tableau Docs:**  
[Configure SAML with Azure Active Directory](https://help.tableau.com/current/online/en-us/saml_config_azure_ad.htm) \(Tableau Online\)  
[Configure Server-Wide SAML](https://help.tableau.com/current/server/en-us/config_saml.htm) \(Tableau Server\)

## Tableau Online SAML

As part of the SAML authentication flow attributes are passed between the IdP \(Azure\) and the Service Provider \(Tableau\). Getting them right is key to a successful SSO.

### Configuration

There are two main properties that TOL is interested in, your **Email** and **Display Name**. The Email attribute is mapped to the username in Tableau Online and must match a licensed user stored in the Tableau Server Repository. The Display Name maps to the Full Name field in Tableau Online, it is populated with the assertions for First name and Last name or Full name. ****If they are not provided in the authN flow then the email address is used, so are actually optional.

![](../.gitbook/assets/image%20%2868%29.png)

![TOL Attribute Configuration](../.gitbook/assets/image%20%2861%29.png)

### Claims

Claims are widely referred to in Azure AD but not really in other major IdP's like Okta or OneLogin as they tend to refer to attributes. Claims are information about user and groups that are shared between the identity provider and the service provider in the SAML token. In the SAML token they are usually contained in the Attribute Statement, so you only really need to consider them as attributes. A _claim type_ provides context for the _claim value_. It is usually expressed as a Uniform Resource Identifier \(URI\). This URI is what is required to be put into the TOL configuration.

[SAML Token Claims Reference](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-saml-tokens)

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Name</td>
      <td style="text-align:left">Provides a human readable value that identifies the subject of the token.
        This value is not guaranteed to be unique within a tenant and is designed
        to be used only for display purposes.</td>
      <td style="text-align:left">
        <p><code>&lt;Attribute Name=&quot;http://schemas.xmlsoap.org</code>
        </p>
        <p><code>/ws/2005/05/identity/claims/name&quot;&gt;</code>
          <br /><code>&lt;AttributeValue&gt;frankm@contoso.com&lt;AttributeValue&gt;</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">First Name</td>
      <td style="text-align:left">Provides the first or &quot;given&quot; name of the user, as set on the
        Azure AD user object.</td>
      <td style="text-align:left">
        <p><code>&lt;Attribute Name=&quot;http://schemas.</code>
        </p>
        <p><code>xmlsoap.org/ws/2005/05/identity/claims/givenname&quot;&gt;</code>
          <br
          /><code>&lt;AttributeValue&gt;Frank&lt;AttributeValue</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Last Name</td>
      <td style="text-align:left">Provides the last name, surname, or family name of the user as defined
        in the Azure AD user object.</td>
      <td style="text-align:left">
        <p><code>&lt;Attribute Name=&quot; http://schemas.xmlsoap.</code>
        </p>
        <p><code>org/ws/2005/05/identity/claims/surname&quot;&gt;</code>
          <br /><code>&lt;AttributeValue&gt;Miller&lt;AttributeValue&gt;</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

### Email

We ask for the Email attribute in the configuration which in turn is used as the TOL username and **cannot be changed**. The guidance we give on the Azure AD attributes to use has a couple of options:

* If all accounts you’re giving access to are sourced from **Microsoft accounts** \(e.g. outlook.com/gmail.com, or any consumer email domain that has signed up for an account\) this will be: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`
* If all accounts are sourced from **Microsoft Azure Active Directory**, use the following value:

  `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

It is not possible to map directly to the UPN claim as that is one of the [restricted claim sets ](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-claims-mapping#claim-sets)in Azure AD. These _can't be modified using policy. The data source cannot be changed, and no transformation is applied when generating these claims_. So in this instance that is why the Name claim is used instead.

> UPDATE: There is a lot of new functionality in preview for Azure AD to allow you to create claim types that was previously restricted. [How to: Customize claims emitted in tokens for a specific app in a tenant \(Preview\)](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-claims-mapping)

You can choose to map `mail` or`userPrincipalName` as the `Email` in TOL. However as there isn't a separate email address attribute in TOL whatever is defined as `Email` must be a working email address as that value is what will be used to send out the subscriptions you have setup to views or workbooks.

![Workbook subscriptions](../.gitbook/assets/image%20%2869%29.png)

### **Display Name**

Enter an assertion name for either the first name and last name, or for the full name, depending on how the IdP stores this information. Tableau Online uses these attributes to set the display name.

* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
* `http://schemas.microsoft.com/identity/claims/displayname`

So you can select either `givenname` + `surname` OR `displayname`

### The Minimum!

As it states in Azure, the only required claim is actually the Name ID. So I deleted all other for Name, Given Name etc.

![](../.gitbook/assets/image%20%2874%29.png)

The identifier format is email address and in this instance I have mapped it to the Source Attribute of userPrincipalName in the directory \(more information: [How to: customize claims issued in the SAML token for enterprise applications](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-claims-customization)\).

![](../.gitbook/assets/image%20%2872%29.png)

When logging on I received an error logging in:

_The site is configured to use the "_[_http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name_](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name)_" attribute as the username. Login failed because this is not included in the IdP Response. To fix this problem, the site admin must either choose the correct attribute or clear it to rely on the NameID._

By deleting the 'name' claim URI from the setting in Tableau Online \(shown below\) it forces SAML to rely on only the Name ID in the response.

![No claims defined for Email attribute](../.gitbook/assets/image%20%2875%29.png)

You can then login successfully with only relying on the name identifier, but you won't get any useful information like `First Name` and `Surname`.

### The Recommended!

I am primarily interested in Enterprise organizations so consumer accounts are less of a consideration. In Azure AD and Microsoft 365 the `userPrincipalName` \(UPN\) is the attribute that is used to sign in to Office 365 services. Many organizations are likely to want to map that to the username in TOL as they know it is likely to change less than the email address.   
  
So the best guidance is to ask:

* Is the`UPN` the same value as the users working email account, to receive subscriptions?
* Which out of the `mail` or `UPN` is more likely to change? 
  * If it did change, in Azure AD this will not be reflected in Tableau Online. 
  * [Follow these steps](https://kb.tableau.com/articles/howto/changing-tableau-online-user-name) to provision a new account, migrate content and delete old account.

### User Experience

The Azure AD Tableau Online app **always** uses a SP-initiated flow. This means that the user experience involves a number of redirects. There are some configuration options that can smooth this experience.

#### Remember me

You should advise your users to check the **Remember me** option when signing in to TOL. Without this checked the SAML flow will take the user back to the initial TOL SSO page to input the username before redirecting to the IdP.

![](../.gitbook/assets/image%20%289%29.png)

**Keep me signed In \(KMSI\)**

As part of the Azure AD flow you are asked whether you want to stay signed in to the service. I would also ensure users to select this to prevent them having to input credentials again for an extended period of time. 

## Tableau Server

The Tableau Server Azure AD app supports SP-initiated sign on. You need to manage provisioning separately.

### Configuration

Downloading the metadata file from Tableau Server and then uploading to Azure AD simplifies the first steps in configuring the IdP. This gives you the correct URLs at least in my instance:

![Your friendly URLs](../.gitbook/assets/image%20%28108%29.png)

### Do you have a claim?

The claims we define in our docs are:

![Our docs ask for this...](../.gitbook/assets/image%20%28114%29.png)

But if you go to the Server UI it asks you to match these assertions.

![The UI shows this...](../.gitbook/assets/image%20%28117%29.png)

The default claims defined in the Azure AD app actually just work and shouldn't require changing but include unnecessary claims.

![The default Azure app gives you this...](../.gitbook/assets/image%20%28107%29.png)

When you look closer at the claims it also shows the Name ID which actually contradicts our docs, but again works.

![](../.gitbook/assets/image%20%28116%29.png)

These are the results from SAML trace for a successful login.

![](../.gitbook/assets/image%20%28113%29.png)

![](../.gitbook/assets/image%20%28109%29.png)

### The Minimum!

So I'll go through the same process again and get down to what is actually needed...



### Common setup issues...

...or at least the mistakes that I made.

1\) Remember to add your user to the App in Azure AD first.

![](../.gitbook/assets/image%20%28112%29.png)

2\) Ensure that you use test it with an account that is synchronised from Active Directory. 

I used an Azure AD created identity and it didn't have an `on-premisessAMAccountName` that matches the username in Tableau. So it didn't pass the username attribute as required by the Tableau server identity store and claims.

![](../.gitbook/assets/image%20%28115%29.png)

