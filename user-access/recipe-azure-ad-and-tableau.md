---
description: 1kg of Azure AD, 500g of SAML, and 100g of TCL and 50g of TS
---

# Recipe: Azure AD and Tableau

## Scope

Microsoft provides Azure AD apps that can be used to simplify the integration between Tableau Online and Azure AD. The goal is to create a good onboarding and user experience to the Tableau services. It  explains the attributes required by the service to authenticate and plays around with the configuration to demonstrate how it works.&#x20;

#### Change Log

Update - 22nd December 2021: Added in a Provisioning section for Tableau Online.

Update - 5th August 2022: Updated Provisioning Section to include the latest [Tableau SCIM integration](https://help.tableau.com/current/online/en-us/scim\_config\_azure\_ad.htm)&#x20;

Update - 14th October 2022: Further SCIM updates

## Features

The two Azure AD apps have different feature sets. The Tableau Online application supports the following two features, whereas Tableau Server only supports SSO.

1. SP-initiated SSO
2. _REST API user provisioning_

Neither apps support [IdP-initiated sign-on](https://duo.com/blog/the-beer-drinkers-guide-to-saml). This means that if you publish the app in the Azure MyApps portal it will still do an SP-initiated Authentication request and therefore have the usual browser redirections for that flow. Also, SP-Initiated Single Logout (SLO) is not possible due to the use of HTTP-Post bindings.

## How to test it yourself

#### M365 Developer Program

{% embed url="https://developer.microsoft.com/en-us/microsoft-365/dev-program" %}

This gives you a free renewable 90 day M365 subscription, and M365 gives you Azure AD! Nice.

## Documentation

There are articles to configure the authentication steps. The Microsoft ones are little simpler to follow as they have screenshots.

**Microsoft Docs:**\
[Tutorial: Azure Active Directory single sign-on (SSO) integration with Tableau Online](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauonline-tutorial)\
[Tutorial: Azure Active Directory single sign-on (SSO) integration with Tableau Server](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauserver-tutorial)\
**Tableau Docs:**\
[Configure SAML with Azure Active Directory](https://help.tableau.com/current/online/en-us/saml\_config\_azure\_ad.htm) (Tableau Online)\
[Configure Server-Wide SAML](https://help.tableau.com/current/server/en-us/config\_saml.htm) (Tableau Server)

## Tableau Cloud: SAML

As part of the SAML authentication flow attributes are passed between the IdP (Azure) and the Service Provider (Tableau). Getting them right is key to a successful SSO.

### Configuration

There are two main properties that TOL is interested in, your **Email** and **Display Name**. The Email attribute is mapped to the username in Tableau Online and must match a licensed user stored in the Tableau Server Repository. The Display Name maps to the Full Name field in Tableau Online, it is populated with the assertions for First name and Last name or Full name. **** If they are not provided in the authN flow then the email address is used, so are actually optional.

![](<../.gitbook/assets/image (68).png>)

![TOL Attribute Configuration](<../.gitbook/assets/image (61).png>)

### Claims

Claims are widely referred to in Azure AD but not really in other major IdP's like Okta or OneLogin as they tend to refer to attributes. Claims are information about user and groups that are shared between the identity provider and the service provider in the SAML token. In the SAML token they are usually contained in the Attribute Statement, so you only really need to consider them as attributes. A _claim type_ provides context for the _claim value_. It is usually expressed as a Uniform Resource Identifier (URI). This URI is what is required to be put into the TOL configuration.

[SAML Token Claims Reference](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-saml-tokens)

| Name       | Description                                                                                                                                                                               | Type                                                                                                                                                                                                    |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name       | Provides a human readable value that identifies the subject of the token. This value is not guaranteed to be unique within a tenant and is designed to be used only for display purposes. | <p><code>&#x3C;Attribute Name="http://schemas.xmlsoap.org</code></p><p><code>/ws/2005/05/identity/claims/name"></code><br><code>&#x3C;AttributeValue>frankm@contoso.com&#x3C;AttributeValue></code></p> |
| First Name | Provides the first or "given" name of the user, as set on the Azure AD user object.                                                                                                       | <p><code>&#x3C;Attribute Name="http://schemas.</code></p><p><code>xmlsoap.org/ws/2005/05/identity/claims/givenname"></code><br><code>&#x3C;AttributeValue>Frank&#x3C;AttributeValue</code></p>          |
| Last Name  | Provides the last name, surname, or family name of the user as defined in the Azure AD user object.                                                                                       | <p><code>&#x3C;Attribute Name=" http://schemas.xmlsoap.</code></p><p><code>org/ws/2005/05/identity/claims/surname"></code><br><code>&#x3C;AttributeValue>Miller&#x3C;AttributeValue></code></p>         |

### Email

We ask for the Email attribute in the configuration which in turn is used as the TOL username and **cannot be changed**. The guidance we give on the Azure AD attributes to use has a couple of options:

* If all accounts you’re giving access to are sourced from **Microsoft accounts** (e.g. outlook.com/gmail.com, or any consumer email domain that has signed up for an account) this will be: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`
*   If all accounts are sourced from **Microsoft Azure Active Directory**, use the following value:

    `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

It is not possible to map directly to the UPN claim as that is one of the [restricted claim sets ](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-saml-tokens)in Azure AD. These _can't be modified using policy. The data source cannot be changed, and no transformation is applied when generating these claims_. So in this instance that is why the Name claim is used instead.

> UPDATE: There is a lot of new functionality in preview for Azure AD to allow you to create claim types that was previously restricted. [How to: Customize claims emitted in tokens for a specific app in a tenant (Preview)](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-claims-mapping)

You can choose to map `mail` or`userPrincipalName` as the `Email` in TCL. However as there isn't a separate email address attribute in TCL whatever is defined as `Email` must be a working email address as that value is what will be used to send out the subscriptions you have setup to views or workbooks.

![Workbook subscriptions](<../.gitbook/assets/image (69).png>)

### **Display Name**

Enter an assertion name for either the first name and last name, or for the full name, depending on how the IdP stores this information. Tableau Online uses these attributes to set the display name.

* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
* `http://schemas.microsoft.com/identity/claims/displayname`

So you can select either `givenname` + `surname` OR `displayname`

### The Minimum!

This is just a bit of an exercise in playing around with the configuration. Go to the [recommended](recipe-azure-ad-and-tableau.md#the-recommended) section below for the real steps to follow!

As it states in Azure, the only required claim is actually the Name ID. So I deleted all other for Name, Given Name etc.

![](<../.gitbook/assets/image (74).png>)

The identifier format is email address and in this instance I have mapped it to the Source Attribute of userPrincipalName in the directory (more information: [How to: customize claims issued in the SAML token for enterprise applications](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-claims-customization)).

![](<../.gitbook/assets/image (72).png>)

When logging on I received an error logging in:

_The site is configured to use the "_[_http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name_](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name)_" attribute as the username. Login failed because this is not included in the IdP Response. To fix this problem, the site admin must either choose the correct attribute or clear it to rely on the NameID._

By deleting the 'name' claim URI from the setting in Tableau Online (shown below) it forces SAML to rely on only the Name ID in the response.

![No claims defined for Email attribute](<../.gitbook/assets/image (75).png>)

You can then login successfully with only relying on the name identifier, but you won't get any useful information like `First Name` and `Surname`.

### The Recommended!

So first off you should always follow the Tableau guidance [here](https://help.tableau.com/current/online/en-us/saml\_config\_azure\_ad.htm#match-assertions) and match the assertions for Email, First name and Last name, or for Full name.&#x20;

If you are trying to decide whether to use mail or UPN then consider the following:

I am primarily interested in Enterprise organizations so consumer accounts are less of a consideration. In Azure AD and Microsoft 365 the `userPrincipalName` (UPN) is the attribute that is used to sign in to Office 365 services. Many organizations are likely to want to map that to the username in TOL as they know it is likely to change less than the email address. \
\
So the best guidance is to ask:

* Is the`UPN` the same value as the users working email account, to receive subscriptions?
* Which out of the `mail` or `UPN` is more likely to change?&#x20;
  * If it did change, in Azure AD this will not be reflected in Tableau Online.&#x20;
  * [Follow these steps](https://kb.tableau.com/articles/howto/changing-tableau-online-user-name) to provision a new account, migrate content and delete old account.

### User Experience

The Azure AD Tableau Online app **always** uses a SP-initiated flow. This means that the user experience involves a number of redirects. There are some configuration options that can smooth this experience.

#### Remember me

You should advise your users to check the **Remember me** option when signing in to TOL. Without this checked the SAML flow will take the user back to the initial TOL SSO page to input the username before redirecting to the IdP.

![](<../.gitbook/assets/image (9).png>)

**Keep me signed In (KMSI)**

As part of the Azure AD flow you are asked whether you want to stay signed in to the service. I would also ensure users to select this to prevent them having to input credentials again for an extended period of time.&#x20;

## Tableau Cloud: Provisioning

Azure AD is a fully supported provisioning method to Tableau Online and has a SCIM integration. Earlier versions of the Azure AD app relied on the Tableau REST API.&#x20;

The benefits of the SCIM integration over the REST API are:

* Industry standard SCIM integration
* A user can be a member of multiple groups in Azure, but they will only receive the most permissive site role in Tableau Online. For example, if a user is a member of two groups with site roles Viewer and Creator, Tableau will assign the Creator site role.
* Removes the need for a named user account to sync users/groups and supports a bearer token for authentication

There is a good tutorial from Microsoft that describes how to configure Azure AD provisioning which works by integrating with our REST API. There are however a number of constraints and choices to make on how you provision users and groups.&#x20;

If you want to test out a simple scenario then follow these steps to get up and running:

{% embed url="https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableau-online-provisioning-tutorial" %}
Strong MSFT doc...
{% endembed %}

The following points are my take on the things to look out for with the integration...

### Update your Tableau Cloud application to use the Tableau Cloud SCIM 2.0 endpoint <a href="#update-a-tableau-cloud-application-to-use-the-tableau-cloud-scim-20-endpoint" id="update-a-tableau-cloud-application-to-use-the-tableau-cloud-scim-20-endpoint"></a>

If you are already using the previous REST API app then [follow these steps](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableau-online-provisioning-tutorial#update-a-tableau-cloud-application-to-use-the-tableau-cloud-scim-20-endpoint) to migrate over and benefit from the SCIM features. Points to note:

The update process uses the Graph API to DELETE the current provisioning configuration of the sync that was using the REST API. You then POST the update in the next step that calls the SCIM endpoint. It requires broad permissions on the Graph API (**Directory.ReadWrite.All)** so I would  only be applying this using a specific admin account and revert the permissions after the change.



### Scope

There are really 2 modes of operation for provisioning with Azure AD, which is configured in the 'Settings' part of the provisioning configuration.&#x20;

1. <mark style="color:green;">**SUPPORTED:**</mark>** Sync only assigned users and groups:** You pre-assign users and groups to the Azure AD Enterprise application to be included in the scope of provisioning and those are sync'ed to Tableau
2. <mark style="color:red;">**NOT VIABLE:**</mark>** Sync all users and groups:** You allow all users and groups in Azure AD to be within scope of the Azure AD Enterprise Application provisioning and then proceed to use [Scoping Filters](recipe-azure-ad-and-tableau.md#scoping-filters) to only provision those required to Tableau.&#x20;

<mark style="color:red;">It is important to ensure you DO NOT use 'Sync all users and groups' this is not a viable method of provisioning for Tableau.</mark>

<mark style="color:red;"></mark>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### SiteRole

If you choose to **Sync only assigned users and groups** to the Azure AD application before they can be provisioned then you have the option to define the SiteRole for Tableau. This is the default SiteRole for the group and is the simplest method for assigning Site Roles to Tableau groups.

Take note of [our documentation ](https://help.tableau.com/current/online/en-us/scim\_config\_azure\_ad.htm#create-groups-for-site-roles)as despite 13 different roles presented in the Azure UI (below) only specific ones are workable and I have found this to depend upon the Azure tenant.&#x20;

[Microsoft advises ](https://learn.microsoft.com/en-us/azure/active-directory/saas-apps/tableau-online-provisioning-tutorial#valid-tableau-site-role-values)If you select a role that is not in the below list, such as a legacy (pre-v2018.1) role, you will experience an error. _Creator_, _SiteAdministratorCreator_, _Explorer_, _SiteAdministratorExplorer_, _ExplorerCanPublish_, _Viewer_, or _Unlicensed_. However I noticed an issue with assigning _Explorer (Can Publish)_ in my tenant. If you do then please contact Azure AD support.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Grant License on Sign In

This feature is not supported by any of the SCIM integrations with Tableau.&#x20;

### Nested Groups

When you attempt to assign the group you are prompted with the message that users within nested AAD groups will not be provisioned, but the actual group will sync.

![No nested groups... whaaaat?!](<../.gitbook/assets/image (121).png>)

### Scoping Filters

Azure AD provides scoping filters for both Users and Groups. Again they provide a handy Tutorial. You can use Scoping Filters even when you _'Sync assigned user and groups'._

{% embed url="https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts" %}

This allows you to filter the Group objects that are provisioned/de-provisioned. Also, to filter the users based on attributes.

It took me a bit of playing around with it to figure out the relationship between the two as the app refers to it as a way to "Define which users are in scope for provisioning". Basically they work **independently**. The group filter, filters the group objects in Tableau, and the user filter, filters the user objects. It's blooming obvious now...

So for example. I defined the scoping filters:

#### Users:

![](<../.gitbook/assets/image (134).png>)

#### Groups:

![](<../.gitbook/assets/image (138).png>)

#### **Result:**&#x20;

Both filters are applied independently so that...

* All groups that matched 'AD' in their description are provisioned
* All users that are in 'Sales' are provisioned.&#x20;
* All users NOT in Sales are not provisioned even if they were members of the scoped and provisioned Groups.

## Tableau Server: SAML

The Tableau Server Azure AD app supports SP-initiated sign on. You need to manage provisioning separately.

### Configuration

Downloading the metadata file from Tableau Server and then uploading to Azure AD simplifies the first steps in configuring the IdP. This gives you the correct URLs at least in my instance:

![Your friendly URLs](<../.gitbook/assets/image (108).png>)

### Do you have a claim?

The claims we define in our docs are:

![](<../.gitbook/assets/image (136).png>)

But if you go to the Server UI it asks you to match these assertions.

![The UI shows this...](<../.gitbook/assets/image (117).png>)

The default claims defined in the Azure AD app actually just work and shouldn't require changing but include unnecessary claims.

![The default Azure app gives you this...](<../.gitbook/assets/image (107).png>)

When you look closer at the claims it also shows the Name ID which actually contradicts the Tableau docs, but again works.

![](<../.gitbook/assets/image (116).png>)

These are the results from SAML trace for a successful login.

![](<../.gitbook/assets/image (113).png>)

![](<../.gitbook/assets/image (109).png>)

## Common setup issues...

...or at least the mistakes that I made.

1\) Remember to assign your user to the App in Azure AD first if you have "Sync only assigned users and groups" :smile:

![](<../.gitbook/assets/image (112).png>)

2\) Ensure that you use test it with an account that is synchronised from Active Directory.&#x20;

I used an Azure AD created identity and it didn't have an `on-premisessAMAccountName` that matches the username in Tableau. So it didn't pass the username attribute as required by the Tableau server identity store and claims.

![](<../.gitbook/assets/image (115).png>)

##
