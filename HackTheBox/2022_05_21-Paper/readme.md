# Paper - Writeup

**Date**: 21/05/2022

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Paper](https://app.hackthebox.com/machines/Paper)

---

First things first. let’s test the connection with the target machine: 

![Untitled](images/Untitled.png)

The ttl value of 63 may indicate that the target machine is Linux.

Let’s launch a nmap scan in order to discover the open tcp ports:

![Untitled](images/Untitled%201.png)

There are 3 ports open: 22 (ssh), 80 (http), 443 (https). 

![Untitled](images/Untitled%202.png)

Let’s see what is hosted in the http and https ports:

![Untitled](images/Untitled%203.png)

![Untitled](images/Untitled%204.png)

![Untitled](images/Untitled%205.png)

![Untitled](images/Untitled%206.png)

Seems to be the same page.

![Untitled](images/Untitled%207.png)

Wappalizer confirms the versions of apache and openssl. I’m going to search if any of this services has a vulnerability I can use:

![Untitled](images/Untitled%208.png)

![Untitled](images/Untitled%209.png)

Not apparently… Let’s enumerate the directories using wfuzz:

`wfuzz -c --hc 404,403 -L -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.11.143/FUZZ`

![Untitled](images/Untitled%2010.png)

![Untitled](images/Untitled%2011.png)

Looks like a standard page…

Ok, no clues. Let’s go back and see what we found so far…

Taking a look to the whatweb response, there is something that looks like a domain… `office.paper` let’s add it to the /etc/hosts file and take a look to it in the web browser:

![Untitled](images/Untitled%2012.png)

Yeah, there is a website here!

![Untitled](images/Untitled%2013.png)

This site is using Wordpress 5.2.3

Let’s take a look to the page content…

![Untitled](images/Untitled%2014.png)

The post says that the only user in the blog is `Prisonmike`, but another user (`nick`) replied telling him that he has secret information in the blog drafts. If we gain access to the administration panel we should take a look to the drafts.

There is nothing interesting in the other 2 post available, but we can find other 2 posts if we click on `Search` button:

![Untitled](images/Untitled%2015.png)

A simple test post and another one of Nick reminding him to not write secrets in the drafts.

We didn’t found anything that could be a password for Prisonmike user, so let’s try to login with default credentials:

![Untitled](images/Untitled%2016.png)

`admin` is not a valid user, but `prisonmike` is. But we still don’t know the password.

![Untitled](images/Untitled%2017.png)

![Untitled](images/Untitled%2018.png)

Using searchsploit I found a exploit that seems capable of view unauthenticated posts…

![Untitled](images/Untitled%2019.png)

Let’s try it!

![Untitled](images/Untitled%2020.png)

So, yeah, we have access to the draft posts contents… There is one with a “secret” url that seems interesting… Let’s add `chat.office.paper` to /ect/hosts file and visit it with the web-browser

![Untitled](images/Untitled%2021.png)

It is a register page, let’s register a new user:

![Untitled](images/Untitled%2022.png)

![Untitled](images/Untitled%2023.png)

Automatically I get invited to a chat:

![Untitled](images/Untitled%2024.png)

Let’s take a look to the chat messages:

![Untitled](images/Untitled%2025.png)

![Untitled](images/Untitled%2026.png)

So, let’s open a private chat with Recyclops and see if we can enumerate something:

![Untitled](images/Untitled%2027.png)

![Untitled](images/Untitled%2028.png)

![Untitled](images/Untitled%2029.png)

Let’s see if it’s vulnerable to path traversal:

![Untitled](images/Untitled%2030.png)

Yep, it is… and we should have access to user flag this way:

![Untitled](images/Untitled%2031.png)

Not that easy… yep, it is only readable by the owner… there will a ssh key?

![Untitled](images/Untitled%2032.png)

Nope… but the .hubot_history sounds interesting:

![Untitled](images/Untitled%2033.png)

There is a connect command? I tried to use it, but it doesn’t seems to work.

![Untitled](images/Untitled%2034.png)

![Untitled](images/Untitled%2035.png)

![Untitled](images/Untitled%2036.png)

![Untitled](images/Untitled%2037.png)

![Untitled](images/Untitled%2038.png)

![Untitled](images/Untitled%2039.png)

![Untitled](images/Untitled%2040.png)

woah, we found credentials: `recyclops:Queenofblad3s!23`

Let’s see if we can login as recyclops in the chat:

![Untitled](images/Untitled%2041.png)

![Untitled](images/Untitled%2042.png)

Nope, we can’t… Recyclops is a bot made by Dwight… Will him be reusing credentials? Let’s check it via ssh:

![Untitled](images/Untitled%2043.png)

Yeah!

![Untitled](images/Untitled%2044.png)

Escalation

i.sh

![Untitled](images/Untitled%2045.png)

![Untitled](images/Untitled%2046.png)

![Untitled](images/Untitled%2047.png)

![Untitled](images/Untitled%2048.png)