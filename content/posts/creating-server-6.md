---
title: "Turning Your Old Laptop into a Bare Metal Server Part 6 - Automation, with Jenkins!"
date: 2023-07-14T22:50:07+07:00
draft: false
featured_image: ""
tags: ["linux"]
---

# The Grand Finale

We got everything we need to finally move to the final part, where we configure jenkins. Sure, there are other, maybe better ways to sync your website, but this is a good opportunity to learn a proper deployment tool.

Let's install jenkins in our server.
```
$ sudo pacman -S jenkins

// Deb users
$ sudo apt install jenkins
```
After jenkins is installed, you can enable and start it.
```
$ sudo systemctl enable jenkins
$ sudo systemctl start jenkins

```
Now, in Arch, jenkins is a bit wonky. If you check the status,
```
$ sudo systemctl status jenkins
```
Apparently, it couldn't find our java directory, even though installing jenkins also automatically pulls in java as dependencies. [Arch wiki](https://wiki.archlinux.org/title/Jenkins) to the rescue! All we need to do is to point our java directory directly in the command line.
```
$ sudo nano /etc/conf.d/jenkins
```
You would want to change the JAVA variable to point to the correct directory. Apparently in Arch, you can just remove the **/bin** part. Like so.
{{< figure src="/images/baremetal/37.png" >}}

Also, in Debian, the default port is 8080. It's 8090 in Arch for some reason. I'll assume 8080 is the proper port, and we have do proper port forwarding for 8080 in our router earlier anyway, so let's change the JENKINS_PORT variable to 8080.

Now go ahead and restart jenkins
```
$ sudo systemctl restart jenkins
```
Run into another error? Something along the line of ? Somehow I got this error too when I'm using my fresh install from VM. The solution is simple actually, taken from this bad boy right [here.](https://stackoverflow.com/questions/19641449/jenkins-deployment-awt-is-not-properly-configured-on-this-server-djava-awt-he) You just install fontconfig and somehow it will works (im not sure why either??). 
```
$ sudo pacman -S fontconfig
$ sudo systemctl restart jenkins
```
Now, access your server for the first time. You can either use domain_name:8080 or local_ip:8080. I'll use my local ip for now and open 192.168.100.7:8080 from my browser.
{{< figure src="/images/baremetal/38.png" >}}

The default password is in **/var/lib/jenkins/secrets/initialAdminPassword**. We can use cat to see what is it.
```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Be wary though, after first usage, this password will be removed. So you better set up a very secure password after this.

When presented with the option to install plugin, just choose the suggested plugins. Let it run for a while, take a break while you are at it.
{{< figure src="/images/baremetal/39.png" >}}

Alright, now create an admin user. Choose a very very very secure password, i recommend you to use a password manager/generator for this one.
{{< figure src="/images/baremetal/40.png" >}}

Next is to set up your jenkins url. You will need to put your domain in here. This will be used later for Github webhooks.
{{< figure src="/images/baremetal/41.png" >}}

Before we use jenkins any further, do note that jenkins will create a user called jenkins in your machine. This jenkins user will have some permission fuckery with certain directories by default. Two directories what we need to think about is **/tmp/hugo_cache**, and **/var/www/html** (or wherever your site root leads to). Simply put, we need to change the owner of those two directory to be that of jenkins user. Before that, let's get rid of everything inside it first
```
$ rm -rf /tmp/hugo_cache/*
$ rm -rf /var/www/html/*
$ chown -R jenkins:jenkins /tmp/hugo_cache
$ chown -R jenkins:jenkins /var/www/html
```
We are now ready to create our first jenkins job.

In the left side of the dashboard, create a new item. Call it however you want. Save it as a freestyle project.
{{< figure src="/images/baremetal/42.png" >}}

Here are some configurations you want to enable.
{{< figure src="/images/baremetal/43.png" >}}

Replace your **github project** with whatever is the url for your site. To save space, you can also set **Days to keep builds** and **Max # of builds to keep** to 1. This will delete old builds that are no longer used, and since this is just s static site, we really don't care about saving previous versions.

{{< figure src="/images/baremetal/44.png" >}}

Enable git for source code management. Put the url of your repository. The difference between this url and earlier url is now you just need to add .git in the end. Basically like how you git clone, if you are familiar with it.

Make sure the branch points main, because that's how git and github works now, we no longer uses "master".

I'm not sure if this next part is needed, someone should correct me. For the tricky part, credentials would be empty by default. You need to add one first.
{{< figure src="/images/baremetal/45.png" >}}

Put your github username on the username field. For the password? First, you need to go to your github settings, and go to the [tokens page.](https://github.com/settings/tokens) We will be creating our personal access tokens. I will generate a classic token. 
{{< figure src="/images/baremetal/46.png" >}}

Now for this part, I think you can just enable the repo part. But you can also be a lazy fuck and enable everything. Your choice. Your key will not be shown. **MAKE SURE YOU KEEP THIS KEY VERY PROPERLY, THEY WON'T SHOW IT AGAIN (i believe)**.

Okay, now go back to your jenkins, and on the password field, put your credentials in there.

Sweet, now next option that you might need.
{{< figure src="/images/baremetal/47.png" >}}

Make sure to put build triggers for GITScm polling. And now we go back to our github. Go to the settings of your site repository (not your user settings page!). We will add a webhook here. 
{{< figure src="/images/baremetal/48.png" >}}

Remember how we configure our jenkins url earlier? This is where we need it. Now you add **your_domain:8080/github-webhook/** to your github webhooks. Leave the other at default settings, and save it. From now on, whenever we made a changes to main branch, github will ping our jenkins server to let them know that we just updated our site.

Finally, we add build steps. Add an execute shell. This will be simple three (four maybe) line script.
{{< figure src="/images/baremetal/49.png" >}}

You may be able to infer what the code does based on that. But basically, after pulling our git repo, it will run **hugo** to compile our site, remove everything in the **/var/www/html** directory to ensure a clean reset, and only after that, copy the entire **public** directory there. Save the settings, you are now ready to edit your site.

Now, from your server, let's create a new site

```
$ hugo new posts/second-post.md
```
Edit it however you want, and the push the changes to our repository.

```
$ git add *
$ git commit -m "Added second post yay!"
$ git push
```
Since we created an access token earlier, it might prompt you for your user and a password. Use your token as the password.

You can now see the build logs in your jenkins.
{{< figure src="/images/baremetal/50.png" >}}

If it fails, you can see what's going on from the console output in that logs. If it has something to do with permission, all you had to is to give jenkins permission to write into that directory.

We did it bros and girls. Now we can focus on writing actual content on our blog post. And whenever you are done with it, you just push the changes to your git repository. You now have a baremetal server for blogging and life is good again.

# Afterwords

"What now", you may wonder. "What other stuff can I do in my server other than a static site?". Worry not my friend, for the possibilities are endless. You can use it as a personal email server, a personal file storage, and anything you want basically. I would recommend you can visit [landchad.net](https://landchad.net/) on where to start (they use debian VPS there though, so some commands might be different!).
