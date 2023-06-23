---
layout: post
author: remotephone
title:  "CyberDefenders - Hawkeye"
date:   2023-06-22 20:33:33 -0500
categories: [lab, workflow, networking, dfir, pcap]
largeimage: /images/avatar.jpg

---

# Hawkeye

Well I'm back after a year. I've been busy building and doing things at work and in life, so haven't had much time to post. I figured I'd get back into the groove of it with a CyberDefenders challenge and then post more stuff as I ramp back up. Their site has gone through a lot of changes, there's more content, more paid access, and generally looks like they've come quite a long way since I last visted. It's exciting to see Blue team resoruces come along and I'm looking forward to seeing what they do next.

This is a network pcap analysis challenge from Cyber Defenders. To get set up, I installed wireshark on an MacOS M1 system. I did not have to install any of the BPF extensions to analyze the pcap. Some of these questions have aged out, so its impossible to answer the questions as the original exercise intended. I marked those pretty clearly and ended up just exhausting the hints where I had to or moving on and finding the answer later.

And away we go!

## Question 1 - How many packets does the capture have?

Once open, you can simply scroll down and see the numbrered packets in the `Packet List` (the top half of your screen) to get the last number the total count of packets. My numbering starts at 1 but if yours starts at zero for some reason, adjust accordingly  .

## Question 2 - At what time was the first packet captured?

To make this easy for yourself, go to View > Time Display Format > `UTC Date and Time of Day`. You'll still have to massage it a bit to fit the required format, but that's what you need. 

## Question 3 - What is the duration of the capture?

Probably a thousand ways to do this one, but I grabbed the epoch time from each packet (Double-click the packet, expand the Frame section, right click Epoch Time and copy the value) and then used python to do the math. 

```py
>>> import datetime
>>> str(datetime.timedelta(seconds=3821))
'1:03:41'
```

## Question 4 - What is the most active computer at the link level?

For this one, I went to Statistics > IPv4 Statistics > All Addresses. Apply a filter, either `eth` or `not tcp and not udp` works too 10.4.10.132 comes to the top. Find a packet where its the source or destination, find the respective mac for that IP, and then that's your answer.

## Question 5 - Manufacturer of the NIC of the most active system at the link level?

With the mac, you can do a mac vendor lookup anywhere like https://macvendors.com/ and get the answer. Make sure you get the right format, they expect a specific special character.

## Question 6 - Where is the headquarter of the company that manufactured the NIC of the most active computer at the link level?

Just google the HQ of the company and provide the city. 

## Question 7 - The organization works with private addressing and netmask /24. How many computers in the organization are involved in the capture?

Counting up the 10. addresses will yield the wrong answer. There's a subset of the IPs you need to exclude because they're not actually devices on the network. Take your count and substract that amount and you'll have your answer.

## Question 8 - What is the name of the most active computer at the network level?

You should be able to see NetBIOS traffic that includes a windows systems name. Use the filter `udp.port == 137` and see what name the most active talker is announcing themselves as. 

## Question 9 - What is the IP of the organization's DNS server?

Use the filter `udp.port == 53` and see who is answering questions. 

## Question 10 - What domain is the victim asking about in packet 204?

Go to that packet and see what it's looking for. Under Queries > Name you'll find the value.

## Question 11 - What is the IP of the domain in the previous question?

If you follow the UDP stream its overkill since its just two packets, but copy the answer and there you go. 

## Question 12 - Indicate the country to which the IP in the previous section belongs.

Use a geolocation service to get this answer and hope the IP hasn't been reassigned to a different country.

## Question 13 - What operating system does the victim's computer run?

Find a user agent from a system thats doing malware looking traffic. The packet that tipped me off included a download of a packet that began with `MZ`. Using the user agent, you can get the OS.

## Question 14 - What is the name of the malicious file downloaded by the accountant?

I have Copilot running in the VS Code session of this browser and I think its interesting how its autocompleting the questions. As I answer one, I type `## Question 14` and switch windows to copy the question text and usually copilot completes with a reasonable question. Some have been pretty wacky, but this one was `What is the name of the malicious file downloaded by the accountant?` and I thought that was pretty cool.

Anyway, look at the URL in the packet for that request and you'll see the file name.

## Question 15 - What is the md5 hash of the downloaded file?

This time it tried to autocomplete it with `What is the name of the malicious file downloaded by the accountant?` which is a good example of how the AI is predicting the question, it doesn't know it. 

To get the hash, go to File > Export Objects > HTTP and export the file. This is live malware, so be careful. md5sum it and boom. 

## Question 16 - What is the name of the malware according to Malwarebytes?

You can use virustotal to search up the md5 and see what they call it - https://www.virustotal.com/gui/file/62099532750dad1054b127689680c38590033fa0bdfa4fb40c7b4dcb2607fb11 - It seems to have changed since this challenge was written as they now call it `Inject.Exploit.Shellcode.DDS`. [Here](https://www.malwarebytes.com/blog/detections/spyware-hawkeyekeylogger) is their blog post for it, but I'm not sure how to connect the hash to their name unless you connect specific IOCs back to their report. :shrug:

## Question 17 - What software runs the webserver that hosts the malware?

We can go back to the pcap and see the response headers for the download request. It's a specific flavor of webserver software, check the `Server` header

## Question 18 - What is the public IP of the victim's computer?

To figure this one out, I had to follow TCP streams. You'll see one that will contain a response of the querying systems IP. Look for TCP streams immediately after the malware is downloaded. Pay attention to the domain and it'll jump out at you which one you want to see the response for.

## Question 19 - In which country is the email server to which the stolen information is sent?

For this, we can simply use `smtp` as the filter to get all mail traffic. Make sure its coming from our known victim and see wher its going. Make sure you geolocate the IP and then put it in the expected format.

## Question 20 - What is the domain's creation date to which the information is exfiltrated?

So first we need the domain. If we have the IP, we can use a filter like `dns.a == <ip>` to see the queries for that IP answer.

The current answer just doesn't work here 

```
└─(19:55:00 on master ✭)──> whois macwinlogistics.in                                                                                                                 ──(Thu,Jun22)─┘
% IANA WHOIS server
% for more information on IANA, visit http://www.iana.org
% This query returned 1 object

refer:        whois.registry.in

domain:       IN

organisation: National Internet Exchange of India
address:      6C, 6D, 6E Hansalaya Building 15, Barakhamba Road
address:      New Delhi 110 001
address:      India

contact:      administrative
name:         Rajiv Kumar
organisation: National Internet Exchange of India
address:      6C, 6D, 6E Hansalaya Building 15, Barakhamba Road
address:      New Delhi 110 001
address:      India
phone:        +91 11 48202011
fax-no:       +91 11 48202013
e-mail:       registry@nixi.in

contact:      technical
name:         Rajiv Kumar
organisation: National Internet Exchange of India
address:      6C, 6D, 6E Hansalaya Building 15, Barakhamba Road
address:      New Delhi 110 001
address:      India
phone:        +91 11 48202011
fax-no:       +91 11 48202013
e-mail:       rajiv@nixi.in

nserver:      NS1.REGISTRY.IN 2001:dcd:1:0:0:0:0:12 37.209.192.12
nserver:      NS2.REGISTRY.IN 2001:dcd:2:0:0:0:0:12 37.209.194.12
nserver:      NS3.REGISTRY.IN 2001:dcd:3:0:0:0:0:12 37.209.196.12
nserver:      NS4.REGISTRY.IN 2001:dcd:4:0:0:0:0:12 37.209.198.12
nserver:      NS5.REGISTRY.IN 156.154.100.20 2001:502:2eda:0:0:0:0:20
nserver:      NS6.REGISTRY.IN 156.154.101.20 2001:502:ad09:0:0:0:0:20
ds-rdata:     54739 8 2 9f122cfd6604ae6deda0fe09f27be340a318f06afac11714a73409d43136472c

whois:        whois.registry.in

status:       ACTIVE
remarks:      Registration information: http://www.registry.in

created:      1989-05-08
changed:      2023-02-10
source:       IANA

# whois.registry.in

No Data Found
URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of WHOIS database: 2023-06-23T01:18:56Z <<<
```

## Question 21 - Analyzing the first extraction of information. What software runs the email server to which the stolen data is sent?

Look for the first STMP packet using `smtp` as the filter. Look at the response headers and you'll see the software version. 

## Question 22 - To which email account is the stolen information sent?

Look at the RCPT command and where it's going. That'll be your answer.

## Question 23 - What is the password used by the malware to send the email?

You'll have an SMTP packet that transmits the password in base64 encoding. Decode it and you'll have your answer.

## Question 24 - Which malware variant exfiltrated the data?

Well we don't have any obvious smoking guns for this, so let's look in the packets and see what we get. We have a large base64 blob being sent out. If you decode it, you'll get some output like this

```text
HawkEye Keylogger - <REDACTED>
Passwords Logs
roman.mcguire \ BEIJING-5CD1-PC

==================================================
URL               : https://login.aol.com/account/challenge/password
Web Browser       : Internet Explorer 7.0 - 9.0
User Name         : roman.mcguire914@aol.com
Password          : P@ssw0rd$
Password Strength : Very Strong
User Name Field   : 
Password Field    : 
Created Time      : 
Modified Time     : 
Filename          : 
==================================================

==================================================
URL               : https://www.bankofamerica.com/
Web Browser       : Chrome
User Name         : roman.mcguire
Password          : P@ssw0rd$
Password Strength : Very Strong
User Name Field   : onlineId1
Password Field    : passcode1
Created Time      : 4/10/2019 2:35:17 AM
Modified Time     : 
Filename          : C:\Users\roman.mcguire\AppData\Local\Google\Chrome\User Data\Default\Login Data
==================================================

==================================================
Name              : Roman McGuire
Application       : MS Outlook 2002/2003/2007/2010
Email             : roman.mcguire@pizzajukebox.com
Server            : pop.pizzajukebox.com
Server Port       : 995
Secured           : No
Type              : POP3
User              : roman.mcguire
Password          : P@ssw0rd$
Profile           : Outlook
Password Strength : Very Strong
SMTP Server       : smtp.pizzajukebox.com
SMTP Server Port  : 587
==================================================
```

I replaced the answer with REDACTED but there you go. 

## Question 25 - What are the bankofamerica access credentials? (username:password)

With the decoded blob, we can see the username and password for the bank of america account.

## Question 26 - Every how many minutes does the collected data get exfiltrated?

You can filter by the EHLO command to figure out the interval with the filter `smtp.req.command == "EHLO"` and then look at the timestamps.


# Conclusion

Well that was interesting. A great little walk through a PCAP. The questions in the beginning were pretty simple, but it was great how the exercise ramped up in difficulty as it progressed. I guess this was made by @malware_traffic so you know it's gotta be good. Hope CyberDefenders updates the questions that aged out, but honestly it's not that big of a deal. 

If you work through the later questions you'll find answers for one of them but the DNS resolution I guess will stay unanswerable for newcomers unless you have access to historical IP resolution data, which I've only seen available in paid services. Thanks for reading!