---
layout: post
author: remotephone
title:  "My Homelab - Part 1 - Hardware and Networking"
date:   2022-02-18 16:33:00 -0600
categories: lab workflow networking lxc
largeimage: /images/avatar.jpg

---

# My Homelab

My last post made me think about how much my homelab has changed and that maybe it's time to write about how I am doing things in it again. Most of my hardware has stayed mostly the same, but I've simplified the networking and set up quite a bit, hope this helps someone answer questions about their own set up if they need to.

## Hardware

I run a few things, most are hardware I have around the house and the rest run in AWS, I'll break them down in this and future posts.


### Networking

I use a router with [DD-WRT](https://dd-wrt.com/) instaled. DD-WRT was the king of its day, and can still provide quite a few features and capabilities the average home router won't provide. With DD-WRT, you replace the firmware that's running your home router (it supports many different models and versions) and run a custom firmware. If you look at the DD-wrt page, you will see old and outdated versions of the firmware. For the latest update, you should be pulling versions from their [FTP downloads site](https://download1.dd-wrt.com/dd-wrtv2/downloads/betas/2022/01-16-2022-r48128/). Find your model number, get a version, install it. 

I have some issues with the normal installation process and the most fool proof process I've run into is:

1. Download the firwmare to a computer with a wired connection to the router
2. Take a backup of the router's firmware in `Administration > Backup > Backup`. Guard this file, it includes passwords.
3. Perform a factory reset of your router. I do it through the web GUI, it's fine.
4. Once it comes up back up, get back into the router using an incognito browser. I have had success with Chrome, but I usually use Firefox.
5. Go to ` Administration > Firmware Upgrade > Choose File` and upload the firmware you downloaded. Once you click upgrade, just leave it alone. Keep an eye on the bottom of the browser and you'll see the upload progress. 
6. If it stalls and gets stuck, you should reset the router to factory settings and try again.
7. If the upload finishes, the router will reboot. Log back in, set any username/password you want, and then go to `Administration > Backup > Restore Configuration`
8. Upload the backup file you downloaded earlier. 
9. When the router reboots you'll be back with everything up to date and your old configurations. 

Use whatever local IP space you want for your configuration, it doesn't matter. I would recommend you use a /22 for your network at least because a /24 can get crowded quickly. Since DD-WRT does not support VLANs very easily (I've spent some time on it and never been able to get it to work, but maybe it's just me), I just mentally separate my devices.

## ProxMox

I will go more in depth about my proxmox hosts later, but I run two proxmox hosts on my network. Both of them are running on Lenovo tiny form factor PCs. 

## DNS

I'll make a separate post about my AWS environment supporting my lab, but I have a single domain lots of what I use is on, that domain is managed by AWS Route 53. For my homelab to work both inside and outside my network, I need to do some DNS trickery in the router. I wanted names on a public domain to resolve internally without having to publish DNS records to a public DNS server. I am using dnsmasq to do most of the work here.

1. In `Setup > Basic Setup > Network Address Server Settings` check the box for `Use DNSMasq for DNS` and `DHCP-Authoritative`.
2. In `Services > Services > DHCP Server`, set `LAN Domain` to your domain.
3. In `Services > Services > Dnsmasq`, click Enable for Dnsmasq. 
4. I have these `Additional Dnsmasq Options` set. The comments are not in my real config
  ````
  local=/my.domain/                       # Assign my domain as the local domain
  address=/docker.my.domain/192.168.1.1   # Assign my swarm entrypoint to an IP
  rebind-domain-ok=/my.domain/            # all DNS rebind, only matters if I ever do want to put private names on my public domain
  address=/vm.my.domain/99.99.99.100      # random other host on my domain, just a VM in google cloud on a public IP
  expand-hosts                            # turn docker to docker.my.domain automatically
  ```
5. Save and reboot your router.  

## Port forwarding

I forward a single port from the internet to my cluster. 

## I think that's it for now

So that's the basics of it. This set up allows me to deploy services to my swarm, access them remotely and mostly securely, and have networking configured that mostly just works. The next post will cover what I do in AWS for DNS and some services I run in my cluster to glue it all together or maybe my proxmox cluster, I dunno. 
