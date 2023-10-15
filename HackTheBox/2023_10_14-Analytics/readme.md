# Analytics - Writeup

**Date**: 14/10/2022

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Analytics](https://app.hackthebox.com/machines/Analytics)

---

# Enumeration

Let's start by scanning the open TCP ports of the target machine:

![Untitled](img/Pasted image 20231014164407.png)

**nmap** reported tcp ports 22(ssh) and 80(http) to be open. Let's scan them further:

![Untitled](img/Pasted image 20231014164537.png)

**nmap** exhaustive scan reported that the service running on port 22 is OpenSSH 8.9p1. According to lauchpad, the target machine OS may be Ubuntu Jammy:

![Untitled](img/Pasted image 20231014164702.png)

Regarding the http service on port 80, the service running is **nginx 1.18.0**. The scan also reported that a redirection may be being applied to http://analytical.htb, so the server may be applying virtual hosting. Before adding this domain to the **/etc/hosts** file, let's check quickly the petition to the website using Burpsuite:

![Untitled](img/Pasted image 20231014165136.png)

As we can expect, the website is applying the redirection, and there is no information here we can use. So, let's add the domain to **/etc/hosts** file:

![Untitled](img/Pasted image 20231014165342.png)

The **nmap** script **http-enum** didn't find any common file in the server.

Let's see what technologies are being used in the website apart from nginx:

![Untitled](img/Pasted image 20231014165648.png)

The **whatweb** tool reports 2 email addresses (demo@analytical.com and due@analytical.com). It also reports that the website is using JQuery v3.0.0. This version of JQuery is outdated and is vulnerable to XSS.



Let's see how this page looks using the browser:

![Untitled](img/Pasted image 20231014165932.png)

There is a section called "Our Team" that shows photos, names and positions of workers. Let's write down this data that may be useful to check usernames.

![Untitled](img/Pasted image 20231014170151.png)

```

Jonny Smith - Chief Data Officer

Alex Kirigo - Data Engineer

Daniel Walker - Data Analyst

```


There is a contact form at the bottom on the page that doesn't seem to work.

By hovering the top menu "Login" link we can see that it will redirect us to http://data.analytical.htb

![Untitled](img/Pasted image 20231014170427.png)

Let's add this subdomain in the **/etc/hosts** file.

![Untitled](img/Pasted image 20231014170638.png)

Now, let's click on the link:

![Untitled](img/Pasted image 20231014170727.png)

It redirects us to a website where there is a service called Metabase. Let's search what is this:

![Untitled](img/Pasted image 20231014170853.png)



# Exploitation

After some research, I found this interesting blog entry at MetaBase's official webpage:

https://www.metabase.com/blog/security-incident-summary\

In this blog is explained that there were some programming errors that made the application vulnerable in older versions of it. 

![Untitled](img/Pasted image 20231014174703.png)

Checking for the setup token in the website of the target machine I found it:

![Untitled](img/Pasted image 20231014174923.png)

So, maybe the version of MetaBase is outdated and it is vulnerable.

I found this script written in Python that automates the exploitation process: https://github.com/robotmikhro/CVE-2023-38646

![Untitled](img/Pasted image 20231014175622.png)

After exploiting it, we gained a reverse shell!



![Untitled](img/Pasted image 20231014180625.png)

But it seems that we are inside a docker container. Let's see how can we escape from it.

# Docker breakout

If we check the environmental variables with `env`:

![Untitled](img/Pasted image 20231014185850.png)

We can find the credentials `metalytics:An4lytics_ds20223#`.

Let's try to check if they are valid to connect to the target machine via ssh:

Yes, the credentials are valid.

![Untitled](img/Pasted image 20231014190110.png)

![Untitled](img/Pasted image 20231014190349.png)

And that's how we got the user flag

# Privilege Escalation

Let's see the version of the Ubuntu and the kernel:

![Untitled](img/Pasted image 20231014195638.png)

It's an Ubuntu jammy, as we guessed in the enumeration phase. The Linux kernel is 6.2.0.

If we search for vulnerabilities of this kernel, we find this page: https://www.wiz.io/blog/ubuntu-overlayfs-vulnerability

In that article is explained that multiple versions of the linux kernel have a vulnerability related to the OverlayFS module that can be used to perform a Privilege Escalation. Apparently a similar vulnerability was detected and fixed back in 2021, but it happened again.

![Untitled](img/Pasted image 20231014200149.png)

According to the article, the version of the kernel that the target Ubuntu is using is vulnerable.

![Untitled](img/Pasted image 20231014200243.png)

The article also says that the old exploits still work for this vulnerability, so I'm going to use this one I found:

https://github.com/briskets/CVE-2021-3493

So, I downloaded the .c file and compiled it in my machine. Then I shared it with the target machine using an http server:

![Untitled](img/Pasted image 20231014200646.png)

Then, from the target machine, I downloaded the compiled exploit, gave it execution permissions and executed it.

![Untitled](img/Pasted image 20231014200812.png)

And this way I escalated privileges to root easily and read the root flag.



# New things learned

- The **environmental variables** should be checked every time.

- It's important to check the OS and kernel version and look for vulnerabilities.

