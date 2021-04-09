---
layout: post
author: remotephone
title:  "DD-WRT as a home router"
date:   2021-02-11 23:45:00 -0600
categories: homelab dd-wrt
---

## DD-WRT - That still works?

This will be a quick post. DD-WRT has served me well over the years. It's not as slick as the Ubiquiti gear (much cheaper) and not as feature-full as pfsense (can be cheaper), but it does what it needs to do. I wanted to write up some of the things I have trouble finding documented outside of forums

## Firmware Updates

Search for DD-WRT firmware and you're likely to find hits for stable versions over a decade old and a pretty small list of supported routers. There is a [download](https://dd-wrt.com/support/router-database/) link on the main page where you can search your router. My model is the [Netgear R7000](https://www.netgear.com/home/wifi/routers/r7000/) and while supported, the latest version you will find on the website is from November 2020 (at the time of this writing). But it turns out they're pumping out beta versions several times a week, but you need to browse directly to their file server to find them. [This](https://download1.dd-wrt.com/dd-wrtv2/downloads/betas/) will get you to the archive, from 2010-2021. Click into the year, pick a build, find your model and you'll have an updated firmware version to install.

Each build contains two files: the `factory-to-dd-wrt.chk`, a file you can use to flash a factory device to dd-wrt, and an updated `{model}-webflash.bin` to install if you're already running DD-WRT. 

## Installation Process

This assumes you have DD-WRT already installed. 

1. I use Firefox, I do not use Chrome. There were [issues](https://forum.dd-wrt.com/phpBB2/viewtopic.php?t=317974&sid=a437d219ab843ab5d65e64a00ee65067) that caused updates to hang for a while, but it seems t hey were fixed. 
2. Use a private browsing session, this avoids having anything cached that could cause trouble for your update process
3. Perform a backup of your existing configuration
    1. Go to `https://192.168.0.1/config.asp` or whatever IP you use
    2. Simply click Backup
    3. Protect this file like its a secret, it stores all your passwords and keys in there
4. Go to `https://192.168.0.1/Factory_Defaults.asp` to set the router to default settings.
    1. There was (maybe still is?) an issue where custom configurations take up too much space in some models. It can't take the new update and have the existing configs at the same time or maybe theres a different problem, but that seemed to be it at the time. 
5. When the router comes back up, update the firmware using the GUI. Allow it to reboot
6. When  it comes back up again, upload your backup at `https://192.168.0.1/config.asp` to restore all your settings

That's it! Verify the firmware updated by checking the build info at the top right. I tend to update once a month or so and the whole process takes me 20 minutes or so. 

## A Modern SMB Server

I run a simple file share from my router. I don't know when this happened, but around the time SMBv1 and SMBv2 became an accepted bad idea, the capabilties for SMBv3 were added, but they're not on by default. 

To enable them, 

1. Go to `https://192.168.0.1/NAS.asp`
2. Set the Minimum and Maximum protocol versions you want like so 

![New Topic]({{site.url}}/images/ddwrt-nas.png){: .center-image }

3. Make sure your configs support them. In my /etc/fstab file in my proxmox servers, I had to set the version explicitly.

    ~~~
    //192.168.0.1/NAS /media/nas cifs credentials=/etc/.smbcredentials,vers=3.0 0 0
    ~~~ 

4. Remount your drives and it should work

## OpenVPN Server

The openvpn server is pretty nice. I use it when I'm in places where I trust my home network more than the network I am in, and typically on a phone or laptop. I put together [this tool](https://github.com/remotephone/openvpn_cert_generator) to simplify the process of generating certificates. This also uses TLS instead of password authentication because why not. 

## That's it

That's all, I hope someone finds this and it makes what they're trying to do with openvpn easier. 