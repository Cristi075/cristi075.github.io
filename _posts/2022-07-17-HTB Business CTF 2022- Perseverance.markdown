---
layout: post
title:  "HTB Business CTF 2022 - Perseverance writeup"
date:   2022-07-17 22:00:00 +0300
categories: HTB_Business_CTF_2022 CTF forensics
summary: Writeup for the Perseverance challenge from HTB's Business CTF from 2022. 
---

[Perseverance](https://ctftime.org/task/22938) was a forensics challenge from HTB's Business CTF (2022).  
For this challenge we got a zip archive that contains some WMI logs and the challenge text mentioned investigating a possible compromise.  

After unziping the archive that we got, we get a .BTR file, three .MAP files and a .DATA file.  
![ZIP contents]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/zip_contents.png){: .center-image}

The .BTR and .MAP files don't to contain anything interesting on their own after looking at what binwalk finds in them.  

DATA.OBJ has many interesting strings in it, but at first I wanted to see if there is a way to parse that file.  
After a bit of searching, I found out that there are some python scripts that can parse the WMI logs: a good example (the one that I used here) would be [https://github.com/davidpany/WMI_Forensics](https://github.com/davidpany/WMI_Forensics)  
We get that script from github and use python2 to run it and we get a single interesting result.  

![WMI powershell command]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/wmi_powershell.png){: .center-image}

There is a powershell instance that is executing a base64 encoded command.  
We can use CyberChef in order to decode that command. 

![Decoding the command with CyberChef]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/cyberchef_1.png){: .center-image}

The second CyberChef operation (text decode) was needed because the base64 string decoded to a string of wide chars (UTF-16, the standard on Windows).  
The decoded command uses [Win32_MemoryArrayDevice](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-memoryarray) in order to access the host's memory.  

After seeing that reference, I tried to look for other things that are related to that type of memory access using grep.  

{% highlight python %}
strings OBJECTS.DATA | grep -i "memoryArray" -C5
{% endhighlight %}
Note: The -C5 argument will display 5 lines above and below the line matched by grep.
![Finding another base64 string with grep]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/found_string.png){: .center-image}

It looks like we managed to find another string that might be related to the first one.  

We try decoding this one with CyberChef too. After using the "From Base64" we get a string that doesn't resemble anything.  
However, by using the "Magic" operation, it detected that by applying the "Raw Inflate' operation we'll get an executable file (you can see the MZPE header)

![Cyberchef: Magic & Raw Inflate]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/cyberchef_2.png){: .center-image}

We actually got an .exe file after applying that operation.  
By inspecting it manually, we can see something that looks like an encoded string. 

![Using wide chars]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/wide_chars.png){: .center-image}

We can use the "Decode Text" operation like before in order to see the text better.  
However, that will have the side-effect of turning everything else unreadable.  

![Base64 string]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/base64_flag.png){: .center-image}

The string that starts with "SFR" is one of the strings that look like base64.  
Now if we take that string and base64 decode it, we get the flag.  

![Base64 string]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/perseverance/flag.png){: .center-image}