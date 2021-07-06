---
header:
    teaser:
title: "Buffer Overflow Basics"
layout: single
classes: wide
date: 2021-06-30 17:53
categories: [Ethical Hacking]
---
# Introduction

After learning how to perform basic buffer overflow attacks I decided to break down all the scripts used throughout the process so that I had one script per 'stage', whilst it doesn't cover the complete process it does help me work through basic BoF attacks. I wrote these scripts out and placed them in the following github repository, ![here](https://github.com/MJWyattCyber/Buffer-Overflows). The original 'body' of the scripts have been taken from the course, 'Practical Ethical Hacking' authored by 'TheCyberMentor' which is awesome. I have simply added a few more extra variables to make them repeatable to other situations and added some comments here and there for clarity.

This code was tested against a local instance of 'VulnServer' mileage may vary with other vulnerable applications but hopefully the methodolgy remains the same.

#  Overview

A Buffer Overflow can be broken down into 7 'stages' or steps that can be carried out, it's important to note that to carry out these attacks you will require a debugging tool that you are comfortable with, I utilised ImmunityDebugger and will make reference to it throughout. 

The stages of a buffer overflow can be summarised by:

1. Spiking the vulnerable program
2. Fuzzing the command
3. Finding the Offset
4. Overwriting the EIP
5. Finding Bad Characters
6. Finding the right module
7. Generating shellcode and executing the overflow

## Spiking the vulnerable program

With a vulnerable program identified It will need to be running and loaded/attached to your debugger of choice, make sure the application is running and attempt a connection to it via your attack box. We can utilise netcat to assist with this:

    nc -nv <IP Address> <Port>

If we're lucky we'll connect to the program and be able to issue commands, typically speaking most applications will have a 'help' command or will inform the user upon login of available commands. The goal of spiking is to work through each command in the vulnerable application to see which of the commands are vulnerable. 

A tool called **Generic TCP** can help us automate this process, this tool will utilise the **'spiking.spk'** script, it is required to place the name of the command we are spiking in the spiking script and the final command will need to be provided a few arguments:
    
    generic_send_tcp <IP Address> <Port> spiking.spk 0 0 

If the program doesn't crash then the command is not vulnerable, simply move to the next command and try again. Repeat this process through all the available commands within our vulnerable application. When the program crashes you should look at the output in the debugger application, within Immunity Debugger you can look at the top right 'pane' and you should take note of two pieces of information here:
- First, it appends '/.:/' to our command of 'TRUN '. This will form our 'vulncommand' variable in future scripts.
- Second, the EIP is followed by '41414141', 41 is hex for A this tells us we can overwrite the EIP. Perfect.

## Fuzzing the vulnerable command

This step of a Buffer Overflow is similar to Spiking, however, it simply focuses on the vulnerable command and we can utilise the script, **fuzzing.py** to help with this stage. To make this work you will need to replace the IP address, port and vulncommand variables to match your situation and application. In our example the vulnerable command was called 'TRUN' and combining this with our output from ImmunityDebugger this formulated the vulncommand = 'TRUN /.:/' Again, replace this to match your situation.

The script here can be ran and will open a connection with the target IP on the specified port, it waits until the program crashes and outputs at how many bytes it took to crash the program, we can take the number of bytes into our next stage.

## Finding the Offset

The idea in this stage is to find 'where' the EIP is, and to do that we need to generate a payload that we will be able to search to locate our EIP's position. We will be utilising **pattern-create** which is part of the metasploit frameworks and as such is preinstalled on Kali Linux. We will use the following syntax:

    /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l <output of our fuzzing script in bytes>

The integer that follows the -l switch will vary depending on the output from fuzzing.py, for example if fuzzing.py returns "The program crashed at 2700 bytes" we would provide 3000 to this command. After a few seconds a random string of characters will be generated, this can be copied and pasted into the 'offset' variable in the **Finding_the_offset.py** script.

With the variable set, and your debugger tool set up and waiting, run the script. The goal here is that the program will crash and the EIP will display a string of characters, make note of this output. Now it's time to find that string's position, to do that we will use a tool called **pattern-offset**, again, part of the metasploit framework and the syntax should look like:

    /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l <Number of bytes that caused the program to crash as used in the pattern_create command> -q <String that the EIP was displaying as>

This should return the number of bytes that this string is at, this will tell us the number of bytes we have to 'overwrite' in order to get to the EIP.

## Overwriting the EIP

Now we can utilise the **overwrite_the_EIP.py** script to confirm that we can overwrite the EIP and set this to something of our chosing. Simply enter in the variables we've set previously: IP Address, port, vulncommand and also set the number of 'A' characters we wish to send to match the number of bytes before our EIP location, in the example it was 2003. Start up your debugging tool and make sure the application is running, run the script against our target and with any luck the program should crash and the EIP should now be displaying as all 42's this is hex for the letter B thus letting us know we've successfully overwritten the EIP.

## Finding Bar Characters

This is a case of setting the variables discovered throughout and ensuring that the debugger and application is running, then run the script **finding_bad_characters.py** against the target application. This should cause the program to crash. In Immunity debugger the information we are after is in the bottom left pane, Look for the value that says 'ESP Value' and select this, right click and select 'Follow in dump', this should display a series of characters and numbers in sequence. This step will basically be looking for anything that is out of place or missing, for example, if the number 7 was missing we would make a note that 07 is a bad character. Complete this for all characters, making a note of all the bad characters identified. 