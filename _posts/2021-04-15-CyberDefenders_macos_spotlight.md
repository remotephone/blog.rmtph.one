---
layout: post
author: remotephone
title:  "CyberDefenders.org - macOS Spotlight - Forensics Walkthrough"
date:   2021-04-15 23:22:00 -0600
categories: lab homelab forensics macos workflow
largeimage: /images/avatar.jpg

---

# Another go at CyberDefenders.org 

Having another go at a [CyberDefenders](https://cyberdefenders.org/) challenge, this one about macOS image forensics. You can find a link to the challenge [here](https://cyberdefenders.org/labs/34).

## Environment

I did this largely on my Windows workstation. I used Autopsy to analyze the image. I used Windows Subsystem for Linux 2 to install mac_apt on. I cannot overstate how convenient it is to have very transparent access to Linux tools via my Windows workstation. I will boot into Linux every now and then if I need to, but I find I need to less and less as WSL gets better and better, especially once WSL2 came out. 

I set this up and installed stuff one night, worked on this over 2 nights and ran into some issues. I'll call out where it was night 2 and I had some time to think about the issues. Taking time off during work like this does me wonders because I can approach problems with a fresh mind later.

 The big difference on night 2 is that I installed apfs-fuse and used it to mount the drive and examine it manually. Somethings didn't work through Autopsy and I don't know why, but mounting the image and browsing let me access some files I couldn't access via Autopsy. Keep in mind this is not necessarily forensically sound unless you mount read only.


## Some notes

Autopsy failed to complete the download a couple of times, might have been my network having hiccups. Chrome's download resume feature came in handy, same for the forenics image.

I recommend you do this on an SSD. The image is 27gb, you'll need to extract it before you can work on it, and faster the disk the better. 500gb NVME drives are as cheap as 60 bucks now which is insane considering the speed an value, so if you have space for one, do it. 

I'm using a Ryzen 5 2600 and 16gb of ram, this took a long time. Even with a speedy drive, expanding the files took 10-15 minutes and once expanded, it took another 50 minutes to get the image loaded into Autopsy. Just be ready to keep busy as things get set up.  

This is my command line history for installing and configuring apfs-fuse, there were a few minor snags I ran into that got answered by browsing issues (see references, but this is what worked for me).

~~~bash
mkdir /tmp/mount
sudo apt install ewf-tools
ewfmount
ewfmount Desktop/c18-spotlight/Spotlight/FruitBook.E01 /tmp/mount
cd /tmp/mount
mkdir /tmp/mountpoint
sudo apt install sleuthkit
sudo apt install fuse libfuse3-dev bzip2 libbz2-dev cmake git libattr1-dev zlib1g-dev
sudo apt install clang
cd
git clone https://github.com/sgan81/apfs-fuse.git
cd apfs-fuse
git submodule init
git submodule update
mkdir build
cd build
ls
cmake ..
make
apfs-fuse
ls
./apfs-fuse /tmp/mount/ewf1 /tmp/mountpoint
sudo apt install fuse3 && sudo apt install libfuse3-dev
./apfs-fuse /tmp/mount/ewf1 /tmp/mountpoint
~~~

I watched Godzilla (2014) while doing this. This movie is excellent and I love it. 


## Question 1: What version of macOS is running on this image? 

I figured the BASICINFO plugin from mac_apt would give me this, but since I was installing it and loading the image into autopsy at the same time and autopsy finished slightly first, I was able to find the image info in Autopsy. Click into Results > Extracted Content > Operating System Information, Click SystemVersion.plist under the Listing tab on the right, and the at the bottom right, select the Text tab and you can read the plist file and pretty easily see we're running version 10.15.

## Question 2: What "copetitive advatge" did Hansel lie about in the file AnotherExample.jpg? (two words)	

I got zero points for this one and I don't know what's wrong. I used all the hints to get the answer

~~~bash
computer@computer: /mnt/c/Users/computer/Desktop/c18-spotlight/1/Export
$ strings AnotherExample.jpg                                                                 [22:50:08]
(mac_apt)  
computer@computer: /mnt/c/Users/computer/Desktop/c18-spotlight/1/Export
$ ls -aslh                                                                                                                                                                                                                    [22:50:11]
total 0
0 drwxrwxrwx 1 computer computer 4.0K Apr 14 22:44 .
0 drwxrwxrwx 1 computer computer 4.0K Apr 14 22:48 ..
0 -rwxrwxrwx 1 computer computer    0 Apr 14 22:44 AnotherExample.jpg
(mac_apt)  
computer@computer: /mnt/c/Users/computer/Desktop/c18-spotlight/1/Export
$                                                                             
~~~

I tried extracting the file. You can find the file in vol5 > APFS Pool > vol0 > Users > Shared > AnotherExample.jpg. It was easy enough to find but I tried extracting the file and got an empty file. I tried to view the file in the hex editor and nothing there either. The file has a file size listed but it just didn't work. Who knows what's up here.

#### Night 2

I ended up mounting the drive in WSL2 and was able to browse to the file, strings it, and get the answer very easily. This is my command history and the image was mounted to /tmp/mountpoint

~~~bash
cd /tmp/mountpoint/root
ls
cd Users/Shared/
ls
file AnotherExample.jpg
cp AnotherExample.jpg /mnt/c/Users/computer/Desktop
cat secret
strings AnotherExample.jpg
~~~

## Question 3: How many bookmarks are registered in safari?	

This file is the bookmarks file we're looking for- "/img_FruitBook.E01/vol_vol5/APFS Pool/vol_vol0/Users/hansel.apricot/Library/Safari/Bookmarks.plist" You can see in the bottom right table of autopsy theres 13 to look through. 

## Question 4: What's the content of the note titled "Passwords"?	

I tried doing this through Autopsy and while I could find the Notes sqlite db and extract it, I was having trouble going through it manually with sqlite3. 

I used the mac_apt plugin NOTES with this syntax. 

~~~bash
`python3 mac_apt.py E01 /mnt/c/Users/computer/Desktop/c18-spotlight/Spotlight/FruitBook.E01 -o /mnt/c/Users/computer/Desktop/c18-spotlight/mac_apt/ NOTES -x`
~~~

This gave me excel formatted output which was very easy to read. The flag is "Passwords".

## Question 5: Provide the MAC address of the ethernet adapter for this machine.

This syntax was the quickest way to grab it. Maybe it's in autopsy, but I didn't bother looking.

`python3 mac_apt.py E01 /mnt/c/Users/computer/Desktop/c18-spotlight/Spotlight/FruitBook.E01 -o /mnt/c/Users/computer/Desktop/c18-spotlight/mac_apt/ NETWORKING -x`

as mac_apt writes new files to your output folder, it numbers them intelligently by appending 01, 02 etc to each file.  The flag for this one was 00:0C:29:C4:65:77 which is easy to find on the Interfaces tab of the output. 

## Question 6: Name the data URL of the quarantined item.	

We can use the QUARANTINE plugin for this. The only file in the quarantine tab is `https://futureboy.us/stegano/encode.pl`

## Question 7: What app did the user "sneaky" try to install via a .dmg file? (one word)	

We can use Autopsy for this one. Find the user's folder. All macOS user folders are under /Users an Vol5 with the APFS pool has the appropriate mapping for us. Browsing through their files we find something interesting in the Trash folder. `/img_FruitBook.E01/vol_vol5/APFS Pool/vol_vol0/Users/sneaky/.Trash/silenteye-0.4.1b-snowleopard.dmg`

`silenteye` is your flag. Always check the Trash.

## Question 8: What was the file 'Examplesteg.jpg' renamed to?	

I feel like I brute forced this one too. I don't know if I loaded the image wrong or really have no clue what I'm doing, but I tried searching file change events and this file didn't seem to show up. There were several thousand find, but whacha gonna do. 

Ultimately, I found it by browsing `/img_FruitBook.E01/vol_vol5/APFS Pool/vol_vol0/Users/Shared/` since I knew there was action there. 

If I went to the timeline, reset all filters, and added only the filter of name `Examplesteg.jpg` I got zero results. I don't know what I'm missing here, but the fact that question gives you the first letter gives you what you need to find the answer. 


## Question 9: How much time was spent on mail.zoho.com on 4/20/2020?	
 
Autopsy was not very useful for this.

`$ python3 mac_apt.py E01 /mnt/c/Users/computer/Desktop/c18-spotlight/Spotlight/FruitBook.E01 -o /mnt/c/Users/computer/Desktop/c18-spotlight/mac_apt/ DOMAINS NETUSAGE SAFARI SCREENTIME -x`

I tried to add the times from the Safari tab but didn't get the write answer. I had to use all the hints here before I realized I had alsoc pulled the SCREENTIME function to get 20:58 for the screentime. That's the flag.

## Question 10: What is the name of the file that has a QuickLook bitmap data location of 166472?	

I recognized this file from the hints in the flag prompt easy, but the answer doesn't make sense with my output. I am not certain I have a valid extraction, so I would be interested to see what the output for the correct answer is here.

![macos file bitmap location]({{site.url}}/images/macos_forensics_01.png){: .center-image }

The flag here is `Get A NEW PHONE TODAY!.jpg`


## Question 11: What's hansel.apricot's password hint? (two words)	

`$ python3 mac_apt.py E01 /mnt/c/Users/computer/Desktop/c18-spotlight/Spotlight/FruitBook.E01 -o /mnt/c/Users/computer/Desktop/c18-spotlight/mac_apt/ USERS -x`

We can use this syntax to generate an excel sheet with user info, including the password hint. Our flag is `Family Opinion`


## Question 12: The main file that stores Hansel's iMessages had a few permissions changes. How many times did the permissions change?	


I actually get 5 changes here. I used the `FSEVENTS` module and exported to Excel. Filtered to the Filepath `Users/hansel.apricot/Library/Messages/chat.db` and got 5 `PermissionChange` events. I brute forced the answer and got 7 though. Not sure why. Even if we include `FolderCreated|PermissionChange` I get only 6 changes.


## Question 13: What's hansel.apricot's password hint? (two words)	

This is asking for the user rsponsible for connecting USB devices. Some simple standard google searches didn't get me anything I could use. I ended up using the hints and I got to the user usbmuxd described [here](https://github.com/libimobiledevice/usbmuxd). Your flag is `213`, but I feel like this is one of those things you just gotta know. 

I tried the IDEVICEINFO plugin for mac_apt and got nothign useful, so there may have been another way to this answer but I didn't find it. 

## -- Everything past here is night 2. --

## Question 14: Find the flag in the GoodExample.jpg image. It's hidden with better tools.	

So we have the file in our mounted drive, and it says they used better tools to hide the data in the image. Stegonography is a method of hiding data inside an image without affecting the image quality. You can use `info` to get information about what's embedded and hope you either have the password or its black.

~~~zsh
computer@computer: /mnt/c/Users/computer/Desktop/gits/mac_apt master!
$ steghide info /tmp/mountpoint/root/Users/Shared/GoodExample.jpg
"GoodExample.jpg":
  format: jpeg
  capacity: 7.6 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase:
  embedded file "steganopayload27635.txt":
    size: 106.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
computer@computer: /mnt/c/Users/computer/Desktop/gits/mac_apt master!
$ steghide extract -sf /tmp/mountpoint/root/Users/Shared/GoodExample.jpg --passphrase ""
wrote extracted data to "steganopayload27635.txt".
(mac_apt)
computer@computer: /mnt/c/Users/computer/Desktop/gits/mac_apt master!
$ cat steganopayload27635.txt
Our latest phone will have flag<helicopter> blades and 6 cameras on it. No
other phone has those features!%
(mac_apt)
~~~

And our flag is `helicopter`!

## Question 15: What was exactly typed in the Spotlight search bar on 4/20/2020 02:09:48	

We can parse the spotlight database with this syntax `$ python3 mac_apt.py E01 /mnt/c/Users/computer/Desktop/c18-spotlight/Spotlight/FruitBook.E01 -o /mnt/c/Users/computer/Desktop/c18-spotlight/mac_apt/ SPOTLIGHTSHORTCUTS -x`. The SPOTLIGHTSHORTCUTS plugin shows you what was typed into the searchbar. I used a sqlite db browser, it doesn't matter which you use, but the table with the data is SpotlightShortcuts.

## Question 16: What is hansel.apricot's Open Directory user UUID?	

FINALLY. Using the `USERS` plugin for mac_apt again, you can get quite a bit of information about all users on the system. In the Users table of the mac_apt output, you will see the UUID for hansel.apricot, `5BB00259-4F58-4FDE-BC67-C2659BA0A5A4` and that's our final flag. 



## All done

Excellent exercise again. There were a few things that I'm not sure if I did wrong or what, but I have some pretty clear takeaways about the tools. Autopsy is great, but I really don't think there's anything I couldn't have done with mac_apt. I am really impressed by the quality and comprehensiveness of that tool. It's well documented, full of useful features, and the output is incredibly easy to parse and analyze. 

Knowing what I know now, I would have skipped using Autopsy entirely and just gone for mac_apt and mounting the drive. Most of the disk forensics I've done is in linux and this feels very close to that. APFS is kinda wacky with where stuff gets laid out, but you can poke around and find your footing pretty easily if you're familiar with macOS. 

Finally, thanks again to [CyberDefenders.org](https://cyberdefenders.org) and [@champdfa](https://twitter.com/champdfa) for putting this together.


## References
[Macos Bookmarks Location](https://apple.stackexchange.com/questions/401556/where-are-the-safari-bookmarks-stored-on-a-computer)
[How to Mount an EWF image file](https://www.andreafortuna.org/2018/04/11/how-to-mount-an-ewf-image-file-e01-on-linux/)
[mac_apt](https://github.com/ydkhatri/mac_apt)
[apfs-fuse](https://github.com/sgan81/apfs-fuse)
[Missing Fuse3 Linux](https://github.com/sgan81/apfs-fuse/issues/87)