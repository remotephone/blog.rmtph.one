---
layout: post
author: remotephone
title:  "Notetaking and Copilot"
date:   2023-08-16 21:18:33 -0500
categories: [lab, workflow, copilot]
largeimage: /images/avatar.jpg
---

# Notetaking and Copilot

I take notes a lot, on all sorts of things and wanted to put some thoughts down on using Copilot or other AI assistants for it. I think it can be great at taking and making notes efficiently and wanted to share some of the ways I use it.

I'm taking this course on Yara rule writing from [Applied Network Defense](https://www.networkdefense.co/courses/yara/) so most of my examples will be around that, the course is excellent and while I have learned quite a bit working through the material they provide, you learn a lot more by going through their material and testing and iterating on their examples as they give them. I'll walk through how I do that.

A lot of these methods or ideas won't work for you because you don't take notes the way I do. Feel free to try some of them out and discard the ones that don't work.

## Filling in some blanks

When writing notes, there's lots of extrapolation you might want to do. I write everything in markdown because I expect to convert it or save it to GitHub where it renders nicely for me. So you can start with one or two bullet points like this:

![bullet points]({{ site.url }}/images/notes_copilot_01.png){: .center-image }

And then tab to complete the bullet pointed list, adding items as you go. Since it's so easy to add things you can throw them out and regenerate them quickly. You can quickly end up with something like this

![bullet points complete]({{ site.url }}/images/notes_copilot_02.png){: .center-image }

It's easy to get carried away with that though, so then you can take the bulleted lists into Copilot Chat and ask it to iterate on it. Make it shorter, make it even shorter, and then convert to sentences when you decide you'd rather have something more conversational.

![bullet points to text]({{ site.url }}/images/notes_copilot_03.png){: .center-image }

This is a quick way, especially when watching a video to add notes to a list and check as they're being populated.

This is no replacement for linting or proofreading, and familiarizing yourself with the hot keys or calling the command from the command pallette quickly will help a lot too, but especially if a video is going quickly, these shortcuts help keep up.

## Generating Examples

During the course, examples and techniques are given on how to write yara rules to look for strings, and several strings will be used in an example. You can use copilot to clean them up for use in a yara rule.

For example, if you have `String1`, `String2`, and `String3`, I find it tedious to write the quotes and format and name variables especially as the list gets longer, so I'll have copilot do that for me.

![strings to yara]({{ site.url }}/images/notes_copilot_04.png){: .center-image }

This is a quick way to get a list of strings into a yara rule. Then you're just responsible for writing the conditions and formatting. If you want to generate a template to fill in, simply ask chat for one and you'll get something you can modify and update easily.

![yara template]({{ site.url }}/images/notes_copilot_05.png){: .center-image }

You can then go back and edit the names of the variables to something more meaningful if you don't like what's generated, or you can use a better prompt.

You can also use comments in document to prompt the generation of the string like this

![yara template]({{ site.url }}/images/notes_copilot_07.png){: .center-image }

## Repetitive strings

As I've been writing this document, if you look at the source, you'll notice I name my images sequentially (notes_copilot_01.png, notes_copilot_02.png, etc). Once you have that done once or twice, copilot begins to get the hint and you can autocomplete insert things like url image links.

![image link]({{ site.url }}/images/notes_copilot_06.png){: .center-image }

## Changing formats

It's pretty straight forward to write some python to convert json to yaml or a list to yaml, but a lot of times, I will be working in a virtual environment that doesn't have it installed and I just need one set of data converted. In those situations, I will throw it in copilot and have it change it for me.

In this case, I just googled `nested json` and this [page from IBM](https://www.ibm.com/docs/no/db2/11.5?topic=documents-json-nested-objects) was the first hit. Throw that into copilot to convert it to yaml is quick, but you often need to surround the text in the triple ticks for code blocks.

![json to yaml]({{ site.url }}/images/notes_copilot_08.png){: .center-image }

It's also REALLY great for tables. Take [this example](https://yara.readthedocs.io/en/stable/writingrules.html#table-1) from the yara rules documentation. There is a table here that's really nice, and you could link to it, but if you want to keep the text in your notes themselves, it doesn't copy paste well.

![table to md table]({{ site.url }}/images/notes_copilot_09.png){: .center-image }

Instead, copy paste the text and ask copilot to convert to raw markdown text. Being specific helps here as sometimes it just renders the markdown for you and you can't copy paste that either.

Then you can copy the raw table and get a very neatly formatted markdown table like this. I manually edited the the header row to linkify it, but otherwise its how it came out of copilot.

| [YARA keywords](https://yara.readthedocs.io/en/stable/writingrules.html#table-1) |            |            |            |            |            |            |
|---------------|------------|------------|------------|------------|------------|------------|
| all           | and        | any        | ascii      | at         | base64     | base64wide |
| condition     | contains  | endswith  | entrypoint | false      | filesize  | for        |
| fullword      | global     | import     | icontains | iendswith | iequals   | in         |
| include       | int16      | int16be    | int32      | int32be    | int8       | int8be     |
| istartswith   | matches   | meta       | nocase     | none       | not        | of         |
| or            | private    | rule       | startswith | strings    | them       | true       |
| uint16        | uint16be  | uint32    | uint32be  | uint8      | uint8be   | wide       |
| xor           | defined    |            |            |            |            |            |

## Conclusion

So that was a quick and simple run down through how I use AI assistants like copilot to help me write notes. This is no replacement for actually learnign the content, its definitely no replacement for listening and taking your own notes, but when you know what you're going to write and its just a matter of getting the words out, prompts and autocompletes can help speed that up quite a bit.