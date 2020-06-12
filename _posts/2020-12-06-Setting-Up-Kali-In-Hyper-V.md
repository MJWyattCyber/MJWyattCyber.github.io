---

layout: single
title: "Ironing out the kinks with Kali running in Hyper-V"
date: 2020-06-12 13:01

---


Getting Started

In order to dip my toes into the world of ethical hacking and participating in CTF's such as HackTheBox or CyberSecLabs I was going to need a Kali Box. Everyone and their dog advises on running Kali in a VM, not just so it's isolated but primarily so you don't have to nuke your host operating system in the process... That and not everyone has a spare machine lying around.

It sounds simple. Get yourself a hypervisor, grab the latest ISO for Kali Linux, setup a new Virtual Machine and away you go. Right? Wrong. What I was unaware of, is how fiddly Hyper-V makes your life. I swear it's purpose is to just make your life difficult, unlike other options such as VMWare player or Oracle Virtualbox where you can just setup your VM and you're done, Hyper-V has a few more.. 'edits' that need to be made before the virtual machine feels useable. 

Immediately I noticed that Hyper-V's enhanced session option was unavailable, it soon became clear that it was also impossible to change the resolution of the virtual machine to my native screen resolution, even when editing the resolution in the OS which was more than frustrating. On top of this the whole performance of the Virtual Machine felt.. Sluggish, no matter the amount of resources I provided it just felt laggy.

Now I'm not saying Hyper-V is bad, and I'm aware it's technically a Type 1 hypervisor so behaves and interacts differently with the hardware. You could be asking why I didn't just switch to using a different hypervisor, and well, I did. I just didn't like it, the UI felt clunky and it felt wrong.. That and me, being stubborn I refused to let a piece of software stop me and was determined to get things working the way I wanted.. Even if it meant a few hours of googling and fiddling. With my ramblings out of the way I'll get into how I resolved the above issues.

Changing the Resolution

Unfortunately there's no easy way to change the screen resolution whilst running your Linux VM in Hyper-V. The way I achieved this was by editing the grub config file. Open the grub config file in your favourite editor of choice:

	Sudo vi /etc/default/grub

I chose vi but feel free to chose whatever editor you wish. Scroll down to the line that starts with: 

	GRUB_CMDLINE_LINUX_DEFAULT=

If, like me you're using vi, press i for insert mode and edit the line to make it read:

	GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video=hyperv_fb:1920x1080"

The resolution at the end will vary to match whatever resolution you are going to be running the VM at, for full screen enter the native resolution of your display. Once you're happy press esc followed by :wq to save your changes. 

The whole file should now look similar to this:

![GRUBLoaderScreenshot](/assets/images/GRUBLoader.png)

As the top comment in this file suggests we now need to update grub to apply our changes, to do this run:

	Sudo update-grub
	
Finally, reboot the machine, once the machine is back you should have the desired resolution. Victory is near!

'Fixing' the sluggish feeling

With the resolution resolved I still couldn't remove the feeling that the virtual machine was sluggish, despite giving it a couple of extra cores and plenty of RAM that could satisfy Chrome. After some research it soon became clear that again this was another quirk of using Hyper-V, it seemed the cleanest resolution would be to follow that which is documented on the distributions site:

Click [here](https://www.kali.org/docs/virtualization/install-hyper-v-kali-guest-vm/) for the original post.

For those of you that don't want to open an extra link I'll place the steps below. Full credit goes to the author, mimura1133.
Open up a terminal and enter the following:

	Sudo git clone https://github.com/mimura1133/linux-vm-tools

Navigate into the directory that was created, the full path should look similar to "linux-vm-tools/kali/2020.x/". Once at the appropriate location we can change the permission for the install script. (Yes you can specify the full path if you don't want to navigate into the directory.. I was just curious)

	Sudo chmod 555 install.sh
	
Finally we can run the install script, execute it with the following command.

	Sudo .\install.sh
	
This script installs the XRDP client and setups the configuration file, once complete you will be prompted to reboot. Shut the VM down rather than rebooting and close the remote console.

*The next step is carried out on the Host OS.*

This assumes you are running windows as the host OS, open a PowerShell prompt as Administrator and issue the following command:
	
	Set-VM "Your VM Name Here" -EnhancedSessionTransportType HVSocket

Finally, start up the virtual machine if everything was successful you should be met with an archaic looking login prompt, otherwise known as xrdp.

![XRDP](/assets/images/XRDPSplash.png)

Leave the Xorg in place and login with your virtual machines credentials and voila! A smooth, full screen experience. Bliss. Totally worth it. Honest.
