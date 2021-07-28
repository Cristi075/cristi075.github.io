---
layout: post
title:  "HTB Business CTF 2021 - NoteQL writeup"
date:   2021-07-27 20:00:00 +0300
categories: HTB_Business_CTF_2021 CTF
summary: Writeup for the NoteQL challenge from HTB's Business CTF from 2021. The challenge involved an unsecured GraphQL endpoint.
---

[NoteQL](https://ctftime.org/task/16673) was a challenge at the HTB Business CTF 2021 from the 'Web' category.  
After spawning the container for this challenge we got an URL that lead to a simple note-taking app.  

![Note taking app]({{app.baseurl}}/assets/img/HTB_Business_CTF_2021/noteQL_http.png){: .center-image}

If we are taking a look at what the app is doing, we can see a series of graphQL queries being made in the background.  
For example, this one of the requests seen in BurpSuite.

![GraphQL query]({{app.baseurl}}/assets/img/HTB_Business_CTF_2021/noteQL_graphQL_req.png){: .center-image}

And this is its response.

![GraphQL query]({{app.baseurl}}/assets/img/HTB_Business_CTF_2021/noteQL_graphQL_resp.png){: .center-image}

This means that we could probably change the query and see other parts of the database.  
First let's try learning more about the database.

<p>

{% highlight json %}

{ 
    "query": "{ __schema { queryType { name, fields { name, description } } } }" 
}

{% endhighlight %}
</p>

![GraphQL database info]({{app.baseurl}}/assets/img/HTB_Business_CTF_2021/noteQL_all.png){: .center-image}

Before, the application was querying 'MyNotes'. But we see that we also have 'Note', 'NotesFrom' and 'AllNotes'.  
Let's start by getting the content of 'AllNotes' since that seems to be the most comprehensive.  

<p>
{% highlight json %}

{
    "query":"{
        AllNotes {
            id,
            title,
            completed
        }
    }"
}

{% endhighlight %}
</p>

After sending that query we get this response.

![GraphQL query]({{app.baseurl}}/assets/img/HTB_Business_CTF_2021/noteQL_flag.png){: .center-image}

And the flag is easily readable in there.  
I guess this was pretty easy.