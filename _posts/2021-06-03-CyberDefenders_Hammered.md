---
layout: post
author: remotephone
title:  "Cyberdefenders.org - Hammered Walkthrough"
date:   2021-06-03 23:22:00 -0600
categories: lab homelab training workflow
largeimage: /images/avatar.jpg

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

Download the content, unzip it, and you'll see a bunch of text documents, with auth.log and kern.log making up the bulk of the content. For analyzing text files, I will do some quick exploration with grep and then often go straight into Splunk if the file is big enough. Splunk is a log aggregation and data analysis platform, extensible and flexible. They also provide a very convenient [docker container](https://hub.docker.com/r/splunk/splunk/) to get started.

These logs ended up being enough to handle without going there, some I'm just going to grep and awk and sort my way through them. 

I am working in Windows Subsystem For Linux 2, so first I move the download to a working directory, install p7zip-full (a full featured archive manager), and extract the file. I removed some typos for brevity's sake.

~~~


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

If you look at kern.log, you can see an example of something more interesting happening. Shortly after boot completes, there's a lull in kernel messages, and then we see a USB mounted to sdb1

~~~

May  2 23:05:47 app-1 kernel: : [   47.345208] lo: Disabled Privacy Extensions
May  2 23:05:47 app-1 kernel: : [   47.385335] ip6_tables: (C) 2000-2006 Netfilter Core Team
May  2 23:05:47 app-1 kernel: : [   49.315860] NET: Registered protocol family 17
May  2 23:07:15 app-1 kernel: : [  139.105967] usb 2-1: new high speed USB device using ehci_hcd and address 2
May  2 23:07:15 app-1 kernel: : [  139.313663] usb 2-1: configuration #1 chosen from 1 choice
May  2 23:07:16 app-1 kernel: : [  140.351817] usbcore: registered new interface driver libusual
May  2 23:07:16 app-1 kernel: : [  140.360218] Initializing USB Mass Storage driver...
May  2 23:07:16 app-1 kernel: : [  140.364171] scsi3 : SCSI emulation for USB Mass Storage devices
May  2 23:07:16 app-1 kernel: : [  140.366121] usbcore: registered new interface driver usb-storage
~~~

It looks like this is someone collecting the logs we're looking at. Neat, but not relevant I don't think. 

I'm also creating a folder called `worked` so I can move files when I'm done with them and not retrace my steps. I might create other folders later to help me organzie my work as I go depending on what I need. 

Going through dpkg.log, it looks like things were pretty normal until 4/24/2010 when someone installed a gnome desktop environment. Then we see nmap, exim, some interesting python-packages, (python urlgrabber, python-libxml2, python-rpm, etc) and yum??? `term.log` shows exim installed and starting, the output of t he packages being installed, and other bits we've seen elsewhere.

## Getting Juicy

In daemon.log, we have some interesting bits. This message shows up a few times, but tells us mysql isn't following good practices. 

```
May  2 23:05:54 app-1 /etc/mysql/debian-start[4770]: Checking for insecure root accounts.
May  2 23:05:54 app-1 /etc/mysql/debian-start[4774]: WARNING: mysql.user contains 2 root accounts without password!
May  2 23:05:54 app-1 /etc/mysql/debian-start[4776]: Checking for crashed MySQL tables.
```

Something happened on the 28th, either mysql crashed or the system restarted in a not graceful way.

~~~
Apr 24 20:21:24 app-1 /etc/mysql/debian-start[5427]: WARNING: mysql.user contains 2 root accounts without password!
Apr 24 20:21:24 app-1 /etc/mysql/debian-start[5429]: Checking for crashed MySQL tables.
Apr 28 07:34:24 app-1 mysqld_safe[4711]: started
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4761]: Upgrading MySQL tables if necessary.
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4766]: Looking for 'mysql' in: /usr/bin/mysql
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4766]: Looking for 'mysqlcheck' in: /usr/bin/mysqlcheck
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4766]: This installation of MySQL is already upgraded to 5.0.51a, use --force if you still need to run mysql_upgrade
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4777]: Checking for insecure root accounts.
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4782]: WARNING: mysql.user contains 2 root accounts without password!
Apr 28 07:34:26 app-1 /etc/mysql/debian-start[4784]: Checking for crashed MySQL tables.
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: WARNING: mysqlcheck has found corrupt tables
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: django.experiments_dailyconversionreport
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: warning  : 1 client is using or hasn't closed the table properly
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: django.experiments_dailyconversionreportgoaldata
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: warning  : 1 client is using or hasn't closed the table properly
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]:
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]:  Improperly closed tables are also reported if clients are accessing
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]:  the tables *now*. A list of current connections is below.
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]:
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: +----+------------------+-----------+----+---------+------+-------+------------------+
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: | Id | User             | Host      | db | Command | Time | State | Info             |
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: +----+------------------+-----------+----+---------+------+-------+------------------+
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: | 7  | debian-sys-maint | localhost |    | Query   | 0    |       | show processlist |
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: +----+------------------+-----------+----+---------+------+-------+------------------+
Apr 28 07:34:27 app-1 /etc/mysql/debian-start[5032]: Uptime: 3  Threads: 1  Questions: 89  Slow queries: 0  Opens: 76  Flush tables: 1  Open tables: 64  Queries per second avg: 29.667
Apr 28 09:35:28 app-1 mysqld_safe[6494]: ended
~~~


It doesn't look like we have any audit logs for the system, but we can typically see privileged command execution in auth.log. You'll see users sudoing to execute commands and that might give us an idea of interesting things. One interesting section is this 

~~~
Apr 19 18:15:10 app-1 sudo:     user3 : TTY=pts/0 ; PWD=/opt/software/web/templates/input ; USER=root ; COMMAND=/bin/su
Apr 19 23:02:37 app-1 sudo:     root : TTY=pts/2 ; PWD=/root ; USER=root ; COMMAND=/usr/bin/apt-get install build-essential
Apr 19 23:21:08 app-1 sudo:      dhg : user NOT in sudoers ; TTY=pts/1 ; PWD=/home/dhg/psybnc-linux/psybnc ; USER=root ; COMMAND=alien lsb-build-4.0.9-2.src.rpm
Apr 19 23:24:30 app-1 sudo:      dhg : user NOT in sudoers ; TTY=pts/1 ; PWD=/home/dhg/psybnc-linux/psybnc ; USER=root ; COMMAND=root
~~~ 

[Alien](https://wiki.debian.org/Alien) is a tool to run rpms (and other typically non-compatible packages) on a debian based system. In this case, we see [psybnc](https://servermania.com/kb/articles/how-to-install-and-setup-psybnc/) and [eggdrop](https://www.eggheads.org/) as two interseting packages under the dhg user, both IRC related. 

Focusing on SSH logs in auth.log, we can filter out failed logins with something like this `grep sshd worked/auth.log | grep -vE "Failed|error|Invalid|failure|unknown"` and see what's left. To look at accepted logins, we can do quick stats on them with something like this:

~~~
┌─(~/working)─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(22:38:59)──> grep sshd worked/auth.log| grep Accepted | awk '{print "logins by " $9 " from " $11}'  | sort | uniq -c | sort -rk 4                                      ──(Thu,Jun03)─┘
      1 logins by user3 from 65.195.182.120
      1 logins by user3 from 208.80.69.74
      2 logins by user3 from 208.80.69.69
      3 logins by user3 from 192.168.126.1
      4 logins by user3 from 10.0.1.4
     13 logins by user3 from 10.0.1.2
      5 logins by user2 from 71.132.129.212
     22 logins by user1 from 76.191.195.140
      1 logins by user1 from 67.164.72.181
      6 logins by user1 from 65.88.2.5
      1 logins by user1 from 65.195.182.120
      6 logins by user1 from 208.80.69.74
      1 logins by user1 from 208.80.69.70
      1 logins by user1 from 166.129.196.88
      1 logins by root from 94.52.185.9
      1 logins by root from 61.168.227.12
      1 logins by root from 222.66.204.246
      1 logins by root from 222.169.224.197
      4 logins by root from 219.150.161.20
      1 logins by root from 201.229.176.217
      1 logins by root from 193.1.186.197
      1 logins by root from 190.167.74.184
      1 logins by root from 190.167.70.87
      3 logins by root from 190.166.87.164
      4 logins by root from 188.131.23.37
      1 logins by root from 188.131.22.69
      1 logins by root from 151.82.3.201
      1 logins by root from 151.81.205.100
      1 logins by root from 151.81.204.141
      2 logins by root from 122.226.202.12
      2 logins by root from 121.11.66.70
      1 logins by root from 10.0.1.2
      1 logins by fido from 94.52.185.9
      2 logins by dhg from 190.167.74.184
     20 logins by dhg from 190.166.87.164
~~~

Looks like root logins went pretty wild, and all users have a few login locations. Not looking good. Searching for user creations in the auth.log, we might find something like a timeline

~~~
┌─(~/working)────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(22:58:48)──> grep "new user" worked/auth.log                                                                      ──(Thu,Jun03)─┘
Mar 16 08:12:13 app-1 useradd[4692]: new user: name=user4, UID=1001, GID=1001, home=/home/user4, shell=/bin/bash
Mar 16 08:12:38 app-1 useradd[4703]: new user: name=user1, UID=1001, GID=1001, home=/home/user1, shell=/bin/bash
Mar 16 08:12:55 app-1 useradd[4711]: new user: name=user2, UID=1002, GID=1002, home=/home/user2, shell=/bin/bash
Mar 16 08:25:22 app-1 useradd[4845]: new user: name=sshd, UID=104, GID=65534, home=/var/run/sshd, shell=/usr/sbin/nologin
Mar 18 10:15:42 app-1 useradd[5393]: new user: name=Debian-exim, UID=105, GID=114, home=/var/spool/exim4, shell=/bin/false
Mar 18 10:18:26 app-1 useradd[6966]: new user: name=mysql, UID=106, GID=115, home=/var/lib/mysql, shell=/bin/false
Apr 19 22:38:00 app-1 useradd[2019]: new user: name=packet, UID=0, GID=0, home=/home/packet, shell=/bin/sh
Apr 19 22:45:13 app-1 useradd[2053]: new user: name=dhg, UID=1003, GID=1003, home=/home/dhg, shell=/bin/bash
Apr 24 19:27:35 app-1 useradd[1386]: new user: name=messagebus, UID=108, GID=117, home=/var/run/dbus, shell=/bin/false
Apr 25 10:41:44 app-1 useradd[9596]: new user: name=fido, UID=0, GID=1004, home=/home/fido, shell=/bin/sh
Apr 26 04:43:15 app-1 useradd[20115]: new user: name=wind3str0y, UID=1004, GID=1005, home=/home/wind3str0y, shell=/bin/bash
~~~

Packet was created with UID and GID 0, very suspicious. dhg was created next, so let's look around those times, 22:38:00 on April 19. 
~~~
┌─(~/working)────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:01:12)──> grep Accepted worked/auth.log | grep root                                                            ──(Thu,Jun03)─┘
Mar 29 13:27:26 app-1 sshd[21556]: Accepted password for root from 10.0.1.2 port 51784 ssh2
Apr 19 05:41:44 app-1 sshd[8810]: Accepted password for root from 219.150.161.20 port 51249 ssh2
Apr 19 05:42:27 app-1 sshd[9031]: Accepted password for root from 219.150.161.20 port 40877 ssh2
Apr 19 05:55:20 app-1 sshd[12996]: Accepted password for root from 219.150.161.20 port 55545 ssh2
Apr 19 05:56:05 app-1 sshd[13218]: Accepted password for root from 219.150.161.20 port 36585 ssh2
Apr 19 10:45:36 app-1 sshd[28030]: Accepted password for root from 222.66.204.246 port 48208 ssh2
Apr 19 11:03:44 app-1 sshd[30277]: Accepted password for root from 201.229.176.217 port 54465 ssh2
Apr 19 11:15:26 app-1 sshd[30364]: Accepted password for root from 190.167.70.87 port 49497 ssh2
Apr 19 22:37:24 app-1 sshd[2012]: Accepted password for root from 190.166.87.164 port 50753 ssh2
Apr 19 22:54:06 app-1 sshd[2149]: Accepted password for root from 190.166.87.164 port 51101 ssh2
Apr 19 23:02:25 app-1 sshd[2210]: Accepted password for root from 190.166.87.164 port 51303 ssh2
Apr 20 06:13:03 app-1 sshd[26712]: Accepted password for root from 121.11.66.70 port 33828 ssh2
~~~


### WWW logs

Digging though these, there's really not a lot. I checked the subnet from the error logs, didn't see it doing much of anything interesting in the logs. 

~~~
193.109.122.56 - - [20/Apr/2010:00:00:01 -0700] "CONNECT 72.51.18.254:6677 HTTP/1.0" 301 - "-" "pxyscand/2.1" oFs91QoAAQ4AAAQFlmcAAAAL 1213441
193.109.122.33 - - [20/Apr/2010:00:00:05 -0700] "GET http://72.51.18.254:6677 HTTP/1.0" 400 401 "-" "-" - 29798270
193.109.122.57 - - [23/Apr/2010:10:34:20 -0700] "CONNECT 72.51.18.254:6677 HTTP/1.0" 301 - "-" "pxyscand/2.1" 1l730AoAAQ4AACkeBjsAAAAB 156351
193.109.122.18 - - [23/Apr/2010:10:34:12 -0700] "GET http://72.51.18.254:6677 HTTP/1.0" 400 401 "-" "-" - 29802301
193.109.122.52 - - [24/Apr/2010:18:51:52 -0700] "CONNECT 72.51.18.254:6677 HTTP/1.0" 301 - "-" "pxyscand/2.1" 54ydwAoAAQ4AAEHECacAAAAB 571386
193.109.122.15 - - [24/Apr/2010:18:51:54 -0700] "GET http://72.51.18.254:6677 HTTP/1.0" 400 401 "-" "-" - 29801077
~~~

pxyscand is a IRC proxy scanner, so that's cool, but not super useful. Since we saw some IRC related packages getting installed earlier in the logs we looked at, we can probably assume this is a symptom and not the cause. Looking at user agents in aggregate, we see these results

~~~
┌─(~/working)──────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(22:56:00)──> cat www-access.log| awk -F "\"" '{print $(NF-1)}' | sort | uniq -c | sort -rn                                      ──(Thu,Jun03)─┘
    272 Apple-PubSub/65.12.1
     20 Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6; en-US; rv:1.9.2.3) Gecko/20100401 Firefox/3.6.3
     18 WordPress/2.9.2; http://www.domain.org
     15 Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
     13 Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)
      8 -
      6 Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_2; en-us) AppleWebKit/531.21.8 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10
      4 Mozilla/4.0 (compatible; NaverBot/1.0; http://help.naver.com/customer_webtxt_02.jsp)
      3 pxyscand/2.1
      3 Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/532.5 (KHTML, like Gecko) Chrome/4.1.249.1059 Safari/532.5
      1 Mozilla/5.0 (Windows; U; Windows NT 5.1; es-ES; rv:1.9.0.19) Gecko/2010031422 Firefox/3.0.19
      1 Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/532.5 (KHTML, like Gecko) Chrome/4.1.249.1045 Safari/532.5
      1 Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_2; en-us) AppleWebKit/531.22.7 (KHTML, like Gecko) Version/4.0.5 Safari/531.22.7
~~~

Again, cool, but nothing I that screams cause of incident. SSH seems most suspicious and the most likely original point of entry because we don't see any other obvious places of entry followed by signs of modification to the SSH service or user creation. It looks like they really just logged in as root.  :|

## What We Know So far

So the story so far seems this was booted up, an app was running on the system probably django with a mysql backend, users (authorized or not) were installing cross platform packages, we have at least 3 users (user1, user3, dhg) running commands, we know people logged in from all over the place, www-access.log has a lot in it, but the entry point seems to SSH and it's likely 190.166.87.164 was involved based on timestamps.

## Questions

### Which service did the attackers use to gain access to the system?	

`ssh` is the answer here, see above.

### What is the operating system version of the targeted system? (one word)	

I've seen the Ubuntu string, so a simple grep in messages gets us the answer, `4.2.4-1ubuntu3`

~~~
┌─(~/working/worked)─────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:03:47)──> grep Ubuntu messages                                                                                 ──(Thu,Jun03)─┘
Apr 28 07:34:22 app-1 kernel: : [    0.000000] Linux version 2.6.24-26-server (buildd@crested) (gcc version 4.2.4 (Ubuntu 4.2.4-1ubuntu3)) #1 SMP Tue Dec 1 18:26:43 UTC 2009 (Ubuntu 2.6.24-26.64-server)
~~~

### What is the name of the compromised account?	

It seems to be root, based off the logs and timestamps above.

### Consider that each unique IP represents a different attacker. How many attackers were able to get access to the system?	

Their answer was 6, but I had to guess. I don't know if that's right?? I used the following syntax and got 17 unique IPs back, excluding the one local IP.

~~~
┌─(~/working)────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:13:47)──> grep sshd worked/auth.log| grep "Accepted password for root" | awk '{print $11}'  | sort | uniq -c | sort -n
      1 10.0.1.2
      1 151.81.204.141
      1 151.81.205.100
      1 151.82.3.201
      1 188.131.22.69
      1 190.167.70.87
      1 190.167.74.184
      1 193.1.186.197
      1 201.229.176.217
      1 222.169.224.197
      1 222.66.204.246
      1 61.168.227.12
      1 94.52.185.9
      2 121.11.66.70
      2 122.226.202.12
      3 190.166.87.164
      4 188.131.23.37
      4 219.150.161.20
┌─(~/working)────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:14:00)──> grep sshd worked/auth.log| grep "Accepted password for root" | awk '{print $11}'  | sort | uniq -c | wc -l
18
~~~

### Which attacker's IP address successfully logged into the system the most number of times?	

We can count them with a command like this `grep sshd worked/auth.log| grep Accepted | grep root | awk '{print $11}'  | sort | uniq -c | sort -n`. This gives us 219.150.161.20.

### How many requests were sent to the Apache Server?	

The answer is 365, which is the count of lines in www-access.log, but I think it should also include www-media.log at least, no?

### How many rules have been added to the firewall?	

6 rules were added.  My command gives 8, but that includes duplicates.

~~~
┌─(~/working)─────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:17:26)──> grep -r ufw * | grep allow | awk '{print $NF}' | sort -u | uniq -c                                                ──(Thu,Jun03)─┘
      1 113
      1 113/Identd
      1 113/identd
      1 22
      1 2685/tcp
      1 2685/telnet
      1 53
      1 telnet
┌─(~/working)─────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:17:33)──> grep -r ufw * | grep allow | awk '{print $NF}' | sort -u | wc -l                                                  ──(Thu,Jun03)─┘
8
~~~


### One of the downloaded files to the target system is a scanning tool. Provide the tool name.	

We saw `nmap` above in the installed packages file, dpkg.log. Easy

### When was the last login from the attacker with IP 219.150.161.20? Format: MM/DD/YYYY HH:MM:SS AM	

Grep and convert the time to the requested format, badabing badaboom
~~~
┌─(~/working)─────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:20:36)──> grep 219.150.161.20 worked/auth.log | grep Accepted                                                               ──(Thu,Jun03)─┘
Apr 19 05:41:44 app-1 sshd[8810]: Accepted password for root from 219.150.161.20 port 51249 ssh2
Apr 19 05:42:27 app-1 sshd[9031]: Accepted password for root from 219.150.161.20 port 40877 ssh2
Apr 19 05:55:20 app-1 sshd[12996]: Accepted password for root from 219.150.161.20 port 55545 ssh2
Apr 19 05:56:05 app-1 sshd[13218]: Accepted password for root from 219.150.161.20 port 36585 ssh2
~~~

### The database displayed two warning messages, provide the most important and dangerous one.	

We saw this earlier poking around and know it's the mysql user alert, `mysql.user contains 2 root accounts without password!`

### Multiple accounts were created on the target system. Which one was created on Apr 26 04:43:15?	

Easiest way for this one is just grep the time string, it should be unique enough to give us the answer.

~~~
┌─(~/working)─────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/2)─┐
└─(23:20:43)──> grep 04:43:15 worked/auth.log                                                                                     ──(Thu,Jun03)─┘
Apr 26 04:43:15 app-1 groupadd[20114]: new group: name=wind3str0y, GID=1005
Apr 26 04:43:15 app-1 useradd[20115]: new user: name=wind3str0y, UID=1004, GID=1005, home=/home/wind3str0y, shell=/bin/bash
~~~

### Few attackers were using a proxy to run their scans. What is the corresponding user-agent used by this proxy?	

We saw this in the logs too. We review the notes above and see its `pxyscand/2.1`.



## OK All done

That was an interesting one. Not sure what was up with that one answer where we got different results, but otherwise interesting to look at. Honestly, I expected something wacky in the logs and thought a root brute force login would be too simple, but sometimes it really is just that simple. All together, to write this up and work the exercise, it took me a little under 2 hours. I think I would have been quicker if I went straight to the questions, but there's some value in just poking around not knowing what you're looking for. 

Thanks again to CyberDefenders and The Honeynet Project for putting this together. 