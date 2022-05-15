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

