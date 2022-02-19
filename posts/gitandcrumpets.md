---
layout: default
title : muzec - Git and Crumpets TryHackMe writeup
---

Ahhhhh man so we are doing THM today it been long i know we try my possible best to keep upating the THM writeups without wasting to much of time let jump in already so the fun can begin.

![image](https://user-images.githubusercontent.com/69868171/154799848-ffed4fa7-0f8a-4eea-9e6d-1fcdbd41696d.png)


We always start with an nmap scan.....

```nmap -p- --min-rate 10000 -oN nmap/full.tcp -v 10.10.84.166```

```
# Nmap 7.91 scan initiated Fri Feb 18 07:49:36 2022 as: nmap -p- --min-rate 10000 -oN nmap/full.tcp -v 10.10.84.166
Increasing send delay for 10.10.84.166 from 0 to 5 due to 11 out of 18 dropped probes since last increase.
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
adjust_timeouts2: packet supposedly had rtt of 8399426 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of 8399426 microseconds.  Ignoring time.
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
adjust_timeouts2: packet supposedly had rtt of 8037721 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of 8037721 microseconds.  Ignoring time.
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
Nmap scan report for 10.10.84.166
Host is up (4.3s latency).
Not shown: 65532 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
9090/tcp closed zeus-admin

Read data files from: /usr/bin/../share/nmap
# Nmap done at Fri Feb 18 07:55:24 2022 -- 1 IP address (1 host up) scanned in 347.80 seconds
```

No we know we have just 2 open ports let throw in some service detection and default nmap scripts to know what we are dealing with but we already know `22 is SSH` and `80 is HTTP` but no hard feeling in trying it.

```
nmap -sC -sV -oN nmap/normal.tcp -p 22,80 10.10.84.166
```

```
# Nmap 7.91 scan initiated Fri Feb 18 07:57:10 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p 22,80 10.10.84.166
Nmap scan report for 10.10.84.166
Host is up (0.20s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
80/tcp open  http    nginx
| http-title: Hello, World
|_Requested resource was http://10.10.84.166/index.html

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Feb 18 07:58:53 2022 -- 1 IP address (1 host up) scanned in 103.06 seconds
```

Now more better let check what we have on HTTP.

![image](https://user-images.githubusercontent.com/69868171/154800103-64ab1f48-b286-47ce-8c0e-ca8f32fa449b.png)

But strange just by opening the IP we got redirected to youtube damn we just got `rickroll` XD let use `curl` and see what we have in the source.

![image](https://user-images.githubusercontent.com/69868171/154800229-2035710a-b0a1-433c-99ff-8c91ca030c26.png)

Now let add that to our `/etc/hosts` file.

![image](https://user-images.githubusercontent.com/69868171/154800272-234abdc0-2f60-49c2-a530-396b48c1f630.png)

Now that is cool we have a `gitea` when i try checking for `version` seems it hidden let try and register.

![image](https://user-images.githubusercontent.com/69868171/154800316-9eb5369b-7c83-4f76-91e0-77bd50fd63f4.png)

Now we hit on `sign up` and boom boom.

![image](https://user-images.githubusercontent.com/69868171/154800357-17bfa8f5-24db-4036-aabd-32387116ec6e.png)

Now let click on `Explore` to see what we have.

![image](https://user-images.githubusercontent.com/69868171/154800380-186db584-0481-43b6-a8bb-cba22df78333.png)

We have two `repos` let see what we can dig out in the deleted commits it always cool to check that.

![image](https://user-images.githubusercontent.com/69868171/154800443-98c460e2-6062-4d97-b175-bf92cd31f4ac.png)

Five commits now that should be interesting everybody likes commits XD.

![image](https://user-images.githubusercontent.com/69868171/154800614-12333ce7-95aa-44b9-af57-b0f23c24b7ae.png)

Now that is interesting but who does that hidding Password in an avatar ahhhh let confirm it.

![image](https://user-images.githubusercontent.com/69868171/154800700-8b606bae-332e-4f00-9d9a-d2812cd59124.png)

So i downloaded it on my target to confirm if `scones` is right about his password gotten hidden in an avatar.

![image](https://user-images.githubusercontent.com/69868171/154800818-3d4e7e21-f1e5-4aef-97fc-073938ea8e41.png)

Now that should be easy seems `scones` password is `Password` ahhhhhh let confirm it.

![image](https://user-images.githubusercontent.com/69868171/154800949-eee60747-c382-47b6-8483-228d47856cc6.png)

Boom we are in going through `hydra` repo and found nothing but seems the version of the `gitea` is hidden why not check if we have `Git Hooks` enabled which should be possible to get RCE, how do you know?? WTF have some dignity and do some research XD lol just kidding.

![image](https://user-images.githubusercontent.com/69868171/154801102-f4adc644-0af8-4d2d-a2c4-d749e8653fa8.png)

Which was publish back in 2021, February got the exploit code but i decided to do it manully to know what am doing.

![image](https://user-images.githubusercontent.com/69868171/154801182-e518a034-d0ec-4121-a24a-bb87c8fbda18.png)

Now let click on `settings >> Git Hooks >> post-receive` .


![image](https://user-images.githubusercontent.com/69868171/154801312-e21fe84b-745b-4389-83ef-ad8cbfb5f6df.png)


So let add our reverse shell payload and update the `hooks` .

![image](https://user-images.githubusercontent.com/69868171/154801421-3c12281e-9f9e-4b87-b31e-e7bf77ec5ec7.png)

Now let move to the next step let start our `ncat listener` and make a commit to trigger our payload.

![image](https://user-images.githubusercontent.com/69868171/154801562-fc1ab6da-9097-4c72-b5bc-1132327016d1.png)

```
touch shell.md
git init
```

![image](https://user-images.githubusercontent.com/69868171/154801867-ba56ff1c-7941-4d04-ae8e-2e8e96bf8afe.png)

```
git add shell.md
git commit -m "Initial commit"
```

![image](https://user-images.githubusercontent.com/69868171/154801940-ee9930db-a8ee-4173-8ddc-c2b853252e42.png)

```
git remote add origin http://git.git-and-crumpets.thm/scones/cant-touch-this.git
git push -u origin master --force
```

So we add `scones` and `Password`  to push the commit which trigger the payload and boom we have a reverse shell. Let spawn a full tty shell.

![image](https://user-images.githubusercontent.com/69868171/154802101-9e9b674b-4603-41ec-8949-3f1f26082f7b.png)

Now let get the `user.txt` in the home directory.

![image](https://user-images.githubusercontent.com/69868171/154802230-133e5c8d-f6eb-4633-b05b-d5cfa68b283d.png)


### Privilege Escalation

Going through directories the only promising one is `/var/lib/gitea/data` we found `gitea.db` let check what we have on it using `sqlite3` .

![image](https://user-images.githubusercontent.com/69868171/154802716-7ec600d0-54c3-4f78-9fa1-56af8b05b5e4.png)

Now let type `sqlite3` to open the db file.

![image](https://user-images.githubusercontent.com/69868171/154802766-67bf4c41-304e-4951-9b61-64815d8134e4.png)

Boom let drop the `user` table.

![image](https://user-images.githubusercontent.com/69868171/154802829-315adef1-2009-46fe-8ef8-dbbe925eb6f0.png)

we know user `scones` is not an admin on `gitea` but since we have all permission to the `gitea.db` we can change all users to `admin` or change each users `password` let do that now.

```
select lower_name, is_admin from user;
```

![image](https://user-images.githubusercontent.com/69868171/154802915-609d3316-78d8-4b76-bdf4-d84df613f41c.png)

Only `hydra` have access has admin we can easily change `scones` ID to `1` which he will also have admin privilege let update that now.

```
UPDATE user SET is_admin=1 WHERE lower_name="scones";
```

![image](https://user-images.githubusercontent.com/69868171/154803013-77d74578-6c87-452f-bf00-1c51e19900ae.png)

Now let log back in to `gitea` .

![image](https://user-images.githubusercontent.com/69868171/154803063-d80dfff5-ba29-46ab-92e1-5858525ca5ed.png)

We are `admin` now time to reset all users `Password` .

![image](https://user-images.githubusercontent.com/69868171/154803086-d038c4f5-32b6-4238-9e86-d43a28f8dd7c.png)

Now i added new `password` for each users and reset `2FA` also XD.

![image](https://user-images.githubusercontent.com/69868171/154803143-ed068dc2-9e5a-440d-9135-165703a86a54.png)


![image](https://user-images.githubusercontent.com/69868171/154803158-0b949023-eef1-42b9-a495-2841f7945664.png)

Now let click on `update user account` and we should be good i do it for all users. we should get user account has been updated.

![image](https://user-images.githubusercontent.com/69868171/154803238-ac499524-a48f-4fe8-9df0-533aff618e50.png)

Now we can easily use the `password` to access each account to check there commits i firstly go for user `hydra` but got nothing on the commits let check user `root` now.

![image](https://user-images.githubusercontent.com/69868171/154803465-d9fd22a4-f61e-4116-a06b-2910127bec58.png)

Boom we got a `backup` repo with some pushed and deleted commits but the `ssh` with a `password` i guess caught my eyes let check it out.

![image](https://user-images.githubusercontent.com/69868171/154803525-5a23f06e-173c-4c9e-9892-62ce17ee154b.png)

Four commits let click on it and see what we have in store for us.

![image](https://user-images.githubusercontent.com/69868171/154803618-ab9c6d10-2f43-4985-ae75-80ec7f69f1a6.png)

Ahhhhhhhh awesome let see what we have on the `ssh` .

![image](https://user-images.githubusercontent.com/69868171/154803732-57cf39bc-6026-4def-a098-26122bc6d2fc.png)

Boom we found a private key ahhhhhh finally some progress i quickly copy and save it on my attacking machine let try using it with the `root` user on SSH.

![image](https://user-images.githubusercontent.com/69868171/154803876-41319dbf-edea-4fbe-a6f0-d6951e387f47.png)

Boom we are root and done for the private key password you already know it `Sup3rS3****` just look close and that all.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
