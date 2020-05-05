---
title: Legacy
date: 2020-05-04 19:16:00
description: Hack The Box Writeup
featured_image: '/images/blog/legacy/legacy.jpg'
tags: ['Hack The Box']
---

<p>Legacy is the second machine on Hack The Box and aimed for beginners as it only requires one exploit to obtain root access.</p>

<h2>Hack the box</h2>

<ul>
	<li>Operating System: Windows</li>
	<li>Difficulty: Easy</li>
</ul>

<h2>Information gathering</h2>

<p>Let's start with enumeration by using nmap. As always, I like to use:</p>


<img src="/images/blog/general/nmapscan.jpeg" alt="nmap scan">


<p>Once the nmap scan completes, we can take a look at the results:</p>


<img src="/images/blog/legacy/nmapresults.jpg" alt="nmap results">


<p>It looks like we have port 139 and 445 open, and based on the nmap scan, we might have also found the SMB OS, <i>Windows XP</i>. Before we exploit this box, let's verify that the OS is the right one by using Metasploit's auxiliary module to find out the SMB version.</p>

<p>To open up Meatsploit in Terminal, type in:</p>

<code>msfconsole</code>

Once Metasploit opens, we can use Metasploit's auxiliary module, smb_version. There's a lot of them auxiliary modules, so if you're not sure which one to use, we can search for it by typing in Metasploit: 

<code>search smb_version</code>


<img src="/images/blog/legacy/smbversion.jpg" alt="smb version">


<p>From the search results, we can see that there is an auxiliary module called smb_version. To use it, we can type in Metasploit:</p>

<code>use auxiliary/scanner/smb/smb_version</code>

<p>As always, I like to see what options I have when I use either an auxiliary or exploit module by typing in <code>options</code>. After taking a look at the options, I set the following: </p>

<ul>
	<li><code>rhosts</code>: The IP address of the Legacy box.</li>
	<li><code>target</code>: To see if there are any targets to set--in this case, there is none.</li>
	<li><code>run</code>: To run the auxiliary module.</li>
</ul>


<img src="/images/blog/legacy/auxiliary.jpg" alt="auxiliary scan">


<p>Perfect! The results show that the box is currently running <i>Windows XP SP3</i>. Now that we have confirmed the OS that the SMB is on, let's try to see if we can find any existing exploits.</p>

<h2>Threat modelling and vulnerability analysis</h2>

<p>In the last box (Lame), we used Metasploit's search feature to find an exploit, this time to make it interesting, we can use Google search. I normally like to look for results from Rapid7 (which is the same as Metasploit) or Exploit-db.</p>

<p>After searching <i>Windows XP SP3 exploit</i> in Google search, I came across this one from Rapid 7:</p>

<a href="https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi">MS08-067 Microsoft Server Service Relative Path Stack Corruption</a>

<p>Here's what it says on the exploit's description:</p>

<p><i>This module exploits a parsing flaw in the path canonicalization code of NetAPI32.dll through the Server Service. This module is capable of bypassing NX on some operating systems/service packs.</i></p>

<p>This exploit looks promising, so let's try using it.</p>

<h2>Exploitation</h2>

<p>To use the exploit that we found in the threat modelling and vulnerability analysis phase, we can follow the instructions found in the above link and type in:</p>

<code>use exploit/windows/smb/ms08_067_netapi</code>

<p>Again, let's take a look at our options by typing in <code>options</code>.</p> 


<img src="/images/blog/legacy/exploit.jpg" alt="nmap scan">

<p>In this case, I set the rhost to the IP address of the box, 10.10.10.4, by typing in:</p>

<code>set rhosts 10.10.10.4</code>

<p>And that's it, let's try running the exploit!</p>

<img src="/images/blog/legacy/runexploit.jpg" alt="exploit results">

<p>It worked! We now have a Meterpreter session. Meaning, we have access to the box. We can type in:

<ul>
	<li><code>getuid</code>: To see what access level we have.</li>
	<li><code>pwd</code>: To check which directory we're currently in.</li>
</ul> 

<p>Since we have root access, we can type in <code>shell</code> to that we have access to the command line and locate the root.txt and user.txt file.</p>

<p>And that's it! We've now completed the Legacy box.</p>

