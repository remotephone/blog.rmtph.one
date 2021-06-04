---
layout: post
author: remotephone
title:  "Cyberdefenders.org - Hammered Walkthrough"
date:   2021-06-03 23:22:00 -0600
categories: lab homelab training workflow
---

# Hammered Walthrough

Back again with another Cyberdefenders.org challenge, this time it's [Hammered](https://cyberdefenders.org/labs/42). Gonna do this a little different once since my previous post about Mac forensics was fine, but it felt more like "do this, do that, this is the answer" and didn't give good insight into my process. 

Based of the description, we have a honeypot that may or may not have been compromised. 

~~~
This challenge takes you into the world of virtual systems and confusing log data. In this challenge, figure out what happened to this webserver honeypot using the logs from a possibly compromised server.

Thanks, th3c0rt3x for reviewing the challenge.
~~~

What I'll do this time is download the content, set up an analysis environment, draw some conclusions, and then see what questions I answered and then go back and fill in the blanks.

## Set up

Download the content, unzip it, and you'll see a bunch of text documents, with auth.log and kern.log making up the bulk of the content. For analyzing text files, I will do some quick exploration with grep and then often go straight into Splunk. Splunk is a log aggregation and data analysis platform, extensible and flexible. They also provide a very convenient [docker container](https://hub.docker.com/r/splunk/splunk/) to get started.

To get the container running, I already have docker installed (I'm on Windows using the WSL2 backend, but whatever you have works) and ran the container with this command `$ docker run -d -p 8000:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunk splunk/splunk:latest`. I am using a throwaway password since this will only be exposed locally and I'll terminate it when I am done.

I am working in Windows Subsystem For Linux 2, so first I move the download to a working directory, install p7zip-full (a full featured archive manager), and extract the file. I removed some typos for brevity's sake.

~~~
$ docker run -d -p 8000:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=<password>" --name splunk splunk/splunk:latest


┌─(/mnt/c/Users/computer)───────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:31:33)──> mv Downloads/c26-Hammered.zip ~/                                                        ──(Thu,Jun03)─┘
┌─(/mnt/c/Users/computer)───────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:32:13)──> cd ~                                                                                    ──(Thu,Jun03)─┘
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:32:15)──> mkdir working                                                                           ──(Thu,Jun03)─┘
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:32:17)──> mv c26-Hammered.zip working                                                             ──(Thu,Jun03)─┘
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:32:24)──> cd working                                                                              ──(Thu,Jun03)─┘
┌─(~/working)───────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:33:50)──> sudo apt install p7zip-full                                                       130 ↵ ──(Thu,Jun03)─┘
Reading package lists... Done
Building dependency tree
Reading state information... Done
Suggested packages:
  p7zip-rar
The following NEW packages will be installed:
  p7zip-full
0 upgraded, 1 newly installed, 0 to remove and 94 not upgraded.
<SNIP>
Unpacking p7zip-full (16.02+dfsg-7build1) ...
Setting up p7zip-full (16.02+dfsg-7build1) ...
Processing triggers for man-db (2.9.1-1) ...
┌─(~/working)───────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:34:23)──> 7z e c26-Hammered.zip                                                                   ──(Thu,Jun03)─┘

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=C.UTF-8,Utf16=on,HugeFiles=on,64 bits,4 CPUs Intel(R) Core(TM) i5-5300U CPU @ 2.30GHz (306D4),ASM,AES-NI)

Scanning the drive for archives:
1 file, 965944 bytes (944 KiB)

Extracting archive: c26-Hammered.zip
--
Path = c26-Hammered.zip
Type = zip
Physical Size = 965944


Enter password (will not be echoed):
Everything is Ok

Folders: 4
Files: 18
Size:       13913116
Compressed: 965944
┌─(~/working)───────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(21:34:39)──> ls -aslh                                                                                ──(Thu,Jun03)─┘
total 15M
4.0K drwxr-xr-x  6 computer computer 4.0K Jun  3 21:34 .
4.0K drwxr-xr-x 12 computer computer 4.0K Jun  3 21:34 ..
4.0K drwxr-xr-x  2 computer computer 4.0K Jul  3  2010 Hammered
4.0K drwxr-xr-x  2 computer computer 4.0K Jul  3  2010 apache2
4.0K drwxr-xr-x  2 computer computer 4.0K Jul  3  2010 apt
9.9M -rw-r-----  1 computer computer 9.9M Jul  3  2010 auth.log
<SNIP>
~~~

The syntax `7z e` will extract everything to the folder you're in, without preserving paths. This is good or bad, I don't care since this is just an exercise and thats the first flag I used from memory, but if preserving that folder structure is important, use the `7z x` syntax. Otherwise, all looks good.

## Initital Impressions

First I look at the small files I can read visually just to get anything out of the way I can quickly. That's user.log, checkfs, checkroot, fontconfig.log, and www-error.log. The www-error.log has some lines worth noting because of requests from a subnet (193.109.122.0/24), but nothing I can do with them now.

~~~
[Tue Apr 20 00:00:35 2010] [error] [client 193.109.122.33] request failed: error reading the headers
[Fri Apr 23 10:34:42 2010] [error] [client 193.109.122.18] request failed: error reading the headers
[Sat Apr 24 18:52:24 2010] [error] [client 193.109.122.15] request failed: error reading the headers
~~~

Next, I go to dmesg. There's two, dmesg.0 (last session the system booted for) and dmesg (current session at time logs were collected). Nothing too interesting. A shortcut I'll use here is look at the timestamp. If someone mounts a drive or does something once the system has booted, you'll often see older timestamps, but the fact that both logs end before 60 seconds of uptimes means we're probably not gonna find anything super interesting in them.  