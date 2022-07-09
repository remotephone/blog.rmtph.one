---
layout: post
author: remotephone
title:  "My AWS Organization - Part 1"
date:   2020-1-20 23:28:00 -0600
categories: [lab, homelab, AWS, cloud, workflow]
largeimage: /images/avatar.jpg

---



# AWS As Your Home Away From Home Lab

I don't need to tell you the clouds big business and you should know how to work in it. It makes sense to have a place to experiment and test things in, so hopefully, you'll read these posts and be well on your way to having an Organization you can build stuff and play around in. This will be the first of a few that should walk you through the choices I made in setting up my organization. I spend about 2-3 dollars a month, run a few lambdas and billing alerts that do fun things for me, and wrote this in hopes  you'll learn something from reading it.

## How I Learned This Stuff

I had a job at one time that was going head first into the cloud and figuring it out as they went. The security team was catching up to the speed things were moving out there and they asked for someone to take an interest and lead the way. I was that person and got thrown head first into learning cloud stuff. I took advantage of being in over my head at work and studying at home to catch up.

I used some training material from [acloud.guru](https://acloud.guru/) to learn fundamentals while jumped into work projects. The [AWS documentation](https://docs.aws.amazon.com/) is indispensible of course, but hard to dive right into. You'll find yourself in there a lot to answer specific questions, but to get pointed in a solid direction to start learning, you'll want to spend more time in repos like the [AWS Samples](https://github.com/aws-samples) repo. There's a ton of projects you can follow along with to understand how things are built well.

I did a few certs throug A Cloud Guru and one thing they emphasize well is reading all the [white papers](https://aws.amazon.com/whitepapers). Reading the [Well Architected Framework](https://d1.awsstatic.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf?did=wp_card&trk=wp_card) paper is important, it gives you a perspective on the rest of things you'll learn and a good understqanding on how this stuff all fits in together. There's a bunch of specific ones you'll find interesting too, depending on your focus. The [Incident Response Guide](https://d1.awsstatic.com/whitepapers/aws_security_incident_response.pdf?did=wp_card&trk=wp_card) is excellent for incident responders and you can see how it fits into their more general guidance about architecting things well to begin with.

## Target Audience

Folks hoping to follow along with this post should have some comfort with AWS services, including some basic experience with some of the more common ones like Identity and Access Management (IAM), Elastic Compute Cloud (EC2), Simple Storage Service (S3), Simple Notification Services (SNS), and Cloudwatch. If you know what they are but still want to brush up, here are some great resources (some a little dated, but with solid core ideas):

- IAM - Some overlap, but both great
  - [AWS re:Invent 2018: Become an IAM Policy Master in 60 Minutes or Less (SEC316-R1)](youtube.com/watch?v=YQsK4MtsELU&list=PLF4BbYDJBqjqyqGGLyoUze0MWw34ta4c4&index=25)
  - [AWS re:Invent 2017: IAM Policy Ninja (SID314)](https://www.youtube.com/watch?v=aISWoPf_XNE&t=3s)
- S3
  - [AWS re:Invent 2016: Deep Dive on Amazon S3 (STG303)](https://www.youtube.com/watch?v=bMhWWkhydFQ)
- Organizations
  - [AWS re:Inforce 2019: Managing Multi-Account AWS Environments Using AWS Organizations (FND314)](https://www.youtube.com/watch?v=fxo67UeeN1A)
- SNS
  - [Quick Tips to use Amazon Simple Notification Service (SNS)](https://www.whizlabs.com/blog/aws-sns/)

## Some Perspective

Doing labs is great, chasing certs is great, and working at home is great, but there's nothing to compare to the scale companies work at. You won't get important experience and troubleshoot surprisingly common "edge cases" unless you throw yourself into these projects at work or as part of a service that serves a significant number of people and see how common these "edge cases" actually are. The basic features of the cloud are amazing, but they really start to make sense when you see them interact to serve requests from a large number of people.

None of this should discourage you from building a lab and running services of your own, but you should understand that it makes a big difference to do this stuff at scale. If you get the basics out of the way in your lab, you'll be ready when the weird stuff hits you in production environments.

## Organizations Overview

An AWS Organization is the highest level of the AWS resource hierarchy. An organization is a collection of accounts managed via a single, master account. This master account can manage permissions in member accounts (via Service Control Policies), allow access to sub accounts in certain configurations, and centralizes billing for all member accounts. A company will use an organization to centralize control of all sub accounts, maintain a centralized inventory, and (attempt to) control costs.

We're going to build one. It's going to look like this:

![Organization Map]({{site.url}}/images/orgmap.png){: .center-image }

The Primary Account manages all the other accounts, the Active accounts have minimal Service Control Policies (SCP) set to restrict activity in the account, the Restricted accounts are test groups for SCPs (maybe restricting to a single region, service, or condition), and finally Inactive accounts are accounts I created but don't need so they're locked down entirely by SCPs.

You'll need to sign up for an AWS account with an email address (eventually you'll need a few). This should not be the same email and AWS account you use to order underwear and  light bulbs. I recommend you set up a separate gmail or whatever account. The email address should tell you something about the purpose - blogpostdemo-master@gmail.com or whatever. Configure it to use hardware 2fa and a complex password. The hardware 2FA is going to come in handy because we're going to use it over and over. If you were a fancy organization with complex security needs, you'd manage this differently - we are not and you're welcome to go beyond the scope of these posts to set this up differently.

## Your Primary Account

Create your [AWS account.](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) You'll have to add a payment method because at some point we will be spending money. Every account you create is under the [AWS Free Tier](https://aws.amazon.com/free/) and you should take advantage of this. For the first year, take advantage of the services you can use and know your limits. For example, while you can always run 1 million lambda function executions (within limits) per month even after the first year, you only get to run a free t2/3.micro ec2 instance for a year.  

This account will be the master account of your organization. Here are the guidelines I follow for mine and recommend you follow for yours:

- Minimum resources run in this account except for logging and billing
- Billing Alerts configured in this account
- Root account protected by hardware 2FA, rarely if ever used
- Single Admin IAM (Identity and Access Management) User protected with phone app 2FA
- IAM User protected with 2FA IAM policy (more later)

Other than that, I tend to keep services out of this account. Everything that runs here expands the attack surface. If you lose your root account, you lose your organization, so its absolutely critical you keep this secure. After using it for long enough, I feel like AWS is pretty secure, but the more holes you punch in it, the more likely something is to go wrong.

## A Note on 2FA and Your Options

There's plenty of opinions on this and SIM swapping is a real threat for some carriers. Generally, most people are fine with SMS 2FA and any 2FA is better than none at all. I'm not going to pretend to resovle this argument or be an authority on it, but I can tell you what I do.

I use hardware 2FA for my sensitive accounts because I'm cheap. I think its worth the ~20 dollars to buy a hardware key and protect my accounts because its so much cheaper than screwing up and getting an account compromised. The [Feitian keys](https://www.amazon.com/Feitian-ePass-NFC-FIDO-Security/dp/B01M1R5LRD) are nice, the [Yubikeys](https://www.amazon.com/Yubico-YubiKey-USB-Authentication-Security/dp/B07HBD71HL) are nicer but offer features I've been able to live with out.

When I have the chance to add multiple 2FA methods (which I love) I disable SMS, enable hardware tokens and a phone or phone app login. I keep the hardware token safe at home as a backup and keep my phone with me. I've had my phone fail me and its a pain to recover a bunch of 2FA protected accounts.

AWS Accounts now support Hardware 2FA tokens for console logins, but not for CLI access. My strategy has been 2 protect my root accounts with hardware 2FA and my IAM Users with something like Google Authenticator. If I lose access to the authenticator app, I can use the root account to regain access. This is, typically, the only time I use the root account, but there are [other uses](https://docs.aws.amazon.com/general/latest/gr/aws_tasks-that-require-root.html).

## Protecting IAM Users with 2FA and Custom Policies

IAM Users can easily protect console logins by configuring 2FA. If you provision AWS Access keys and Secret Keys though, those are not protected by anything else. Those keys, by default, are able to run with permissions granted to the IAM user by anyone with access to the keys. If you create a user with Full Admin permissions, they are only slightly less powerful than the root user, but still capable of causing plenty of damage.

To help mitigate this, each of my accounts has a single IAM user with Full Admin Rights active at any single time, however, I use a custom policy to enforce 2FA on CLI commands. Each user has this policy attached:

~~~json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Admin2FA",
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
~~~

The key statement is  `"aws:MultiFactorAuthPresent": "true"` which requires all requests to be authenticated with MFA. For console access, that means you used you 2nd factor to login. For CLI access, this means the only useful commands you can are STS GetSessionToken (and a handful of others). This uses long lived key and secret key to request temporary credentials that are then used to authenticate each API request.

I wrote a [simple tool](https://github.com/remotephone/aws-learning/blob/master/aws-sts.py) that you can use to request temporary credentials. This automatically requests and updates your existing ~/.aws/credentials file so next time you authenticate, you are protected with MFA. This is what my credentials file looks like. After running an `aws configure`, I manually add the MFA token ARN to the file.

~~~
  1 [test]
  2 aws_access_key_id = AKIAACCESSKEY
  3 aws_secret_access_key = MYSECRETKEYMYSECRETKEYMYSECRETKEY
  4 mfa_serial = arn:aws:iam::123456789012:mfa/mytestadmin
~~~

Here's a run through making a request before requesting my session token and after (with sensitive details removed):

~~~

[me@workstation:~/] 
$ aws configure --profile test
AWS Access Key ID [****************SHHH]: AKIAACCESSKEY
AWS Secret Access Key [****************SHHH]: MYSECRETKEYMYSECRETKEYMYSECRETKEY
Default region name [us-east-1]: 
Default output format [json]: 
[me@workstation:~/] 
$ aws s3api list-buckets --profile test

An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
[me@workstation:~/] 
$ vi ~/.aws/credentials     #### Added line 4 from above
[me@workstation:~/] 
$ python3 aws-sts.py --profile test -t 923485
Configured profile "test-mfa" to use "us-east-1" region
[me@workstation:~/] 
$ aws s3api list-buckets --profile test-mfa
{
    "Buckets": [
        {
            "Name": "redactedbucketname",
            "CreationDate": "2018-07-21T03:59:01.000Z"
        },
....
~~~

Pretty sure since I put this together, other tools have come out that do similar things, but this works and I haven't had much of a need to look further into it. Maybe when it breaks I'll look at other options or rewrite it to be less clunky, but it works well enough for now. Tokens will expire 4 hours after creation unless you specify a different time.

## That's about enough

This is starting to get long winded, so I'm wrapping up. If you're following along, you should be caught up on AWS concepts, have a single account that will be the primary account for your organization, protected it with 2FA, and know how to securely authenticate and make requests. Part 2 will go into things like billing alerts, logging, and creating your subaccounts. Thanks for reading!
