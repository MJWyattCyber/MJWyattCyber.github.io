---
header:
    teaser: /assets/images/TryHackMe/SimpleCTF/Banner.png/
title: "TryHackMe Walkthrough: Simple CTF"
date: 2020-11-15 20:57
categories: [Walkthroughs]
class: wide
layout: single
---

This post will be focusing on a box that can be found on the TryHackMe platform. TryHackMe is a great starting point for anyone looking to get grounded with the basics of CTF and ethical hacking. Simple CTF is a linux box that is rated as 'easy difficulty' so without further ado, let's deploy the machine and make a start. 

# Initial Scan

Firstly we need to get ourselves connected to the TryHackMe OpenVPN - this is covered in detail on the website so I won't go into detail here - once connected let's get an nmap scan going and go put the kettle on whilst we wait for the results!

The results are in!
![nmap](/assets/images/TryHackMe/SimpleCTF/nmap.png)

## Open Ports

It looks like there is very little open on this machine, only 3 ports being returned;

1. Port 21; FTP
2. Port 80; Web Server
3. Port 2222; SSH running on port 2222

Let's get started with the enumeration!

# Enumeration

Starting off with FTP we can see from the nmap results that anonymous login is permitted which makes ftp worth checking out! 

## ftp - Port 21

We can login using anonymous and can have a look around, it doesn't appear there is much here, however there is one directory. Navigating into the directory and using wget to pull down the file to our own machine we can see a note 'ForMitch'. This is potentially a giveaway of our username of Mitch, not only that but the message also hints towards potential password re-use and that this user uses an easy to crack password - worth keeping in mind.

![ftp](/assets/images/TryHackMe/SimpleCTF/ftp.png)

This seems as far as we can go here, our permissions are limited and there isn't much else on show.

## WebServer - Port 80

Navigating out to the IP address in a browser returns a default apache webpage, whilst this isn't ideal to be on display it doesn't appear that there is a whole lot going here - let's put this through a directory busting tool such as [dirsearch](https://github.com/maurosoria/dirsearch). This quickly returned a directory called 'simple' which we took a look at.

![simplecms](/assets/images/TryHackMe/SimpleCTF/simplecms.png)

This page is interesting, there is a lot on offer here - particularly versions of software. It seems this webserver is running some kind of 'CMS Made Simple software'. If we scroll to the bottom of the page we can see the version of the software that is being ran, which is 2.2.8. Whenever we find a version it's always worth throwing it at google to see if any exploits are known. In this case it seems there are a few results, in particular a CVE which can be found [here](https://www.exploit-db.com/exploits/46635). As we've enumerated most our options and found a result lets move onto exploitation.

# Exploitation

## CVE-2019-9053

This CVE seems to apply to the software version we have found so let's take a deeper dive. Looking at the the first few lines here we can see we're asked to supply a url, wordlist and a crack option. I've prepared a passwords.txt file to use which is sourced from the github page [here](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Common-Credentials). Time to run the exploit:

![exploit](/assets/images/TryHackMe/SimpleCTF/exploit.png)

This took a few minutes but eventually returned the following:

![results](/assets/images/TryHackMe/SimpleCTF/exploitresults.png)

Great! Our the exploit worked and I now have a username and password combination, let's try to plug this information into a SSH connection.

## SSH

With a username and password we were able to login as our user, sadly this is just a low level user so let's grab our user flag for the box and then take a look at what commands we can run by utilising 'sudo -l'. 

![sudo](/assets/images/TryHackMe/SimpleCTF/sudo.png)

I found out that the user can run VIM without the need to enter the root/sudo password, excellent this may be an avenue to exploit. Utilising [GTFOBins](https://gtfobins.github.io/) I searched for VIM > Sudo, I can see I can enter the command:

    sudo vim -c ':!/bin/sh'

The console can get a bit messy here but if you ever appear to be in VIM then simply type :q! and hit enter which should drop you out to a shell, in my case this worked and running 'whoami' after returning to my shell it returned I was root! To finish up we can navigate to /root/ and grab the root flag. 

![rootflag](/assets/images/TryHackMe/SimpleCTF/rootflag.png)

Thanks for reading!