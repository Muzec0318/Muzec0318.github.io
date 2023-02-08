---
layout: default
title : muzec - Kioptrix 2 Writeup
---

![Image](https://imgur.com/nh457od.png)

Let Get Started With an Nmap Scan...

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri Apr  2 15:19:10 2021 as: nmap -sC -sV -oA nmap 172.16.139.134
Nmap scan report for 172.16.139.134
Host is up (0.0041s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 3.9p1 (protocol 1.99)
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http       Apache httpd 2.0.52 ((CentOS))
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            805/udp   status
|_  100024  1            808/tcp   status
443/tcp  open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-10-08T00:10:47
|_Not valid after:  2010-10-08T00:10:47
|_ssl-date: 2021-04-02T23:19:23+00:00; +3h59m59s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC4_64_WITH_MD5
631/tcp  open  ipp        CUPS 1.1
| http-methods: 
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
808/tcp  open  status     1 (RPC #100024)
3306/tcp open  mysql      MySQL (unauthorized)

Host script results:
|_clock-skew: 3h59m58s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr  2 15:19:24 2021 -- 1 IP address (1 host up) scanned in 14.68 seconds
```

Some really Interesting ports but let hit HTTP port 80.

![Image](https://imgur.com/nh457od.png)

A Remote System Administration Login page let try admin and admin for username and password but no luck let try some simple sql injection on it.

![Image](https://imgur.com/WWSo1f7.png)

And boom weare in.

![Image](https://imgur.com/XOkFMgB.png)

Hmm a ping machine network let it 127.0.0.1 yea a localhost.

![Image](https://imgur.com/ym7yWGP.png)

Cool we can easily execute command with that by adding some system commands to it.

![image](https://imgur.com/MJE1JZN.png)

![Image](https://imgur.com/4hnCs32.png)

Now let get a reverse shell and make sure you start an ncat listener.

![Image](https://imgur.com/wDwpoQM.png)

And boom  we have shell.

![Image](https://imgur.com/FSh0T4i.png)

Cool now let Privilege Escalate to get root on the target.

![Image](https://imgur.com/zbPGW8f.png)

Yea pretty old CentOs 4.5 let use searchsploit to find some exploit.

![Image](https://imgur.com/68hgAz7.png)

That look promising let try it.

![image](https://imgur.com/vzlpLK7.png)

Cool guess we have to complie it on the target let get the exploit to our target with python SimpleHTTPServer, now that we have the exploit on our target let compile it

```gcc -o exploit exploit.c```

![Image](https://imgur.com/IuMXHxC.png)

Now let run it.

![Image](https://imgur.com/z4g6h66.png)

And Boom Box Rooted.....


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
