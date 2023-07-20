---
title: "Turning Your Old Laptop into a Bare Metal Server Part 5 - Getting a Domain"
date: 2023-07-14T22:50:06+07:00
draft: false
featured_image: ""
tags: ["linux"]
---

# "But public IP changes constantly, what if it suddenly changes??? I can't access my site ???????"

That my friend is why we need a dynamic dns service. This will allow you to map your ip to a domain, and change it automatically whenever it changes. Very handy! 

A recommended free dynamic dny server, that even got mentioned by Arch Wiki, is a service called [dynu.](https://www.dynu.com/en-US/) Go ahead and create an account there. Worried about privacy? Use a dummy email to sign up.

Once you are logged in, go to add dynamic dns page [here.](https://www.dynu.com/en-US/ControlPanel/AddDDNS) Use option one, to use their domain name (I never get the second option to work, not sure why lmao). Add the name to whatever you want.

{{< figure src="/images/baremetal/32.png" >}}

The IP address for that domain should be automatically filled by dynu. Now you can access your server using this domain provided by dynu!
{{< figure src="/images/baremetal/33.png" >}}

**A bit of note if you are planning to create a mail server with dynu, free accounts need to wait until their domain is 30 days old before you can set the TXT record. So do plan ahead.**

When accessing your domain for the first time, you should see a blank/error nginx page. That's because we haven't told our nginx server what domain it should look for. Go ahead and ssh into your machine, and change a couple of lines in your sites-available file.
```
$ ssh derpen@yourdomain // you can use your domain now!
$ sudo nano /etc/nginx/sites-available/site
```
Change the server_name line with whatever domain you just choose for your dynu.

{{< figure src="/images/baremetal/34.png" >}}

Finally, restart nginx
```
$ sudo systemctl restart nginx
```
Got your own domain name you can use? You can go to the dns panel of that domain provider, and add a CNAME record pointing to your current dynu domain. Something like this for example.

{{< figure src="/images/baremetal/35.png" >}}

If you do this, don't forget to configure your nginx to point to that new domain.

# Certbot

We are going to use a tool called certbot to give ssl certificate to our website, this will enable free https for our website!

First, install certbot,
```
$ sudo pacman -S certbot
```
In debian, or anything that use apt,
```
$ sudo apt install python-certbot-nginx
```
Now run 
```
# certbot --nginx
```
Read the prompt properly, it will prompt you for email and stuff. As long as you configure your nginx properly, certbot should have no problem getting ssl certificate for you. Got problem? Couple of ways you can troubleshoot this
- See if port 80 is open. This includes your sites-available configuration file.
- See if your sites-available points to the correct domain.
- Restarting nginx service after checking all of above.

Now by default, you need to manually rerun **certbot renew** command once in a while. This will be tedious, let's set up a cronjob to handle this. Cronjob is basically how you tell your system to run commands according to certain schedule.

On my Arch, I'm going to use cronie as a cron handler.
```
$ sudo pacman -S cronie
```
On debian, cron is enough
```
$ sudo apt install cron
```
And then, as root, you would want to edit cronjobs.
```
$ sudo EDITOR=nano crontab -e
```
Note: Remove EDITOR environment if you are a real man and want to use vi.

We will add the following line to cron.
```
1 1 1 * * certbot renew
```
The syntax of cron is a bit wacky at first, you can use [this](https://crontab-generator.org/) cheatsheat to kinda get an idea of how it works, but what this line actually means is to run certbot renew, every MONTH.

Next we enable our cronjob.
```
$ sudo systemctl enable cronie
$ sudo systemctl start cronie

// Or, for cron users
$ sudo systemctl enable cron
$ sudo systemctl start cron
```


This should all be it for ssl.

# Will my ip change automatically in dynu now?

Not yet! We need to constantly check our ip and update it whenever it changes! Let's create a shell script to handle this for us!

 Thankfully, dynu has made it easy for us to do this, by simply calling their api using curl.

```
curl "http://api.dynu.com/nic/update?myip=$ip&myipv6=no&username=USERNAME&password=PASSWORD"
```
You will need to replace your USERNAME and PASSWORD with your actual username and password. And for password, dynu recommends you hash your password first. Dynu has provided a tool to hash your password [here.](https://www.dynu.com/NetworkTools/Hash).

"What about $ip ? Where do I get that?". Well my friend, we are going to call this command to get our ip. Dynu already provided another api to check for your public ip.
```
curl checkip.dynu.com | awk '{print $4}'
```
Now we got both the commands we need to create our script. Go ahead and create a script file, maybe in your home directory.
```
$ nano update.sh
```
Add this to that file
```
#!/bin/sh

ip=$(curl checkip.dynu.com | awk '{print $4}')

curl "http://api.dynu.com/nic/update?myip=$ip&myipv6=no&username=USERNAME&password=PASSWORD"
```

Yes, you would want to include the first line that says **#!/bin/sh**. Don't leave that out. Change USERNAME and PASSWORD accordingly. Save and quit.

Now we need to  make it executable.
```
$ chmod +x update.sh
```
You can then run the script with
```
$ ./update.sh
```
Lastly, we add this script to our cron. Add a second line to our **sudo crontab -e**
```
* * * * * /home/derpen/update.sh
```
Replace the directory with wherever directory you put your script in. What this tells cron is to run the script, every MINUTE. We would want our ip to be updated as quick as possible, so doing it every minute is necessary. To test if it works, restart your router, wait a minute, and check your dynu control panel to see if ip changes.

Congratz, you got yourself, a proper domain. You can now access your machine, from the internet, using your domain, and not having to fear about not connecting to it (except me, I still have to fear about blackout, thanks PLN). In the next and final part, we will install and configure jenkins, so that our hugo site automatically gets updated whenever we push to our github.
