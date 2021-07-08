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
