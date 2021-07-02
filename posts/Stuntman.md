---
layout: default
title : muzec -  Stuntman Mike  Writeup
---

PwnTillDawn Online Battlefield is a penetration testing lab created by wizlynx group where participants can test their offensive security skills in a safe and legal environment, but also having fun! The goal is simple, break into as many machines as possible using a succession of weaknesses and vulnerabilities and collect flags to prove the successful exploitation. Each target machine that can be compromised contains at least one “FLAG” (most of the times a file and typically located in the user’s Desktop, or the user’s root directory), which you must retrieve, and submit in the application. The flag is in the majority of the cases in a SHA1 format but not always.

![image](https://user-images.githubusercontent.com/69868171/121824198-53436680-cc78-11eb-829c-4fed65b720e1.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PTD/10.150.150.166]
└─$ nmap -sC -sV -oA nmap  10.150.150.166 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-13 15:55 EDT
Nmap scan report for 10.150.150.166
Host is up (0.17s latency).
Not shown: 983 closed ports
PORT      STATE    SERVICE              VERSION
22/tcp    open     ssh                  OpenSSH 7.6p1 (protocol 2.0)
| ssh-hostkey: 
|   2048 b7:9e:99:ed:7e:e0:d5:83:ad:c9:ba:7c:f1:bc:44:06 (RSA)
|   256 7e:53:59:7b:2d:6c:3b:d7:21:28:cb:cb:78:af:99:78 (ECDSA)
|_  256 c5:d2:2d:04:f9:69:40:4c:15:34:36:fe:83:1f:f3:44 (ED25519)
1050/tcp  filtered java-or-OTGfileshare
1334/tcp  filtered writesrv
1580/tcp  filtered tn-tl-r1
1666/tcp  filtered netview-aix-6
2608/tcp  filtered wag-service
3325/tcp  filtered active-net
3551/tcp  filtered apcupsd
3659/tcp  filtered apple-sasl
4126/tcp  filtered ddrepl
5087/tcp  filtered biotic
5822/tcp  filtered unknown
8089/tcp  open     ssl/http             Splunkd httpd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2019-10-25T09:15:13
|_Not valid after:  2022-10-24T09:15:13
8254/tcp  filtered unknown
9595/tcp  filtered pds
40911/tcp filtered unknown
65129/tcp filtered unknown

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 269.79 seconds
```

We are having only 2 open ports SSH and Spluck going through spluck first.

![image](https://user-images.githubusercontent.com/69868171/121824525-ccdc5400-cc7a-11eb-8b5f-aa023bc0c311.png)

Try some default credentials but no luck so i decide to go back to the SSH port let try and connect to SSH.

![image](https://user-images.githubusercontent.com/69868171/121824561-057c2d80-cc7b-11eb-8ff0-9ad8a501739a.png)

Boom we have the first flag also a name let try to brute force SSH with hydra.

![image](https://user-images.githubusercontent.com/69868171/121824590-3fe5ca80-cc7b-11eb-9120-66f35b667b58.png)

Cool now let log in SSH with the credentials.

![image](https://user-images.githubusercontent.com/69868171/121824638-96eb9f80-cc7b-11eb-9e39-484bd70fb036.png)

We are in and also we got another flag time to get root.

### Privliege Escalation

Typing `sudo -l`  and inserting the password.

![image](https://user-images.githubusercontent.com/69868171/121824678-f053ce80-cc7b-11eb-963f-7c4b774b0ee0.png)

We can run all to get root access so i type `sudo su` and we are root.

![image](https://user-images.githubusercontent.com/69868171/121824698-1da07c80-cc7c-11eb-9221-b16a6ebfc958.png)

We have the last flag and we are root.

PWNTILLDAWN LABS IS THE BEST WHY NOT TRY IT: [Stuntman Mike On PwnTillDawn Click Here](https://online.pwntilldawn.com/Target/Show/1)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
