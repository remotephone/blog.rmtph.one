---
layout: post
title:  "Docker Swarm in LXC, Part 1.5 - The Kerneling"
date:   2010-03-09 10:31:00 -0600
categories: lab homelab proxmox ansible docker workflow
---

Well life catches up with you fast I guess. I havent posted any updates because I was posting as I was building and I hit a roadblock. I'll first talk about that in case Google gets you here while you're running into the same problem and then give the pieces I did figure out. 

## Docker Swarm in Proxmox LXC Containers

Docker swarm can run real easily in VM's. Install docker, create a swarm, add nodes to the swarm, toss some stuff in it, you're basically done. Within an LXC container, some restrictions are going to give you problems. For the pct.conf file, I ended up with this configuration that works. 

~~~
arch: amd64
cores: 2
hostname: swarm-wrkr3-lxc
memory: 2048
mp0: /lib/modules/4.15.18-11-pve,mp=/lib/modules/4.15.18-11-pve,ro=1
net0: name=eth0,bridge=vmbr0,gw=192.168.1.1,hwaddr=C6:1F:1A:5A:A2:83,ip=192.168.1.4/22,type=veth
onboot: 1
ostype: ubuntu
rootfs: local-lvm:vm-111-disk-0,size=25G
swap: 512
unprivileged: 0
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
~~~

The important bits are the ones at the end. Proxmox provides a handful of app armor profiles that allow you to run certain applications that would be blocked from running sinde a container. Unconfined isn't great from a security perspective, but since docker will still isolate processes from the LXC container, you do preserve some level of isolation. This profile allows you to get a container running to begin with.

After running my docker-ce and swarm install playbooks, things would look up and running, but containers would always fail to launch on lxc worker nodes. My docker-compose file spun up traefik and another network to allow things to talk to each other without being exposed publicly. A `docker network ls` on my worker showed:

~~~
root@swarm-wrkr3-lxc:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3bca5c1794b6        bridge              bridge              local
6ec1975c097b        host                host                local
f6c66b4943e1        none                null                local
root@swarm-wrkr3-lxc:~# 
~~~

But listing the networks on the manager, running in a virtual machine, showed:

~~~
root@swarm-mgr-vm1:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
6cee66aec471        bridge              bridge              local
a7cd74d3722a        docker_gwbridge     bridge              local
mq2gpjfvhgz0        exposed             overlay             swarm
b4ef8f4e3714        host                host                local
1mkakpah4inc        ingress             overlay             swarm
fa853d2b4f5e        none                null                local
7ozgltfb5q9j        sample              overlay             swarm
xwyi8zbxy488        traefik             overlay             swarm
~~~

A significant number of networks were not being created. A `docker info` on the worker showed why. 

~~~
<SNIP>
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
Product License: Community Engine

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
~~~

I have ipv6 disabled on my proxmox hosts, so the ipv4 bit is the one causing problems. Docker needs iptables to route traffic appropriately and manage your network traffic. The problem is with the linux kernel, not anything I was doing (which was frustrating to find out now). The problem is out of my area of expertise, but the br_netfilter kernel module doesn't handle namespaces correctly. 

This [issue in lxd](https://github.com/lxc/lxd/issues/3306) is where I finally found someone describing it and the fix. That leads you [here](https://github.com/lxc/lxd/issues/5193#issuecomment-433759048) and this discussion in [lkml](https://lkml.org/lkml/2018/11/7/680). The short of it is theres a proposed solution, some back and forth on it, and more work to do. 

## So what now?

I have my swarm running VMs now. It works, traefik serves the traffic for all my hosts, I watch stuff with portainer, and just have a few services in there to do random things. I will continue to run it in my VM's until the changes are merged into the kernl and migrate to LXC from there. Fortunately, I can add nodes and remove them as I scale services up and down. I'll update this when the feature is available. 