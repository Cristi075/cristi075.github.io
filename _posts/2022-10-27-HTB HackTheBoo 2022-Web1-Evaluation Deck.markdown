---
layout: post
title:  "HTB HackTheBoo 2022 - (Web) Evaluation Deck writeup"
date:   2022-10-27 16:00:00 +0300
categories: HackTheBoo_2022 CTF command_injection
summary: Writeup for the Evaluation Deck challenge (Web1/5) from HackTheBoo 2022. This challenge involved exploiting a command injection vulnerability in a Flask application.
---


'Evaluation Deck' was a web challenge (day 1 out of 5) from HackTheBox's HackTheBoo CTF.  
Getting the flag involved exploiting a simple command injection vulnerability in a Flask app.

### What we got

For this challenge, we got access to a Docker container running a web app and the source code of that web app (without the flag, obviously).

### Running the webapp locally 

First, let's try to run the app locally.  
According to the Dockerfile, the only requirement for running this is having Flask installed.  
After we install Flask (in a virtual environment), we can run the app (python3 run.py).  

The app is a small game in which you flip cards that have various effects.  
From what I see, they can either increase or decrease the HP of your 'opponent' (the ghost that you see on-screen)  

![The web application]({{site.baseurl}}/assets/img/HackTheBoo_2022/evaluation_deck/webapp.png){: .center-image}

Now, let's take a look at the request that is made when flipping a card (using BurpSuite).

![Request example]({{site.baseurl}}/assets/img/HackTheBoo_2022/evaluation_deck/request.png){: .center-image}

It looks like the opponent's HP value, the attack power of the current card and the 'operator' (+ or -) gets sent to the server.  
You can change those values and make win the game easily like this. The server will trust any input from the user (so the cards that you pick are irrelevant for the game).  

But winning the game is not the point here. So let's take a look at the code to see how we can get the flag.

### Analyzing the code

The /api/get_health route is the only one implemented in this app (besides /, which returns index.html).  
The 'count' method from 'blueprints/route.py' is the method associated with /api/get_health.  
Let's take a closer look at how that looks like.
![Code for /api/get_health]({{site.baseurl}}/assets/img/HackTheBoo_2022/evaluation_deck/count_method.png){: .center-image}

The count() method gets the 3 parameters that we've seen above, it verifies that all 3 parameters exist and then it computes the result.  
In order to compute the result, the *compile* and *exec* functions are used.  
You can read more about those functions in the official python documentation ([compile](https://docs.python.org/3/library/functions.html#compile) and [exec](https://docs.python.org/3/library/functions.html#exec)) but the general idea is that they are used to run code that is defined at runtime.  

The call to *compile* is where user input is involved, so let's look at that.  
The *current_health* and *attack_power* variables are converted to integers so these will be harder to attack.  
However, operator is not sanitized or transformed in any way. We are free to add whatever we want there.  

### Code injection. Developing the payload

We found a way to modify the string that gets processed by *compile* so now let's craft the payload.  

Since we're running the app locally, we can make use of the following trick to make payload development easier: add some print statements so you can see how the variables that you are trying to control look like.  
This is how the method looks like after I added 2 print statements: I'm printing the variable that I can control (operator) and the same string that is used as an input for *compile*.

![Modified code]({{site.baseurl}}/assets/img/HackTheBoo_2022/evaluation_deck/modified_code.png){: .center-image}

We want a payload that retrieves the contents of the flag.txt file (it's in one folder above the app.py file so that would probably be ../flag.txt).  
We'll use the following things in our payload:
- The variable named 'result' is the one that gets returned to the user. So if we assign something to 'result', the server will return that value to us
- We can chain multiple python statements by putting semicolons between them.  
Example: print(1); print(2); print(3)

Also, in order to have valid code, we need to take care of the following two things:
- We have something that gets assigned to 'result' before we get to our injection point. We'll have to make sure that we have a valid statement there  
This can be done by having the 'operator' field start with a semicolon (;) or by having a valid operation before the semicolon (ex: -0;)
- We have the second integer that comes after our injection point. We have to introduce a valid statement there too.  
For this, I added another statement that does not use the 'result' variable in order to avoid overwriting its value

Now for the main part, I opened the '../flag.txt' file using 'open' and read its content. Hopefully this should do the trick.  

So my final payload looks like this: -0;result=open('../flag.txt','r').read();trash=0+

You can also see how it will look when it gets passed to *compile*.
![Injection example]({{site.baseurl}}/assets/img/HackTheBoo_2022/evaluation_deck/injection_example.png){: .center-image}

### Getting the flag

The payload works as we expect and we receive the fake flag.  
So now it's time to start the docker container and try the same payload in the app that has the real flag.  

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/evaluation_deck/flag.png){: .center-image}

It works without any changes :) 