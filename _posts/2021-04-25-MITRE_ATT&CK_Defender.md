---
layout: post
author: remotephone
title:  "MITRE ATT&CK Defender - My Thoughts on Training and Certification"
date:   2021-04-25 23:22:00 -0600
categories: lab homelab training workflow
---

# MITRE ATT&CK Defender - New and Educational

I was lucky enough to be able to go through the MITRE ATT&CK Defender training and certification process and I wanted to share my thoughts on it. Overall, I was pleased with it, but people should be realistic about what skills it will introduce you to and develop. I think it delivers good value for the price, especially if they are able to deliver their third course within the first year. 

## My Background

I have been doing incident response work for about 7 years now, but only took a haphazard approach to threat intelligence. Over the last few years, I have invested a lot of time and effort into applying frameworks like MITRE ATT&CK to defenses at work in order to evaluate coverage and make recommendations for changes. 

By the time I signed up for this training, I had had experience evaluating existing detections in the context of MITRE ATT&CK, mapping attacker techniques to the matrix, using the ATT&CK navigator to visualize detection and attack coverage, using MITRE ATT&CK to explain attack and defense to non-incident responders, and generally felt pretty comfortable with the concepts, but knew I had a lot to learn and improve on.

What I expected to get from the course was a structured introduction to concepts I had previously approached out of order. I hoped to get an understanding of how to apply these concepts systematically, minimize bias in how I approached topics, and learn some new approaches to using data and concepts I thought I understood. 

## Course Format

The content is currently broken up into two tracks with one introductory course as a prerequisite for both. You'll take the introduction course called MITRE ATT&CK Fundametals before moving on to the main content. You'll cover how the MITRE ATT&CK matrix is set up, where the data behind it comes from, all different parts of what maeks up a technique. The last part was most interesting as I'd seen a lot these pages, but honestly didn't put thought into very section every time I went to one. 

For example, on [Command and Scripting Interpreter: PowerShell](https://attack.mitre.org/techniques/T1059/001/), we have the description of the technique with some useful technical details. Then we have a long list that personally I don't always scroll all the way through of `Procedure Examples`. Each of these examples is interesting on their own, but some of the most important content that's easy to miss is underneath at `Mitigations` and `Detections`. I would usually read through the description at the top and then go chase down 3rd party sources but its nice to have a reminder a lot of it's here. One thing I really appreciated about the training was it highlighted the importance and potential uses for some of these sections. 

Each course consists of several modules broken down into several 5-15 minute lessons. Once you complete a module, you'll take a knowledge check. Especially early on, I found these pretty straigthforward. Each module corresponds to a test and each test completed with 80% correct or more will earn you a completion badge. The tests are not cumulative and focus only on the content in the corresponding module, though the material does build on itself. 

The complete list of badges that correspond to modules are as follow:

- ATT&CK® Fundamentals  
  - ATT&CK® Fundamentals Badge  
- Cyber Threat Intelligence  
  - CTI From Narrative Badge  
  - CTI From Raw Badge  
  - Storing and Analyzing CTI Badge  
  - Defensive Recommendations from CTI Badge  
- SOC Assessment Certificate  
  - ATT&CK® Fundamentals Badge  
  - SOC Assessments Fundamentals Badge  
  - SOC Analysis Badge  
  - SOC Synthesis Badge  
 
Earn all badges under a certificate and you earn the cert. Pretty straight forward. 


## Cyber Threat Intelligence

This was my main interest in taking the course. I found the material relevant, informative, and it helped me take notes I could apply to my day to day work very easily. I've read through threat intel reports and identified techniques and subtechniques written about when not flagged by the writer, but the description of how they recommend you go through narrative reports was very interesting. Their methods included focusing on verbs ("The attacker scanned", "The attacker comrpomised", etc), identifying all of them, and then finding the relevant subtechnique or technique while keeping in mind the attacker's goal during the phase of the attack to make sure you are mapping techniques that appear under multiple tactics to the same tactic. 

I find I learn the best when I can understand what people more skilled than me are thinking when they go through their processes. Understanding why someone with experience makes the decisions they do has been a very effective way for me to internalize those concepts and be abl to apply them. Advice like that  through this course was excellent for me, I was able to quickly apply it in my day to day work. 

## Security Operations Center Assessment

This was not the msot interesting part of the course for me when I started, but as I went through the material, it really grew on me. It applies the lessons of ATT&CK to reviewing where a SOC stands and how it can improve. Their definition of a SOC seems a little different than the one I am familiar with. When I hear "SOC", I think of operational people responding to alerts, notifications, and incidents, and escalating them within the SOC until they're resolved. Their definition of the SOC seems to wrap up most of the security organization, from management, the SOC SOC I think of, detection engineers, red teamers, and others. 

This section covered how to take the MITRE ATT&CK framework and apply it to an organizations processes, detections, and mitigations to see where coverage of known attacks exists and where it can be improved. It provides several complimentary methods for conducting the assessment, ranging from interviewing staff to actual tests of controls through red team type exercises. 

A large part of this part of this course revolved around using the MITRE ATT&CK Navigator. This is a tool you've probably used, but it's likely not at the level this course will have you use it. The course gets you familiar with many of the features of the tool, having you map detections and scenarios against the matrix to get specific scores for coverage in a simulated organzation. You'll get pretty experienced making and combining layers to create cumulative heat maps.

Finally, there is a big focus in those part of the course on how you present the information you gather to others. This is important because we don't operate in a vacuum, and while the MITRE ATT&CK heat map you might generate from an evaluation is extremely interesting and informative to look at, some choices (distracting or misleading coloring, non-informative gradients of colors, conveying the right information to the wrong audience, etc) can your presentation more effective than others. The course covers this well and gives you a good framework for thinking about how to present the results of an evaluation.

## My Exam Hiccup

While the SOC Assessment part of the training had a lot of very useful, practical information, it was also the part with the only exam I didn't pass with an adequate score on the first try. You're required to gett 80% or greater to pass and did on all lessons, except the last one where I got a 70%. A lot of the questions in this section had you apply their formula to add and subtract coverage levels to techniques in the matrix. The questions require you to apply their formula very consistently and arrive at the exact values they calculated for coverage. 

At the time I took the exam, I felt the training introduced me to conducting assessments that were part art and part science, but the exam asked me to treat them as exact sciences. I don't want to go too much into specific details about exam questions, but it should suffice to say some questions require very exact answers when the course material itself said there were several ways to score and present the same results, depending on the audience and goals of the assessment. 

I am certain that I was under more time pressure the first time I took the test. The second time, I spent the time to review the notes as I took the test, create the layers one by one, and add them following their formula, reviewing examples as I did, and passed. My point here isn't to complain about the exam, I think MITRE did an excellent job of developing this material with the inherent constraints of applying multiple choice and fill in the blank questions to processes that can be tough to teach without repeated hands on practice.

I would be interested to talk to someone else who went through this material to see what their impression of the course and exams looked like. I felt pretty well prepared for the exam but in retrospect, I could have spent more time intentionally working through the questions. 


## Concluding Thoughts

While I did pass the exam on the second try and earn all badges necessary to earn both certifications, I know that the point of the courses isn't to earn the certifications, but to learn the material. The course covers the meterial in depth, applies it to practical situations, and has tests to ensure you gained the knowledge. I took a ton of notes throughout the course, with a big focus on the how of applying the concepts. 

I feel pretty good knowing that the approach I had been taking to reviewing published detections was pretty close to what the course authors recommended, but I learned a lot about how to think about the evaluations and how to present what I learned to others. As a result of this course, I present information about the MITRE ATT&CK framework to people differently than I might have previously and look back on some material I've created in the past and see good opportunities for improvement.

Overall, I would recommend this course to folks interested in threat intelligence or approaching improving their detections in a systematic way. It won't take you from zero to hero or anything, but it will give you structure around how you think about MITRE ATT&CK and prepare you with concrete processes you can apply to evaluating and improving your detections and mitigations. 


## References
[MITRE ATT&CK Defender Course Catalog](https://mad.mitre-engenuity.org/course-catalog/)
