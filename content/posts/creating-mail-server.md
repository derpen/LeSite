---
title: "Creating a Bare Metal Mail Server, on Arch Linux"
date: 2023-07-17T16:06:30+07:00
tags: ["linux"]
draft: true
---

# For Debian User

Just use this script xd.

[Luke Smiths email server setup script.](https://github.com/LukeSmithxyz/emailwiz)

# Anything Else (Arch Linux specifically) User.

This is where the real fun begins :)

You can either follow along, or scroll down to the very bottom where I have uploaded the script. Do still check this post if you get stuck or contact me if something goes completely wrong.

# Introduction

A mail server consists of several components that basically works together to make sure that you can receive/send mails, prevent spams, not being blacklisted, and all those sorts. Now, I am by no means an expert, in fact, this entire post is basically just me covering my journey on how I try to port Luke Smiths script for Arch Linux system. This will either be the greatest thing I've ever done in my life, or something that I will regret for years to come.

This is also a continuation of my previous posts, where I demonstrate on how to turn your old laptop into a server accessible from the Internet. So some steps might be specific to my case. This place is still a good reference though if you want to get an idea on how to a mail server works.

# Domain, Nginx, and SSL Preparation

First, get a new domain for your machine. Make sure it points to the same ip as our original domain.

{{< figure src="/images/mail/1.png" >}}

The reason we are doing this, is because free dynu account are not allowed to create more than 4 dns records per domain, so if you plan to have multiple subdomains (blog, mail, etc), you won't have enough space. Thankfully 4 records are exactly the amount we need to create a mail server. Moreover, we can kinda just create a second domain that points to the same ip and dynu will just gladly accept it. So not only our first domain is free from extra dns entries, this also organizes our mail domains into one. Pretty cool eh. 

Now add a dns record in that new domain. Right now we are just going to add an extra MX records. Go ahead and put your domain the hostname and "mail" as the node name.

{{< figure src="/images/mail/2.png" >}}

Next, ssh to our server. We will be adding a new nginx server entry. To do that, you can create a new file with this configuration.

```
$ sudo su
# vim /etc/nginx/sites-available/mail 
```
``` nginx
server {
        listen 80 ;
        listen [::]:80 ;

        server_name mail.baremetalmail.loseyourip.com www.mail.baremetalmail.loseyourip.com ;

        root /var/www/mail;

        index index.html index.htm index.nginx-debian.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
Replace the server name, with whatever you just added in the records earlier.

Then, we rerun certbot
```
# certbot --nginx
```
Choose both of your mail server domain. Accept, and let it run. We now have a certificate for this domain. Next is start installing stuff.

First we run umask.
```
umask 0022
```


--------------- <+>Add github link -----------------
