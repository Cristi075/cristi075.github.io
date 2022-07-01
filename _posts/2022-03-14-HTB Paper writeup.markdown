---
layout: post
title:  "HTB Paper writeup"
date:   2022-03-14 12:00:00 +0300
categories: HTB_writeup
summary: Writeup for HackTheBox.eu's Paper machine. Notes on obtaining the user and root flags for this machine. 
---

This is my writeup for the Bucket machine from HackTheBox.eu and it contains my notes on how I obtained the root and user flags for this machine.  

![Paper info card]({{site.baseurl}}/assets/img/HTB/paper/info_card.png){: .center-image}
[Paper](https://www.hackthebox.com/home/machines/profile/432) is a Linux machine released on 2022-02-05 and its difficulty level was <b>easy</b>.

### Recon

We begin this with a nmap scan.
{%highlight bash%}
nmap 10.10.11.143
{%endhighlight%}

![port scan result]({{site.baseurl}}/assets/img/HTB/paper/nmap.png){: .center-image}
Note: there was more than one nmap scan being run, but the simplest one (top ports) gave us everything that was needed for this machine.  

Only a http/https server and a ssh service are available.  
Since that's the only path we have right now, we'll take a look at the web server.

### The web server - uninteresting results, mostly

This looks like a default Apache installation.  
Also, enumeration doesn't return any meaningful results.   
We take a look at the server's response to see if we can find any hints there.  

![Interesting header]({{site.baseurl}}/assets/img/HTB/paper/backend_header.png){: .center-image}

And we found it. There's a 'X-Backend-Server' header and it looks like besides paper.htb, there's also a office.paper virtual host.  
We add the new virtual host to /etc/hosts so that we can access it, and check out the contents available there.  

### New virtual host

The application available on the new virtual host (office.paper) is a wordpress installation.  
Our first action in this case should be running wpscan against it.  
While the scan runs, I also gathered a list of usernames based on the displayed names for post authors and for commenters. (This was left unused)  
  
Now, wpscan finished scanning so we take a look at the results.  

![wpscan results]({{site.baseurl}}/assets/img/HTB/paper/wpscan.png){: .center-image}

Wpscan finds a vulnerability that allows us to read unpublished drafts from the wordpress instance.  
Here's a [link](https://0day.work/proof-of-concept-for-wordpress-5-2-3-viewing-unauthenticated-posts/) to a blogpost describing that vulnerability.  
  
This looks like the correct way to proceed with this machine given that a comment warns the poster that they should stop putting secrets in drafts as they aren't secure.  
So, in order to exploit this we access the following URL: http://office.paper/?static=1  

![wordpress exploit]({{site.baseurl}}/assets/img/HTB/paper/wp_exploit.png){: .center-image}

This shows us that there's another virtualhost, probably with a different application.  
We add chat.office.paper to /etc/hosts and try to see what we can find there.

### Exploiting the chatbot
At chat.office.paper we find a rocketchat instance where we can register.  
Once registered and logged in, we notice that there's a chatbot (recyclops) that allows users to send some commands to it.  
The most interesting ones are listing some files and displaying their content. Maybe the time command might also be interesting.  

First, let's try using these two features in the 'intended' way.  
By telling the bot to give us a list of files in the sales directory, it returns what looks like the output of 'ls -la' in that directory.  
![Listing the sales directory]({{site.baseurl}}/assets/img/HTB/paper/ls_example.png){: .center-image}

Now for the other feature, by telling the bot to retrieve a file it looks like we're getting the content of that file. This is probably done by running 'cat'.
![Retrieving a file]({{site.baseurl}}/assets/img/HTB/paper/cat_example.png){: .center-image}

It looks like you need your directory to begin with sales in order for this to work but you can access .. in order to go back.  
This allows us to list the files in any directory from the filesystem.

![Exploiting the listing feature]({{site.baseurl}}/assets/img/HTB/paper/ls_home.png){: .center-image}

Here's an example with the home directory of the current user.  
The command for retrieving files works in a similar manner so we can also read files.  

### Exploring the filesystem

The first thing that I looked for was ssh keys in /home/dwight/.ssh  
However, that folder was empty so no luck getting in like that.  

Another idea would be trying to inject commands. This could make 'ls -la sales' become 'ls -la sales;whoami'  
However, it looks like the application checks for that and disallows it.
![Command injection doesn't work]({{site.baseurl}}/assets/img/HTB/paper/command_injection.png){: .center-image}

Now back to looking for files.  
In the user's home folder we see a bot_restart.sh file. By reading it we can see that the files for this chatbot are in /home/dwight/hubot.  
Let's take a look at what is in that folder.  

![Bot directory]({{site.baseurl}}/assets/img/HTB/paper/bot_directory.png){: .center-image}

The things that instantly grabs my attention here is the .env file. That's a very common way to store credentials or secrets.  
Let's check it out.  

![.env file]({{site.baseurl}}/assets/img/HTB/paper/env_file.png){: .center-image}

Looks like we have the bot's username and password. Now let's try using them for something

### Getting user-level access & the user flag

First, I tried logging in as the bot hoping that it has access to more resources or to some secrets in its chat history.  
However, bots are not allowed to login using the web interface.  
![Web login is not allowed]({{site.baseurl}}/assets/img/HTB/paper/login_not_allowed.png){: .center-image}

I did not miss an open port for this machine so I am also not able to connect to the API.  

However, there's another common attack that we still have to try: *password reuse*.  
Maybe the user has the same password for the bot account and for their account on the server.  

![Logging in by using SSH]({{site.baseurl}}/assets/img/HTB/paper/ssh_login.png){: .center-image}

And they do.  
We now have user-level access and first flag for this machine.  

### Privilege escalation

The privilege escalation is quite simple for this.  
Let's begin by uploading [LinPeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) (using scp) and running it on the machine.  

![LinPeas results]({{site.baseurl}}/assets/img/HTB/paper/linpeas_info.png){: .center-image}

This looks like the first thing that we should try. CVE-2021-3560 affects Polkit and it can be used for privilege escalation.  
We'll now use [this exploitation script](https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation) from GitHub.  
Fun fact: the script was written by the author of this machine

Note: I named the script pwnkit.sh there, but this is not the pwnkit exploit. That's a different CVE (CVE-2021-4034).

![Running the exploit]({{site.baseurl}}/assets/img/HTB/paper/exploit.png){: .center-image}

I run the script with its default parameters (so it'll use the hardcoded username/password) a few times until I manage to login with the newly created user.  
This is to be expected as this is a time-based attack so it will not usually go right from the first try.  
This is also written in the bash script itself, which you should always read before running :).

![Getting the root flag]({{site.baseurl}}/assets/img/HTB/paper/root_flag.png){: .center-image}
And this is how we got the root flag too.