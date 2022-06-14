---
layout: post
author: Razor
title: "Hackthebox Delivery"
categories: [Hackthebox, Linux]
tags: [easy-box, enumeration, custom exploitation, privilage escalation, boot2root]     # TAG names should always be lowercase
image: assets/htb_delivery/htb-delivery-1.png
toc: true
---



### Nmap
As always we will start with nmap to scan for open ports and services:

![nmap](/assets/htb_delivery/image_nmap.png)

As we can see, we have 2 port open like ssh and webserver. Look at port 80 on browser

![image_web](/assets/htb_delivery/image_web.png)

Then you must see the source code. in the source code we found "delivery.htb" and "helpdesk.delivery.htb", so we can add domain to your host like this :

> $ sudo nano /etc/hosts

![etc_hosts](/assets/htb_delivery/etc_hosts.png)

when you have done the above, then we can immediately visit the web helpdesk.delivery.htb like this :

![image_helpdesk](/assets/htb_delivery/image_helpdesk.png)

### Open Ticket

Then let go to "Open new ticket" and fill it like this:

![create_ticket](/assets/htb_delivery/create_ticket.png)

So now you must remember the email and id ticket for future and if necessary you must Screenshot it

![create_ticket](/assets/htb_delivery/create_ticket2.png)

And now lets check ticket status with email that we made before

![create_ticket](/assets/htb_delivery/create_ticket3.png)

So now can view message at ticket, now we just have to look at others

![create_ticket](/assets/htb_delivery/create_ticket4.png)

let's look at the source code again and after that we'll see the link : delivery.htb:8065 then click on that and we create a new user using the email that we previously remembered has the domain @ delivery.htb

![create_ticket](/assets/htb_delivery/create_ticket5.png)

To see the verification submission and to activate the email we have to go again to the helpdesk and go to the "View ticket thread" tab

![create_ticket](/assets/htb_delivery/create_ticket6.png)

then click the verification link and then you will go in the internal channel like this :

![create_ticket](/assets/htb_delivery/create_ticket6.png)

if you have read all the words from the channel, there is a user credential. So we can login that mail delivery on SSH:

> User Credential : " maildeliverer:Youve_G0t_Mail! "

![create_ticket](/assets/htb_delivery/create_ticket7.png)

### Privilage Escalation

At this stage we need to find the highest possible access from this box. So first let's see what can be accessed :
> $ sudo -l
>
> //for check group sudo user
>
> $ id
>
> //for check id
>
> $ find / -perm -u=s -type f 2>/dev/null
>
> //for check SUID premission

![image_priv](/assets/htb_delivery/image_priv.png)

as we know the website uses mattermost chat service, let's find the file with the following command :
> $ find / -name "mattermost*" > sampah
>
> //and if done lets check this
>
> $ cat sampah

![create_ticket9](/assets/htb_delivery/create_ticket9.png)

### Mysql

we see there is something interesting in the opt let's enter into the folder. After logging in we can see the config.json which contains the credentials from mysql

![create_ticket9](/assets/htb_delivery/create_ticket10.png)

let's login the mysql account with the following command :
>$ mysql -h localhost -u mmuser -pCrack_The_MM_Admin_PW

![create_ticket11](/assets/htb_delivery/create_ticket11.png)

we are login with mysql account, so we can look any user credential in that. Lets following this command :
> show databases;
>
> //for showing databases
>
> show use mattermost;
>
> show tables;
>
> //for showing tables in mattermost

![create_ticket11](/assets/htb_delivery/create_ticket12.png)

lets see the username and password from table Users

> select username,password from Users;

![create_ticket11](/assets/htb_delivery/create_ticket13.png)

### Create Random Password
as we saw in the chat channel someone gave the hint of the root password, then now we have to develop the hint of the password using the various possibilities that we created with hashcat. let's follow this command

> sudo hashcat -r hint > Password.txt

![create_ticket11](/assets/htb_delivery/create_ticket14.png)
![create_ticket11](/assets/htb_delivery/create_ticket15.png)

### Crack Password
Now Lets crack the hash!!,
> sudo hashcat -D 2 -m 3200 hash Password.txt

![create_ticket11](/assets/htb_delivery/create_ticket16.png)

Lets see if hashcat done :

> sudo hashcat -D 2 -m 3200 hash Password.txt --show
>
> $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO:PleaseSubscribe!21

And Boom!! we got the password of root

![create_ticket11](/assets/htb_delivery/create_ticket17.png)
Happy hacking and keep it up :D