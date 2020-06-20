---
title: Injection
date: 2020-05-29 21:58
description: One of the OWASP Top 10 Web Apps Security Risks
featured_image: '/images/blog/injection/injection.png'
categories: owasp
---

As a part of the web application series, I will be using Juice Shop. Juice Shop is an unsecure web application created by Bjorn Kimminich. It contains OWASP's Top Ten vulnerabilities and many other security flaws found in real-world applications.

You can read more about OWASP Juice Shop here: [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)

In this post we'll be going through one of the OWASP Top Ten vulnerabilities, Injection, using Juice Shop to show what it looks like and how we can defend our web applications against these types of attacks.

## Introduction 

Before we start, OWASP has a great article that goes over Injection: [OWASP: Injection](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A1-Injection)

Essentially, Injection is when an attacker can send data to an interpreter to gain sensitive information. An example of an Injection attack is SQL injection, which is what we’ll be using in Juice Shop.

## The Setup 

First, we'll need to install Juice Shop. Here's GitHub link that will show you how to install it:

[OWASP Juice Shop](https://github.com/bkimminich/juice-shop#from-sources)

We'll also need Burp Suite which should already be installed in Kali. Burp Suite is a web application security software. There is a community edition that's available for free, or you can pay for the Pro edition which gives you more features. In this scenario, I'll be using the community edition: 

[Burp Suite by PortSwigger](https://portswigger.net/burp)

Lastly, here's the Gitbook that has a list of Injection challenges that we'll be completing. There's also some helpful tips in each challenge that Bjorn mentions. 

[Injection Challenges - Juice Shop](https://bkimminich.gitbooks.io/pwning-owasp-juice-shop/part2/injection.html)

Once you have all of these tools ready, we can get started. 

## Two Star Challenges

### Login Admin

 * Description: Log in with the administrator's user account

The first thing we can do is attempt to login to Juice Shop while having Burp Suite open. We can enter in something random for our initial login attempt: 

* username: admin
* password: password

Since Burp Suite was open, we can collect the HTTP Request in the Proxy tab, and if we right-click, we can send it to the Repeater.

<img src="/images/blog/injection/admin.jpg" alt="burp suite admin login">

Now, if we head into the Repeater tab and press the Send button, we get a response that says authentication denied. Let's see if we can add SQL to it to bypass the login authentication to gain admin access.

So instead of "admin" let's type in "admin' or 1=1--" in the Request section.

<img src="/images/blog/injection/adminrequest.jpg" alt="submitting a request through burp suite">

Now, if we press send... it worked! We now have access to the admin account. We can also see the admin's email address **admin@juice-sh.op** at the bottom of the Response section too which may come in handy later. 

Why did this work? Well, by adding: 

* `'` = it allows us to inject SQL commands. 
* `or 1=1` = means or True. 
* `--` = will comment out the rest of the code.

In other words, because the username is defined as *True* and it ignored the password field for authentication, we were allowed to login in as admin.

## Three Star Challenges

### Login Bender/Jim 

* Description: Log in with Bender's (or Jim's) user account

*FYI, the steps below work for either Bender or Jim's account, but for example purposes, I will be attempting to login to Bender's account.*

One of the tips that Bjorn mentions is that the email address is similar to the admin's email address (admin@juice-sh.op). So, it's possible that Bender's email address is bender@juice-sh.op.

We can follow the similar steps in the Login Admin challenge, and edit the username field in the Request section, but this time add `'--` at the end, so that it looks like: 

`"bender@juice-sh.op--"`

<img src="/images/blog/injection/bender.jpg" alt="login attempt for benders account">

And if we press the Send button again, it worked! We get a success message back in the Response section, so now we also have access to Bender's account. If you want, you can repeat the same steps to gain access to Jim's account too.

### Christmas Special 

* Description: Order the Christmas special offer of 2014

Since the Christmas special is hidden, we can try to search for it within the database. But.. we have to find it first, so let's see if we have any luck with the search bar while running Burp Suite. 

When we search for something in Juice Shop, we see the HTTP Request in Burp Suite: 

	GET /rest/products/search?q= 

If we right-click, send this to the Reapeter, add a `'` after the `q=` and press send, we can see the Response:

	{"status":"success","data":[]}

A secure web application would not have provided us with a success status, but because it did, we can try adding additional SQL to complete this challenge.

So if we enter in `'))--` we might be able to get a response that shows us everything within the database... and it did! Why? Because we're changing the SQL query to say: 

"SELECT * FROM Products WHERE ((name LIKE '%**'))—-**

Meaning, `'))` closes the original query and `--` commented out the rest of it. 

This is why the response gave us a list of all the products listed in the database. 

Now that we can see all of the products, we can search for the word "Christmas" to locate the Christmas special. Note that the ID number is 10. 

<img src="/images/blog/injection/christmas.jpg" alt="searching for christmas special">

If we go back into the Juice Shop and add a random item, like an Apple, to the cart but change the ID number in Burp Suite to 10, we might be able to replace the Apple with the Christmas special instead. 

<img src="/images/blog/injection/productid.png" alt="searching for christmas special">

Look at that! We now have the Christmas special added to our shopping basket.

<img src="/images/blog/injection/basket.png" alt="added christmas special to basket">

### Database Schema

* Description: Exfiltrate the entire DB schema definition via SQL Injection

In the Christmas special, we found out that we can add parameters after the `q`, and if we play around with the parameters and entered something where the output would produce errors, the output would also display that we are using an SQL database called SQLite.

SQLite uses a database schema that is stored in a table called sqlite_master.

> Every SQLite database has an SQLITE_MASTER table that defines the schema for the database.


Read more about SQLite here: [SQLite FAQs](https://www.sqlite.org/faq.html#q7)

Meaning, if we edited the Request to `')) UNION SELECT * FROM sqlite_master--` and sent the Request in Burp Suite, the Response area tells us that there is an error:

<img src="/images/blog/injection/dbschema.png" alt="SQL error">

This means that to get a successful response back, we need to enter the same number of columns in the database. Let's give this a try. 

We can start adding columns one by one, spoiler alert.. the answer is going to be 9, so by the time you've reached 9, it should look something like this: 

	')) UNION SELECT '1', '2', '3', '4', '5', '6', '7', '8', '9' FROM sqlite_master--

And in the Response area, we should get a successful query. 

<img src="/images/blog/injection/success.png" alt="success attempt to access database">

Next, we'll want to remove the unwanted product results by adding `qwert` and change the `1` value with `sql` so that we get the original table. The final result should look something like this: 

	qwert')) UNION SELECT sql, '2', '3', '4', '5', '6', '7', '8', '9' FROM sqlite_master--

> Why sql? The sql field is the text of the original CREATE TABLE or CREATE INDEX statement that created the table or index.

And if we press the Send button in Burp Suite, we should get the original table.. and we did! 

<img src="/images/blog/injection/finaldbschema.png" alt="completed dbschema challenge">

Another challenge completed. 

## Four Star Challenges

### User Credentials

* Description: Retrieve a list of all user credentials via SQL Injection.

This challenge is very similar to how attackers can gain sensitive information from a database if the sytem is vulnerable to SQL injection. 

In the Database Schema challenge, we found out that the `q` paramter is suspectable to SQL injection. As a result, we can use a similar payload but alter it slightly so that we're pulling data from the Users table: 

	qwert')) UNION SELECT sql, '2', '3', '4', '5', '6', '7', '8', '9' FROM Users--

Next, we'll want to replace the fixed values with the correct column names. We can either guess them or remember them from previous SQL errors when we were attacking the Login form. 

	qwert')) UNION SELECT id, email, password, '4', '5', '6', '7', '8', '9' FROM Users--

Once we click the Send button, we should get the response: 

<img src="/images/blog/injection/userlist.jpg" alt="completed dbschema challenge">

There we go! We did it, a list of all the users and their passwords.


