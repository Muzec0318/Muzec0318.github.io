---
layout: default
title : muzec - May Writeup
---

### Nmap Scanning.........

```Nmap -sC -sV -p- -oA nmap <Target-IP>```


```
# Nmap 7.91 scan initiated Mon Oct  4 07:08:39 2021 as: nmap -sC -sV -p- -oA nmap 172.16.109.139
Nmap scan report for 172.16.109.139
Host is up (0.00021s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 94:fb:c0:76:f2:b3:ff:4a:ed:61:6a:ae:a1:ca:86:c1 (RSA)
|   256 d0:29:99:fd:69:68:21:e3:b4:a6:48:e4:4e:a1:7e:f4 (ECDSA)
|_  256 2a:1b:1f:3d:ab:0a:00:5b:43:75:89:67:8a:98:21:df (ED25519)
80/tcp    open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://may.hmv
10000/tcp open  http    MiniServ 1.979 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct  4 07:09:18 2021 -- 1 IP address (1 host up) scanned in 39.71 seconds
```

We are having 3 open ports which seems interesting 22,80 and 10000 let start our enumeration on port 80 which is the HTTP.

![image](https://user-images.githubusercontent.com/69868171/137708864-6ab6c9ae-e641-41a7-badf-09041fd421f9.png)

Checking the page source we have nothing let hit some directory brute forcing.

![image](https://user-images.githubusercontent.com/69868171/137709418-cf65738f-5060-4458-82ce-f58c8d0df37f.png)

Nothing also let brute force for sub-domain.

![image](https://user-images.githubusercontent.com/69868171/137709625-70f66b72-2e34-4dd9-b9de-ec82c9e42fcc.png)

Adding to `/etc/hosts` file.


![image](https://user-images.githubusercontent.com/69868171/137709828-f45f363d-0d1e-4b45-a92f-8cb41ceb5c11.png)

A login page seems interesting i try some sql injection no luck but seems we have some valid users i guess.

```
admin
marie
alice
```

Let try brute forcing the login page with burp suite with the valid users intercept request and send to intruder.

![image](https://user-images.githubusercontent.com/69868171/137712798-0d58e21f-7c99-4683-9734-a8fe423fcb08.png)

Payload Positions and attack type already selected prove of screenshot below:/

![image](https://user-images.githubusercontent.com/69868171/137713097-25792fed-e90d-4518-ba23-d00ca3fb0277.png)

Payload tab:/

![image](https://user-images.githubusercontent.com/69868171/137713733-3baea415-e524-43e5-96e8-5a60425591c0.png)

I decided to split my rockyou.txt password wordlist to make it easy.

```
head -n 50000 file.lst > rockyou50.txt

or

split -l 50000 file.lst rockyou50.txt
```

![image](https://user-images.githubusercontent.com/69868171/137714481-98e4e3f0-a10b-45aa-ac60-aa923238a023.png)

Passwordlist loaded now let hit on attack.

![image](https://user-images.githubusercontent.com/69868171/137714862-31cb62c3-c653-4e1b-b54c-a1ca3b81e52b.png)

Boom we have credentials for marie let log in.

![image](https://user-images.githubusercontent.com/69868171/137715431-63ef2bdd-1585-4a27-a312-e527339420ab.png)

But man seems like a dead i check the source found nothing now time to check `ssh.may.hmv` .

![image](https://user-images.githubusercontent.com/69868171/137716093-6d7ee6af-e953-4250-a579-734c903bdbb9.png)

A login page also trying the marie credentials but got no luck but something seems strange with the portal cookie when not give it a try on the ssh web page also.

![image](https://user-images.githubusercontent.com/69868171/137716901-ff99c0de-b8a8-4c88-bc7f-8efd8567b993.png)

Edit with cookie editor and we have our private key time to use SSH now.

![image](https://user-images.githubusercontent.com/69868171/137717722-b97347c9-a487-4f27-8ceb-6ecaba808fb9.png)

We have user.txt time to get root.

![image](https://user-images.githubusercontent.com/69868171/137718640-fb7916dd-17fc-423d-8d90-0e2414fbb645.png)

We have write permission on the conf file now let try to abuse it to get root shell.

![image](https://user-images.githubusercontent.com/69868171/137719040-b31361b3-1cd6-4138-bbaa-756421e078aa.png)

Generating Perl payload with `msfvenom` .

![image](https://user-images.githubusercontent.com/69868171/137720001-66b36274-59fe-4a90-83f8-a3c7dd7444a6.png)

Transfer to the target.

![image](https://user-images.githubusercontent.com/69868171/137720209-cb59b557-9f71-4535-9af4-d8155712b11a.png)

Our listener is ready also now let access the webmin.

![image](https://user-images.githubusercontent.com/69868171/137720326-da144c72-ed9e-4653-8396-b09dbf9c0fa2.png)

Putting a wrong credentials will give us a root shell.

![image](https://user-images.githubusercontent.com/69868171/137720706-7f084d61-6f02-460e-a122-b2eaed55f432.png)

Back to our listener.

![image](https://user-images.githubusercontent.com/69868171/137720836-6bd4a0b6-98e4-47cd-aa7b-58038820e2f6.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
