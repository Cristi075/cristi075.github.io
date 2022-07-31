---
layout: post
title:  "TenableCTF 2022 OSINT Challenges Writeup"
date:   2022-06-13 20:00:00 +0300
categories: CTF_writeup TenableCTF_2022 OSINT
summary: Writeups for 3 out of 4 OSINT challenges from TenableCTF 2022
---



### Find me if you can

The hint for this challenge was made of these two images (the first one was a originally gif):
![Hint for 'Find me if you can']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_1_hint.png){: .center-image}

The second one is quite a clear reference to the recent "4shell" vulnerabilities (mainly Log4Shell & Spring4Shell).  
So based on the first image, it's more likely to be about Spring4Shell because the image contains ... springs.  
The "spring" part took me way too long to figure (It was actually pointed out by someone else from our team) as I was trying to come up with solutions that involved social media (commonly seen in OSINT challenges).  

However, once you made the connection with Spring4Shell you can try to do a Google search for "Spring4Shell Tenable" and among the first results you will probably find Tenable's blogpost about the exploit.  
This is the article:  
[https://www.tenable.com/blog/spring4shell-faq-spring-framework-remote-code-execution-vulnerability](https://www.tenable.com/blog/spring4shell-faq-spring-framework-remote-code-execution-vulnerability)

And if you look at the source code for that page you will spot that the flag is hidden in there.
![Flag for 'Find me if you can']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_1_flag.png){: .center-image}


### Can you dig it?

The hint for the second challenge was this image:
![Hint for 'Can you dig it?']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_2_hint.png){: .center-image}

The main hint is probably something to do with "zero clicks" (as in, a zero-click exploit).  
Other hints/keywords that might be derived from this image might be: three and skulls. Not sure if there's any recent exploit that relates to those.  

So first, we try using the same method as the last challenge and we search for "zero click" on Tenable's website.  
We go through the results and search for "flag{" in the page's source code.  
After a few tries we get to the following article that has the flag hidden in it:  
[https://www.tenable.com/blog/cve-2022-30190-zero-click-zero-day-in-msdt-exploited-in-the-wild](https://www.tenable.com/blog/cve-2022-30190-zero-click-zero-day-in-msdt-exploited-in-the-wild)

![Flag for 'Can you dig it?']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_2_flag.png){: .center-image}


### Lets network

The hint for the third challenge was made of these two images:
![Hint for 'Lets network']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_3_hint1.gif){: .center-image}
![Hint for 'Lets network']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_3_hint2.gif){: .center-image}

So the main hint is Cisco RV. That's a series of routers made by Cisco.  
Keeping the trend from the previous challenges, we search for "Cisco RV" in their vulnerability database.  
Among the first results we can find this page:  
[https://www.tenable.com/blog/cve-2022-20699-cve-2022-20700-cve-2022-20708-critical-flaws-in-cisco-small-business-rv-series](https://www.tenable.com/blog/cve-2022-20699-cve-2022-20700-cve-2022-20708-critical-flaws-in-cisco-small-business-rv-series)
Which had the flag embedded in it.

![Flag for 'Lets network']({{site.baseurl}}/assets/img/TenableCTF_2022/osint/osint_3_flag.png){: .center-image}

### Another way of solving this

Those three flags weren't hard to find after we made the connection with Tenable's vulnerability database.  
Howver, there is a different method of obtaining these flags.  
Tenable's search feature looks at the code that isn't rendered too.  

So one could search for "flag{" and get the three pages that contained flags as results:
[https://www.tenable.com/blog/search?field_blog_section_tid=All&combine=flag%7B](https://www.tenable.com/blog/search?field_blog_section_tid=All&combine=flag%7B)  
This might not work anymore when you are reading it; the flags could be removed. However, it looked something like this.

![Search result]({{site.baseurl}}/assets/img/TenableCTF_2022/osint/flag_search.png){: .center-image}

We only came up with this idea after the CTF was over but this was an interesting and probably unintended way of solving this challenge.
