---
layout: post
title:  "HTB HackTheBoo 2022 - (Web) Horror Feeds writeup"
date:   2022-10-27 16:00:00 +0300
categories: HackTheBoo_2022 CTF sql_injection
summary: Writeup for the Horror Feeds challenge (Web3/5) from HackTheBoo 2022. This challenge involved exploiting a SQL Injection vulnerability in a Flask application ... with a bit of a twist.
---


'Horror feeds' was a web challenge (day 3 out of 5) from HackTheBox's HackTheBoo CTF.  
Getting the flag involved exploiting a SQL injection vulnerability on an INSERT statement.

### What we got

Like the other web challenges from this CTF, we got access to a Docker container running a web app and the source code of that web app.

### Running the webapp locally 

First, let's take a look at the Dockerfile
<details>
  <summary>Dockerfile (click to expand)</summary>
<p>
{% highlight docker %}
FROM python:3.8-alpine

# Install packages
RUN apk add --no-cache --update mariadb mariadb-client supervisor gcc musl-dev mariadb-connector-c-dev

# Upgrade pip
RUN python -m pip install --upgrade pip

# Install dependencies
RUN pip install Flask flask_mysqldb pyjwt bcrypt colorama

# Copy flag
COPY flag.txt /flag.txt

# Setup app
RUN mkdir -p /app

# Switch working environment
WORKDIR /app

# Add application
COPY challenge .

# Setup supervisor
COPY config/supervisord.conf /etc/supervisord.conf

# Expose port the server is reachable on
EXPOSE 1337

# Disable pycache
ENV PYTHONDONTWRITEBYTECODE=1

# create database and start supervisord
COPY --chown=root entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
{% endhighlight %}
</p>
</details>

We are going to need Flask and some other modules (flask_mysqldb, pyjwt, bcrypt, colorama).  
We also have to install the 'libmysqlclient-dev' package (at least that's how it is called on Ubuntu).  
And we have to prepare a MySQL server to be used be the app. For this part, I started a MariaDB container using docker.  
After the database server is up, we have to create the DB and the table used by this app. You can use the SQL queries from the *entrypoint.sh* file.  

We access the application and we have two options: login and register.  
First we register a new account and then we login using it.  
We see that the application displays 4 .mp4 files and ... that's everything.
![The target app]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/webapp.png){: .center-image}

Since we couldn't find anything that looked interesting, let's look at the app's code.

### Code analysis. Identifying SQL injection

We find out that the flag is referenced in the *templates/dashboard.html* file.  
The flag is part of a table that is being rendered only if the current user's name is 'admin'.  

![Flag referenced in template]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/flag_template.png){: .center-image}

So now we know that we have to login as the 'admin' user or at least trick the server into thinking that we did.  

An obvious choice would be to register the 'admin' account, but that account was already created as part of the initial setup (in the entrypoint.sh file).  
We can also see why we can't create duplicate accounts by reading the code from the *database.db* file.  
By reading that file, we also see that one of the queries is vulnerable to SQL Injection.


![Vulnerable code]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/code.png){: .center-image}

Nearly all the queries made in this file use query_db, which will prevent SQL Injection attacks if they are detected.  
However, when registering a new user, the INSERT query is using an [f-string](https://peps.python.org/pep-0498/) to add variables to the query instead of the usual method of using the second argument of db_query in order to transmit variables.  

### Testing for SQL injection

First, we add a print statement that prints the SQL query that gets executed. This will help us with troubleshooting our payloads.  
![Modified code]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/modified_code.png){: .center-image}

Then, I tried to add another user named 'admin' by modifying the query to insert two users instead of one.  
The payload that I used for this was
{% highlight json %}
{"username":"testuser\", \"invalidpassword\"), (\"admin","password":"adminpassword123"}
{% endhighlight %}

This way, the SQL query became
{% highlight sql %}
INSERT INTO users (username, password) VALUES ("testuser", "invalidpassword"), ("admin", "$2b$12$1JTDmMTYgD/ejwtCy3rwLOF8U6oMKYKeejwkzOCMPbmb1/R2qUVim")
{% endhighlight %}

We can use the print statement that we added to confirm that the query was modified.
![Debug message: payload1]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/debug_payload1.png){: .center-image}

But this will fail because the 'admin' user already exists. When trying this, I haven't noticed that the 'username' flag had the UNIQUE flag set and I was hoping that I could create multiple users with the same name.
![Payload1 - failed]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/payload1.png){: .center-image}

At least we know that the attack works like we expect. We just need a different payload.  

### New payload

While looking for solutions, I found this blogpost: [https://labs.detectify.com/2017/02/14/sqli-in-insert-worse-than-select/](https://labs.detectify.com/2017/02/14/sqli-in-insert-worse-than-select/)  
It describes methods for exploiting SQL injection in insert statements and in 'duplicate key-based insertion' we can see a method that updates existing fields by using the "ON DUPLICATE KEY UPDATE" statement.  
You can read more about that SQL statement [here](https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html). Basically, when you attempt to insert a row that already exist, it will update the existing row instead of inserting a new one.  
We can use this in order to either change the admin's password to a known one and then use it to login or to change the admin's username so that we're able to create an 'admin' user.  
I went with the later and chose to update the *username* field.  

For that, my payload looked like this
{% highlight json %}
{"username":"testuser\", \"invalidpassword\"), (\"admin\", \"admin\") ON DUPLICATE KEY UPDATE username=\"notadmin\" #--", "password":"test123"}
{% endhighlight %}

And the resulting SQL query looked like this
{% highlight sql %}
INSERT INTO users (username, password) VALUES ("testuser", "invalidpassword"), ("admin", "admin") ON DUPLICATE KEY UPDATE username="notadmin" #--", "$2b$12$975Zz1I892U7CuGWFqIlBemvN7yBrbuqRYvDu0liYXcrK5XKMJgSK")
{% endhighlight %}

Again, I could see that the query was the expected one in the application's output
![Debug message: payload2]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/debug_payload2.png){: .center-image}

And the server returned a 200 status this time.
![Payload1 - success]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/payload2.png){: .center-image}

We can also verify that the user was update by looking directly at the database.

![DB users]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/db_users.png){: .center-image}

### Getting the flag

Now that we changed the name of the existing account, we can create an account named 'admin' and use it for logging it.

After that, we reproduce the attack on the 'production' version of the app (the one from their docker container) and we get the real flag.

![Getting the flag]({{site.baseurl}}/assets/img/HackTheBoo_2022/horror_feeds/flag.png){: .center-image}
