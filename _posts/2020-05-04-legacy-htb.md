---
title: Legacy
date: 2020-05-04 19:16:00
description: Hack The Box Walkthrough
featured_image: '/images/blog/legacy/legacy.jpg'
tags: ['Hack The Box']
---

Legacy is also one of the first machines released on Hack The Box (HTB) and aimed for beginners. It is a Windows box that is vulnerable to SMB bugs and I'll be using Metasploit to exploit them.

## Hack the box
 
* Operating System: Windows
* Difficulty: Easy


## Information gathering

Let's start with enumeration by using nmap. As always, I like to use:

<img src="/images/blog/legacy/ipaddress.jpg" alt="nmap scan">

Once the nmap scan completes, we can take a look at the results:

<img src="/images/blog/legacy/nmapresults1.jpg" alt="nmap results">

It looks like we have port 139 and 445 open, and based on the nmap scan, we might have also found the SMB OS, *Windows XP*. 

<img src="/images/blog/legacy/nmapresults2.jpg" alt="nmap results">

Before we exploit this box, let's verify that the OS is the right one by using one of Metasploit's auxiliary modules.

To open up Meatsploit whilst in the command line interface, type in:

`msfconsole`

Once Metasploit opens, we can use Metasploit's auxiliary module, smb_version. There are a lot of auxiliary modules, so if you're not sure which one to use, we can search for it by typing in Metasploit: 

`search smb_version`

<img src="/images/blog/legacy/smbversion.jpg" alt="smb version">

From the search results, we can see that there is an auxiliary module called smb_version. To use it, we can type in Metasploit:

`use auxiliary/scanner/smb/smb_version`

As always, I like to see what options I have when I use a Metasploit module by typing in `options`. After taking a look at the options, I set the following: 

* `rhosts`: The IP address of the Legacy box.
* `target`: To see if there are any targets to set--in this case, there is none.
* `run`: To run the auxiliary module.

<img src="/images/blog/legacy/auxiliary.jpg" alt="auxiliary scan">

Perfect! Based on the results above, the box is currently running *Windows XP SP3*. Now that we have confirmed the OS that the SMB is on, let's look for any existing exploits.


## Threat modelling

In the last box (Lame), we used Metasploit's search feature to find an exploit. This time to make it interesting, we can use Google search. I normally like to look for results from Rapid7 (which is the same as Metasploit) or Exploit-db.

After searching <i>Windows XP SP3 exploit</i> in Google search, I came across this one from Rapid 7:

<a href="https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi">MS08-067 Microsoft Server Service Relative Path Stack Corruption</a>

Here's what it says on the exploit's description:

<i>This module exploits a parsing flaw in the path canonicalization code of NetAPI32.dll through the Server Service. This module is capable of bypassing NX on some operating systems/service packs.</i>

This exploit looks promising, so let's try using it.


## Exploitation

To use the exploit that we found, we can follow the instructions found in the above link and type in:

`use exploit/windows/smb/ms08_067_netapi`

Again, let's take a look at our options by typing in `options`. 


<img src="/images/blog/legacy/exploit.jpg" alt="preparing exploit">

In this case, I set the rhost to the IP address of the box, 10.10.10.4, by typing in:

`set rhosts 10.10.10.4`

And that's it, let's try running the exploit!

<img src="/images/blog/legacy/runexploit.jpg" alt="exploit results">

It worked! We now have a Meterpreter session. Meaning, we have access to the box. We can type in:


* `getuid`: To see what access level we have.
* `pwd`: To check which directory we're currently in.
 

Since we have root access, we can type in `shell` to that we have access to the command line and locate the root.txt and user.txt file.

And that's it! We've now completed the Legacy box.


