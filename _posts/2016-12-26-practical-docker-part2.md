---
layout: post
title: "Practical Docker for Security Admins - Part 2"
date:   2016-12-20 16:29:15 -0600
categories: lab homelab budget testing
---

## More test cases, but first, some housekeeping

Docker will fill up disk space quickly if you're not managing it actively or planning for it. Since I use throw away VMs for my docker containers, I don't give them a lot of disk space to fill up. When your splunking through large indexes of data, you can very quickly fill up a virtual disk. There's a few ways I've found to manage the data.

### Clean up after yourself

Similar to how you run something with docker run, docker has it's own utilities to manage and inspect containers. Docker ps allows you to view running containers on your system. docker ps -qa gives you a quick view of just the container names. So far, on this VM, I've built my splunk docker image from the last page, I've created a container with splunk, and added some data to it. Let's see what docker ps and some related commands show us:

![docker ps]({{ site.url }}/images/dockerps.png){: .center-image }
