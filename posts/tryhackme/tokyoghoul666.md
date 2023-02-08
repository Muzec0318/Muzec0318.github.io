---
layout: default
title : muzec - Tokyo Ghoul Writeup
---

![Image](https://i.imgur.com/tuzTqo4.gif)


Tokyo Ghoul Hmmmmm Anime i love, Ok Let's do some scanning .

```Nmap -sC -sV -oA nmap <Target-IP>```

![Image](https://imgur.com/0od6jyl.png)

We got 3 open ports 21,22 and 80 cool let check out the port 80 first.

![Image](https://imgur.com/RqqfDyN.png)

Ken Kaneki is a regular high school teenager who decides to go on a date with a girl named Rize Kamishiro. Kaneki fails to notice that there is something unusual about her. The girl then shows her true form and transforms into a ghoul who is hungry for Kaneki flesh. But suddenly, steel beams fall on her from the ceiling and she is instantly killed. Left in a very critical state, Ken is rushed to a hospital nearby. When he regains his consciousness, the doctor informs him that his organs have been replaced with that of Rize .

Kaneki is kidnapped by Jason. He then uses the most brutal ways to torture him by cutting off parts of him but gives him just enough time to regenerate again. While Kaneki seems to take the physical torture like a champ, he begins to struggle when he is reminded of the two other ghouls who gave him hopes of escaping.

Clicking on ```Can you help him escape?``` take us to a seperate page.

![Image](https://imgur.com/3leEQRx.png)

Just an image ok let view the page source maybe an hint is left or maybe not.

![Image](https://imgur.com/qTLDf6i.png)

Boom we are right some hints is left behind let check the hint.

```look don't tell jason but we will help you escape , here is some clothes to look like us and a mask to look anonymous and go to the ftp room right there you will find a freind who will help you```
  
  Now let hint the ftp since Anonymous FTP login allowed.
  
  
 ![Image](https://imgur.com/tgFZ3KK.png)
 
 So we get all the files we can get on the Ftp server to our machine now let go through it.

A note from Aogiri tree interesting let read it.

 ![Image](https://imgur.com/nlFN9JG.png)

Let move to the next file so one of the file is an ELF file so i give it permission and run it.

 ![Image](https://imgur.com/HE1eqwp.png)
 
 Interesting so we need a paraphrase cool let strings the file and check maybe it our lucky day.
 
 ![Image](https://imgur.com/NnT2E48.png)
 
 Boom we got something let try it.
 
 ![Image](https://imgur.com/KtKBiWk.png)
 
 I guess we still have one more file to check which is an image let try steghide with the new password we just found.
 
 ![Image](https://imgur.com/kqfAAoh.png)
 
 A secret txt file let cat it an see.
 
 ![Image](https://imgur.com/lJ8yAyi.png)
 
 We got some kind of weird Messages let use cyber chef to make it more clear.
 
 From Morse ----> Hex -----> Base64  -------> We got the hidden secret directory.
 
 ![Image](https://imgur.com/dQk7t8t.png)
 
 
 ![Image](https://imgur.com/eJRx1nO.png)
 
 Hmmm let burst dirs maybe some directory is still hidden.
 
 ![Image](https://imgur.com/ZAikanx.png)
 
 Yea one more hidden directory gotten let access it.
 
 ![Image](https://imgur.com/20whXGn.png)
 
 So let check around going through the pages i found something interesting probably an (LFI) Local File Inclusion vulnerability not so sure but why not let try it.
 
 ![Image](https://imgur.com/rwlqRCj.png)
 
 So the first payload i use i got nothing in return just an annonying message back lol  ```no no no silly don't do that```  so i try encoding the payload url encoding ```../../../../../../../etc/passwd``` to ```%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2Fetc%2Fpasswd```
 
 ![Image](https://imgur.com/nyGCV11.png)
 
 And i was able to read the passwd file which we found a username with and hash to crack which you can easily use john the ripper for that i guess i don't need to explain that part now let log in SSH with the credentials.
 
 
 ![Image](https://imgur.com/WXqpKTp.png)
 
 Boom we are in and we got user.txt time to get the root flag.
 
 ![Image](https://imgur.com/IHK1aTe.png)
 
 Cool we can run sudo on the jail.py let cat it and see what we have.
 
 ![Image](https://imgur.com/aXgslAB.png)
 
 With some little research on Google i found out it a Python jail and we can break out of it.
 
 ![Image](https://imgur.com/aXgslAB.png)
 
A glance at the source code and we can figure out what to do. We need to somehow exec a command of our choice beating the condition checks. The program checks for existence of eval, exec, import, open, os, read, system, write in our input. The challenge is to use python builtins to break out of this jail.
 
To start with lets read how python evaluates these statements. Example: if you write “import os” in a python script, python must be getting a function object “import” and passes it “os” as input and gets a class of “os” with the relevant methods.

Python allows us to use built in objects using the __builtins__ module. Lets check it out: https://docs.python.org/3/library/builtins.html

Now let run ```sudo /usr/bin/python3 /home/kamishiro/jail.py```  

Now let copy and paste -------> ```__builtins__.__dict__['__IMPORT__'.lower()]('OS'.lower()).__dict__['SYSTEM'.lower()]('/bin/bash')```

![Image](https://imgur.com/XSqd8b2.png)

Boom we are root..........Box Rooted.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>








