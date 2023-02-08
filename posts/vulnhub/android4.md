---
layout: default
title : muzec - Android4 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119974277-7389da80-bf82-11eb-91b8-7cd384f6bde7.png)

Boom am so excited to try my first android VM was dreaming all about it through the night wow i love hacking the excitement is 1337 yes it l33t let hit it.


We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/Android4]
└─$ cat nmap.nmap                                                                    
# Nmap 7.91 scan initiated Fri May 28 02:10:25 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.188
Nmap scan report for 172.16.139.188
Host is up (0.00098s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE  VERSION
5555/tcp  open  freeciv?
8080/tcp  open  http     PHP cli server 5.5 or later
|_http-title: Deface by Good Hackers
22000/tcp open  ssh      Dropbear sshd 2014.66 (protocol 2.0)
| ssh-hostkey: 
|   1024 b3:98:65:98:fd:c0:64:fe:16:d6:30:36:aa:2b:ef:6b (DSA)
|   2048 19:e2:9e:6c:c6:8d:af:4e:86:7c:3b:60:91:33:e1:85 (RSA)
|_  521 46:13:43:49:24:88:06:85:6c:75:93:73:b5:1d:8f:28 (ECDSA)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 28 02:12:13 2021 -- 1 IP address (1 host up) scanned in 108.72 seconds
```

We having some few port and some strange one since it my first time dealing with a Android VM but with the power of research i think anything is possible so let get back to it checking port `8080` .

![image](https://user-images.githubusercontent.com/69868171/119975483-e9db0c80-bf83-11eb-9ef8-005d263351a3.png)

Something interesting words here; 

```
If you r Smart Dan find Backdoor access...and safe your machine

we like POST things only.
```

So i try using Curl to send a POST request  with the url.

![image](https://user-images.githubusercontent.com/69868171/119976181-b8af0c00-bf84-11eb-8a69-ff28668b2986.png)

But got nothing special now time to inspect the ports we have doing research on the first port `5555` so i stumble on (ADB) Android Debug Bridge.

### What Is ADB?

Android Debug Bridge (adb) is a versatile command-line tool that lets you communicate with a device. The adb command facilitates a variety of device actions, such as installing and debugging apps, and it provides access to a Unix shell that you can use to run a variety of commands on a device

Usually, developers connect to ADB service installed on Android devices using a USB cable, but it is also possible to use ADB wireless by enabling a daemon server at TCP port 5555 on the device.

Since we know that know i install ADB on my machine now let try to connect to it.

`adb connect 172.16.139.188:5555` //NOTE:- probably the IP we be different at you end.

![image](https://user-images.githubusercontent.com/69868171/119977322-47705880-bf86-11eb-9803-d18e8851ad4c.png)

Now let drop into shell with `adb shell` .

![image](https://user-images.githubusercontent.com/69868171/119977427-6a027180-bf86-11eb-92fa-86404fdf7876.png)

Having shell nice very interesting so first thing i do is to type `su` and see what happened and guess what i was root direct.

![image](https://user-images.githubusercontent.com/69868171/119977851-e5642300-bf86-11eb-8a2c-1c061284728a.png)

Sweet right?? now let check the `data/root` folder and we have our flag.

![image](https://user-images.githubusercontent.com/69868171/119978024-1e9c9300-bf87-11eb-9b13-025820086440.png)

But not done yet since we have root but the android phone screen is still locked nah am not happy with that.

![image](https://user-images.githubusercontent.com/69868171/119978184-53a8e580-bf87-11eb-8bb6-57cd083cbd09.png)

Probably we should have a way to remove the lock screen so let dig more into the phone.

![image](https://user-images.githubusercontent.com/69868171/119978305-7804c200-bf87-11eb-9680-73ef9a0b9a1a.png)

Going into the `data/system` folder we found a key file probably holding the password.

![image](https://user-images.githubusercontent.com/69868171/119978571-cb771000-bf87-11eb-8511-4ae2bf87db05.png)

What about if we remove the `password.key` can we get access to the phone?? let try that .

![image](https://user-images.githubusercontent.com/69868171/119978773-08430700-bf88-11eb-9fc2-da43dc1833d6.png)

Now let check the Phone do we have access yet .

![image](https://user-images.githubusercontent.com/69868171/119978893-31fc2e00-bf88-11eb-9123-82c9b08ca70e.png)

Boom we are in ahhhhhhhhhh.

![image](https://user-images.githubusercontent.com/69868171/119978964-48a28500-bf88-11eb-9124-7ee6816b3d72.png)

Guess we are done now.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
