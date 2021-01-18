---
layout: post
title:  "HTB Bucket writeup"
date:   2021-01-09 18:00:00 +0300
categories: HTB_writeup
summary: Writeup for HackTheBox.eu's Bucket machine. Notes on obtaining the user and root flags. 
---

This is my writeup for the Bucket machine from HackTheBox.eu and it contains my notes on how I obtained the root and user flags for this machine.  

![Bucket info card]({{site.baseurl}}/assets/img/HTB/bucket/info_card.png){: .center-image}
[Bucket](https://www.hackthebox.eu/home/machines/profile/283) is a Linux machine released on 2020-10-17 and its difficulty level was <b>medium</b>.

### Recon
We begin this by running a port scan with nmap.
{%highlight bash%}
nmap -p- -T4 -A -v 10.10.10.194
# This is also the command used by using "Intense scan, all TCP ports" in zenmap
{%endhighlight%}
![Port Scan]({{site.baseurl}}/assets/img/HTB/bucket/port_scan.png){: .center-image}
We can see that this machine is exposing 2 services:  
- ssh on port 22 
- an Apache web server on port 80.

### Web server (port 80)
The only next step available at this point is going to the web server on port 80.  
The html page looks broken at first but after looking at the source code (html) we can see some references to *bucket.htb* and *s3.bucket.htb*.
![found URLs]({{site.baseurl}}/assets/img/HTB/bucket/bucket_links.png){: .center-image}

So we should add these to the /etc/hosts file and try again.  
Also, it will be easier to use the domain name instead of the IP address now that we made this change.  
This is how the page looks like now.  
![Web homepage]({{site.baseurl}}/assets/img/HTB/bucket/web_page.png){: .center-image}

By accessing s3.bucket.htb we find what looks like an on-prem version of Amazon's S3 (S3 stands for Simple Storage Service)  
As part of the standard steps for this kind of target, we should start doing some enumeration with a tool like dirbuster (or gobuster, my choice in this case).  
![Enumeration result]({{site.baseurl}}/assets/img/HTB/bucket/gobuster_result.png){: .center-image}

We see some interesting results there: /health and /shell.  
First, we access /health and we see that there are two services running on the server: S3 and DynamoDB.  
And /shell looked like a web shell for DynamoDB. I ended up not using that for anything.  

![Enumeration result]({{site.baseurl}}/assets/img/HTB/bucket/health_page.png){: .center-image}

Those services are probably going to lead to the next piece of this puzzle.  
I know for sure that unsecured S3 buckets were a security problem (and still are).  
That being said, [this article](https://medium.com/@narenndhrareddy/misconfigured-aws-s3-bucket-enumeration-7a01d2f8611b) has some basic information on common S3 issues. 

### S3 and DynamoDB

The best way to explore both of these services (S3 and DynamoDB) is to use *awscli*. (sudo apt install awscli, no Debian-based distros)  

First, let's take a look at DynamoDB (since that was the last one that we discovered).  
We can get a list of tables by running list-tables. The full command would be 
{%highlight bash %}
aws dynamodb list-tables --endpoint-url http://s3.bucket.htb
{%endhighlight%}

We have to specify that we want to connect to a local instance and not to the one hosted by Amazon.  
You might also have to set up some API keys by running 'aws configure'.  
You can just put some dummy data in there since we're not accessing Amazon's services. You might also want to set the default output format to 'text'.  

We see that there is a single table named 'users' in the database. We'll use the 'scan' command in order to dump its contents.
{%highlight bash %}
aws dynamodb scan --table-name=users --endpoint-url http://s3.bucket.htb
{%endhighlight%}

The result looks like this. We got three username/password pairs.  
I tried using all their combinations for authentication using ssh but with no success.

![Users]({{site.baseurl}}/assets/img/HTB/bucket/users.png){: .center-image}

Next, we'll try to see what we can find by exploring the S3 service.  
{%highlight bash %}
aws  s3 ls --endpoint-url http://s3.bucket.htb
{%endhighlight%}
First, we enumerate the buckets that are available.  
We notice two buckets: foobar and adserver.  

![Bucket list]({{site.baseurl}}/assets/img/HTB/bucket/buckets.png){: .center-image}

Foobar doesn't seem to have anything interesting but adserver looks like it might be useful.

### Exploring S3 bucket

We should check out the contents of the 'adserver' bucket now.

{%highlight bash %}
aws --endpoint-url http://s3.bucket.htb s3 ls adserver
aws --endpoint-url http://s3.bucket.htb s3 ls adserver/images
{%endhighlight%}

If we download these files we notice that they are the files that can be seen on the web server that we've seen at first (on port 80).

![Files in S3 bucket]({{site.baseurl}}/assets/img/HTB/bucket/list_files.png){: .center-image}

### Getting a reverse shell

![Uploading and executing shell]({{site.baseurl}}/assets/img/HTB/bucket/shell_exec.png){: .center-image}

![Obtaining the reverse shell]({{site.baseurl}}/assets/img/HTB/bucket/shell1.png){: .center-image}

### Obtaining user-level access (privilege escalation)

![Privilege escalation]({{site.baseurl}}/assets/img/HTB/bucket/escalation1.png){: .center-image}

### The user flag

![User flag]({{site.baseurl}}/assets/img/HTB/bucket/user_flag.png){: .center-image}

### Trying to get root access

### Obtaining the root flag