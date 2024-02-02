---
title: Certified Red Team Operator review
date: 2022-09-24 11:15:00 +01:00
modified:
tags: CRTO AD CobaltStrike
description: My honest review of the CRTO course.
---

# Introduction
On 20 September  2022, I officially became a Certified Red Team Operator 🥳. I was studying for the eCPTX certification, however due to my bad experience with their labs I decided to go for another Active Directory certificate: Certified Red Team Operator. The main reason for me to take the course was because of the extra experience you gain with Cobalt Strike, which in my opinion is valuable. The course was created by RastaMouse and is available on the [Zero-point Security](https://training.zeropointsecurity.co.uk/courses/red-team-ops) platform. The course teaches the basic principles, tools, and techniques, that are synonymous with red teaming.

It covers the core concepts of adversary simulation using Cobalt Strike, and how to plan an engagement from beginning to end. It teaches the attack lifecycle from initial compromise to full domain takeover, data hunting, and exfiltration. With keeping operation security (OPSEC) in mind. 🥷


# About the Course
The RTO course consists of several videos and slides presented by Zero-point Security, which includes a lab environment. Some of the things that are mentioned in the course:
- Setting up an engagement (Scoping, planning, RoE);
- Command & Control;
- Reconaissance and persistence;
- Lateral Movement;
- Data Protection API;
- Domain Dominance;
- Bypassing Defences;
- Extending Cobalt Strike.

# About the Exam
The exam in my experience wasn't that hard. It was a bit harder than the CRTP exam I did earlier but wasn't anything special. Once you plan the exam, you receive a document with all the information you need to build your threat profile and additional information you need. The exam had a few bugs, which made it a bit harder. In the image below you see my scoreboard for the overall exam. 

<img src="/assets/img/scoring.png">

The great thing about the exam is that you have to build your threat profile, to simulate a real-life APT. And you have your build your own tools to bypass any security restrictions that may be in place. To pass the exam, you need to find 6/8 flags, without sending a rapport 😀

# My Opinion
In my opinion, the course is great, however, if you don't have a lot of experience with Active Directory I would suggest you start doing the CRTP first. The course was clear and easy to read through. I had no problems understanding any of the explained attacks or methodology. If you want to pass the RTO exam, you should familiarize yourself with everything in the course and you'll be ready to take on the exam. It took me around 2-3 months of learning combined with a 4-5 day job.

# Tips
Some tips I've used during my RTO exam:
- Build your threat profile, before starting the actual exam. This will save you a lot of time! 
- Understand how you can build/modify your tools, so they won't get caught.
- Take notes during the course, and create a cheat sheet. This will save you a lot of time. You can simply copy and paste commands into your beacons.

# So what’s next?
After passing the CRTO course, I will now try and get my OSCP certification. I'll try to keep you people up to date with my experiences with Offensive Security.
