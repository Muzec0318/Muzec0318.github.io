---
layout: default
title : muzec - Watcher Writeup
---

![Image](https://imgur.com/08smS1S.png)

We always start with an nmap scan.....


```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu Apr 15 06:03:39 2021 as: nmap -sC -sV -oA nmap 10.10.90.237
Nmap scan report for 10.10.90.237
Host is up (0.21s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:80:ec:1f:26:9e:32:eb:27:3f:26:ac:d2:37:ba:96 (RSA)
|   256 36:ff:70:11:05:8e:d4:50:7a:29:91:58:75:ac:2e:76 (ECDSA)
|_  256 48:d2:3e:45:da:0c:f0:f6:65:4e:f9:78:97:37:aa:8a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Jekyll v4.1.1
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Corkplacemats
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 15 06:05:02 2021 -- 1 IP address (1 host up) scanned in 83.77 seconds
```

3 open ports FTP,SSH and HTTP checking FTP first.

![Image](https://imgur.com/ImSUWk4.png)

Yea no anonymous logins allow now let hit port HTTP which is 80.

![Image](https://imgur.com/mUYTadA.png)

Always check robots.txt.

![Image](https://imgur.com/3VzPEgq.png)

Cool we got the 1st flag and some secret txt file again.

![Image](https://imgur.com/PblVGi2.png)

Yes i know i hide the flag let move on with the secret txt file but when i try to check the ` /secret_file_do_not_read.txt` 403 forbidden ahhhh why ok let go back to the homepage.

![image](https://imgur.com/anTQpT3.png)

Back to the homepage clicking on one of the post i notice the URL was looking strange probably is it vulnerable to LFI or RCE come on let give it a try.

![Image](https://imgur.com/u2oFx7W.png)

And boom we got some LFI vulnerability in the house.

![Image](https://imgur.com/5hOyJ2z.png)

Now let try reading the `/secret_file_do_not_read.txt` with the LFI.

![Image](https://imgur.com/51uImFe.png)

Cool some FTP details now let hit FTP back.

![Image](https://imgur.com/mKWJJfy.png)

Boom flag2 retrieve now let check the files folder but it was empty let try to put some reverse shell in it and give it permission `chmod 777 rev.php`.

![image](https://imgur.com/gvJCdNP.png)

`Hi Mat, The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files.`

So we know the folder our reverse shell is save in now let start an ncat listener `nc -nvlp 4444` .

![Image](https://imgur.com/IEzHytf.png)

Accessing the reverse shell path give us back a shell back to our terminal.

![Image](https://imgur.com/rgy1EQN.png)

![Image](https://imgur.com/FGQMjhk.png)

Spawing a TTY shell you can check out my cheat sheet here [Spawing TTY Shell](https://muzec0318.github.io/posts/Ttyshells.html) .

![image](https://imgur.com/RKrs8ra.png)

`python3 -c 'import pty; pty.spawn ("/bin/bash")'`

![image](https://imgur.com/gpi8YcU.png)

And we have flag3 also now let check the home directory but we have no access to any file now let try `sudo -l` .

![Image](https://imgur.com/PyFsw4g.png)

Cool our way to user toby now type `sudo -u toby /bin/sh` and we are toby.

![Image](https://imgur.com/DF9QNZZ.png)

Done with flag4.

![Image](https://imgur.com/QCKSWNF.png)



