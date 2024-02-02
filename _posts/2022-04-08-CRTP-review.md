---
title: Certified Red Team Professional Review
date: 2022-04-08 11:15:00 +01:00
modified:
tags: CRTP AD
description: My personal review on the Certified Red Team Professional course.
---

# Introduction
On 10 March 2022, I officially became a Certified Red Team Professional. After passing the eCPPT I wanted to gain more knowledge of Active Directory exploitation so I decided to give PentesterAcademy a try. I am not completely new to Active Directory since I've worked as a System Administrator for some time, and had several classes in school about setting up AD environments. At the time the course was around 250$ included with 1 exam attempt. So I decided to buy it after some good recommendations. The reason I want to gain more knowledge of AD exploitation is that 90% of Enterprise networks are based on Active Directory. And sooner or later you will stumble across it.

# About the CRTP Course
The CRTP course consists of several videos presented by Nikhil Mittal, included with a lab environment and a PDF file. Some of the following topics were covered:
- AD Enumeration (Users, Groups, Computers, ACLs, GPOs, Permissions, etc..)
- Bypassing Defenses (AMSI, Constrained Language Mode, AV)
- Local/Domain Privilege Escalation
- Domain Persistence Techniques
- Cross Trust Attacks
- Abusing SQL server trusts
- Defending Active Directory

# About the Exam
The exam is my experience is really fun! You can start the exam whenever you want. You should keep in mind that the support team is located in India and may have a different time zone, so they may have long response times. You will get 24 hours to exploit 5 machines that are all up to date. The only way to can exploit a machine is by using misconfigurations. After completing the exam you have another 48 hours to write a report which should have detailed steps on how you exploited the machines. I started my exam on 5 march, send my report on 6 March and on 10 March I received the good news that I successfully passed the CRTP! 🎉

<img src="/assets/img/crtp.png">

The good thing about the exam is that it is not very CTF like, it could be a real-life scenario which in my opinion is a big plus. You can pass this exam using all the tools that are used throughout the course. Keep in mind that these tools are not pre-installed in the environment, you have to import them yourself. Despite being stuck on the exam for a few hours I think 24 hours is easily enough to pass the exam. Bloodhound is always your friend and has been my go-to in this exam! 🙃

# My Opinion
In my opinion, the course is great, it is perfect for beginners with little to no knowledge of Active Directory. In the video Nikhil (the instructor) goes over the CRTP lab and gives a few learning objectives afterward. This concept worked well for me since it tests if you understand everything. If you understand all the videos and objectives you will be 100% able to pass the exam in one go. Another thing to note is that sometimes the material can be confusing because they overcomplicate things. I put some good resources at the end of the blog post. 👀

# Tips
If you want to also get the CRTP certification below are some of my tips that helped me a lot during the exam.

- **Creating a cheat sheet:** Within the exam you will find yourself looking up command syntaxes every time. To save time create a list of the commands, so you can copy and paste them without looking up the entire thing every time.
- **Prepare the reporting:** With "Prepare the reporting" I mean that you should make a report template to your liking, and find a good screenshot tool. I use [**Greenshot**](https://getgreenshot.org/). Preparing for this will prevent you from having to go back to the lab, take a screenshot and continue (If you still have access to it.. 😇)
- **Get comfortable with the lab objectives**: Everything that is mentioned in the course will come back in the lab objectives. This means if you understand every lab objective, what you're exactly doing, and why you're doing it. You can easily nail the exam.
- **BloodHound**: If you find yourself being stuck on the exam, you don't know what the next step is. Answer is: BloodHound 💓. I've found myself being stuck on some parts within the exam which is completely normal. Especially the "help" function is really good since it tells you what to do.

Another good source of informatie is an official PentesterAcademy discord group I found. Instead of e-mailing the support team everytime you can also ask you questions [**here**](https://discord.gg/w5HHwDcfeZ). Besides that anoter good website called [**ired.team**](https://www.ired.team/) helped my alot to understand certain attacking techniques.

*Example of what the front page of my report looked like:*
<img src="/assets/img/rapport.png">

# So what’s next?
Soo... after passing the CRTP exam, I am currently trying to get my hands on the eCTPX from eLearnSecurity. The eCPTX also focuses on Active Directory exploitation using Kali and C2 frameworks. I think I'll be busy for quite some time with the eCPTX because of the ginormous amount of learning material and lab objectives. I'll be posting on interesting topics of this certification once in a while.
