---
layout: post
author: remotephone
title:  "CyberDefenders.org - Malware Traffic Analysis Walktrhough"
date:   2021-04-08 00:40:00 -0600
categories: lab homelab malware traffic-analysis workflow
---

# CyberDefenders.org

[CyberDefenders](https://cyberdefenders.org/) is a free, community built site hosting security challenges. I'd heard about this site and it's interesting, so I thought I'd use a little bit of the pandemic to work through some of their exercises. I've also previously worked through exercises from and learned a lot from @malware_traffic (Brad Duncan of Palo Alto) so I thought his would be a good place to start. I'm starting on his first post [here](https://cyberdefenders.org/labs/17). I will be defanging the IOCs, but use caution.


## Setup

I am using a Windows computer. If you don't have one, grab one of the VMs [here](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/) or adjust the instructions for your OS. I've never used the tool, but Brad recommends using Brim, which brings together Suricata, Zeek, and Wireshark like functionality all in one too. I like all those tools, so let's try it out. You'll optionally want Wireshark installed.

1. Download the Brim [installer](https://www.brimsecurity.com/download/) and install it. The install screen is weird, just let it do its thing for a few minutes.  
2. Install [Suricata](https://suricata-ids.org/download/). You won't need to run it as an IDS, we'll import it into Brim though.  
3. Launch Brim, go to File > Settings and point the Suricata runner to your executable. The default path should be at `C:\Program Files\Suricata\suricata.exe`. Restart Brim  
4. Download an unzip the challenge materials.  
5. Import the file. 

Now you're ready to go. On successful import, you'll have something like this

![Brim ready to go]({{site.url}}/images/brim_mta1_1.png){: .center-image }


## Analysis

### Question 1: What is the IP address of the Windows VM that gets infected?

Brim seems to have a bunch of prepackaged queries that will probably help us investigate. If you look in the Queries section, you can see things like `HTTP Requests`, `File Activity`, etc etc. `Suricata alerts by Source and Destination` is going to be the first place to go. We see 2 alerts, and right away the internal host `172.16.165.165` stands out. Now, right click and `Pivot to logs` to get more detailed information. The query defaults to `event_type=alert src_ip=172.16.165.165 dest_ip=37.200.69.143`, but we can swap that to `event_type=alert dest_ip=172.16.165.165` so we can see what's headed to our host we're suspicious of. Going to the first alert by time, we see an alert for a possible GoonEK encrypted binary. 

We can look up the rule by googling `suricata rule 2018297` and seeing the source [here](https://github.com/OISF/suricata-update/blob/master/tests/emerging-current_events.rules#L2661). This rule looks at traffic for the hex content `89 b4 f4 6a 24 1f 46 14`. [Cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')&input=ODkgYjQgZjQgNmEgMjQgMWYgNDYgMTQ) doesn't decode that to anything obvious, so without going too much into it we'll take their word for it.  

![Suricata GoonEK Alert]({{site.url}}/images/brim_mta1_2.png){: .center-image }

Sorting the signatures with this host as the destination, we start to see a clearer picture and can probably answer question 1 with some condfidence. 

![Suricata GoonEK Alert List]({{site.url}}/images/brim_mta1_3.png){: .center-image }


### Question 2-3: What is the hostname/mac address of the Windows VM that gets infected?  

This is pretty straight forwad, we should expect to see this in DHCP traffic. The query `_path=dhcp` shows you the events and the host_name field comes up pretty easily. Same for the mac address.  

### Question 4-8: What is the IP/hostname of compromised website and exploit delivery? Redirect URL?  

So we probably have to work backwards in time a little more here. The alerts for EK delivery were from `37.200.69.143`, but that may not have been the original compromised site. We can use this query to search for all HTTP Traffic: `_path=http | cut ts, id.orig_h, id.resp_h, id.resp_p, method,host, uri | uniq -c | sort ts`. If we add `referrer` to the query, we can more easily see how this progressed. We can see the original host that began the malicious redirects is `www.ciniholland[.]nl` at `82.150.140[.]30`.  

We can also pull the IP from the EK delivery from the alerts earlier, and map that to the hostname `stand.trustandprobaterealty[.]com`.  Finally the redirect URL is the intermediary. Where did they go in between exploited site and delivery?  `hxxp://24corp-shop[.]com/` will do it.  


### Question 9: Other than CVE-2013-2551 IE exploit, what other exploit(s) sent by the EK? Format: comma-separated in alphabetical order	

For this, using the search `_path="files" | sort mime_type` was a quick way to find interesting events. The flash and java archives stick out and by right clicking the hash, you can very easily look up the files in virus-total and see the tags associated with the CVEs. I didn't really understand the format it wanted it. These are the links [here](https://www.virustotal.com/gui/file/e2e33b802a0d939d07bd8291f23484c2f68ccc33dc0655eb4493e5d3aebc0747/detection) and [here](https://www.virustotal.com/gui/file/178be0ed83a7a9020121dee1c305fd6ca3b74d15836835cfb1684da0b44190d3/detection).  

I used up the hints here because I couldn't figure out what format they wanted the answer in. The answer is `Flash,Java`.  

### Question 10: How many times was the payload delivered?	

To do this, let's go to alerts again. Sort with this query `event_type=alert alert.severity=1 alert.category="Exploit Kit Activity Detected" | sort alert.signature` and we see the GoonEK delivered 3 times.

### Question 11: What are the SIDs of the triggered Snort alerts in the Network Trojan category? Format: comma-separated in ascending order	

I didn't get this one. I used the files functionality in Brim to right click and search the file in VirusTotal, but nothing showed snort rule activity. I uploaded the pcap and was able to see quite a few rules fire off, but my final list did not match the answer to the test. The list of unique rules I ended up with 2014473, 2021430, 27816, 30934, and 30936, which I found by submitted the PCAP, doing a CTRL+F for `Network Trojan`, and viewing the rule IDs.  

The correct answer was `25562,27816,30934,30936`. This is the rule from the answer I did not have in my list, available [here](https://github.com/codecat007/snort-rules/blob/master/snortrules-snapshot-3000/rules/snort3-file-java.rules#L52)  

```
# alert tcp $EXTERNAL_NET $FILE_DATA_PORTS -> $HOME_NET any ( msg:"FILE-JAVA Oracle Java obfuscated jar file download attempt"; flow:established,to_client; flowbits:isset,file.jar; file_data; content:"Obfuscation by Allatori Obfuscator",fast_pattern,nocase; metadata:policy max-detect-ips drop; service:ftp-data, http, imap, pop3; reference:url,attack.mitre.org/techniques/T1027; reference:url,attack.mitre.org/techniques/T1140; classtype:trojan-activity; sid:25562; rev:9; )

```


### Question 12: The compromised website has a malicious script with a URL. What is this URL?	

This is easy, we know the URL we are going to end up at is `http://24corp-shop[.]com/` from earlier, and that's the answer. I went through wireshark manually reviewing each of the javascript files and didn't find this URL string in any of them, but I did learn we're dealing with a Plesk server and those have some bad luck.

### Question 13: Extract the exploit file(s). What is (are) the MD5 file hash(es)?	

This query `_path="files" 37.200.69.143 in tx_hosts | sort mime_type` will get you all files from where the exploit was delivered from. View the java-archive and x-shockwave-flash files, go to the bottom right, and you'll see the hashes. Submit those two as the answer. 


### Question 14: VirusTotal doesn't show how many times a specific rule was fired under the "Suricata alerts" section for the pcap analysis. Run the pcap file against your local Suricata (Emerging Threats Open ruleset) and provide the rule number that was fired the most.	

For this one, the query `event_type=alert | count() by alert.signature_id | sort count` will bubble up the answer. Submit the signature ID without commas that fired the most, `2018297`. 


## OK that's it.

Good stuff. I enjoyed this exercise and will probably work through more. I submitted feedback on the ones that didn't work as expected, so maybe I did something wrong or they can get updated. Either way, I really appreciate the work that went into this, this is a great learning resource. 

## Edit

I submitted that support ticket and got a response the same night that the content would get updated. Really impressed with their turnaround.