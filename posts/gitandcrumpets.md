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

Now let click on `setting` 
