---
layout: default
title : muzec - Penetration Testing With Nessus To Perform A Vulnerability Scan Against A Target 
---

### Description

In this lab you will have to use and configure Nessus in order to perform a vulnerability scan against the target machine. However you are not told where the target machine is in the network. You only know it is in the same lab network you are connected to.

### Goal

The goal of this lab is to learn how to properly configure Nessus depending on the services running on the target machine.

### Tools To Be Use

```
Fping
Nmap
Nessus
Metasploit
```

### Now The First Step Is To Find A Target In The Network

Since we do not have any information about our lab network and the hosts attached to it, the first step is to find our target!

![image](https://user-images.githubusercontent.com/69868171/139267986-beff2a09-69c3-425a-b029-fb2de58b1345.png)

We know our target is in the range of `192.168.99.0/24` now using cidr notation with fping to find alive host `fping -a -g 192.168.99.0/24 > scan.txt 2>/dev/null` .

![image](https://user-images.githubusercontent.com/69868171/139269027-c9ea7325-322b-4ff7-b34a-f6202c5c9a50.png)

We know `192.168.99.70` is our tap0 IP assign to us by the VPN connection so the target is `192.168.99.50` now using nmap to scan our target.

### Fingerprinting & Scanning

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/Penetration]
└─$ nmap -sC -sV -oA nmap 192.168.99.50 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-28 11:54 WAT
Nmap scan report for 192.168.99.50
Host is up (0.58s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 6h29m53s, deviation: 4h57m01s, median: 2h59m51s
|_nbstat: NetBIOS name: ELS-WINXP, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:a5:b3:99 (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: els-winxp
|   NetBIOS computer name: ELS-WINXP\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-10-28T06:56:09-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 178.49 second
```

Closly looking at our nmap result we can know we are dealing with a windows operating system.


### Installing Nessus

![image](https://user-images.githubusercontent.com/69868171/139275953-70f6fff3-078e-4028-b37b-c2b5298da4ac.png)

Downloaded but seems we need a activation.

![image](https://user-images.githubusercontent.com/69868171/139272971-53302f23-5e8e-4681-b1a7-95efbca08454.png)

![image](https://user-images.githubusercontent.com/69868171/139273819-7cf0f9c1-97f8-4954-bcf1-184f1abf3410.png)

Let Register for Nessus Essentials.

![image](https://user-images.githubusercontent.com/69868171/139274112-0e811f26-8891-4cd2-a561-a3c271b84642.png)

Done we can check our mail for the activation code but we are going to get to that part in a bit. now is time to install the nessus we downloaded.

```
dpkg -i Nessus-8.15.2-debian6_amd64.deb
```

![image](https://user-images.githubusercontent.com/69868171/139277283-3dbab624-d28e-49dd-aaff-36a71e06288a.png)

Now let start nessus scanner.

```
/bin/systemctl start nessusd.service
```

![image](https://user-images.githubusercontent.com/69868171/139278681-6648ff2d-9c2f-4816-9e3f-d4913166931a.png)


Checking our mail for the activation code to activate our nessus.

![image](https://user-images.githubusercontent.com/69868171/139279153-09410d5b-0278-43f7-a7fc-27a0c4694fb2.png)

User created now let wait.

![image](https://user-images.githubusercontent.com/69868171/139280859-10290f34-1ded-4c82-bbc6-c2d59a18ea5f.png)
