---
title: Nibbles
date: 2020-06-24 21:58
description: HTB Walkthrough without Metasploit
featured_image: '/images/blog/nibbles/nibbles.jpg'
categories: htb
---

Nibbles is a Hack the Box (HTB) machine that hosts a vulnerable instance of nibbleblog. We’ll be doing a lot of enumeration to gain information about the box before we attack it—all without the use of Metasploit!

## Hack The Box

* Operating System: Linux
* Difficulty: Easy

## Enumeration 

I changed things up a bit this time around with the nmap scan. 

<img src="/images/blog/nibbles/nmap.jpg" alt="nmap scan">

	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
	|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
	|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
	80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
	|_http-server-header: Apache/2.4.18 (Ubuntu)
	|_http-title: Site doesn't have a title (text/html).
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

From the results we can see that port 22 and port 80 are open, so let's begin enumerating. 

### Website Information

The first thing we can do is open the web browser and access `10.10.10.75`

<img src="/images/blog/nibbles/front.png" alt="front page of target site">

Not much here.. but if we look at the source code, it guides us to a /nibbleblog directory so we can try to head there next. 

<img src="/images/blog/nibbles/code.png" alt="target code">

We can access the /nibbleblog directory by adding it to the target IP address. It should look something like `10.10.10.75/nibbleblog/`

<img src="/images/blog/nibbles/blog.png" alt="nibbleblog website">

Interesting. It looks like it's a blogging system hosted by [nibbleblog](https://www.nibbleblog.com/).. great! Let's continue on with our enumeration phase. 

### Dirb

Next, we'll use Dirb. Dirb is a web content scanner that looks for existing and/or hidden directories so it will help us do some more digging. Dirb should already be preinstalled on the Kali system too. 

To use Dirb, we can enter the following into the Kali terminal:

	dirb http://10.10.10.75/nibbleblog/

And here are our results:

<img src="/images/blog/nibbles/dirb.png" alt="dirb results">

As we're going through the directories that Dirb found, we can see that there's an `admin.php` directory and if we head there, it looks like a login page. The .php extension also tells us that the target can run .php files.

<img src="/images/blog/nibbles/admin.png" alt="admin page for nibbleblog">

Let's take a look at the other directories to see if there's anything else that might be interesting—and there was! There's a **user.xml** file that displays the list of users. One of which is called "admin" so we can try using this username to login to the admin page. 

<img src="/images/blog/nibbles/users.png" alt="dirb results">

We could try guessing the password based on what we know, luckily "nibbles" worked! 

### Nibbleblog Admin

Now that we're logged in as admin, we can browse a bit more to see if there's anything that we can use. Taking a look, we can see that this Nibbleblog is running on version 4.0.3. Since we know the version number, we can see if there are any existing exploits... and there is: [CVE-2015-6967](https://nvd.nist.gov/vuln/detail/CVE-2015-6967)

> Unrestricted file upload vulnerability in the My Image plugin in Nibbleblog before 4.0.5 allows remote administrators to execute arbitrary code by uploading a file with an executable extension, then accessing it in content/private/plugins/my_image/image.php.

Oo, and if we go back into Nibbleblog, we see that we have access to the My Image plugin, so let's try this exploit! 

<img src="/images/blog/nibbles/image.png" alt="dirb results">

## Exploitation

Before we start exploiting the box, let’s recap what we found during the enumeration phase:

* We found out that port 80 was open and it was a blog running on Nibbleblog.
* From Dirb, we found the login screen, users, and verified the target can run .php files. 
* Because we found out the user "admin," we were able to guess the password.
* Once we logged in, we found out the version number, which helped locate an exploit.

Alright, now that we've recapped what we know, let's move on to the exploitation phase. 

### Gaining Reverse Shell

Since we know that the platform was built on PHP (because of the .php files), we can use [Pentest Monkey's](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) PHP reverse shell script to gain access to the machine. Here are the steps to do it: 

1. Download the PHP reverse shell file. 
2. Open the file and change the IP address to the one in Kali. 
3. Edit the file name (if you want) to image.php. 
4. Upload the file into the My Image plugin section. 

Once the .php file is uploaded, we can create a Netcat listener in Kali: 

	nc -lnvp 1234

Then we can go to the following URL to run the reverse shell that we uploaded: 10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php

<img src="/images/blog/nibbles/reverse.png" alt="reverse shell results">

Perfect! We now have shell access. If we use the `whoami` command, we're currently logged in as **nibbles**—but we want root access, so let's move onto the next phase, privilege escalation.

### Privilege Escalation

Whenever you have shell access, it's always a good habit to see what type of privileges we currently have by typing in `sudo -l`. 

<img src="/images/blog/nibbles/priv.png" alt="nibbles' privileges">

Interestingly, we can see that nibbler can run the following **monitor.sh** bash script without being a root user. So we can probably edit this script so that it would escalate our privileges to root. 

Before doing so, we can try to see what's inside the file by typing:

	cat home/nibbler/personal/stuff/monitor.sh

.. but nothing was there. Meaning, we can create one ourselves. Here are the steps to do it: 

1. Go to home/nibbler/ and create the directories /personal/stuff/ 
2. Create a monitor.sh file: `echo "#!/bin/bash\nbash" > monitor.sh`
3. Change the permission of the file: `chmod +x monitor.sh`
4. Run the script: `sudo ./monitor.sh`

<img src="/images/blog/nibbles/root.png" alt="root access">

Amazing! We did it. We now have root access. 

## Conclusion

This was honestly pretty easy and really shows how important it is to enforce the use of strong passwords. If we weren’t able to easily guess the password, we would not have been able to gain administrative privileges, which led us to obtain root privileges—also, please patch your software!