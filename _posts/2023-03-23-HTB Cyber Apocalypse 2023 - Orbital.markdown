---
layout: post
title:  "HTB Cyber Apocalypse 2023 - (Web) Orbital"
date:   2023-03-23 15:00:06 +0300
categories: HTB_Cyber_Apocalypse_2023 CTF web
summary: Writeup for the Orbital (Web, Easy) from HTB Cyber Apocalypse 2023. This very simple challenge involved SQL Injection and a path traversal attack.  
---


'Orbital' was one of the challenges in the 'Web' category at HTB's Cyber Apocalypse 2023.  
Its difficulty was 'Easy' and it involved exploiting SQL Injection and a path traversal vulnerability.  

### Challenge description

For this challenge, we got access to a Docker container running a web app and the source files of that webapp.  

This is how the web app looks like:
![Login webpage]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/main_page.png){: .center-image}

And these are the files that we have:
![List of challenge files]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/challenge_files.png){: .center-image}

### Code analysis

First, let's take a look at the code for this app.  
The app is built using python & Flask.  
The first interesting thing that I seen was in the database.py file.  

### Identifying the first vulnerability SQL Injection

{% highlight python %}
def login(username, password):
    # I don't think it's not possible to bypass login because I'm verifying the password later.
    user = query(f'SELECT username, password FROM users WHERE username = "{username}"', one=True)

    if user:
        passwordCheck = passwordVerify(user['password'], password)

        if passwordCheck:
            token = createJWT(user['username'])
            return token
    else:
        return False
{% endhighlight %}

The query appends the username without any kind of filtering or sanitization; this makes the query vulnerable to SQL Injection.  

The first result of the query is taken and the user-supplied password is checked against the retrieved one (MD5 hashes match).  
This will make exploiting this a bit trickier but it shouldn't be that hard.

### Logging extra information

First, let's make the application print some extra information so that we can build our malicious query easier.  
We'll  change the login function like this:

{% highlight python %}
def login(username, password):
    # I don't think it's not possible to bypass login because I'm verifying the password later.
    print('----')
    print('Query:')
    print(f'SELECT username, password FROM users WHERE username = "{username}"')
    user = query(f'SELECT username, password FROM users WHERE username = "{username}"', one=True)
    print('Result:')
    print(user)
    print('----')

    if user:
        passwordCheck = passwordVerify(user['password'], password)

        if passwordCheck:
            token = createJWT(user['username'])
            return token
    else:
        return False
{% endhighlight %}

In order to make the output of those print statements visible, we also have to modify supervisord.conf

For that, we change this line
{% highlight python %}
command=python /app/run.py
{% endhighlight %}

Into this 
{% highlight python %}
command=python -u /app/run.py
{% endhighlight %}

This is a quick solution found [here](https://stackoverflow.com/questions/29663459/python-app-does-not-print-anything-when-running-detached-in-docker)

Next, I build and run the container by running the ./build-docker.sh script.  

And try logging in with test/test

![Application logs (with extra information)]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/logs.png){: .center-image}

### The SQL Injection attack

Now let's build our SQL Injection attack.  

First, let's assume we don't know the account so we append "or 1=1" to the query.  
**Note:** Since we see how the container is built, we know that the account is admin. This is probably the same in the live version of the container.  
However, ignoring that information will lead to building a more robust attack vector.  

First, we'll use this simple query

{% highlight sql %}
test" OR 1=1#--
{% endhighlight %}

![Injection testing]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/injection_test.png){: .center-image}

Then, we want to control the first value that's returned.  
For that, we add an UNION to the query and append our own data.  
The real data from the database will still come first, but we can add 'ORDER BY username DESC' to sort the returned data and make our injected result be the first one.

{% highlight sql %}
test" OR 1=1 UNION SELECT "test","test" ORDER BY username DESC#--
{% endhighlight %}

![Injection 1]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/injection_1.png){: .center-image}

Success!  

Now we'll need to replace the second injected "test" with the MD5 value of "test" (since that's what I used as my password here).  

{% highlight sql %}
test" OR 1=1 UNION SELECT "test","098f6bcd4621d373cade4e832627b4f6" ORDER BY username DESC#--
{% endhighlight %}
![Injection 2]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/injection_2.png){: .center-image}

And we're in.

![Webpage after login]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/webpage_logged_in.png){: .center-image}

The flag isn't right on this page, so we have to keep looking.

### The path traversal attack

In the routes.py file we see the following code

{% highlight python %}
@api.route('/export', methods=['POST'])
@isAuthenticated
def exportFile():
    if not request.is_json:
        return response('Invalid JSON!'), 400
    
    data = request.get_json()
    communicationName = data.get('name', '')

    try:
        # Everyone is saying I should escape specific characters in the filename. I don't know why.
        return send_file(f'/communications/{communicationName}', as_attachment=True)
    except:
        return response('Unable to retrieve the communication'), 400
{% endhighlight %}

communicationName is appended to the path without any kind of sanitization.  
This makes that function vulnerable to path traversal.  

Let's try making a crafted request using BurpSuite to read the content of /etc/passwd.  
According to the Dockerfile, the working directory is /app. So the path to /etc/passwd should be ../etc/passwd (Full path:/app/../etc/passwd)
![Reading /etc/passwd]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/etc_passwd.png){: .center-image}

It works without any issue.

### Getting the flag

Now let's use the same technique to get the flag.

First, we have to find out where we can find the flag in the container.  
In the dockerfile, we can see the following lines.  

{% highlight Docker %}
# copy flag
COPY flag.txt /signal_sleuth_firmware
COPY files /communications/
{% endhighlight %}

So the flag is copied at /signal_sleuth_firmware

![Fake flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/fake_flag.png){: .center-image}

By requesting '../signal_sleuth_firmware' we got the flag (the fake one, for now)

Now, let's use the the same two attacks on the live app.  

![Real flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/web_orbital/real_flag.png){: .center-image}

The two attacks work flawlessly for the live app too.  
And we get the flag: **HTB{T1m3\_b4$3d\_$ql1\_4r3\_fun!!!}**

Wait ... time based? 
