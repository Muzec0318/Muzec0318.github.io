---
layout: default
title : muzec -  Juno  Writeup
---

PwnTillDawn Online Battlefield is a penetration testing lab created by wizlynx group where participants can test their offensive security skills in a safe and legal environment, but also having fun! The goal is simple, break into as many machines as possible using a succession of weaknesses and vulnerabilities and collect flags to prove the successful exploitation. Each target machine that can be compromised contains at least one “FLAG” (most of the times a file and typically located in the user’s Desktop, or the user’s root directory), which you must retrieve, and submit in the application. The flag is in the majority of the cases in a SHA1 format but not always.

![image](https://user-images.githubusercontent.com/69868171/123945514-3b542e00-d96c-11eb-9599-15f818c1cbee.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PTD/10.150.150.224]
└─$ cat nmap.nmap
# Nmap 7.91 scan initiated Thu Jun 10 07:34:33 2021 as: nmap -sC -sV -oA nmap 10.150.150.224
Nmap scan report for 10.150.150.224
Host is up (0.16s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 10 07:35:12 2021 -- 1 IP address (1 host up) scanned in 39.56 seconds
```

We are having only one port which is HTTP 80, nice seems we are bursting directory with gobuster.

`gobuster dir -u http://10.150.150.224 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html` checking the directory below.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PTD/10.150.150.224]
└─$ gobuster dir -u http://10.150.150.224 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.150.150.224
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2021/06/12 02:38:01 Starting gobuster in directory enumeration de
===============================================================
/index.html           (Status: 200) [Size: 10701]
/login.php            (Status: 200) [Size: 1213] 
```

Now let try and access ` http://10.150.150.224/login.php ` to check what login page we have maybe it need brute forcing just guessing lol.

![image](https://user-images.githubusercontent.com/69868171/123946818-a2bead80-d96d-11eb-92ce-54455b5e61a2.png)

We have the login page requesting for a Pin also and a link to download an APP so i decided to download the APP let try and check what the APP is all about.


### Decompile The APK

Mobile Security Framework (MobSF) is an automated, all-in-one mobile application (Android/iOS/Windows) pen-testing, malware analysis and security assessment framework capable of performing static and dynamic analysis. MobSF support mobile app binaries (APK, XAPK, IPA & APPX) along with zipped source code and provides REST APIs for seamless integration with your CI/CD or DevSecOps pipeline.The Dynamic Analyzer helps you to perform runtime security assessment and interactive instrumented testing.

**How To install** .

```
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
./setup.sh
```

This command will install all the required dependencies to run Mobile Security Framework. Now to run it After the installation complete we can use this tool by using run.sh command. As we previously told that this is a web based tool so we need to run it on our localhost server. To run it on our localhost with port 9999 (we can use any other port) by using following command:

```
./run.sh 127.0.0.1:9999
```

![image](https://user-images.githubusercontent.com/69868171/123949862-11e9d100-d971-11eb-8694-660b382bac36.png)

Now let access it on our browser `http://127.0.0.1:9999/` .

![image](https://user-images.githubusercontent.com/69868171/123950083-51b0b880-d971-11eb-904b-dc358a0fdcf7.png)

Now let upload the APK file to analyze it.

![image](https://user-images.githubusercontent.com/69868171/123950388-9e948f00-d971-11eb-8b6d-12a2849d1e97.png)

Going through it and i was able to get FLAG43.

![image](https://user-images.githubusercontent.com/69868171/123950639-e74c4800-d971-11eb-8ef4-bc8b3c0bf4c4.png)

Time to get the PIN i actually spend some time here lot of trial and error before getting it but i did lol screenshot of PIN below.

![image](https://user-images.githubusercontent.com/69868171/123950911-37c3a580-d972-11eb-95ee-21efc7248a51.png)

Now let try using the PIN on the LOGIN page.

![image](https://user-images.githubusercontent.com/69868171/123951095-66da1700-d972-11eb-9d50-e1f7381731b6.png)

Boom we are in and having all the FLAGS but one is encoded now let try decoding it to finish the machine.

![image](https://user-images.githubusercontent.com/69868171/123951891-46f72300-d973-11eb-8c86-aeffc4ab56e8.png)

Took sometime but i was able to decode it and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
