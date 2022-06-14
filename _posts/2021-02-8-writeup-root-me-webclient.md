---
layout: post
author: Razor
title: "Root-me Client"
categories: [root-me]
tags: [web-client]     # TAG names should always be lowercase
image: assets/root-me/bg.jpeg
toc: true
---


### HTML - disabled buttons 
```#
All you have to do is open a browser inspectior of your choosing and examine the code. (If you are using chrome, you can get to this by pressing Ctrl + Shift + I, or by right clicking and choosing ’Inspect’.)

The focus of this challenge is on the input fields underneath the "Under Construction’ header. The code should look like this:

<input disabled type="text" name="auth-login" value="" />
<input disabled type="submit" value="Member access" name="authbutton" /> 
What you need to do is remove the ’disabled’ attributes from both lines of code. This should allow to use both the button and entry fields. After that, enter something into the textbox and click the button. It does not matter what you type as the page will recognize it as a valid ’access’. However, if you just click the button without entering anything, the page will simply refesh and the textbox and button will be disabled again.
```
### Javascript Authentication
```#
first : login use admin /admin OK it’s js login from alert  😄
second: press ctrl+shift+c " Inspect Element"
third : choose network
fourth: refresh the page
fifth:after refresh page in top click in login.js then click to response >>the password is sh.org  😄
```
### Javascript - Obfuscation 1
```#
first : click cancel button after that click OK button
second : press ctrl+u to view source pass ’%63%70%61%73%62%69%65%6e%64%75%72%70%61%73%73%77%6f%72%64’
third : open terminal linux then type
echo %63%70%61%73%62%69%65%6e%64%75%72%70%61%73%73%77%6f%72%64 | xxd -r -p

the password is >> cpasbiendurpassword
```
### Javascript - Obfuscation 2 
```#
<script>
var pass = unescape("unescape%28%22String.fromCharCode%2528104%252C68%252C117%252C102%252C106%252C100%252C107%252C105%252C49%252C53%252C54%2529%22%29");

document.write(unescape(pass)+"<br>")
document.write(String.fromCharCode(104,68,117,102,106,100,107,105,49,53,54));

</script>
```
### Javascript - Obfuscation 3
```#
String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));

Using one of my favourite tools, CyberChef, it was real simple to bake:
Open https://gchq.github.io/CyberChef/ and paste in the hex above. Drag "From Hex" to the recipe box. Now you’ve got the Decimal ASCII representation:
55,56,54,79,115,69,114,116,107,49,50

...So convert from Decimal now, by dragging "From Decimal" to the recipe, under "From Hex", and change delimeter to "Comma". Bam! Done, ez pz.
7[.....]2
```

