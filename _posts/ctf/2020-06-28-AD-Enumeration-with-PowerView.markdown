---
layout: single
title:  "PowerView AD Enumeration"
date:   2020-06-28 15:22:23 +0100
categories: windows
excerpt: An overview of the useful commands and functions using PowerView.ps1 for Active Directory enumeration
---

## Intro

Hello all, today I will give a quick overview of PowerView.ps1 and its usefull AD enumeration commands. Firstly, I'll go over loading the PowerView Module into PowerShell and then I will show you some of the commands with a sample output, think of this like a cheat sheet for PowerView.

Download Link: [PowerSploit Repo](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)

## Loading PowerView.ps1
Firstly, make sure you are in the same directory as the PowerView.ps1 file. We also need to unblock the file with the `Unblock-File` cmdlet.

Downloading:
{% highlight powershell linenos %}
curl https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1 -O PowerView.ps1
{% endhighlight %}


Unblocking:
{% highlight powershell linenos %}
PS C:\Users\Administrator> Unblock-File .\PowerView.ps1
PS C:\Users\Administrator>
{% endhighlight %}

Loading:
{% highlight powershell linenos %}
PS C:\Users\Administrator> . .\PowerView.ps1
{% endhighlight %}


## Enumeration Commands

### Domain Information

{% highlight powershell linenos %}
Get-NetDomain
{% endhighlight %}
Shows information about the current domain, includign all domain controllers
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetDomain

Forest                  : lab.local
DomainControllers       : {LAB-DC-1.lab.local}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  :
PdcRoleOwner            : LAB-DC-1.lab.local
RidRoleOwner            : LAB-DC-1.lab.local
InfrastructureRoleOwner : LAB-DC-1.lab.local
Name                    : lab.local

PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell linenos %}
Get-NetDomainController
{% endhighlight %}
Shows us the ip and location of the DC
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetDomainController

Forest                     : lab.local
CurrentTime                : 28/06/2020 17:29:18
HighestCommittedUsn        : 16443
OSVersion                  : Windows Server 2019 Standard
Roles                      : {SchemaRole, NamingRole, PdcRole, RidRole...}
Domain                     : lab.local
IPAddress                  : fe80::10cd:dd9b:b02:d70%6
SiteName                   : Default-First-Site-Name
SyncFromAllServersCallback :
InboundConnections         : {}
OutboundConnections        : {}
Name                       : LAB-DC-1.lab.local
Partitions                 : {DC=lab,DC=local, CN=Configuration,DC=lab,DC=local, CN=Schema,CN=Configuration,DC=lab,DC=local,
                             DC=DomainDnsZones,DC=lab,DC=local...}

PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell linenos %}
Get-DomainPolicy
{% endhighlight %}
Shows us the Domain Policy Info
{% highlight powershell %}
PS C:\Users\Administrator> Get-DomainPolicy
Name                           Value
----                           -----
Kerberos Policy                {MaxTicketAge, MaxServiceAge, MaxClockSkew, MaxRenewAge...}
System Access                  {MinimumPasswordAge, MaximumPasswordAge, LockoutBadCount, PasswordComplexity...}
Version                        {Revision, signature}
Registry Values                {MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash}
Unicode                        {Unicode}

PS C:\Users\Administrator> (Get-DomainPolicy)."system access"
Name                           Value
----                           -----
MinimumPasswordAge             {1}
MaximumPasswordAge             {42}
LockoutBadCount                {0}
PasswordComplexity             {1}
RequireLogonToChangePassword   {0}
LSAAnonymousNameLookup         {0}
ForceLogoffWhenHourExpire      {0}
PasswordHistorySize            {24}
ClearTextPassword              {0}
MinimumPasswordLength          {7}

PS C:\Users\Administrator>
{% endhighlight %}


### Users, Groups and Computers


{% highlight powershell linenos %}
Get-NetUser
{% endhighlight %}
Gives us information on all the users in the domain, this can be made more concise with the select command
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetUser | select samaccountname

samaccountname
--------------
Administrator
Guest
krbtgt
svc-backup
svc-tgt


PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell linenos %}
Get-NetComputer
{% endhighlight %}
Lists all computers joined to the domain, you can get more info with -FullData flag
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetComputer -FullData

pwdlastset                    : 27/06/2020 19:27:27
logoncount                    : 9
msds-generationid             : {138, 162, 14, 106...}
serverreferencebl             : CN=LAB-DC-1,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=lab,DC=local
badpasswordtime               : 01/01/1601 00:00:00
distinguishedname             : CN=LAB-DC-1,OU=Domain Controllers,DC=lab,DC=local
objectclass                   : {top, person, organizationalPerson, user...}
lastlogontimestamp            : 27/06/2020 20:26:58
name                          : LAB-DC-1
objectsid                     : S-1-5-21-1086465519-1901182224-3361014284-1000
samaccountname                : LAB-DC-1$
localpolicyflags              : 0
codepage                      : 0
samaccounttype                : 805306369
whenchanged                   : 27/06/2020 19:36:56
accountexpires                : 9223372036854775807
countrycode                   : 0
adspath                       : LDAP://CN=LAB-DC-1,OU=Domain Controllers,DC=lab,DC=local
instancetype                  : 4
msdfsr-computerreferencebl    : CN=LAB-DC-1,CN=Topology,CN=Domain System
                                Volume,CN=DFSR-GlobalSettings,CN=System,DC=lab,DC=local
objectguid                    : 69302386-0a0e-4824-8f29-6517ba022186
operatingsystem               : Windows Server 2019 Standard
operatingsystemversion        : 10.0 (17763)
lastlogoff                    : 01/01/1601 00:00:00
objectcategory                : CN=Computer,CN=Schema,CN=Configuration,DC=lab,DC=local
dscorepropagationdata         : {27/06/2020 18:27:12, 01/01/1601 00:00:01}
serviceprincipalname          : {Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/LAB-DC-1.lab.local,
                                ldap/LAB-DC-1.lab.local/ForestDnsZones.lab.local,
                                ldap/LAB-DC-1.lab.local/DomainDnsZones.lab.local, TERMSRV/LAB-DC-1...}
usncreated                    : 12293
lastlogon                     : 28/06/2020 12:05:21
badpwdcount                   : 0
cn                            : LAB-DC-1
useraccountcontrol            : 532480
whencreated                   : 27/06/2020 18:27:08
primarygroupid                : 516
iscriticalsystemobject        : True
msds-supportedencryptiontypes : 28
usnchanged                    : 12775
ridsetreferences              : CN=RID Set,CN=LAB-DC-1,OU=Domain Controllers,DC=lab,DC=local
dnshostname                   : LAB-DC-1.lab.local

PS C:\Users\Administrator>
{% endhighlight %}


{% highlight powershell linenos %}
Get-NetGroup
{% endhighlight %}
Lists the groups in the domain and can be more specific with: Get-NetGroup -GroupName “Domain Admins”
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetGroup
Administrators
Users
Print Operators
Remote Desktop Users
Network Configuration Operators
RDS Remote Access Servers
RDS Endpoint Servers
RDS Management Servers
Hyper-V Administrators
Access Control Assistance Operators
Remote Management Users
Storage Replica Administrators
Domain Computers
Domain Controllers
Schema Admins
Enterprise Admins
Cert Publishers
Domain Admins
Domain Users
Domain Guests
Group Policy Creator Owners
RAS and IAS Servers
Server Operators
Account Operators
Key Admins
Enterprise Key Admins
DnsAdmins
DnsUpdateProxy
PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell linenos %}
Get-NetUser -SPN
{% endhighlight %}
Lists all users in the domain with a [Service Principal Name](https://adsecurity.org/?page_id=183) set.
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetUser -SPN

logoncount            : 0
badpasswordtime       : 01/01/1601 00:00:00
distinguishedname     : CN=svc-backup,CN=Users,DC=lab,DC=local
objectclass           : {top, person, organizationalPerson, user}
displayname           : svc-backup
userprincipalname     : svc-backup@lab.local
name                  : svc-backup
objectsid             : S-1-5-21-1086465519-1901182224-3361014284-1103
samaccountname        : svc-backup
admincount            : 1
codepage              : 0
samaccounttype        : 805306368
whenchanged           : 28/06/2020 17:17:26
accountexpires        : 9223372036854775807
countrycode           : 0
adspath               : LDAP://CN=svc-backup,CN=Users,DC=lab,DC=local
instancetype          : 4
usncreated            : 12780
objectguid            : ad3fcb7a-397d-48be-9c9c-3c85c1a0f056
lastlogoff            : 01/01/1601 00:00:00
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=lab,DC=local
dscorepropagationdata : {27/06/2020 19:41:28, 27/06/2020 19:39:59, 01/01/1601 00:00:00}
serviceprincipalname  : LAB/Backup
givenname             : svc-backup
memberof              : CN=Backup Operators,CN=Builtin,DC=lab,DC=local
lastlogon             : 01/01/1601 00:00:00
badpwdcount           : 0
cn                    : svc-backup
useraccountcontrol    : 66048
whencreated           : 27/06/2020 19:39:59
primarygroupid        : 513
pwdlastset            : 27/06/2020 20:39:59
usnchanged            : 16431

PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell linenos %}
Invoke-UserHunter -Unconstrained -ShowAll
{% endhighlight %}
Enumerates servers that allow unconstrained kerberos delegation and show all users logged in
{% highlight powershell %}
PS C:\Users\Administrator> Invoke-UserHunter -Unconstrained -ShowAll


UserDomain      : LAB
UserName        : Administrator
ComputerName    : LAB-DC-1.lab.local
IPAddress       : 192.168.0.49
SessionFrom     :
SessionFromName :
LocalAdmin      :

UserDomain      : LAB
UserName        : LAB-DC-1$
ComputerName    : LAB-DC-1.lab.local
IPAddress       : 192.168.0.49
SessionFrom     :
SessionFromName :
LocalAdmin      :

PS C:\Users\Administrator>
{% endhighlight %}



### Group Policy and Shares


{% highlight powershell linenos %}
Invoke-ShareFinder
{% endhighlight %}
Will list all of the shares on all computers in the domain
{% highlight powershell %}
PS C:\Users\Administrator> Invoke-ShareFinder
\\LAB-DC-1.lab.local\ADMIN$     - Remote Admin
\\LAB-DC-1.lab.local\C$         - Default share
\\LAB-DC-1.lab.local\IPC$       - Remote IPC
\\LAB-DC-1.lab.local\NETLOGON   - Logon server share
\\LAB-DC-1.lab.local\SYSVOL     - Logon server share
PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell linenos %}
Get-NetGPO
{% endhighlight %}
Shows us all of the GPO for the domain, can reveal interesting information such as a disabled antivirus
{% highlight powershell %}
PS C:\Users\Administrator> Get-NetGPO

usncreated              : 16448
displayname             : Disable Windows Defender
whenchanged             : 28/06/2020 18:34:44
objectclass             : {top, container, groupPolicyContainer}
gpcfunctionalityversion : 2
showinadvancedviewonly  : True
usnchanged              : 16453
dscorepropagationdata   : 01/01/1601 00:00:00
name                    : {5DDDD650-72B3-4F8A-94F6-74D5BDCBDE8D}
adspath                 : LDAP://CN={5DDDD650-72B3-4F8A-94F6-74D5BDCBDE8D},CN=Policies,CN=System,DC=lab,DC=local
flags                   : 0
cn                      : {5DDDD650-72B3-4F8A-94F6-74D5BDCBDE8D}
gpcfilesyspath          : \\lab.local\SysVol\lab.local\Policies\{5DDDD650-72B3-4F8A-94F6-74D5BDCBDE8D}
distinguishedname       : CN={5DDDD650-72B3-4F8A-94F6-74D5BDCBDE8D},CN=Policies,CN=System,DC=lab,DC=local
whencreated             : 28/06/2020 18:34:44
versionnumber           : 0
instancetype            : 4
objectguid              : cb0c35eb-c890-471c-98b1-1b99e64b67cc
objectcategory          : CN=Group-Policy-Container,CN=Schema,CN=Configuration,DC=lab,DC=local

PS C:\Users\Administrator>
{% endhighlight %}


## Conclusions 

Well, I hope you found this article somewhat interesting or at the very least, informative and if you did consider dropping me a +rep on HTB or a follow on twitter, if you want to read up more about these commands then take a look here, [GitHub Gist](https://gist.github.com/HarmJ0y/3328d954607d71362e3c).