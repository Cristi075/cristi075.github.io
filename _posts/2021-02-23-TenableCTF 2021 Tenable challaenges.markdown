---
layout: post
title:  "TenableCTF 2021 Tenable challenges writeup"
date:   2021-02-23 20:00:00 +0300
categories: CTF_writeup TenableCTF_2021
summary: A writeup for the Tenable challenges that I solved for TenableCTF 2021 and some of my thoughts on the one that I did not solve.
---
This is my writeup for the challenges from the Tenable category that were part of [TenableCTF 2021](https://ctftime.org/event/1266).  

This being a CTF organized by Tenable, they had a category where you used their own tool (Nessus Essentials in this case) in order to find flags.  
They offered a NessusDB file (a fancy sqlite, basically) and you had to import it in your instance of Nessus Essentials.  

The flags that I obtained were:
- mutant
- dead
- knowledge
- command

I got them in a different order (mutant->command->knowledge->dead) but I will present them in this order.  

Also, I could not get the last flag (supercession).  
I will include my thoughts on that one at the end of this writeup. At the moment of writing this I still don't know where that one was hidden.  

### Setting up Tenable
You can get a license/activation key of Nessus Essentials that is able to scan up to 16 IP addresses for free on [their website](https://www.tenable.com/products/nessus/nessus-essentials).  
This is what they recommended for these challenges in their discord channel.  

In order to get it up, first you have to get an activation key with the method mentioned above (it requires an email & name).  
Then, you download an installer from [here](https://www.tenable.com/downloads/nessus?loginAttempted=true). You should get the correct installer for your platform and run it.  
You are probably looking to get a .msi file for Windows, a .dmg for MacOS or a .deb, .rpm or a similar format for Linux.  
I got the .deb file and installed it on a Ubuntu Server VM using dpkg (sudo dpkg -i ./Nessus-8.13.1-debian6_amd64.deb).

After the installation is done, you should go to the web UI to complete the setup and create an account on that Nessus instance.  
When you're done. You'll see something like this.  

### The NessusDB
The file received was Linux_Scan.db.  
It looks like a regular SQLite file but it cannot be opened with a sqlite explorer even if we know the password.  
Instead, we should import the file into our instance of Nessus Essentials that we installed before.  


### The 'mutant' flag

After the import was done, I chose to export a .nessus file. That is an XML file so at least I'm dealing with a text file.  
The first command that I ran was.


<p>
{% highlight bash %}
cat aaa | grep flag
{% highlight %}
</p>

And we already found the first flag (mutant). This was easy.

### The 'dead' flag

### The 'knowledge' flag

### The 'command' flag