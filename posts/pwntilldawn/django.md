---
layout: default
title : muzec -Django Writeup
---

PwnTillDawn Online Battlefield is a penetration testing lab created by wizlynx group where participants can test their offensive security skills in a safe and legal environment, but also having fun! The goal is simple, break into as many machines as possible using a succession of weaknesses and vulnerabilities and collect flags to prove the successful exploitation. Each target machine that can be compromised contains at least one “FLAG” (most of the times a file and typically located in the user’s Desktop, or the user’s root directory), which you must retrieve, and submit in the application. The flag is in the majority of the cases in a SHA1 format but not always.

![image](https://user-images.githubusercontent.com/69868171/121949232-382f3000-cd26-11eb-9456-426bc15f1ada.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Mon Jun 14 12:40:49 2021 as: nmap -sC -sV -oA nmap 10.150.150.212
Nmap scan report for 10.150.150.212
Host is up (0.18s latency).
Not shown: 986 closed ports
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220-Wellcome to Home Ftp Server!
|     Server ready.
|     command not understood.
|     command not understood.
|   Help: 
|     220-Wellcome to Home Ftp Server!
|     Server ready.
|     'HELP': command not understood.
|   NULL, SMBProgNeg: 
|     220-Wellcome to Home Ftp Server!
|     Server ready.
|   SSLSessionReq: 
|     220-Wellcome to Home Ftp Server!
|     Server ready.
|_    command not understood.
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drw-rw-rw-   1 ftp      ftp            0 Mar 26  2019 . [NSE: writeable]
| drw-rw-rw-   1 ftp      ftp            0 Mar 26  2019 .. [NSE: writeable]
| drw-rw-rw-   1 ftp      ftp            0 Mar 13  2019 FLAG [NSE: writeable]
| -rw-rw-rw-   1 ftp      ftp        34419 Mar 26  2019 xampp-control.log [NSE: writeable]
|_-rw-rw-rw-   1 ftp      ftp          881 Nov 13  2018 zen.txt [NSE: writeable]
|_ftp-bounce: bounce working!
| ftp-syst: 
|_  SYST: Internet Component Suite
80/tcp    open  http         Apache httpd 2.4.34 ((Win32) OpenSSL/1.0.2o PHP/5.6.38)
|_http-server-header: Apache/2.4.34 (Win32) OpenSSL/1.0.2o PHP/5.6.38
| http-title: Welcome to XAMPP
|_Requested resource was http://10.150.150.212/dashboard/
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.34 ((Win32) OpenSSL/1.0.2o PHP/5.6.38)
|_http-server-header: Apache/2.4.34 (Win32) OpenSSL/1.0.2o PHP/5.6.38
| http-title: Welcome to XAMPP
|_Requested resource was https://10.150.150.212/dashboard/
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: PWNTILLDAWN)
3306/tcp  open  mysql        MariaDB (unauthorized)
8089/tcp  open  ssl/http     Splunkd httpd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Splunkd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2019-10-29T14:31:26
|_Not valid after:  2022-10-28T14:31:26
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  unknown
49158/tcp open  msrpc        Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.91%I=7%D=6/14%Time=60C78702%P=x86_64-pc-linux-gnu%r(NULL
SF:,35,"220-Wellcome\x20to\x20Home\x20Ftp\x20Server!\r\n220\x20Server\x20r
SF:eady\.\r\n")%r(GenericLines,79,"220-Wellcome\x20to\x20Home\x20Ftp\x20Se
SF:rver!\r\n220\x20Server\x20ready\.\r\n500\x20'\r':\x20command\x20not\x20
SF:understood\.\r\n500\x20'\r':\x20command\x20not\x20understood\.\r\n")%r(
SF:Help,5A,"220-Wellcome\x20to\x20Home\x20Ftp\x20Server!\r\n220\x20Server\
SF:x20ready\.\r\n500\x20'HELP':\x20command\x20not\x20understood\.\r\n")%r(
SF:SSLSessionReq,89,"220-Wellcome\x20to\x20Home\x20Ftp\x20Server!\r\n220\x
SF:20Server\x20ready\.\r\n500\x20'\x16\x03\0\0S\x01\0\0O\x03\0\?G\xd7\xf7\
SF:xba,\xee\xea\xb2`~\xf3\0\xfd\x82{\xb9\xd5\x96\xc8w\x9b\xe6\xc4\xdb<=\xd
SF:bo\xef\x10n\0\0\(\0\x16\0\x13\0':\x20command\x20not\x20understood\.\r\n
SF:")%r(SMBProgNeg,35,"220-Wellcome\x20to\x20Home\x20Ftp\x20Server!\r\n220
SF:\x20Server\x20ready\.\r\n");
Service Info: Hosts: Wellcome, DJANGO; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 3h51m37s, deviation: 2s, median: 3h51m35s
| smb-os-discovery: 
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: Django
|   NetBIOS computer name: DJANGO\x00
|   Workgroup: PWNTILLDAWN\x00
|_  System time: 2021-06-14T20:35:22+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-06-14T20:35:26
|_  start_date: 2020-04-02T14:41:43

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun 14 12:44:05 2021 -- 1 IP address (1 host up) scanned in 196.54 seconds
```

We have ports open lol FTP,HTTP,SMB lot of it i know we have `anonymous` access to FTP but i decided to check SMB first.

![image](https://user-images.githubusercontent.com/69868171/121950959-4bdb9600-cd28-11eb-84eb-ed83702c008e.png)

But no shares let check port 80 first before coming back to the FTP.

![image](https://user-images.githubusercontent.com/69868171/121951024-64e44700-cd28-11eb-8ab5-fae41c03d275.png)

Dashboard cool able to access the `phpinfo.php` page just some information about the web server now let try check the `phpmyadmin` page.

![image](https://user-images.githubusercontent.com/69868171/121951361-d623fa00-cd28-11eb-84c9-4c29eb8808c8.png)

Trying default credentials no luck so let get back to the FTP.

![image](https://user-images.githubusercontent.com/69868171/121951534-0b304c80-cd29-11eb-9c38-cbbe9465961b.png)

Nice we have the `FLAG19.txt` but nothing stand out after the FLAG directory so i try `FTP Directory Traversal` and luckily i was able to see all directory.

![image](https://user-images.githubusercontent.com/69868171/121951799-63ffe500-cd29-11eb-9438-d7703c25e0af.png)

Now let change directory using `cd /..` and boom we are in.

![image](https://user-images.githubusercontent.com/69868171/121951908-885bc180-cd29-11eb-86e3-6a34cd01de05.png)

Checking the `xampp` directory and we have FLAG20.txt also a passwords.txt file containing logins details for Phpmyadmin.

![image](https://user-images.githubusercontent.com/69868171/121952489-4aab6880-cd2a-11eb-8766-7ff4d9f90a4c.png)

Now let log in Phpmyadmin with the credentials.

![image](https://user-images.githubusercontent.com/69868171/121952682-94944e80-cd2a-11eb-8fee-c721c96b6eec.png)

FLAG 18 gotten also time to get the last FLAG going back to the FTP server and i found the last FLAG.

![image](https://user-images.githubusercontent.com/69868171/121954452-c27a9280-cd2c-11eb-9a56-1aeed6fa7f95.png)

Easy right also no need to get a reverse shell back to our terminal guess we are done.

PWNTILLDAWN LABS IS THE BEST WHY NOT TRY IT: [Django On PwnTillDawn Click Here](https://online.pwntilldawn.com/Target/Show/7)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


