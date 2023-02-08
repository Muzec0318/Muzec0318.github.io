---
layout: default
title : muzec - Toppo Writeup
---

![Image](https://imgur.com/2ACQ4OB.png)

We always start with an nmap scan.....

```Nmap -sC -p- -sV -oA nmap <Target-IP>```

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-05 11:56 EDT
Nmap scan report for 172.16.139.135
Host is up (0.010s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 ec:61:97:9f:4d:cb:75:99:59:d4:c1:c4:d4:3e:d9:dc (DSA)
|   2048 89:99:c4:54:9a:18:66:f7:cd:8e:ab:b6:aa:31:2e:c6 (RSA)
|   256 60:be:dd:8f:1a:d7:a3:f3:fe:21:cc:2f:11:30:7b:0d (ECDSA)
|_  256 39:d9:79:26:60:3d:6c:a2:1e:8b:19:71:c0:e2:5e:5f (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Clean Blog - Start Bootstrap Theme
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          43095/tcp   status
|   100024  1          43321/udp6  status
|   100024  1          49606/udp   status
|_  100024  1          53014/tcp6  status
43095/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.91 seconds
```

Cool let burst some dirs maybe we have any hidden dirs to be reveals.

![image](https://imgur.com/f2qexKZ.png)

Cool some nice dirs let check the one with admin.

![image](https://imgur.com/gdWzDBv.png)

Cool a note let check out.

![Image](https://imgur.com/wGcL0Pz.png)

Ok a password and observing the password i think i got the username already `ted` now let use it to log into ssh.

![image](https://imgur.com/0lPw5rk.png)

And boom we are in and i try checking sudo lol it not found ok let look around.

`find / -perm -u=s -type f 2>/dev/null`

![Image](https://imgur.com/2mx0e4z.png)

Cool Python on SUID let hit gtfobins to check it out.

![Image](https://imgur.com/487InlT.png)

And boom we are root.

![image](https://imgur.com/syCUMA3.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
