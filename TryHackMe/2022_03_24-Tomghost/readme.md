# tomghost CTF - Writeup

Are you able to complete the challenge?

**Date**: 24/03/2022

**Difficulty**: Medium

**CTF**: [https://tryhackme.com/room/tomghost](https://tryhackme.com/room/tomghost)

# Compromise this machine and obtain user.txt

First of all, let’s do a quick scan of the open ports of the target:

![Untitled](images/Untitled.png)

Nmap discovers 4 open ports: 22, 53, 8009 and 8080 let’s do a proper scan to those ports:

![Untitled](images/Untitled%201.png)

Let’s see if there is anything in the port 8080 we can view using the web browser:

![Untitled](images/Untitled%202.png)

So, it seems like the owner of the target system has been installed tomcat recently. The version installed is the 9.0.30. Let’s see if we find any vulnerability that allow us to exploit the target.

![Untitled](images/Untitled%203.png)

As we can see in the fixes of newer versions of Tomcat, there were some important vulnerabilities with the version 9.0.3. Let’s check exploit database to see if there is any exploit we can use.

Looking for the CVE-2020-1938 a exploit appears:

![Untitled](images/Untitled%204.png)

![Untitled](images/Untitled%205.png)

Let’s use it with msfconsole:

![Untitled](images/Untitled%206.png)

Let’s configure it:

![Untitled](images/Untitled%207.png)

The RHOSTS parameter is the IP adress of the target, but I’m not sure about the File name, so I’ll keep it as it is and try once:

![Untitled](images/Untitled%208.png)

How lucky! We obtained what it looks like a username and a password hash! Let’s try to crack it using John The Ripper! But first, we should discover in which format has the password been hashed:

![Untitled](images/Untitled%209.png)

Strange, I supposed it to be a password hashed, not a password in plain text... In the first scan we have seen that there is a ssh service open, let’s try to log in with this credentials:

![Untitled](images/Untitled%2010.png)

Woah, it worked... Let’s look around for the user.txt file

![Untitled](images/Untitled%2011.png)

It catched my eye the .asc and .pgp files

![Untitled](images/Untitled%2012.png)

Let’s see if we can use the tryhackme.asc key to de-encrypt the credential.pgp file:

![Untitled](images/Untitled%2013.png)

![Untitled](images/Untitled%2014.png)

The key has a password, let’s transfer the key file to our machine and try to crac;k the password using John The Ripper:

![Untitled](images/Untitled%2015.png)

Before trying to crack it with John The Ripper we have to convert the file using `gpg2john`

![Untitled](images/Untitled%2016.png)

And then, let’s try to crack it!

![Untitled](images/Untitled%2017.png)

We got the password in no time. Let’s go back and try to find the user.txt file first:

![Untitled](images/Untitled%2018.png)

Let’s see if we can read it with the current user:

![Untitled](images/Untitled%2019.png)

Yes, we can and that’s how we get the first flag of this CTF.

# Escalate privileges and obtain root.txt

It’s unlikely, but let’s see if we can find the root.txt file with the current user:

![Untitled](images/Untitled%2020.png)

Nope, let’s de-encrypt the credential.gpg file:

![Untitled](images/Untitled%2021.png)

we got what it looks like a username and password again? Let’s try to switch to that user:

![Untitled](images/Untitled%2022.png)

We can’t, I double checked. It would be the ssh password? Let’s try:

![Untitled](images/Untitled%2023.png)

It is. Let’s see if this user has permissions to read the /root folder:

![Untitled](images/Untitled%2024.png)

Nope... So we have to do more privesc. Let’s see if this user can run sudo commands:

![Untitled](images/Untitled%2025.png)

Niiice, so we can run zip as root! This is exploitable for sure, let’s check it out at GTFOBins:

![Untitled](images/Untitled%2026.png)

Let’s try it!

![Untitled](images/Untitled%2027.png)

It worked! Let’s find the root flag!

![Untitled](images/Untitled%2028.png)

And that’s it! I’ve been lucky finding the sudo permission at first try!