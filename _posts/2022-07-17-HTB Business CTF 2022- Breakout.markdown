---
layout: post
title:  "HTB Business CTF 2022 - Breakout writeup"
date:   2022-07-17 22:00:00 +0300
categories: HTB_Business_CTF_2022 CTF reverse_engineering
summary: Writeup for the Breakout challenge from HTB's Business CTF from 2022. 
---


[Breakout](https://ctftime.org/task/22947) was a challenge at the HTB Business CTF 2022 from the 'Reversing' category.  
For this challenge, we got an IP address and a port.  

### Recon 

It looks like the target port has a http service running on it.  
The http service allows the user to access the filesystem of a linux server.  

One entry from that listing stands out as it's not something you would see on a standard linux installation: a "bkd" (backdoor?) binary placed in /.  

![Directory listing]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/breakout/dir_list.png){: .center-image}

### The BKD binary

We download the 'bkd' file and identify it as being an ELF file.  

This is probably what is running the server that we're interacting with. We should confirm that first.  
To do that, we access /proc/self/cmdline and see what was the cmdline used to run the current process.  

![Cmdline of BKD]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/breakout/proc_cmdline.png){: .center-image}

Now we know for sure that's the binary that we're interacting with.

### Analyzing the binary

We use some simple tools like binwalk and strings in order to find information about this binary.  
Binwalk detects a HTML file. I use foremost to extract it and read it. Mainly, this reveals the existance of an /exec path on the server but nothing more.  

![Strings found in the BKD binary]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/breakout/strings.png){: .center-image}

We open the binary in Ghidra and start looking around. We find strings related to the "/exec" endpoint and also a /ping endpoint and a /secret endpoint.  
We try accessing them on the server.  
- /ping responds with "pong" 
- /secret responds with a 403 response (Forbidden).

Using Ghidra, we look at the function that's most likely used for handling the /secret endpoint (the one named 'secretGet') and find some integers in there that look out of place.  

![Int variable]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/breakout/ints.png){: .center-image}

### Decoding the flag

We try to put the value of those integers in CyberChef and use the "From Hex" option.

![First attempt at decoding]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/breakout/cyberchef_incomplete.png){: .center-image}

That ends up looking similar to what the flag might look like, but it's not exactly it.  
The data seems to be reversed: we can 'fix' that by using the swap endianness operation from CyberChef.  
I'm not familiar enough with this to know if this the data should be stored using the little endian format or if this was just some obfuscation thing.  

![Reveal the flag with cyberchef]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/breakout/cyberchef.png){: .center-image}

But in the end, by doing that we got our flag: HTB{th3_pr0c_f5_15_4_p53ud0_f1l35y5t3m_wh1ch_pr0v1d35_4n_1nt3rf4c3.....}