---
layout: default
title : muzec - Gift Writeup
---

![image](https://user-images.githubusercontent.com/69868171/121042795-03e5cd80-c782-11eb-9f13-330a939eb490.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/gift]
└─$ nmap -sC -p- -sV -oA nmap 172.16.139.202   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-07 08:17 EDT
Nmap scan report for 172.16.139.202
Host is up (0.00020s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
| ssh-hostkey: 
|   3072 2c:1b:36:27:e5:4c:52:7b:3e:10:94:41:39:ef:b2:95 (RSA)
|   256 93:c1:1e:32:24:0e:34:d9:02:0e:ff:c3:9c:59:9b:dd (ECDSA)
|_  256 81:ab:36:ec:b1:2b:5c:d2:86:55:12:0c:51:00:27:d7 (ED25519)
80/tcp open  http    nginx
|_http-title: Site doesn't have a title (text/html).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.48 seconds
```

We have two open ports checking HTTP.

![image](https://user-images.githubusercontent.com/69868171/121042795-03e5cd80-c782-11eb-9f13-330a939eb490.png)

Let check the source code.

![image](https://user-images.githubusercontent.com/69868171/121043355-88d0e700-c782-11eb-8000-d6e930d159f0.png)

Hmmm let try SSH brute forcing with username `root` .

![image](https://user-images.githubusercontent.com/69868171/121044191-54115f80-c783-11eb-9639-81e1ab9ee8e4.png)

Boom we have the root credentials let log in SSH.

![image](https://user-images.githubusercontent.com/69868171/121044333-799e6900-c783-11eb-9d6e-20e1e08dbfca.png)

Root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
