---
layout: post
author: remotephone
title:  "My AWS Organization - Part 3 - Billing Alerts"
date:   2020-2-23 00:40:00 -0600
categories: lab homelab AWS cloud workflow
---

# Part 3 - Billing Alerts - How You Know You're In Trouble

If you run an AWS Account for long enough, you will inevitably be shocked at how much you accidentally spent one month. For me, it was AWS Config rules under the [old billing model](https://aws.amazon.com/blogs/aws/new-updated-pay-per-use-pricing-model-for-aws-config-rules/). I turned on a bunch at once and didn't realize I was going to be paying $2 dollars per rule. I ended up spending 30 bucks that month, which isn't catastrophic, but was way more than I wanted to spend. As annoying as that was, it would have been much more annoying to find out whe I reviewed my bill. By that time, the monthly charges would have restarted and added insult to injury.  

To help you avoid or at least detect lessons like that, this post will walk you through setting billing alerts up and getting them sent to yourself. To follow along, you'll need an AWS Account, a phone number to receive messages at, and an IAM user with rights to Cloudwatch and SNS. I'll be using my 2FA protected full admin user in my primary or root organization account.


## The Importance of the Bill

The bill in AWS is your best source of truth for what's going on in your account. Some basic visibility into your bill with automated notice is a good way to know when you're spending money and a great way to know when someone has compromised your account and is spending money for you. 

When you receive your monthly bill, make it a habit of going through and understanding what's charging you money where. You'll find opportunities to save yourself money and know when something's amiss if you keep a better eye on things, starting with your bill. The bill, however, has a delayed effect. Unless you're logged into and watching Cost Explorer, you aren't likely getting anything like real-time feedback for how much you're spending. 

Setting up billing alarms gives you closer to real-time notice that you're spending more money than you want to spend.

## Making Yourself Available

AWS has a few different ways to let you know something's happening, but the easiest to start with is their [Simple Notification Service](https://aws.amazon.com/sns/). With 1 million free messages a month, unless you go absolutely wild, you're unlikely to have to really spend money on it. Go to the [console](https://us-east-1.console.aws.amazon.com/sns/v3/home) to start with the service. We're going to everything in US-East-1 to be consistent, but you can pick whatever region you want.

In SNS, messages are pushed to topics. Different subscriptions can be created per topic to fan messages out to multiple places. In this way, you can have a single source generate an alert and send that to your phone, email, and a webhook. First you want to to go the [Create Topic](https://console.aws.amazon.com/sns/v3/home?region=us-east-1#/create-topic) page from the SNS console.

I'm creating a topic called "Billing_Alerts_Demo" and leaving all options default, no encryption, nothing else. Give it a name and friendly name and then you will have a topic with no subscriptions. These are the bare minimum settings we need to get off the ground. If you're sending anything sensitive, you'll want to encrypt or worry more about the other features. 

![New Topic]({{site.url}}/images/new_topic.png){: .center-image }


Click Create Subscription. This is where you'll want your phone number handy. Set your phone number as the subscriber to the topic. You should receive a confirmation message you'll respond to if you've never used this number for SNS before. 

![See all the subscriptions I give!]({{site.url}}/images/sns_subscription.png){: .center-image }


Now that the topic is created and has a phone number subscribed to it, click Publish Message at the top right of the subscriptions screen. Fill out the information like so, and you're on your way. Notice you don't specify which subscription it's going to, all subscriptions under a Topic will receive the message.  

![Publish a test message]({{site.url}}/images/test_publish.png){: .center-image }


And if all went well, here you are!

![Great Success!]({{site.url}}/images/text.png){: .center-image }


## Making this Useful

Ok if you wanted to text yourself you could do it without an AWS account. Let's get this hooked up with [Cloudwatch](https://aws.amazon.com/cloudwatch/). Cloudwatch is AWS's metrics as a service offering. You can monitor slews of things here, CPU usage on an EC2 instance, data stored in an S3 bucket, arbitrary system logs, and billing data! Since AWS knows how much you're spending, you can set an alert to tell you when it thinks you're going to reach your limit. 

Go to the Cloudwatch console and find Billing on the left, click it. That'll get you to this page. 

![Default Console]({{site.url}}/images/billing0.png){: .center-image }


Click Create Alarm, Select Metric, and then you'll find yourself dumped here. Lots of options to get lost in, but we're here for something specific. 

![Billing Alert Settings]({{site.url}}/images/billing1.png){: .center-image }


Click Billing > Total Estimated Charge and then select USD and then Select Metric at the bottom. That'll get you here. I am setting the period to 6 hours, any shorter time period and it seems to act inconsistently. A six hour time period tends to generate alerts within about 15 minutes of billable actions being taken. I also left mostly defaults, but I am setting it to tell me anything projected cost is greater than $1.50. Since I'm over that this month, it should trigger something pretty quick.

There's an interesting option here to use [anomaly detection](https://aws.amazon.com/blogs/aws/new-amazon-cloudwatch-anomaly-detection/). If your bill is going to be X dollars greater than usual, an alert will be triggered. AWS uses machine learning, AI, and probably the blockchain to figure out how much a metric usually is and notify you when the expected values changes too much.

![Metric Settings]({{site.url}}/images/billing_metric.png){: .center-image }


So now we are going to send this to our SNS topic. Why it says my SMS subscription is an email address is beyond me, but it will work anyway. You can verify all is set correctly in the SNS console page if you want. 

Note that you could also trigger Autoscaling or EC2 actions here. This could, for example, be used to scale up or down instances when resource utilization meets certain conditions. That's beyond the scope of what we're doing here, but you can read about it [here](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-instance-monitoring.html).

![Configure some actions]({{site.url}}/images/configure_actions.png){: .center-image }


Name it, preview and create it, and then you'll land back in your cloudwatch console where you can see your alarms. You'll notice I created one for $1.50, $3, $5, and $10. If one goes off, that's good to know. If two go off at once, something suspicious is up and I should check my account ASAP. 

On the night of the great Config charges, I set the config rules up and hung out for a while until 3 text messages showed up all at once. It was a good way to learn a lesson that could have been much more expensive.

![My Alarms]({{site.url}}/images/billing_alarms.png){: .center-image }


When enough data is populated, you should get notified. I played with time periods for these alerts to see what the shortest one I could get is. Cloudwatch does not like 1 minute or less alarms for billing, but also doesn't seem to give 1 minute time periods enough data to alert. At a 6 hour time period, I was able to trigger an alert in about 12 minutes. 

![Oh no we're broke]({{site.url}}/images/billing_text.png){: .center-image }


I went ahead and did this a few times. I get notified at 3, 5, and 10 dollars because I manage to stay within the free tier and if I were to ever see more than one alarm at once, that would be quite the time to panic. Find a billing amount your comfortable with, set it, and hope they don't go off. 


## That's all for today

Ok so that's about enough. If you followed along, you should have billing alerts for amounts you want to know about, get them delivered directly to you so you can respond quickly. The bill is where good management and security start, so learn to pay attention to it early and save yourself a lot of trouble.

This post was pretty picture heavy, but the next one I'll do will be taking this very simple deployment and turning it into something you can deploy with CloudFormation and/or Terraform, depending on how much I get done. Thanks for reading!

** Special thanks to Ravi for pointing out some formatting and other kind of tweaks I could make to this. I appreciate the feedback!