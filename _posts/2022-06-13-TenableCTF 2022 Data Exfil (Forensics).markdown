---
layout: post
title:  "TenableCTF 2022 Data Exfil (Forensics) Writeup"
date:   2022-06-13 20:00:00 +0300
categories: CTF_writeup TenableCTF_2022 Forensics ICMP_Tunneling
summary: Writeups for the "Data Exfil" challenge from TenableCTF 2022
---

"Data Exfil" was a forensics challenge from TenableCTF 2022.  
For this challenge, we got a pcap file containing a network traffic capture.  

We open that file in wireshark and the main thing we see is a lot of ICMP traffic.  
For now, it looks like this might be ICMP tunneling.  

![Viewing in wireshark]({{site.baseurl}}/assets/img/TenableCTF_2022/forensics/exfil1.png){: .center-image}

We notice that the first ICMP packets have a length of 100 but some later ones have a length of 1044 (the reply, the request has a length of 44).  
It might be worth taking a look at the ICMP stream that contains those 1044 bytes long packets.  

![ICMP Stream]({{site.baseurl}}/assets/img/TenableCTF_2022/forensics/exfil2.png){: .center-image}

These packets seem to contain some raw data. By decoding some of that raw data using CyberChef (using "From Hex") we can also see what looks like a PNG signature.  

So first, we will isolate those packets. For that, I'll use tshark and run this simple command (it extracts all the ICMP packets that have a length of 1044)

{% highlight bash %}
tshark -r dataexfil.pcap -x -T json -n "icmp && frame.len == 1044" >> icmp_data.json
{% endhighlight %}

We got those packets in a JSON file so right now we can now parse them with a python script.  
After a while, we get this script that gets all that data and outputs it into a file.  

{% highlight python %}
import json

infile = open('icmp_data.json', 'r')
packets = json.load(infile)

reconstructed = []

for packet in packets:
    data = packet['_source']['layers']['icmp']['data_raw']
    reconstructed.append(data[0])
print(''.join(reconstructed))

infile.close()
{% endhighlight %}

The output is hex-encoded so we'll decode it using xxd this time (I'm too lazy to use CyberChef just for this).  
By checking the resulted file with 'file', we can see that this is recognized as a PNG file.  

![Reconstructing the data]({{site.baseurl}}/assets/img/TenableCTF_2022/forensics/exfil3.png){: .center-image}

So we try to open it with image viewing/editing software (GIMP in this case).  
The image contained the flag.  

![Viewing the flag]({{site.baseurl}}/assets/img/TenableCTF_2022/forensics/flag.png){: .center-image}

& that's how this challenge ends.  