---
layout: post
author: Razor
title: "Tryhackme Classic Passwd"
categories: [Tryhackme]
tags: [cracking, reversing binary]     # TAG names should always be lowercase
image: assets/thm_classic_passwd/classic_passwd_thm.png
toc: true
---

### Introduction 
in this room you will train your skills in reverse engineer and bypass login. you can see room right here : [https://tryhackme.com/room/classicpasswd](https://tryhackme.com/room/classicpasswd)

### radare2

We can use radare2 to open binary and if you dont have it you can download right here : [https://github.com/radareorg/radare2](https://github.com/radareorg/radare2)
Then open radare2 and here is output:

```bat
$ r2 Challenge.Challenge                                                   1 ⨯
Warning: run r2 with -e io.cache=true to fix relocations in disassembly
[0x000010a0]> aa
[x] Analyze all flags starting with sym. and entry0 (aa)
[0x000010a0]> pdf @main
            ; DATA XREF from entry0 @ 0x10bd
┌ 31: int main (int argc, char **argv, char **envp);
│           0x000012f6      55             push rbp
│           0x000012f7      4889e5         mov rbp, rsp
│           0x000012fa      b800000000     mov eax, 0
│           0x000012ff      e881feffff     call sym.vuln
│           0x00001304      b800000000     mov eax, 0
│           0x00001309      e87bffffff     call sym.gfl
│           0x0000130e      b800000000     mov eax, 0
│           0x00001313      5d             pop rbp
└           0x00001314      c3             ret
[0x000010a0]> 

```
we don't get the any hint here therefore we use ltrace to simplify our analysis
then lets use ltrace 

### ltrace
if your dont have you can install in linux :


> sudo apt update && sudo apt install ltrace

now lets tun this with ltrace : 
> $ chmod +x Challenge.Challenge

```bat
$ ltrace ./Challenge.Challenge
printf("Insert your username: ")                 = 22
__isoc99_scanf(0x555b547fe01b, 0x7ffdad52ce30, 0, 0Insert your username: razor
) = 1
strcpy(0x7ffdad52cda0, "razor")                  = 0x7ffdad52cda0
strcmp("razor", "A---------7")                 = 49
puts("\nAuthentication Error"
Authentication Error
)                   = 22
exit(0 <no return ...>
+++ exited (status 0) +++e

```
And we can see the username " strcmp("razor", "A---------7") "
then input "A---------7"
### Flag
```bat
$ ./Challenge.Challenge                                                  130 ⨯
Insert your username: A---------7

Welcome
THM{6---------6}
```
and we get a flag 

Happy hacking and keep it up :D

