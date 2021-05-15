---
description: 'Yes, ThOse caPitals are coRRect'
---

# sAMAccountName & userPrincipalName

### In the beginning was...

`sAMAccountName` which was an attribute used as the logon name for earlier versions of Windows; NT4 and all that. When Microsoft realized the internet was a thing, they introduced a logon name that was [RFC 822](https://www.ietf.org/rfc/rfc0822.txt) compliant the `userPrincipalName` \(UPN\). It wasn't broadly adopted \(except for PKI\) until Office 365 required its use as the primary logon name for their cloud services. Since then the logon name has coalesced around UPN as the standard and the most important logon identity attribute in their directory.

{% embed url="https://docs.microsoft.com/en-us/windows/win32/ad/naming-properties" %}

A common deployment for Tableau Server integrates the server with Active Directory as the external identity store. As Tableau Server has been around for a while it still uses `sAMAccountName` when you sync users into the repository from Active Directory; and this then becomes your Tableau Server username.

![](.gitbook/assets/image%20%2881%29.png)

{% hint style="info" %}
**At present it is not possible to change this to UPN for Tableau Server using Active Directory as an External Identity Store.**
{% endhint %}

### Exceptions to sAMAccountName \(support issues\)

There are some exceptions to this AD user sync logic described in our [docs](https://help.tableau.com/current/server/en-us/users_manage_ad.htm) when the normal sync of`sAMAccountName` doesn't work. But this is **not** how to import UPN as a choice.

_Exception 1: If the UPN prefix of the user specified is greater than 20 characters, **and** the search string matches the full UPN, **and** is entered with the Windows login format \(domain\UPN\)._

![UPN prefix over 20 characters](.gitbook/assets/image%20%28101%29.png)

![Import like this](.gitbook/assets/image%20%28104%29.png)

This imported the user with the UPN prefix as the username and not the sAMAcountname:

![](.gitbook/assets/image%20%2897%29.png)

_Exception 2: If the user name you specify includes an `@`symbol in the UPN prefix **and** the search string you enter is either in the Windows domain login format \(`example.lan\jsmith@domain`\) or is the full UPN._

![Another @!](.gitbook/assets/image%20%2898%29.png)

![](.gitbook/assets/image%20%2896%29.png)

### So do these exceptions occur with Groups?

In short, no these exceptions do not occur if you sync an Active Directory user group. Tableau will sync all users from the group using the `sAMAccountName`. Importing with Groups has a number of advantages anyway with Grant Licence on Login for example.

### But I've already sync'd my users

> If user names were inadvertently imported using UPN names, you can delete the accounts in Tableau Server and then reimport those accounts using the `sAMAccountName` value for the user name

### What are the options if you must use/change to sync'ing UPN?

1\) Use Active Directory Lightweight Directory Services to present AD as an LDAP service to Tableau

2\) Local Identity Store \(requires provisioning of users at scale\)

3\) Use LDAP

Migration considerations at a 'very' high level:

* Map the user to the new username
* Migrate the content potentially using Site Export and Import

### How do I use sAMAccountName with Azure AD Authentication?

Conveniently, if you already synchronize your directory to Azure AD then there is the `onpremisessAMAccountName` attribute. This is available to be passed as a claim in your SAML authentication flow to match your on-premises Active Directory provisioned Tableau user. This is described in more detail [here](https://johnthompson365.gitbook.io/iamtableau/user-authentication/recipe-azure-ad-saml-and-tableau-online#tableau-server).

### What happens if you have multiple UPN suffixes?

#### What's a UPN suffix?

> Active Directory Domain Services \(AD DS\) domains have two types of names: Domain Name System \(DNS\) names and NetBIOS names. In general, both names are visible to end users. The DNS names of Active Directory domains include two parts, a prefix and a suffix. When creating domain names, first determine the DNS prefix. This is the first label in the DNS name of the domain. The suffix is determined when you select the name of the forest root domain.

{% embed url="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/assigning-domain-names" %}

You can create additional UPN suffixes, often to match the email addresses in use in your org so your users just have to remember 1 identity.

![More suffixes please](.gitbook/assets/image%20%28111%29.png)

UPN suffixes are then assigned to your users within your Active Directory forest. 

![A user with a different UPN suffix to the ](.gitbook/assets/image%20%28102%29.png)

As you can see below you can have a different UPN suffix but still be in the same domain.

![Repository](.gitbook/assets/image%20%28103%29.png)

This is also confirmed by running `tabcmd listdomains` where I still only have the THOMPSON365 domain visible to the server.

![tabcmd listdomains](.gitbook/assets/image%20%28118%29.png)

However, if you logon to Tableau with a different UPN suffix than your 'Domain Name' then you will receive this error.

![](.gitbook/assets/image%20%2882%29.png)

Checking out the vizportal logs:

![](.gitbook/assets/image%20%2880%29.png)

You need to ensure that all of your suffixes are added to the Tableau domain allow list similar to if you were working with a multi-domain deployment. 

{% embed url="https://help.tableau.com/current/server/en-us/cli\_configuration-set\_tsm.htm\#wgserver-domain-acceptlist" %}

![](.gitbook/assets/image%20%28105%29.png)

#### Sync/Logging strangeness

I can add users as part of a group with a different UPN suffix and do not receive the DEBUG event that it has recognised the user with a different UPN.

![](.gitbook/assets/image%20%28119%29.png)

