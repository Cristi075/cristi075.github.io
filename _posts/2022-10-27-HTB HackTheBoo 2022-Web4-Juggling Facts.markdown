---
layout: post
title:  "HTB HackTheBoo 2022 - (Web) Juggling Facts writeup"
date:   2022-10-27 16:00:00 +0300
categories: HackTheBoo_2022 CTF type_juggling php
summary: Writeup for the Juggling Facts challenge (Web4/5) from HackTheBoo 2022. This challenge involved exploiting a type juggling vulnerability in a php application.
---


'Juggling Facts' was a web challenge (day 4 out of 5) from HackTheBox's HackTheBoo CTF.  
Getting the flag involved exploiting a type juggling issue in a PHP app.

### What we got

Like all the other web challenges from this CTF, we got the code for the app and a docker container running it.  

Since I didn't want to install the php dependencies or build the container, I started with their container this time.  

### Analyzing the app

The webapp displays facts about pumpkins 🎃.  
These are grouped into three categories: "spooky", "not so spooky" and "secret".
![The webapp]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/webapp.png){: .center-image}

If we try to access the "secret" facts, we're told that we need admin permissions.
![The 'Secret' option]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/webapp_secret.png){: .center-image}

Let's take a look at the request made for that (using BurpSuite)
![Request made for 'secret' facts]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/request.png){: .center-image}

The error message is a different one; this ones states that we can only access the 'secret' facts through localhost.  
Let's take a look at the code in order to understand what's happening there.

### Analyzing the code

First, based on *entrypoint.sh*, we can confirm that the flag is stored in the facts table with the 'secret' type.  

In *indexController.php* we see the method used for retrieving the facts.  
The check that prevents us from retrieving those is checking for the 'type' variable to be equal to 'secrets' and for the server to address to be 127.0.0.1.  

I don't know how these requests could make to appear to be from localhost and I don't see a way of interfering with that variable.  
However, we notice that a strict comparison (===) is used for the 'security' check and then a switch statement is used to select the type of facts that are being retrieved.

![Vulnerable code]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/vulnerable_code.png){: .center-image}

The comparison made by default in a switch statement is equal to a loose comparison (==) in PHP.  
The difference between the two comparison types makes this code vulnerable to type juggling attacks. Also, the name of the challenge is a good hint in that direction too.

You can read more about type juggling vulnerabilities [here](https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09).  
Another very good resource that I found is this presentation from OWASP Day 2015: [link](https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf).

The presentation linked above also has this table that helps with visualizing what you might expect when comparing two different types (using the loose comparison).
![Type juggling table]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/type_juggling_table.png){: .center-image}

So, we have to find an input value that is not the string 'secrets' (so it won't match on the strict comparison) but that returns true when compared to the 'secrets' string using a loose comparison.

### Type juggling

To make this a bit easier, I installed php8 on a VM and created a short script that would compare my input with the 'secrets' string using both loose and strict comparisons.  

<details>
  <summary>test.php (click to expand)</summary>
<p>
{% highlight php %}
<?php

$input = json_decode($argv[1], true);

echo('JSON:' . PHP_EOL);
var_dump($input);
echo(PHP_EOL . 'Type field:' . PHP_EOL);
var_dump($input['type']);

echo(PHP_EOL . 'Loose' . PHP_EOL);
var_dump($input['type'] == 'secrets');

echo(PHP_EOL . 'Strict' . PHP_EOL);
var_dump($input['type'] === 'secrets');

?>
{% endhighlight %}
</p>
</details>

The most common payloads for type juggling involve comparing strings to integers.  
However, those won't work anymore in php8. You can find this being mentioned in the PayloadsAllTheThings repository, [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Type%20Juggling/README.md).

Based on the table shown above (this can also be found in the github repository linked above) another choice might be to use the 'true' boolean value.  
It should return true when compared with any string. In our case, the comparison with 'secrets' is the first one made inside the switch block so we can make use of this.

But first, let's test it using the script that I mentioned before.

![Type juggling]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/type_juggling.png){: .center-image}

It seems to work well: we get false when using a strict comparison and true when using a loose comparison.


### Getting the flag

Now let's try using it on the actual app.

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/juggling_facts/flag.png){: .center-image}
 
 Success!

### Can this vulnerability be fixed?

After seeing this I was thinking about if this vulnerability could've been prevented.  
According to [this](https://stackoverflow.com/questions/3525614/make-switch-use-comparison-not-comparison-in-php) thread on StackOverflow, it can't.  
Switch statements will always use loose comparisons.  


### Other web challenges from this CTF

This CTF released a challenge in each of its 5 categories each day.  
I have posted writeups for all the web challenges. Here are some links to them:
- Day 1 - [Evaluation Deck](/HTB-HackTheBoo-2022-Web1-Evaluation-Deck)
- Day 2 - [Spookifier](/HTB-HackTheBoo-2022-Web2-Spookifier)
- Day 3 - [Horror Feeds](/HTB-HackTheBoo-2022-Web3-Horror-Feeds)
- Day 4 - Juggling Facts (you are here)
- Day 5 - [Cursed Secret Party](/HTB-HackTheBoo-2022-Web5-Cursed-Secret-Party)
