---
layout: post
title: The OSCP And Me
---

## Why the OSCP?  

I’ve been doing software development, but I felt like making a switch to security.  Since I didn’t have any experience, I thought the [OSCP](https://www.offensive-security.com/information-security-certifications/oscp-offensive-security-certified-professional/) would help me get my foot in the door.  It’s widely respected because the exam is hands-on.  You’re required to break into several machines in 24 hours.  Most of the other certs are multiple-choice.

## Background

I’ve done work in iOS and Python.  I’ve also done some web development (MySQL, JavaScript, PHP, etc.).  That experience came in handy when I had to modify exploit code and automate grunt work.  I’ve set up VPS servers as backends, which gave me exposure to Linux. You don’t need to be a sysadmin or a developer to pass this course, but you should know the basics.

## Course

How much lab time should you sign up for?  That depends on your background and your schedule.  If you have a lot of pentesting experience and free time, it’s possible to do it in 30 days.  If you have solid sysadmin skills and you can code, try 60 days.  For everyone else, I’d recommend 90 days.

I signed up for 90 days, and then I extended that by another 30 days. You get a free exam attempt for signing up.  If you buy more time, they give you another free exam attempt, but they don’t stack up. So if you buy 60 days, don’t take the exam, and then buy another 30 days, you only get one attempt.

After you sign up, you get a connection pack that gives you access to the lab via VPN.  They also email you links to the course manual and lecture videos.  Download those ASAP, since these links expire.  Also, store backup copies on a USB.  If you request another, they’ll charge you $100.

## Labs

In the lab, there are 50 machines you can break into.    You don’t have to do all of them. Some boxes are dual-homed so check every machine’s ifconfig or ipconfig output.  If it’s dual-homed, there will probably be a network-secret.txt file you can use to unlock the other networks.

The machines are there for you to develop your process.  This is the single most important thing you can do to prepare for the exam.  Whenever I got a shell or root, I updated my process to check for that vector.  

Look for the low hanging fruit. This will help in the exam, as well.  If you’re stuck, and you’ve already done a lot of enumeration, move on the next host, then circle back. Some of the machines rely on an obscure detail that’s easy to miss.  Others hosts rely on clues found in another machine, so do your post-exploitation.

After a while, you’re going to run out of easy machines, and then you’ll hit a wall.  If you’ve never experienced it before, there’s nothing quite like grinding away, hour after hour, trying one attack vector after another, and getting that sinking feeling that you’ve wandered down into a rabbit hole.  This is when you’ll learn the true meaning of the Offsec motto: “Try Harder.”

## The Forums Are Your Friend

If you’ve done your enumeration and you’re still stuck on a machine, check the forums.  The admins will remove any obvious spoilers.  What’s left is a collection of vague references that will only make sense to someone who’s done the enumeration.

The forums also cover the course modules as well.  Sometimes, you’ll run into instances where the examples in the manual don’t work on your machine.  When that happens, it’s helpful to ask if it worked for anyone else.  

I didn’t use the IRC channel, but from what I’ve heard, the admins are always going to err on the side of telling you to try harder.  If you’ve shown them you’ve done all the enumeration, they might give you a clue.

## Scripting

Learn how to script everything with Python, Perl, or bash.  This applies to both enumeration and post-exploitation.  There are a number of scripts that do enumeration for you, but you should also look around manually as well.  Check the user’s home directories, desktop, /etc/, and anything that looks interesting.

## Backups

If you’re running Kali in a VM, take frequent snapshots.  Try to take one daily, or after you’ve made progress on a host.

## Metasploit

Learn to use Metasploit, but don’t rely on it.  In the exam, they only let you use the post and exploit modules on one host.  You can use multi/handler, meterpreter, and msfvenom on any host, so familiarize yourself with them.  Lastly, don’t give up if a payload doesn’t work. Sometimes, you just need to try a different one.

## Before the Exam

Offsec will give you a link to the exam instructions ahead of time.  Be sure to read ALL the directions.  In particular, read the part about how to do proper screenshots for the proofs.  If your monitor can’t fit all the lines in one screenshot, try temporarily decreasing the font size of the terminal app.  Also, make sure you have your shells, privesc exploits, and other files organized into folders, for quick access.

## Exam

For the exam, there are five hosts, each of which is worth 10-25 points. You need at least 70 points to pass.  To get full points, you need to get root on a machine.  Getting a limited shell only gives you partial credit.  

At 2 PM, they emailed me the connection files.  I logged in and started scanning the network.  The first scan found some interesting ports, so I worked on those, while the other scans completed. I also started brute-forcing the directory names and logins.    At 8 PM, I got a limited shell on a 20-point machine.  At 2 AM, I got admin access on the machine.   I hadn’t been working on just that one machine the whole time. I had been doing research on all the services I found on the other machines, so I had promising leads on most of them.

Around 5 AM, I got a shell on a 25-point machine.  Two and half hours later, I got the privilege escalation working.  After a nap, I worked on the buffer overflow machine.  If you’ve done the homework, you shouldn’t have a problem writing the exploit.  I had 70 points, which was enough to pass, but I wanted some more just in case.  At 12:30 PM, I knocked over the 10-point machine, giving me 80 points.  For the last 20-point machine, I had a pretty good idea of what I had to do to get a shell.  I had done something similar in the labs, but this one had a different feature.  After you pass the exam, they give you access to a “graduate” forum, so I found out I had been on the right track.  Anyways, at that point, I had been up for 22 hours, so I called it a day.

Overall, I thought the exam machines were more straightforward than some of the lab machines. If you do your enumeration properly, you should be able to pass.  

## Report

After staying up all night for the exam, I crashed and slept in.  I then relaxed for a few hours before starting the report.  Mistake!  Start your report ASAP after you wake up.  Better yet, start setting it up before the exam.  If you’re not a MS Word guru, you don’t want to fiddle with the formatting while you’re in a rush.  Before you start the exam, open up the report template and look at the vulnerabilities section.  There are two sample entries (IP address, severity, remediation, etc.)  Due to the formatting, copy and paste doesn’t work as well as you think it would.  Have multiple blank entries ready to go, each with the same formatting as the sample ones. Make sure each item has a source code section.  You can always delete that if you don’t need it.   

Other tips:

* Make sure you save multiple copies while you’re writing the report.  
* Fill in the report directly.  I wrote mine in Keepnote first, and then transferred it, which took longer than I expected.
* If you don’t know what a vulnerability’s severity rating is, look it up on [cvedetails.com](https://cvedetails.com).
* Pay attention to the directions for submitting your report. Use the web app to submit your report.  Their server rejected my email because of the pdf attachment.

## After the exam

After the exam, Offsec emails you the result within 3 business days.  I got an email a day later:

![alt text](/images/results.png "Results")

I received my certificate a month after the exam. Printed on the folder were the words, “I tried harder.”

## What’s next?

Getting a job.  In the meantime, I’m working on appsec.  The course covered it, but not in depth.  I’m working through this book and also learning how to use Burp Suite.  
