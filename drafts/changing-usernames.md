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

![UPN prefix over 20 characters](../.gitbook/assets/image%20%2898%29.png)

![Import like this](../.gitbook/assets/image%20%28101%29.png)

This imported the user with the UPN prefix as the username and not the sAMAcountname:

![](../.gitbook/assets/image%20%2897%29.png)

_Exception 2: If the user name you specify includes an `@`symbol in the UPN prefix and the search string you enter is either in the Windows domain login format \(`example.lan\jsmith@domain`\) or is the full UPN._

![Another @!](../.gitbook/assets/image%20%2895%29.png)

![](../.gitbook/assets/image%20%2896%29.png)

### So can I import Groups this way?

In short, no. If you import an Active Directory user group, Tableau will import all users from the group using the `sAMAccountName`.  


### But I've already imported my users

> If user names were inadvertently imported using UPN names, you can delete the accounts in Tableau Server and then reimport those accounts using the `sAMAccountName` value for the user name

### What are the options if you must use/change to UPN?

With LDAP you can choose the exact attribute in their identity store as the username. If this is a current installation then it may the need to migrate the content because I don’t believe we can do an update of the attribute and have tableau update the repository.

### 

### What happens if you have multiple UPN suffixes?

UPN suffixes are a maintained list of supported suffixes that can be used for your users within your Active Directory forest.

![A user with a different UPN suffix to the ](../.gitbook/assets/image%20%2899%29.png)

This:

![](../.gitbook/assets/image%20%2882%29.png)

Checking out the vizportal logs:

![](../.gitbook/assets/image%20%2880%29.png)

You need to ensure that all of your suffixes are added to the Tableau domain allow list.

[https://help.tableau.com/current/server/en-us/users\_manage\_ad.htm\#support-for-multiple-domains](https://help.tableau.com/current/server/en-us/users_manage_ad.htm#support-for-multiple-domains)

If Tableau Server connects to multiple domains, you must also specify the other domains that Tableau Server connects to, by setting the `wgserver.domain.acceptlist` option with TSM. 

[https://help.tableau.com/current/server/en-us/cli\_configuration-set\_tsm.htm\#wgserver-domain-acceptlist](https://help.tableau.com/current/server/en-us/cli_configuration-set_tsm.htm#wgserver-domain-acceptlist)

![](../.gitbook/assets/image%20%28102%29.png)

![Repository](../.gitbook/assets/image%20%28100%29.png)

{% embed url="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/assigning-domain-names" %}



### 

### SAML

What does this mean for SAML?

SAML Authentication Fails in Multiple Domain Environment??





