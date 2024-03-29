---
description: >-
  1 cup of OneLogin, 1/2 cup of Tableau, Tablespoon of SAML and a pinch of
  Active Directory
---

# Recipe: OneLogin with Tableau

## Scope

My goal is to setup OneLogin as the Identity Provider for Tableau Server. I want to synchronise users from Active Directory to Tableau and OneLogin; then use those accounts to sign in with SAML to Tableau. Simple, right?

## OneLogin Developer Tenant

First off, sign up for a free OneLogin developer account. This gives me the ability to test out all the features I need.

{% embed url="https://www.onelogin.com/developer-signup" %}
Click the link to get your OneLogin developer account here!&#x20;
{% endembed %}

## Tableau Server Identity Store

During Tableau setup I have configured Active Directory as the External Identity store. So before I can authenticate with a user, I need to ensure there is actually a user account created within Tableau. I used the [Tableau AD Users and Group Sync ](https://help.tableau.com/current/server/en-us/users\_manage\_ad.htm)to import _Adam.Wally_ from Active Directory. The Tableau username is based on the `sAMAccountName` attribute in AD:

![AD user account properties](<../.gitbook/assets/image (24).png>)

So when imported:

![Tableau Users administration table](<../.gitbook/assets/image (19).png>)

As described [here](https://help.tableau.com/v2020.4/server/en-us/security\_auth.htm)...If you configure Tableau Server to use Active Directory during installation, then NTLM will be the default user authentication method. So my test with `Adam.Wally` proved I could authenticate with my AD username and password.

## Active Directory and OneLogin Directories

I did not want to manually populate OneLogin with user accounts, so I saw they had an [Active Directory Connector](https://onelogin.service-now.com/kb\_view\_customer.do?sysparm\_article=KB0010442) and decided to use that.

The Active Directory Connector was straightforward to download and setup. The connections were all outbound so in my lab it just worked straight away, you know how labs do. However, in an Enterprise you will need to configure the network security for your proxy or egress. So for any OneLogin agent, domains, ip addresses and ports are listed [here](https://onelogin.service-now.com/support?id=kb\_article\&sys\_id=e1899821db8c60103de43e0439961952\&kb\_category=a0b5e130db185340d5505eea4b961957)... _Use **IP whitelisting** for on-premises agents, like Active Directory Connector_

![ADC download option](<../.gitbook/assets/image (11).png>)

Once downloaded the setup is simple. I chose the Create OneLogin Service Account option by inputting a password to setup the account via the wizard.

![ADC setup service account option](<../.gitbook/assets/image (8).png>)

![Builtin\Administrators service account](<../.gitbook/assets/image (33).png>)

_...It creates a domain service account named OneLoginADC with Builtin\Administrators credentials in the local Domain..._ This was fine in my lab but I would follow the principle of least privilege in production and there is an article to guide you on how to [Create a Domain Service Account to run Active Directory Connector](https://onelogin.service-now.com/support?id=kb\_article\&sys\_id=e73c3a35dbf0441024c780c74b961909).

The ADC Setup also configures OneLogin Desktop SSO feature, now called [Windows Domain Authentication](https://onelogin.service-now.com/kb\_view\_customer.do?sysparm\_article=KB0010313). This allows for Integrated Windows Authentication if you require it for Windows and Mac desktops. Not a feature I required in my scenario but a default part of the ADC setup.

![Windows Domain Authentication port](<../.gitbook/assets/image (4).png>)

The setup is a little confusing here, as it refers to selecting the domains, but also includes the Users container. I just followed with the choice as is.

![Domain selection for sync](<../.gitbook/assets/image (6).png>)

Once the setup is complete you go back to the OneLogin portal and is gives you the option to select the OU's/Containers that you would like to synchronise. Always, when I am testing a directory sync I start small. I know I only have 100 test users in my OU so know this should be a pretty quick exercise. I have seen many customers try to sync their entire domain and get into issues with large directories. Start small, test early and then incrementally add if you have 1000s of objects.

![Portal user import option](<../.gitbook/assets/image (16).png>)

![ADC Connector status in portal](<../.gitbook/assets/image (14).png>)

Even though the ADC sync was communicating with the Onelogin service. I did not have any new users in the directory. When I check the logs in `C:\Program Data\OneLogin, Inc\logs` I was getting an error about missing attributes.

![Missing attributes in the logs](<../.gitbook/assets/image (13).png>)

The attributes shown below are required and luckily I knew straight away that my lab users didn't have an email address.

![Required sync attributes from AD to OneLogin](<../.gitbook/assets/image (15).png>)

I added the email address for one of my AD users re-synced and away we went!

![awally](<../.gitbook/assets/image (17).png>)

So the synchronisation is working, now I just need to setup SAML to test signing in with this user.

### Advanced Settings

After a time I found I was being continually prompted for password expiry. I figured this wasn't driven from AD as I was manually changing the password and still getting prompted. You can check advnaced settings on the directory sync and there are a whole host of configuration options so I advise reviewing before deploying. **** The offending one for me was this one which was enabled by default:

![](<../.gitbook/assets/image (51).png>)

## Tableau Server-Wide SAML setup

The standard setup instructions for Tableau Server are below:

{% embed url="https://help.tableau.com/current/server/en-us/config_saml.htm" %}

The obvious steps apply of defining your entity ID, uploading certificates and sharing of metadata between Tableau as the service provider and OneLogin as the Identity Provider.

![](<../.gitbook/assets/image (7).png>)

### Certificates

I chose to use the same certificates I used for the Server SSL. In addition to [all the normal requirements](https://help.tableau.com/current/server/en-us/saml\_requ.htm#Certific) for your SSL certificate you [also need to ensure that](https://help.tableau.com/current/server/en-us/saml\_requ.htm#Cert\_Name) your certificate for SAML only includes the certificate that applies to Tableau Server and not any other certificates or keys.

## Onelogin Apps and SAML

There are a lot of Tableau apps in the Onelogin directory. I selected the **Tableau Server SAML 2.0** app. _This was my first mistake ;)_

![](<../.gitbook/assets/image (12).png>)

Once installed I was looking for an option to upload metadata but there isn't (similar to Okta). So the only option I had was to define the Server Name and protocol, it seemed strange as normally I would have to input the Entity ID **and** ACS when configuring Tableau Online with an IdP, but I ploughed on regardless _(mistake #2)_.

![](<../.gitbook/assets/image (21).png>)

The default settings for SAML 2.0, included a Standard Strength Certificate and SHA-1 signing algorithm. All of this information I was able to download in the metadata and then import into Tableau.

![](<../.gitbook/assets/image (18).png>)

The user attribute mappings are pretty simple for Tableau.

![](<../.gitbook/assets/image (20).png>)

After uploading the metadata I applied the pending changes in Tableau which required a service restart. I love 'em. I wanted to test a basic SAML configuration without adding in things like SLO or different client SAML options.

![My favourite progress bar.](<../.gitbook/assets/image (23).png>)

The services restarted and I attempted my first sign in and... FAIL!\
The error I was getting was:

![](<../.gitbook/assets/image (26).png>)

I attempted both an IdP-initiated sign on and SP-initiated and got the same error.

## Troubleshooting

I reset the password in OneLogin and checked that I could sign in. I checked the account in OneLogin and I made sure that I was 'Authenticated by' OneLogin and **not** Active Directory. If it was set to Active Directory I could use my AD username and password to sign in.

![](<../.gitbook/assets/image (22).png>)

In Tableau I wanted to check the usernames in the Identity Store. Weirdly, I couldn't find a simple way to do this, as I had enabled Server-Wide SAML and was automatically being redirected to the OneLogin portal for all authentication requests. I looked at using **tabcmd** but couldn't see an obvious cmd that just listed users (maybe I missed something). Also TSM doesn't contain user information it is only the higher-level server configuration that is managed from the portal.

I found these [instructions](https://kb.tableau.com/articles/howto/exporting-user-list) to access the Server Repository which is a PostgreSQL database. This is a lot of work but I found it useful to look at the underlying database. A quicker method with no server restart would be to connect via the [REST API](https://help.tableau.com/current/api/rest\_api/en-us/REST/rest\_api\_get\_started\_tutorial\_intro.htm) and get the information.

I had to ensure I had network access on port 8060 and run a tsm command first as explained [here](https://help.tableau.com/current/server/en-us/perf\_collect\_server\_repo.htm).

![](<../.gitbook/assets/image (28).png>)

![Connection details](<../.gitbook/assets/image (34).png>)

Once I had connected I could then browse the \_users table:

![The \_users table in all its glory](<../.gitbook/assets/image (37).png>)

I checked I was passing the value in the Name column as the username attribute in SAML (_adam.wally_). This corresponds with the sAMAccountName shown at the top of the article in AD and mapped in OneLogin. I'm a big fan of [SAML-tracer for Firefox](https://addons.mozilla.org/en-US/firefox/addon/saml-tracer/) and Chrome. This showed I was passing the correct attribute:

![SAML-tracer SAML summary](<../.gitbook/assets/image (25).png>)

The next step was to look at the logs on the Tableau side to see if there was anything obvious reported.

{% embed url="https://help.tableau.com/current/server/en-us/saml_trouble.htm" %}

To log SAML-related events, `vizportal.log.level` must be set to `debug`. For more information, see [Change Logging Levels](https://help.tableau.com/current/server/en-us/logs\_debug\_level.htm).

```
tsm configuration set -k vizportal.log.level -v debug
tsm pending-changes apply

tsm configuration set -k vizportal.log.level -d
tsm pending-changes apply
```

Check for SAML errors in the following files in the unzipped log file snapshot:`\vizportal\vizportal-<n>.log`

![Ahh, no valid audiences...?](<../.gitbook/assets/image (36).png>)

I did some searching for the error message but I wasn't clear what Audiences was referring to. I then did what I probably should have done at the start and seen if someone else had already set this up. I found this [great article](https://medium.com/@kannanmadhav/configuring-saml-for-tableau-server-with-onelogin-3e9a58cb2931) by a colleague of mine Madhav Kannan that identified my issue. At the very start I had chosen the wrong OneLogin Tableau app. I need to have chosen **Tableau Server(Signed Response)**.

![](<../.gitbook/assets/image (35).png>)

This allows me to configure what OneLogin refers to as the SAML [Audience](https://support.okta.com/help/s/article/Common-SAML-Terms?language=en\_US), but we reference as the value **Entity ID**.

![Remember this from the Tableau SAML configuration?](<../.gitbook/assets/image (39).png>)

So this time the OneLogin app makes more sense.

![](<../.gitbook/assets/image (32).png>)

## Client Exceptions

I mentioned earlier that with Server-Wide SAML configured every user had to authenticate using SAML, there is no mixed authentication methods. However, there is an interesting feature within the SAML configuration that allows you to apply exceptions for the Desktop client.

![](<../.gitbook/assets/image (47).png>)

> There is updated guidance around Mobile client option shown above as it no longer works for newer clients: \
> Allow using SAML to sign in from older versions of Tableau Mobile app. Devices running Tableau Mobile app version 19.225.1731 and higher ignore this option. To disable devices running Tableau Mobile app version 19.225.1731 and higher, disable SAML as a client login option on Tableau Server.

When I choose to sign in to the Tableau Server using the desktop client I get challenged by the normal SAML flow (shown below).

![SAML Authentication via Desktop client](<../.gitbook/assets/image (45).png>)

By **deselecting** the SAML sign in from Tableau Desktop the flow reverts to the username/password or NTLM AuthN.

Unfortunately it requires a server restart so go get a cup of tea.

![Oh, ok.](<../.gitbook/assets/image (43).png>)

After the restart you will see that the login prompt has changed from being redirected to your SAML IdP, to the normal Tableau server prompt. You will then be able to sign in with your AD/LDAP credentials or Tableau Local username and password.

![Sign in with your AD credentials or basic username/password](<../.gitbook/assets/image (46).png>)

## Single Logout

After logging on to the server I noticed I did not have option to sign out of Tableau. Tableau Server and Tableau Online supports Single Logout (SLO) for server-wide SAML (only SP-initiated not IdP-initiated). If you have configured Site SAML in Tableau Server neither SP or IdP-initiated are supported.

This feature is enabled by importing the SAML SLO Endpoint as part of the OneLogin IdP metadata. However despite uploading the correct metadata it was not available:

![How can I sign out?](<../.gitbook/assets/image (41).png>)

Looking at the OneLogin metadata XML below you can see there are different HTTP methods defined for SLO and SSO, _redirect_ and _post_ respectively.

![HTTP Redirect](<../.gitbook/assets/image (40).png>)

As our [SAML Requirements](https://help.tableau.com/current/server/en-us/saml\_requ.htm#xml\_requirements) documentation states... **\*\*\_**HTTP POST\*\*: Tableau Server accepts only HTTP POST requests for SAML communications. HTTP Redirect is not supported.\_ Unfortunately this means SLO is not currently going to work with OneLogin.

### **How it should work:**

![My nice shiny Sign Out](../.gitbook/assets/image-3.webp)

## Conclusion

### OneLogin

Getting Active Directory sync'ed up to OneLogin was surprisingly easy in my little lab which is a good sign. I like the outbound ADC agent, (similar to the Azure connectors) in not requiring reverse proxies. Naturally it would take a lot more planning in an Enterprise but if you get stuck in a lab that would be more concerning. The number of Tableau apps in OneLogin is confusing so pay attention and pick the right one, ...do what I say not what I do!

### SAML

If you setup Server Wide SAML then know that it means **all accounts** and you can't login with any other AuthN method. So, ensure all necessary accounts are sync'ed into your IdP (such as admins) or be ready to disable SAML for a workaround.

### **Single Logout**

With the lack of Single Logout, I don't see this as an issue for many users where speed is important by just closing a browser; but with more advanced users who are used to signing out as good practice this may come as a surprise. However, this does highlight a risk around [session hijacking ](https://owasp.org/www-community/attacks/Session\_hijacking\_attack)particularly in shared computer or embedded scenarios. Standard mitigations for this would be configuring the browser or running a job to regularly delete persistent cookies and session cookies, and then use threat detection and security in depth with TLS and anti-malware.
