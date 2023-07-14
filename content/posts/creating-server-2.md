---
title: "Turning Your Old Laptop into a Bare Metal Server Part 2 - Creating basic blog post with hugo"
date: 2023-07-14T22:50:03+07:00
draft: false
featured_image: ""
tags: ["linux"]
---

# Getting Started with Hugo

Hugo is a static site builder written in go where you can focus on writing your blog using markdown. First, we need to install hugo and git.
```
$ sudo pacman -S hugo git
```

To create a new site,
```
$ hugo new site MySite
```
This will create a new folder called MySite. This folder will contain the bare minimum you need to create a static site.

We will also need to set up a theme before we start.

```
$ cd MySite
$ git init
$ git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
$ echo "theme = 'ananke'" >> hugo.toml
```
This will add a theme called ananke to your site. This is a very basic theme and is a good starting theme for noobs. You can inspect the content of **hugo.toml** if you want and play around with it.

To create a page for our posts, we can do something like this. You will need to do something like this anytime you want to create a new post. You can change the file name anyhow you please.
```
$ hugo new posts/first-post.md
```
Hugo will create a file in content/posts directory. Let's edit it.
```
$ nano content/posts/first-post.md
```
{{< figure src="/images/baremetal/21.png" title="" >}}

Everything that you write under this should be in syntax markdown. Let's add some stuff in it to test it. But first, notice how the third line says "draft: true". You would want to change this to false. Save it after you are done, and quit.

{{< figure src="/images/baremetal/22.png" title="" >}}

To see our website in action, you can do
```
$ hugo server --bind 192.168.100.7
```
If you didn't change the draft to false, you can also include draft files in our server by using this command
```
$ hugo server --buildDrafts --bind 192.168.100.7
// or
$ hugo server -D --bind 192.168.100.7
```

If you are running locally, you don't need to **--bind** and hugo will run on localhost:1313. If you are running it from external machine (like what we did here), we absolutely must add the **--bind** option (this took me an hour to figure out holyyyyy fuuuu). Replace ip with the ip of the your server. If my case, I can now access my site from a browser by going to 192.168.100.7:1313

If you are using debian or the sorts, I think port 1313 is blocked by default. To open it, simply
```
# ufw allow 1313
```

{{< figure src="/images/baremetal/23.png" title="" >}}

One last thing, you can just run hugo like so
```
$ hugo
```
Hugo will automatically compile all your posts, and put it into the **public** directory (The directory is literally called public). When deploying your site, you will need to copy the contents of this directory into the root folder of your web server. We will get to this in the next part, where I explain the basics of Nginx.

And, that's it. You can now focus on creating your site!

For more information on how to customize your site, you can visit hugo documentations [here.](https://gohugo.io/documentation/)

# Uploading to Github

We will store our site externally to Github. I prefer to store my code in my own git server (not like I have one), but for simplicity, this tutorial will use Github.

I'll assume you already have an account and get the gist of how Github works. We will create a new repository first in Github.

{{< figure src="/images/baremetal/24.png" title="Call the repo whatever you want" >}}

Then, assuming this is the first time you run git, run these commmands
```
$ git remote add origin https:github.com/your_username/your_repo_name.git
$ git branch -M main
$ git add *
$ git commit -m "Initialize"
$ git config --global user.email "your_email"
$ git config --global user.name "your_name"
$ git push -u origin main
```
{{< figure src="/images/baremetal/25.png" title="" >}}

You now have your site uploaded on Github. We will use this later so we can auto reload our website using Jenkins!
