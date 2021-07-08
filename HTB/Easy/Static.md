# Active HackTheBox Writeup

### Initial Enumeration
```
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5722/tcp  open  msdfsr           syn-ack ttl 127
9389/tcp  open  adws             syn-ack ttl 127
47001/tcp open  winrm            syn-ack ttl 127
49152/tcp open  unknown          syn-ack ttl 127
49153/tcp open  unknown          syn-ack ttl 127
49154/tcp open  unknown          syn-ack ttl 127
49155/tcp open  unknown          syn-ack ttl 127
49157/tcp open  unknown          syn-ack ttl 127
49158/tcp open  unknown          syn-ack ttl 127
49169/tcp open  unknown          syn-ack ttl 127
49171/tcp open  unknown          syn-ack ttl 127
49182/tcp open  unknown          syn-ack ttl 127
```
This is a windows box so I started by enumerating the SMB shares using smbclient:

```
smbclient -L 10.10.10.100
```

![image](https://user-images.githubusercontent.com/65077960/124631316-2cdda900-de7b-11eb-94b6-2915a7aed9f0.png)

We don't have access to Users so I started enumerating Replication.

![image](https://user-images.githubusercontent.com/65077960/124631702-8b0a8c00-de7b-11eb-91ac-fb9b6d25ea23.png)

There's an interesting file called Groups.xml, which we can grab the entire share using RECURSE in smbclient or using mount, for ease I used smbclient.
![image](https://user-images.githubusercontent.com/65077960/124632549-5c40e580-de7c-11eb-8452-9b0aecab2f28.png)

### User own

Now that I've got the entire share, I can use grep to search for creds or any potential footholds. I managed to find a hashed password:
![image](https://user-images.githubusercontent.com/65077960/124632910-a924bc00-de7c-11eb-9d09-c4ddd23f7dd4.png)
We can decrypt this using gpp-decrypt and get the password from it
![image](https://user-images.githubusercontent.com/65077960/124633327-0fa9da00-de7d-11eb-82ac-f6638fb4cfe8.png)

Now that we have access to SVC_TGS, we can use smbclient to access the user flag
```
smbclient \\\\10.10.10.100\\Users -u svc_tgs
```

### Root own
Using svc_tgs's creds, we can now use GetUserSPNs.py to grab the administrator's hash

![image](https://user-images.githubusercontent.com/65077960/124644060-bac09080-de89-11eb-8fc9-cf5555caa4db.png)

And decrypt it with hashcat:
![image](https://user-images.githubusercontent.com/65077960/124644117-cdd36080-de89-11eb-98cc-769edfac3291.png)

After getting the clear text password, we can use wmiexec to access a shell and read root.txt

![image](https://user-images.githubusercontent.com/65077960/124644211-e93e6b80-de89-11eb-845f-de8944a179dc.png)









