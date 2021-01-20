# Draft Recipe: SAML with Onelogin

My goal is to setup OneLogin as the Identity Provider for Tableau Server. I want to synchronise users from Active Directory to OneLogin and use those accounts to sign in via SAML to Tableau.

### Tableau Server Identity Store:

During Tableau setup I have configured Active Directory as the External Identity store. So before I can authenticate with a user, I need to ensure there is actually a user account created within Tableau. I used the Tableau AD Users and Group Sync to import Adam.Wally from Active Directory. The Tableau username is based on the `sAMAccountName` attribute in AD:

![](../.gitbook/assets/image%20%2824%29.png)

So when imported:

![](../.gitbook/assets/image%20%2819%29.png)

As described [here](https://help.tableau.com/v2020.4/server/en-us/security_auth.htm)...If you configure Tableau Server to use Active Directory during installation, then NTLM will be the default user authentication method. So my test with `Adam.Wally` proved I could authenticate with my AD username and password.

### OneLogin Developer Tenant

First off, sign up for a free OneLogin developer account. This gives me the ability to test out all the features I need.

{% embed url="https://www.onelogin.com/developer-signup" %}

### Active Directory and OneLogin Directories

The Active Directory Connector was straightforward to download and setup. The connections were all outbound so in my lab it just worked straight away, you know how labs do. However, in an Enterprise you will need to configure the network security for your proxy or egress. So for any OneLogin agent, domains, ip addresses and ports are listed [here](https://onelogin.service-now.com/support?id=kb_article&sys_id=e1899821db8c60103de43e0439961952&kb_category=a0b5e130db185340d5505eea4b961957)... _Use **IP whitelisting** for on-premises agents, like Active Directory Connector_

![](../.gitbook/assets/image%20%2811%29.png)

Once downloaded the setup is simple. I chose the Create OneLogin Service Account option by inputting a password to setup the account via the wizard.

![](../.gitbook/assets/image%20%288%29.png)

![](../.gitbook/assets/image%20%289%29.png)

### Question: has this any special permissions?

![](../.gitbook/assets/image%20%284%29.png)

### What is OneLogin Desktop SSO - [https://onelogin.service-now.com/kb\_view\_customer.do?sysparm\_article=KB0010313](https://onelogin.service-now.com/kb_view_customer.do?sysparm_article=KB0010313)

![](../.gitbook/assets/image%20%286%29.png)

Once the setup is complete you go back to the OneLogin portal and is gives you the option to select the OU's/Containers that you would like to synchronise. Always, when I am testing a directory sync I start small. I know I only have 100 test users in my OU so know this should be a pretty quick exercise. I have seen many customers try to sync their entire domain and get into issues with large directories. Start small, test early and then incrementally add if you have 1000s of objects.

![](../.gitbook/assets/image%20%2816%29.png)

![](../.gitbook/assets/image%20%2814%29.png)

Even though the ADC sync was communicating with the Onelogin service. I did not have any new users in the directory. When I check the logs in `C:\Program Data\OneLogin, Inc\logs` I was getting an error about missing attributes.

![](../.gitbook/assets/image%20%2813%29.png)

The attributes shown below are required and luckily I knew straight away that my lab users didn't have an email address. 

![](../.gitbook/assets/image%20%2815%29.png)

I added the email address for one of my AD users re-synced and away we went!

![](../.gitbook/assets/image%20%2817%29.png)

So the synchronisation is working, now I just need to setup SAML to test signing in with this user.

### SAML

The standard setup instructions for Tableau Server are below:

{% embed url="https://help.tableau.com/current/server/en-us/config\_saml.htm" %}

The obvious steps apply of defining your entity ID, uploading certificates and sharing of metadata between Tableau as the service provider and OneLogin as the Identity Provider.

![](../.gitbook/assets/image%20%287%29.png)

#### Certificates:

I chose to use the same certificates I used for the Server SSL.

In addition to [all the normal requirements](https://help.tableau.com/current/server/en-us/saml_requ.htm#Certific) for your SSL certificate you [also need to ensure that](https://help.tableau.com/current/server/en-us/saml_requ.htm#Cert_Name) your certificate for SAML only includes the certificate that applies to Tableau Server and not any other certificates or keys.

### Onelogin Apps and SAML

There are a lot of Tableau apps in the Onelogin directory. I selected the Tableau Server app.

![](../.gitbook/assets/image%20%2812%29.png)

Once installed I was looking for an option to upload metadata but there isn't \(similar to Okta\). So the only option I had was to define the Server Name and protocol.

![](../.gitbook/assets/image%20%2821%29.png)

The default settings for SAML 2.0, included a Standard Strength Certificate and SHA-1 signing algorithm. All of this information I was able to download in the metadata and then import into Tableau.

![](../.gitbook/assets/image%20%2818%29.png)

The user attribute mappings are pretty simple for Tableau.

![](../.gitbook/assets/image%20%2820%29.png)

After uploading the metadata I applied the pending changes in Tableau which required a service restart. I love 'em. I wanted to test a basic SAML configuration without adding in things like SLO or different client SAML options.

![My favourite progress bar.](../.gitbook/assets/image%20%2823%29.png)

Troubleshooting

![](../.gitbook/assets/image%20%2825%29.png)

![](../.gitbook/assets/image%20%2822%29.png)

{% embed url="https://help.tableau.com/current/server/en-us/saml\_trouble.htm" %}

Next Steps:

Import users from OneLogin into Tableau \(maybe turn off SAML to do that if no _cli_ way of doing it\)  
Test out attrbiutes in the viz portal log  
Check with email aliging with samaccountname  
Check with AuthN method as AD and ONelogin in OL  
Check how to connect Desktop to the Tableau users table






