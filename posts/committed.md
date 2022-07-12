---
layout: default
title : muzec - Committed THM Writeup
---

![image](https://user-images.githubusercontent.com/69868171/178382896-41f05777-5e9c-435d-8986-055876bb2924.png)

Damn it been a while guys hehehehe yes i know you missed me XD without wasting to much of time today we will be working on a easy machine name `committed` it all about `git` man it easy just follow up.

Oh no, not again! One of our developers accidentally committed some sensitive code to our GitHub repository. Well, at least, that is what they told us... the problem is, we don't remember what or where! Can you track down what we accidentally committed?

Access this challenge by deploying the machine attached to this task by pressing the green "Start Machine" button. You will need to use the in-browser view to complete this room. Don't see anything? Press the "Show Split Screen" button at the top of the page.

The files you need are located in `/home/ubuntu/commited` on the VM attached to this task.

It a zip file already we just need to unzip it and access the file.

![image](https://user-images.githubusercontent.com/69868171/178384104-f35a80bd-f282-45ec-829c-80664cf35a1d.png)

Now that is what we need let get in and see what logs we can find.

![image](https://user-images.githubusercontent.com/69868171/178384312-9058b1da-add6-4d4f-8227-6b657a931ae3.png)

We can see some commit made on the repo but what about if we are missing some importants commits possible it deleted let try checking with `git log --reflog` .

![image](https://user-images.githubusercontent.com/69868171/178384763-182276ad-bb08-4632-afb8-7cc1477ea4d9.png)

Now we have more commits to check and without wasting to much of time we got the flag.

![image](https://user-images.githubusercontent.com/69868171/178384941-5e2f73aa-b122-42ad-ab4a-c072429f8ea7.png)

Boom right and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
