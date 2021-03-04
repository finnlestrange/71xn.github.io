---
layout: single
title:  "Commando VM - My Windows pen-testing Environment"
date:   2020-06-19 18:06:23 +0100
categories: windows
excerpt: An overview of Commando VM, how to install it and why I use it for almost all Windows CTF's
---
![Commando](/images/commando-vm/Commando.png)

## Introduction

Hello all, today I just wanted to give a simple overview of my Windows based Pen-Testing environment, Commando VM, made by the guys over at FireEye. It may seem strange but I use Windows for the majority of all my cybersec work, as almost all business-like engagements are running some sort of Windows, and if you run the same system you can often interect with their system far better. For example native support for mapping smb shares, connecting your machine to a domain and browsing AD with tools like ldp, AD Users and Computers and Powershell modules like DSInternals and PowerView.

So firstly, I am going to go over the install process, the pros and the cons of the all-in-one installer. Then I will show you the base system, and finally I will show you what I have customised to make it more useful.

## Install Process

Firstly, you will want to have a Windows vm configured with at least 4 Gb or Ram and a 60 Gb virtual HDD. You should also only install Commando VM in a virtual machine as you can create snapshots and if the installer fails it could break your windows install.

To start off the install process you need to make sure that you have no pending windows updates, which means you may have to update, reboot and then check again.

![updates](/images/commando-vm/2020-06-19_18-42.png)

Once you have all the updates downloaded you can head over to the [Commando VM GitHub Page](https://github.com/fireeye/commando-vm), download the zip and extract it to your downloads folder.

![github](/images/commando-vm/2020-06-19_18-45.png)

![unzipped](/images/commando-vm/2020-06-19_18-47.png)

Now open an admin powershell session in the same directory

![shell](/images/commando-vm/2020-06-19_18-49.png)

### Running the script

Now that the script is downloaded we need to set the Execution Policy to Bypass and we need to Unblock the file so the installer works correctly.
`Unblock-File .\install.ps1` and `Set-ExecutionPolicy Unrestricted -f`

![shell](/images/commando-vm/2020-06-19_18-52.png)

Now we can run the script with `.\install.ps1`

![install-ps1](/images/commando-vm/2020-06-19_18-53.png)

Let the script run and make sure you take a snapshot of your vanilla install. Once it is done you should be booted into a desktop with the wallpaper set and the README on the desktop.

![wallpaper](/images/commando-vm/desktop.png)

## Pros and Cons 

Pros:
1. The all-in-one installer is quite good as it is completly unattended so you can just leave it to run
2. The installer makes sure your Windows environment is set-up in the right way before it proceeds.
3. You can customize what package are installed by editing the `profile.json` file.

Cons:
1. The installer take so damn long, 7 hours with 60+ Mbps internet, this is mostly caused by slow mirrors of packages you dont necessarily need like GIMP and Adobe PDF Reader.


## What I have changed since install

### Adding an anonymous SMB Share
The first thing I added was an anonymously accessable smb share to exfil files from windows machines, I have also used it for making copies of ntds.dit databases when I have compromised a backup operator, or admin account using `wpadmin.exe` 

Firstly you need to create a folder in the C drive called `shared`

![share](/images/commando-vm/2020-06-19_19-04.png)

Next we need to open the properties and select, share this folder

![prop](/images/commando-vm/2020-06-19_19-05.png)

Now head over to the security tab and add the guest and anonymous access as read/write

![anon](/images/commando-vm/2020-06-19_19-28.png)

Finally, open `local security policy` 
![lsp](/images/commando-vm/2020-06-19_19-37.png)

and drill down to, local policies, security options. Set these options and enable the guest account

![epic](/images/commando-vm/2020-06-19_19-39.png)

![epic](/images/commando-vm/2020-06-19_19-38.png)

Bingo! Now anyone can read and write to that share.

### Extra programs
As Commando VM uses the [Choco Package Manager](https://chocolatey.org/) we can seach the repo to find just about any program, a few I have installed are:

1. Cherrytree - for notes, `cinst cherrytree`

2. Ghidra  - for Reversing and Malware Analysis, `cinst ghidra`

3. Immunity Debugger - [Here](https://www.immunityinc.com/products/debugger/)

4. Softerra LDAP Browser - [Here](https://www.ldapadministrator.com/softerra-ldap-browser.htm)

5. NC for Windows - useful for reverse shells, exfil etc. [Here](https://eternallybored.org/misc/netcat/)

## Conclusions

I really like Commando VM and the relativly simple install process combined with package customisation makes this my hands down favorite Windows pen-test platform, (not that there are many) but I will continute to use it for Windows engagements. Also, as Commando VM comes with Kali WSL installed, all you kali tools, like ammass and sqlmap are right here. This is why it is my choice for nearly every windows machine and what you will see me using in future Windows CTF writeups.

Thank you for reading and I hope you found this useful in some way.

