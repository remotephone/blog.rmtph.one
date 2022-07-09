---
layout: post
author: remotephone
title:  "HoneyPot Forensics"
date:   2021-12-08 22:49:00 -0600
categories: [lab, workflow, forensics, malware]
largeimage: /images/avatar.jpg

---

# A Helpful Tweet

I saw this [tweet](https://twitter.com/SecShoggoth/status/1468740840759697420?s=20) and thought it would be interesting to take a look around. I will pick something specific in the evidence, take a look at it, and write about what I find.

## Some Stumbles

I'm on a M1 macbook air and tried to install volality to look at the memory image, no go there. I ran into a few errors, the big one seeming to be

```shell
<snip>
[pipenv.exceptions.InstallError]:   In file included from leechcorepyc.c:21:
[pipenv.exceptions.InstallError]:   ./oscompatibility.h:28:10: fatal error: 'byteswap.h' file not found
[pipenv.exceptions.InstallError]:   #include <byteswap.h>
[pipenv.exceptions.InstallError]:            ^~~~~~~~~~~~
[pipenv.exceptions.InstallError]:   1 error generated.
[pipenv.exceptions.InstallError]:   error: command '/usr/bin/clang' failed with exit code 1
[pipenv.exceptions.InstallError]:   ----------------------------------------
<snip>
```

I did some googling around and it looks like this is probably something to do with running this on the wrong architecture and quite a bit out of my area of expertise. I didn't spend too much time on it tbh, I'd probably save myself a lot of trouble and just run this in a virtual machine on some different hardware if I really wanna do memory analysis.

## Moving Along

Let's take a look at running processes. I started with the file at `./live_response/process/running_processes_full_paths.txt`. This was collected with <https://github.com/tclahr/uac> and I've never worked with it but it seems pretty straight forward.

A quick browse shows two processes marked as deleted.

```shell
└─(23:24:54)──> grep deleted process/running_processes_full_paths.txt                                   ──(Wed,Dec08)─┘
lrwxrwxrwx 1 daemon           daemon           0 Dec  8 18:51 /proc/24330/exe -> /tmp/agettyd (deleted)
lrwxrwxrwx 1 root             root             0 Dec  8 18:51 /proc/609/exe -> / (deleted)
```

So we got two proceses that might be interesting to look at. An actor will get a process running on a linux system and then delete it off the disk to complicate investigations, but you can do a lot with the proc virtual filesystem to recover it. [Craig Rowland](https://twitter.com/CraigHRowland) of Sandfly security talks a lot about this and I've learned a lot reading his posts.

I'm picking 24330.

## 24330

If we look in the directory we can see these files. These are extractions of files in the proc filesystem, we'd typically find these in `/proc/24330/` on the host.

```shell
┌─(~/gits/forensics/image/live_response)──────────────────────────────────────────────────────────────────────────────(computer@MacBook-Air:s000)─┐
└─(23:30:59)──> ls process/proc/24330                                                                                               ──(Wed,Dec08)─┘
cmdline.txt comm.txt    environ.txt fd.txt      maps.txt    strings.txt.gz
```

Let's check open file descriptors, these are where processes can take inputs and send outputs. We see output to /dev/null is typically used to ignore and silence error output and we got a socket open.

```shell
┌─(~/gits/forensics/image/live_response)──────────────────────────────────────────────────────────────────────────────(computer@MacBook-Air:s000)─┐
└─(23:33:48)──> cat process/proc/24330/fd.txt                                                                                       ──(Wed,Dec08)─┘
total 0
dr-x------ 2 daemon daemon  0 Dec  8 19:03 .
dr-xr-xr-x 9 daemon daemon  0 Dec  8 19:02 ..
lr-x------ 1 daemon daemon 64 Dec  8 19:03 0 -> /dev/null
l-wx------ 1 daemon daemon 64 Dec  8 19:03 1 -> /dev/null
lr-x------ 1 daemon daemon 64 Dec  8 19:03 10 -> anon_inode:inotify
lrwx------ 1 daemon daemon 64 Dec  8 19:03 11 -> anon_inode:[eventfd]
lr-x------ 1 daemon daemon 64 Dec  8 19:03 12 -> /dev/null
lrwx------ 1 daemon daemon 64 Dec  8 19:03 14 -> socket:[112553823]
l-wx------ 1 daemon daemon 64 Dec  8 19:03 2 -> /dev/null
lrwx------ 1 daemon daemon 64 Dec  8 19:03 3 -> anon_inode:[eventpoll]
lr-x------ 1 daemon daemon 64 Dec  8 19:03 4 -> pipe:[102136945]
l-wx------ 1 daemon daemon 64 Dec  8 19:03 5 -> pipe:[102136945]
lr-x------ 1 daemon daemon 64 Dec  8 19:03 6 -> pipe:[102136943]
l-wx------ 1 daemon daemon 64 Dec  8 19:03 7 -> pipe:[102136943]
lrwx------ 1 daemon daemon 64 Dec  8 19:03 8 -> anon_inode:[eventfd]
lrwx------ 1 daemon daemon 64 Dec  8 19:03 9 -> anon_inode:[eventfd]
```

Let's see if we can see what that socket is for.

```shell
┌─(~/gits/forensics/image/live_response)──────────────────────────────────────────────────────────────────────────────(computer@MacBook-Air:s000)─┐
└─(23:36:01)──> grep 24330 network/*                                                                                                ──(Wed,Dec08)─┘
network/lsof_-nPli.txt:agettyd   24330        1   14u  IPv4 112585467      0t0  TCP 10.0.0.4:44214->107.178.104.10:443 (ESTABLISHED)
```

Look up that IP and we can find some intel on it - <https://otx.alienvault.com/indicator/ip/107.178.104.10>. I'm getting real suspicious. Let's look at the environment variables for the process.

```shell
┌─(~/gits/forensics/image/live_response)──────────────────────────────────────────────────────────────────────────────(computer@MacBook-Air:s000)─┐
└─(23:36:22)──> cat process/proc/24330/environ.txt                                                                                  ──(Wed,Dec08)─┘
CONTENT_TYPE=application/x-www-form-urlencoded
GATEWAY_INTERFACE=CGI/1.1
LD_LIBRARY_PATH=/usr/lib
SHLVL=1
REMOTE_ADDR=5.2.72[.]226
QUERY_STRING=
OLDPWD=/tmp
HTTP_USER_AGENT=curl/7.79.1
REMOTE_PORT=47374
DOCUMENT_ROOT=/usr/share/apache2/default-site/htdocs
HTTP_ACCEPT=*/*
SERVER_SIGNATURE=
CONTENT_LENGTH=103
CONTEXT_DOCUMENT_ROOT=/usr/lib/cgi-bin/
SCRIPT_FILENAME=/bin/bash
HTTP_HOST=13.82.150[.]103
REQUEST_URI=/cgi-bin/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/bin/bash
_=/bin/sh
SERVER_SOFTWARE=Apache/2.4.49 (Unix)
REQUEST_SCHEME=http
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/tmp:/bin:/usr/bin:/usr/local/bin:/usr/sbin:/tmp:/bin:/usr/bin:/usr/local/bin:/usr/sbin
SERVER_PROTOCOL=HTTP/1.1
LANG=C.UTF-8
REQUEST_METHOD=POST
SERVER_ADMIN=you@example.com
SERVER_ADDR=10.0.0.4
CONTEXT_PREFIX=/cgi-bin/
PWD=/tmp
LC_ALL=C.UTF-8
SERVER_PORT=80
SCRIPT_NAME=/cgi-bin/../../../../../../../bin/bash
SERVER_NAME=13.82.150[.]103
```

I defanged the public IPs above but otherwise left it unmodified. The 13.x.x.x IP is probably the IP the VM lived on in Azure. `5.2.72[.]226` has [intel hits](https://otx.alienvault.com/indicator/ip/5.2.72.226) for tor nodes but hard to say if thats right in this context. Glancing through strings its easy enough to see what this binary is and does.

```shell
┌─(~/gits/forensics/image/live_response)──────────────────────────────────────────────────────────────────────────────(computer@MacBook-Air:s000)─┐
└─(23:46:50)──> cat process/proc/24330/strings.txt | less                                                                           ──(Wed,Dec08)─┘
..........SNIP..........
XMRig 6.16.1
 built on Nov 29 2021 with GCC
failed to export hwloc topology.
hwloc topology successfully exported to "%s"
```

e83658008d6d9dc6fe5dbb0138a4942b is the hash, but it's unknown to Virus Total. If I were on an x86 CPU, at this point we'd take the memory image, extract the EXE and see if we can confirm if it's what we think it is,an xmr miner running and using up CPU.

If we look at `top`, we can be pretty sure we're right.

```shell
┌─(~/gits/forensics/image/live_response)──────────────────────────────────────────────────────────────────────────────(computer@MacBook-Air:s000)─┐
└─(23:56:39)──> cat process/top_-b_-n1.txt                                                                                          ──(Wed,Dec08)─┘
top - 18:52:03 up 60 days,  3:45,  1 user,  load average: 11.48, 10.20, 9.31
Tasks: 138 total,   3 running,  79 sleeping,   0 stopped,   0 zombie
%Cpu(s): 20.6 us,  7.1 sy,  0.2 ni, 68.4 id,  1.4 wa,  0.0 hi,  2.3 si,  0.0 st
KiB Mem :   939060 total,   131164 free,   625840 used,   182056 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   158732 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
24330 daemon    20   0  308944 266044      0 S 26.4 28.3   2442:03 agettyd
 7777 root      20   0   12644   1688    236 R 22.6  0.2 370:09.57 hv_kvp_daem+
27968 root      20   0  387384  20404   2648 R 13.2  2.2 270:58.21 python3
 8295 root      20   0   41012   3508   2952 R  9.4  0.4   0:00.15 top
 ```

## All Done

So that's neat. This process took about 45 minutes from getting started writing this to getting to this paragraph while I enjoyed a movie. I'd recommend you read some of Craig's blog posts and look into Hal Pomeranz's course on [linux forensics](https://twitter.com/hal_pomeranz/status/1242539945144745986?s=20) if you're interested in learning and doing more of this stuff. Thanks for reading.
