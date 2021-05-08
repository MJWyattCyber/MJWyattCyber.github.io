---
header:
    teaser: /assets/images/MsSurface.jpg
title: "Setting Up Kali Linux in Azure"
layout: single
date: 2021-05-08 21:29
categories: [Azure]
classes: wide
---

I want to start of by saying I am a huge Azure advocate, whilst I appreciate it's not the solution for everyone it does have it's benefits. For only a few pounds a month you can use it to get a Virtual Private Server (VPS) that will pack a punch, and the best part is that if you have a few days where you don't use it, you won't pay for it! What's more having such device seperated from your home network is probably a good idea. One such use case would be to have a Kali instance hosted in Azure, if you're like me and have a career or interest in Cyber Security then you may want to partake in CTF events, or sites such as HackTheBox, TryHackMe etc. You may even want to partake in BugBounties, whilst you could arguably get away with just a lightweight Ubuntu instance for BugBounties, you will still want the device dissasociated from your home network. The one downside to utilising Azure for a Kali instance is that you'll either have to upload your own image to build the machine from or make use of the provided marketplace image, whilst this is convenient to deploy it is out dated. Fear not, during this post I will cover every step needed to deploy the marketplace image, troubleshoot a few issues and run a full update to get your instance running on the latest version.

===

## Deploying the Kali Virtual Machine

As mentioned there is a Kali image in the Azure marketplace which will make our lives much easier as we won't have to mess around with a custom image from a VHD. To get things started simply navigate over to the Azure Portal and log in with your microsoft account. If you don't have an account then today is your lucky day! When you sign up you will receive £150 worth of free Azure credits for the month to carry out all your testing. Once you are signed in you'll want to create a new resource, to do this head over to the marketplace and search for 'Kali Linux', once you hit 'Create' you will be taken to the VM creation page where you can customise a few settings.

### Project Details

- **Subscription**: This should be auto-populated, if you have more than one active subscription make sure you are on the correct one.
- **Resource Group**: This will try to create a new resource group by default, you can create a new resource group or chose from previously selected details

### Instance Details

- **Virtual Machine Name**: Name the virtual machine appropriately, if there are multiple users in your subscription you'll want to stick to a naming convention. If it's just a personal playground then name the VM however you see fit. Personally, I went with a Norse theme.
- **Region**: You'll want to pick a region close to your location to reduce the latency. 
- **Image**: This should be auto populated and contain "Kali Linux - Gen 1"
- **Size**: Expand out all the sizes and scroll through whats available. I went with one of the A1_V2 sizes to keep costs low but remember, you can always change the size later on!

### Administrator Account

- **Authentication Type**: Choose between a SSH key pair or a password, if you're familiar with linux then I'd advise selecting a Key Pair but if you want to stick to a normal password that's fine. I went with the key pair.
- **Username**: Specify a username for your account
- **SSH Public Key**: Use an existing key pair or generate a new pair.
- **Key Pair name**: Provide a name for your key pair, this can remain the default or it can be changed to suit.

### Inbound port Rules

- **Public inbound ports**: Select allow selected
- **Select inbound ports**: SSH (22)

### Disks

After pressing next you will be asked to define the Disks, for the disks this may differ on personal need, and the size of the VM that you selected earlier. I kept my option fairly simple in order to keep costs down.

- **OS Disk Type**: Dependant on your VM it will support different disks there are three types to choose from, Standard HDD, Standard SSH and Premium SSD. 
- **Encryption**: To reduce overhead I kept this to the default. You can modify this if you want to manage your own encryption keys.
- **Extras**: It's worth noting that if you wish to add an extra disk or two, you can do this here. Alternatively, you can resize the OS disk or add more later on.

### Networking

Pressing next will take you to the networking section, arguably one of the most important sections when it comes to your Virtual Machines security.

- **Virtual Network**: If you have an existing network you can use that, you can also create a new network to place this in
- **Subnet**: If you're using an existing network chances are you have a subnet configured, if not create a new subnet to house the Virtual Machine.
- **Public-IP**: As before if you have a spare IP then feel free to select this, otherwise create a new one and provide a suitable name (Or keep the default).
- **NIC Network Security Group**: If you plan to add a security group later or you have a NSG applied to the subnet select None, If you want to have a NSG set up with simple rules select Basic, if you have an existing NSG to add to the VM then select advanced and choose it from the drop down.
- **Public inbound ports**: Select allow selected.
- **Select Inbound ports**: SSH (22) 

### Monitoring

I'd advise that if you're new to Azure you spend some time reading the help tips related to this section, as it does a good job of explaining each option. It's the section that is probably down to personal preference the most.

- **Boot Diagnostics**: If you wish to troubleshoot booting issues or collect further logs then keep this enabled, however, it does incur a few extra costs as the logs need to be stored in a storage account.
- **Guest OS diagnostics**: Again, it's another case of providing more logs and have the option of full diagnostics. Like the Boot diagnostics these logs require storage and as such the costs increase.
- **Auto-Shutdown**: This setting can be a life saver to keep the costs down, enabling this lets you set a time for the virtual machine to shutdown and de-allocate saving you some money if you forget and leave it running overnight.

I left the next two options of 'Advanced' and 'Tagging' at the default options, feel free to have a look through and make any changes you feel fit. Tagging can be useful for billing and identification of resources.

Download your SSH key when prompted, this is an important step - do this now to save you hunting for it later, store this somewhere safe.

===

## Amending the Security Groups

During the VM creation we defined the Inbound ports to be SSH (Port 22) as a result this has added a rule that permites inbound access on this port from any location. I want to reign in that control a little bit as I don't want any device to be able to access my machine, only those on my home network. To do this I need to change the source to be my public IP address, if you don't know how to find out your IP address, simply google "What is my IP address" and make a note of the result. To amend the rule navigate to the Azure Portal and search for 'Network Security Group' from here:

- Select the appropriate Network Security Group associated with your virtual machine or subnet. 
- Select 'Inbound Security Rules' > Select the rule called 'SSH' 
- Change the Source drop down to 'IP Address' in the box that appears enter your Public IP discovered earlier.
- Don't forget to save your changes.

If you see fit you can of course add other rules in here to control the flow of traffic to your VM, one option may be to setup an RDP rule to allow connections on port 3389, granted this would take a bit of setup on the Kali box to get running but it is possible!

===

## Connecting to your VM

To connect to your VM you'll need a terminal client installed, a popular well-known client is PuTTY. However, if you're like me and you don't like having your terminal look like it was created when Windows 95 was around there are plenty of other options. Personally I really like running a local Ubuntu dsitribution on top of Windows Subsystem for Linux (WSL) as it gives me the ability to quickly hop into a Linux environment and use a terminal that's really customisable. The method you connect by will vary depending on your desired choice, the Azure portal itself does have some pointers to help you out. If you navigate to to your Virtual Machine overview you can press 'Connect' > 'SSH' which will provide you with some helpful instructions. I'm going to be assuming that you are using a some form of command line/shell that is capable of utilising the ssh command. You will also need to know the path to your .pem private key you downloaded earlier, with these two requirements met simply enter the command:

    ssh -i <path to your private key> <your username>@<your virtual machines IP address> 

All being well you should be told that the host is unknown (you haven't connected to it before) and would you like to continue, type 'yes' and hit enter. You should now be logged in!

===

## Setting up a WinSCP Connection

This section is completely optional so feel free to skip it if you are comfortable with file transers on a Linux machine. There are some times where dealing with a GUI is just a whole lot easier, as such I utilise WinSCP, WinSCP is an open source tool which allows you to set up a connection and copy files to and from the source and target. When you first start WinSCP you are prompted to setup a new connection, enter the public IP addresss of your virtual machine in the hostname, leave the port at 22. Set the username to match your username you selected during setup. Click on the advanced button and a new window will appear, under the SSH setting select authentication. Finally, select the three dots to browse for a private key file, this will open File explorer. Change the file type to show 'All files' and browse to the location of your .pem file. Selecting your .pem file you will be prompted to save this as a .ppk format, continue and save this as a .ppk file in a safe location. You should now be able to press 'Ok' and login via WinSCP to your Kali virtual machine, allowing you to transfer files to and from.

===

## Updating Kali Linux

Before we can use our brand new Kali installation there are a few commands we need to run to get ourselves up-to-date. As the image is built of a 2019.2 build there are numerous packages that will need to be updated. When I deployed my installation I ran into a few issues with pulling the updates due to the age of the build the archive keyring package had expired, luckily the folks over at Offensive Security have published a way to do this on their Twitter account, the original tweet can be found here. There is one pre-requisite to this, in that we need to be running as true root to do this so first things first, we'll want to change the password of the root account so that we have root access. We'll then want to change into that root account, this can be achieved by running the two commands below:

    sudo passwd root

    su root

When changing the root password it's important to chose a unique, strong password. You'll be prompted to confirm the password and then also prompted to enter the password when attempting to switch into the root user account. Once we are logged in as root (you should see the terminal prompt change from your username to root) we can run the following command:

    wget -q -O - archive.kali.org/archive-key.asc | apt-key add

With this command out the way we can run a full-upgrade on the instance to get put on the latest version of Kali. This may take some time and you may need to interact with a few prompts during the process, take care to read each prompt to understand what you are being asked. When ready, run the following:

    sudo apt-get update -y && sudo apt full-upgrade -y

Once the command is complete you can confirm the version of Kali by entering 

    cat /etc/os-release

At the time of writing the latest version is Kali 2021.1. To wrap things up and finish our installation we'll want to remove unneeded and legacy packages, simply run the command

    sudo apt autoremove

That's just about that, if all steps were followed you should now have a fully functioning Kali Linux installation in Azure running the latest version of Kali linux. 

