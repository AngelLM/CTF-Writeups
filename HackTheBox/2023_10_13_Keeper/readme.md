# Pilgrimage - Writeup

**Date**: 13/10/2023

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Keeper](https://app.hackthebox.com/machines/Keeper)


---



# Enumeration

First of all let's check with TCP ports are open in the target machine using **nmap**:

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.227 -oG allPorts`

![Untitled](img/Pasted%20image%2020231013184126.png)

There are 2 TCP ports open: ssh (22) and http (80).

Let's do an exhaustive scan on this ports using **nmap** again:

`nmap -p22,80 -sCV 10.10.11.227 -oN targeted`

![Untitled](img/Pasted%20image%2020231013184329.png)

This scan reports the version of the OpenSSH service. If we search it in launchpad, the target machine appears to be an Ubuntu Jammy:

![Untitled](img/Pasted%20image%2020231013184544.png)

On the other hand, the http service is a **nginx 1.18.0**. 

`nmap --script=http-enum -p80 10.10.11.227 -oN webContent`

Let's run the nmap http-enum script to do a quick enumeration of common files and directories:

![Untitled](img/Pasted%20image%2020231013184833.png)

Nothing found, let's use **whatweb** to obtain more info about the applications and tools this website is using:

`whatweb http://10.10.11.227`

![Untitled](img/Pasted%20image%2020231013184955.png)

Nothing appart from the nginx version.

Let's take a look to the website using the browser:

![Untitled](img/Pasted%20image%2020231013185128.png)

I see, the server may be using virtual hosting, so we need to add the domain `keeper.htb` and `tickets.keeper.htb` subdomain to the `/etc/hosts` file:

![Untitled](img/Pasted%20image%2020231013185313.png)

After doing it, the keeper.htb page shows the same message, so let's check the tickets.keeper.htb subdomain:

![Untitled](img/Pasted%20image%2020231013185436.png)

Apparently the site is using an application called "Request Tracker" made by "BEST PRACTICAL" which is a real product.

A quick search shows that the default user for this application is **root** and its password is **password**. Let's check if it works:

![Untitled](img/Pasted%20image%2020231013194725.png)

Yes, it worked.

![Untitled](img/Pasted%20image%2020231013194909.png)

We can see info about users at Admin>Users menu.

There is a user called **lnorgaard**

![Untitled](img/Pasted%20image%2020231013195114.png)When we click in it's name, more info is displayed about this user. It says that the initial password of this user has been set to `Welcome2023!`



![Untitled](img/Pasted%20image%2020231013195342.png)At User Summary we can see that this user has requested a ticket about "Issue with Keepass Client on Windows"

![Untitled](img/Pasted%20image%2020231013195508.png)

The history of the ticket shows a conversation between the **root** user and **Inorgaard**. If we gain access as Inorgaard we should take a look to the crash dump file he says he saved into his home folder.

Let's try to log via ssh as `lnorgaard:Welcome2023!`.

![Untitled](img/Pasted%20image%2020231013200857.png)

Woah, we're in.

![Untitled](img/Pasted%20image%2020231013202858.png)

Inside the user folder we found the user flag and some interesting files mentioned previously in the ticket conversation. Lets download them:

![Untitled](img/Pasted%20image%2020231013203630.png)

After decompressing the ZIP file it appears that the content is the same we downloaded.

![Untitled](img/Pasted%20image%2020231013203816.png)

So, we have a memory dump of KeePass (dmp file) and a KeePass database (kdbx file).

KeePass is _a free open source password manager_. Passwords can be stored in an encrypted database, which can be unlocked with one master key. Old versions had a vulnerability that allowed to extract most of the characters of the master key from a memory dump.

[KeePwn](https://github.com/Orange-Cyberdefense/KeePwn) is a tool that automatizes the extraction of the characters of the master key from the memory dump.

![Untitled](img/Pasted%20image%2020231013205644.png)

The tool successfully found the master password! `rødgrød med fløde`

As we have the master password, we can see the content of the KeePass database we downloaded. To do so, we need a client for KeePass but I found this WebClient that works with KeePass files (https://app.keeweb.info/):

![Untitled](img/Pasted%20image%2020231013210548.png)

So, here we have a rsa-key for the user **root**.

And a password: `F4><3K0nd!`



According to [this website](https://www.baeldung.com/linux/ssh-key-types-convert-ppk) this is a Putty ID-RSA key, and we cannot use it to access via OpenSSH, but there is a way to convert it in order to use it with OpenSSH:

First step is to create a ppk file with the Putty RSA key content:

![Untitled](img/Pasted%20image%2020231013211803.png)

Next step is convert it by using this command: `puttygen id_rsa.ppk -O private-openssh -o id_rsa.pub`

![Untitled](img/Pasted%20image%2020231013212236.png)

The private key is extracted. If we also wanted to extract the public key, this is the command we should use: `puttygen id_rsa.ppk -O public-openssh -o id_rsa.pub`



Now, with the private key, let's try to connect via OpenSSH to the target machine as the **root** user:

![Untitled](img/Pasted%20image%2020231013212632.png)

We gained a root shell and found the root flag.



# New things learned

- How to extract the master password from **KeePass** dump.

- Convert a Putty ID_RSA key into an OpenSSH usable one.
