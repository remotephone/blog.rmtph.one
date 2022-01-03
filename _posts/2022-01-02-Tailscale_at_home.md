---
layout: post
author: remotephone
title:  "Tailscale at Home"
date:   2022-01-02 22:49:00 -0600
categories: lab workflow networking lxc
largeimage: /images/avatar.jpg

---

# What I'm Doing Now

I have a homelab of sorts, I previously wrote about it [here](https://remotephone.github.io/posts/Proxmox_Lab_on_a_Slightly_Larger_Budget_Part1/) and [here](https://remotephone.github.io/posts/Docker-Swarm-in-LXC_Part-1.5/). It's changed a little bit since I wrote those, but I still had a need (more of a want tbh) to be able to access everything remotely. This is a logical diagram of what my network looks like:

![network]({{site.url}}/images/homenet.png)

To provide secure access to the swarm, I have everything behind [Traefik reverse proxy](https://traefik.io/) exposed to the internet. This is great because it provides HTTPS and 2FA protected SSO for all my services and was mostly easy to set up. Traefik also supports SSH, but there are some [minor gotchas](https://community.traefik.io/t/ssh-proxy-from-traefik-to-lxc/608) involved that I didn't want to work around and I don't run any other services that use SSH in my swarm, only HTTPS. I also have things scattered around AWS, other VPS services, and who knows what else I may want to access one day. 

To connect to other services outside my docker swarm in my home network or lab I have been using OpenVPN with TLS authentication. This is mostly secure and functional, but with Traefik handling most of my access needs, I'm basically using OpenVPN to give me an easy way to SSH into hosts in my network. It however, has been a hassle to regenerate certificates and I ended up creating some hacky tools like [this](https://github.com/remotephone/openvpn_cert_generator) to help make that easier. I want to get rid of this eventually and there's no time like the new year.

I needed something that just worked and I wanted something to make that externally accessible and easy. Enter [Tailscale](https://tailscale.com/).


## Doing it Wrong on Purpose

Tailscale allows you to install an agent onto any host anywhere and access it "directly" from any other authenticated Tailscale host. It supports most platforms and requires little to no configuration. It can help you eliminate jumpboxes, vpns, and simplify connecting to hosts on other networks. For my purposes, the easiest way for me to get using and familiar with the service was to use it to expose a jumpbox that I could access my other hosts through. 

A more appropriate way to use Tailscale would be to deploy the agent to every host I want to connect to and connect directly. I haven't done that because I'm still learning the service (literally got started on this last night) and if there's any gotchas I might run into and because most of the hosts I will connect to are going to be short lived and built ad hoc to test and try new things. 

Using an ansible role like [this](https://github.com/artis3n/ansible-role-tailscale) as part of deploying new hosts seems very interesting if I decide to move forward with this, but for now I am running a single LXC container as a jumpbox. I also installed the agent on my two proxmox hosts that run my cluster so I can access them directly. 


## An LXC Tailscale Jumpbox

First sign up for the service [here](https://login.tailscale.com/start). Then you'll want to enable MagicDNS by setting a `Global Nameserver`, I simply used `8.8.8.8`. With MagicDNS enabled, you can ssh to the hostname instead of the IP, (eg, `jumpbox` instead of `100.100.100.1` or whatever you're assigned).

I deployed a default Debian 11 LXC container and loaded my public SSH key into it. This is the view in Proxmox. 

![proxmox-container]({{site.url}}/images/jumpbox_proxmox.png)


I followed the installation instructions [here](https://tailscale.com/kb/1017/install/), both with the manual install and simply curling to `sh`. The configuration for the container is mostly default, but I was unable to bring up the tunnel. I was getting an error like this:

```s
Jan 02 04:02:37 jumpbox tailscaled[3796]: wgengine.NewUserspaceEngine(tun "tailscale0") ...
Jan 02 04:02:37 jumpbox tailscaled[3796]: is CONFIG_TUN enabled in your kernel? `modprobe tun` failed with: modprobe: FATAL: Module tun not found in directory /lib/modules/5.11.22-5-pve
Jan 02 04:02:37 jumpbox tailscaled[3796]: tun module not loaded nor found on disk
Jan 02 04:02:37 jumpbox tailscaled[3796]: wgengine.NewUserspaceEngine(tun "tailscale0") error: tstun.New("tailscale0"): CreateTUN("tailscale0") failed; /dev/net/tun does not exist
Jan 02 04:02:37 jumpbox tailscaled[3796]: wgengine.New: tstun.New("tailscale0"): CreateTUN("tailscale0") failed; /dev/net/tun does not exist
Jan 02 04:02:38 jumpbox tailscaled[3813]: wgengine.NewUserspaceEngine(tun "tailscale0") ...
Jan 02 04:02:38 jumpbox tailscaled[3813]: is CONFIG_TUN enabled in your kernel? `modprobe tun` failed with: modprobe: FATAL: Module tun not found in directory /lib/modules/5.11.22-5-pve
Jan 02 04:02:38 jumpbox tailscaled[3813]: tun module not loaded nor found on disk
Jan 02 04:02:38 jumpbox tailscaled[3813]: wgengine.NewUserspaceEngine(tun "tailscale0") error: tstun.New("tailscale0"): CreateTUN("tailscale0") failed; /dev/net/tun does not exist
```

I was able to find a post in the [proxmox forums](https://forum.proxmox.com/threads/passing-usb-device-on-lxc-not-working-after-upgrade-to-7-0.92178/#post-401606) that helped me fix it. This is what my container config looks like now, I had to edit the file to add the last two lines. 

```conf
root@pve1:~# cat /etc/pve/lxc/108.conf
arch: amd64
cores: 1
features: nesting=1
hostname: jumpbox
memory: 1024
net0: name=eth0,bridge=vmbr0,firewall=1,gw=redacted,hwaddr=FE:C6:71:E2:34:1F,ip=redacted,type=veth
ostype: debian
rootfs: local-lvm:vm-108-disk-0,size=12G
swap: 512
unprivileged: 1
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
lxc.cgroup2.devices.allow: c 10:200 rwm
```

I also added the following lines to my SSH config to let me use SSH Agent forwarding. 

```
Host jumpbox
  ForwardAgent yes
```

To summarize, my steps would be (doing this from scratch):

1. Create the LXC container
2. Edit the container config to allow the tunnel
3. Boot the container
4. Install tailscale, bring the service up
5. Authenticate
6. Modify SSH config
7. SSH into the jumpbox and test access.


And it works. I can SSH into hosts pretty easily, get around my network freely, and its encrypted, authenticated, and supports more stringent access control lists should I want to use them. 

## Conclusion

So now, I have a good solid HTTPS and SSH access path to hosts in my network, SSO authenticated and encrypted all around. I am very pleased with Tailscale, especially for the price (free for 20 devices) and will continue to look for ways to use it. I am excited to see how this service develops, what options I have as a home user, and what features they may add in the coming years. I am using the free service now, but they make a very convincing case to turn me into a paying customer. I am very impressed by how easy and quick it was as my first hosts were up and connected in under 15 minutes of deciding to start working on this.

Hope this helps someone get a deployment off the ground if you wanna test this, good luck and happy new year.