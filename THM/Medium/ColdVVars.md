# Cold VVars TryHackMe Writeup

### Initial enumeration
```
PORT     STATE SERVICE     REASON         VERSION
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
8080/tcp open  http        syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8082/tcp open  http        syn-ack ttl 63 Node.js Express framework
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.11 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.91%E=4%D=7/10%OT=139%CT=%CU=41081%PV=Y%DS=2%DC=T%G=N%TM=60E99075%P=x86_64-pc-linux-gnu)
SEQ(SP=101%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)
OPS(O1=M506ST11NW6%O2=M506ST11NW6%O3=M506NNT11NW6%O4=M506ST11NW6%O5=M506ST11NW6%O6=M506ST11)
WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)
ECN(R=Y%DF=Y%T=40%W=F507%O=M506NNSNW6%CC=Y%Q=)
T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)
IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 44.802 days (since Wed May 26 18:05:11 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host: INCOGNITO

Host script results:
|_clock-skew: mean: -59m58s, deviation: 1s, median: -59m59s
| nbstat: NetBIOS name: INCOGNITO, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   INCOGNITO<00>        Flags: <unique><active>
|   INCOGNITO<03>        Flags: <unique><active>
|   INCOGNITO<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 58670/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 46410/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 16428/udp): CLEAN (Failed to receive data)
|   Check 4 (port 30799/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: incognito
|   NetBIOS computer name: INCOGNITO\x00
|   Domain name: \x00
|   FQDN: incognito
|_  System time: 2021-07-10T11:20:04+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-10T11:20:03
|_  start_date: N/A
```

Using enum4linux we can get some information on SMB:  
![image](https://user-images.githubusercontent.com/65077960/125161314-84c92800-e179-11eb-8dc8-85205e33c0fd.png)  

We have some user accounts too:   
![image](https://user-images.githubusercontent.com/65077960/125161347-b3df9980-e179-11eb-8b37-6c4a456d01bc.png)  

We can't access the "SECURED" share  
![image](https://user-images.githubusercontent.com/65077960/125161400-0a4cd800-e17a-11eb-9eb8-a94b32cc432f.png)  
Let's check out the website on port 8080 instead.   

![image](https://user-images.githubusercontent.com/65077960/125161438-5009a080-e17a-11eb-8ad1-38a1976e5873.png)    
> An Ubuntu machine with SMB shares? That's rare  

There's two files (got from Uniscan), index.html and index.php  
![image](https://user-images.githubusercontent.com/65077960/125161476-85ae8980-e17a-11eb-957b-f33d99fd9cbe.png)  
Nothing but the word "data", or at least that's all we can see. What about 8082?  

![image](https://user-images.githubusercontent.com/65077960/125161505-b098dd80-e17a-11eb-860b-0adfdc8363e5.png)    
That's a little more interesting, we'll run uniscan against this too.  
### User Own

We've got /login, we might be able to bypass it? Let's try SQLI, we'll use the following payload:  
```
" or 1=1 or "
```
We get 4 usernames and passwords (Including ArthurMorgan). We can try sign in with the creds but we just get the creds echoed back at us. Let's try that SMB share instead:  
![image](https://user-images.githubusercontent.com/65077960/125162242-a973ce80-e17e-11eb-9156-acc52f74f7f1.png)  
We see there's a note in the SECURED share, we'll download it and see what it says.  
```
Secure File Upload and Testing Functionality
```
That's not particulary helpful, or is it?   
I took a step back and looked at port 8080, there's a note on the website too:   
![image](https://user-images.githubusercontent.com/65077960/125162308-ff487680-e17e-11eb-8782-283f277b37f4.png)  
What if we upload a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell), let's try it.  
![image](https://user-images.githubusercontent.com/65077960/125162370-5a7a6900-e17f-11eb-97aa-ee2bf9c26762.png)  

We're in, we'll upgrade our shell and su into Arthur's account.
> python3 -c 'import pty;pty.spawn("/bin/bash")'

![image](https://user-images.githubusercontent.com/65077960/125162418-98778d00-e17f-11eb-8941-06c0d2caaea6.png)  
We can find the user flag in Morgan's home directory

### Root Own  
We have a file in our home directory called ideas:  
```
I don't know why I don't get any ideas to write here...
```
Nothing that interesting, let's keep looking. I'll upload linpeas via a python server and execute it.  
There's a strang environment variable:  
![image](https://user-images.githubusercontent.com/65077960/125162872-2b192b80-e182-11eb-9234-ccc75230da6b.png)  
We can set up a netcat listener on the shell that we've got and see what it does.  
![image](https://user-images.githubusercontent.com/65077960/125162906-5a2f9d00-e182-11eb-9fd4-281fd5a49476.png)  
Let's see what we can do! We can select the 4th option and get access to vim. We'll spawn a shell using:
```
:set shell=/bin/sh
:shell
```
![image](https://user-images.githubusercontent.com/65077960/125162971-d5914e80-e182-11eb-91be-21d63ee55f5d.png)  
Our shell is pretty unstable, let's create another reverse shell. We'll upgrade our shell using python then exploit tmux to get a shell as root.
```
export TERM=xterm
tmux attach-session -t 0
```
and then exit over and over until we get our shell. The root flag can be found in root's home directory!


