---
layout: default
title : muzec - JoyStick Writeup
---

![Image](https://miro.medium.com/max/700/1*KdskuMV6EJIHUvyfsLXNFg.png)

# PORT SCANNING.
```
Nmap 7.80 scan initiated Thu Jul 16 10:08:10 2020 as: nmap -sC -sV -oA nmap 10.10.191.17
Nmap scan report for 10.10.191.17
Host is up (0.16s latency).
Not shown: 997 closed ports
PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 3.0.3
|_ftp-anon: got code 500 “OOPS: vsftpd: refusing to run with writable root inside chroot()”.
22/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 c7:ce:5d:fa:24:68:3a:10:63:f9:28:1b:f4:6d:e5:bc (RSA)
| 256 6b:7b:f5:12:e0:db:bb:b0:ca:f8:f8:c0:84:bc:27:e6 (ECDSA)
|_ 256 1b:d4:20:23:d0:5b:32:16:ad:c2:a9:cd:99:1c:e6:6e (ED25519)
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: JoyStick Gaming
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Thu Jul 16 10:08:53 2020–1 IP address (1 host up) scanned in 42.44 seconds
```

# 3 OPEN PORTS 21, 22 AND 80.

Let try FTP login maybe we have access to anonymous details but unfortunately i get some error.

![Image](https://miro.medium.com/max/700/1*cGvVSrX5zRwXUYmR6VE6gg.png)

Now let try to access the IP

![Image](https://miro.medium.com/max/700/1*jK7xc081TtBrVX2XkVA2zA.png)

Just a simple page let view the page source maybe we can found a hint or anything strange.

![Image](https://miro.medium.com/max/700/1*YVe7i0axxmnXQqDy2NLgeg.png)

Cool we found something a message to one of the user let some some ssh brute forcing calling Daddy Hydra to work lol.

![Image](https://miro.medium.com/max/700/1*zoxWBcKCfH1ph_tWMSzuCQ.png)

Boom we have the details trying the rockyou.txt is taking to long changing wordlist to `usr/share/wordlists/fasttrack.txt` we have the credential with no time no move to next step.

![Image](https://miro.medium.com/max/700/1*H0Z_ep7n_xoQXy89YJYpIg.png)

Boom we are in and we have user.txt let go for root now.

![Image](https://miro.medium.com/max/700/1*-46KMAxuE94pxevUWeR1ZQ.png)

Going back to the home dir we found another user notch which root flag boom box completed.


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
