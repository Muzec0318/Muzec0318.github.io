### Hello I'm Abzee. 
I will be the one walking you through the CTF i participated which is very fun and I learnt a lot.

ShellCTF 2022 which is organise by Shellpwn Team the CTF lasted from 15:30 EAT of August 12th to 15:30  EAT of 14th of August so i decided to share the walkthrough of the challenges I did by myself.

### Misc: World's Greatest Detective

![image](https://user-images.githubusercontent.com/69868171/184542367-4188bb6b-7744-4d13-a818-bc87ba61c8d9.png)

So we downloaded the png file but it was a strange one thou but looking at the png file name it's said Wakanda_Translator. I quickly rush to my browser and search for Wakanda Translator cipher online.

![image](https://user-images.githubusercontent.com/69868171/184542670-fa1bf162-eae2-40b0-99ad-7429c46ff756.png)

Using this site https://www.dcode.fr/wakanda-alphabet I was able to decrypt the msg and got our flag.

![image](https://user-images.githubusercontent.com/69868171/184543096-ae6491eb-2a9b-4c9b-b14a-326c7b657167.png)

Flag: SHELLCTF{W4kandA_F0rev3r}.

Easy Peasy right.


### Crypto: Tweeeet

![image](https://user-images.githubusercontent.com/69868171/184543322-6d455d91-0b39-4ca3-ba64-4995f1702ee5.png)

These was really easy pals after we downloaded the jpeg file we were task to find out what are the birds sitting on the wire are trying to say. 

![image](https://user-images.githubusercontent.com/69868171/184543589-5000026f-d008-44f8-befa-540ab77066fd.png)

First of all I rush to google and search about birds cipher online to check maybe I can get anything but guess what we where able to find one by using this this link https://www.dcode.fr/birds-on-a-wire-cipher.

![image](https://user-images.githubusercontent.com/69868171/184544047-ad26cfe2-23f2-477f-ae90-605e5b5e4608.png)

Boom we have our flag: SHELL{WELOVESINGING}.

### Crypto: Tring Tring.... 

![image](https://user-images.githubusercontent.com/69868171/184544200-549ee715-363d-4bb3-8c9f-c6e4f0c8e85b.png)

We all know that this is a Morse code cipher but that's not the end of the encryption. Let check it out and see what we are going to get, So I decided to use online tools called Cyberchef.com.

![image](https://user-images.githubusercontent.com/69868171/184544520-93b4ebb7-8949-4477-a635-d1a6ba0e39c3.png)

Hmmm interesting I think we got a number when decrypting the Morse code and that number look like an SMS phone code. So quickly search for SMS phone code cipher online and I got one using this link https://www.dcode.fr/multitap-abc-cipher.

![image](https://user-images.githubusercontent.com/69868171/184544784-3853bb6c-2ded-4489-982e-5f5c1beeefcb.png)

Subarashii we where able to get the flag: SHELL{YOUCANREADMYSMS}.

### Crypto: MALBORNE

![image](https://user-images.githubusercontent.com/69868171/184544987-c9f0850e-42bf-48fc-9f88-f519bb8ba603.png)

This challs really gave me a tough time after doing a lot of research I decided to use the header of the challenge which is Malborne. From trying and error i was able to get a site that look similar to the name of the challs so I decided to give a shot and guess what buddy the flag just pop up using this site https://malbolge.doleczek.pl/.

![image](https://user-images.githubusercontent.com/69868171/184545289-08db3017-b46a-43b2-81b3-9d62067e5927.png)

Now we have the flag: SHELL{m41b01g3_15_my_n3w_l4ngu4g3}

### Crypto: OX9OR2

![image](https://user-images.githubusercontent.com/69868171/184545369-c58617af-b992-41cc-99fb-5ee228e41d2b.png)

Finally we are dealing with python crypto thou I'm not too good in python but this one is quite easy that I decided to use two ways to get the flag.

![image](https://user-images.githubusercontent.com/69868171/184545770-4fd1e266-486e-496e-8971-42421da19ee6.png)

First of all I check the content of the encryption.py and the encrypted file. So before we can run anything here we have to know the key of the encrypted file so I decided to write a code out and name it exp.py.

![image](https://user-images.githubusercontent.com/69868171/184545881-c358dd8d-dbe7-4273-b393-7ddcbaf03808.png)

This is the python code that I wrote so that I will be able to get the key.

![image](https://user-images.githubusercontent.com/69868171/184545933-06fc99d0-cc39-4d78-9889-ece5b4f905e6.png)

After executing it there is something that caught my eye there which is XORISC in the image above. That's means we have gotten half of our key which you can brute force for the rest and also guess it so the missing part I guess it by myself as XORISCOOL because it really make sense like that don't you think so Lol.

Now that we have gotten the Key which is XORISCOOL. I wrote a new code so that the key can be fit in I named It exploit.py.

![image](https://user-images.githubusercontent.com/69868171/184546415-577d3b5e-ce1b-4ab4-9020-62716d7188ce.png)

Now let run it with python3 and confirm maybe we are right.

![image](https://user-images.githubusercontent.com/69868171/184546473-a404a8cc-57f2-49c1-982c-e78e1d8875f0.png)

Wow look like it work, Here goes our flag: SHELL{X0R_1S_R3VeR51BL3}.

The other way to get the flag is using Cyberchef unless you know the key.

![image](https://user-images.githubusercontent.com/69868171/184546610-6e3fb4fc-de5b-4b90-9e7a-ed6c1ffc71db.png)

By uploading the encrypted file and knowing the key we where able to get the flag.


### Forensics: Alien Communication

![image](https://user-images.githubusercontent.com/69868171/184546829-2bc19a33-7781-4491-83df-5ecc0056c0fe.png)

Let downloaded the audio and use Sonic visualization on it maybe we can get something from it.

![image](https://user-images.githubusercontent.com/69868171/184547086-68b2d575-a5f9-4dcb-97ea-12cff71be3c0.png)

Next step to take is to add it to your Spectrogram and check.

![image](https://user-images.githubusercontent.com/69868171/184547196-9fcff680-0e96-458f-a46b-f3aa6a154249.png)

There goes our flag: SHELL{y0u_g07_7h3_f1ag}.

### Forensics: Secret Document

![image](https://user-images.githubusercontent.com/69868171/184547337-e7e856c8-cb56-41e9-8ed6-de6aadfd0457.png)

Looking at the description of the challenge we are already been given the key which is shell and a hint which is Xorry. Should know in your mind that xorry means from xor so I quickly rush to cyberchef after I downloaded the data.

![image](https://user-images.githubusercontent.com/69868171/184547531-3a7e9a63-00e8-4ae9-8317-b43afd45f4e4.png)

I uploaded it to cyberchef then choose from Xor and put the key that was given and If you can see the data is showining us that it PNG image. Now all we have to do is to convert it to image,

![image](https://user-images.githubusercontent.com/69868171/184547999-21203bd3-e197-431d-91a2-7585f7a6fe08.png)

Here goes our flag: SHELL{y0u_c4n_s33_th3_h1dd3n}.

### Forensics: Hidden File

![image](https://user-images.githubusercontent.com/69868171/184548097-5763a676-1eb6-449d-9121-03327de6a9d5.png)

First of all let download the jpg image and try a tools call Exiftool on it.

![image](https://user-images.githubusercontent.com/69868171/184548167-6a22bdcf-c219-4076-b532-da7078c5897e.png)

If can see there is a password there which is shell so let try steghide on the image.

![image](https://user-images.githubusercontent.com/69868171/184548336-a8947dc4-0246-42d2-8c31-9933226f83ef.png)

Using steghide we were able to get a hidden zip file. Now let unzip the file with a command called unzip.

![image](https://user-images.githubusercontent.com/69868171/184548447-a55a0be4-45e1-4ab2-af5e-9ad76248c876.png)

When unzipping it we were able to retrieve three file which is se3cretf1l3.pdf, something.jpg and flag.zip. When trying to unzip the flag.zip it was asking for password then i started digging to see maybe i can see something, first of all i tried zip2john but i got not then i decided to go through se3cretf1l3.pdf and look well then i got the password for flag.zip file.

![image](https://user-images.githubusercontent.com/69868171/184548766-796fd66d-6b19-495c-9e5d-74798cdee390.png)

As you can see the password is shellctf. Now let unzip it with the password.

![image](https://user-images.githubusercontent.com/69868171/184549090-0eed34ab-fd27-492c-a268-40aefd69a0d1.png)

After unzipping it we were able to get our flag: shell{y0u_g07_th3_flag_N1c3!}.













