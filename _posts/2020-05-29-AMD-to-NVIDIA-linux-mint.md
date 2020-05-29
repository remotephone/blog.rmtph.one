---
layout: post
title:  "AMD to Nvidia - Linux Mint and Cinnamon Crashes"
date:   2020-05-29 14:40:00 -0600
categories: homelab workflow
---

I upgraded from an AMD RX-480 to an Nvidia RTX 2070 Super. What an ugprade. The windows side of thing was pretty painless except for the fact that I had to completely reinstall my OS. Fortunately, for that, I had [this](https://gist.github.com/remotephone/948ae20e1e02c05a9b3bb6bcb43abf50) installer script I lifted (with some small tweaks) from [jessfraz](https://github.com/jessfraz) and it proved super helpful. For my Linux workstations, I have an ansible [playbook](https://github.com/remotephone/remotephone-desktop-ansible) I run that helps if I'm starting from scratch, but not so much if I just need to make a hardware change.


When I'd login, I'd get an error that Cinnamon had crashes and it would ask if I wanted to reload it, which didn't help. Turns out my AMD drivers were still installed and conflicting, even though I'd already gone into Driver Manager and told it to use the latest Nvidia proprietary drivers. 

Finding out how to fix this from forum posts was a pain since so many old fixes don't apply any more and you have to get multiple pages deep before you find anything. This hopefully helps someone in the same boat.

## Where you might be

If you're like me, you've previosuly installed the [AMD proprietary drivers](https://www.amd.com/en/support/kb/faq/gpu-635) and once you've done the hardware swap, when you boot into linux you find cinnamon crashing every time you login. 

## Get Ready

Make sure you system is fully upgraded, all these steps were done on Linux Mint 19.3 Tricia.

```
sudo apt update
sudo apt ugprade -y
sudo apt autoremove -y
sudo apt autoclean -y 
```

## Tweak the Uninstaller

You'll need to remove those proprietary drivers first. The script explicitly checks that you're running Ubuntu by looking in the `/etc/os-release` file and checking for the ID. You'll need to edit the `/usr/bin/amdgpu-pro-uninstall` script, replace "ubuntu" with "linuxmint" and run the uninstallation script again.

Once the drivers are removed, reboot, and you should be back with properly working drivers. Reboot your machine

## That's it. 

Not much fanfare here, just wanted to write this down in case it helps someone else. 

