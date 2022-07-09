---
layout: post
author: remotephone
title:  "AWS SNS - Spending All My Money"
date:   2021-10-24 22:49:00 -0600
categories: [lab, workflow, aws, sns]
largeimage: /images/avatar.jpg

---

# AWS SNS and their pricing changes

AWS recently underwent some changes to their SMS pricing and billing. They rolled out a new service, Amazon Pinpoint, and now they work hand in hand. I am kinda surprised about how little I've seen written about this since SMS is such a convenient way to notify yourself of changes in your accounts. I set it up and I'll write down what my thoughts.

## How I Used SNS for SMS Notifications

I plugged in quite a bit into SMS notifications. Billing notifications, various account automations, and other things would send me a covenient text message. This was also one of the easiest ways for me to go from AWS to the "real world", no portal I had to check for updates, no chat rooms or anything that might have their notifications silenced, just good ol fashioned text messages.

The typical flow was something like

```text
AWS Lambda -> AWS SNS -> Text Message
```

This was easy an convenient and basically free, I was limited to thousands of text messages per month before I passed the free tier and I never even got close.

## The Changes

It was too good to last. AWS published a [notification](https://aws.amazon.com/blogs/mobile/changes-coming-to-aws-amplifys-sms-based-authentication-workflows/) of their changes that I didn't appreciate at the time. It notified customers that in several months, they would no longer be able to send simple messages without taking some additional steps.

In short, [users had to rent a phone number](https://docs.aws.amazon.com/sns/latest/dg/channels-sms-originating-identities.html), whether short code, 10 digit number, or toll free number from Amazon to route your requests through. These numbers come at at an additional charge, outlined [here](https://aws.amazon.com/sns/sms-pricing/).

## Moving on

Working with this new requirement is pretty easy, unfortunately it will cost you. My amazon bill was typically $2-4 dollars a month and this will effectively double my AWS monthly expenses. Unfortunate, but manageable.

To make the change, go to the Amazon SNS console. If you go into AWS SNS > Topics > Subscriptions and you'll see a banner that you don't have a number registered.

![add number]({{site.url}}/images/aws_sns01.png){: .center-image }

Click `View Origination Number` and then `Provision Pinpoint Number`. Click `Provision Phone Number`, Choose your country (I'm doing US), and then select Toll Free. Uncheck Voice, because least privilege and all that, and the rest can remain default. You'll almost instantly be provisioned a number.

Go back to your SNS topic, send a test message, and you shoulkd receive it shortly.

![get_text]({{site.url}}/images/aws_sns02.png){: .center-image }

Then hang out and wait for your bill I guess.

## That's all

I'm disappointed this happened but it happened as a result of regulation and it's really targetting potential abuse far above the scale I'm operating at. There's other ways to connect things to each other that will take more work. I have built a discord bot and that's pretty easy, so I'll probably be sending messages to discord or slack if I get tired of paying the $2 a month.

Thanks for reading
