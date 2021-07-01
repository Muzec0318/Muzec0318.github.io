---
layout: default
title : muzec - Couch Writeup
---

![image](https://user-images.githubusercontent.com/69868171/124112379-3c529180-da38-11eb-89b5-5dfb21f12090.png)

We always start with an nmap scan.....

```Nmap -sC -p1-6000 -sV -oA <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/couch]
└─$ nmap -sC -p1-6000 -sV -oA nmap 10.10.128.130
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-01 03:45 EDT
Stats: 0:04:58 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 42.33% done; ETC: 03:56 (0:06:46 remaining)
Stats: 0:11:15 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 80.25% done; ETC: 03:59 (0:02:46 remaining)
Nmap scan report for 10.10.128.130
Host is up (0.21s latency).
Not shown: 5998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:9d:39:09:34:30:4b:3d:a7:1e:df:eb:a3:b0:e5:aa (RSA)
|   256 a4:2e:ef:3a:84:5d:21:1b:b9:d4:26:13:a5:2d:df:19 (ECDSA)
|_  256 e1:6d:4d:fd:c8:00:8e:86:c2:13:2d:c7:ad:85:13:9c (ED25519)
5984/tcp open  http    CouchDB httpd 1.6.1 (Erlang OTP/18)
|_http-server-header: CouchDB/1.6.1 (Erlang OTP/18)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 904.98 seconds
```

### Now time to answer all the tasks questions it easy.

Scan the machine, how many ports are open?

Our Nmap only show 2 ports so the answer is `2`
    
### What's is the database management system installed on the server?

Nmap result also show the database which the answer is `couchdb`

### What port is the database management system running on?

We can see it from the Nmap result also answer is `5984`

### What's is the version of management system installed on the server?

`1.6.1`

### What is path for the web administration tool for this database management system?

`_utils`

### What is path for list all databases in the web browser of the database management system?

`_all_dbs`

### What is the credentials founed in the web administration tool?

Now it when the fun begin let navigate `http://10.10.128.130:5984/_utils/` yes it such a nice interface i know .

![image](https://user-images.githubusercontent.com/69868171/124116182-a5d49f00-da3c-11eb-8cd3-68dc712017f0.png)

We all love secret lol so am checking first clicking on the secret database.

![image](https://user-images.githubusercontent.com/69868171/124116375-e502f000-da3c-11eb-9039-1a77c5e88076.png)

Now the key let hit it.

![image](https://user-images.githubusercontent.com/69868171/124116563-17ace880-da3d-11eb-9903-7481417ed503.png)

Boom we have the credentials.

### Compromise the machine and locate user.txt

Now let use the credentials to log in SSH.

![image](https://user-images.githubusercontent.com/69868171/124117062-aa4d8780-da3d-11eb-9f78-7dd1e36ebe7a.png)

We have user.txt nice right time to get root.

### Escalate privileges and obtain root.txt

![image](https://user-images.githubusercontent.com/69868171/124117261-e7b21500-da3d-11eb-8586-d668d764b945.png)

`127.0.0.1:2375` port 2375 so i try doing some research on it and i found out docker is runnig on the port nice is it possible to escape it let give it a try.

Remote API is running by default on 2375 port when enabled. The service by default will not require authentication allowing an attacker to start a privileged docker container. By using the Remote API one can attach hosts / (root directory) to the container and read/write files of the host’s environment.

`Default port: 2375`

Now for our payload `docker -H 127.0.0.1:2375 run --rm -it --privileged --net=host -v /:/root alpine` let try running it on the target.

![image](https://user-images.githubusercontent.com/69868171/124117847-99e9dc80-da3e-11eb-8a5d-4653f7c76783.png)

We are root nice and we have our root flag.

### Probably Unintended Way To Obtain Root.

So after finishing the box i try to dig deeper on the kernal version.

![image](https://user-images.githubusercontent.com/69868171/124118067-e6cdb300-da3e-11eb-911c-b2de7917c9a2.png)

Yes all running old version so let try the OverlayFS - Local Privilege Escalation - CVE-2021-3493 which i have already here on my blog [OverlayFS - Local Privilege Escalation - CVE-2021-3493 (POC)](https://muzec0318.github.io/posts/overlayfs.html)

![image](https://user-images.githubusercontent.com/69868171/124118725-a02c8880-da3f-11eb-954a-b635fda97b11.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


