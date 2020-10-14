---
layout: default
title : muzec - Overpass Writeup
---

![Image](https://miro.medium.com/max/490/1*tWD5crPD4r7m_DY8T17uIg.png)

```What happens when some broke CompSci students make a password manager?```

So i finally root Overpass on Tryhackme took me days with some little help from friends also a little cheating looking at the Privilege Escalation in the root part let get started.

# SCANNING.


```Nmap 7.80 scan initiated Sat Jul 18 08:33:44 2020 as: nmap -sC -sV -oA nmap 10.10.113.243
Nmap scan report for 10.10.113.243
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
| 256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_ 256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open http Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Sat Jul 18 08:34:43 2020–1 IP address (1 host up) scanned in 58.78 seconds```


# ACCESSING IP.

![Image](https://miro.medium.com/max/700/1*wS7uBmj3vheJA00MePJWBg.png)

I try checking some pages nothing interesting about it ```About Us, Downloads``` also page source nothing special so i try to busts some dirs with Gobuster.


![Image](https://miro.medium.com/max/700/1*e1wIvE2fj4NXXMQadvqtXg.png)

The only interesting dir is ```/admin``` let try to access it maybe luckily we can find anything boom yea boom lol.



![Image](https://miro.medium.com/max/700/1*Pe1kIBYe9W3Ab8gnd68kYQ.png)

Not the boom am expecting it Overpass administrator login page hmm let try a simple username and password admin and admin hmm no luck let try simple SQL injection admin'— 1=1' — no luck let view some page source.


![Image](https://miro.medium.com/max/700/1*dMZYHeXTcYqWNdW6e7UHSA.png)

The login.js let check maybe a hint is hidden or anything that can lead us to be able to log in.


![image](https://miro.medium.com/max/700/1*XQAl95i9h2D13R4YlwrIPA.png)

Ok i just got brainfuck lol what i find interesting is the cookie and cookie ```Cookies.set(“SessionToken”,statusOrCookie)``` checking the page with cookie editor we have no active cookies.


![Image](https://miro.medium.com/max/700/1*47kxsxCqrthlgAkvBKbLqQ.png)

Can i create a fake cookies?? maybe or maybe not can i bypass the login page no harm in trying let give it a try.


![Image](https://miro.medium.com/max/700/1*qPZOSqyBLVRrnnOamxfBKA.png)

So i created two cookies with cookie editor 1 ```statusOrCookie with the incorrect Credentials``` 2 ```SessionToken with correct Credentials.```

![Image](https://miro.medium.com/max/700/1*j-0FZXc3pFxCea09sxGPfw.png)


![Image](https://miro.medium.com/max/700/1*de7giRzl9o1Ub5y6YnpyKQ.png)

Ok let refresh the page.


![Image](https://miro.medium.com/max/700/1*aCl0FI0xCms5Ti7jskY4bw.png)

That the boom i want RSA PRIVATE KEY for SSH ok let move on to the next step.


![Image](https://miro.medium.com/max/700/1*cYf7Y7j3CxxJX81Zh_tPeg.png)

The username should be james let save the SSH keys with the name ```id_rsa``` also give it permission with ```chmod 600 id_rsa``` and log in.


![Image](https://miro.medium.com/max/700/1*Fn7r7f2YmNKbx99iHWJsSQ.png)

Hmm we need a passphrase to be able to log in time to call john the ripper using the ssh2john to crack the SSH key ssh2john id_rsa after that copy the text you see in the screen save it.


![Image](https://miro.medium.com/max/700/1*c9IDZXI6tWEPUeCknR81Hw.png)

To crack the file you save use the command ```sudo john — wordlist=rockyou.txt``` with the file you save in no time you will have the password.


![Image](https://miro.medium.com/max/700/1*SosiCVXz1_J7En5dm-lBWA.png)

Am in now we have user.txt time to get root Privilege Escalation part not that easy for me lol.


![Image](https://miro.medium.com/max/480/1*Tbv_3JSS_8pJdeIV6Pj3tw.gif)

Trying ```Sudo -l``` no luck ok i use a SimpleHTTPServer to transfer Linenum to the victim machine from my own machine cool right? let run LinEnum.sh.


![Image](https://miro.medium.com/max/700/1*Q2AwPMGlkzz_iRCPtrwiFA.png)

Cronjob running also we have write permission in the /etc/hosts cool let try to create a fake files to get a reverse shell to machine by editing the /etc/hosts adding our VPN IP to get connection back to our machine.


![Image](https://miro.medium.com/max/700/1*1rLtuZQFNZAmXTBPQ_uPbA.png)

Going back to our machine let create a folder like /downloads/src/buildscript.sh the buildscript.sh will contain our reverse shell code which you can get here [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).


![image](https://miro.medium.com/max/700/1*AHpz2oTAk07kgIWEbXR8wg.png)

Ok start an ncat connection to listen to incoming connection also a SimpleHTTPServer to transfer the file to the victim machine to get the root shell.


![Image](https://miro.medium.com/max/700/1*GuYbKvEnu4Dw97w-w5flCw.png)

Boom and we have root shell let get our root.txt and we are done rooting overpass.

![Image](https://miro.medium.com/max/700/1*amxGRcZXloEaKPjy5o3ykw.png)

We are Done....Box Rooted


![Image](https://miro.medium.com/max/500/1*cf9w0s7sMTGbPSdYVKnEKQ.gif)


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


