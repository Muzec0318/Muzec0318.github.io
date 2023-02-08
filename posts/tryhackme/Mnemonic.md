---
layout: default
title : muzec - Mnemonic Writeup
---

![Image](https://imgur.com/NDPWlCS.png)


```You Guys miss Me Lol??? Sure Have Been Recieving Lots Of Mail Lately For Some Writeups So Let Hit Now.```


# NMAP Scan

Nmap -sV -p- IP -T5

Let Me Break It The Down The Nmap Comand I Use.

-sV = Probe open ports to determine service/version info

-p- = Scan For Full Ports

-T5 = Set timing template Making The Scanning Fast

```
# Nmap 7.80 scan initiated Wed Oct 21 05:45:24 2020 as: nmap -sC -sV -oA nmap 10.10.170.178
Nmap scan report for 10.10.170.178
Host is up (0.28s latency).
Not shown: 993 closed ports
PORT     STATE    SERVICE       VERSION
21/tcp   open     ftp           vsftpd 3.0.3
1337/tcp open     ssh           OpenSSH 7.6p1
80/tcp   open     http          Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
1119/tcp filtered bnetgame
5001/tcp filtered commplex-link
6580/tcp filtered parsec-master
7019/tcp filtered doceri-ctl
9618/tcp filtered condor
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Oct 21 05:46:49 2020 -- 1 IP address (1 host up) scanned in 85.32 seconds
```

1.  How many open ports? 

```REDACTED  (Read The Scan Result)```

2. what is the ssh port number?

```REDACTED (Read The Scan Result)```

3. what is the name of the secret file?

Time To Use Gobuster To Burst Some Directorys Port 80 HTTP

gobuster dir -u http://IP -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

dir = Dircectory

-u = Target URL

-w = Wordlists

![Image](https://imgur.com/kNCwg9D.png)

A dir let try to access it


![Image](https://imgur.com/hthuK4c.png)

just a plain white page hmmm let hit gobuster again with the dir we just found.

![Image](https://imgur.com/U3kKSLl.png)

The admin page was not useful also blank plain white let continue to burst some dirs.

![Image](https://imgur.com/YLXeI5D.png)

/backups pretty interesting let hit it with gobuster with gobuster extension command -x with .zip,tar.txt,php maybe it can be our lucky day.

![Image](https://imgur.com/8ut1oLe.png)

cool a backups zip file let download it.

![Image](https://imgur.com/TfO4W72.png)

file protected with password let try to crack it.

![Image](https://imgur.com/hy4u0pE.png)

sweet zip file cracked let extract the information we need in it.

![Image](https://imgur.com/DYW4x1N.png)

1.  ftp user name? 
```REDACTED (What do you see in the note.txt)```

2. ftp password?
```REDACTED (Use Hydra to bruteforce)```

3. What is the ssh username?
```REDACTED (Log in FTP With usernmae and password you just found)```

4. What is the ssh password?
```REDACTED (crack the id_rsa private key you found in FTP)```

5. What is the condor password?
```REDACTED (Log in SSH with the creds You just Obtained)```

i will explain how to get the Condor password because it not that easy a little OSINT the hint (mnemonic encryption "image based")

![Image](https://imgur.com/GLqtF9L.png)

Boom we have the tools it on github Now just go back to the SSH you just log in to find the image Condor password is hidden in and decrypt it.


![Image](https://imgur.com/b1YUHoK.png)

cool we are in let get the User.txt and Privilege Escalation to Get Root.txt.

A little hint to the user.txt flag it encoder in base......Now let get the Root.txt flag.

![Image](https://imgur.com/OKwoK6g.png)

So i cat the python file the line i only find interesting is dis.

```
if select == 0: 
                        time.sleep(1)
                        ex = str(input("are you sure you want to quit ? yes : "))

                        if ex == ".":
                                print(os.system(input("\nRunning....")))
                        if ex == "yes " or "y":
                                sys.exit()
```

So let run ```sudo /usr/bin/python3 /bin/examplecode.py```

![Image](https://imgur.com/A3GriVJ.png)

![image](https://imgur.com/asu0UlW.png)

Boom Box Rooted..............

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
