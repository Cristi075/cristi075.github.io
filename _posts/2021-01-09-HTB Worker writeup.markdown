---
layout: post
title:  "HTB Worker writeup"
date:   2021-01-09 18:00:00 +0300
categories: HTB_writeup
summary: Writeup for HackTheBox.eu's Worker machine. Notes on obtaining the user and root flags and also some failures in trying to get a root shell. 
---

This is my writeup for the Bucket machine from HackTheBox.eu and it contains my notes on how I obtained the root and user flags for this machine.  

![Worker info card]({{site.baseurl}}/assets/img/HTB/worker/info_card.png){: .center-image}
[Worker](https://www.hackthebox.eu/home/machines/profile/283) is a Windows machine released on 2020-08-15 and its difficulty level was <b>medium</b>.

### Recon

We begin this with a nmap scan.
{%highlight bash%}
nmap -p- -T4 -A -v 10.10.10.203
{%endhighlight%}

![Port Scan]({{site.baseurl}}/assets/img/HTB/worker/nmap.png){: .center-image}
We can see that this machine is exposing 3 services:  
- an IIS server on port 80
- SVN on port 3690
- another Microsoft-specific service on port 5985

The first look that we take is at the web page on port 80 and we only see a default IIS page. Nothing interesting here.  
Running gobuster with some standard wordlists doesn't find anything useful either.  

**Note:** Based on my experience with other machines, I already added worker.htb to my /etc/hosts file.

### SVN

Next, we try checking out the SVN server. For that, we use the svn client that is already installed (at least on ParrotOS).  
Apache Subversion (AKA svn) is a version control system (similar to git).  

We can use 'svn list' to see what is available on that server and we can use 'svn checkout' to clone the contents locally (I would compare this to how 'git clone' works).
{%highlight bash%}
svn list -v svn://worker.htb/
svn checkout svn://worker.htb/ --quiet
{%endhighlight%}

We find a folder named 'dimension.worker.htb' and a file named 'moved.txt'

![Inspecting SVN repo]({{site.baseurl}}/assets/img/HTB/worker/svn_01.png){: .center-image}

By looking at the folder's name and by reading the file, we learn about the existence of two more domains:
- dimension.worker.htb
- devops.worker.htb 

We add them to /etc/hosts and try to access them to see their content.  
devops.worker.htb leads to a page that asks for authentication.  

A common mistake on git is to commit credentials or other sensitive information to the repository.  
I could not find any traces of that in the files that I have obtained from that repository.  
However, it is common (on git at least) to cover this up by removing it from the repository. But because version control systems keep track of your changes, that is still available in your history (at least usually).  
SVN allows you to see the history of the repo by using the -r <nr> parameter (-r coming from 'revision'). It appears that we are on revision nr. 5 right now so let's try to see how the older revisions looked.  

{%highlight bash%}
svn checkout svn://worker.htb/ --quiet -r 4
{%endhighlight%}
Revision 4 does not have anything interesting. The moved.txt file is not present yet.  

{%highlight bash%}
svn checkout svn://worker.htb/ --quiet -r 3
{%endhighlight%}
Revision 3 has a new file: a powershell script named deploy.ps1.  
It appears to contain a username (**nathen**) and a comment indicating that the password should not be put here.  
This might mean that going a bit further might also reveal the password. 

![Revision3]({{site.baseurl}}/assets/img/HTB/worker/svn_rev3.png){: .center-image}

{%highlight bash%}
svn checkout svn://worker.htb/ --quiet -r 2
{%endhighlight%}
Revision 2 has the same powershell script. And as expected, the user's password is available there in cleartext.

![Revision2]({{site.baseurl}}/assets/img/HTB/worker/svn_rev2.png){: .center-image}

Now we can try to authenticate to devops.worker.htb with the newly found credentials and we'll obtain access.  
This seems to be an instance of Azure Devops Server.

### Azure Devops Server

![Devops server]({{site.baseurl}}/assets/img/HTB/worker/devops_01.png){: .center-image}
After we login with the user that we found (nathen) we see that we have access to a single project: SmartHotel360.  
The first thing that I checked was the pipelines tab. There were multiple pipelines setup but I could not create new ones nor modify existing ones.  
I was hoping to add my own code to one of these.  

![Access denied - pipelines]({{site.baseurl}}/assets/img/HTB/worker/pipelines_denied.png){: .center-image}

Inside the project, there are multiple repositories. Judging by their names and the pipelines that I've seen, every repository is used to build a specific subdomain (named the same as the repository).  
An example would be the 'spectral' repository and spectral.worker.htb. I decided to use this one for the next steps (picked it at random, they were quite similar).  

![Spectrum subdomain]({{site.baseurl}}/assets/img/HTB/worker/spectral.png){: .center-image}

**Note:** I had to add spectral.worker.htb to /etc/hosts like I did with the other domains.

### Exploiting Azure Devops Server

We are allowed to add new files to the repositories that are used to build the webapps available on the server (like the 'spectral' one from above).  
This can be done by using the web UI. However, I ran into a problem: commits cannot be made to the main branch.
![Failed commit]({{site.baseurl}}/assets/img/HTB/worker/commit_01.png){: .center-image}

So we must create a new branch, make the change, commit it, submit a pull request, approve it and then wait for the application to be rebuilt.  

The applications run on .aspx so I used [this reverse shell](https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx) from GitHub.
![Reverse shell setup]({{site.baseurl}}/assets/img/HTB/worker/shell_01.png){: .center-image}

First, create a new branch (I called it 'test123'). Then, add a new file and put the content in it (paste the code for the shell and modify the IP/port in this case).  
Then, commit your changes. 
After that, create a new pull request.

![New pull request]({{site.baseurl}}/assets/img/HTB/worker/shell_02.png){: .center-image}
You can approve your own pull request.  
However, you will notice that you need a 'work item' to be assigned to the pull request in order to be able to approve it.  
Create a new work item and assign it to this pull request. You will have to go into another menu from Azure DevOps and add a bug or a similar task there.  

![Approved pull request]({{site.baseurl}}/assets/img/HTB/worker/shell_03.png){: .center-image}
And you are done, now we wait for the application to be rebuilt.

### Getting a reverse shell

After waiting for a short period of time, we can access the reverse shell that we introduced there.  
Start a listener and go to the URL of your .aspx file.  
In my case it was http://spectral.worker.htb/testdev.aspx  

![Obtained reverse shell]({{site.baseurl}}/assets/img/HTB/worker/shell_obtained.png){: .center-image}

Now we obtained a limited-user shell.

### Enumeration

Now that we have access to the system, we start enumerating it. It displays a large (nr. user offerings)  
When we tried listing the contents of C:\Users only one account that looks like it belongs to an user came back. That is probably the user that we are looking for: **robisl**.  

![Users]({{site.baseurl}}/assets/img/HTB/worker/users.png){: .center-image}

In order to get more information I downloaded [WinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) on my machine and then moved it to the target using curl & a webserver.  

![Downloading WinPEAS]({{site.baseurl}}/assets/img/HTB/worker/download_winpeas.png){: .center-image}

Probably the most useful information for now is that there is another drive available, **W:** 

![Available drives]({{site.baseurl}}/assets/img/HTB/worker/drives.png){: .center-image}

There are multiple folders on W:. While exploring them, we found a list of credentials in W:\svnrepos\www\conf\passwd

![Passwd file]({{site.baseurl}}/assets/img/HTB/worker/passwd_file.png){: .center-image}

The credentials that we already know (for nathen) are already there and are correct. We notice that there's also an entry for **robisl**, the user that we are looking for.  
Now we only need a way to use these credentials. This being a Windows machine, it does not come with the su command like the Linux ones.


### Privilege escalation & the user flag

### More enumeration

### Back to Azure Devops

### Pipelines!

### A small PoC

### Many failed attempts to obtain a shell

### More failed attempt to add a new user (and use it)

### Giving up and getting the root flag