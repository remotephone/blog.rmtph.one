---
layout: post
author: remotephone
title: "Security Labs on a (no) budget"
date:   2016-12-20 16:29:15 -0600
categories: lab homelab budget testing
---

## So you want a lab?

It's going to cost money. You can't do this for free. Luckily, you've probably spent most of it already. I do everything I need to do on my desktop which does triple duty as a workstation and gaming machine. You don't need as much storage as I have, but it comes in handy.  



Here's my desktop configuration:

* AMD FX-6350 - overclocked and Hyper212 Evo knockoff Cooler
* 16GB of ram
* 2 SSD's - 120GB for Windows Install & 256 GB drive split 30GB for Linux install and 220GB for VM drives
* 2TB internal storage drive - Windows image backups, media, other stuff
* 750 GB drive - 500GB for games, 250 GB for random backups
* AMD RX-480 8GB
* External 2TB drive for backing up everything important

The logic here is I have enough cores to run multiple VMs at once and enough storage to keep them all. Other processors are better and I'd upgrade if I needed to, but my processor does pretty well overclocked, I think I'm at 4.5ghz. 

I use multiple drives because I'm always tinkering with and reinstalling OSes. Windows gets the full 120GB because it tends to be space hungrier and that's where I keep in progress work, the 220GB partition on the other SSD just houses VMs. You could probably get away with 1 single SSD at 500gb, 100gb for windows, 30gb for linux, and the rest for data and VMs. I really like the 250GB of random backup space, I keep installation files of all the applications I use in there and just reinstall without having to download them all over again every time I nuke the install.  

I also have two monitors. I don't know how people got along before dual monitor setups, but it makes a world of difference. I can full screen a VM on one window, have notes anda  webbrowser open on the other monitor, and not have to pop out of full screen all the time. If I'm doing something between VMs and don't really need the desktop real estate on the VM, I can keep multiple VM desktops open next to each other to work on. Once you have it, you'll never want to let it go. 

### Virtualization

I'm not a fancy man, so I don't pay for things if I can avoid it. I highly recommend Virtualbox for most of your virtualization. There's some machines I've gotten (for the OSCP for example) that require VMWare Player, but the free copy does well enough for those 1 offs. Virtualbox supports snapshotting for free which makes a world of difference if you need to turn your computer off or want to quickly wipe a workstation to defaults.

![VBox Console]({{ site.url }}/images/vboxconsole.png){: .center-image }

### OSes and where to get them

Glad you asked. The linux ones are easy enough to find, go to distro watch or go to a download repository and just navigate backwards through the directories till you find old versions. There's also handy things like the [Centos Vaults](http://vault.centos.org/) where you can get any old version you want. These come in handy if you're testing specific exploits or versions of something. 

Windows is trickier, but there's some real nuggets of gold out there. Microsoft has TechNet where you can download some versions of Windows Server through their [Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2012-r2) assuming you're using their license appropriately. Make sure you read and follow their rules, I can't tell you if you're ok to download them or not, but they are available.

More interestingly, for consumer OSes from Windows 7 - 10, you can download VHD files that run neatly in Virtualbox. They are free to try and likely expire sometime. They used to offer Windows XP images, but those no longer work. Check em out [here](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/).


### Save your work

I can't emphasize how much good stuff I lost because I didn't save it. My recommended approach to your VMs is to build them and then create a snapshot before you break anything serious. If you're going to install a bunch of applications or test an exploit, make sure you save your work.

There's a lot of guides and how-to's out there that I've run into that work only part way and then I'm forced to back up a few steps and do something differently or maybe I need to install a certain version of something or the distro package doesn't work and I need to install from source or whatever, but you're going to run into problems and you need to have a state where you can revert back to and not lose everything.


### Take Notes

I take notes of a lot of what I do. I keep OneNote documents for all sorts of stuff, and they're great, but they're limited in how I can share them. With the new versions of Office, everything has to go into OneDrive and that's just lame and the new interface sucks and just no. 

I use notedpad ++ a lot. This is a screenshot of me figuring out some Jekyll stuff. It's real jot it down as I go kinda stuff, but it works. I want to write another article about notepad++ and other tools because I really think it's one of the most useful things out there. 

![notepadnotes]({{ site.url }}/images/notepadnotes.png){: .center-image }

It might look like scratch, but this is what I go back to when I redocument what I'm doing later. 

### Some thoughts and links

Write stuff down. Have enough saves so you can throw it away and be back to a fresh start quickly. Don't spend all your money to do this. These are some links to get some great stuff.

* [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
* [VMWare Player](http://www.vmware.com/products/player/playerpro-evaluation.html) - The free one
* [Microsoft Technet Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2012-r2)
* [Windows IE VMs for Developers](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)
* [PC Part Picker](https://pcpartpicker.com/) In case you need stuff
