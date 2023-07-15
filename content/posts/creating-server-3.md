---
title: "Turning Your Old Laptop into a Bare Metal Server Part 3 - Nginx"
date: 2023-07-14T22:50:04+07:00
draft: false
featured_image: ""
tags: ["linux"]
---

# Engine-X

Nginx (pronounced as engine-x) is how you turn your machine into a real server. What we will do with nginx is to basically allow a folder and a port for our user to access. Now, by default, every port in Arch is opened, so you don't have to worry about firewall and such. But any other distro need to care about firewall.

First, install nginx
```
$ sudo pacman -S nginx
```

Switch to root user. We are going to modify a lot of stuff in /etc directory
```
$ sudo su
```

Navigate to /etc/nginx
```
# cd /etc/nginx
```

Create two folders. In debian, these folders are already there.
```
# mkdir sites-available
# mkdir sites-enabled
```
If you are using Arch, you need to modify **nginx.conf** to read sites-available folder by default. Add a line to include the sites-available folder.

{{< figure src="/images/baremetal/26.png" title="" >}}

We can now start adding a configuration for our site in the sites-available folder. Create a file inside sites-available, call it whatever you want.
```
# nano sites-available/site
```
Here is a simple default configuration to get you working.
```
server {

        listen 80 ;
        listen [::]:80 ;

        server_name example.com www.example.com;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
Save and quit. What this basically does, is to accept request that comes from the domain in server_name and http port. Then, it tries to find index file in /var/www/html. If it can't find it, it will return a 404. Sounds simple enough, right?

But, this site is not yet enabled. For that, we need to create a symbolic link from this file and put it into sites-enabled. You'll do something like this.
```
# ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/site
```
{{< figure src="/images/baremetal/27.png" title="" >}}

Cool, site is enabled. But we still have nothing in our /var/www/html directory. In fact, at least on Arch, /www/html does not even exist! Let's create it first.
```
# mkdir /var/www
# mkdir /var/www/html
```
Note: There is a way to do this in one command, but I forgot xP

Now, we need to get our hugo site in there. Remember how we can run **hugo** in our site directory, and it will compile all the files needed? Well, let's do that now.
```
# cd /home/derpen/MySite //replace with whatever is your site directory
# hugo
```
Now, we have a new directory called public. This contains all the good stuff. Now we copy the entire thing to /var/www/html.
```
# cp -r public/* /var/www/html/
```
Nice, we got everything that we need there. The last step is to activate nginx itself.
```
# systemctl enable nginx
# systemctl start nginx
```
Note: There is also a way to do this in one command xDDD

Oh, and if you are using debian, as always, you need the firewall to enable the port. Let's enable port 80 and 443. Port 80 is used for http and 443 for https. Now, we don't have https yet, we are going to use a tool called certbot later to get free ssl for our web. Free stuff :o wowwwie~!!
```
# ufw allow 80
# ufw allow 443
```

That should be all! You can now open the ip address of your server into your browser, and you should see your hugo server!
