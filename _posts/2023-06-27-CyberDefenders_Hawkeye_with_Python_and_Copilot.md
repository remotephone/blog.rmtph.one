---
layout: post
author: remotephone
title:  "CyberDefenders - Hawkeye - Now with Python and Copilot"
date:   2023-06-27 19:45:33 -0500
categories: [lab, workflow, networking, dfir, pcap]
largeimage: /images/avatar.jpg

---

# Hawkeye - Better Living Through Automation

The first run through on Hawkeye was pretty straight forward. I've spent a lot of time over the past few years trying to automate work that I can, so let's use that here. I approach automation from the perspective that I don't have to fully automate something completely to reap huge dividends on a process. I'll lean heavily on Copilot and accept most of what it spits out for me since this isn't anything I need to deliver anywhere, this is just a toy script to run on exercises.

## The Goal

I want to be able to run a script that will parse a pcap file and output a json file with the following information

- timestamp
- source ip
- destination ip
- source port
- destination port
- source mac
- destination mac
- protocol
- payload
- geoip information
- mac vendor information
- interesting content

And I wanted this to be a bit more general purpose than just for this exercise, so I avoided automating answers to every question.

## Setup

To follow along, you'll need VS Code Insiders, GitHub Copilot, and GitHub Copilot Chat installed. You'll also need Python, I'm using version 3.11. I use pipenv for virtual environments, but you can use whatever you like.

## The Script

You can see the full content of the script here - <https://gist.github.com/remotephone/f6fcc5e5f2c11eb702df2db7f7d728d4>

I'll walk through the script and explain what I did and why.

### Imports

There's apparently a lot of ways to do parse packets in python. You can use scapy, dpkt, or pyshark. I've used dpkt in this post, but tried a few functions with scapy before ultimately going back to dpkt. The cool, thing about Copilot is it makes it very easy to experiment, there's a very low cost to getting things wrong since you can quickly regenerate code and try again.

Some of the libraries that I used the most were

- `dpkt` - parse and process packets. The heavy lifter
- `manuf` - get mac address vendor information
- `geoip` - get geoip information using the free maxmind DB

I chose `dpkt` based on some google searches, I needed something that would parse packets, well featured, and had quite a bit of documentation and examples behind it. `manuf` I found through trial and error, but it appeared to be the quickest way to get vendor information. `geoip` was a known slam dunk, if you've done any geolocation of IP addresses, you've probably used maxmind's free DB.

## Getting Started

I started building out the code inside of the function `open_pcap_and_read_it` first. To begin working with a pca, I knew I had to be able to read the file into memory and do some basic operations on it. I started with the following code.

```python
packets = []
with open(pcap_path, "rb") as pcap_file:
    # read pcap file
    pcap = dpkt.pcap.Reader(pcap_file)
    # for each packet in the pcap file
    for ts, buf in pcap:
        packets.append(buf)
print(len(packets))
```

Since I had already worked through the exercise manually, I had a good point of reference to check for errors in my work. I knew that the pcap file had 4003 packets in it, so I could check my work by printing out the length of the packets array. I also knew that I needed to parse the packets, so I started looking for ways to do that.

First, I thought about what I wanted out of each packet. I crafted a dictionary that looked like this, with empty values

```python
packet = {
    "timestamp": ""
    "source_ip": ""
    "destination_ip": ""
    "source_port": ""
    "destination_port": ""
    "source_mac": ""
    "destination_mac": ""
    "protocol": ""
    "payload": ""
    "geoip": ""
}
```

I knew that I could get the timestamp from the pcap file, so I added that to the dictionary. I also knew that I could get the source and destination IP addresses, source and destination ports, and the protocol. I added those to the dictionary as well. I also knew that I could get the payload, but I wasn't sure how to get the source and destination MAC addresses or the geoip information. After some massaging of the code and prompting copilot with things like "How do i get the source IP from a packet", I ended up with the following code.

```python
packets = []
# open pcap file
with open(pcap_path, "rb") as pcap_file:
    # read pcap file
    pcap = dpkt.pcap.Reader(pcap_file)
    # for each packet in the pcap file
    for ts, buf in pcap:
        eth = dpkt.ethernet.Ethernet(buf)
        ip = eth.data
        protocol = ip.data
        if isinstance(protocol, dpkt.tcp.TCP) or isinstance(protocol, dpkt.udp.UDP):
            packet = {
                "timestamp": datetime.datetime.utcfromtimestamp(ts).timestamp(),
                "source_ip": socket.inet_ntop(socket.AF_INET, ip.src),
                "destination_ip": socket.inet_ntop(socket.AF_INET, ip.dst),
                "source_mac": ":".join("%02x" % b for b in eth.src),
                "destination_mac": ":".join("%02x" % b for b in eth.dst),
                "protocol": buf[23],
                # ensure payload is a human readable string if possible
                "payload": protocol.data.decode(errors="ignore"),
            }

with open("results.json", "w") as f:
    json.dump(pcap_contents, f, indent=4)

```

I also knew I wanted to get the results into a file, so I used the json library to dump it to a file in the same folder. I quickly ran into some errors trying to serialize some of the data it generated. Typically, I might go through value by value to figure out how to convert them to a format they can be dumped to json, but instead, I prompted copilot to give me a function that'll handle it for me. I accepted this function without modification.

```py
def convert_to_serializable(obj):
    if isinstance(obj, bytes):
        try:
            # Attempt to decode the byte string
            decoded_string = obj.decode("utf-8")
        except UnicodeDecodeError as e:
            # Handle the error by ignoring or replacing the invalid characters
            decoded_string = obj.decode("utf-8", errors="ignore")
            # Or
            decoded_string = obj.decode("utf-8", errors="replace")
        return decoded_string
    elif isinstance(obj, datetime.datetime):
        return obj.timestamp()
    else:
        raise TypeError(f"Object of type {type(obj)} is not JSON serializable")
```

From here on out, we're adding feature by feature. I went through the answers in questions and focused on the ones I could answer generically. For example, `Question 20 - What is the domain’s creation date to which the information is exfiltrated?` is one I could query whois information for and get data for every domain, but I didn't want to do that for every domain, so I skipped it. I also skipped `Question 18 - What is the public IP of the victim’s computer?` because it felt like that was going to be a very pcap specific question and looking for all answers to the public IP finder domain would not be usable everywhere.

Some things I did decide to do for all pcaps it processes were:

- Get the count of packets
- Get the duration
- Get the DNS servers used
- Get some high level statistics about the communicators
- Pull HTTP headers
- Get DNS queries ran

## Where Copilot Shined

Copilot was really helpful in a few places. Specifically, it can often "read" documentation and generate useful functions and parsers for me. For example, to get the mac addresses, I began popualting the keys above it. So while it's pretty straight forward to use `datetime.datetime.utcfromtimestamp(ts).timestamp()` for the timestamps, since we know we're looping through the packets, when I got to the next value to fill, it prompted pretty quickly with a snippet to get the mac address, `":".join("%02x" % b for b in eth.src)`.

If you look in the dpkt docs, specifically the examples [here](https://dpkt.readthedocs.io/en/latest/_modules/examples/print_icmp.html#mac_addr), you can see an example where the library does something similar to get the mac address. Copilot was able to read that and generate a snippet for me.

Copilot was also good at reading errors and fixing them. Especially with functions it just created. As I would add code to the script, I would rerun it and make sure it worked as expected. When I got the following error, it pretty quickly identified the issue and fixed it for me.

![copilot error fixing]({{site.url}}/images/01_pcap_copilot.png){: .center-image }

## Where it didn't

I did get a few hallucinations, made up functions that didn't exist. I also had a few times where code that was generated needed to be changed and updated to work in my script. Specifically, I took a funciton copilot generated to parse HTTP trafficand asked it to generate something similar for SMTP, but the example it returned had different names assigned to some imported functions, accessed them at different levels, and generally would not work out of the box.

![copilot generating similar functions]({{site.url}}/images/02_pcap_copilot.png){: .center-image }

Even then, it saved an enormous amount of timing fixing replicating and fixing small errors in the code.

Another thing it struggled with generating something that could put multiple packets back together as part of a TCP Stream to extract a file. I tried a bunch of different prompts for this and tried a lot of different minor tweaks in what it generated, but since that functionality works so well in something like Wireshark, I would likely just move to a different tool if I needed something to get done quickly.

## So How Did it do?

With the script written, we can see what we can solve easily in Hawkeye and try it on another exercise. Here's the truncated output from the script, excluding the raw packets except for a single one.

```json
{
    "total_packets": 4003,
    "size_of_pcap": 2454198,
    "file_name": "pcap_files/stealer.pcap",
    "pcap_first_packet_time": 1554946627.12973,
    "pcap_last_packet_time": 1554950448.690963,
    "pcap_duration": "1:03:41.561233",
    "most_active_ethernet_source": "a4:1f:72:c2:09:6a",
    "most_active_ethernet_destination": "ff:ff:ff:ff:ff:ff",
    "total_unique_source_ips": 6,
    "total_unique_destination_ips": 11,
    "total_unique_local_source_ips": 2,
    "total_unique_local_destination_ips": 5,
    "total_unique_global_source_ips": 4,
    "total_unique_global_destination_ips": 6,
    "dns_queries": {
        "proforma-invoices.com": [
            1554946673.791017
        ],
        "Beijing-5cd1-PC.pizzajukebox.com": [
            1554949291.9465,
            1554949291.949589,
            1554949486.601691,
            1554949487.615667,
            1554949487.621062,
            1554950387.780967,
            1554950387.784043
        ],
        "_ldap._tcp.Default-First-Site-Name._sites.PizzaJukebox-DC.pizzajukebox.com": [
            1554946653.377476
        ],
        "bot.whatismyipaddress.com": [
            1554946695.672284,
            1554947300.054661,
            1554947904.365588,
            1554948510.14105,
            1554949114.24755,
            1554949718.412799,
            1554950322.557987
        ],
        "isatap.localdomain": [
            1554949291.512456,
            1554949292.209193,
            1554949294.177486,
            1554949486.872143,
            1554949490.053157
        ],
        "dns.msftncsi.com": [
            1554946653.911651,
            1554949294.452779,
            1554949489.914889,
            1554949489.914889,
            1554949489.914889,
            1554949490.92056,
            1554949491.937583,
            1554949493.961702
        ],
        "update.googleapis.com": [
            1554947278.641867
        ],
        "_ldap._tcp.PizzaJukebox-DC.pizzajukebox.com": [
            1554946653.378245
        ],
        "PizzaJukebox-DC.pizzajukebox.com": [
            1554949294.270745,
            1554949490.054858,
            1554950448.46233
        ],
        "macwinlogistics.in": [
            1554946695.832004,
            1554947300.203215,
            1554947904.523825,
            1554948510.285774,
            1554949114.412311,
            1554949718.568485,
            1554950322.706958
        ]
    },
    "dns_servers_queried": [
        "10.4.10.4"
    ],
    "combined_smtp_message": "EHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 20:38:08 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox\nEHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 20:48:18 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox\nEHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 20:58:22 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox\nEHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 21:08:27 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox\nEHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 21:18:30 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox\nEHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 21:28:33 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox\nEHLO Beijing-5cd1-PC\nAUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==\nSales@23\n0\u0002\u000b\nRCPT TO:<sales.del@macwinlogistics.in>\nDATA\nMIME-Version: 1.0From: sales.del@macwinlogistics.inTo: sales.del@macwinlogistics.inDate: 10 Apr 2019 21:38:36 +0000Subject: =?utf-8?B?SGF3a0V5ZSBLZXlsb2dnZXIgLSBSZWJvcm4gdjkgLSBQYXNzd29yZHMgTG9ncyAtIHJvbWFuLm1jZ3VpcmUgXCBCRUlKSU5HLTVDRDEtUEMgLSAxNzMuNjYuMTQ2LjExMg==?=Content-Type: text/plain; charset=utf-8Content-Transfer-Encoding: base64\nHawkEye Keylogger - Reborn v9\r\nPasswords Logs\r\nroman.m\nry Strong\n smtp.pizzajukebox",
    "packets": [
        {
            "timestamp": 1554946674.727276,
            "source_ip": "10.4.10.132",
            "destination_ip": "217.182.138.150",
            "source_mac": "00:08:02:1c:47:ae",
            "destination_mac": "20:e5:2a:b6:93:f1",
            "protocol": 6,
            "payload": "GET /proforma/tkraw_Protected99.exe HTTP/1.1\r\nAccept: */*\r\nAccept-Encoding: gzip, deflate\r\nUser-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/7.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)\r\nHost: proforma-invoices.com\r\nConnection: Keep-Alive\r\n\r\n",
            "source_port": 49204,
            "destination_port": 80,
            "http_headers": {
                "Accept": "*/*",
                "Accept-Encoding": "gzip, deflate",
                "User-Agent": "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/7.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
                "Host": "proforma-invoices.com",
                "Connection": "Keep-Alive"
            },
            "source_geoip": {},
            "destination_geoip": {
                "continent": {
                    "code": "EU",
                    "geoname_id": 6255148,
                    "names": {
                        "de": "Europa",
                        "en": "Europe",
                        "es": "Europa",
                        "fr": "Europe",
                        "ja": "\u30e8\u30fc\u30ed\u30c3\u30d1",
                        "pt-BR": "Europa",
                        "ru": "\u0415\u0432\u0440\u043e\u043f\u0430",
                        "zh-CN": "\u6b27\u6d32"
                    }
                },
                "country": {
                    "geoname_id": 2635167,
                    "iso_code": "GB",
                    "names": {
                        "de": "Vereinigtes K\u00f6nigreich",
                        "en": "United Kingdom",
                        "es": "Reino Unido",
                        "fr": "Royaume-Uni",
                        "ja": "\u30a4\u30ae\u30ea\u30b9",
                        "pt-BR": "Reino Unido",
                        "ru": "\u0412\u0435\u043b\u0438\u043a\u043e\u0431\u0440\u0438\u0442\u0430\u043d\u0438\u044f",
                        "zh-CN": "\u82f1\u56fd"
                    }
                },
                "location": {
                    "latitude": 51.5,
                    "longitude": -0.13,
                    "time_zone": "Europe/London"
                },
                "registered_country": {
                    "geoname_id": 2635167,
                    "iso_code": "GB",
                    "names": {
                        "de": "Vereinigtes K\u00f6nigreich",
                        "en": "United Kingdom",
                        "es": "Reino Unido",
                        "fr": "Royaume-Uni",
                        "ja": "\u30a4\u30ae\u30ea\u30b9",
                        "pt-BR": "Reino Unido",
                        "ru": "\u0412\u0435\u043b\u0438\u043a\u043e\u0431\u0440\u0438\u0442\u0430\u043d\u0438\u044f",
                        "zh-CN": "\u82f1\u56fd"
                    }
                }
            },
            "source_mac_vendor": "HewlettP",
            "destination_mac_vendor": "Netgear"
        },
            <snip...>
    ]
}
```

## Conclusion

That's quite a few questions answered without breaking a sweat! We have answers to the packet counts, mac vendor questions, dns query questions, HTTP headers are easy to read, geoip information is included inline, we know who the chattiest talkers are, and a few others.

The real conclusion to draw here is there's a good case to be made that I didn't learn as much as I could have. Spending significantly longer than I did in the documentation to understand how the libraries work and what I was parsing would be a good and useful learning exercise. I'm not sure I would have learned anything new about the data, but I would have learned more about the tools I was using.

If you're going to use tools like Copilot to help you generate usable code, I would encourage you to read the explanations it generates along with the code and make sure you understand them. If this was code I intended to put into production at a job, I would be doing quite a bit more testing, validation, and error handling. I would also be doing more to make sure the code was performant. I'm not sure how well this would scale to a large data set, but I suspect it would do as well as it's going to do until I needed to look for something more purpose built. In a production environment, I would probably be looking at leveraging something like Zeek to process this data, since it's purpose built and extremely effective.

The next step is to run this against an exercise we haven't done before and see if it performs as well. I'll do that in another post. Thanks for reading!
