# Active - Writeup

**Date**: 27/06/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/148](https://app.hackthebox.com/machines/148)

---

Let’s start with the classic ping to test the connection with the target machine:

![Untitled](images/Untitled.png)

1 packet emitted, 1 packet received. The ttl shows a value of 127 which in HTB means that we are probably against a Windows machine.

Let’s do a scan of the TCP ports to find which ones are open:

![Untitled](images/Untitled%201.png)

Wow, it shows a bunch of open TCP ports. Let’s do a further scan in these ports:

![Untitled](images/Untitled%202.png)

We have much information here. First of all we have kerberos, RPC and ldap services. We also have a DNS service in port 53 and a http service running on port 47001.

Let’s see if we can any info from the DNS service:

![Untitled](images/Untitled%203.png)

Apparently nothing… Let’s see the http service:

![Untitled](images/Untitled%204.png)

Ok, we also have the port 445 open which is usually used by SMB… Let’s try to obtain more info using crackmapexec:

![Untitled](images/Untitled%205.png)

![Untitled](images/Untitled%206.png)

If we search the Build version, we can find that the target server is a Windows Server 2008 R2, SP1.

Now we know that the domain is `active.htb` let’s add it to the `/etc/hosts`.

![Untitled](images/Untitled%207.png)

![Untitled](images/Untitled%208.png)

But the http service looks the same.

Let’s try to enumerate the smb:

![Untitled](images/Untitled%209.png)

We have READ permissions to the folder Replication. Let’s look inside!

![Untitled](images/Untitled%2010.png)

![Untitled](images/Untitled%2011.png)

![Untitled](images/Untitled%2012.png)

Every folder at this level was empty.

![Untitled](images/Untitled%2013.png)

![Untitled](images/Untitled%2014.png)

It seems like it may have interesting files… let’s download all the folder to navigate more quickly:

`smbget -R smb://10.129.81.48/Replication`

![Untitled](images/Untitled%2015.png)

![Untitled](images/Untitled%2016.png)

![Untitled](images/Untitled%2017.png)

![Untitled](images/Untitled%2018.png)

![Untitled](images/Untitled%2019.png)

![Untitled](images/Untitled%2020.png)

![Untitled](images/Untitled%2021.png)

Maybe we have credentials here?

`active.htb\SVC_TGS : edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ`

![Untitled](images/Untitled%2022.png)

Is not that simple… Let’s see if the username at least is valid using kerbrute:

![Untitled](images/Untitled%2023.png)

Yes, it is. So we have a valid username but not its password I guess.

Doing some research, I found [this](https://vk9-sec.com/exploiting-gpp-sysvol-groups-xml/):

![Untitled](images/Untitled%2024.png)

So the password seems to be encrypted in AES-256 and we can crack it using gpp-decrypt.

![Untitled](images/Untitled%2025.png)

`SVC_TGS:GPPstillStandingStrong2k18`

Let’s save this credential in a file.

![Untitled](images/Untitled%2026.png)

And now let’s test it:

![Untitled](images/Untitled%2027.png)

Yes, it’s valid!

![Untitled](images/Untitled%2028.png)

Now, using this credentials we have access to more folders. Let’s look into `Users`:

![Untitled](images/Untitled%2029.png)

Can we list the Administrator folder?

![Untitled](images/Untitled%2030.png)

Nope. Let’s try with the rest:

![Untitled](images/Untitled%2031.png)

Apparently the userflag is in `Users/SVC_TGS/Desktop` path. Let’s download it!

![Untitled](images/Untitled%2032.png)

After enumerate the SMB I have found nothing else interesting, let’s try to do a `ldapdomaindumpH`

![Untitled](images/Untitled%2033.png)

![Untitled](images/Untitled%2034.png)

Como vimos antes, se trata de un Windows Server 2008 R2 SP1.

![Untitled](images/Untitled%2035.png)

Apparently there are only 4 users

- SVC_TGS: we have its credentials
- krbtgt: Key Distribution Center Service Account
- Guest
- Administrator

Let’s try a Kerberoast attack:

`❯ sudo python3 /home/angellm/THM/CTF/Relevant/impacket/build/scripts-3.9/GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.129.104.47 -request`

![Untitled](images/Untitled%2036.png)

Let’s now try to crack the hash using hashcat:

`hashcat -m 13100 -a 0 kerberoast_result /usr/share/wordlists/rockyou.txt`

![Untitled](images/Untitled%2037.png)

Cracked! The password was `Ticketmaster1968`

Let’s see if this credentials are correct: `Administrator:Ticketmaster1968`

![Untitled](images/Untitled%2038.png)

Yeah, Pwned! Let’s go for the root flag:

![Untitled](images/Untitled%2039.png)

![Untitled](images/Untitled%2040.png)

Done!