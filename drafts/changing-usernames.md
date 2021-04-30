---
description: sAMAccountName v userPrincipalName (UPN)
---

# Draft: Changing usernames

The userPrincipalName is becoming the standard in many companies as **the** most important identity attribute in their directory.

{% embed url="https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats" %}

{% embed url="https://docs.microsoft.com/en-us/windows/win32/ad/naming-properties" %}

When importing users into Tableau this relied on the sAMAccountName. More specifically it would import your user from Active Directory in the full `domain\sAMAccounName` format and this would end up as your Tableau username.

Many organisations would like to use userPrincipalName instead. **This is how you do it.**



![](../.gitbook/assets/image%20%2881%29.png)

### How do you change this?

We have guidance for it our [docs](https://help.tableau.com/current/server/en-us/users_manage_ad.htm):

![](../.gitbook/assets/image%20%2883%29.png)

### So can I import Groups this way?

In short, no. 

If you import an Active Directory user group, Tableau will import all users from the group using the `sAMAccountName`.  


### But I've already imported my users

> If user names were inadvertently imported using UPN names, you can delete the accounts in Tableau Server and then reimport those accounts using the `sAMAccountName` value for the user name

What happens to my data?



### What happens if you have multiple UPN suffixes?

This:

![](../.gitbook/assets/image%20%2882%29.png)

Checking out the vizportal logs:

![](../.gitbook/assets/image%20%2880%29.png)

You need to ensure that all of your suffixes are added to the Tableau domain allow list.

[https://help.tableau.com/current/server/en-us/users\_manage\_ad.htm\#support-for-multiple-domains](https://help.tableau.com/current/server/en-us/users_manage_ad.htm#support-for-multiple-domains)

If Tableau Server connects to multiple domains, you must also specify the other domains that Tableau Server connects to, by setting the `wgserver.domain.acceptlist` option with TSM. 

[https://help.tableau.com/current/server/en-us/cli\_configuration-set\_tsm.htm\#wgserver-domain-acceptlist](https://help.tableau.com/current/server/en-us/cli_configuration-set_tsm.htm#wgserver-domain-acceptlist)

`tsm configuration set -k wgserver.domain.accept_list -v "example.org, domain.com"`

This requires a server restart!





### SAML

What does this mean for SAML?

SAML Authentication Fails in Multiple Domain Environment??





