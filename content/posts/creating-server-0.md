---
title: "Turning Your Old Laptop into a Bare Metal Server Part 0 - Getting Ready"
date: 2023-07-11T16:26:39+07:00
draft: true
---

Let's turn your old and unused laptop/pc into a server.

Here, I will show you how to create a basic blog post, using just your laptop, and that trash default router provided by your ISP.  So this is practically a project you can do for free.

Overall, the steps to achieve this is are as follows:

0. Get that unused device out of the closet and install linux on it.
1. Install and configure ssh
2. Install and trying hugo
3. Install and configure nginx
4. Do some port forwarding from your router
5. Getting a domain with a dynamic dns service
6. Install and configure jenkins (to auto reload your site)

For my router, I use the default router given by my ISP (some random HUAWEI from Indihome). And for my laptop, I will be using my HP Pavilion Sleekbook 14, with a dual core AMD E1-1200 APU running at an amazing 1.4Ghz. Its equipped with a 256 Gb SSD and 8 Gb of ram.

# Preparing your bad boy

Surely you have at least one machine in your home that you don't use (if not, unlucky I guess). Obviously, make sure that little fella can still power on. In my case, my laptop has a dead battery, so I'm forced to always plug it to the wall. Next step is to go to the bios of your device. One of the most important bios setting that you need is the "auto power on" setting. More modern device would have an option to have your device automatically turn on when it either receives AC power or Lan power. This option is mega important, enable it.

Unfortunately, my laptop is too old for this option to exist, so I can't really show you how to do it. And that also means that I'm stuck with turning it back on manually whenever a blackout happens (Thanks PLN).

Other thing that you might want to do is to install ssd and upgrade the ram. The difference between using an ssd and hdd is extremely noticeable even as a server, especially if you have a machine with shit processor like mine. Save yourself from the pain, buy ssd and ram upgrade for your laptop.

Next, you would need to install a linux distro of your choice. Here, I will be using Arch. Mainly for one reason, I can't get wifi to work properly on any other distro. Heck, some distro (fedora and openbsd (not a distro, but you know)) just outright refuses to install. And since I will be using ssh anyway to access my laptop, I don't need any GUI garbage installed. For newbies I would recommend to just install debian, but you can still use any other distro that you want (Fedora, Ubuntu, Mint, Alpine, Gentoo, etc).

# Installing Arch

Here, I'll tell you how to install Arch. If you don't plan on using Arch, you can skip this section and scroll down to the next part where we will be doing postinstall configuration (auto login without password and stop laptop from suspending whenenver lid is closed).

Installing and using Arch is not an easy feat, but succesfully doing so would give you so much knowledge and insights on how linux system works. If you are just starting to learn linux, eventually, at some point in your life, you will learn to install Arch. It's an unavoidable pipeline, so might as well have it happen now, right? After this section, you can finall tell your friends, that you indeed, "use Arch btw".

```
# arch-chroot

```

# PostInstall

Hopefully by now, you now have an machine runnning Arch. If you do, congratulations! Otherwise, I'm very dissapointed Anyway, time to get two things configured now. Auto login without pass and stopping your laptop from suspending when lid is closed. We would want the device to automatically login whenever we power it on. This saves us the pain from having to relogin manually everytime you have a blackout in your home. Obviously you dont want to enable this if you have another person who knew how to use the command line in the vicinity. Plus systemd will run services at boot before you even login. For those reason, this step might be unnecessary for you, but it's handy nonetheless.

Next, if you are using laptop, you might prefer to have it running with the lid closed. 

If you are using a Desktop computer, you could just let it run headless, a.k.a. unplug the monitor and the keyboard. But don't do that yet, we still haven't installed SSH and configure it. In the next part we will go through the process of getting ssh ready so you can conveniently access your server from your main machine.
