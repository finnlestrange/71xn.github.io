---
layout: single
title:  "DSInternals Overview"
date:   2020-06-27 17:06:23 +0100
categories: windows
excerpt: This is an overview of how to install, configure and use dsinternals for managing or attacking an AD environment
---

![logo](/images\dsinternals\DSInternals.png)

## Introduction
Today I will go through the install process and use of the [Directory Services Internals PowerShell Module and Framework](https://github.com/MichaelGrafnetter/DSInternals). This powershell tool can be quite useful for extracting password hashes from offline ntds.dit databases, as a good allternative to mimikatz, and for dumping credentials on a live system using DRS. DSInternals can also be used for legitmate puerposes such as, Password Auditing and Domain Controller Recovery. Firstly, we will go over the install process and then I will show you some of the many uses this framework has.

## Installation
The framework and powershell module can either be installed using [PowerShell Gallery](https://www.powershellgallery.com) or from [Source](https://github.com/MichaelGrafnetter/DSInternals/releases)

### Powershell Gallery
To install using PowerShell Gallery simpley open a PowerShell Terminal as Admin and enter the following command

{% highlight powershell linenos %}
Install-Module DSInternals -Force
{% endhighlight %}

Output:
{% highlight powershell %}
PS C:\Users\Administrator> Install-Module DSInternals -Force

NuGet provider is required to continue
PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet
provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' Do you want PowerShellGet to install and import the NuGet provider now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y

PS C:\Users\Administrator>
{% endhighlight %}

### Manual Install
To Manually install, first downloaded the latest release from the [GitHub Page](https://github.com/MichaelGrafnetter/DSInternals), Then open an admin powershell session in the same directory. Next run `Unblock-File download.zip` to unblock the file and it's contents. Finally, extract the contects to either, `C:\Windows\system32\WindowsPowerShell\v1.0\Modules\DSInternals` or `C:\Users\username\Documents\WindowsPowerShell\Modules\DSInternals`.

## Useful Commands for Managing and Attacking an AD Environement
Now that we have installed DSInternals we can connect to an AD domain and start running commands. These commands can be run either in a remote PowerShell Session or from a computer enroled in the domain and, logged in with a user who has Domain Admin rights.

### Commands for Online AD Environments
These are commands that a sysadmin would most likely use as they require you to be logged in with an account that has admin privelleges. These commands are most useful  for auditing domain security, such as comparing password hashes, calculating Kerberos Keys and grabbing users from Azure AD. 

Below will be a list of useful commands with a sample output and a short description of what they do:

{% highlight powershell %}
Get-SamPasswordPolicy
{% endhighlight %}
Shows the domain's password policy
{% highlight powershell linenos%}
PS C:\Users\Administrator> Get-SamPasswordPolicy

Domain: lab.local

MinPasswordLength           : 7
ComplexityEnabled           : True
ReversibleEncryptionEnabled : False
MaxPasswordAge              : 42.00:00:00
MinPasswordAge              : 1.00:00:00
PasswordHistoryCount        : 24

PS C:\Users\Administrator>
{% endhighlight %}



{% highlight powershell %}
Get-LsaPolicyInformation
{% endhighlight %}
Retrieves AD-related information from the Local Security Authority Policy of the local computer
{% highlight powershell linenos%}
PS C:\Users\Administrator> Get-LsaPolicyInformation

Domain/Workgroup Name : LAB
Forest DNS Name       : lab.local
Domain DNS Name       : lab.local
Domain GUID           : db72abe4-9c89-47f3-8e0d-51366e5cbf72
Domain SID            : S-1-5-21-1086465519-1901182224-3361014284
Account Domain Name   : LAB
Account Domain SID    : S-1-5-21-1086465519-1901182224-3361014284
Local Domain Name     : LAB-DC-1
Local Domain SID      : S-1-5-21-2644844979-1164627081-3792583456
Machine Account SID   : S-1-5-21-1086465519-1901182224-3361014284-1000

PS C:\Users\Administrator>
{% endhighlight %}


### Commands for Offline AD Environments
These are commands a sysadmin or attacker could use on an offline domain controller (or on an offline copy of the ntds.dit and SYSTEM), as when a DC is online the dtds.dit database is offline. For most of these commands you will need a copy of the ntds.dit and SYSTEM files, [here](https://www.cyberis.co.uk/2014/02/obtaining-ntdsdit-using-in-built.html) is a good article explaining how to do that.

{% highlight powershell %}
Get-BootKey [-SystemHiveFilePath] <String> [<CommonParameters>]
{% endhighlight %}
Grabs the system boot key from a copy of the SYSTEM registry file
{% highlight powershell linenos%}
PS C:\pentest\registry> Get-BootKey -SystemHiveFilePath .\SYSTEM
fd52aec47e2443decd3b37caf6d822ff
PS C:\pentest\registry>
{% endhighlight %}



{% highlight powershell %}
Get-ADDBAccount [-All] [-BootKey <Byte[]>] -DatabasePath <String> 
{% endhighlight %}
Reads account information from an offline ntds database
{% highlight powershell linenos%}
PS C:\pentest\registry> Get-ADDBAccount -BootKey fd52aec47e2443decd3b37caf6d822ff -SamAccountName svc-backup -DatabasePath '..\Active Directory\ntds.dit'

DistinguishedName: CN=svc-backup,CN=Users,DC=lab,DC=local
Sid: S-1-5-21-1086465519-1901182224-3361014284-1103
Guid: ad3fcb7a-397d-48be-9c9c-3c85c1a0f056
SamAccountName: svc-backup
SamAccountType: User
UserPrincipalName: svc-backup@lab.local
PrimaryGroupId: 513
Enabled: True
UserAccountControl: NormalAccount, PasswordNeverExpires
AdminCount: True
Deleted: False
LastLogonDate:
DisplayName: svc-backup
GivenName: svc-backup
SecurityDescriptor: DiscretionaryAclPresent, SystemAclPresent, DiscretionaryAclAutoInherited, SystemAclAutoInherited,
DiscretionaryAclProtected, SelfRelative
Owner: S-1-5-21-1086465519-1901182224-3361014284-512
Secrets
  NTHash: 7facdc498ed1680c4fd1448319a8c04f
  LMHash:
  NTHashHistory:
    Hash 01: 7facdc498ed1680c4fd1448319a8c04f
  LMHashHistory:
    Hash 01: 8e42bf8e046b8516e5d9e966c63f829e
  SupplementalCredentials:
    ClearText:
    NTLMStrongHash: efe59e65131eb1ef7213b17b9350788e
    Kerberos:
      Credentials:
        DES_CBC_MD5
          Key: f268581fd61c2a98
      OldCredentials:
      Salt: LAB.LOCALsvc-backup
      Flags: 0
    KerberosNew:
      Credentials:
        AES256_CTS_HMAC_SHA1_96
          Key: bc548d1a02a3179c542483a7b6343a9e3916330a98983000f09c8d40e4b3fe79
          Iterations: 4096
        AES128_CTS_HMAC_SHA1_96
          Key: 7291b6d73bc6f0f7d0e90565acd47381
          Iterations: 4096
        DES_CBC_MD5
          Key: f268581fd61c2a98
          Iterations: 4096
      Salt: LAB.LOCALsvc-backup
      DefaultIterationCount: 4096
      Flags: 0
    WDigest:
      Hash 01: 80095ef69cabb3ef8be2b4b06341feb5
      Hash 02: 3d8b8417a6c1ee5156021db75d016603
      Hash 03: ae4aaef5406f2e7a3257c8b65bd00c09
      Hash 04: 80095ef69cabb3ef8be2b4b06341feb5
      Hash 05: 3d8b8417a6c1ee5156021db75d016603
      Hash 06: 0841120a5e8e52747bc5bc3272edfdc3
      Hash 07: 80095ef69cabb3ef8be2b4b06341feb5
      Hash 08: a4221a314685ffb3854da8705340a9f5
      Hash 09: a4221a314685ffb3854da8705340a9f5
      Hash 10: 381732502709fc4234e25ac1ffc51ca4
      Hash 11: 781d4ecc2faf2a4083d815d48c742532
      Hash 12: a4221a314685ffb3854da8705340a9f5
      Hash 13: a56787883595e73758f91f29f43d28df
      Hash 14: 781d4ecc2faf2a4083d815d48c742532
      Hash 15: 657afb076157ffc8e0127c3ddf55fe26
      Hash 16: 657afb076157ffc8e0127c3ddf55fe26
      Hash 17: 3709a19dea664e52e0eb505f41809795
      Hash 18: 273feeab771458f602d99d41ebe4ec20
      Hash 19: 7199f9962b26ba6a95fb1c3a337b8966
      Hash 20: 776d4668f3f33993ce2fb4d68b5a8645
      Hash 21: d175c3c4cee0081366528f84aa3ef8b4
      Hash 22: d175c3c4cee0081366528f84aa3ef8b4
      Hash 23: 3af98e76714f060f532ffab8ae14bcaa
      Hash 24: c87b75995569b28d816d5e92f5e345d1
      Hash 25: c87b75995569b28d816d5e92f5e345d1
      Hash 26: 8833426d8c3c9ce0b969d6b3a9536bc6
      Hash 27: 16346819f1ab7eb3598668db37455da2
      Hash 28: c20d361da1eab95cf8d5461b9131c3ed
      Hash 29: 001ebebc41bf16007505cac450f80551

PS C:\pentest\registry>
{% endhighlight %}



{% highlight powershell %}
Set-ADDBAccountPassword -NewPassword <SecureString> -BootKey <Byte[]> [-SkipMetaUpdate] [-SamAccountName] <String> -DatabasePath <String>
{% endhighlight %}
Sets the password for a user, computer, or service account stored in a ntds.dit file.
{% highlight powershell linenos%}
PS C:\pentest\registry> Set-ADDBAccountPassword -NewPassword Password123! -SamAccountName svc-backup -BootKey fd52aec47e2443decd3b37caf6d822ff -DatabasePath '..\Active Directory\ntds.dit'

PS C:\pentest\registry>
{% endhighlight %}



{% highlight powershell %}
Get-ADDBBackupKey -BootKey <Byte[]> -DatabasePath <String>
{% endhighlight %}
Reads the DPAPI backup keys from a ntds.dit file.
{% highlight powershell linenos%}

FilePath          : ntds_legacy_b116cbfa-b881-43e6-ba85-ef3efa64ba22.key
KiwiCommand       : 
Type              : LegacyKey
DistinguishedName : CN=BCKUPKEY_b116cbfa-b881-43e6-ba85-ef3efa64ba22 
                    Secret,CN=System,DC=contoso,DC=com
KeyId             : b116cbfa-b881-43e6-ba85-ef3efa64ba22
Data              : {1, 0, 0, 0...}

FilePath          : 
KiwiCommand       : 
Type              : PreferredLegacyKeyPointer
DistinguishedName : CN=BCKUPKEY_P Secret,CN=System,DC=contoso,DC=com
KeyId             : b116cbfa-b881-43e6-ba85-ef3efa64ba22
Data              : {250, 203, 22, 177...}

FilePath          : ntds_capi_290914ed-b1a8-482e-a89f-7caa217bf3c3.pvk
KiwiCommand       : REM Add this parameter to at least the first dpapi::masterkey 
                    command: /pvk:"ntds_capi_290914ed-b1a8-482e-a89f-7caa217bf3c3.pvk"
Type              : RSAKey
DistinguishedName : CN=BCKUPKEY_290914ed-b1a8-482e-a89f-7caa217bf3c3 
                    Secret,CN=System,DC=contoso,DC=com
KeyId             : 290914ed-b1a8-482e-a89f-7caa217bf3c3
Data              : {2, 0, 0, 0...}
{% endhighlight %}


## Conclusions

I know this article was not to indepth, but I just wanted to give a basic overview of what it can do for pentesters. I really like DSInternals, especially because it allows the  extraction of hashes from ad databases without having to use a tool like mimikatz, which usually triggers or is blocked by antivirus, I also like how the framework can be used in any .NET application. Though it is not as useful for AD enumeration as `PowerUp.ps1` or `PowerView.ps1` it is an absolute life saver when it comes to password and hash extraction. 

