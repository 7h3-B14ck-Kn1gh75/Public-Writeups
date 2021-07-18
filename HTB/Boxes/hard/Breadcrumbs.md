# Breadcrumbs HackTheBox Writeup

### Initial enumeration

![image](https://user-images.githubusercontent.com/65077960/124653190-e7c67080-de94-11eb-820d-a64e32d1abf9.png)

We'll start by enumerating port 80 (Their website).

![image](https://user-images.githubusercontent.com/65077960/124653340-180e0f00-de95-11eb-839e-be9548e78cf6.png)

It appears to be a bookstore, we can search for books on the check books page (http://10.10.10.228/php/books.php)

Using gobuster I found /PORTAL which linked to this page:
![image](https://user-images.githubusercontent.com/65077960/124653841-b4381600-de95-11eb-8a64-bf1262453995.png)
Using this we can make a list of admins.
* Alex
* Emma
* Jack
* John
* Lucas
* Olivia
* Paul
* William

This didn't really lead anywhere so I went back and looked into the books.php page and messed around with request
![image](https://user-images.githubusercontent.com/65077960/124653572-5dcad780-de95-11eb-8268-98b11a3366a4.png)

### User own

I managed to cause an error exposing some information and giving me a potential attack vector.
![image](https://user-images.githubusercontent.com/65077960/124654698-c5cded80-de96-11eb-91f3-6ae9bd17a205.png)

file_get_contents may lead to LFI, let's try it. I managed to grab the db.php file.
![image](https://user-images.githubusercontent.com/65077960/124655667-1134cb80-de98-11eb-9303-f3c6ad3ca82a.png)
This gives us the creds bread:jUli901

We can also get authcontroller.php

```
$secret_key = '6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e'
```
And cookie.php
```
<?php\r\n\/**\r\n * @param string $username  Username requesting session cookie\r\n * \r\n * @return string $session_cookie Returns the generated cookie\r\n * \r\n * @devteam\r\n * Please DO NOT use default PHPSESSID; our security team says they are predictable.\r\n * CHANGE SECOND PART OF MD5 KEY EVERY WEEK\r\n * *\/\r\nfunction makesession($username){\r\n    $max = strlen($username) - 1;\r\n    $seed = rand(0, $max);\r\n    $key = \"s4lTy_stR1nG_\".$username[$seed].\"(!528.\/9890\";\r\n    $session_cookie = $username.md5($key);\r\n\r\n    return $session_cookie;\r\n}
```
We can change our PHPSESSID to paul47200b180ccd6835d25d034eeb6e6390 and get access to the admin portal. We may then use a double extension to get a [reverse shell](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) and navigate the directories.

There are some credentials in 
```
C:\\Users\\www-data\\Desktop\\xampp\\htdocs\\portal\\pizzaDeliveryUserData\juliette.json
```
that we may use to SSH into Juilette's account and get the user flag.
![image](https://user-images.githubusercontent.com/65077960/124662250-7a204180-dea0-11eb-9630-a580552ce337.png)

![image](https://user-images.githubusercontent.com/65077960/124662418-b9e72900-dea0-11eb-982d-66284d851eee.png)

### Root own

After looking around the box and our current privilleges. I found Julie's sticky notes at:
```
C:\Users\juliette\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState>
```
![image](https://user-images.githubusercontent.com/65077960/124663341-dafc4980-dea1-11eb-9b50-8e253f299c7f.png)

I copied the files using SCP:
```
scp -r juliette@breadcrumbs.htb:"C:\Users\juliette\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\\" ./
```
![image](https://user-images.githubusercontent.com/65077960/124664618-9ec9e880-dea3-11eb-9469-6222792bc6e1.png)

We find the password for the development user and ssh into his account (fN3)sN5Ee@g)

Using netstat, we can see some ports that weren't previously open to us:
![image](https://user-images.githubusercontent.com/65077960/124665508-b5bd0a80-dea4-11eb-8920-1e52d5687db7.png)
Using curl we can see what they do, port 1234 looks interesting so we'll forward it to us.
![image](https://user-images.githubusercontent.com/65077960/124666053-62978780-dea5-11eb-8181-8632e59884e1.png)

We can use SQLMAP to automate our exploit
``` sqlmap -u "http://passmanager.htb:1234/index.php" --data="method=select&username=Administrat&table=password" -p username --dump ```

We're given the admin's hash which we can decrypt and get the password: p@ssw0rd!@#$9890./

