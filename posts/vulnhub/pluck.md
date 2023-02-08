---
layout: default
title : muzec - Pluck Writeup
---


![image](https://user-images.githubusercontent.com/69868171/123263862-6e5b7500-d4c7-11eb-8fd9-5815542bb0e5.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/pluck]
└─$ cat nmap.nmap  
# Nmap 7.91 scan initiated Thu May 27 09:53:42 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.187
Nmap scan report for 172.16.139.187
Host is up (0.00019s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.3p1 Ubuntu 1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e8:87:ba:3e:d7:43:23:bf:4a:6b:9d:ae:63:14:ea:71 (RSA)
|   256 8f:8c:ac:8d:e8:cc:f9:0e:89:f7:5d:a0:6c:28:56:fd (ECDSA)
|_  256 18:98:5a:5a:5c:59:e1:25:70:1c:37:1a:f2:c7:26:fe (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Pluck
3306/tcp open  mysql   MySQL (unauthorized)
5355/tcp open  llmnr?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 27 09:56:10 2021 -- 1 IP address (1 host up) scanned in 148.17 seconds
```

We have our scan result 22,80,3306 and 5355 we know the the web server is running apache let try accessing it to see what we have in the home page.

![image](https://user-images.githubusercontent.com/69868171/123263862-6e5b7500-d4c7-11eb-8fd9-5815542bb0e5.png)

Seems we have some pages let click on `admin` page we have login page trying some default credentials also i try some sql injection but we have no luck.

![image](https://user-images.githubusercontent.com/69868171/123265262-d9f21200-d4c8-11eb-8001-b464ab74a4e7.png)

Let try to check other pages.

![image](https://user-images.githubusercontent.com/69868171/123265485-1aea2680-d4c9-11eb-8b2e-6736e4ab8a8e.png)

Now the page parameter look like it vulnerable to LFI let give it a shot.

![image](https://user-images.githubusercontent.com/69868171/123265922-87fdbc00-d4c9-11eb-9142-0f4a45f5b56e.png)

It vulnerable to LFI cool we can view in page source to see the passwd file in order.

![image](https://user-images.githubusercontent.com/69868171/123268318-edeb4300-d4cb-11eb-896e-ea8a511e2059.png)

`backup-user:x:1003:1003:Just to make backups easier,,,:/backups:/usr/local/scripts/backup.sh` seems cool let try to read it with the LFI.

![image](https://user-images.githubusercontent.com/69868171/123268629-3440a200-d4cc-11eb-82ce-3c99c38aa606.png)

We know the script is backing up directories of the home folders with files to the backups directory so we can get it via tftp.

### What Is TFTP??

Trivial File Transfer Protocol is a simple lockstep File Transfer Protocol which allows a client to get a file from or put a file onto a remote host. One of its primary uses is in the early stages of nodes booting from a local area network. 


Now let get the backup file on the tftp client.

![image](https://user-images.githubusercontent.com/69868171/123272886-2ab93900-d4d0-11eb-809d-fed418da8073.png)

We extract the tar file and we have the home directory so i decide to go through it but the user paul folder have some interesting files in it.

![image](https://user-images.githubusercontent.com/69868171/123274315-686a9180-d4d1-11eb-9a9d-0d85d1e6822a.png)

SSH private keys checking the size i was able to know the real SSH key now let SSH into the machine with user paul.

![image](https://user-images.githubusercontent.com/69868171/123274650-b384a480-d4d1-11eb-9911-e6537eac3f28.png)

But we are stuck in Pdmenu screen.

### What Is Pdmenu??

Pdmenu is a full screen menuing system for Unix. It is designed to be easy to use, and is suitable as a login shell for inexperienced users, or it can just be ran at the command line as a handy menu. ... It was developed on Linux, and has now been compiled on many other unixes without problems.

Let try to escape it.

`Edit file`

![image](https://user-images.githubusercontent.com/69868171/123275439-52a99c00-d4d2-11eb-9e6f-03308e34bb4a.png)


Now let type in anything for the filename and hit enter.

![image](https://user-images.githubusercontent.com/69868171/123275581-6ead3d80-d4d2-11eb-8de6-73b946079aab.png)

And we have vim now let type.

```
:set shell=/bin/sh
:shell 
```

Hitting enter give us a shell.

![image](https://user-images.githubusercontent.com/69868171/123276013-ce0b4d80-d4d2-11eb-9d62-53084ca28c3e.png)


### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/123276658-5ee22900-d4d3-11eb-980d-a68d51c94e97.png)

I transfer `linpeas.sh` on the target now let run it.

![image](https://user-images.githubusercontent.com/69868171/123277576-3b6bae00-d4d4-11eb-9758-04f1891865a2.png)

Seems we have what to abuse to give us root shell with some little research i found the exploit.

![image](https://user-images.githubusercontent.com/69868171/123277846-779f0e80-d4d4-11eb-8162-f9a97f2048d4.png)

Now let get the exploit to the target and run it.

![image](https://user-images.githubusercontent.com/69868171/123280370-b766f580-d4d6-11eb-9ab1-deda4257c4e8.png)

we have root after running the exploit and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
