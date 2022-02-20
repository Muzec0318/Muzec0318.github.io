---
layout: default
title : muzec - Hommie Writeup
---


### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/hommie]
└─$ nmap -sC -sV -p- -oA nmap 172.16.139.240 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-22 13:05 WAT
Nmap scan report for 172.16.139.240
Host is up (0.00021s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0               0 Sep 30  2020 index.html
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.16.139.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 c6:27:ab:53:ab:b9:c0:20:37:36:52:a9:60:d3:53:fc (RSA)
|   256 48:3b:28:1f:9a:23:da:71:f6:05:0b:a5:a6:c8:b7:b0 (ECDSA)
|_  256 b3:2e:7c:ff:62:2d:53:dd:63:97:d4:47:72:c8:4e:30 (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.27 seconds
```

Boom we have three open ports ftp,ssh and http seems we have anonymous access to ftp let try it.

![image](https://user-images.githubusercontent.com/69868171/138478641-296a1405-ce7e-4530-acf9-8c7ad2b944f9.png)

But it lead to a rabbit hole let brute force directory.

![image](https://user-images.githubusercontent.com/69868171/138478983-2b408755-7b01-4fda-bff1-59dabb4c350d.png)

Nothing also just a empty private key so i go back to scan for UDP ports.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/hommie]
└─$ sudo nmap -sU -sV -p1-100 172.16.139.240
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-22 13:11 WAT
Stats: 0:01:04 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 75.40% done; ETC: 13:12 (0:00:21 remaining)
Stats: 0:02:35 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Stats: 0:02:40 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Nmap scan report for 172.16.139.240
Host is up (0.00041s latency).
Not shown: 98 closed ports
PORT   STATE         SERVICE VERSION
68/udp open|filtered dhcpc
69/udp open|filtered tftp
MAC Address: 08:00:27:53:9D:C2 (Oracle VirtualBox virtual NIC)
```
Interesting we have the `tftp` port open let try to get the id_rsa key on it.

![image](https://user-images.githubusercontent.com/69868171/138480164-3d57a9f0-0658-469f-be49-63912accc04b.png)

we know the username is `alexia` let hit SSH.

![image](https://user-images.githubusercontent.com/69868171/138480359-05f429e0-eae4-49c9-b05f-088518edf118.png)

### Privilege Escalation 

![image](https://user-images.githubusercontent.com/69868171/138480609-f789b0a3-a265-4e3e-82d8-03f240b715ae.png)


Command to check for SUID:- `find / -perm -u=s -type f 2>/dev/null` .

We have ```/opt/showMetheKey``` so i decided to strings it to see what the SUID is doing.


![image](https://user-images.githubusercontent.com/69868171/138481293-c989a5e9-6ce5-4493-8f33-19fc8dabb7fd.png)

We know anything we run the SUID binary it check the home directory path and check for the id_rsa private key.

![image](https://user-images.githubusercontent.com/69868171/138481540-766cd78d-9b49-4412-ac8b-0420e202bf9c.png)

Now let abuse it.

![image](https://user-images.githubusercontent.com/69868171/138481884-c9324ef4-4445-494b-b092-a6c488bd0a1e.png)


![image](https://user-images.githubusercontent.com/69868171/138481833-139753ae-5550-4260-a8fb-00b8f025af42.png)

We are root.


### Second Method To Root

Exporting PATH to root.

![image](https://user-images.githubusercontent.com/69868171/138483906-ea08a643-10d4-48e4-9309-295bb6308377.png)

Now let run the SUID binary again.

![image](https://user-images.githubusercontent.com/69868171/138485297-489b9b28-feb2-437a-aeb3-9b751401f117.png)

Now SSH with private key.

![image](https://user-images.githubusercontent.com/69868171/138485394-dd469823-b1f3-480d-b800-24809949d917.png)

we are root but the root flag is missing.

![image](https://user-images.githubusercontent.com/69868171/138485636-0e0a38d1-b507-4af2-9d34-5db680fa99a8.png)

![image](https://user-images.githubusercontent.com/69868171/138485723-08c59216-dd34-4a14-b925-4caa3c97e122.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

