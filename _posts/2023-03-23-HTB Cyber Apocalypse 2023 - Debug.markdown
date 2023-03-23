---
layout: post
title:  "HTB Cyber Apocalypse 2023 - (Hardware) Debug"
date:   2023-03-23 15:00:02 +0300
categories: HTB_Cyber_Apocalypse_2023 CTF hardware UART
summary: Writeup for the Debug (Hardware, Easy) from HTB Cyber Apocalypse 2023. This challenge involved analyzing an UART signal.
---

'Debug' was one of the challenges in the 'Hardware' category at HTB's Cyber Apocalypse 2023.  
Its difficulty was 'Easy' and it involved UART signals captured by a logic analyzer.

We got a single file for this challenge: hw_debug.sal.  
I already knew about the .sal extension from the previous challenge so I opened the file in Logic2 right away.

![Viewing signals]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/debug/debug_01.png){: .center-image}

There are two visible signals: TX and RX. We can quickly see that only RX has any kind of activity.  

First, I tried adding an Async Serial analyzer from Logic2 to see if I can decode any of the data transferred there.  
I tried the standard baud rates: 1200, 2400, 4800, 9600, 19200, 38400, 57600, and 115200.  
115200 seemed to be the one that worked.

![Challenge files]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/debug/debug_02.png){: .center-image}

Now Logic2 also displays data that seems to be valid.  

![Challenge files]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/debug/debug_03.png){: .center-image}

I export that and get/copy those hex bytes that were transferred.  

Then, I put them in CyberChef and use the 'From Hex' operation.  

![Challenge files]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/debug/debug_04.png){: .center-image}

This seems to be the output of a debug shell.  
Let's take a look at what is written to that shell.  

![Challenge files]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/debug/debug_05.png){: .center-image}

At the end of the boot sequence, the device prints the flag in multiple lines and then asks the user to login (over UART, probably).

We put the pieces of that flag together and we have our flag: **HTB{547311173\_n37w02k\_c0mp20m153d}**


### More hardware challenges

These are the other hardware challenges from this CTF
- Hardware 1/5 - Very Easy - [Critical Flight](/HTB-Cyber-Apocalypse-2023-Critical-Flight)
- Hardware 2/5 - Very Easy - [Timed Transmission](/HTB-Cyber-Apocalypse-2023-Timed-Transmission)
- Hardware 3/5 - Easy - Debug (you are here)
- Hardware 4/5 - Easy - [Secret Code](/HTB-Cyber-Apocalypse-2023-Secret-Code)
- Hardware 5/5 - Medium - [HM74](/HTB-Cyber-Apocalypse-2023-HM74)
