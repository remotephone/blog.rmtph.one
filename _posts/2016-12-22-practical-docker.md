---
layout: post
title: "Practical Docker for Testing"
date:   2016-12-20 16:29:15 -0600
categories: lab homelab budget testing
---

[Docker](https://www.docker.com/) is pretty nifty. The people behind it seem to get really annoyed if you say it's lightweight VMs, so it's not, but it's kinda like lightweight VMs for applications. Kinda. You can run applications or even entire OSes inside containers. The beauty of it is how throwaway it is. I aw a presentation where someone started an Ubuntu container, rm -rf /'ed themselves, and then restarted the container and it was like nothing ever happened. Amazing stuff.  

I won't pretend I'm getting the most possible out of docker, but the learning curve required to get a whole lot of functionality out of it is really shallow. I have Docker installed in a VM. Like I described earlier, I got it working well enough and then snapshotted it. If you're not paying attention to what you're doing, you can run into a lot of errors and annoyances you don't see coming. 

## Some Use Cases and Ideas

Docker is a really handy way to run things temporarily. There's an excellent implementation of [Mutillidae](https://github.com/citizen-stig/dockermutillidae) that makes for a good test range, base Ubuntu Images, and on and on and on. 

One of my favorites is [Splunk](https://www.splunk.com/). Sometimes, I'll get a large amount of logs or data I need to go through quickly. If you've never used Splunk, it really is an amazing tool. It indexes data quickly and lets you run very complex searches. The problem is the full version is expensive and the free version has limits. There's this happy in-between trial enterprise version. You can index a certain amount of data until it warns you, but it won't stop you. You can injest several gigabytes of data into a single splunk index, do what you need to do, and then get rid of it. If you're snapshotting or pausing the container, you can get that data right back without reindexing it. If you forget, just spin up a new container and you're ready to restart. 

There's another really interesting project called [CyberChef](https://github.com/gchq/CyberChef.git). Some nice fellow at GCHQ spent their free time building it. I'll write a post on it later, but the short of it is you can take data, encrypt, decrypt, encode, decode, alter, arrange, whatever it until you have what you want. It's really an amazing tool.  
