---
title: OSCP Cheat Sheet
date: 2020-06-21 15:30:00
description: Most commonly used commands to help with the OSCP certification.
featured_image: '/images/blog/oscp/oscp.jpg'
---

As I’m studying for the OSCP certification exam, I decided to write all of the commands and techniques I have used during PWK labs and similar machines. This list is still a work in progress, so if you have any methods or tips that are missing from this list, please let me know!

## Enumeration

In the enumeration section, you'll find information on the different ways to enumerate a service. 

### DNS (53/tcp)

The Domain Name System (DNS) is responsible for translating domain names into IP addresses. By using DNSRecon, we can automate DNS enumeration.

	dnsrecon -d <TARGET>

### SMB (139/tcp and 445/tcp)

The Server Message Block (SMB) is a protocol used to share resources like files, printers, etc. There are a number of tools that we can use to complete SMB enumeration.

**nmblookup**

The nmblookup tool collects NetBIOS over TCP/IP client used to lookup NetBIOS names. The `-A` option allows the ability to lookup a specific IP address.

	nmblookup -A <TARGET>

**nmap scripts**

Nmap contains many useful NSE scripts, for SMB there are: 
* `smb-enum-shares` = finds open SMB shares.
* `smb-os-discovery` = determine OS of SMB service.
* `smb-enum-users` = enumerate the SMB users.

To use nmap scripts, enter into Kali terminal:

	nmap -v -p 139,445 --script=<SCRIPT> <TARGET>

### NFS (111/tcp)

The Network File System (NFS), is a distributed file system protocol that allows users on a computer to access files over a network. 

If you find any NFS-related services, we can enumerate them by using NSE scripts. The `*` option acts as a wildcard, so it will allow us to run multiple nfs scripts. 

	nmap -p 111 --script nfs* <TARGET>

If there are any NFS shares, mount them by using the `mount` command along with `-o no lock` to disable file locking. It's also a good habit to check if you can read/write files or change the permissions too by creating a new user with a certain UID.

### SMPT (25/tcp)

The Simple Mail Transport Protocol (SMTP) supports *VRFY* and *EXPN*. They can be used to verify existing users on a mail server. Here's an example: 

	nc 10.11.1.217 25
	[...]
	VRFY root
	252 2.0.0 root
	VRFY idontexist
	550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient table

### SNMP (161/udp)

The Simple Network Management Protocol (SNMP) is based on UDP, and is suspectible to IP spoofing and replay attacks. To scan for SNMP ports, we can use the `nmap` tool. The `-sU` option is used to perform UDP scanning and the `--open` will only display open ports.

	sudo nmap -sU --open -p 161 <TARGET> 

