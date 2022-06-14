---
layout: post
author: Razor
title: "Root-me Cryptography"
categories: [root-me]
tags: [Cryptography]     # TAG names should always be lowercase
image: assets/root-me/bg.jpeg
toc: true
---

### Encoding - ASCII
```#
this challange come from root-met of type Cryptography

In GNU/linux, you can use the xxd command :
$ echo " 4C6520666C6167206465206365206368616C6C656E6765206573743A203261633337363438316165353436636436383964356239313237356433323465 " | xxd -r -ps
```
### Encoding - UU
```#
Below is python3 code

import uu
uu.decode('ch1.txt','output')
and then type linux command:

cat output
```
Hash - Message Digest 5
```#
Another answer posted that they used a wordlist, a pre compiled list of words, often already cracked or plaintext passwords from dumps to guess the right password. But MD5 i such a fast hash that i can brute force it instead.

$ hashcat -a 3 -m 0 7ecc19e1a0be36ba2c6f05d06b5d3058 -i --increment-max 12 -O -w 3

Explanation of command:
hashcat = the best GPU/CPU cracking software AFAIK, check it out! Free and opensource Hashcat
-a 3 = Attack mode 3. Attack mode 3 is brute force
-m 0 = Means hash-type 0, which in hashcats case is MD5.
7ecc19e1a0be36ba2c6f05d06b5d3058 = is the MD5 hash to crack
-i = This option tells hashcat to increment the guessing. On the first guess it only guesses one character space until it exhausts all guesses, on the next guess pass two character spaces, the next three and so on. x, xx, xxx, xxxx etc
—increment-max 12 = This option tells hashcat to keep increasing the length of the characters to guess until it reaches 12 characters in lengths.
-O = Optimized kernel (limits password length) usually used for faster cracking.
-w 3 = Workload profile 3. Hashcat supports workload profiles from 1-4 to increase or decrease the parallel workload in the GPU to make cracking faster. Number 3 is a good workload but 4 is "freeze my screen heavy workload".

After about 0.something seconds hashcat returned
7ecc19e1a0be36ba2c6f05d06b5d3058:weak
```
### Hash - SHA-2
```#
Hashes must have a-f and 0-9 characters only.
Following this rule we must exclude the "k" from the hash.
96719db60d8e3f498c98d94155e1296aac105ck4923290c89eeeb3ba26d3eef92
becomes
96719db60d8e3f498c98d94155e1296aac105c4923290c89eeeb3ba26d3eef92

$ hashid 96719db60d8e3f498c98d94155e1296aac105c4923290c89eeeb3ba26d3eef92
Analyzing '96719db60d8e3f498c98d94155e1296aac105c4923290c89eeeb3ba26d3eef92'
[+] Snefru-256 
[+] SHA-256 
[+] RIPEMD-256
Input the hash ’96719db60d8e3f498c98d94155e1296aac105c4923290c89eeeb3ba26d3eef92’ into https://crackstation.net/ and you’ll get ’4dM1n’

Then, finally, run:
echo -n "4dM1n" | sha1sum
a7c9d5a37201c08c5b7b156173bea5ec2063edf9
```
### Shift cipher 
```#
A quick way to find the offset we want (instead of just going through all potential values) is to have a look at the source ciphertext that we can reasonably expect to be a full sentence:
$ hexdump -C ch7.bin
00000000  4c 7c 6b 80 79 2b 2a 5e  7f 2a 7a 6f 7f 82 2a 80  |L|k.y+*^.*zo..*.|
00000010  6b 76 73 6e 6f 7c 2a 6b  80 6f 6d 2a 76 6f 2a 7a  |kvsno|*k.om*vo*z|
00000020  6b 7d 7d 2a 63 79 76 6b  73 72 7f 14 0a           |k}}*cyvksr...|
0000002d 
Those * characters (ASCII code 0x2A) look an awful lot like spaces (ASCII code 0x20) between words and applying the difference (-10) to the last byte also gives us a neat 0x00 end of string value...
Confirming it is quick:
>>> print "".join([chr(ord(c)-10) for c in open("ch7.bin", "rb").read()])
```

