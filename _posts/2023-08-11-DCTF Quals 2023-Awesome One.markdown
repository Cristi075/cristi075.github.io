---
layout: post
title:  "DCTF Quals 2023 - Awesome One"
date:   2023-10-22 13:00:00 +0300
categories: DCTF_2023 CTF reverse
summary: Writeup for the AwesomeOne challenge from the D-CTF Qualifiers 2023. This was very relatively simple reverse engineering.
---


[AwesomeOne](https://dctf23-quals.cyber-edu.co/challenge/9a67b863-3837-4c23-9331-261693b61c4c) was a challenge at the D-CTF Qualifiers from 2023.  
This was a very simple reverse engineering challenge which I used as a reason to install Ghidra again :)

### The challenge

For this challenge, we got a binary that was asking for a password and then returned some messages & some numbers.
![Program outputs]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_output.png){: .center-image}

If I ran strings on it, I could see all their funny messages.  
It looks like the program just picks one at random.

![Strings]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_strings.png){: .center-image}

### Some reverse engineering

I opened the binary in Ghidra and looked at the output of the decompiler.  
The program asks the user to input a password (the flag), and then checks if it is the correct one by using the check_password function.

![The main function]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_ghidra_main.png){: .center-image}

By taking a look at the check_password function, I could see that the flag is stored in a variable: enc_flag.  
The flag is not plaintext: it is probably encoded or encrypted somehow.  

And I could also see some kind of XOR operation there. Some quick analysis of that operation:  
The program was XOR-ing between the flag length, the user input length and the current character from the user input.  
Then, it makes a bitwise OR between the values obtained like that for each character.  

The two lengths should be equal when you input the flag, so that would go away (X XOR X = 0).  
After that, we have a bitwise OR between all the characters in the user input.  
This probably leads to all the bits in the result being set to 1 in the end (for a large input).  

So this is probably not an useful operation. Let's look at the the enc_flag variable.  

![check_password function]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_ghidra_check.png){: .center-image}

That contains an address (remember that in C string variables are just pointers to the beginning of a char array!!!).  
By looking at that address, I could notice an array of characters. Those don't look printable so there is some encoding going on here.  

![encoded flag]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_ghidra_enc_flag.png){: .center-image}

I copied the value of those bytes by selecting them, right clicking and then using 'Copy Special' (if you are not familiar with Ghidra :) )

![copying the flag]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_ghidra_copy_special.png){: .center-image}

And chose to copy the byte string. 

I got something that looks like this.  
I can exclude the last character (00) since that's just the null terminator for this string.


{% highlight shell %}
06 11 03 3e 23 26 76 24 71 74 24 70 72 72 23 23 74 75 72 7d 73 24 77 23 21 27 23 26 24 21 74 7d 20 23 71 72 20 24 72 7d 21 71 77 73 24 71 72 21 75 7c 72 24 71 7c 20 76 7d 75 76 23 72 20 7c 26 75 20 7c 73 38 00
{% endhighlight %}

I wanted to decode the string so I tried some stuff in CyberChef.  
As expected, just using 'From Hex' doesn't result in anything useful or readable.

However, if we try using the 'XOR Brute Force' operation.  
I just set the crib to 'CTF' since I knew the flag format and for key=45 I got a valid flag.  

Why XOR?  
The 'result' value obtained obtained in check_password seemed to be garbage to me.  
However, I was thinking that if the first OR was replaced with another XOR, the value might've resembled some kind of hash (a XOR between all the characters of the input).  
So XOR was among the very first things that I thought of for transforming this string into something usable.

![Obtaining the flag]({{site.baseurl}}/assets/img/DCTF_Quals_2023/dctf_awesomeOne_cyberchef.png){: .center-image}

Flag obtained: **CTF{fc3a41a577ff10786a2fdbfcad18ef47ea78d426a47d097a49e3803f7e9c0e96}**