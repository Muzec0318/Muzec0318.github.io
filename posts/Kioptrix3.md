---
layout: default
title : muzec - Kioptrix 3 Writeup
---

![Image](https://imgur.com/Kct5rYN.png)

Let Get Started With an Nmap Scan...

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri Apr  2 16:27:25 2021 as: nmap -sC -sV -oA nmap 172.20.10.8
Nmap scan report for 172.20.10.8
Host is up (0.0018s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_  2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Ligoat Security - Got Goat? Security ...
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr  2 16:27:33 2021 -- 1 IP address (1 host up) scanned in 7.88 seconds
```

We got 2 open ports only let hit port 80 first.

![Image](https://imgur.com/PVcUjSk.png)

Wow LotusCMS let quickily hit searchsploit to check if it vulnerable, maybe we have any exploit on it.

![Image](https://imgur.com/GVoXLr3.png)

![Image](https://imgur.com/PJvY7mg.png)

Ahhh for metsaploit nah am really trying to avoid using that let do some research on it.

![image](https://imgur.com/vXAUAWi.png)

Some i found a script on it on github so i try going through it by downloading it on my machine.

![Image](https://imgur.com/0DMYsNF.png)

it was so easy to understand so i try url decoding on one of the payload i found on the script.

![Image](https://imgur.com/XPl4p2u.png)

So i try editing it to and url encode it back.

![image](https://imgur.com/S9ndEoB.png)

And boom we are able to execute command.

![image](https://imgur.com/fEzOSUY.png)

Now let inject it with some reverse shell payload let repeat the same process url encoding it.

![image](https://imgur.com/0kk41Xk.png)

![Image](https://imgur.com/WOUWXdb.png)

And boom we got shell.

![Image](https://imgur.com/DkwdRZg.png)

Wow it pretty old system.

![Image](https://imgur.com/G4Z9D7e.png)

![Image](https://imgur.com/0mE0Nyi.png)

![Image](https://imgur.com/k7HQYJ9.png)

So we download the exploit and get it on the attack machine now let complie and run it.

`gcc -pthread dirty.c -o dirty -lcrypt`

Boom now let run it after doing that let log in ssh with the new credentials we just got or we can easily just use the `su firefart` and the password we created with it

![image](https://imgur.com/Xd1HxYN.png)

![Image](https://imgur.com/gu2AJwn.png)

![Image](https://imgur.com/gAmjzwQ.png)

![Image](https://imgur.com/lnTl8Ge.png)

And boom we are root.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
