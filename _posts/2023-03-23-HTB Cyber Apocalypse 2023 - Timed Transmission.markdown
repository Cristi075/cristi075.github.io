---
layout: post
title:  "HTB Cyber Apocalypse 2023 - (Hardware) Timed Transmission"
date:   2023-03-23 15:00:01 +0300
categories: HTB_Cyber_Apocalypse_2023 CTF hardware very_simple
summary: Writeup for the Timed Transmission (Hardware, Very Easy) from HTB Cyber Apocalypse 2023. This very simple challenge involved a capture from a logic analyzer.
---

'Timed Transmission' was one of the challenges in the 'Hardware' category at HTB's Cyber Apocalypse 2023.  
Its difficulty was 'Very easy' so it was a very simple challenge. This makes it a very good challenge to try for people who tried their first CTF.  

For this challenge, we got an archive containing a single file: Captured_Signals.sal.  

After searching for it online, I found out that this extension is used by Saleae's Logic2 software.  
This can be downloaded from the vendor's website ([here](https://www.saleae.com/downloads/))

After installing Logic2, I opened the Captured_Signals.sal file and I could see 5 signals being captured.  

After I spent almost half an hour trying to find a protocol that uses those signals like that (JTAG was a close one in my mind), I finally tried zooming out.  
![First part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/timed_transmission/flag_01.png){: .center-image}

This looks a lot like the beginning of a flag: **HTB{b39**  

So I scrolled through the timeline to see if the whole flag can be found like this.  

![Second part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/timed_transmission/flag_02.png){: .center-image}
Here's the second part: **1N\_tH3\_**

![Third part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/timed_transmission/flag_03.png){: .center-image}
The third part: **HArdWA**
![Fourth part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/timed_transmission/flag_04.png){: .center-image}
Fourth part: **r3\_QU3**

![Last part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/timed_transmission/flag_05.png){: .center-image}
And the last part: **3St}**


And we put those together to get the full flag: **HTB{b391N\_tH3\_HArdWAr3\_QU3St}**  
This was easy ... once I zoomed out

### More hardware challenges

These are the other hardware challenges from this CTF
- Hardware 1/5 - Very Easy - [Critical Flight](/HTB-Cyber-Apocalypse-2023-Critical-Flight)
- Hardware 2/5 - Very Easy - Timed Transmission (you are here)
- Hardware 3/5 - Easy - [Debug](/HTB-Cyber-Apocalypse-2023-Debug)
- Hardware 4/5 - Easy - [Secret Code](/HTB-Cyber-Apocalypse-2023-Secret-Code)
- Hardware 5/5 - Medium - [HM74](/HTB-Cyber-Apocalypse-2023-HM74)
