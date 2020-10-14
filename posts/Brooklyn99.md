---
layout: default
title : muzec - Brooklyn 99 Writeup
---

![Image](https://miro.medium.com/max/700/1*4GgvWifY72NFwXcRbugWBw.jpeg)

This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.

Refreshing For Beginner………..

# SCANNING

```Nmap 7.80 scan initiated Sat Jul 25 23:15:08 2020 as: nmap -sC -sV -oA nmap 10.10.131.19
Nmap scan report for 10.10.131.19
Host is up (0.23s latency).
Not shown: 997 closed ports
PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r — r — 1 0 0 119 May 17 23:17 note_to_jake.txt
| ftp-syst:
| STAT:
| FTP server status:
| Connected to ::ffff:10.0.0.20
| Logged in as ftp
| TYPE: ASCII
| No session bandwidth limit
| Session timeout in seconds is 300
| Control connection is plain text
| Data connections will be plain text
| At session startup, client count was 2
| vsFTPd 3.0.3 — secure, fast, stable
|_End of status
22/tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
| 256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_ 256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open http Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn’t have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Sat Jul 25 23:15:34 2020–1 IP address (1 host up) scanned in 26.38 seconds```

We have three open ports 21,22 and 80 very interesting observing my scan output we find out port 21 which is the FTP we have access to the Anonymous FTP login cool guess it our lucky day.

![Image](https://miro.medium.com/max/700/1*LUZ84uNfw7Bpj-il27qNTA.png)

And we are in a note hmm a secret note to jake we can easily use get 
```note_to_jake.txt``` to get it to our machine.

![Image](https://miro.medium.com/max/700/1*WvNq_k7zUyCXoJY7TBXSmw.png)

Done let cat the txt file to see what is the secret note.

![Image](https://miro.medium.com/max/700/1*my34spF-H_5WPvxsI5E3AQ.png)

WTF! only username i was thinking i will get some full details like username and password for SSH but no problem let try some SSH brute force with hydra.

![Image](https://miro.medium.com/max/700/1*-uTFkSdxBdJD9wwNVIu-tA.png)

Hell Yea… we have the password time to log in SSH to get user flag that was way to easy.

![Image](https://miro.medium.com/max/700/1*baE01IMKoIMppMxYOlrGXg.png)

Boom we are in let get the flag.

![Image](https://miro.medium.com/max/700/1*Ioiuj3ZLMtl_viIbgAnaEw.png)

We have user.txt let go for the root flag now time for Privilege Escalation.

![Image](https://miro.medium.com/max/700/1*WVQwx8Lgxel87rZI0sFDnA.png)

Let try checking some sudo right using ```sudo -l```

![Image](https://miro.medium.com/max/700/1*ntDUTirHB-6xWaf3zDw9Fg.png)

Let try it.

![Image](https://miro.medium.com/max/700/1*Z7yG8FCgQDRPejkz0yaqtQ.png)

Boom we are root and we also have the root flag.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

