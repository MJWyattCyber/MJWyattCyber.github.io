---
header:
    teaser: /assets/images/Legacy/legacycard.png/
title: "HTB Walkthrough: Legacy"
date: 2020-06-30 22:08
categories: [Walkthroughs]
class: wide
layout: single
---

This is going to be one of many in a series of writeups looking into a list of machines on the Hack The Box platform which have been included on the following [list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#). This list is 100% community provided but is regarded as being as accurate as can be, the list is designed to give you a feel for machines that are easier than and similar to the OSCP certification. 

The box featured in this walk through is the retired box called Legacy @ 10.10.10.4, this can be accessed by users that hold a VIP subscription to the service.

# Initial scanning

To begin scanning this machine to see what was open I performed a scan with nmap;

    nmap -Pn -A -T4 10.10.10.4

This performs an aggressive scan which returns alot of information regarding the services and versions that are running on the system, the results for this scan are as follows:

    matt@Kali:~/HTB/Legacy$ nmap -Pn -A -T4 10.10.10.4
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-30 15:36 BST
    Nmap scan report for 10.10.10.4
    Host is up (0.014s latency).
    Not shown: 997 filtered ports
    PORT     STATE  SERVICE       VERSION
    139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
    445/tcp  open   microsoft-ds  Windows XP microsoft-ds
    3389/tcp closed ms-wbt-server
    Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

    Host script results:
    |_clock-skew: mean: 5d00h27m38s, deviation: 2h07m16s, median: 4d22h57m38s
    |_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:9d:cb (VMware)
    | smb-os-discovery: 
    |   OS: Windows XP (Windows 2000 LAN Manager)
    |   OS CPE: cpe:/o:microsoft:windows_xp::-
    |   Computer name: legacy
    |   NetBIOS computer name: LEGACY\x00
    |   Workgroup: HTB\x00
    |_  System time: 2020-07-05T19:34:40+03:00
    | smb-security-mode: 
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    |_smb2-time: Protocol negotiation failed (SMB2)

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 62.13 seconds

This only really gave me one option to explore here in SMB which operates over ports 139 and 445. 

# Enumeration

## SMB (139, 445)

As I mentioned earlier this box really only had the one option here to explore which was SMB, to get started I like to see if I can list the SMB Shares out to see what we're working with, this is done with:

    smbclient -L \\\\10.10.10.4\\

Unfortunately this returned no results so there wasn't much further I could go here. Going back to my nmap scan I could see that the potential operating system is Windows XP, searching for 'SMB windows xp exploit' returned the following [link](https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi) to a rapid7 post. The good thing about rapid 7 posts is that it commonly means there is a metasploit plugin to run the exploit.

# Exploitation

As mentioned in the section above we now have a potential exploit to be carried out, there is the option to run this manually however that's beyond my current understanding. As a result I will be using metasploit, particularaly the module following: /exploit/windows/smb/ms08_067_netapi/

Configuring the options here once the module is selected:

![msfconsoleoptions](/assets/images/Legacy/metasploitoptions.png)

Here I am setting the target host and confirming the options are set before execution. 

![msfexploitation](/assets/images/Legacy/exploit.png)

Running this returned a shell first time! As this is a meterpreter shell which is built into metasploit we can use 'getuid' to see what level of access we have.

![accesslevel](/assets/images/Legacy/accesslevel.png)

Awesome! We're already Authority system which are the keys to the castle! From here we can grab both the user flags and the root flags.

## User Flag

This flag is located at C:\Documents and Settings\john\Desktop\users.txt

![userflag](/assets/images/Legacy/userflag.png)

## Root Flag

This flag is located at C:\Documents and Settings\Administrator\Desktop\root.txt

![rootflag](/assets/images/Legacy/rootflag.png)

# Summary

That's it for this box! This is the probably the easiest box on the platform, a very nice and simple SMB exploit here. The root cause of the exploit being linked to the operating system being way end of life at this point. A combination of this and old protocols resulted in this being possible.