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
