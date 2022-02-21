---
layout: default
title : muzec - Kevin - Proving Ground Practice 
---


### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v IP`

```
# Nmap 7.91 scan initiated Sun Nov 28 22:30:45 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 192.168.95.45
Increasing send delay for 192.168.95.45 from 0 to 5 due to 65 out of 216 dropped probes since last increase.
Warning: 192.168.95.45 giving up on port because retransmission cap hit (10).
Increasing send delay for 192.168.95.45 from 640 to 1000 due to 467 out of 1555 dropped probes since last increase.
Nmap scan report for 192.168.95.45
Host is up (0.23s latency).
Not shown: 41649 closed ports, 23876 filtered ports
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
3573/tcp  open  tag-ups-1
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sun Nov 28 22:32:12 2021 -- 1 IP address (1 host up) scanned in 87.23 seconds
```

Too much ports ahhh which is cool i guess now let run some default nmap scripts and service detection on the ports.

`nmap -sC -sV -oA nmap/normal -p 80,135,139,445,3389,3573,49152,49153,49154,49155 192.168.95.45`

```
# Nmap 7.91 scan initiated Sun Nov 28 22:33:31 2021 as: nmap -sC -sV -oA nmap/normal -p 80,135,139,445,3389,3573,49152,49153,49154,49155 192.168.95.45
Nmap scan report for 192.168.95.45
Host is up (0.41s latency).

PORT      STATE SERVICE            VERSION
80/tcp    open  http               GoAhead WebServer
| http-title: HP Power Manager
|_Requested resource was http://192.168.95.45/index.asp
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=kevin
| Not valid before: 2021-11-28T00:29:02
|_Not valid after:  2022-05-30T00:29:02
|_ssl-date: 2021-11-29T00:34:46+00:00; +2h59m55s from scanner time.
3573/tcp  open  tag-ups-1?
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 4h59m55s, deviation: 4h00m00s, median: 2h59m54s
|_nbstat: NetBIOS name: KEVIN, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:bf:8b:72 (VMware)
| smb-os-discovery: 
|   OS: Windows 7 Ultimate N 7600 (Windows 7 Ultimate N 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::-
|   Computer name: kevin
|   NetBIOS computer name: KEVIN\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-11-28T16:34:33-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-11-29T00:34:33
|_  start_date: 2021-11-29T00:29:57

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Nov 28 22:34:51 2021 -- 1 IP address (1 host up) scanned in 80.04 seconds
```

Some pretty cool services running let check what we have on SMB first.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/PG Practice/kevin]
└─$ smbclient -L //192.168.127.45/ -N                                                                                                                              1 ⨯
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
SMB1 disabled -- no workgroup available
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/PG Practice/kevin]
```

Hmmmm we have no share let hit port 80 now which is HTTP.

![image](https://user-images.githubusercontent.com/69868171/143794485-940e0cf6-2e95-47f8-ae63-d4e2e54c084d.png)

 HP Power Manager  with a login page let try some default credentials maybe we can get access.
 
 ![image](https://user-images.githubusercontent.com/69868171/143794542-9071b768-c9ae-4909-ba0b-43bbe935cff6.png)

Using admin/admin and we are in cool let check which version the HP Power Manager is running.

![image](https://user-images.githubusercontent.com/69868171/143794624-3fbda177-a584-4d17-b6a8-cb90092543c2.png)

Boom we have it let hit google to see if we have an exploit on it.

![image](https://user-images.githubusercontent.com/69868171/143794695-4d6d2f6a-29b8-4c42-a690-a344540221b2.png)

Boom exploit downloaded let run it and see.


![image](https://user-images.githubusercontent.com/69868171/143794852-61fb1627-2004-485a-911f-9d403272db6e.png)

we are system and done.


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
