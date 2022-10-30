---
layout: post
title:  "HTB HackTheBoo 2022 - (Forensics) Trick of Breach"
date:   2022-10-27 18:00:00 +0300
categories: HackTheBoo_2022 CTF forensics network_forensics dns_exfiltration
summary: Writeup for the Trick of Breach challenge (Forensics2/5) from HackTheBoo 2022. This challenge involved analyzing some DNS traffic and reconstructing exfiltrated data.
---


'Trick or Breach' was a forensics challenge (day2 out of 5) from HackTheBox's HackTheBoo CTF.  

### Network analysis. Exporting data

The only thing that we got for this challenge was a pcap file.  

We open the file with wireshark and the only thing that we can see is DNS traffic.  
Each request is a query for *hex_string*.pumpkincorp.com. For each request we can also see a response from the DNS server.

![Opening the pcap in wireshark]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/wireshark.png){: .center-image}

A good starting point would be to get those hex strings and check if they mean anything.  
For that, the first step is exporting the data using tshark. (you can also use wireshark for this, tshark was easier for me)  

{% highlight bash %}
tshark -r capture.pcap -x -T json -n "dns" >> data.json
{% endhighlight %}

![Exporting data with tshark]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/tshark.png){: .center-image}

### Extracting the data

Now we got the data in a json file we can process it with a tool like [jq](https://github.com/stedolan/jq) in order to get only the relevant parts.

First, we want to get only the DNS query out so we'll use this query for that.

{% highlight bash %}
jq '.[]._source.layers.dns.Queries | keys | .[0]' data.json'
{% endhighlight %}

![Extracting data - jq]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/extracting_data1.png){: .center-image}
Note: you can (and should) develop that query one step at a time. You start from scratch and add each of its components one at a time so you can understand why each piece is there.

Something that clearly looks 'wrong' is that every line appears twice.  
That's because we extracted both the DNS query and the server's response. It might've been a smart thing to either use a tshark filter or a select in jq to get only one of those.  
However, we can fix this easily by using awk. We'll display only the lines that have an even index (so, every other line).  

{% highlight bash %}
jq '.[]._source.layers.dns.Queries | keys | .[0]' data.json | awk 'NR % 2 == 0'
{% endhighlight %}
![Extracting data - remove every other line]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/extracting_data2.png){: .center-image}

Now we have the data without any duplicates. However, we still need to remove the quotes at the beginning and the last part of each line (".pumpkincorp.com: type A, class IN").  
The lines are the same length so this is easy to do with awk: we'll remove the first character and the last 35 characters from every line.  
After we add that part, the command looks like this.

{% highlight bash %}
jq '.[]._source.layers.dns.Queries | keys | .[0]' data.json | awk 'NR % 2 == 0' | awk '{ print substr($0, 2, length($0)-36) }'
{% endhighlight %}
![Extracting data - extract hex string]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/extracting_data3.png){: .center-image}

### Recovering the file

We put the hex strings in CyberChef and use the "Form Hex" operation.  

![Decoding hex]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/decoding_hex.png){: .center-image}

The first characters (PK) hint that this could be a zip file. So we download it and try to unzip it.  

![Unzipping]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/unzip.png){: .center-image}

The file gets recognized as a Microsoft Excel file. We can unzip that ... so we do.  

Then, we take a quick look at xl/sharedStrings.xml and we notice our flag.  
No need to open the file in Excel :)

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/trick_or_breach/flag.png){: .center-image}

### Other forensics challenges from this CTF

This CTF released a challenge in each of its 5 categories every day.  
I have writeups for all the forensics challenges. Here are some links to them:
- Day 1 - [Wrong Spooky Season](/HTB-HackTheBoo-2022-Forensics1-Wrong-Spooky-Season)
- Day 2 - Trick or Breach (you are here)
- Day 3 - [Halloween Invitation](/HTB-HackTheBoo-2022-Forensics3-Halloween-Invitation)
- Day 4 - [POOF](/HTB-HackTheBoo-2022-Forensics4-POOF)
- Day 5 - [Downgrade](/HTB-HackTheBoo-2022-Forensics5-Downgrade)
