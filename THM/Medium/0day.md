# 0 Day TryHackMe Writeup

### User own  
> Initial hint: Exploit Ubuntu
> Skipping enumeration because the exploit is obvious

A turtile in a hurricane, shellshock. Let's exploit it:  
![image](https://user-images.githubusercontent.com/65077960/125212074-d7eaca00-e2a2-11eb-86cb-05958d06cd31.png)  
We'll use metasploit's exploit for this and set our options:  
![image](https://user-images.githubusercontent.com/65077960/125212157-6b23ff80-e2a3-11eb-8712-7f88ea7bb6a3.png)  

We can just run "exploit" and get our shell:  
![image](https://user-images.githubusercontent.com/65077960/125212190-97d81700-e2a3-11eb-977f-8207f6205521.png)  
We can get the flag at /home/ryan/user.txt

### Root own  
Our hint is that the OS is old, we take a look at the uname and see this:  
![image](https://user-images.githubusercontent.com/65077960/125212682-09fe2b00-e2a7-11eb-9bc9-1c20123caf0c.png)  
The current OS is vulnerable to [overlayfs Local Priv Esc](https://www.exploit-db.com/exploits/37292).  
Let's upload it and try it:  
![image](https://user-images.githubusercontent.com/65077960/125212859-0fa84080-e2a8-11eb-93d9-c06e7e6723f7.png)  

It worked! We can get root's flag from /root/root.txt
