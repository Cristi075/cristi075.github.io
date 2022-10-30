---
layout: post
title:  "HTB HackTheBoo 2022 - (Forensics) Downgrade"
date:   2022-10-27 18:00:00 +0300
categories: HackTheBoo_2022 CTF forensics windows_logs
summary: Writeup for the Downgrade challenge (Forensics5/5) from HackTheBoo 2022. This challenge involved analyzing Windows evtx logs and identifying a suspicious login.
---


'Downgrade' was a forensics challenge (day5 out of 5) from HackTheBox's HackTheBoo CTF.  

### The format. What we got

For this challenge, we got access a zip file that contained Windows evtx logs.
We also got a server that we accessed using telnet that sent questions that you had to answer.  
At the end, you would receive the flag after answering all the questions correctly. 

### The first 3 questions

The first three questions could be answered without the logs. The answers could be easily found online by reading documentation if they were not already known.  

##### First question: Which event log contains information about logon and logoff events? (for example: Setup)
The **Security** logs contain the logon and logoff events.  
Reference: [Microsoft forum/Q&A](https://social.technet.microsoft.com/Forums/ie/en-US/03f0ad3d-4a52-49cb-ac4b-cac84cb03d0b/event-log-list-of-evtx-files-content-meanning?forum=win10itprogeneral)

##### Second question: What is the event id for logs for a successful logon to a local computer? (for example: 1337)
The answer here is **4624**, you can find those on [Microsoft's website](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)

##### Third question: Which is the default Active Directory authentication protocol? (for example: http)
The default protocol in AD is **Kerberos**. You can read more about it [here](https://sectona.com/pam-101/authentication/active-directory-based-authentication/)


### Question 4, Parsing the logs
Now we get to the fun part: the fourth question requires us to actually analyze the logs.
##### Fourth question: Looking at all the logon events, what is the AuthPackage that stands out as different from all the rest? (for example: http)

First, let's export the data in a more readable format.  
For that we're going to use [EvtxECmd](https://github.com/EricZimmerman/evtx).

{% highlight powershell %}
EvtxECmd.exe -f Security.evtx --json logs.json
{% endhighlight %}

![Exporting the data]({{site.baseurl}}/assets/img/HackTheBoo_2022/downgrade/extract_logs1.png){: .center-image}

In the output file, we now have a JSON string on each line.  
They aren't grouped into an array so we can't use [jq](https://stedolan.github.io/jq/) to filter the data.  
In order to fix this, we'll use a short python script that will do the following:
- read each line and fix some issues that we found
    - each double quote is preceded by a backslash. We'll have to remove those backslashes
    - when dealing with nested objects, each bracket { is preceded by a double quote and each } is succeeded  
- parse the JSON content of each line and add it to an array
- output the array using json.dump  
Note: since we're using jq anyway, pretty printing is not necessary so we're not doing it here

The script looks like this

{% highlight python %}
import json

logs = []

f = open('output.json', 'r', encoding='utf-8-sig')
for line in f:
        line = line.strip()
        line = line.replace('\\"', '"')
        line = line.replace('"{', '{')
        line = line.replace('}"', '}')
        logs.append(json.loads(line))
f.close()

f = open('output1.json', 'w')
json.dump(logs, f)
f.close()
{% endhighlight %}

And after running it we get a JSON array in the *output1.json* file. 

![Transforming the output]({{site.baseurl}}/assets/img/HackTheBoo_2022/downgrade/transform_output.png){: .center-image}

Now we can start looking for the protocols.  
We'll use jq to query the data. The main parts of our query are:
- .Payload.EventData - this is used for accessing the .Payload.EventData fields  
- select(type=="object") .Data - some entries have empty strings or null as their EventData. We want to filter those out and keep only the entries that contain more fields  
In this case, the type of the entries that we want is 'object' while the type for the others is either null or string
- .[] - the .Data field contained an array of objects. We want to iterate through those
- select(."@Name"=="AuthenticationPackageName") - then, we only want those that have "AuthenticationPackageName" as their "name"
- ."#text"' - and finally, we want to get the value contained within "text" for those entries (this is where the protocol is)

After jq outputs all the protocols used for authentication, we use 'sort -u' in order to get a list that contains each protocol only once.  
If you want to learn how these queries work start from scratch and add one element at a time.  

The command that we use will look like this

{% highlight bash %}
jq '.[] | .Payload.EventData | select(type=="object") .Data | .[] | select(."@Name"=="AuthenticationPackageName") | ."#text"' output1.json | sort -u
{% endhighlight %}

![Finding the protocols]({{site.baseurl}}/assets/img/HackTheBoo_2022/downgrade/check_protocols.png){: .center-image}

Now that we have the results, we can see that the outlier here would be the NTLM protocol.  
So, the answer to the fourth question is **NTLM**

### Question 5, analyzing access logs

##### Fifth question: What is the timestamp of the suspicious login (yyyy-MM-ddTHH:mm:ss) UTC? (for example, 2021-10-10T08:23:12)

We modify the previous query and make it print all the logs where the protocol was NTLM.  

{% highlight bash %}
jq '.[] | . as $log | .Payload.EventData | select(type=="object") .Data | .[] | select(."@Name"=="AuthenticationPackageName") | select(."#text"=="NTLM") | $log' output1.json
{% endhighlight %}

![Searching login events]({{site.baseurl}}/assets/img/HackTheBoo_2022/downgrade/searching_for_events.png){: .center-image}

This way, we got a list of login events. Since it a relatively low number of events, we can go through them manually.  

If we look at the last event, we can see that this is the only one where the RemoteHost is named 'kali'. Quite a big hint that this is what we're looking for.

![Kali login]({{site.baseurl}}/assets/img/HackTheBoo_2022/downgrade/kali_login.png){: .center-image}

The answer to the fifth question is the timestamp of this event, so **2022-09-28T13:10:57**

### Getting the flag

The fifth question was the last one, so after we sent that answer we got the flag.

Here you can see all the questions (and their answers) and the flag returned by the server.  
![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/downgrade/flag.png){: .center-image}

### Other forensics challenges from this CTF

This CTF released a challenge in each of its 5 categories every day.  
I have writeups for all the forensics challenges. Here are some links to them:
- Day 1 - [Wrong Spooky Season](/HTB-HackTheBoo-2022-Forensics1-Wrong-Spooky-Season)
- Day 2 - [Trick or Breach](/HTB-HackTheBoo-2022-Forensics2-Trick-of-Breach)
- Day 3 - [Halloween Invitation](/HTB-HackTheBoo-2022-Forensics3-Halloween-Invitation)
- Day 4 - [POOF](/HTB-HackTheBoo-2022-Forensics4-POOF)
- Day 5 - Downgrade (you are here)
