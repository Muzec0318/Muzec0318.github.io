---
layout: default
title : muzec - Soccer HTB Writeup
---


![image](https://user-images.githubusercontent.com/69868171/218117279-f27eebec-fbed-4900-a1e9-7cdd0c83c12c.png)


#### RECON

Enumeration is the key to everthing so we always kick off with an nmap scan on the target IP.

```
└─$ nmap -p- --min-rate 2000 -sC -sV -oN nmap/soccer.tcp 10.10.11.194
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-10 15:38 WAT
Warning: 10.10.11.194 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.194
Host is up (0.27s latency).
Not shown: 38489 filtered tcp ports (no-response), 27044 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
|_  256 5797565def793c2fcbdb35fff17c615c (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 286.53 seconds
```

Now it connfirm we have SSH 22 and HTTP 80 port presently open the target which is interesting since it rated easy i think we already know we don't need to dig that much we can start a quick enumeration on the HTTP port already.

```
└─$ curl -i http://10.10.11.194
HTTP/1.1 301 Moved Permanently
Server: nginx/1.18.0 (Ubuntu)
Date: Fri, 10 Feb 2023 14:48:02 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://soccer.htb/

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
```

Some old habit i always use to print out the response headers in the output using `curl` command now that we have the domain let add to our hosts file.

```
sudo vim /etc/hosts

.......
10.10.11.194        soccer.htb
.......
```

Now that we have that set let access the web-server surely we are using the browser XD.

![image](https://user-images.githubusercontent.com/69868171/218122187-0bc713c6-ee50-4e23-bcf2-0cc4a0a3294a.png)


Interesting a web page that talk on how soccer is the best sports in the world which is true no cap but i still take hacking like sports XD which mean sports is second to me.

#### Subdomain Fuzz

```
┌──(muzec㉿muzec-sec)-[~/Documents/CTF Hacking/HTB/soccer]
└─$ wfuzz -u http://soccer.htb/ -H "Host: FUZZ.soccer.htb" -w /opt/wordlists/subdomains.txt --hh 178
```

![image](https://user-images.githubusercontent.com/69868171/218126218-472ce0bd-c87a-472d-b8e2-15bfd2b8e7b7.png)


Which i end up getting none so let go ahead amd fuzz for some directories.



#### Directories Fuzz
```
┌──(muzec㉿muzec-sec)-[~/Documents/CTF Hacking/HTB/soccer]
└─$ feroxbuster -u http://soccer.htb/ -w /opt/wordlists/2.3-medium.txt --quiet -t 80
```

![image](https://user-images.githubusercontent.com/69868171/218125962-afbc5da8-f77a-4648-882a-bfecb702a677.png)

Now that is a hit let see what we really have on the tiny web directory.

![image](https://user-images.githubusercontent.com/69868171/218126759-f4bee2e0-bcea-4240-874a-458003ae062d.png)


Tiny File Manager and we seems to need a credentials to get access let try doing some research on a default credentials.

![image](https://user-images.githubusercontent.com/69868171/218127164-91c79c4e-0ca3-48a5-8c79-554495d607d4.png)

Default username/password: 

```
admin/admin@123
user/12345
```

![image](https://user-images.githubusercontent.com/69868171/218127761-74141392-38ee-4ae4-9171-387a6d53e297.png)


We got access to the Tiny File Manager and seems to find the version it running a quick research again.

![image](https://user-images.githubusercontent.com/69868171/218128210-f73afd5b-84a5-47a6-92ae-248d922afa64.png)


Which come to our notice that tiny file manager is vulnerable to an authenticated remote code execution allowing a malicious user to upload a php file to be able to execute a system command on the webserver now that we know what we are dealing with let get our malicious php file ready and upload on the webserver to get a reverse shell.

![image](https://user-images.githubusercontent.com/69868171/218128951-d015947c-16f6-49ab-99a3-f83c0033efa3.png)


Created a PHP file added our tun0 IP, Port to receive a reverse shell back to us.

![image](https://user-images.githubusercontent.com/69868171/218130364-444fbf10-8b2f-416b-9bf6-a3ef05678ba4.png)


```
nc -nvlp 80
```


![image](https://user-images.githubusercontent.com/69868171/218130939-3af7db85-7fba-460f-ba25-017298cf7a32.png)


Navigating to http://soccer.htb/tiny/uploads/muzec.php to trigger our php file and we got a reverse shell back to our listener.

![image](https://user-images.githubusercontent.com/69868171/218131686-0a031b5b-c304-4be4-afb0-0c9f260112b7.png)

We spawn a full tty shell to make our shell more stable and we go ahead and start enumerating once again to find a way to move our privilege to a higher user by checking all files sysetem on the target.


