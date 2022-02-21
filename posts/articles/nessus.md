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

User created now let wait the installing of the plugins and compiling take time.

![image](https://user-images.githubusercontent.com/69868171/139280859-10290f34-1ded-4c82-bbc6-c2d59a18ea5f.png)

In a bit.

![image](https://user-images.githubusercontent.com/69868171/139287442-a3a38481-f785-449f-a267-716a98570a36.png)

Done now let set up a new policies for windows.

![image](https://user-images.githubusercontent.com/69868171/139288259-b11be4a4-4bb7-4bf6-9fc2-a2545d6589c1.png)

Smooth seems we have some really cool Vulnerabilities scanners.

![image](https://user-images.githubusercontent.com/69868171/139288614-09211aa6-edba-40f8-9f39-8012e2c6e631.png)

But let click on advanced scan.

![image](https://user-images.githubusercontent.com/69868171/139289491-375541d1-3287-47f6-b29a-f7f0f772bf2e.png)

Now let click on `DISCOVERY` > `PORT SCANNING` .

![image](https://user-images.githubusercontent.com/69868171/139290060-23b5870b-088f-4147-896c-8b836d92569a.png)

Now let tick `TCP` .

![image](https://user-images.githubusercontent.com/69868171/139290153-bcb9ef9a-176a-46df-bdfe-19c2de704325.png)

Now let save it.

![image](https://user-images.githubusercontent.com/69868171/139290926-e30bdd19-e4df-4cf9-82c1-b1a62c02ff99.png)

Now back to My scan and click on New Scan.

![image](https://user-images.githubusercontent.com/69868171/139291920-2d192012-12f5-4e60-9549-165de80fc7f9.png)

Click on User Defined.

![image](https://user-images.githubusercontent.com/69868171/139292117-9e5473e8-54b1-4121-9177-70edf60b9e4a.png)

Now let click on our new policies we created.

![image](https://user-images.githubusercontent.com/69868171/139292683-bac53e68-4bda-4c6e-837c-b94f914827ab.png)

Add target and click on save.

![image](https://user-images.githubusercontent.com/69868171/139292901-e53d8ae5-cb1a-4643-b2d7-86e5b36ab250.png)

Now let click on the icon to launch.

![image](https://user-images.githubusercontent.com/69868171/139294074-b5302d4f-472f-4dab-9bec-12f7a332bd60.png)

Completed now let check the result.

![image](https://user-images.githubusercontent.com/69868171/139298008-53197ae0-c6ba-415b-9a48-45faa547ae31.png)

Now let export our report to see what we found export report in html.

![image](https://user-images.githubusercontent.com/69868171/139298350-c8c00ffa-f517-446f-b808-364c7de30194.png)

Some very interesting reports vulnerabilities to exploit.

![image](https://user-images.githubusercontent.com/69868171/139298765-16a0329d-1bab-4ba0-bfc8-83bfdc0b2a58.png)


### Vulnerabilities Exploitation With Metasploit

Let start with the first one:-  	MS08-067: Microsoft Windows Server Service Crafted RPC Request Handling Remote Code Execution (958644) (ECLIPSEDWING) (uncredentialed check)

Starting `Msfconsole` now let search the exploit.

![image](https://user-images.githubusercontent.com/69868171/139300997-c35e2175-20ff-4657-8df4-85c87af0e08f.png)

Now let use the exploit.

```
use exploit/windows/smb/ms08_067_netapi
```
Setting up `rhost and lhost` rhost which is the target IP and lhost our listen address.

![image](https://user-images.githubusercontent.com/69868171/139302096-1c76e377-2f66-4bb0-ad93-1b43bbc2c475.png)

Now let exploit.

![image](https://user-images.githubusercontent.com/69868171/139303497-ead4a087-bb7c-4ddc-b3e1-e0f87acd45c9.png)

Boom exploit completed.

Now for the second one:- MS09-001: Microsoft Windows SMB Vulnerabilities Remote Code Execution (958687) (uncredentialed check)

![image](https://user-images.githubusercontent.com/69868171/139304950-305c4a77-3e17-4dd4-a9fa-90ebc2201519.png)

Exploit that crashes the system.

![image](https://user-images.githubusercontent.com/69868171/139305343-c87133ec-3064-4d3b-a096-6327258765aa.png)

Boom overflowing till it crashes.

Now for the third one:- MS17-010: Security Update for Microsoft Windows SMB Server (4013389) (ETERNALBLUE) (ETERNALCHAMPION) (ETERNALROMANCE) (ETERNALSYNERGY) (WannaCry) (EternalRocks) (Petya) (uncredentialed check)

![image](https://user-images.githubusercontent.com/69868171/139305838-9a7d00f1-5e00-4c82-8a2e-389070a05c90.png)

Setting it up before exploiting.

![image](https://user-images.githubusercontent.com/69868171/139306138-0fe73015-db74-44a4-94bd-b3e17179b822.png)

Now let exploit it.

![image](https://user-images.githubusercontent.com/69868171/139542944-5a9b031b-e6aa-4310-a838-949c1350fad9.png)

Hmm interesting it vulnerable but the metasploit module is for x64 bit targets only you can use a quick python script to exploit it if you want to try it i think that from now see you next in the blackbox penetration testing series.

Hope you learn one or two from my article peace out guys.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
