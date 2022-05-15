---
layout: default
title : muzec - Cysec CTF 2022 Qualifier Writeup
---

A little words to keep us going XD

```
Do you have what it takes to beat the REST and be the BEST? 
```

![image](https://user-images.githubusercontent.com/69868171/168483729-be459be2-8f3b-412d-b456-c4b84ca27937.png)


Now that is a fun CTF challanges without wasting to much of a time so let jump in already to the fun part solving challanges we are starting with `WEB` category first.


### Easy WEB -- 500 Point

![image](https://user-images.githubusercontent.com/69868171/168483904-32c29500-545a-4fac-9ece-6fed5d57b799.png)

Now that we have the target URL let hit it.

![image](https://user-images.githubusercontent.com/69868171/168483982-3d755b2e-ea01-4e55-9207-9a268d250fa5.png)

Now that is a plain website quickly checking `robots.txt` and we have nothing we just got the same homepage back to us why not check the `source` for hint or way to move forrward.

![image](https://user-images.githubusercontent.com/69868171/168484087-36daee98-c896-439e-90b6-63dccf6da32c.png)

Now that is interesting we can see the `Add user` with a comment but the most intersting one that caught my eyes was `/api?id=source` we all love `API` XD so let check it out.

![image](https://user-images.githubusercontent.com/69868171/168484200-ef6ea876-9632-408b-9ef9-5aee1fc2c435.png)

That is a step forward let see what we have in the source code we just downloaded i will be using my terminal.

![image](https://user-images.githubusercontent.com/69868171/168484462-1d1205b2-0c07-4f14-917d-d19ad054d65e.png)

Let just get the flag already using `Curl` .

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF]
└─$ curl -X GET https://cysec-easy-web.herokuapp.com/api --data "id=gimme-flag"
{"message":"CYSEC{eazy_peasy_l3m0n_squeezy}"}                                                                                                                                                                       
```

Flag:- `CYSEC{eazy_peasy_l3m0n_squeezy}`


### Breadcrumbs -- 1000 Point

![image](https://user-images.githubusercontent.com/69868171/168488158-762d8851-de3c-4860-8f0e-185ecb63c3cb.png)


A fun one seems like we are stealing a cookie lol let hit the webpage to see what we have.

![image](https://user-images.githubusercontent.com/69868171/168488229-069ca61b-aa6a-4694-a3d4-67262edf764b.png)

Seems we have a `login` page and `register` page and we have no valid credentials and it was stated already no `brute forcing` why not get our self a register account yes bro i mean let register XD.


![image](https://user-images.githubusercontent.com/69868171/168488482-988d0427-f4e4-4af1-8439-78940a1afd4c.png)

Now that we have an account let log in.

![image](https://user-images.githubusercontent.com/69868171/168488577-21995164-845f-466d-857a-f178d7296dd2.png)

In a dashboard hehehehehe don't be happy yet it just the normal account i created and seems we have some interesting pages to explore.

![image](https://user-images.githubusercontent.com/69868171/168488658-1b491275-0d52-46ff-9bd7-9acf64deb268.png)

The support page seems cool maybe XSS i just don't know yet but let try using `ngrok` to see if we can get a hit back to the listener i set up.

![image](https://user-images.githubusercontent.com/69868171/168488823-152d709b-6900-4fdd-96c6-b7de8d591a0c.png)

Now back to fill the support page.

![image](https://user-images.githubusercontent.com/69868171/168488853-bbd768c8-3f21-4d0e-9ea9-799a082d7a5f.png)

Let hit the `Submit` and wait for a hit back and boom i got a hit back but no cookie.

![image](https://user-images.githubusercontent.com/69868171/168489715-931b650f-b618-409a-9368-1647103cbddb.png)

Now that is strange right let try using `ngrok` web interface which we can always access on `http://localhost:4040` if you have ngrok running to inspect traffic.

![image](https://user-images.githubusercontent.com/69868171/168489791-64b688bf-4a8f-44f9-b6a9-a70a970e6614.png)

Waiting for some min and boom we got the cookie now let go claim or flag.

```
cnVkZWZpc2g6REM4VG1zd1FldG5kZTlSUTZuNjdOS1hGdHg3VUFyTVFOelJ6SnA2ejQ0Z0NGcGpnNVVlRlJBNUN1Q2JWNnRHSHNKdlRk
```

![image](https://user-images.githubusercontent.com/69868171/168489913-b621971d-2f34-443d-8a63-ae89984a7279.png)

Now we have our flag but am still curious to know what was encoded in the cookie.

![image](https://user-images.githubusercontent.com/69868171/168490005-0979e50e-232f-48f0-b70e-4dc007b38094.png)

Ahhhh you can't guess that right.

Flag:- `CYSEC{itz__easssssszzzzzzy0876#}`


### The Examiner -- 1000 Point


![image](https://user-images.githubusercontent.com/69868171/168490455-f24da123-28db-4a43-99cc-42f4de8dd473.png)

Seems `The Examiner` web challange is hosted on THM platform let connect to our VPN to access it.

![image](https://user-images.githubusercontent.com/69868171/168491492-eaee8563-2395-4bf9-9860-52e8e5ffa7c8.png)

Deploying and we got our IP not going to waste to much time on using `nmap` to scan for full ports and we got one only which is port `5000` .

![image](https://user-images.githubusercontent.com/69868171/168491572-edf997cb-ea4d-4956-a482-574672a03bad.png)

Boom and we landed on a port scanner page let see what we have on the source code.

![image](https://user-images.githubusercontent.com/69868171/168491612-6873aadc-d9f1-46c5-bd26-d053788cd0fe.png)

Now that is the interesting part of the form let try intercepting our request using burp to undertand it more on how it works.

![image](https://user-images.githubusercontent.com/69868171/168491843-bf503b1e-4c7f-408b-8880-30fb7c0e5cd3.png)

Intercepted and send to repeater now let play with it.

![image](https://user-images.githubusercontent.com/69868171/168491868-c22963e4-26ff-4048-a24d-35882e66a408.png)

But i got `Invalid IP format detected.` i have no reason i keep swaping IP and Port but the same error now seems we already have the source code and our craft code i decided to change the `Content-Type` to `application/json` back in crafting our command.

![image](https://user-images.githubusercontent.com/69868171/168492285-c95c426a-b974-424a-a70d-49c85870342d.png)

Boom and we got it time to get a reverse shell back to our terminal.

![image](https://user-images.githubusercontent.com/69868171/168492426-37003502-e6d6-48a1-9228-1b4ac9ac1132.png)

Listener ready.

![image](https://user-images.githubusercontent.com/69868171/168492455-702f1566-b39c-45c8-ab32-a4f681b4036c.png)

One liner reverse shell payload to execute ready also let hit and we should have a reverse shell back to our terminal.

![image](https://user-images.githubusercontent.com/69868171/168492491-1ad3aa83-942b-4ce6-a957-52fa82d00dea.png)

Now spawning a full TTY shell to make our shell more stable and strong.

![image](https://user-images.githubusercontent.com/69868171/168492716-11fb1990-0955-4082-8c08-4ccf9b88f596.png)

Now it ready time for more enumeration.

![image](https://user-images.githubusercontent.com/69868171/168492807-0f7c6734-e5ad-48da-9227-035f30af8305.png)

User `Rudefish` can run `sudo` let hit gtfobins.

![image](https://user-images.githubusercontent.com/69868171/168492845-8f8b3ee0-be26-48ef-a038-2bb173cf4863.png)

Running that and we are root time to hunt for the flag.

![image](https://user-images.githubusercontent.com/69868171/168493029-344353e6-0561-4c10-907a-5525186f7a5c.png)

Getting into root directory seems we have no flag in it but seems we have the `.bash_history` which store the history of a user command let see what we have in it.

![image](https://user-images.githubusercontent.com/69868171/168493412-77eb7932-93d0-4e17-9c44-417d4c090f3e.png)


![image](https://user-images.githubusercontent.com/69868171/168493375-20b2b21e-62cc-4e40-b6e6-c3a35eca056c.png)

We can see the flag being remove in the `.bash_history` i try using the `find` command to see if the flag is hidden elsewhere but no luck so let check what port is running locally.

![image](https://user-images.githubusercontent.com/69868171/168494114-3c9ac44f-c003-457c-8df7-6ce6a42079a6.png)

Seems we have `SSH` port on which is cool seems we re root we can easily change any user password and have access to it also we can easily change to any  user without a password which can be done with `su sshuser` .

![image](https://user-images.githubusercontent.com/69868171/168494385-ffc351e8-614c-4e21-b40d-46e1d63ba69a.png)

Now we can easily generate a SSH private key for the user `sshuser` so we can easily access SSH with it.

![image](https://user-images.githubusercontent.com/69868171/168494991-38a0d2ec-87b2-42c0-800e-90328cb987b5.png)

```
ssh-keygen -t rsa
```

We have our private key and also added the `authorized_keys` now let try to access SSH.

![image](https://user-images.githubusercontent.com/69868171/168495070-f26c5fb1-e4c7-4fa1-8b94-2d9b2dfb1cf9.png)

Boom we have the flag and we are done.

Flag:- `CYSEC{y0u_scaNNed_ME!}`
