---
layout: default
title : muzec - DC-2 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119254510-a9931d00-bb84-11eb-84ab-b87d72fcaba8.png)

Yet again today we be working on another OSCP like box DC2 On vulnhub you can grab a copy here [Download DC2](https://www.vulnhub.com/entry/dc-2,311/) .

Note:- You need to add `172.16.139.179` dc-2 to `/etc/hosts` probably your IP we be different from mine.

We always start with an nmap scan.....

```Nmap -sC -p- -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/DC2]
└─$ cat nmap.nmap 
# Nmap 7.91 scan initiated Wed May 19 13:51:26 2021 as: nmap -p- -sC -sV -oA nmap 172.16.139.179
Nmap scan report for 172.16.139.179
Host is up (0.00020s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Did not follow redirect to http://dc-2/
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 52:51:7b:6e:70:a4:33:7a:d2:4b:e1:0b:5a:0f:9e:d7 (DSA)
|   2048 59:11:d8:af:38:51:8f:41:a7:44:b3:28:03:80:99:42 (RSA)
|   256 df:18:1d:74:26:ce:c1:4f:6f:2f:c1:26:54:31:51:91 (ECDSA)
|_  256 d9:38:5f:99:7c:0d:64:7e:1d:46:f6:e9:7c:c6:37:17 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed May 19 13:51:38 2021 -- 1 IP address (1 host up) scanned in 11.88 seconds
```

Port 80 and 7744 going through the port 80 first found our first flag with some extra hint to generate a wordlists to brute force the wordpress admin login page .

![image](https://user-images.githubusercontent.com/69868171/119254755-325e8880-bb86-11eb-9c23-cf9e56dafaa5.png)

### Running CeWL To Generate A Wordlist .

`cewl -m 7 -w wordlist.txt -d 5 -v http://dc-2/`

In this command, -m 7 specifies that we want our gathered words to be no less than 7 characters in length. -w followed by a text file name provides a place for our wordlist to be captured. If this isn’t specified, CeWL will only display monitor output and no file will be created.

-d 5 (default is 2) explicitly specifies that we want CeWL to crawl 5 links deep, meaning we only want to gather words from 5 pages. The -v flag followed by the URL above tells CeWL that we want it to crawl the Wikipedia Game of Thrones page in order to gather words.

When the command is executed, we’ll see activity on the console. Simply wait until it’s done and a wordlist.txt file will be created.

![image](https://user-images.githubusercontent.com/69868171/119255254-c2053680-bb88-11eb-833e-8023c5e61656.png)

Now we have the wordlist for passwords now let try to enumerate the blog to find some usernames using `wpscan` .

`wpscan --url http://dc-2/ --enumerate u ` and we found 3 users .

![image](https://user-images.githubusercontent.com/69868171/119255346-5374a880-bb89-11eb-9dd7-0386063b0926.png)

1. admin
2. jerry
3. tom

Now let save in a wordlist name user.txt now let brute force it using `wpscan` .

`wpscan --url http://dc-2/ --usernames user.txt --passwords wordlist.txt` and we have the right credentials .

![image](https://user-images.githubusercontent.com/69868171/119255637-de09d780-bb8a-11eb-89d8-6661e8a60d16.png)

Got access to the admin panel but it just a simple author credentials both the tom and jerry account .

![image](https://user-images.githubusercontent.com/69868171/119255747-5a041f80-bb8b-11eb-9ded-e9aebbb134c3.png)

So let try using it on SSH and boom we got in with the tom credential .

![image](https://user-images.githubusercontent.com/69868171/119255863-ef071880-bb8b-11eb-92f7-9fa0c17d228d.png)

But it a resticted rbash shell now let try to escape it using `vi` .

![image](https://user-images.githubusercontent.com/69868171/119255960-51f8af80-bb8c-11eb-817a-8e601663d17c.png)

Press enter and type `:shell` .

![image](https://user-images.githubusercontent.com/69868171/119255992-76548c00-bb8c-11eb-9953-d79770471f8d.png)

And boom escape but seems we can't still using some commands probably the `/bin/sh` path is not set.

![image](https://user-images.githubusercontent.com/69868171/119256036-a69c2a80-bb8c-11eb-9a05-21e50545b7ad.png)

Now let export it `export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` and we are good to go i guess lol .

![image](https://user-images.githubusercontent.com/69868171/119256206-90429e80-bb8d-11eb-9b5d-47b01068e708.png)

Now let try to change user `su jerry` with the password we found when brute forcing the wordpress login page.

![image](https://user-images.githubusercontent.com/69868171/119256298-08a95f80-bb8e-11eb-8c5e-a06137dd543c.png)

Boom we are in now let check `sudo -l` .

![image](https://user-images.githubusercontent.com/69868171/119256351-44442980-bb8e-11eb-9d5c-320c0b92e94b.png)

Cool we can run `/usr/bin/git` to get root now let hit `gtfobins` .

![image](https://user-images.githubusercontent.com/69868171/119256408-9f761c00-bb8e-11eb-8814-521b7852782f.png)

`sudo git -p help config` and let type `!/bin/sh` to get our root shell .

![image](https://user-images.githubusercontent.com/69868171/119256462-e06e3080-bb8e-11eb-8f0c-60b3859220dc.png)

And enter give us the root shell .

![image](https://user-images.githubusercontent.com/69868171/119256489-fbd93b80-bb8e-11eb-8f19-1a8317310e3f.png)

And we have the final flag also box rooted .

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
