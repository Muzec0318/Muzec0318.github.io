---
layout: default
title : muzec - Potato - Vulnhub
---


### Enumeration With Nmap

```
# Nmap 7.91 scan initiated Thu Nov 11 08:29:19 2021 as: nmap -sC -sV -oA nmap 192.168.227.101
Nmap scan report for 192.168.227.101
Host is up (0.21s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
|   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
|_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Potato company
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Nov 11 08:30:25 2021 -- 1 IP address (1 host up) scanned in 66.47 seconds
```
We have 2 open ports but let scan for full ports to be sure.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/Potato]
└─$ nmap -p- 192.168.152.101 -T4
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-11 10:21 WAT
Warning: 192.168.152.101 giving up on port because retransmission cap hit (6).
Stats: 0:06:05 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 29.42% done; ETC: 10:42 (0:14:36 remaining)
Stats: 0:14:07 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan                                                                                            
Connect Scan Timing: About 58.10% done; ETC: 10:45 (0:10:11 remaining)
Stats: 0:22:11 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 92.85% done; ETC: 10:45 (0:01:42 remaining)
Nmap scan report for 192.168.152.101
Host is up (0.24s latency).        
Not shown: 65447 closed ports, 85 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh                                                                                                                                                     
80/tcp   open  http                                                                
2112/tcp open  kip
                                                                                   
Nmap done: 1 IP address (1 host up) scanned in 1451.87 seconds      
```

Confirming what is running on the port 2112.


```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/Potato]
└─$ nmap -p2112 -sC -sV 192.168.152.101 -T4
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-11 10:47 WAT
Nmap scan report for 192.168.152.101
Host is up (0.37s latency).

PORT     STATE SERVICE VERSION
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.63 seconds
```

Now it getting interesting seems we have another open port which is running FTP server smooth right? and we have anonymous access let hit it.

### Enumeration On Port 2112 FTP


```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/Potato]
└─$ ftp  192.168.152.101  2112
Connected to 192.168.152.101.
220 ProFTPD Server (Debian) [::ffff:192.168.152.101]
Name (192.168.152.101:muzec): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.49.152 !
230-
230-The local time is: Thu Nov 11 12:54:32 2021
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   2 ftp      ftp          4096 Aug  2  2020 .
drwxr-xr-x   2 ftp      ftp          4096 Aug  2  2020 ..
-rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
226 Transfer complete
ftp> get index.php.bak
local: index.php.bak remote: index.php.bak
200 PORT command successful
150 Opening BINARY mode data connection for index.php.bak (901 bytes)
226 Transfer complete
901 bytes received in 0.00 secs (8.0305 MB/s)
ftp> 
```

Smooth we have a backup file of the `index.php` we will get back to it let check what we have on the HTTP web server.


### Enumeration On Port 80 HTTP


![image](https://user-images.githubusercontent.com/69868171/141302528-f54cd952-bf19-4149-a3e6-00655c9386f9.png)

Potato Company with a dress lol checking the source page i got nothing seems it time to brute force for some directories.

![image](https://user-images.githubusercontent.com/69868171/141303675-3501076d-518e-4b0d-a500-6f11cbf67522.png)

We found an admin directory that lead to a login page.
