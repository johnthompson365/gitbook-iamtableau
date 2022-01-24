# Draft: Kerberos Constrained Delegation

## Goal

My goal is to configure KCD to a SQL server data source, with the user authentication being [SAML with OneLogin](https://app.gitbook.com/@johnthompson365/s/tableau/\~/drafts/-MSUgSon0V7lwJisPCjI/authentication/draft-recipe-saml-with-onelogin). My servers and workstations are all joined to an Active Directory domain and I am using AD as the Tableau external Identity store.

## Requirements

The requirements for [Enabling Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos\_delegation.htm) authentication to a database for Tableau are:

* The Tableau Server information store must be configured to use LDAP - Active Directory.
* The computer where Tableau Server is installed must be joined to Active Directory domain.
* A domain account must be configured as the Run As service account on Tableau Server.
* Delegation configured. Grant delegation rights for the Run As service account to the target database Service Principal Names (SPNs).

_If I read this_ [_Enabling Kerberos Delegation for SQL Server_](https://community.tableau.com/s/question/0D54T00000CWcplSAD/enabling-kerberos-delegation-for-sql-server?\_fsi=JnpHaLWS&\_fsi=JnpHaLWS&\_ga=2.182895846.1348344596.1612210017-159812869.1601602564&\_fsi=JnpHaLWS) _the first step it tells me to configure Kerberos authentication on the server. However I assumed you wouldn't require this to be configured for KCD to a data source. If I then review the guidance on keytab requirements it refers to_ [_Data Source Delegation_](https://help.tableau.com/current/server/en-us/kerberos\_keytab.htm#datasource-delegation) _four key points_

1. _The computer account for Tableau Server (Windows or Linux) must be in Active Directory domain._
2. _The keytab file that you use for Kerberos delegation can be the same keytab that you use for Kerberos user authentication (SSO)._
3. _The keytab must be mapped to the service principal for Kerberos delegation in Active Directory._
4. _You may use the same keytab for multiple data sources._

We also have the article [Use SAML SSO with Kerberos Database Delegation](https://help.tableau.com/current/server/en-us/saml\_with\_kerberos.htm), which doesn't refer to enabling kerberos as a user authentication scheme. However it does refer to using _...the Tableau Server keytab_ to access the database.

Additionally, some guidance in the [Configuring Kerberos authentication on Tableau server running on Windows](https://medium.com/@tableauman/configuring-kerberos-authentication-on-tableau-server-1917d127b6e3) article from my colleague Andrija Marcic.

### Supported Scenarios:

* Active Directory Kerberos

### Unsupported Scenarios:

* MIT Kerberos

## SQL and SPN's

### [Register a Service Principal Name for Kerberos Connections](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections?view=sql-server-ver15)

> ...A Service Principal Name (SPN) must be registered with Active Directory, which assumes the role of the Key Distribution Center in a Windows domain. The SPN, after it's registered, maps to the Windows account that started the SQL Server instance service. If the SPN registration hasn't been performed or fails, the Windows security layer can't determine the account associated with the SPN, and Kerberos authentication isn't used...

I have SQL installed and joined to an AD domain, and currently set to the defaults. The accounts that are running the services are local [Virtual Accounts](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#New\_Accounts) which are auto-managed and can logon to the network and register SPN's.

![](<../.gitbook/assets/image (44).png>)

> **Security Note:** Always run SQL Server services by using the lowest possible user rights. Use a [MSA](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#MSA), [gMSA](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#GMSA) or [virtual account](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#VA\_Desc) when possible. When MSA, gMSA and virtual accounts are not possible, use a specific low-privilege user account or domain account instead of a shared account for SQL Server services. Use separate accounts for different SQL Server services. Do not grant additional permissions to the SQL Server service account or the service groups. Permissions will be granted through group membership or granted directly to a service SID, where a service SID is supported...

I wanted to check the registering of SPN's worked with virtual accounts, so I queried AD for the SPN's related to the SQL server (screenshot below), I then deleted those SPN's using:`setspn -D MSSQLSvc/sql-win2019.thompson365.com sql-win2019`and `setspn -D MSSQLSvc/sql-win2019.thompson365.com:1433 sql-win2019`and restarted the SQL Database Engine and found they were recreated.

We query for the SPN based on the computer name because "_...Services that run as virtual accounts access network resources by using the credentials of the computer account in the format \<domain\_name>**\\**\<computer\_name>**$**..."_

![Using SetSPN](<../.gitbook/assets/image (52).png>)

Tableau SPN config

```
setspn -Q */tableau-win2016

setspn -s HTTP/tableau-win2016.thompson365.com thompson365\tableausvc
setspn -s HTTP/tableau-win2016 thompson365\tableausvc

setspn -D HTTP/tableau-win2016.thompson365.com thompson365\tableausvc
setspn -D HTTP/tableau-win2016. thompson365\tableausvc
```

![More SetSPN results this time for Tableau](<../.gitbook/assets/image (48).png>)

\*\*\*\*

**Put something about the security implications of this on the runas account:**

**Without the Tableau HTTP SPN configured you do not get the option to configure delegation.**

![](<../.gitbook/assets/image (54).png>)

![](<../.gitbook/assets/image (50).png>)

## Testing (reverse engineering)

This great article provides a useful SQL script to confirm your client is using kerberos : [https://www.red-gate.com/simple-talk/sql/database-administration/questions-about-kerberos-and-sql-server-that-you-were-too-shy-to-ask/](https://www.red-gate.com/simple-talk/sql/database-administration/questions-about-kerberos-and-sql-server-that-you-were-too-shy-to-ask/)

| Configuration                                                                 | Test                                                                                                                       | Result                                                                                                                                                              |
| ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Delete keytab                                                                 | Add SQL as a datasource in Desktop and check klist/sql script                                                              | No issues. Keytab not relevant.                                                                                                                                     |
| Delete Tableau SPN                                                            | <p>Delete SPN whilst SQL service is still running.</p><p>Add SQL as a datasource in Desktop and check klist/sql script</p> | This had no impact on desktop as it is making a SQL connection not an HTTP connection.                                                                              |
| Remove MSSQL SPN from Runas account                                           | Go users and computers and deselect MSSQLSvc from the delegation tab                                                       | <p>With the Tableau HTTP SPN deleted, there is not the delegation option available to delete anything.</p><p>(However, even without that you can still sign in)</p> |
| Delete SQL SPN (`setspn -D MSSQLSvc/sql-win2019.thompson365.com sql-win2019)` | Add SQL as a datasource in Desktop and check klist/sql script                                                              | I am able to sign in via Desktop. It doesn't pick up a kerberos ticket and it connects via NTLM                                                                     |

When I signed in using Tableau Desktop to the SQL server I received this cached ticket when running klist.

![ ](<../.gitbook/assets/image (57).png>)

Running the SQL Script shows that the connection from the tableau-desktop workstation is using kerberos.

![](<../.gitbook/assets/image (59).png>)

After I'd removed the Tableau SPN's (which removes the delegation config too) the user connection fell back to NTLM.

![](<../.gitbook/assets/image (60).png>)

After recreating the SPN's and delegation configuration ont he runas account, I needed to reboot the workstation to revert back to Kerberos from NTLM.

## Keytab in my lab

Useful notes in the [article](https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx) talk about how:

> Kerberos keytabs, also known as key table files, are only employed on non-Windows servers. In a homogenous Windows-only environment, keytabs will not ever be used, as the AD service account in conjunction with the Windows Registry and Windows security DLLs provide the Kerberos SSO foundation.



{% embed url="https://help.tableau.com/current/server/en-us/kerberos_keytab.htm#batch-file-set-spn-and-create-keytab-in-active-directory" %}

With Tableau it is important to understand the purpose and use case for the keytab file. We have 3 separate uses of the keytab file.

1. Operating system - which I would refer to as User Authentication&#x20;
2. Directory Service - read-access to your AD/LDAP
3. Datasource delegation - accessing data sources that have kerberos authentication enabled

We have a batch file that provides guidance on how to configure your keytab.

{% embed url="https://docs.microsoft.com/en-us/archive/blogs/pie/all-you-need-to-know-about-keytab-files?_fsi=JnpHaLWS" %}
Great article &#x20;
{% endembed %}

```
ktpass /princ http/tableau-win2016.thompson365.com@thompson365.com -SetUPN /mapuser thompson365\<tableau runas account> /pass <user password> /crypto AES256-SHA1 /ptype KRB5_NT_PRINCIPAL /out tableau.keytab
```

![](<../.gitbook/assets/image (55).png>)

{% embed url="https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx" %}
Useful write-up with good additional links
{% endembed %}

## Browser IWA

Logging in to browser on Tableau Server and the workstation. Likely IWA issue with browser below. _(Can’t connect to Microsoft SQL Server Detailed Error Message \[Microsoft]\[ODBC SQL Server Driver]\[SQL Server]Login failed for user 'THOMPSON365\tableau-runas'. Integrated authentication failed. 2021-02-02 **15:02:39.841,** (YBlpjyzPedSdTWKp5GyAsQAAAuo,1:0)_

![](<../.gitbook/assets/image (56).png>)
