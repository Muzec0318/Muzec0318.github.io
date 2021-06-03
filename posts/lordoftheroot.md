---
layout: default
title : muzec - Lord Of The Root Writeup
---

Lord Of The Root a cool box on vulnhub which help others learn some basic CTF hacking strategies and some tools like Port Knocking,Sqlmap etc you can easily grab a copy here [Lord Of The Root Download Here](https://www.vulnhub.com/entry/lord-of-the-root-101,129/)

![image](https://user-images.githubusercontent.com/69868171/120629067-6dd53e80-c433-11eb-8bab-cf0138683786.png)


We always start with an nmap scan.....

```Nmap -sC -sV -Pn -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/lord]
└─$ nmap -sC -Pn -sV -oA nmap 172.16.139.200
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-03 02:59 EDT
Nmap scan report for 172.16.139.200
Host is up (0.00071s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3c:3d:e3:8e:35:f9:da:74:20:ef:aa:49:4a:1d:ed:dd (DSA)
|   2048 85:94:6c:87:c9:a8:35:0f:2c:db:bb:c1:3f:2a:50:c1 (RSA)
|   256 f3:cd:aa:1d:05:f2:1e:8c:61:87:25:b6:f4:34:45:37 (ECDSA)
|_  256 34:ec:16:dd:a7:cf:2a:86:45:ec:65:ea:05:43:89:21 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.21 seconds
```
We have only one port SSH that is strange let try checking the SSH for some clue.

![image](https://user-images.githubusercontent.com/69868171/120630276-ab869700-c434-11eb-8563-9493e40b3b4d.png)

Knock Friend To Enter the hint is clear seems we have to use port knocking on it `Easy as 1,2,3` yes the 1 2 3 that only what we have let try it.

Port knocking is a stealth method to externally open ports that, by default, the firewall keeps closed. It works by requiring connection attempts to a series of predefined closed ports. With a simple port knocking method, when the correct sequence of port "knocks" (connection attempts) is received, the firewall opens certain port(s) to allow a connection. 

### Port Knocking

Port Knocking command `knock 172.16.139.200 1 2 3` .

![image](https://user-images.githubusercontent.com/69868171/120631489-f228c100-c435-11eb-87e0-bfb5875570e0.png)

Now let scan it again with Nmap.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/lord]
└─$ nmap -sC -Pn -p- -sV -oA nmap 172.16.139.200
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-03 03:38 EDT
Nmap scan report for 172.16.139.200
Host is up (0.00060s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3c:3d:e3:8e:35:f9:da:74:20:ef:aa:49:4a:1d:ed:dd (DSA)
|   2048 85:94:6c:87:c9:a8:35:0f:2c:db:bb:c1:3f:2a:50:c1 (RSA)
|   256 f3:cd:aa:1d:05:f2:1e:8c:61:87:25:b6:f4:34:45:37 (ECDSA)
|_  256 34:ec:16:dd:a7:cf:2a:86:45:ec:65:ea:05:43:89:21 (ED25519)
1337/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 116.26 seconds
```
Boom we have a new port open `1337` HTTP cool the power of knocking is cool now time to hit the port on HTTP.

![image](https://user-images.githubusercontent.com/69868171/120632575-1b961c80-c437-11eb-8f5d-5f2aa80899ee.png)

A simple page checking source code nothing i try bursting directory nothing also strange right.

![image](https://user-images.githubusercontent.com/69868171/120632780-5435f600-c437-11eb-8bd6-48a502c9d735.png)

Running `nikto` also nothing.

![image](https://user-images.githubusercontent.com/69868171/120633015-a2e39000-c437-11eb-9bc4-b098f93da9df.png)

Some i decided to check the `robots.txt` always check it.

![image](https://user-images.githubusercontent.com/69868171/120633138-c9a1c680-c437-11eb-8b2c-1c1550aa410c.png)

Now i try checking the source code again now boom some secret text probably encoded.

![image](https://user-images.githubusercontent.com/69868171/120633255-efc76680-c437-11eb-8c69-ca68736495db.png)

Now let try decoding it to see what we have.

![image](https://user-images.githubusercontent.com/69868171/120633654-62384680-c438-11eb-9c20-14e347418b17.png)

A secret directory `/978345210/index.php` nice let hit it.

![image](https://user-images.githubusercontent.com/69868171/120633896-aa576900-c438-11eb-808a-684a0a9b5467.png)

Now we have a login page i have the habit of always testing for SQL injection first so i try the basic sql injection command now luck so let intercept it and use `sqlmap` .

![image](https://user-images.githubusercontent.com/69868171/120634991-ec34df00-c439-11eb-9bb9-97fd74fe2b1c.png)

Save in a txt file, sacnning with `sqlmap`

`sqlmap -r scan.txt --dbs --columns` command am using for the `sqlmap` .

![image](https://user-images.githubusercontent.com/69868171/120635930-028f6a80-c43b-11eb-8145-67e62f7a3ae7.png)

We are right it vulnerable to SQL injection let dump it.

