---
description: 1kg of Azure AD, 500g of SAML, and 100g of TOL and 100g of TS
---

# Recipe: Azure AD and Tableau

## Scope

Microsoft provides Azure AD apps that can be used to simplify the integration between Tableau Server and Tableau Online and Azure AD. The goal is to create a good onboarding and user experience to the Tableau services. It  explains the attributes required by the service to authenticate and plays around with the configuration to demonstrate how it works.&#x20;

Update - 22nd December 2021: Added in a Provisioning section for Tableau Online.

## Features

The two Azure AD apps have different feature sets. The Tableau Online application supports the following two features, whereas Tableau Server only supports SSO.

1. SP-initiated SSO
2. _REST API user provisioning_

Neither apps support [IdP-initiated sign-on](https://duo.com/blog/the-beer-drinkers-guide-to-saml). This means that if you publish the app in the Azure MyApps portal it will still do an SP-initiated Authentication request and therefore have the usual browser redirections for that flow. Also, SP-Initiated Single Logout (SLO) is not possible due to the use of HTTP-Post bindings.

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

## Tableau Online SAML

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

It is not possible to map directly to the UPN claim as that is one of the [restricted claim sets ](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-claims-mapping#claim-sets)in Azure AD. These _can't be modified using policy. The data source cannot be changed, and no transformation is applied when generating these claims_. So in this instance that is why the Name claim is used instead.

> UPDATE: There is a lot of new functionality in preview for Azure AD to allow you to create claim types that was previously restricted. [How to: Customize claims emitted in tokens for a specific app in a tenant (Preview)](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-claims-mapping)

You can choose to map `mail` or`userPrincipalName` as the `Email` in TOL. However as there isn't a separate email address attribute in TOL whatever is defined as `Email` must be a working email address as that value is what will be used to send out the subscriptions you have setup to views or workbooks.

![Workbook subscriptions](<../.gitbook/assets/image (69).png>)

### **Display Name**

Enter an assertion name for either the first name and last name, or for the full name, depending on how the IdP stores this information. Tableau Online uses these attributes to set the display name.

* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
* `http://schemas.microsoft.com/identity/claims/displayname`

So you can select either `givenname` + `surname` OR `displayname`

### The Minimum!

This is just a bit of an exercise in playing around with the configuration. Go to the [recommended](recipe-azure-ad-saml-and-tableau-online.md#the-recommended) section below for the real steps to follow!

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

## Tableau Server

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

### Common setup issues...

...or at least the mistakes that I made.

1\) Remember to add your user to the App in Azure AD first.

![](<../.gitbook/assets/image (112).png>)

2\) Ensure that you use test it with an account that is synchronised from Active Directory.&#x20;

I used an Azure AD created identity and it didn't have an `on-premisessAMAccountName` that matches the username in Tableau. So it didn't pass the username attribute as required by the Tableau server identity store and claims.

![](<../.gitbook/assets/image (115).png>)

## Provisioning: Tableau Online

Azure AD is not currently a supported provisioning method to Tableau Online although it does work and is used by many customers. There is a good tutorial from Microsoft that describes how to configure Azure AD provisioning which works by integrating with our REST API. There are however a number of constraints and choices to make on how you provision users and groups.&#x20;

If you want to test out a simple scenario then follow these steps to get up and running:

{% embed url="https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableau-online-provisioning-tutorial" %}
Strong MSFT doc...
{% endembed %}

### Sync Account

The tutorial describes how you connect to your TOL site and what account to use. Currently the credentials required for the provisioning need to be a Site Administrator (Explorer or Publisher) in TOL. The Azure App does not support PATs.

### Scope

With Azure AD you have a choice to make whether the Enterprise Application requires users to be assigned to it first (an additional step), or they can be provisioned directly from the Azure AD directory. ('Sync only assigned users and groups' OR 'Sync all users and groups'). I will mainly focus on the option where you assign the groups as this is the simplest method.

![](<../.gitbook/assets/image (135) (1).png>)

### SiteRole

If you choose to assign groups to the Azure AD application before they can be provisioned then you have the option to define the SiteRole within Tableau. This is the default SiteRole for the group. This is the simplest method for assigning Site Roles to Tableau groups tested and known results.

&#x20;![](<../.gitbook/assets/image (131).png>)



### Nested Groups

When you attempt to assign the group you are prompted with the message that users within nested AAD groups will not be provisioned, but the actual group will sync.

![No nested groups... whaaaat?!](<../.gitbook/assets/image (121).png>)

### Scoping Filters

Azure AD provides scoping filters for both Users and Groups. Again they provide a handy Tutorial.

{% embed url="https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts" %}

This allows you to filter the Group objects that are provisioned/de-provisioned. Also, to filter the users based on attributes.

![](<../.gitbook/assets/image (140).png>)



It took me a bit of playing around with it to figure out the relationship between the two as the app refers to it as they way to "Define which users are in scope for provisioning" even on the scoping filter for groups! Basically they work **independently**. The group filter, filters the group objects in Tableau, and the user filter, filters the user objects. Obviously...

So for example. I defined  the scoping filters:

#### Users:

![](<../.gitbook/assets/image (134).png>)

#### Groups:

![](<../.gitbook/assets/image (138).png>)

#### **Result:**&#x20;

Both filters are applied independently so that...

All groups that matched 'AD' in their description are provisioned

All users that are in 'Sales' are provisioned.&#x20;

All users NOT in Sales are not provisioned even if they were in the scoped and provisioned Groups.

These filters are very important if you go on to use the following configuration :point\_down:

### Sync all users and groups

{% hint style="info" %}
This following method was pioneered by a talented colleague and it requires some skilled configuration. I haven't tested this thoroughly so can't confirm it works for all scenarios.&#x20;
{% endhint %}

You can investigate using the method to 'Sync all users and groups' directly from Azure AD. You have to configure the Enterprise App away from the defaults. You will need to change 'Assignment Required' to No which is simply done in the properties of the Enterprise App.

![](<../.gitbook/assets/image (141).png>)

As you are not assigning the groups, there is not an option for you to configure the SiteRole. My testing showed that users without a SiteRole configured failed to provision. This can be done with a rule but it only allows for a single Default Value to be set. In this instance it is chosen to be Unlicensed to take advantage of [Grant License on Sign In](https://help.tableau.com/current/online/en-us/grant\_role.htm) feature in Tableau.

You can modify the Default Value to assign the users as Unlicensed in the Attribute Mappings for the _AppRoleAssignmentsComplex(\[appRoleAssignments]) ._ &#x20;

```
{"id":"97f6d3e9-6e9f-415b-9578-f6aad1f95dae","displayName":"Unlicensed"}
```

__![](<../.gitbook/assets/image (135).png>)__

The 'id' is taken from the App Registration Manifest for the Tableau Online app in Azure AD. It refers to the 'id' of the Unlicensed user.

![](<../.gitbook/assets/image (137).png>)

Once the user is provisioned as Unlicensed you would need to configure the group for [Grant License on Sign In](recipe-azure-ad-saml-and-tableau-online.md#tableau-online-saml) and to specify the minimum Site Role for the group. &#x20;

_NB: Also the mapping can be configured to 'Only apply during object creation'_

{% hint style="info" %}
_Additionally scoping filters are important otherwise you will  provision your whole directory without them!_
{% endhint %}

