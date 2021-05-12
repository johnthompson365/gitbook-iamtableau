# Draft: sAMAccountName & userPrincipalName

The userPrincipalName is becoming the standard in many companies as **the** most important identity attribute in their directory.

{% embed url="https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats" %}

{% embed url="https://docs.microsoft.com/en-us/windows/win32/ad/naming-properties" %}

When importing users into Tableau Server this relies on the sAMAccountName. More specifically it would import your user from Active Directory in the full `domain\sAMAccountName` format and this would end up as your Tableau username.

![](../.gitbook/assets/image%20%2881%29.png)

**At present it is not possible to change this to UPN for Tableau Server using Active Directory as an External Identity Store.**

### Exceptions

There are some exceptions described in our [docs](https://help.tableau.com/current/server/en-us/users_manage_ad.htm) when the normal sync logic of importing sAMAccountName doesn't work. But this is not how to import UPN as a choice.

_Exception 1: If the UPN prefix of the user specified is greater than 20 characters, and the search string matches the full UPN, and is entered with the Windows login format \(domain\UPN\)._

![UPN prefix over 20 characters](../.gitbook/assets/image%20%28101%29.png)

![Import like this](../.gitbook/assets/image%20%28104%29.png)

This imported the user with the UPN prefix as the username and not the sAMAcountname:

![](../.gitbook/assets/image%20%2897%29.png)

_Exception 2: If the user name you specify includes an `@`symbol in the UPN prefix and the search string you enter is either in the Windows domain login format \(`example.lan\jsmith@domain`\) or is the full UPN._

![Another @!](../.gitbook/assets/image%20%2898%29.png)

![](../.gitbook/assets/image%20%2896%29.png)

### So can I import Groups this way?

In short, no. If you import an Active Directory user group, Tableau will import all users from the group using the `sAMAccountName`.  


### But I've already imported my users

> If user names were inadvertently imported using UPN names, you can delete the accounts in Tableau Server and then reimport those accounts using the `sAMAccountName` value for the user name

### What are the options if you must use/change to UPN?

With LDAP you can choose the exact attribute in their identity store as the username. If this is a current installation then it may need content migration because I don’t believe it can do an update of the attribute and have Tableau update the repository.

### How do I use sAMAccountName with Azure AD?

Conveniently, if you already synchronize your directory to Azure AD then there is the `onpremisessAMAccountName` attribute. This is available to be passed as a claim in your SAML authentication flow to match your on-premises Active Directory provisioned Tableau user. This is described in more detail [here](https://johnthompson365.gitbook.io/iamtableau/user-authentication/recipe-azure-ad-saml-and-tableau-online#tableau-server).

### What happens if you have multiple UPN suffixes?

What's a UPN suffix?

> Active Directory Domain Services \(AD DS\) domains have two types of names: Domain Name System \(DNS\) names and NetBIOS names. In general, both names are visible to end users. The DNS names of Active Directory domains include two parts, a prefix and a suffix. When creating domain names, first determine the DNS prefix. This is the first label in the DNS name of the domain. The suffix is determined when you select the name of the forest root domain.

{% embed url="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/assigning-domain-names" %}

You can create additional UPN suffixes, often to match the email addresses in use in your org so your users just have to remember 1 identity.

![More suffixes please](../.gitbook/assets/image%20%28111%29.png)

UPN suffixes are then assigned to your users within your Active Directory forest. 

![A user with a different UPN suffix to the ](../.gitbook/assets/image%20%28102%29.png)

![Repository](../.gitbook/assets/image%20%28103%29.png)

If you logon to Tableau with a different UPN suffix than your 'Domain Name' then you will receive this error.

![](../.gitbook/assets/image%20%2882%29.png)

Checking out the vizportal logs:

![](../.gitbook/assets/image%20%2880%29.png)

You need to ensure that all of your suffixes are added to the Tableau domain allow list.

[https://help.tableau.com/current/server/en-us/users\_manage\_ad.htm\#support-for-multiple-domains](https://help.tableau.com/current/server/en-us/users_manage_ad.htm#support-for-multiple-domains)

If Tableau Server connects to multiple domains, you must also specify the other domains that Tableau Server connects to, by setting the `wgserver.domain.acceptlist` option with TSM. 

[https://help.tableau.com/current/server/en-us/cli\_configuration-set\_tsm.htm\#wgserver-domain-acceptlist](https://help.tableau.com/current/server/en-us/cli_configuration-set_tsm.htm#wgserver-domain-acceptlist)

![](../.gitbook/assets/image%20%28105%29.png)

