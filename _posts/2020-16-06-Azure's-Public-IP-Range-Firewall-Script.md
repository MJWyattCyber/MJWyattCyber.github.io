---
layout: single
title: "Azure's public IP Ranges Firewall script"
date: 2020-06-16 22:49
categories: [Scripting][Azure]
teaser: /assets/images/MsSurface.jpg
classes: wide
---

Azure's pretty big these days and also pretty common in the workplace, whether it's full cloud, hybrid cloud, 
personal use or a combination there is a plethora of services and resources available and it shows no sign of slowing down any time soon. With all of these resources available to us it's utilising an ever growing number of public IP ranges and subnets, this can prove to be a problem when it comes to providing access from an organisations stand point. How do you provide access to the resources whilst maintaining security? There's the two main contenders of a Site-to-Site VPN or an Express Route, however, these can rack up costs pretty quickly. A third option is to route the traffic through a Remote Desktop Gateway. There is one main drawback to this, that being the public IP addresses of Azure's Data Centres need to be allowed through the firewall.

Luckily for me we had a script written by an old colleague of mine, which would take the XML file, pull out all the subnets by region and add them to windows firewall with an appropriate name. Everything was nice and simple, every so often (or once a user complained they couldn't access something) the script would be ran and the issues would be resolved. Yes yes... Automation. I know. Given the restraints we had to work within this wasn't possible to fully automate, so a reminder to do this monthly was the best option. Happy Days... Until it changed.

## Microsoft's Changes

Microsoft currently publish the public IP addresses of their data centres by region and in the format of an XML document.  This is all changing very soon, this has been in the works for some time and if you tried grabbing the old format of IPs you'd have noticed that there is a big ol' [DEPRECATING] on the name of the download link. The install states that as of the 30th June they will swap over to the new json file format, arranged by services offered. 

If you're wondering why you should care as you can RDP to your personal resources in Azure just fine, chances are this won't affect you. However, this does bring about some changes to most businesses/Enterprise environments. On one hand a chunk of enterprises will be using Site-to-site VPNs or Express Routes to connect direct to their Azure resources, although if you're anything like my organisation and use a Remote Desktop Gateway to connect to cloud resources then there is a good chance this will have an impact on you too. 

## Amending the Script

I was tasked with updating the script to deal with the new json file type as well as the change of format from region to services. I first took a look at the original script to see what I could re-use and salvage with a few tweaks, thankfully I could keep the concepts behind the iteration of the file the same. All I needed to work out is how PowerShell could deal with json and how to get the script to 'look' at the right properties.

At this point, I'd already heckled the author of the original script to inform him that I was messing with his old work, and surprisingly he'd offered his help if I needed it.. (Spoiler: I did). He gave me a useful first pointer to take a look at the command [ConvertFrom-Json](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json?view=powershell-7) which was a life saver! This command takes a json file and converts it to a custom PS Object, this allows you to 'easily' call out certain properties of the json file and iterate through them appropriately. 

I got to work attempting to get the correct output which could then be used in a firewall rule. I ran into a few issues along the way but luckily the original author was on hand to dangle the carrot down in-front of me and put me on the right track. Seriously, huge shout-outs for not just providing the answer from the get go but making me work to get there! 

Finally, I had something that returned the right values which could be used to create the firewall rules, at this point I re-used this code with a few naming convention tweaks to suit the new format. You can see a snippet of the script below or check it out on Github [here.](https://github.com/MJWyattCyber/Azure-Public-IP-Script)

    #Author: Matt Wyatt
    #Co-Author: Matt Hann
    #Description: Takes the latest Json file which is placed at $path and adds RDP over 3389 for all services into the local windows firewall.

    $path = "C:\Scripts\AzurePublicIP.Json"

    #Imports the json file which will have to be manually downloaded and placed at the following location and name
    $import = Get-Content $path | ConvertFrom-Json

    #Grabs all the old rules
    $oldRules = Get-NetFirewallRule -Group "Azure RDP*"

    #Iterates through the json file, particularly 
    $firewallRule = foreach($service in $import.values){
                        if($service.properties.region -eq ""){
                            $ServiceName = $service.name
                            $ServiceIPs = $service.properties.addressPrefixes
                            New-NetFirewallRule -DisplayName "Azure RDP $ServiceName" -Direction Outbound -Protocol TCP -RemotePort 3389 -Group "Azure RDP $((get-date -Format d).ToString())" -Action Allow
                            Set-NetFirewallRule -DisplayName "Azure RDP $ServiceName" -RemoteAddress $ServiceIPs -Verbose
                        }
                    }

    foreach($rule in $oldRules){
        Remove-NetFirewallRule $rule.Name
        }
