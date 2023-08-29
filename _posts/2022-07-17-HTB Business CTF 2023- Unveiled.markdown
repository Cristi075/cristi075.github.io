---
layout: post
title:  "HTB Business CTF 2023 - Unveiled writeup"
date:   2023-07-17 00:00:00 +0300
categories: HTB_Business_CTF_2023 CTF Cloud AWS S3
summary: Writeup for the Unveiled challenge from HTB's Business CTF from 2023. This challenge involved exploiting a misconfigured s3 service.
---


Unveield was a challenge at the HTB Business CTF 2023 from the 'Cloud' category.  
It involved exploiting a misconfigured S3 service by enumerating buckets and their contents, looking at previous versions and obtaining write access to a bucket and using it to upload a shell to the server.

For this challenge, we got an IP address of a server (VPN was needed for this!) and that server was running a single web app.  
![The webpage]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/web_page.png){: .center-image}

A quick look at the source code reveals that the website tries to load resources from s3.unveiled.htb.  
So I added that domain name to /etc/hosts and pointed it to the IP address of the machine that was spawned for this challenge.  

![Link to S3]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/s3_link.png){: .center-image}

Since this is a 'cloud' challenge, I expect to see cloud services.  
So if I see S3, I'll try enumerating for S3 buckets first

{% highlight bash %}
aws s3 --endpoint-url http://s3.unveiled.htb ls
{% endhighlight %}

![Enumerating S3 buckets]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/enumerating_buckets.png){: .center-image}

Success!  
We have two buckets: backups and website.  
Let's check out the content of each of those two buckets.  

{% highlight bash %}
aws s3 --endpoint-url http://s3.unveiled.htb ls s3://unveiled-backups/
aws s3 --endpoint-url http://s3.unveiled.htb ls s3://website-assets/
{% endhighlight %}

![Enumerating files]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/enumerating_files.png){: .center-image}

It looks like I am unable to access website-assets.  
However, I could find a terraform (.tf) file in the backups bucket.  
Let's try reading it.

{% highlight bash %}
aws s3 --endpoint-url http://s3.unveiled.htb cp s3://unveiled-backups/main.tf .
{% endhighlight %}

![Error at copying files]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/error.png){: .center-image}

And ... I can't read it.  

I thought this is part of the challenge (it wasn't) and I wanted to look at the API response that I was getting.  
So I looked up [how to make aws-cli go through a proxy](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-proxy.html) and used BurpSuite to view the requests. 

To my surprise, the response looked correct, despite aws-cli saying it isn't.

![Correct response]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/correct_response.png){: .center-image}

The most interesting part of this response was the part where the backups bucket was configured.  
When configuring it, "versioning" was turned on.  

So maybe there's a way to see older versions of s3 data.  
After some googling, it looks like [s3api](https://docs.aws.amazon.com/cli/latest/reference/s3api/) is what can be used for this.  

{% highlight bash %}
aws s3api --endpoint-url http://s3.unveiled.htb list-buckets
aws s3api --endpoint-url http://s3.unveiled.htb list-objects --bucket unveiled-backups
{% endhighlight %}

![Using S3Api]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/s3_api.png){: .center-image}

First, I tried listing the buckets and the files contained in the backups bucket again.  
And before anything else, I wanted to see if I can retrieve files using this.


{% highlight bash %}
aws s3api --endpoint-url http://s3.unveiled.htb --bucket unveiled-backups --key main.tf
{% endhighlight %}


![Retrieving a file]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/get_file.png){: .center-image}

Yes, I can. No mure BurpSuite required.

Then, I checked out if there are any older versions of the main.tf file.

{% highlight bash %}
aws s3api --endpoint-url http://s3.unveiled.htb list-object-versions --bucket unveiled-backups --key main.tf
{% endhighlight %}

![Listing versions]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/versions.png){: .center-image}

There's a deleted test.txt file and an older version of main.tf that has a few extra bytes compared to the current one.  
Let's start with the older main.tf file.  

{% highlight bash %}
aws s3api --endpoint-url http://s3.unveiled.htb get-object --bucket unveiled-backups --key main.tf --version-id e27a09ec-15db-484e-808e-b176d6eabcd7 main.tf
cat main.tf
{% endhighlight %}

![Retrieving older version]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/older_version.png){: .center-image}

It works and we get some credentials.  
I tried using those credentials for accessing website-assets and I could do it.  
The website-assets bucket contained the files that were server by the webserver.  

So I uploaded a php reverse shell like [this one](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). (s3 could be used for this, s3api wasn't needed anymore)  
And then I accessed the shell.php file in order to trigger it

![Uploading a shell]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/shell_upload.png){: .center-image}

As expected, the shell ran and I got access to the server.  
The shell runs as the www-data user.

![Getting a reverse shell]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/reverse_shell.png){: .center-image}

While trying to take a look at how the app hosted on the server looks, I noticed that /var/www contains a flag.txt file.  
So I read its contents...

![Getting the flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/unveiled/flag.png){: .center-image}

And we got the flag for this challenge: HTB{th3_r3d_p14n3ts_cl0ud_h4s_f4ll3n}

It makes sense; it was a cloud challenge, not a fullpwn one after all.