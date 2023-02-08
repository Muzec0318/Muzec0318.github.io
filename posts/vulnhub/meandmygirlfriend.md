---
layout: default
title : muzec - Me-and-My-Girlfriend-1 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/116999467-a7d4e880-acad-11eb-98de-8f89f62e34c9.png)

Box Description: This VM tells us that there are a couple of lovers namely Alice and Bob, where the couple was originally very romantic, but since Alice worked at a private company, "Ceban Corp", something has changed from Alice's attitude towards Bob like something is "hidden", And Bob asks for your help to get what Alice is hiding and get full access to the company!

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Tue May  4 04:15:28 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.167
Nmap scan report for 172.16.139.167
Host is up (0.00014s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 57:e1:56:58:46:04:33:56:3d:c3:4b:a7:93:ee:23:16 (DSA)
|   2048 3b:26:4d:e4:a0:3b:f8:75:d9:6e:15:55:82:8c:71:97 (RSA)
|   256 8f:48:97:9b:55:11:5b:f1:6c:1d:b3:4a:bc:36:bd:b0 (ECDSA)
|_  256 d0:c3:02:a1:c4:c2:a8:ac:3b:84:ae:8f:e5:79:66:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue May  4 04:15:41 2021 -- 1 IP address (1 host up) scanned in 13.64 seconds
```
So we got 2 open ports only let hit port 80 to see what is running.

![image](https://user-images.githubusercontent.com/69868171/116999467-a7d4e880-acad-11eb-98de-8f89f62e34c9.png)

`Who are you? Hacker? Sorry This Site Can Only Be Accessed local!`

But we get a message saying we can only access the website locally check through the page source we get a hint.

![image](https://user-images.githubusercontent.com/69868171/117000742-8bd24680-acaf-11eb-9499-0c9f7e16ebe4.png)

Now let Intercept the request with Burp to edit the header.

![image](https://user-images.githubusercontent.com/69868171/117001111-f1bece00-acaf-11eb-8a98-d43ba7156942.png)

And boom we got access to the website bypassing it with `x-forwarded-for: 127.0.0.1` .

![image](https://user-images.githubusercontent.com/69868171/117001340-39455a00-acb0-11eb-874e-be1794eca8c4.png)

Since we have some login/register page let try to register first.

![image](https://user-images.githubusercontent.com/69868171/117001602-8fb29880-acb0-11eb-9b10-cc9b6b41361b.png)

Using the same method always to intercept the request to add the `x-forwarded-for: 127.0.0.1`  and register after that let log in.

![image](https://user-images.githubusercontent.com/69868171/117002068-18313900-acb1-11eb-95e8-3d3343fa4b88.png)

And we are in let look around but i got nothing also try some basic LFI payload but nothing but looking at the url found something interesting.

![image](https://user-images.githubusercontent.com/69868171/117002318-68100000-acb1-11eb-86d9-0830f0c70a99.png)

So i note down the user id and and register another user which the id is 16 so what i do is try to access the first user account from the new created user maybe it possible or vulnerable to account takeover.

![image](https://user-images.githubusercontent.com/69868171/117002653-d5bc2c00-acb1-11eb-9d2a-b94a55c34760.png)

And boom guess what i got access to another user account.

![image](https://user-images.githubusercontent.com/69868171/117002917-2cc20100-acb2-11eb-8f96-d4c4ce8c731a.png)

Since we are number 15 register user on the website i use the same mathod to access evervbody account on it and note it down also with the password to view the password is easy.

![image](https://user-images.githubusercontent.com/69868171/117003247-9a6e2d00-acb2-11eb-887c-1336c13112c0.png)

And changing the password form to username we show you the password in plaintext.

![image](https://user-images.githubusercontent.com/69868171/117003373-c9849e80-acb2-11eb-9f6c-d5911a8243a4.png)

![image](https://user-images.githubusercontent.com/69868171/117003517-f20c9880-acb2-11eb-924d-28f0090448de.png)

So i was able to get all the credentials by accessing everybody account.

![image](https://user-images.githubusercontent.com/69868171/117003652-1e281980-acb3-11eb-8902-d08e13250429.png)

Since we have SSH port open let try to use it to brute force it maybe it our way in so i save all the usernames in a seperate file also the password in seperate file.

![image](https://user-images.githubusercontent.com/69868171/117003935-795a0c00-acb3-11eb-956f-7a4c9d6f4ffc.png)

And boom we have the right credentials for SSH let hit it now.

![image](https://user-images.githubusercontent.com/69868171/117004135-ba522080-acb3-11eb-91f3-f6d375886b5f.png)

Access granted and we have the first flag.

### Privilege Escalation

Checking `Sudo -l` .

![image](https://user-images.githubusercontent.com/69868171/117004376-ff765280-acb3-11eb-823e-f810f3a03700.png)


User alice may run the following commands on gfriEND:
    (root) NOPASSWD: /usr/bin/php

Cool let check `gtfobins` .

![image](https://user-images.githubusercontent.com/69868171/117004526-26348900-acb4-11eb-92be-2925055eb9d7.png)

`sudo php -r "system('/bin/sh');"` 

![image](https://user-images.githubusercontent.com/69868171/117004693-5a0fae80-acb4-11eb-802c-3e4477a143c0.png)

We are root let get the second flag in the root directory.

![image](https://user-images.githubusercontent.com/69868171/117004867-94794b80-acb4-11eb-99be-42a484eb157a.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>






