---
layout: default
title : muzec - FullMounty Writeup
---


PwnTillDawn Online Battlefield is a penetration testing lab created by wizlynx group where participants can test their offensive security skills in a safe and legal environment, but also having fun! The goal is simple, break into as many machines as possible using a succession of weaknesses and vulnerabilities and collect flags to prove the successful exploitation. Each target machine that can be compromised contains at least one “FLAG” (most of the times a file and typically located in the user’s Desktop, or the user’s root directory), which you must retrieve, and submit in the application. The flag is in the majority of the cases in a SHA1 format but not always. 

![image](https://user-images.githubusercontent.com/69868171/121382660-d2e9d200-c914-11eb-87bd-24bfdb76792a.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PTD/10.150.150.134]
└─$ cat nmap.nmap 
# Nmap 7.91 scan initiated Wed Jun  9 07:30:18 2021 as: nmap -sC -sV -oA nmap 10.150.150.134
Nmap scan report for 10.150.150.134
Host is up (0.16s latency).
Not shown: 995 closed ports
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 5.3p1 Debian 3ubuntu7.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 f6:e9:3f:cf:88:ec:7c:35:63:91:34:aa:14:55:49:cc (DSA)
|_  2048 20:1d:e9:90:6f:4b:82:a3:71:1e:a9:99:95:7f:31:ea (RSA)
111/tcp  open     rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/udp   nfs
|   100005  1,2,3      34154/tcp   mountd
|   100005  1,2,3      50354/udp   mountd
|   100021  1,3,4      45783/tcp   nlockmgr
|   100021  1,3,4      48262/udp   nlockmgr
|   100024  1          38840/udp   status
|_  100024  1          40110/tcp   status
2049/tcp open     nfs         2-4 (RPC #100003)
4003/tcp filtered pxc-splr-ft
8089/tcp open     ssl/http    Splunkd httpd
|_http-server-header: Splunkd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2019-10-28T09:51:59
|_Not valid after:  2022-10-27T09:51:59
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun  9 07:31:23 2021 -- 1 IP address (1 host up) scanned in 65.62 seconds
```

We have four open ports which is cool but seems port 80 is misiing which is a dead end also lol but guess we still have a cool port NFS which run on port 2049.

Network File System, or NFS, allows remote hosts to mount the systems/directories over a network. An NFS server can export a directory that can be mounted on a remote Linux machine. This allows the user to share the data centrally to all the machines in the network.

### EXploiting NFS Share

`showmount -e 10.150.150.134`

![image](https://user-images.githubusercontent.com/69868171/121383653-9bc7f080-c915-11eb-9fc6-9f86e778e98a.png)

Boom we have a share let create a directory and mount it.

```
mkdir /tmp/share
mount -v -t  nfs  -o vers=3,proto=tcp,nolock 10.150.150.134:/srv/exportnfs /tmp/share
```

![image](https://user-images.githubusercontent.com/69868171/121384153-037e3b80-c916-11eb-86fc-ff283a289c24.png)

Let confirm it by checking the share with the directory we just mount together.

![image](https://user-images.githubusercontent.com/69868171/121384374-33c5da00-c916-11eb-94fe-a84ba1c62a57.png)

We are in the share we have FLAG49 also a private key to SSH into the machine and a public key when we can get the name for the SSH.

![image](https://user-images.githubusercontent.com/69868171/121384554-5fe15b00-c916-11eb-8d9d-53f231dad73c.png)

We can see the username already time to log in make sure you give the private key permission.

![image](https://user-images.githubusercontent.com/69868171/121384769-8f906300-c916-11eb-9456-ec5da07b51f7.png)

Boom we are in time to get root.

### Privilege Escalation

Checking for Kernal version.

![image](https://user-images.githubusercontent.com/69868171/121385203-e6963800-c916-11eb-92ae-8f49e3bf0777.png)

Running some old version cool let hit google forsome exploits.

![image](https://user-images.githubusercontent.com/69868171/121385444-1e9d7b00-c917-11eb-8fe5-58bb684a3d46.png)

Nie but i use the dirtycow exploit.

![image](https://user-images.githubusercontent.com/69868171/121385643-4b519280-c917-11eb-9857-2d2d2dce9f7d.png)

Checking the box we have no `gcc` compiler on it so i decided to compile it on another machine running the same kernal version now let transfer it to the target and run.

![image](https://user-images.githubusercontent.com/69868171/121387043-5bb63d00-c918-11eb-9079-26ac25bd8172.png)

We are root and done.

PWNTILLDAWN LABS IS THE BEST WHY NOT TRY IT: [FullMounty On PwnTillDawn Click Here](https://online.pwntilldawn.com/Target/Show/64)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
