---
layout: post
title: "Practical Docker for Security Admins - Part 2"
date:   2016-12-20 16:29:15 -0600
categories: lab homelab budget testing
---

## More test cases, but first, some housekeeping

Docker will fill up disk space quickly if you're not managing it actively or planning for it. Since I use throw away VMs for my docker containers, I don't give them a lot of disk space to fill up. When your splunking through large indexes of data, you can very quickly fill up a virtual disk. There's a few ways I've found to manage the data.

### Clean up after yourself

Similar to how you run something with docker run, docker has it's own utilities to manage and inspect containers. Docker ps allows you to view running containers on your system. docker ps -qa gives you a quick view of just the container names. So far, on this VM, I've built my splunk docker image from the last page, I've created a container with splunk, and added some data to it. 

Let's see what docker ps and some related commands show us:

![docker ps]({{ site.url }}/images/dockerps.png){: .center-image }

Ok, nothing too crazy. The first command was verbose, showing us all the container data, like ports forwarded, the name, etc etc. The next one was just a quick oneliner of the container ID. Then we killed that container and started up a new one. If we look at running containers again, we only see a single container. The next command is where stuff get's interesting, because docker ps -qa shows us 2 containers, one of which matches the terminated container! It didn't clean up after us, so we need to do that ourselves.

Speaking of didn't clean up after us, remember that container we built? If you watched the build process, you'll remember it spun up some intermediate containers. Let's see if they left anything behind. I saved myself some trouble here by piping it to xargs and cleaning up at once.

![docker danglers]({{ site.url }}/images/dockerdanglers.png){: .center-image }
