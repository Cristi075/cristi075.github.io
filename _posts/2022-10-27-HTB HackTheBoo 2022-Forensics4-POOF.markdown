---
layout: post
title:  "HTB HackTheBoo 2022 - (Forensics) POOF"
date:   2022-10-27 18:00:00 +0300
categories: HackTheBoo_2022 CTF forensics memory_forensics malware_analysis ransomware
summary: Writeup for the POOF challenge (Forensics4/5) from HackTheBoo 2022. This challenge involved performing both network and memory forensics for a host that was infected by ransomware.
---


'poof' was a forensics challenge (day4 out of 5) from HackTheBox's HackTheBoo CTF.  

### What we got

For this challenge, we got the following files:
- a network capture (pcap file)
- a memory dump file (mem.dmp)
- a volatility profile for the Ubuntu version the image was taken on (a zip file)
- an encrypted file

We also got a server that we accessed using telnet that sent questions that you had to answer.  
At the end, you would receive the flag after answering all the questions correctly. 

### Setting up volatility

We got a profile file so we're going to need the original/old [volatility](https://github.com/volatilityfoundation/volatility) and not the newer [volatility3](https://github.com/volatilityfoundation/volatility3).  
You will need python2 in order to run the older version. I recommend using [pyenv](https://github.com/pyenv/pyenv) to install an isolated version of python2.  

### First question. Network analysis
##### Question1: Which is the malicious URL that the ransomware was downloaded from?
We can use Wireshark to open the pcap file and one of the first displayed packets will have the URL that we're looking for.  
In fact, that is the only URL that you can get out of that network capture.  

![Malicious URL]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/wireshark_url.png)
So the answer for this question is: **http://files.pypi-install.com/packages/a5/61/caf3af6d893b5cb8eae9a90a3054f370a92130863450e3299d742c7a65329d94/pygaming-dev-13.37.tar.gz**

### Second question. Memory forensics
##### Question2: What is the name of the malicious process?
For this, we'll have to analyze the memory dump and see a list of processes.  
We'll use the [linux_pstree](https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference#linux_pstree) Volatility plugin to see a list of processes.

{% highlight bash %}
python vol.py -f mem.dmp --profile=Linuxubuntu1804x64 linux_pstree
{% endhighlight %}

![Process tree]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/process.png)

The configure process is among the few ones that are started by the user (uid=1000). It is also the only process that has a bash shell as its parent.  
So the answer to the second question is the name of this process, **configure**.

### Third question. Ransomware file
##### Question3: Provide the md5sum of the ransomware file.

For this, we'll have to obtain that file.  
First, let's use the [linux_bash](https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference#linux_bash) plugin and take a look at the bash history.

{% highlight bash %}
python vol.py -f mem.dmp --profile=Linuxubuntu1804x64 linux_bash
{% endhighlight %}

![Bash history]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/bash_history.png)

So the user downloaded the archive in their home directory, in the Documents folder and then extracted it and ran it.

It would be useful to know the name of the user so that we could have a full path to their homedir.  
For that, we can use the [linux_find_file](https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference#linux_find_file) plugin in order to read the /etc/passwd file. 

First, we have to find the file by path and we'll get an inode. Then, we'll use that inode to get the content of the file.  

{% highlight bash %}
python vol.py -f mem.dmp --profile=Linuxubuntu1804x64 linux_find_file -F "/etc/passwd"
python vol.py -f mem.dmp --profile=Linuxubuntu1804x64 linux_find_file -i 0xffff8a2a34722770 -O passwd
{% endhighlight %}
![Reading /etc/passwd]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/passwd.png)

Now we know that the user is named 'developer13371337' and the homedir can be accessed at /home/developer13371337.  
This means that the location we're looking for is /home/developer13371337/Documents/halloween_python_game/pygaming-dev-13.37/configure  

We try to dump the file from memory and it looks like we can find it.

{% highlight bash %}
python vol.py -f mem.dmp --profile=Linuxubuntu1804x64 linux_find_file -F "/home/developer13371337/Documents/halloween_python_game/pygaming-dev-13.37/configure"
python vol.py -f mem.dmp --profile=Linuxubuntu1804x64 linux_find_file -i 0xffff8a2a3cba00e8 -O configure
{% endhighlight %}
![Obtaining the ransomware file by memdump]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/memdump_configure_file.png)

The resulted file is identified as an ELF. However, we start having some problems with this file.  
First, if we look at the output of 'strings' we see that it is a binary created with pyinstaller.  
![Strings from ransomware sample]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/py_installer_strings.png)
But [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) won't recognize the file and won't extract its contents.  

We also try to compute the md5 hash of this file and it is not the correct answer to the third question.  
We can confirm that we're on the right path by using binwalk and analyzing the outputs: after a while we can find the ransom note that informs the user that their files have been encrypted.  

After a while, I found out the cause of this issue: volatility won't reconstruct an executable file as it was on the filesystem, it will dump it as it exists in-memory.  
This causes some changes in the file and it also causes the hash of the file to be different.  

### Obtaining the ransomware file

The first that I wanted to try was to see if the URL used by the user was still active.  
However, that was turned into a rickroll, so we won't get the original file by accessing that URL.
![URL is inactive]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/rickroll.png)

Another option would be to either find a way to reconstruct the ELF file that we already have or to find another volatility plugin (a community one, not a built-in one) that could do extract and reconstruct the file.    

However, a simple option would be to use the network capture that we have and dump the file from there.  
We can do this by using Wireshark. Just go to File -> Export Objects -> HTTP, select the only file from the pcap and save it.  

By doing this we get a tar archive, we extract its contents in order to get the *configure* file and compute its md5 hash.  
The answer to the third question is the hash of this file, so **c010fb1fdf8315bc442c334886804e00**.

### Analyzing the malware
##### Question4: Which programming language was used to develop the ransomware?

We already determined that this file used pyinstaller because of some of the strings that were present in it.  
So the answer is obviously **python**.

##### Question5: After decompiling the ransomware, what is the name of the function used for encryption?

In order to decompile the malware we first have to extract it by using [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor).  
![Extracted]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/extracted.png){: .center-image}
We get a bunch of files as a result. One of the notable ones is configure.pyc.  
'pyc' files are basically compiled python files.  
We can decompile them and read the code by using [uncompyle6](https://pypi.org/project/uncompyle6/) (for this version of python, for newer versions you should use [decompyle3](https://pypi.org/project/decompyle3/))

After decompiling the file, we can see the code used by the ransomware.

![Decompiled code]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/decompiled_code.png){: .center-image}

The answer to the fifth question is the name of the encryption function. We can recognize that by the call to .encrypt and the AES object being created.  
The answer for this question is **mv18jiVh6TJI9lzY** (the name of that function).


### Sixth question. Decrypt affected files
##### Question6: Decrypt the given file, and provide its md5sum.

Now for the last question, we have to decrypt the other file that we were given and compute its md5 hash.  
First, let's write a decryptor based on the function that we can see. It uses AES and we have both the key and the IV.  
We can just copy that code and replace *encrypt* with *decrypt^.  

The code for the decryptor looks like this
{% highlight python %}
from Crypto.Cipher import AES

data = open('candy_dungeon.pdf.boo', 'rb').read()

key = 'vN0nb7ZshjAWiCzv'
iv = b'ffTC776Wt59Qawe1'
cipher = AES.new(key.encode('utf-8'), AES.MODE_CFB, iv)
ct = cipher.decrypt(data)

open('candy_dungeon.pdf', 'wb').write(ct)
{% endhighlight %}

We run it and we get the MD5 of the resulted file. We can also confirm that the file is identified as a PDF.  

![Decrypting the file]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/decrypting.png){: .center-image}
So the answer to the last question is **3bc9f072f5a7ed4620f57e6aa8d7e1a1**, the hash of the decrypted PDF file.  


### Getting the flag

After we submit the answer to all the 6 questions we receive the flag from the server.  
Here you can see the returned flag and the answers to all the 6 questions.  s

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/poof/flag.png){: .center-image}

### Other forensics challenges from this CTF

This CTF released a challenge in each of its 5 categories every day.  
I have writeups for all the forensics challenges. Here are some links to them:
- Day 1 - [Wrong Spooky Season](/HTB-HackTheBoo-2022-Forensics1-Wrong-Spooky-Season)
- Day 2 - [Trick or Breach](/HTB-HackTheBoo-2022-Forensics2-Trick-of-Breach)
- Day 3 - [Halloween Invitation](/HTB-HackTheBoo-2022-Forensics3-Halloween-Invitation)
- Day 4 - POOF (You are here)
- Day 5 - [Downgrade](/HTB-HackTheBoo-2022-Forensics5-Downgrade)
