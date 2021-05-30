---
layout: default
title : muzec - Gaara Writeup
---

Been having fun solving vulnerable box on vulnhub really so today i will be working on Gaara which can easily be download here [Download Gaara Of The Sand](https://www.vulnhub.com/entry/gaara-1,629/) it pretty easy i know am a fanboy of Naruto lol let hit it.

![image](https://user-images.githubusercontent.com/69868171/120100992-c9d95380-c111-11eb-8707-ab08032c60b8.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-30 04:31 EDT
Nmap scan report for 172.16.139.192
Host is up (0.00021s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:a3:6f:64:03:33:1e:76:f8:e4:98:fe:be:e9:8e:58 (RSA)
|   256 6c:0e:b5:00:e7:42:44:48:65:ef:fe:d7:7c:e6:64:d5 (ECDSA)
|_  256 b7:51:f2:f9:85:57:66:a8:65:54:2e:05:f9:40:d2:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Gaara
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.29 seconds
```

We have 2 open ports 22 and 80 we know we are going after the HTTP port first which is the 80.

![image](https://user-images.githubusercontent.com/69868171/120102546-87b41000-c119-11eb-8a8a-9764e3a7c3d7.png)

Checking the web page just a simple one really with the gaara image having the name `gaara` let try to brute force SSH with it maybe it the way in before burst for directory with gobuster.

![image](https://user-images.githubusercontent.com/69868171/120102741-7c151900-c11a-11eb-8b8b-8dae3594c049.png)

Boom it our way in let SSH now.

![image](https://user-images.githubusercontent.com/69868171/120102826-dd3cec80-c11a-11eb-8c27-ed693f9a70de.png)

We are i checking `sudo -l` but no luck let check what we have in the user folder and we have the first flag.

![image](https://user-images.githubusercontent.com/69868171/120102908-2f7e0d80-c11b-11eb-8a70-5f12b3e6c4c5.png)

Also a message for the kazekage encoded in base64 let decode it.

![image](https://user-images.githubusercontent.com/69868171/120102971-6eac5e80-c11b-11eb-80ba-86fa6ef5cfad.png)

Checking it we have a secret txt file which just end up to be a rabbit hole now let check for SUID.

![image](https://user-images.githubusercontent.com/69868171/120103127-1cb80880-c11c-11eb-8dfe-a4b74f45afe7.png)

Boom we have `/usr/bin/gdb` on SUID let hit Gtfobins.

![image](https://user-images.githubusercontent.com/69868171/120103173-5ab52c80-c11c-11eb-835f-2886a9959d09.png)

let run it and we should be root.

![image](https://user-images.githubusercontent.com/69868171/120103218-8fc17f00-c11c-11eb-8982-8c3c8a8ae7ba.png)

Boom we are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
