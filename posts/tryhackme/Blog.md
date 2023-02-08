---
layout: default
title : muzec - Blog Writeup
---

![Image](https://miro.medium.com/max/700/1*Sx8UMBarKnpgZ6UQXOuOmQ.png)

# Tools I Use To Complete The Room.

1. Nmap
2. Metasploit
3. Wpscan
4. Google
5. ltrace

Let get started.

Billy Joel made a blog on his home computer and has started working on it. It’s going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it! Billy has some weird things going on his laptop. Can you maneuver around and get what you need? Or will you fall down the rabbit hole…

In order to get the blog to work with AWS, you’ll need to add blog.thm to your /etc/hosts file.


# Nmap

```
Nmap 7.80 scan initiated Sun Jul 12 19:14:28 2020 as: nmap -sC -sV -oA nmap blog.thm
Nmap scan report for blog.thm (10.10.96.47)
Host is up (0.14s latency).
Not shown: 996 closed ports
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
| 256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_ 256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp open http Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open netbios-ssn Samba smbd 3.X — 4.X (workgroup: WORKGROUP)
445/tcp open netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
| OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
| Computer name: blog
| NetBIOS computer name: BLOG\x00
| Domain name: \x00
| FQDN: blog
|_ System time: 2020–07–12T23:14:59+00:00
| smb-security-mode:
| account_used: guest
| authentication_level: user
| challenge_response: supported
|_ message_signing: disabled (dangerous, but default)
| smb2-security-mode:
| 2.02:
|_ Message signing enabled but not required
| smb2-time:
| date: 2020–07–12T23:14:59
|_ start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Sun Jul 12 19:15:04 2020–1 IP address (1 host up) scanned in 35.92 seconds
```


We have 4 open ports 22 which is SSH port 80 normal HTTP port 139,445 which is samba.

Let start enumerating.


# 5 What version of the above CMS was being used?

![Image](https://miro.medium.com/max/700/1*zLdcl3MHySfiBclfNMjZZQ.png)

# 4 What CMS was Billy using?

Answer: Wordpress

# 3 Where was user.txt found?

Hehehehe we need to get access to the machine let enumerate hard.

![Image](https://miro.medium.com/max/700/1*cJhBD9dErLzAJzO57BKsIg.png)

A note from mom let check what it said;

![Image](https://miro.medium.com/max/610/1*XL0vPtUUVvCOepWPwmg_4A.png)

Karen Wheeler that is Billy mom cool actually she created the blog for Billy to get things up.

![Image](https://miro.medium.com/max/700/1*6qwPqpqSxSET9Wv8OVnzdw.png)

Let click on the name to get more details on the user.

![Image](https://miro.medium.com/max/700/1*Et5WT53TrAY68U_LRjU6AQ.png)

Cool we have the author name kwheel let try some brute force with wpscan command ```wpscan — url http://blog.thm/ — usernames kwheel — passwords rockyou.txt``` so i use the rockyou.txt wordlist.

Let it fly.

![Image](https://miro.medium.com/max/700/1*YzCXvqUkPXfVc3Y5tirqxw.png)

Cool we have the details let hit the login page to log in ```http://blog.thm/wp-admin```.

![Image](https://miro.medium.com/max/700/1*GqRpSMawXlmAMb7WG4Mw9A.png)

Boom we are in but it just a normal author account not an admin account let do some research.

![Image](https://miro.medium.com/max/700/1*rB_cgaatlxikIxXnZalnmQ.png)

So with the research i came across something interesting on rapid7 about a vulnerable on wordpress 5.0 details below.

WordPress versions 5.0.0 and <= 4.9.8. The crop-image function allows a user, with at least author privileges, to resize an image and perform a path traversal by changing the _wp_attached_file reference during the upload.

The second part of the exploit will include this image in the current theme by changing the _wp_page_template attribute when creating a post. This exploit module only works for Unix-based systems currently.

![Image](https://miro.medium.com/max/700/1*WSNSLGOPwUZwClM6iQ0y0Q.png)

Hit exploit.

![Image](https://miro.medium.com/max/700/1*eX00VXqJOtyH5PVLitdybQ.png)

Boom and we have Meterpreter shell cool let get into normal shell by typing… shell.

![Image](https://miro.medium.com/max/700/1*TC15X6wQnU_w4a73S840cg.png)

Ok let try harder.

![Image](https://miro.medium.com/max/700/1*YBgj7xFSwx242D2PF235ow.png)

So after enumerating so hard i found a custom SUID file checker owned by root let run ltrace on it.

![Image](https://miro.medium.com/max/700/1*ndsvZPCaFLcurP2MNl60Jw.png)

```getenv(“admin”) = nil
puts(“Not an Admin”) = 13
Not an Admin
+++ exited (status 0) +++```

Let try to export a variable and run the checker file.

```export admin=1```

![Image](https://miro.medium.com/max/700/1*XKvk52TxBNEoUiT9jjD7jg.png)

Answer: /media/usb

# 2 User.txt

![Image](https://miro.medium.com/max/700/1*xX5i536L6x7bXMBWoTt1wg.png)

# 1 Root.txt

![Image](https://miro.medium.com/max/700/1*Pct4yaDNh0rTUiZNdXdLNQ.png)

And we are done box rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
