---
layout: post
title:  "Docker Swarm in LXC, Part 1"
date:   2018-09-03 10:31:00 -0600
categories: lab homelab proxmox ansible docker workflow
---

I decided to run a Docker swarm at home because I thought it would be useful to know how to do and didn't really appreciate what I was getting into when I started it all. Looking back, this was a really useful experience and I was able to figure out quite about how to make things work. 

### Some Considerations

First, does it make sense to have a swarm for you? I interact with them some at work so I need to know something about them, so that made this worthwhile for me. It's a big pain in the ass and it's a mystery why some things don't work until you find something obscure someone wrote that describes your exact problem. I'm also partly to blame for my troubles as I am treaitng this as a hobby and not a real project, I didn't read the documentation thoroughly for everything so some things I've run into were clearly documented and I should have just read the whole documentation instead of skimming for the bits I thought I needed. 

Assuming you're willing to put up with the above, let's move on.

### Requirements

If you have a proxmox lab anything like mine, you've got most everything you need. My Swarm set up continues to evolve, but currently looks like this

- 4x LXC containers running my swarm nodes, my manager, and my aptcache
- 4x Static IPs so I don't lose anything
- 3x LXC Containers with docker installed
- Appropriate AppArmor profiles
- NAS from my DD-WRT router
- 2x worker nodes
- 1x manager node

The AppArmor bit is important. By default, LXC has restrictions on it that prevent it from doing many of the cool things docker needs to do to make things work. This [post from Google](https://cloud.google.com/container-optimized-os/docs/how-to/secure-apparmor) and this [post from Docker](https://docs.docker.com/engine/security/apparmor/) were very important for knowing what I was supposed to do here. 




build docker swarm in VMs
get NAS working
Get NAS working in LXC
Get Docker working in LXC
Get simple ubuntu container to run

security.nesting: true
lxc.apparmor.profile: unconfined
lxc.cap.drop:
lxc.cgroup.devices.allow: a



get swarm built
build docker network

get NAS working

change confi again
security.nesting: true
lxc.apparmor.profile: unconfined
lxc.cap.drop:
lxc.cgroup.devices.allow: a
security.privileged: true


useful errors:
7d8ad5c03c7753e1aaf17dae78f02668d7c2     docker-lxc3         Shutdown            Failed 12 seconds ago              "starting container failed: error creating external connectivity network: cannot restrict inter-container communication: please ensure that br_netfilter kernel module is loaded"   

https://github.com/lxc/lxd/issues/2977

touch /.dockerenv


