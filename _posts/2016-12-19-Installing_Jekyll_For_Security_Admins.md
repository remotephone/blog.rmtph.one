---
layout: post
title:  "Installing Jekyll for Security Professionals - or \"I don\'t know what I\'m doing\""
date:   2016-12-19 17:05:15 -0600
categories: installing jekyll 
---

I found Jekyll to be pretty well documented, but some I'm somewhat impatient with documentation. [This](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/) site was the first guide I followed, but you take their layout and then I was unable to make themes happen cleanly. 

From what I can tell, the _layouts and other custom bits override any theme you want to apply. I didn't really figure it out, so I've wiped this repository a few times. 

I'm pretty comfortable on the command line, so next I tired Github's documentation [here.](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/) That worked a lot better, but the layout was super basic. 

Ultimately I mashed a few different tempaltes and guides together. This is risky if you're commiting large changes together one at a time and you're trying to learn github and jekyll and markdown at the same time. 

There are a few things you need to do. 

1. Install your environment. 
	* I used ubuntu 16.04. You can use whatever you want, but you're on your own. 
	* Run the following:
	    sudo apt-get install ruby ruby-dev make gcc git
	* Create a new repository manually using the github documentation 0 also [linked]https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/ above
