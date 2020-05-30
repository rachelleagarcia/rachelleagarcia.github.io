---
title: Injection
date: 2020-05-29 21:58
description: One of the OWASP Top 10 Web Apps Security Risks
featured_image: '/images/blog/injection/injection.png'
tags: ['OWASP']
---

As a part of the web application series, I will be using Juice Shop. Juice Shop is an insecure web application created by Bjorn Kimminich. It contains OWASP Top Ten vulnerabilities and many other security flaws found in real-world applications.

You can read more about OWASP Juice Shop here: [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)

This post will be going through One of OWASP Top Ten vulnerabilities, Injection, and we’ll be using Juice Shop to show what Injection looks like and how we can defend our web applications against these types of attacks.

## Introduction 

Before we start, what does Injection mean, and why is it a security risk?

OWASP has a great guide that goes over Injection: [OWASP: Injection](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A1-Injection)

Essentially, an Injection is when an attacker can send data to an interpreter, in order to gain sensitive information. An example of an injection vulnerability would be something like SQL–hence, SQL injection, which is what we’ll be using in Juice Shop.

## Juice Shop Challenges

Juice Shop has several challenges ranging in difficulty based on their star levels. One star being the easiest and five stars being the hardest level of difficulty. In this post, we'll be going through the Injection challenges and completing all of the star levels.

### Two Stars

**Login Admin - Log in with the administrator's user account**

The first thing we can do is try and login to Juice Shop while having BurpSuite open. We can enter in something random for our login attempt: 

* username: admin
* password: password

Since BurpSuite was open, we can collect the HTTP Request in the Proxy tab and if we right-click, we can send it to the Receiver. 

<img src="/images/blog/injection/admin.jpg" alt="burpsuite admin login">

Next, we can go to the Receiver tab and press the send button, we get authentication denied. So let's see if we can add SQL to it and bypass the login authentication to gain admin access. 

So intead of `"admin"` let's type in `"admin' or 1=1--"` in the Request section. 

<img src="/images/blog/injection/adminrequest.jpg" alt="submitting a request through burpsuite">

Now, if we press send... it worked! We now have access to the admin account. We can see that the admin email address is **admin@juice-sh.op** at the bottom of the Response section. 

Why did this work? Well, by adding: 

* `'` = it allows us to inject SQL commands. 
* `or 1=1` = means or True. 
* `--` = will comment out the rest of the code.

So in other words, because the username is defined as *True* and it ignored the password field for authentication, it allowed us login in as admin.

### Three Stars

**Login Bender/Jim - Log in with Bender's (or Jim's) user account**

*FYI, the steps below work for either Bender or Jim's account, but for example purposes I will be logging into as Bender's account.*

Since we know the admin's email address from the **Login Admin** challenge, let's try editing it so that it matches Bender's email address. For example, bender@juice-sh.op.

We can edit the username field by adding SQL (similar to what we used in the Login Admin challenge), but this time add `'--` at the end, so it would look like: 

`"bender@juice-sh.op--"`

<img src="/images/blog/injection/bender.jpg" alt="login attempt for benders account">

And if we press the Send button again, it worked! So now we also have access to Bender's account. If you want, you can go through and complete the same steps for Jim's account too.

**Christmas Special - Order the Christmas special offer of 2014**
