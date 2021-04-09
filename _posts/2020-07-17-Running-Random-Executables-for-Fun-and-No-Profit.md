---
layout: post
author: remotephone
title:  "Running Random Executables for Fun and No Profit"
date:   2020-07-16 14:40:00 -0600
categories: homelab workflow
---

I saw this [tweet](https://twitter.com/ZoomerX/status/1283161781440057344) which I thought was hilarious and I wanted to see what the binary did. This post will walk you through my process of finding it and looking at it. 

![cve_tweet]({{site.url}}/images/cve_tweet.png){: .center-image }


## Finding it

The tweet itself didn't give any good info. I went to the accounts profile page and there is no link to a github account or anything. So i try some google dorking. The first search is `site:github.com filetype:exe CVE-2020-1350`. That didn't get me anything, so I removed the filetype search and simply searched `site:github.com CVE-2020-1350`. That led me to this [repo](https://github.com/ZephrFish/CVE-2020-1350) and I believe I found the [executable](https://github.com/ZephrFish/CVE-2020-1350/blob/master/CVE-2020-1350.exe) I wanted, or at least something interesting enough to look into. 


## What I'd Normally Do

I don't work with a ton of Windows machines at the day to day anymore, so analyzing unknown windows executables comes up every now and then instead of dozens of times a week. My lazy process is running it in a VM with sysmon, powershell script block logging, stringsing it (or [flossing](https://github.com/fireeye/flare-floss) it now more), and checking known OSINT resources for it. If I'm really stuck, I'll open procmon from SysInternals and see what I find, or run it in a throwaway system monitored by EDR and see what we get out of it. I'm on a linux laptop at home though, with none of my tooling or VMs set up, so I'm just gonna wing it. 


## Stabbing in the Dark

[Capa](https://github.com/fireeye/capa) was released today so I figured I'd go there first and see if it could do anything astounding.  I ran it through and was both disappointed and elated to get this output:

~~~
> ./capa-v1.0.0-linux CVE-2020-1350.exe    
1 functions [00:00, 99.15 functions/s]
WARNING:capa:--------------------------------------------------------------------------------
WARNING:capa: This sample appears to be a .NET module.
WARNING:capa: 
WARNING:capa: .NET is a cross-platform framework for running managed applications.
WARNING:capa: capa cannot handle non-native files. This means that the results may be misleading or incomplete.
WARNING:capa: You may have to analyze the file manually, using a tool like the .NET decompiler dnSpy.
WARNING:capa: 
WARNING:capa: Use -v or -vv if you really want to see the capabilities identified by capa.
WARNING:capa:--------------------------------------------------------------------------------
~~~

I was disappointed that all the work wasn't done for me and elated that I didn't know what I was doing and it pointed me to a good solution, dnSpy. It didn't even work where I tried it and I'm already blown away. I'm very excited to apply this tool in the day to day. I've used dnSpy a handful of times in the past, but since it was long ago and I didn't know what I was doing, it's time to relearn it. 

First though, some due dilligence. Get the filetype. 

~~~
> file CVE-2020-1350.exe
CVE-2020-1350.exe: PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows
~~~

Get the hash, look it up. 
~~~
> sha256sum CVE-2020-1350.exe
9e6da40db7c7f9d5ba679e7439f03ef6aacee9c34f9a3f686d02af34543f2e75  CVE-2020-1350.exe
~~~

This was submitted to [VirusTotal](https://www.virustotal.com/gui/file/9e6da40db7c7f9d5ba679e7439f03ef6aacee9c34f9a3f686d02af34543f2e75/detection) and [HybridAnalysis](https://www.hybrid-analysis.com/sample/41f5636546675a722c27b960c46fbf460c54af090ac9c5b468f6d4577f05f38f/5f0ee194625fc911731016e6) at the time of writing but I won't cheat and look. I promise. 

Strings it and you get stuff, but nothing super interesting. I will usually add a -n flag to strings like -n to find interesting strings longer than 8 bytes. String ehough files and you'll see that 4 bytes you end up with a lot of noise but at 8 or greater, if you're gonna get anything, it'll show up. Again, we have indicators its doing stuff, but nothing specific to go off of to use as IOCs or anything. Let's get weird. 

## Spy on it

[dnSpy](https://github.com/0xd4d/dnSpy) is a .Net disassembler. Take some .Net compiled code, show you its source. A google search for `dnSpy linux` has the first hit of this [issue](https://github.com/0xd4d/dnSpy/issues/1410) which tells me there's not a linux one shot approach to running this. The search `dnSpy linux alternative` gets me [here](https://www.saashub.com/dnspy-alternatives) and leads me to find [ILspy](https://github.com/icsharpcode/ILSpy). ILSpy has a linux version available [here](https://github.com/icsharpcode/AvaloniaILSpy). Download, extract, `chmod +x ILSpy; ./ILSpy` and we're off.

Not having run this and not being anything resembling an expert .Net programmer, let's look at the source and see what it does. Use File > Open to open the file. We'll go to the main function cause that is usuall where the action is. One of the parts we find is this:

~~~
		if (isValid.Equals(obj: true))
		{
			if (TextBox1.get_Text().Equals("127.0.0.1"))
			{
				Interaction.MsgBox((object)"Really?  You want to exploit yourself?  How dumb are you?", (MsgBoxStyle)36, (object)"Are you stupid?");
			}
			Process.Start("iexplore", "-k  https://www.youtube.com/embed/AyOqGRjVtls?autoplay=1&controls=0");
		}
~~~

Excellent, we're off to getting exactly [what we deserve](https://www.youtube.com/embed/AyOqGRjVtls). Once it validates the IP you insert, this code is interesting (defanged for your convenience):

~~~
	private void Main_Load(object sender, EventArgs e)
	{
		WebResponse response = WebRequest.Create("hxxp://canarytokens[.]com/articles/traffic/about/wz1fqpgpihmzsvjukc1tbweq0/post.jsp").GetResponse();
		new StreamReader(response.GetResponseStream()).ReadToEndAsync();
	}
~~~

A request is made to a canary tokens domain with a UID that is probably how this [data](https://twitter.com/ZephrFish/status/1283941996093157383) came to be. 


## A simple conclusion

So, never having executed this in a sandbox or VM or seeing it run, it looks like this creates a gui with some input and buttons, asks you who you want to exploit, ridicules you if you point it at localhost and then kermitrickrolls you, validates the IP anyway, and then calls out to a canary token URL to flag you as a baddy. 

I am thrilled that someone went through the trouble to do this. It's a kind of offensive defense that outs dumber baddies and gives really interesting intel about how quick people are to run random things off the internet they think might cause trouble. Big thanks to @ZoomerX on twitter and ZephrFish on github. This was very interesting to go through and a good example of why you don't run random things off the internet (including ILSpy...). Not everyones out to do harm to you, but its important to either know exactly what random code you're doing or at least get it from somewhere you trust before you run it. 

I'm off to reimage my machine. Thanks for reading. 