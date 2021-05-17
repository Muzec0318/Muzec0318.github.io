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

Was also hidden in hex using the same method on ` Don't let it run `  i was able to retrieve the flag.

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

### Web:-  
