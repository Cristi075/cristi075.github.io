---
layout: post
title:  "HTB HackTheBoo 2022 - (Forensics) Halloween Invitation"
date:   2022-10-27 18:00:00 +0300
categories: HackTheBoo_2022 CTF forensics vba_payload
summary: Writeup for the Halloween Invitation challenge (Forensics3/5) from HackTheBoo 2022. This challenge involved analyzing an infected doc file.
---


'Halloween Invitation' was a forensics challenge (day3 out of 5) from HackTheBox's HackTheBoo CTF.  

### Word file analysis

For this challenge, we got a .doc file.  
Note: the file triggers anti-malware solutions (it is infected...) so you'll have to deal with that by setting exclusions (or better, by using a VM).

We start by unzipping the doc file to see its content.
![Unzipping the file]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/unzip.png){: .center-image}

We can see a *vbaProject.bin* file so this doc probably contains VBA macros.  

We can use [oletools](https://github.com/decalage2/oletools), specifically olevba, in order to extract VBA code from that file.

![Extracting VBA code]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/vba_code.png){: .center-image}

### Analyzing the payload

In the extracted VBA code we notice that there is a large block that only contains concatenated hex strings.  
![Encoded payload - hex]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/encoded_payload.png){: .center-image}
We use the 'find & replace' feature of Visual Studio Code (or any other text editor) to remove the variable names, function names/calls and quotes and leave only the hex strings.

Then, we load the resulting string in CyberChef and use the 'From Hex' operation.
![Decoded payload - hex]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/decoded_1.png){: .center-image}

The result is a list of integers. We try to convert those to text by using 'From Decimal' but this doesn't work.  
It looks like there are some errors in the resulted string: we can see 4 or 5 digit integers there when we would expect to only have 2 and 3 digit integers (<255 for extended ASCII).  
I'm not sure if this was caused by how I extracted those strings, but I have a quick solution.  

There are a few of those errors and we can fix them manually.  
We search for '\d\d\d\d' in regex mode (most text editors can do this) and we highlight all the results.  
This way, those issues are highlighted and we can fix them very quickly.  

![Correcting errors]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/errors.png){: .center-image}

### Getting the flag

With the corrected payload, we go back to CyberChef and try to use 'From Decimal' again.

![Decoding ASCII]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/cyberchef1.png){: .center-image}

The result is a base64 string, so we decode that as well.

![Decoding Base64]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/cyberchef2.png){: .center-image}

Now the result is some powershell code (and the flag, at the end). However, we can see a null character after every valid character.  
In order to get rid of those we can use 'Decode Text' and select a UTF-16 encoding for that (Windows uses wide chars and we are trying to decode 8-bit chars).  
Or, we can use the 'Remove null bytes' operation to remove the extra characters. 

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/halloween_invitation/flag.png){: .center-image}

With this we can both easily see the attacker's payload and read the flag. 

### Other forensics challenges from this CTF

This CTF released a challenge in each of its 5 categories every day.  
I have writeups for all the forensics challenges. Here are some links to them:
- Day 1 - [Wrong Spooky Season](/HTB-HackTheBoo-2022-Forensics1-Wrong-Spooky-Season)
- Day 2 - [Trick or Breach](/HTB-HackTheBoo-2022-Forensics2-Trick-of-Breach)
- Day 3 - Halloween Invitation (you are here)
- Day 4 - [POOF](/HTB-HackTheBoo-2022-Forensics4-POOF)
- Day 5 - [Downgrade](/HTB-HackTheBoo-2022-Forensics5-Downgrade)
