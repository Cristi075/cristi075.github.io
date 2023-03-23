---
layout: post
title:  "HTB Cyber Apocalypse 2023 - (Hardware) Critical Flight"
date:   2023-03-23 15:00:00 +0300
categories: HTB_Cyber_Apocalypse_2023 CTF hardware very_simple
summary: Writeup for the Critical Flight (Hardware, Very Easy) from HTB Cyber Apocalypse 2023. This very simple challenge involved some PCB designs.
---


'Critical Flight' was one of the challenges in the 'Hardware' category at HTB's Cyber Apocalypse 2023.  
Its difficulty was 'Very easy' so it was a very simple challenge. This makes it a very good challenge to try for people who tried their first CTF.  

![Challenge files]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/critical_flight/challenge_files.png){: .center-image}

For this challenge, we got an archive containing two folders: a __MACOSX folder and a folder containing .gbr files.  
The __MACOSX is specific to archives on MacOS so we can ignore that (especially since I'm not using MacOS).  

Let's focus on the .grb files. After a bit of googling, I found out that .gbr is a format used to store designs for Printed Circuit Boards (PCBs).  
I also found out that there's an online viewer available for those files: [https://www.pcbway.com/project/OnlineGerberViewer.html](https://www.pcbway.com/project/OnlineGerberViewer.html).

I dragged all the .gbr files into that viewer and it looks like every file is another layer on the PCB.  

![GRB Viewer]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/critical_flight/gbr_viewer.png){: .center-image}

I can select "layers" and I can select which layers I want to be visible.  
So, I started browsing through the layers to see what they contain.  

The first interesting thing can be seen on the Copper layer on the bottom of the board.
![First part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/critical_flight/flag_01.png){: .center-image}

This is the first part of our flag.  
We just have to rotate the image and copy the first half of the flag: **HTB{533\_7h3\_1nn32\_w02k1n95**  
Maybe the second part is also hidden in one of those layers.  

And after a very short time ... here it is, on the copper layer in the "inner" part of the board.

![Second part of the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/critical_flight/flag_02.png){: .center-image}

The second part is: **\_0f\_313c720n1c5#$@}**

We put them together and we get our full flag: **HTB{533\_7h3\_1nn32\_w02k1n95\_0f\_313c720n1c5#$@}**  
And this was it: very easy indeed.


### More hardware challenges

These are the other hardware challenges from this CTF
- Hardware 1/5 - Very Easy - Critical Flight (you are here)
- Hardware 2/5 - Very Easy - [Timed Transmission](/HTB-Cyber-Apocalypse-2023-Timed-Transmission)
- Hardware 3/5 - Easy - [Debug](/HTB-Cyber-Apocalypse-2023-Debug)
- Hardware 4/5 - Easy - [Secret Code](/HTB-Cyber-Apocalypse-2023-Secret-Code)
- Hardware 5/5 - Medium - [HM74](/HTB-Cyber-Apocalypse-2023-HM74)
