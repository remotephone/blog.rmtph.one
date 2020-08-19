---
layout: post
title:  "Docker Swarm in LXC, Part 1"
date:   2018-09-06 10:31:00 -0600
categories: [lab, homelab, proxmox, ansible, docker] 
---

I decided to run a Docker swarm at home because I thought it would be useful to know how to do and didn't really appreciate what I was getting into when I started it all. Despite the current them of my blog, security is what I'm really interested in and a swarm is going to give me ways to quickly deploy and throw away things to play with. 

Looking back, this was a really useful experience and I was able to figure out quite about how to make things work. This series of posts is going to be written in the order I figured things out, hopefully you can learn from the problems I ran into and how I fixed them.

### Some Considerations

First, does it make sense to have a swarm for you? I interact with them some at work so I need to know something about them, so that made this worthwhile for me. It's a big pain in the ass and it's a mystery why some things don't work until you find something obscure someone wrote that describes your exact problem. I'm also partly to blame for my troubles as I am treating this as a hobby and not a real project, I didn't read the documentation thoroughly for everything so some things I've run into were clearly documented and I should have just read the whole documentation instead of skimming for the bits I thought I needed. 

Assuming you're willing to put up with the above, let's move on.

### Where I want to be

I got interested in [traefik(](https://traefik.io/) because I saw it working and it's really cool. You can expose containers by DNS name and provide HTTPS by only exposing Traefik. They include instructions for running it both locally and in a swarm. If you're running a service like portainer, it would be available at something like https://portainer.mydomain.com. 

I want to run traefik and just make it available on my home network. Since the really out of the box solution they provide is better for internet accessible deployments, we'll need to make some tweaks. 

### Requirements

If you have a proxmox lab anything like mine, you've got most everything you need. My Swarm set up continues to evolve, but currently looks like this

- 4x LXC containers running my swarm nodes, my manager, and my aptcache
- 4x Static IPs so I don't lose anything
- 3x LXC Containers with docker installed
- Appropriate AppArmor profiles
- NAS from my DD-WRT router
- 2x worker nodes
- 1x manager node

The AppArmor bit is important. By default, LXC has restrictions on it that prevent it from doing many of the cool things docker needs to do to make things work. This [post from Google](https://cloud.google.com/container-optimized-os/docs/how-to/secure-apparmor) and this [post from Docker](https://docs.docker.com/engine/security/apparmor/) were very important for knowing what I was supposed to do here. I'm pretty sure there's a working AppArmor profile somewhere, but I can't find it. 


### General Process Notes

My process for doing each of these was pretty iterative. I started by getting things working on my laptop as a single docker host. After I figured out how to do each thing manually, I'd write an ansible-playbook with roles to handle each part of the tasks. Some I wrote myself, some I just took from others and modified to suit my needs. 

Once it worked on my laptop and I could run a docker swarm and spin up stacks with compose files, I created the docker swarm in VMs. Doing it in VMs first eliminated a lot of headaches I had to figure out getting LXC containers working and let me focus on the substance of what I was trying to get done. Once I had this working in VMs, I created LXC containers, got docker working in them, built a swarm, got single containers working, go a stack working, and got traefik configured to serve up traffic. 

I'm a big fan of writing this stuff yourself, but it seems unnecessary to reinvent the wheel every time. Of course, if you're running this stuff on any system of yours, you should be able to read the code. If you can't read something and tell what it's doing, its probably better to write an ugly version yourself than use something else someone wrote.  


### Get it working locally first

Install Docker on your laptop. I wrote a playbook that did that by reading the documentation and turning each step into a task in the docker-ce role.  You can see my playbook [here](https://github.com/remotephone/docker-ubuntu-ansible). Take a look at the docker-ce role and compare it to the docker-compose role. Something I see people who are better at this than I am do pretty frequently is configure everything as variables. This is an example of a task written statically in [https://github.com/remotephone/docker-ubuntu-ansible/blob/master/roles/docker-ce/tasks/main.yml](https://github.com/remotephone/docker-ubuntu-ansible/blob/master/roles/docker-ce/tasks/main.yml).

~~~ bash
- name: Adding Docker repo
  apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable
    state: present

~~~


Here is that same task written a little more dynamically. 

~~~ bash
{% raw %}
- name: Adding Docker repo
  apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu "{{ ansible_distribution_release }}" stable
    state: present
{% endraw %}

~~~

Still, this assumes you're running ubuntu on a 64 bit system and want to use the stable branch. If you're using linux mint, where you often use ubuntu repositories, that won't work, it will try to populate with the Mint version. 

Now we have something simple that will install docker somewhere. Do that. Get a single container running. 

~~~ 
test@test:~$ sudo docker run -dit alpine
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
8e3ba11ec2a2: Pull complete
Digest: sha256:7043076348bf5040220df6ad703798fd8593a0918d06d3ce30c6c93be117e430
Status: Downloaded newer image for alpine:latest
6406e7fdcf29cb5de6fbcb3db6591bcc0c21eeea2f01338e6bbbaa720cc877bf
test@test:~$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6406e7fdcf29        alpine              "/bin/sh"           8 seconds ago       Up 6 seconds                            relaxed_wing++
~~~ 

Some of the issues I ran into that I had to fix before getting this done the first time were mostly around having appropriate permissions (sudo/docker group membership). [Exploits](https://www.rapid7.com/db/modules/exploit/linux/local/docker_daemon_privilege_escalation) exist to get to root if your account in the docker group is compromised, so treat it like granting an account passwordless sudo, know your threat model, and plan for the risk appropriately.  

### Docker Compose

Docker compose allows you to write a single file that can create multiple services as part of a stack. Install it and get it working. I didn't need everything [this role](https://github.com/weareinteractive/ansible-docker-compose) gets up to, so I took it, cut out a lot of parts, and added it as a role to my existing docker install playbook. 

There is a simple docker-compose tutorial worth following [here](https://docs.docker.com/compose/gettingstarted/#step-2-create-a-dockerfile) that will give you a good introduction to them.

The [traefik docker quickstart](https://docs.traefik.io/#the-trfik-quickstart-using-docker) also provides a good foundation to where we're going from here.


### Enough for now

If you're with me so far, you have docker and docker-compose installed on you local workstation, you can launch a container, you can access traefik over http, and you're ready to do more. Subsequent posts will be applying the lessons from [this post](https://github.com/wekan/wekan/wiki/Traefik-and-self-signed-SSL-certs) to provide HTTPS with a self-signed cert to our docker-swarm.