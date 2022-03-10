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

