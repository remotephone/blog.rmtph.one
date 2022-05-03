---
layout: post
author: remotephone
title:  "SNS Costs Too Much"
date:   2022-05-02 22:04:00 -0600
categories: [lab, workflow, AWS]
largeimage: /images/avatar.jpg

---

# AWS SNS, PinPoint, Spending Less

Ever since AWS had to start charging for phone numbers to send SMS messages, my AWS bills have doubled. Three dollars in a month to six dollars isn't the end of the world, but it's more than I intended to pay. I dug through my bill to figure out what the costs were coming from and fixed it.

## AWS Bill Archaeology

When troubleshooting any cost issues in my accounts, I first head to the Cost Explorer in my root account. This lets me see all costs aggregated across all accounts. Here's what I saw after selecting the right time frame. 

![AWS Bill Over Six Months]({{site.url}}/images/sixmonthsbill.png){: .center-image }

So clearly at the end of March I did something that startd driving up my pinpoint costs. After filtering by account and seeing it was my primary organization account, I wanted to double check it was pinpoint and it's pretty clear. 

![Zooming in on the Bill]({{site.url}}/images/billbyservice.png){: .center-image }

Filtering by day, I saw something happened on March 29, but no memory of what it was. I checked Cloudtrail and there were no events for it and nothing in the S3 bucket I keep either. I knew I had some billing alarms set up, but no memory of configuring a pinpoint number for SNS. 

If you check pinpoint and don't have any campaigns set up, it looks like nothing should be running up your charges, which I think is confusing. If you go to AWS SNS, however, you can see Origination Numbers configured and that's where the charges are coming from. 

![AWS SNS Origination Numbers]({{site.url}}/images/snsorigination.png){: .center-image }

I deleted the number, which broke my billing alerts, for now. 

## Fixing the Billing Alerts

I did have a separate account in my organization where I had a number provisioned I had intended to pay for and was already using for some SNS notifications. The easiest and cheapest way to do this was, after decomissioning the other number, to allow my org to post to the SNS topic already connected to an Origination Number. 

The easiest way to do this was to modify the access policy on the SNS topic. I added a stanza like the following:

```json
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:us-east-1:123456789012:mytopic",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "<MYORGID>"
        }
      }
    }
```

I created a simple SNS user to test the publish function and it worked! After updating the Billing Alerts to point to the new topic Arn, things _should_ just work. Good work everybody. 


## All Done

A quick post, but something I wanted to write up because I can't be the only one who got tripped up by this. I know AWS had to make this change, but I've found it frustrating to work with and behaving in unexpected ways. I also kind of lament how it made a lot of hobby projects that just want to send a text even a little bit more complicated. Thanks for reading.