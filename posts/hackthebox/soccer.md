---
layout: default
title : muzec - Soccer HTB Writeup
---


![image](https://user-images.githubusercontent.com/69868171/218117279-f27eebec-fbed-4900-a1e9-7cdd0c83c12c.png)


#### RECON

Enumeration is the key to everthing so we always kick off with an nmap scan on the target IP.

```
└─$ nmap -p- --min-rate 2000 -sC -sV -oN nmap/soccer.tcp 10.10.11.194
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-10 15:38 WAT
Warning: 10.10.11.194 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.194
Host is up (0.27s latency).
Not shown: 38489 filtered tcp ports (no-response), 27044 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
|_  256 5797565def793c2fcbdb35fff17c615c (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 286.53 seconds
```

Now it connfirm we have SSH 22 and HTTP 80 port presently open the target which is interesting since it rated easy i think we already know we don't need to dig that much we can start a quick enumeration on the HTTP port already.

```
└─$ curl -i http://10.10.11.194
HTTP/1.1 301 Moved Permanently
Server: nginx/1.18.0 (Ubuntu)
Date: Fri, 10 Feb 2023 14:48:02 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://soccer.htb/

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
```

Some old habit i always use to print out the response headers in the output using `curl` command now that we have the domain let add to our hosts file.

```
sudo vim /etc/hosts

.......
10.10.11.194        soccer.htb
.......
```

Now that we have that set let access the web-server surely we are using the browser XD.

![image](https://user-images.githubusercontent.com/69868171/218122187-0bc713c6-ee50-4e23-bcf2-0cc4a0a3294a.png)


Interesting a web page that talk on how soccer is the best sports in the world which is true no cap but i still take hacking like sports XD which mean sports is second to me.

#### Subdomain Enumeration

```
┌──(muzec㉿muzec-sec)-[~/Documents/CTF Hacking/HTB/soccer]
└─$ wfuzz -u http://soccer.htb/ -H "Host: FUZZ.soccer.htb" -w /opt/wordlists/subdomains.txt --hh 178
```

