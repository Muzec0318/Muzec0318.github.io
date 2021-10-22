---
layout: default
title : muzec - Noob Writeup
---


### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/noob]
└─$ nmap -sC -sV -p- -oA nmap  172.16.139.238 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-22 12:00 WAT
Nmap scan report for 172.16.139.238
Host is up (0.00016s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 66:6a:8e:22:cd:dd:75:52:a6:0a:46:06:bc:df:53:0f (RSA)
|   256 c2:48:46:33:d4:fa:c0:e7:df:de:54:71:58:89:36:e8 (ECDSA)
|_  256 5e:50:90:71:08:5a:88:62:7e:81:07:c3:9a:c1:c1:c6 (ED25519)
65530/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.29 seconds
```

We have two open ports ssh and HTTP running on 65530 let confirm it.


![image](https://user-images.githubusercontent.com/69868171/138470222-af913a72-4791-4e7e-bc3b-b8441c700c45.png)

404 page not found interesting let burst for directory.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/noob]                                                                                                                 
└─$ gobuster dir -u http://172.16.139.238:65530/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,phtml,html,txt 
```

![image](https://user-images.githubusercontent.com/69868171/138470442-c31a09dd-b6c3-4c88-8e98-8d0739e0d189.png)

We found a single directory let access it.

![image](https://user-images.githubusercontent.com/69868171/138470601-4025c56a-c490-4846-bdca-2ccce3aa5200.png)

Interesting and cool between we all love access to `.ssh` folder lol with some juicy ssh private key.

![image](https://user-images.githubusercontent.com/69868171/138471020-c537b95d-ccfe-42a4-ae02-2f92dba2c189.png)

First the public key to check for the user.

![image](https://user-images.githubusercontent.com/69868171/138471387-0498bd36-573b-41cc-bbeb-78531a75cc0b.png)

We have username now time to get the private key.

![image](https://user-images.githubusercontent.com/69868171/138471542-8a614bfd-1001-484a-bd28-08d4d2575f57.png)

Now let hit SSH.


![image](https://user-images.githubusercontent.com/69868171/138471781-fb29b99d-3009-4c18-8779-b23cf8b9919f.png)

We are in let get root.

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/138472142-14cdcc73-d2bd-4e3e-b43f-d220b0ffe0e0.png)

Check for `SUID` and `sudo -l` we go nothing let try to check process that is running like man i got nothing also running `linpeas.sh` but my thinking was is it possible the `nt4share` is being running by root?? let give it a try creating a symbolic link with the root folder.

![image](https://user-images.githubusercontent.com/69868171/138475210-8816623e-060e-42c9-a43f-0a78f631ff43.png)

Created and listing directories.

![image](https://user-images.githubusercontent.com/69868171/138475469-89bc7705-2e8b-4c03-9b68-6b347b973027.png)

Accessing the `nt4share` again.

![image](https://user-images.githubusercontent.com/69868171/138475740-1edf18a1-2a58-4393-8b03-593e650e165e.png)


Boom we have access to the root folder now let get the ssh private key.

![image](https://user-images.githubusercontent.com/69868171/138476114-c355478c-c8c1-4d56-a3fd-85b9815739a6.png)

Hitting SSH.

![image](https://user-images.githubusercontent.com/69868171/138476447-c7dfcc89-8f5b-4c95-8272-32ce7a4d5a2d.png)



We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
