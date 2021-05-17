---
layout: default
title : muzec - dCTF 2021 CTF Writeup
---

dCTF 2021 was hosted by DragonSec SI the CTF lasted from 14.5.2021 at 15:00 UTC to 16.5.2021 at 21:59 UTC so i decided to share some of the tasks solutions.


### Misc:-  Encrypted the flag I have
![image](https://user-images.githubusercontent.com/69868171/118518636-53e2ee80-b706-11eb-84d6-7f5d206ea8e6.png)

So we download the png file but it was a strange one really but with a bit of research we find out it was Aurebesh alphabet, a writing system used to write in Basic, a language used in the Star Wars universe.

![image](https://user-images.githubusercontent.com/69868171/118519170-de2b5280-b706-11eb-8706-14bcabdf5012.png)

Using `https://www.dcode.fr/aurebesh-alphabet`i was able to decrpyt it and we got our flag.

![image](https://user-images.githubusercontent.com/69868171/118519943-93f6a100-b707-11eb-9ff3-fce5f149069c.png)

`DCTF{MASTERCODEBREAKER}` 

### Misc:-  Don't let it run 
![image](https://user-images.githubusercontent.com/69868171/118522416-0799ad80-b70a-11eb-9371-2d94c9355c34.png)

Downloading pdf file so i open it with `http://icyberchef.com/` going through it saw some hidden hex text so decoding it  and boom we have the flag.

![image](https://user-images.githubusercontent.com/69868171/118523457-264c7400-b70b-11eb-973c-cea8abe1aeaa.png)

`dctf{pdf_1nj3ct3d}`


### Misc:-  Hidden message
![image](https://user-images.githubusercontent.com/69868171/118523652-54ca4f00-b70b-11eb-9526-733cc3c7ccc4.png)

Pretty easy using zsteg and boom we have the flag.

![image](https://user-images.githubusercontent.com/69868171/118524129-d3bf8780-b70b-11eb-89a8-651aa8754bde.png)

`dctf{sTeg0noGr4Phy_101}`

### Misc:-   Leak Spin  
![image](https://user-images.githubusercontent.com/69868171/118524323-09647080-b70c-11eb-8229-9ef822f9902e.png)

We have confident insider report that one of the flags was leaked online. Can you find it? checking twitter nothing so i decided to check github and guess what am right.

![image](https://user-images.githubusercontent.com/69868171/118524701-6cee9e00-b70c-11eb-82b3-92ff27e90df8.png)

One of the challenges for the upcoming DCTF1. Make sure to keep this info private! also A really simple challenge, if you are reading this, you are close! 

![image](https://user-images.githubusercontent.com/69868171/118524913-a7583b00-b70c-11eb-9e04-04188c48d28b.png)

```
name: "Leak Spin"
author: "Miha M."
category: Web

description: We have confident insider report that one of the flags was leaked online. Can you find it?
value: 100
type: standard

flags:
  - dctf{I_L1k3_L1evaAn_P0lkk4}

tags:
  - web

state: visible
  
version: "1.0"
```
And we have our flag .

`dctf{I_L1k3_L1evaAn_P0lkk4}`

### Misc:-  Bad Apple 
![image](https://user-images.githubusercontent.com/69868171/118526737-8bee2f80-b70e-11eb-8a74-02bbbc4133aa.png)

Someone stumbled upon this file in a secure server. What could it mean?

Going through it with sonic visualizer found a barcode .

![image](https://user-images.githubusercontent.com/69868171/118527142-fe5f0f80-b70e-11eb-9867-502380b63b1a.png)

Now converting it to black and white and using some online tools to read it give us the flag .

![image](https://user-images.githubusercontent.com/69868171/118527486-59910200-b70f-11eb-8a56-36db3915d234.png)

`dctf{sp3ctr0gr4msAreCo0l}`

### Misc:-   Show us your ID  
![image](https://user-images.githubusercontent.com/69868171/118527589-7299b300-b70f-11eb-9bdb-2c202e453351.png)

Was also hidden in hex using the same method on `Don't let it run`  i was able to retrieve the flag.

![image](https://user-images.githubusercontent.com/69868171/118528237-1aaf7c00-b710-11eb-8108-b53bda69a164.png)

`dctf{3b0ba4}`


### Crypto:-  Strong password 

![image](https://user-images.githubusercontent.com/69868171/118529141-2a7b9000-b711-11eb-86b3-78b604d738e1.png)

A zip file but protected with password let try to crack it using john the ripper.

![image](https://user-images.githubusercontent.com/69868171/118529505-92ca7180-b711-11eb-96d0-bf340bddc829.png)

Now using john the ripper with the rockyou.txt password list to crack it .

`john --wordlist=/usr/share/wordlists/rockyou.txt crack`

Since i already crack it and have the password no need for me to crack it again password in plain below; 

![image](https://user-images.githubusercontent.com/69868171/118529957-1b491200-b712-11eb-9242-39f42c8840a6.png)

Unzip it with the password and we have some txt file with lot of words lol searching through it and we have the flag.

![image](https://user-images.githubusercontent.com/69868171/118530365-901c4c00-b712-11eb-8e67-8aafbde5942d.png)

`dctf{r0cKyoU_f0r_tHe_w1n}`

### Crypto:-   This one is really basic 

![image](https://user-images.githubusercontent.com/69868171/118531594-01a8ca00-b714-11eb-8ba8-671fcab88ef3.png)

The answer to life, the universe, and everything.

![image](https://user-images.githubusercontent.com/69868171/118532039-8562b680-b714-11eb-9129-e958cdb9e8ab.png)

`dctf{Th1s_l00ks_4_lot_sm4ll3r_th4n_1t_d1d}`

### Web:-  Simple web 

![image](https://user-images.githubusercontent.com/69868171/118532667-38331480-b715-11eb-906e-2dd9ab2fa5cd.png)

Ok we are visited with a page with `i want a flag` but when i click on it i get some error `Not authorized` .

![image](https://user-images.githubusercontent.com/69868171/118532935-81836400-b715-11eb-8f3e-fb7a50d0cafb.png)

![image](https://user-images.githubusercontent.com/69868171/118533045-9bbd4200-b715-11eb-9e30-00d23dfbea12.png)

Now let check the source code to see what we are missing .

![image](https://user-images.githubusercontent.com/69868171/118533163-babbd400-b715-11eb-9649-3185fb86e551.png)

We can see the auth is `0` let try changing it to `1` using inspect element .

![image](https://user-images.githubusercontent.com/69868171/118533324-f191ea00-b715-11eb-95de-f5f0c1b86628.png)

When i click on submit it now boom we have the flag .

![image](https://user-images.githubusercontent.com/69868171/118533440-14bc9980-b716-11eb-8145-31fc03bdbe62.png)

`dctf{w3b_c4n_b3_fun_r1ght?}`

### Web:-   Injection 
![image](https://user-images.githubusercontent.com/69868171/118533982-bb089f00-b716-11eb-88ea-1e30e88c1063.png)

A simple login page but not that simple and not related to sql injection first thing first i try to confirm it if it vulnerable to (SSTI) Server Side Template Injection thanks to my little cheat sheet `${{<%[%'"}}%` ok let confirm it.

![image](https://user-images.githubusercontent.com/69868171/118534492-4bdf7a80-b717-11eb-934a-746cf99194f4.png)

![image](https://user-images.githubusercontent.com/69868171/118535416-7251e580-b718-11eb-988f-0b27d0cea18c.png)

Cool we got `500 Internal Server Error` now let craft our payload all thanks to our captain `@Kiomet` for the hard work here.

![image](https://user-images.githubusercontent.com/69868171/118535468-8269c500-b718-11eb-93a8-49e2316ac00b.png)

Payload ready `http://dctf1-chall-injection.westeurope.azurecontainer.io:8080/%7B%7Bconfig.__class__.__init__.__globals__['os'].popen('ls -la').read()}}`

![image](https://user-images.githubusercontent.com/69868171/118536338-9f52c800-b719-11eb-88c2-545319044446.png)

You can view in page source to make it more clean to see.

`http://dctf1-chall-injection.westeurope.azurecontainer.io:8080/%7B%7Bconfig.__class__.__init__.__globals__['os'].popen('cat lib/security.py').read()}}`

![image](https://user-images.githubusercontent.com/69868171/118537078-7e3ea700-b71a-11eb-99f7-3d2b53e612c7.png)

Now getting the flag let reverse and decode the password we found.

![image](https://user-images.githubusercontent.com/69868171/118537214-b1813600-b71a-11eb-84b7-a9d2c3b5758f.png)

`dctf{4ll_us3r_1nput_1s_3v1l}`

### Web:-    Very secure website 
![image](https://user-images.githubusercontent.com/69868171/118537322-d2e22200-b71a-11eb-8b39-2ca41e83312e.png)

Some students have built their most secure website ever. Can you spot their mistake?

Checking the source code that was left behind for us.

```
 <?php
    if (isset($_GET['username']) and isset($_GET['password'])) {
        if (hash("tiger128,4", $_GET['username']) != "51c3f5f5d8a8830bc5d8b7ebcb5717df") {
            echo "Invalid username";
        }
        else if (hash("tiger128,4", $_GET['password']) == "0e132798983807237937411964085731") {
            $flag = fopen("flag.txt", "r") or die("Cannot open file");
            echo fread($flag, filesize("flag.txt"));
            fclose($flag);
        }
        else {
            echo "Try harder";
        }
    }
    else {
        echo "Invalid parameters";
    }
?> 
```

I try using admin/admin for both username and password but got no luck so i try to use magic hashes for PHP  cheat sheet below;

![image](https://user-images.githubusercontent.com/69868171/118537938-8e0abb00-b71b-11eb-8168-879eb916ec5f.png)


Now using `admin` for username and `LnFwjYqB` for password.

![image](https://user-images.githubusercontent.com/69868171/118538080-ba263c00-b71b-11eb-8611-b5185adcf4a9.png)

And boom we have the flag .

![image](https://user-images.githubusercontent.com/69868171/118538200-e2159f80-b71b-11eb-8f33-08bee1b9fd23.png)

`dctf{It's_magic._I_ain't_gotta_explain_shit.}`

### Web:-     DevOps vs SecOps 
![image](https://user-images.githubusercontent.com/69868171/118538328-07a2a900-b71c-11eb-84cf-cb4edfea05c1.png)

Automatization is amazing when it works, but it all comes at a cost... You have to be careful...

Now back to the github page .

![image](https://user-images.githubusercontent.com/69868171/118538485-3e78bf00-b71c-11eb-8a8d-bd325f3e72f7.png)

Some really interesting hint.

![image](https://user-images.githubusercontent.com/69868171/118538544-53ede900-b71c-11eb-8e60-c2e8e7b2c458.png)

![image](https://user-images.githubusercontent.com/69868171/118538634-72ec7b00-b71c-11eb-8aeb-3f4b72f89620.png)

Clicking on the `.github` .

![image](https://user-images.githubusercontent.com/69868171/118538704-8b5c9580-b71c-11eb-8c8f-f9f996514d79.png)

Now `setup.py` .

![image](https://user-images.githubusercontent.com/69868171/118538796-a7603700-b71c-11eb-8f04-9776dbdb3cce.png)

And we have our flag .

`dctf{H3ll0_fr0m_1T_guy}`

Yea i know all the challenges are cool aand all thanks to my teammates `@fr334aks` it was really fun playing with you all.

![image](https://user-images.githubusercontent.com/69868171/118539196-205f8e80-b71d-11eb-8e34-44ff0eeabcb4.png)

Boom and we stand at 15th place out of 1084 teams.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
