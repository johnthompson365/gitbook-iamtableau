---
description: '1kg of Azure AD, 500g of SAML, and 100g of TOL'
---

# Recipe: Azure AD and Tableau

## Scope

Microsoft provides Azure AD apps that can be used to simplify the integration between Tableau Server and Tableau Online and Azure AD. The goal is to create a good onboarding and user experience to the Tableau services. The initial release of this article focuses on Tableau Online.

## Features

The two apps have a different feature sets. The Tableau Online application supports the following three features, whereas Tableau Server does not support user provisioning.

1. SP-initiated SSO
2. SP-Initiated Single Logout \(SLO\)
3. _REST API user provisioning_

Neither apps support [IdP-initiated sign-on](https://duo.com/blog/the-beer-drinkers-guide-to-saml). This means that if you publish the app in the Azure MyApps portal it will still do an SP-initiated Authentication request and therefore have the usual browser redirections for that flow.

#### Azure AD Premium Trial

{% embed url="https://azure.microsoft.com/en-us/trial/get-started-active-directory/" %}

For 30 day access to AD premium, which would include Enterprise Applications and SAML you can signup above! 

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

Claims are widely referred to in Azure AD but not really in other major IdP's like Okta or OneLogin as they tend to refer to attributes. Claims are information about user and groups that are shared between the identity provider and the service provider in the SAML token. In the SAML token they are usually contained in the Attribute Statement, so you only really need to consider them as attributes. A claim type provides context for the claim value. It is usually expressed as a Uniform Resource Identifier \(URI\). This URI is what is required to be put into the TOL configuration.

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

You can then login successfully by only relying on the name identifier, but you won't get any useful information like First Name and Surname.

### The Recommended!

I am primarily interested in Enterprise organizations so consumer accounts are less of a consideration. In Azure AD and Microsoft 365 the `userPrincipalName` \(UPN\) is the attribute that is used to sign in to Office 365 services. Many organizations are likely to want to map that to the username in TOL as they know it is likely to change less than the email address.   
  
So the best guidance is to ask:

* Is the UPN the same value as the users working email account, to receive subscriptions?
* Is it likely to change? and if so are you comfortable with signing in with an account that is different? 

### User Experience

The Azure AD Tableau Online app always uses a SP-initiated flow. This means that the user experience involves a number of redirects. There are some configuration options that can smooth this experience.

#### Remember me

You should advise your users to check the **Remember me** option when signing in to TOL. Without this checked the SAML flow will take the user back to the initial TOL SSO page to input the username before redirecting to the IdP.

![](../.gitbook/assets/image%20%289%29.png)

**Keep me signed In \(KMSI\)**

I would also ensure users choose to keep themselves signed in to Azure AD, if they have the option to. 

