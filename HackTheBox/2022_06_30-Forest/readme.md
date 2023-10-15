# Forest - Writeup

**Date**: 30/06/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/Forest](https://app.hackthebox.com/machines/Forest)

---

Let’s start testing the connection with the target machine by sending a ping:

![Untitled](images/Untitled.png)

The ttl confirms that we are against a Windows Machine. Let’s move to the nmap scan to see if there are any TCP port open:

![Untitled](images/Untitled%201.png)

There are many ports open! Let’s do a detailed scan to these ports:

![Untitled](images/Untitled%202.png)

![Untitled](images/Untitled%203.png)

Let’s start with the SMB (port 445). I’ll use crackmapexec to gather more info:

![Untitled](images/Untitled%204.png)

Let’s see if we can see something inside the SMB without credentials:

![Untitled](images/Untitled%205.png)

Not with smbmap, let’s try it with smbclient

![Untitled](images/Untitled%206.png)

Ok, apparently we cannot access to the SMB… let’s try to find some usernames using kerbrute:

![Untitled](images/Untitled%207.png)

We found some valid usernames! 

Let’s create a list of valid users:

![Untitled](images/Untitled%208.png)

Any of them would be AS-Rep Roastable?

![Untitled](images/Untitled%209.png)

None of the discovered user has the `UF_DONT_REQUIRE_PREAUTH` set, so no AS-Rrep Roast available.

Maybe is not the best idea, but let’s try to obtain a password of a discovered user using Kerbrute:

```bash
#!/bin/bash

File="validusers"
Lines=$(cat $File)
for Line in $Lines
do
	/opt/kerbrute/kerbrute bruteuser --dc 10.10.10.161 -d htb.local -t 200 /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt $Line
done
```

or do it with a one liner:

`cat validusers | while read LINE; do /opt/kerbrute/kerbrute bruteuser --dc 10.10.10.161 -d htb.local -t 200 /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt $LINE; done`

![Untitled](images/Untitled%2010.png)

But nah, no password has been discovered using this usernames and the password dictionary…

Let’s take a look to the ldap.

First of all, let’s see if it allows anonymous binds. To do so I can use `ldapsearch` tool:

`ldapsearch -H ldap://10.10.10.161:389 -x -b "dc=htb,dc=local”`

![Untitled](images/Untitled%2011.png)

The -x flag is used to specify anonymous authentication, while the -b flag denotes the base dn to start from. We were able to query the domain without credentials, which means null bind is enabled.
Now we can use `windapsearch` to obtain more info from the domain:

`/home/angellm/repos/windapsearch/windapsearch.py -d htb.local --dc-ip 10.10.10.161 -U`

- `-U` : Enumerate all users, i.e. objects with objectCategory set to user.

![Untitled](images/Untitled%2012.png)

![Untitled](images/Untitled%2013.png)

These users are the ones that we previously had… let’s try to obtain even more info using the flag `--custom "objectClass=*"` in order to obtain all the objects in the domain

![Untitled](images/Untitled%2014.png)

![Untitled](images/Untitled%2015.png)

The object `svc-alfresco` catches my attention. Let’s google it:

[Set up authentication and sync](https://docs.alfresco.com/content-services/7.0/admin/auth-sync/)

![Untitled](images/Untitled%2016.png)

So… this account seems to not require Kerberos preauthentication so… maybe we can get a valid TGT form it via AS-REP Roast:

![Untitled](images/Untitled%2017.png)

Yeah, we got a NTLM hash of the user svc-alfresco. Let’s try to crack it using john:

![Untitled](images/Untitled%2018.png)

Yeah, we have a password. Let’s see if it is valid using crackmapexec:

![Untitled](images/Untitled%2019.png)

Yes, its a valid credential: `svc-alfresco:s3rvice`

As we have a valid credential and there is a winrm service active in the port 47001, maybe we can try to gain access to target machine using `evilwinrm`:

![Untitled](images/Untitled%2020.png)

Yeah, we obtained a PowerShell. Let’s look for the user flag:

![Untitled](images/Untitled%2021.png)

Nice.

Now is time to escalate privileges. Let’s see which privileges this user has:

![Untitled](images/Untitled%2022.png)

And let’s see also the groups this account is in

![Untitled](images/Untitled%2023.png)

![Untitled](images/Untitled%2024.png)

I see nothing to use. But let’s use Bloodhound to see it more clearly:

![Untitled](images/Untitled%2025.png)

When imported into BloodHound it I searched SVC-ALFRESCO user and marked as OWNED.

![Untitled](images/Untitled%2026.png)

Double clicking on this user I see that it’s included in 9 groups:

![Untitled](images/Untitled%2027.png)

So, I click on the number 9 to display them:

![Untitled](images/Untitled%2028.png)

Mmmm… That group called ACCOUNT OPERATORS looks interesting. Apparently, members of this group are allowed create and modify users and add them to non-protected groups. So maybe we can use that.

Let’s go to the Analysis “Shortest Paths to High Value Targets”

![Untitled](images/Untitled%2029.png)

Is a little bit messy. One of the paths shows that the Exchange Windows Permissions group has WriteDacl

privileges on the Domain. The WriteDACL privilege gives a user the ability to add ACLs to an

object. This means that we can add a user to this group and give them DCSync privileges.

![Untitled](images/Untitled%2030.png)

![Untitled](images/Untitled%2031.png)

![Untitled](images/Untitled%2032.png)

![Untitled](images/Untitled%2033.png)

![Untitled](images/Untitled%2034.png)

![Untitled](images/Untitled%2035.png)

And now we can use secrets-dump to see the hashes!

![Untitled](images/Untitled%2036.png)

This is the hash of the administrator account: `32693b11e6aa90eb43d32c72a07ceea6`

We can perform a pass the hash attack to log in as the administrator:

![Untitled](images/Untitled%2037.png)

Yeah, we are logged as Administrator. Now is time to search the root flag:

![Untitled](images/Untitled%2038.png)