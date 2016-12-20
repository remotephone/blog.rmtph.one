---
layout:	post
title:  "Installing Jekyll for Security Professionals - or \"I don\'t know what I\'m doing\""
date:   2016-12-19 17:05:15 -0600
categories: installing jekyll 
---

## Why are we here anyway?

Jekyll lets you create static pages easily. I'm a security person, not a website admin and I really didn't want to spend the time learn while also generating content for the site. Maybe one day I'll move this to something more formal, but this really seems to do well enough for now. 

One advantage of this style is learning how to interact with github. Even simple pulls and commits are things I never really had to do, so its nice to least know what's happening when trying to speak to DevOps teams. 

The most obvious advantage is security. Lots of the WYSIWYG CMS's get interesting once you add plugins and that's when you add vulnerabilities. Static pages are just that, static pages. You can get "dynamic" enough with the features i Jekyll and that's good enough for me. Besides, how embarrassing would it be to have my security notes webpage spamming people because it was compromised?

## Why Jekyll?

I found Jekyll to be pretty well documented with plenty of templates and site examples to copy and choose from. [This](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/) site was the first guide I followed, but you take pretty much their entire site layout and then I found it difficult to add and change things without breaking the template \(this may be entirely my fault\). 

I'm pretty comfortable on the command line, so next I tired Github's documentation [here.](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/) That worked a lot better, but the layout was super basic. 

Ultimately I mashed a few different templates and guides together. This is risky if you're committing large changes together all at once, so I have been making changes slowly one at a time. This approach is much easier when you're just figuring this out and you're trying to learn github and jekyll and markdown at the same time. 

## Getting Started - locally and remotely

There are a few things you need to do. 

1. Install and prep your environment. 
	* I used ubuntu 16.04. You can use whatever you want, but you're on your own. 
	* Run the following:
	    sudo apt-get install ruby ruby-dev make gcc git
	* Create a new repository manually using the [github documentation](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)
	* I also turn on a color scheme in VIM \(shine until it fails me\) and enable spellchecking. Add this to your \~/.vimrc file
		* set spell spelllang=en_us
		* colorscheme shine
	* I keep a directory of repositories in my home directory to minimize clutter called gits and do all my work out of there
2. Create your repository through the web console and clone it locally or create it locally and push it out.
	* I tried doing it manually and this was just easier.
3. Make sure you SSH keys are set up to allow you to push commits quickly. I have a crazy password and its a pain to type it and my username all the time instead of just unlocking my ssh key.  
4. Find the sites you want to copy pages to template from, cd into the directory you'll use them and wget the raw github file instead of trying to copy and paste it into vim.
	* Be nice and credit them.
5. Build a single post and make sure you have the basics you want in there. This includes the header at the top that outlines the formatting. 
 
