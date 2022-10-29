---
layout: post
title:  "HTB HackTheBoo 2022 - (Web) Spookifier writeup"
date:   2022-10-27 16:00:00 +0300
categories: HackTheBoo_2022 CTF template_injection
summary: Writeup for the Spookifier (Web2/5) from HackTheBoo 2022. This challenge involved exploiting a template injection vulnerability in a Flask application that used Mako as its templating engine.
---


'Spookifier' was a web challenge (day 2 out of 5) from HackTheBox's HackTheBoo CTF.  
Getting the flag involved exploiting a template injection vulnerability in a Flask app that used Mako as its templating engine.

### What we got

Like in the case of the previous challenge, we got access to a Docker container running a web app and the source code of that web app (without the flag, obviously).

### Running the webapp locally 

First, let's try to run the app locally.  
We start by looking at the Dockerfile to understand the requirements of this app.  
We need to have Flask 2.0.0, Werkzeug 2.0.0, mako and flask_mako installed.  
After installing them in a virtual environment, we can start the app.

The app takes a string as its input and returns 4 variations of that string.  
The variations are created by substituting the original characters with symbols that look like them.  
Also, the fourth output looks identical to the input.

![The web application]({{site.baseurl}}/assets/img/HackTheBoo_2022/spookifier/webapp.png){: .center-image}

The requests don't seem that interesting this time so let's look at the code.

### Analyzing the code. Template injection

All the relevant logic for this app is contained in the 'util.py' file.  

![Code]({{site.baseurl}}/assets/img/HackTheBoo_2022/spookifier/code.png){: .center-image}

We can see that the input gets converted based on 4 dictionaries that are defined in this file.  
Also, we can confirm that font4 (associated with the last output) just maps each character to itself.  
This happens in the *change_font* function.  

Then, the *generate_render* function is used to create a Mako template containing the four outputs and then that template gets rendered.  
We can see that the user input (including the fourth one, which is not changed in any way) is put directly into the template **before** the template is rendered.  
This means that we can influence what things get rendered by modifying the template.  

We can confirm this by using a simple payload like ${1+1} and see that we get "2" as the output.

### Crafting the payload. Getting the flag

In order to get the flag, we'll use a simple payload: 

{% highlight python %}
${open('../flag.txt','r').read()}
{% endhighlight %}

We used this payload on the app running in the docker container spawned for this challenge and we got the flag.

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/spookifier/flag.png){: .center-image}
