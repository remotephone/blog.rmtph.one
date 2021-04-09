---
layout: post
author: remotephone
title:  "What I Learned Running A Honeypot - 2 - Failing to Run it Locally"
date:   2020-09-04 00:20:00 -0600
categories: homelab workflow
---

## I'm not made of money

This honeypot had been running in AWS for a few weeks. On avereage, I was spending about $2.50 each day for the compute and disk resources. That adds up pretty quick. This is a view of my spend since the beginning of 2020. It's pretty easy to see I launched the honeypot late July and killed it mid-August.

![aws_bill_cost_2020]({{site.url}}/images/aws_bill_cost_2020.png){: .center-image }

Since I gathered a significant chunk of data, I want to make sure it's useable outside of AWS without spending more money. I invested in a pretty beefy home desktop and lab so I figured I could just run it at home. I ended up not getting it to work and figured I'd document what I did and tried. 

## Exporting the VM

The export is pretty straight forward and documented [here](https://docs.aws.amazon.com/vm-import/latest/userguide/vmexport.html). First I created an S3 bucket. You also need to attach a specific ACL to the bucket to allow AWS to write the export to it which [varies by region](https://docs.aws.amazon.com/vm-import/latest/userguide/vmexport.html#vmexport-prerequisites).

![export VM ACLs]({{site.url}}/images/export_acl.png){: .center-image }

Once the bucket is ready, you can do the actual export. This is the exact syntax I ran with redacted account specific information.

~~~
aws ec2 create-instance-export-task --description "debian honeypot" --instance-id i-xxxxxxxxxxxxxxxxx --target-environment vmware --export-to-s3-task DiskImageFormat=vmdk,ContainerFormat=ova,S3Bucket=secret-bucket
~~~

If your bucket is created successfully, you'll see a file show up called `vmimportexport_write_verification` created by the export process. Once I knew it was working, it took a while to run. You can use [describe-export-image-task](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-export-image-tasks.html) to check the status of the run. 


## Now the hard part

Once it's exported, you can download it. I created an OVA so it could be imported quickly and easily into Virtualbox. 

What a fool I was. 

The import works. I added enough CPU and memory to get it running and then I also unchecked "Import Hard Drives as VDI" and changed the Mac Address setting as shown. 

![vbox import settings]({{site.url}}/images/vbox_import_settings.png){: .center-image }

Regardless of the settinsg I tweaked, options I selected, or anything really, I could not get it to boot beyond this. 

![export boot progress]({{site.url}}/images/export_boot_progress.png){: .center-image }

## It works in VMWare Player though... 

In VMWare player, I could boot the system, the disk passes the check, and I could get to a terminal to log in. Once I logged in, however, networking is clearly broken. I was missing network interfaces, unable to bring them up, unable to get a network connection in any way I could figure out. 

I tried quite a few things, including exporting the [network manager](https://www.eightforums.com/threads/how-to-add-the-virtual-network-editor-to-vmware-player.5137/) and creating new interfaces, setting static IP ranges to match what was there, and all sorts of things. No luck whatsoever. 


## I don't know

I wish I had something better to end this with, but I just didn't figure out what to do here. I ended up just leaving the VM shut off in my account to turn back on when needed. That's life I guess. 

Thanks for reading, if you have suggestions, I'd like to hear them. 