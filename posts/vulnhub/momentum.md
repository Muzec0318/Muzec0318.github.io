---
layout: default
title : muzec - Momentum Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119359326-8006ee00-bc77-11eb-9808-5beb6f98c3ed.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Momentum]
└─$ nmap -sC -sV -oA nmap 172.16.139.165                                                                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-24 07:05 EDT
Nmap scan report for 172.16.139.165
Host is up (0.00017s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5c:8e:2c:cc:c1:b0:3e:7c:0e:22:34:d8:60:31:4e:62 (RSA)
|_  256 c1:8f:87:c1:52:09:27:60:5f:2e:2d:e0:08:03:72:c8 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Momentum | Index 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.08 seconds
```

Not wasting to much of time hitting port 80 to burst directory but we got nothing.

![image](https://user-images.githubusercontent.com/69868171/119359881-13402380-bc78-11eb-8620-14e8c915ab68.png)

So let try checking it manually to see what we are missing checking the page source for some clue or hidden hint.

![image](https://user-images.githubusercontent.com/69868171/119360220-63b78100-bc78-11eb-97b6-9325c1574914.png)

So we have `http://172.16.139.165/js/main.js` let go through it to see what we have.

```
function viewDetails(str) {

  window.location.href = "opus-details.php?id="+str;
}

/*
var CryptoJS = require("crypto-js");
var decrypted = CryptoJS.AES.decrypt(encrypted, "SecretPassphraseMomentum");
console.log(decrypted.toString(CryptoJS.enc.Utf8));
*/
```
Interesting the window.location seems to be pointing to another directory let confirm it `http://172.16.139.165/opus-details.php?id=` and yes it a directory checking for LFI/RFI vulnerability but no luck i keep gettinf the same input back.

![image](https://user-images.githubusercontent.com/69868171/119361036-37e8cb00-bc79-11eb-8198-64182a0729d7.png)

Spend time here so we have a cookie how can i miss that when i delete the cookie and refreshing the page we get the same cookie back cool not changing probably it our way in let try decrypting it.

AES Decrypt  and yes i think we have the secret key already `AES.decrypt(encrypted, "SecretPassphraseMomentum")` we be using online tools to decrypt it `https://www.browserling.com/tools/aes-decrypt` .

![image](https://user-images.githubusercontent.com/69868171/119362520-c90c7180-bc7a-11eb-8fab-ee654d4a0d83.png)

Decryting and we have the password.

![image](https://user-images.githubusercontent.com/69868171/119362596-dfb2c880-bc7a-11eb-81d0-283c5cec7e34.png)

Now we have a password but no username i know the author name of the box is `alienum` and seems the password look like a 2 usernames let try to confirm it so i save both username in a file now let using hydra to brute force it.

![image](https://user-images.githubusercontent.com/69868171/119363198-89925500-bc7b-11eb-9164-9e78bc92ca33.png)

We have the right credentials let log in SSH now .

![image](https://user-images.githubusercontent.com/69868171/119363480-d7a75880-bc7b-11eb-8707-a46d5b35ef08.png)

And we are in also we have the user.txt moving to root now.

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/119363991-72079c00-bc7c-11eb-9003-bfca210e7a78.png)

Checking `sudo -l` ahhh not installed on the target so i try checking for ruuning port and cool we have the redis port `6379` we should be able to log in without details.

![image](https://user-images.githubusercontent.com/69868171/119364266-b98e2800-bc7c-11eb-86f1-fae027f17629.png)

We are in and we have just one database active let try reading what we have in the database maybe way to root .

![image](https://user-images.githubusercontent.com/69868171/119364576-08d45880-bc7d-11eb-8277-0a88628124e0.png)

Boom password to login as root.

![image](https://user-images.githubusercontent.com/69868171/119364735-2e616200-bc7d-11eb-8174-4a456fed4975.png)

We are root box rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
