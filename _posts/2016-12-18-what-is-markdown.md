---
layout: post
author: remotephone
title:  "What is Markdown anyway?"
date:   2016-12-18 21:14:55 -0600
description: Quick reference for markdown \(mainly for me\)
categories: [quick-reference, whatamidoing]
largeimage: /images/avatar.jpg

---

I don't know markdown syntax but it seems easy enough. This page is intended to be a quick reference for the 5-10 things I will no doubt do over and over and over.

[This page](https://kramdown.gettalong.org/quickref.html) seems pretty interesting, it has a lot of the syntax I'll be using.

This is the format for my post titles. Markup will usually be md for markdown, but maybe I'll get fancy one day.

    ``` YEAR-MONTH-DAY-title.md ```

To create a link, use ``` \[Text\]\(http\(s\)://site\.com\) ```

You can add a excerpt_separator: line to your _config.yml file to set a preview in your post. I haven't gotten it to work yet...

These are the original notes from the Jekyll page creation. Good stuff.

## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/remotephone/remotephone.github.io/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Kramdown Code Blocks in Lists

This was throwing me for a loop, but if you want code blocks in a list, you need to make the beginning of the code block match the indent of the list with spaces. See [here](https://planetjekyll.github.io/sandbox-syntax-highlighter/lists.html).

**Does not display cleanly when everything starts on the first space of each new line:**

* List

~~~bash
echo "hello"
~~~

**Nice and clean:**

* List

  ~~~bash
  echo "hello"
  ~~~
