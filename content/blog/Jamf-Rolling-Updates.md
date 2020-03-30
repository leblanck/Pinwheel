---
title: "Jamf Rolling Updates"
date: 2020-03-26T17:06:36-04:00
draft: false
tags: ["jamf", "updates", "bash", "jss"]
description: "Creating a rolling (phased) update process similar to SCCM"
meta_img: ""
author: "Kyle LeBlanc"
---


**Coming Soon!**


**tl;dr** This process was created to replicate something *similar* to that of phased roll-outs in SCCM. Groups get increasingly larger as the roll out continues. This process is intended to be used with Jamf. It is available on my [github](https://github.com/leblanck/macOS-Scripts). If you'd like to learn more about this process please continue below!

**What, Why, & How?**

If there is one thing I wish that Jamf had built-in it would be a rolling update feature. Something where it will auto populate/scale based on total enrolled machines. 

I have little-to-no experience in SCCM but I do recall co-workers being able to slowly roll-out updates/softare using 'phased update groups'. Doing so would allow to start deploying to 10% of devices, then the following 15%, 25%, 50% each time. 

TABLE OF GROUPS HERE

I am often asked by Product Owners/Scrum Masters if this is easily done and until now my answer was, unfortunatley, no (at least not in an automated, easy way :( ).

![Script Notes](/img/rollingNotes.png)
