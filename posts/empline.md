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


![image](https://user-images.githubusercontent.com/69868171/134551554-a1fd8ef3-6434-49f3-9b4c-2f8f4113af30.png)

Since OpenCATS is an open source i went on `github` to check where the config.php file is located before running our exploit to be sure if it has a config.php file or not.
 
![image](https://user-images.githubusercontent.com/69868171/134551938-1e1475ea-7d9a-4ee7-81ba-e96b46fc247d.png)

Now let zip it back and upload the docx file on the target.

![image](https://user-images.githubusercontent.com/69868171/134552147-161adf24-6bce-45eb-a5a9-5c69fe090289.png)

Uploading the docx file on the target and boom the config in base64 encoding.

![image](https://user-images.githubusercontent.com/69868171/134552894-32b9d928-cc59-49ef-acfa-d7c42d5784c9.png)

Now let decode it using cyberchef.

![image](https://user-images.githubusercontent.com/69868171/134553287-3cc52ba3-de04-4dce-9308-23a035659382.png)

Boom credentials for mysql let hit it.

![image](https://user-images.githubusercontent.com/69868171/134553502-0483c2de-0412-41c1-9131-18bd5802af2a.png)

Now let find some credentials to loot.

![image](https://user-images.githubusercontent.com/69868171/134553784-fa78b151-2c95-44bc-be2a-2ea00512bd18.png)


![image](https://user-images.githubusercontent.com/69868171/134553857-e652fb05-0990-4023-b605-06086908a436.png)

![image](https://user-images.githubusercontent.com/69868171/134553955-e6052919-107a-4571-b0e9-990b9c37fd4f.png)

Boom another credentials probably for SSH first let look up the hash in a databases to crack first.

![image](https://user-images.githubusercontent.com/69868171/134554183-329a0af5-02d4-4fc2-8aac-9de22a7bd4a4.png)

Now let try it on SSH.

![image](https://user-images.githubusercontent.com/69868171/134554352-1e16c28a-ba5a-4ff6-a6c5-7607afcf8761.png)

We are in cool right and we have user.txt already check the passwd file seems we only have 2 users on the system `ubuntu and george` nothing to worry about the `ubuntu` user just some normal user.

![image](https://user-images.githubusercontent.com/69868171/134554855-ae8b42f1-8fc4-42e4-aa14-62f543a6ebd9.png)

Now let get root checking for SUID and Capabilities.

![image](https://user-images.githubusercontent.com/69868171/134555187-b18112e8-6c0c-4712-a751-9b99fe4aefd7.png)

`sudo -l` nothing on that now SUID.

![image](https://user-images.githubusercontent.com/69868171/134555775-1e88b6a0-464a-46fc-ade0-6fc56274cf26.png)

Clean aslo now Capabilities.

![image](https://user-images.githubusercontent.com/69868171/134555874-93f19291-c81a-45cc-956d-f2c7d838a164.png)

```
/usr/local/bin/ruby = cap_chown+ep
```
Ruby seems interesting with the chown capablity command to change the owner of a file system files now let abuse it.

![image](https://user-images.githubusercontent.com/69868171/134556384-6d16b68d-597a-4c2e-8762-238e7c532dff.png)

```
ruby -e 'puts File.chown(1002, 1002, "/etc/shadow")'
```

Now the shadow file belong us we can easily change the root password sweet right??

![image](https://user-images.githubusercontent.com/69868171/134556697-be140f0b-e68d-4dbe-8fc7-329d51ee2113.png)

Now editting the shadow file with nano.

![image](https://user-images.githubusercontent.com/69868171/134556910-0a5bc231-c7cf-41e7-9860-e062fad2dfd6.png)

Save and exit. Now let use the password with just add to the shadow file with the root user.

![image](https://user-images.githubusercontent.com/69868171/134557033-eb166b9e-24b1-4edd-8aba-7a084b570c9b.png)

We are root and done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
