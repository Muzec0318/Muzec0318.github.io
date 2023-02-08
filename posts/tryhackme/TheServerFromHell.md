---
layout: default
title : muzec - The Server From Hell Writeup
---

![Image](https://imgur.com/ns4BoqH.png)

Face a server that feels as if it was configured and deployed by Satan himself. Can you escalate to root?

Start at port 1337 and enumerate your way.
Good luck.

![Image](https://imgur.com/5HCQWyE.png)

When i try to access the IP:1337 i keep getting The connection was reset but i keep seeing some text in the background before it keep getting The connection was reset so i try to intercept it with burp suite.

![Image](https://imgur.com/w64iBM8.png)

So intercept the request send it to repeater and send and we have a message.

```
Welcome traveller, to the beginning of your journey
To begin, find the trollface
Legend says he's hiding in the first 100 ports
Try printing the banners from the ports
```

Ok let hit nmap.....

```Nmap -sV -p1-100 --script=banner <Target-IP>```

![Image](https://imgur.com/H9yb4gC.png)

Looking through the scan output we found a message again.

![Image](https://imgur.com/N6sYErS.png)

Go to port 12345 let hit it.

![Image](https://imgur.com/xIUMtU2.png)

```
NFS shares are cool, especially when they are misconfigured
It's on the standard port, no need for another scan
```
![Image](https://imgur.com/E9o4oQX.png)

Cool let mount it.

![Image](https://imgur.com/FtM1mhY.png)

Boom let change directory to it hmmm a zip file let transfer it to our Desktop.

![Image](https://imgur.com/l5KFSa7.png)

I try to unzip it but it was protected with a password so i try cracking it with John the Ripper, what is John The Ripper: John the Ripper is a free password cracking software tool. Originally developed for the Unix operating system, it can run on fifteen different platforms. 

![Image](https://imgur.com/LjxEHMf.png)

So i use rockyou.txt password list to crack the hash having the password i unzip the zip file.

![Image](https://imgur.com/UPKtfFZ.png)

Boom we have the first flag, a username hades and a private id_rsa key to crack and a hint to move forward.

![Image](https://imgur.com/Z33pkwi.png)

Cool hint scanning from 2500-4500 i think printing out the banner again i think we are searching for the ssh port i guess.

```Nmap -sV -p2500-4500 --script=banner <Target-IP> -oN 2500-4500.txt```

Since we save the scan output to a file let use grep to find the right ssh port.

![Image](https://imgur.com/4kpsT1L.png)

Boom we have the right port now let log in ssh.

![image](https://imgur.com/pSBP8ja.png)

And we are in but a little probs the shell is restricted.

![Image](https://imgur.com/96lhZYj.png)

we can easily bypass it with ```exec "/bin/sh"```

![Image](https://imgur.com/CEIp4iM.png)

We have user.txt let get the root flag now.

![Image](https://imgur.com/twGT7OB.png)

```getcap -r /``` cool it a /bin/tar = cap_dac_read_search+ep time to hit GTFOBins.

![Image](https://imgur.com/8KDmNpx.png)

Now let read the shadow file ```tar xf "/etc/shadow" -I '/bin/sh -c "cat 1>&2"'``` we can easily crack the shadow file with john the ripper i think i don't need to explain that again.

![Image](https://imgur.com/ZB2j67K.png)

After cracking the hash password we found in the shadow file we can easily log in with the root creds and finally we are root.

Box rooted what an hell lol..

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
