---
header:
    teaser: /assets/images/TryHackMe/LFI/LFIIcon.png
title: "TryHackMe Walkthrough: LFI"
date: 2020-11-30 21:24
categories: [Walkthroughs]
class: wide
layout: single
---

# Local File Inclusions

As the headline implies this box exploits a vulnerability around Local File Inclusions, this vulnerability is commonly found on web servers and is exploited when user input contains a certain path to the file that is stored on the server. 

To test for LFI we need to see a parameter on a URL or another input fields within it. e.g. 

    https://somesite.com/home?page=robots.txt 

In this exmaple, 'page' is the parameter and anything after the '=' is the value we are passing.
With a brief explanation out of the way I'll get into the box, it's easier to show LFI in action!

# Enumeration 

As we know from the topic of the room we are looking for an LFI vulnerability as such I won't perform the normal recon on this box that would normally be ran. 

## Website

We're made aware that there is a site running over port 80, so navigating out to the site we are met with a home page of an art shop... splendid.
![artshop](/assets/images/TryHackMe/LFI/homepage.png)

Navigating around the site a bit I notice that the URL changes and looks vaguely similar to the example provided above, the easiest way to probe for LFI is attempting to perform directory traversal, to perform a directory traversal add a few ../../ into the URL request and see if you can navigate to some files that you shouldn't be able to, a prime exmaple is ../../../etc/passwd

![/etc/passwd](/assets/images/TryHackMe/LFI/passwdurl.png)

This is indeed vulnerable to LFI and we can see that the passwd file is returned at the bottom of the page. This looks a bit messy so I decided to open up burp suite and send the request to repeater to mess with this further.

![Burppasswd](/assets/images/TryHackMe/LFI/burppasswd.png)

Bingo. Let's see if we can read the shadow file as this will give us the hashes for these users, and we can! We could take the hashes for the users and try to decrypt this however, I chose a different approach for this box. I decided to navigate to the path /home/falcon/.ssh/id_rsa - for those that may be new to linux this file location is where the private keys are stored, so you guessed it. We are given the private key in the response.

![privatekey](/assets/images/TryHackMe/LFI/privatekey.png)

I decided to copy the private key and save this is as file on my local machine, then I used this file when connecting as the user 'falcon' as a SSH private key:

    ssh -i /root/TryHackMe/falcon falcon@10.10.40.55

Magic, that logged me in as the user falcon, I grabbed the user flag and looked towards escalation.

## Escalation

From here on out I have a low level user, let's see how to get further. One of the first few commands I like to run when I get a shell is 

    sudo -l

This command will return anything that the user can run as root without the need for a password, this can be a quick and easy way to commonly get a quick win. In this room we can see the following

![sudo-l](/assets/images/TryHackMe/LFI/sudol.png)

A quick search of GTFOBins may reveal how we can exploit this - GTFOBins is a great resource! Highly recommended. Searching for our result in GTFOBins gives us the information that this binary (journalctl) doesn't always return to the lower level user when exiting, perfect.

Running the command

    sudo journalctl

Swiftly followed by entering !/bin/bash resulted in the following

![root](/assets/images/TryHackMe/LFI/root.png)

Perfect. That's this box rooted. Thanks for reading, hopefully you learned about LFI!