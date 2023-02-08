---
layout: default
title : muzec - Jack Of All Trades Writeup
---

![Image](https://miro.medium.com/max/700/1*IdVpvTuLGREGcMZeX-1CAA.png)

```Jack-Of-All-Trades Rated medium on TryHackMe Boot-to-root originally designed for Securi-Tay 2020.```

# Enumeration! Enumeration! Time!

![Image](https://miro.medium.com/max/700/1*gTtUqXCwSk2NUNt0BzT-hg.png)


Our Nmap result was kind of strange HTTP running on port 22 and SSH running on port 80 opposite the result am expecting not wasting to much of time i try to access the HTTP port.

![Image](https://miro.medium.com/max/700/1*rW-s4C-tq_CB8iwCb6Yp9g.png)


Strange right?? Firefox is blocking us also hmm tricky so i try messing with my browser HTTP proxy setting.


![Image](https://miro.medium.com/max/700/1*0S3YSSFeUA-09LcDaL94aA.png)

And boom we are in…

![Image](https://miro.medium.com/max/700/1*9fsZkRDJUIra9S4dQgHDWA.png)

Next Stop > View Page Source

![Image](https://miro.medium.com/max/700/1*oeVxJmdvtVjbZywgos28oA.png)

![Image](https://miro.medium.com/max/700/1*x0Ua0QHYqJDlm_b4BhgElA.png)

Next Stop > /recovery.php

![Image](https://miro.medium.com/max/700/1*WqFOPtVeclIWgdm6mtrq-Q.png)

View Page Source Again.

![Image](https://miro.medium.com/max/700/1*f2U28VKpexmLmmIeF3JUtg.png)

Base32 to Hex

![Image](https://miro.medium.com/max/700/1*76nWId2C3-1JuMBhIU91MA.png)

Hex to Rot13

![image](https://miro.medium.com/max/700/1*4h1DUip63EEfiHzqJfQCog.png)

Rot13 to Plaintext

![Image](https://miro.medium.com/max/700/1*_2DH-qOxm2ftKV1PWAaHtg.png)


We are left with a link hint pointing us to a Wikipedia page .

![Image](https://miro.medium.com/max/700/1*0BU3O8Q6U6ntKc56qyfkHA.png)


![Image](https://miro.medium.com/max/700/1*31xgi_M_nRv7kzg0veA2sQ.png)


```Using Steghide with the credentials that you got from the base64```

![Image](https://miro.medium.com/max/700/1*Pt7QRpzokBL3X2U4IESt4A.png)


![Image](https://miro.medium.com/max/700/1*NuEkR-nX_yWRyc5PpV31Ew.png)


![Image](https://miro.medium.com/max/700/1*apPYxtFddkUKtfATFhkVdA.png)

Time to spawn a reverse shell to our terminal.

![Image](https://miro.medium.com/max/700/1*WJ-M96hmMYSZcTmasB8qog.png)

```Start an ncat connection```

![Image](https://miro.medium.com/max/700/1*JUoOc11pmocl2Wjlw53YLg.png)

```incoming connection receive boom.```

Try to make the shell stable by spawning a TTY shell ```python -c ‘import pty; pty.spawn(“/bin/bash”)’```

![image](https://miro.medium.com/max/700/1*xAwVIlohAGdNGlgbFp-4LQ.png)

```After spawning TTY Shell```

![image](https://miro.medium.com/max/700/1*ohV1WrlpZMnp9AOJTghVSA.png)

Changing directory to home we found a password list let’s do some brute-forcing and see if the pass list we found is still containing password used by user Jack.

![Image](https://miro.medium.com/max/700/1*1JSkYJ-NEfpv1b7bQZXIyg.png)


![Image](https://miro.medium.com/max/700/1*fpbNn0-oWWDIWIKj4HqcIg.png)


![Image](https://miro.medium.com/max/700/1*76YzrHB3Y7nCaKs3XgRpCw.png)


Privilege Escalation To Get Root Flag.

![Image](https://miro.medium.com/max/700/1*_WJcF4vXMYOw8Qk_bsfFlA.png)

Trying ```sudo -l``` the respond i get was Sorry, user jack may not run sudo on jack-of-all-trades time to launch ```LinEnum.```

![Image](https://miro.medium.com/max/700/1*OUn6U7gG6CUuNDjOo_8PqA.png)

Checking the SUID files i found something interesting ```/usr/bin/strings``` cool i quickly check up GTFOBINS for the exploit.

![Image](https://miro.medium.com/max/700/1*ALHzL5OK6qy4_y5QjnzKTQ.png)


![Image](https://miro.medium.com/max/700/1*mpIRfsf11QYzlYykGDJEfQ.png)

```Boom we have root.txt```

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
