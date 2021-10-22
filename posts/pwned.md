---
layout: default
title : muzec - Pwned Writeup
---


### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Wed Oct  6 16:11:16 2021 as: nmap -sC -sV -p- -oA nmap 172.16.139.242
Nmap scan report for 172.16.139.242
Host is up (0.0024s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 fe:cd:90:19:74:91:ae:f5:64:a8:a5:e8:6f:6e:ef:7e (RSA)
|   256 81:32:93:bd:ed:9b:e7:98:af:25:06:79:5f:de:91:5d (ECDSA)
|_  256 dd:72:74:5d:4d:2d:a3:62:3e:81:af:09:51:e0:14:4a (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Pwned....!!
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Oct  6 16:11:27 2021 -- 1 IP address (1 host up) scanned in 11.28 seconds
```

We have three open ports and seems we don't have access to the ftp server let give it a try.

![image](https://user-images.githubusercontent.com/69868171/138492678-c6c8357f-24ca-47bb-a786-bdac351cdaf8.png)

But seems we have no access to FTP server now let check HTTP.


![image](https://user-images.githubusercontent.com/69868171/138496460-13772af5-e25d-4dbb-a570-532fd7a9b8b0.png)

let check the page source.

![image](https://user-images.githubusercontent.com/69868171/138496598-d4961b87-c99e-4c33-994b-da05f698a926.png)

But it not that helpful let burst some directories.

![image](https://user-images.githubusercontent.com/69868171/138496969-3ee83782-fdfb-407b-ae10-5a85d74910b3.png)

Checking `robots.txt` .

![image](https://user-images.githubusercontent.com/69868171/138497044-fe464ca8-3824-4a73-8d57-569329f460f8.png)

But it lead to a rabbit hole let use medium list to burst directories.

![image](https://user-images.githubusercontent.com/69868171/138497967-84477e15-ee7c-40b8-a5ad-7343aa15f3d7.png)

So let try accessing it man.

![image](https://user-images.githubusercontent.com/69868171/138498078-e67c88e0-a309-4b6c-be28-e48e0807a8fa.png)

Cool a secret.dic file hehehehehehehe.

![image](https://user-images.githubusercontent.com/69868171/138498172-d713dcc7-7303-46ac-b646-b37e61cb02d4.png)

Let use the secret.dic to burst directories again.

![image](https://user-images.githubusercontent.com/69868171/138498333-f9a9192b-5fa5-439d-9aa2-746822ed504c.png)

Smooth let hit it.

![image](https://user-images.githubusercontent.com/69868171/138498493-3dc040ec-9423-4d42-b254-c14e16fb262d.png)

Checking page source.

![image](https://user-images.githubusercontent.com/69868171/138498560-39613dde-467f-4e7f-9134-33482a634848.png)

Boom we found credentials for FTP server smooth.

![image](https://user-images.githubusercontent.com/69868171/138498874-39a35272-35a5-4f12-b1ff-d551f8538cf6.png)

SSH private key and a note let get it to our machine.

![image](https://user-images.githubusercontent.com/69868171/138499328-ac19be48-46c3-4b13-b73b-3e9bc8ff0042.png)

Username for SSH let hit it.

![image](https://user-images.githubusercontent.com/69868171/138499454-da6b6328-408a-457b-8d9a-5d8fd6ed3562.png)

We are in let get root.

### Privilege Escalation


![image](https://user-images.githubusercontent.com/69868171/138499655-7ed83f95-5543-4da8-bb3a-c6162f463c30.png)

Ruuning sudo on the messenger.sh with user selena.


![image](https://user-images.githubusercontent.com/69868171/138499917-936c3eda-7b35-456b-b03d-b2b326d28bd7.png)

Spawn a tty shell and we are good.

![image](https://user-images.githubusercontent.com/69868171/138500139-bdbf7619-4557-4865-8675-dbb0aaaf181f.png)

Interesting we can see docker on the same group with selena let exploit it to gain root shell.

![image](https://user-images.githubusercontent.com/69868171/138502499-45d95fb0-2867-4676-bc8b-2d338d6ea06d.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

