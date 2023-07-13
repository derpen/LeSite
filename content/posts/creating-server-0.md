---
title: "Turning Your Old Laptop into a Bare Metal Server Part 0 - Getting Ready"
date: 2023-07-11T16:26:39+07:00
draft: true
---

Let's turn your old and unused laptop/pc into a server that you can access from the Internet.

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

Here, I'll tell you how to install Arch. If you don't plan on using Arch, you can skip this section and scroll down to the next part where we will be doing post install configuration (auto login without password and stop laptop from suspending whenenver lid is closed).

Installing and using Arch is not an easy feat, but succesfully doing so would give you so much knowledge and insights on how linux system works. If you are just starting to learn linux, eventually, at some point in your life, you will learn to install Arch. It's an unavoidable pipeline, so might as well have it happen now, right? After this section, you can finall tell your friends, that you indeed, "use Arch btw".

First, get yourself an Arch Linux iso. You can download it from [their site](https://archlinux.org/download/). Choose the closest server to you, I will go with singapore.


{{< figure src="/images/baremetal/1.png" title="" >}}

Clicking any of those link will redirect you to a list of files that you will need to choose. Choose the one that is 800 Mb in size. In this case, there are two file that is probably what we want. the **archlinux-2023.07.01-x86_64.iso** and **archlinux-x86_64.iso**. These file name will vary from time to time. I choose the 2nd option but either one should work. What's the difference between those two? Who fucking know honestly.

{{< figure src="/images/baremetal/2.png" title="" >}}

Next, we need a flaskdisk/thumbdrive/flashdrive and we are going to put your Arch iso on it. To make a bootable flashdisk, you can use stuff like rufus, balenaetcher, and more. I recommend to use [ventoy](https://www.ventoy.net/en/download.html). The process is simple, format your flashdisk, copy and paste the Arch Iso to your drive. Be careful though, formatting will delete everything in that drive, so make sure to back it up first. Cool thing about ventoy is that you can put as many Iso as the drive can fit, and later you can choose between any of them to boot from.

Okay, all done? Time to boot into your baby. Plug that flashdisk, with Arch inside it, and power on your boy. Immediately after powering on, you will then need to press a key on your keyboard to select which drive to boot from. This varies from machine to machine, but its usually either the Escape, Delete, Enter, F1, F2, F10, or F12 button. Your machine will tell you what to press on the monitor for a split second, so make sure you don't miss it. 

What if you fail and you boot into your current operating system? No problem, just turn it back off. Press the power button once more and try again. If you finally got to the select boot device screen, choose your drive that you have plugged. We are now in ventoy select screen. Now choose the Arch iso that we just copied (it's probably the only thing there if you are actually following this). Next during the Arch screen, press enter one more time. We are now ready to install Arch.

{{< figure src="/images/baremetal/3.png" title="Ventoy will look like this" >}}

First things first, we need to connect the machine to internet. To avoid headaches, plug an ethernet cable. Ethernet does not work somehow? We can use a built in command line tool called iwctl.
The syntax to use it are

```
$ iwctl
[iwd]# device list
[iwd]# station devicename scan
[iwd]# station devicename connect SSID

```

*Note: The command start after either the dollar sign, or the hash sign. I added that still just for clarity*

Basically, what those command does is
```
$ iwctl
```
is to get into the iwctl software
```
[iwd]# device list
```
is to list the devices that you have on your machine. Since we are trying to connect using wifi, it's probably going to be called wlan0. You will then replace the next command with wlan0. Like so
```
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect SSID
```
SSID will also needs to be replaced with the name of your wifi. Afterwards, it will prompt you for your password. Once all is done, you can quit out iwctl by either Ctrl+d or Ctrl+c.

To test if you got internet, you can try pinging any website.
{{< figure src="/images/baremetal/4.png" title="" >}}

Now we are rolling. Like with any other DIY distro, first step is to partition your drive. Let's first try the *lsblk* command to see whats available.
{{< figure src="/images/baremetal/5.png" title="" >}}

We got two drives, /dev/sda and /dev/sr0. I'm using a VM to demonstrate this, but most of the time, sda is the drive we want to install Arch on. You can infer which drive is which by the drive size. Now, we are going to partition it. We are going to run the fdisk command on /dev/sda
```
# fdisk /dev/sda
```
{{< figure src="/images/baremetal/6.png" title="" >}}

There are several commands you want to do in fdisk. First of, the **d** command. This will delete the first partition it detects inside the drive. You would want to do this multiple times until there are no partitions left. Basically, just press d and enter again and again until you get this error.
{{< figure src="/images/baremetal/7.png" title="" >}}

Then, we can start making new partition. Generally, you would want three partition. One for boot, one for root, one for home, and optionally, one more for swap. Here, I will just create two. One for boot and one for root. We won't use an entire partition for swap, but instead just a file. We'll get to that later.

For now, use the **n** command. This will create a new partition. It will prompt you to choose some stuff (partition type or whatever). You can just press enter again to choose the default. However, when it ask you for **Last Sector**, that's when we type something in. For the first partition, go ahead and type **+550M**. This will create a 550Mb partition. And then use **n** once more. This time, leave **Last Sector** empty. This will create a partition with the remaining drive size. To check if you did it properly, use the **p** command. You should now have two partition, /dev/sda1 and /dev/sda2.
{{< figure src="/images/baremetal/8.png" title="" >}}
{{< figure src="/images/baremetal/9.png" title="This is how it'll look" >}}

Alrite, now we use the **w** command to write the changes. I don't think I need to say this again, but this **WILL DELETE ALL DATA IN THAT DRIVE**. Afterwards, you can use lsblk again to inspect the drive. Remember, /dev/sda1 will be used for boot stuff, and /dev/sda2 will be for everything else.
{{< figure src="/images/baremetal/10.png" title="" >}}

Next, we will add a filesystem to the partition. We will add ext4 filesystem to /dev/sda2, and fat32 filesystem to /dev/sda1. Run these commands
```
# mkfs.fat -F32 /dev/sda1
# mkfs.ext4 /dev/sda2
```
If you are using non UEFI system, you would need to use ext4 aswell for /dev/sda1. Don't know if your machine uses UEFI or not? you can run
```
# ls /sys/firmware/efi/efivars

```
If a bunch of output shows up, that means you are using UEFI. Any modern laptop, or any laptop created above the year 2013 should be UEFI by default, so realistically you don't really need to worry about this.

Now, we mount those partitions. Use these commands
```
# mount /dev/sda2 /mnt
# mkdir /mnt/boot
# mount /dev/sda1 /mnt/boot
```
{{< figure src="/images/baremetal/11.png" title="Aftermath" >}}

Next, we will configure pacman, the package manager of Arch, and enable parallel downloading. I usually use **Vim** (😎), but let's just use nano, a more beginner friendly text editor.
```
# nano /etc/pacman.conf
```
Scroll down until you see the option of ParallelDownloads. By default, it's commented (there is a hash sign in front of the line), which means its turned off. Delete that hash sign, make sure there are no more leading whitespace, and change the number to however much you want. I usually go with 65. To save and quit in nano, you will need to press Ctrl+x, then press Y to confirm, then press enter to confirm again. You will need to do this again later btw.
{{< figure src="/images/baremetal/12.png" title="" >}}

Now, we are ready to install base Arch on our machine. Go ahead and run this command
```
# pacstrap /mnt base base-devel linux linux-firmware
```
What this command does basically is install the base of Archlinux in the /mnt directory, and also to install base, base-devel, linux, and linux-firmware to it. These are like the basics that you need to get a bare functioning system. Let it run for a bit, go grab a coffee, do some chore, or whatever.

Is it done yet? Good, we can go on to the next step. Setting up fstab.
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
This command will create a new file in /mnt/etc/fstab. This file is read by the system during startup, and will determine where which partition gets mounted to.

Now, all this time, we've been running Arch from the installation media (the flashdisk). Now, we want to do changes directly inside the system that we have installed. Since we installed it in /mnt, we will change our root to there. To achieve that we use this command
```
# arch-chroot /mnt
```

We are now inside our Arch system. Couple of things that we want to do here.
- configure pacman
- change timezone
- change locale
- create hosts and hostname
- configure user
- install and enable networkmanager
- install bootloader

First of, configuring pacman. We will do the same thing earlier, where we edit pacman.conf. Previously we used nano as our text editor. Since we have moved to a fresh system, nano is not here anymore. We need to reinstall it. To install something using pacman, you would want to do
```
# pacman -S package_name
```
So, since we want to edit pacman.conf using nano, the commands would be like so
```
# pacman -S nano
# nano /etc/pacman.conf
```
Change the parallel download like we have done before. And now we would like to change our mirrors for faster download speed. Edit this file
```
# nano /etc/pacman.d/mirrorlist
```
In this file, hopefully servers for multiple region is already listed there. Your task here is to just rearrange the lines, so that regions closest (or at least, you think is closest) is on the very top of the file. You can just highlight the line you need by holding your shift key, cutting it with Ctrl+k, and then pasting with Ctrl+u. In my case, I move the server url that has "asia" in it to the very top. 

Once thats done, we are now changing our timezone. To see all available timezones, you do this
```
# ls /usr/share/zoneinfo/
# ls /usr/share/zoneinfo/region_name/

```
{{< figure src="/images/baremetal/13.png" title="List of timezones" >}}

To actually change the timezone, you would need to create a symbolic link to /etc/localtime

```
# ln -sf /usr/share/zoneinfo/region_name/city_name /etc/localtime

```

Replace **region_name** and **city_name** with whatever matches your situation

Next, we run
```
# hwclock --systohc

```
This will (as far as I'm concerned) use your current system clock that we just configured, and synchronize the hardware clock based on that.

Next, we are generating locale. This will tell the system what language we would want to use. For this, edit the /etc/locale.gen file
```
# nano /etc/locale.conf

```
Here, you will need to uncomment the line that corresponds to what you need. If you just want classic english experience, scroll down a bit until you see something like **en_US.UTF.8**. Uncomment this line. Save, and quit. Setting locale is not done yet, we have to tell the system again what locale we want on another file. And after that, we run a command to generate the locales.
```
# echo "LANG=en_US.UTF-8" >> /etc/locale.conf
# locale-gen
```
Next step, is to create hostname and hosts file. These file will tell your network, how should our system be identified as. For this case, I will call it "baremetal".
```
# echo "baremetal" >> /etc/hostname
```
We are then required to edit /etc/hosts file. Open that guy with nano and edit it like so

{{< figure src="/images/baremetal/14.png" title="" >}}

Replace "baremetal" with whatever you just put earlier in /etc/hostname. Yes, that's a tab. Use tab. What does this file do you may ask? I don't fucking know honestly, I'm writing this at 1:08 AM, I'm not gonna bother researching what tf each line does. Figure it out yourself xd.

Next up, setting up user. First off, we ran **passwd**
```
# passwd
```
Enter the password. This will set up password for root user. Now we also want to set up normal user with normal privileges. I will call my user as derpen.
```
# useradd -m derpen
# usermod -aG wheel,audio,video,storage,optical derpen
# passwd derpen
```
This will add the newly created derpen user to some groups. The most important group is wheel. This group will allow this user to run commands that require root privileges. But we first need to install **sudo** to enable this functionality.
```
# pacman -S sudo
# EDITOR=nano visudo
```
Once you are in visudo, scroll down a bit until you see the Wheel line. Uncomment that shit. This will allow any user in wheel group to use sudo.
{{< figure src="/images/baremetal/15.png" title="" >}}

Next, configuring network.
```
# pacman -S networkmanager
# systemctl enable NetworkManager
```
**systemctl enable NetworkManager** (make sure you type capitalized letter right) will enable the NetworkManager service. What this mean basically is that whenever you reboot your system, NetworkManager will be immediately be on. Kind of like startup applications in MicrosoftWind*ws.

Before we go to the most important part, if you are installing on an ssd, you will want to enable trimming capability. This will increase your ssd lifetime.
```
# systemctl enable fstrim.timer
```
Now we are ready to install our bootloader, the most important part of the system. For this project, I'm going to use grub and efibootmgr
```
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --boot-directory=/boot
# grub-mkconfig -o /boot/grub/grub.cfg
```
This is where problem might happen. Most likely happens because turns our your system is too old for UEFI. When this happens, I think you have different parameters for **grub-install** (I'm not sure bruh). You will also need to format your boot partition as ext4 instead of fat32 like I have mentioned way way back. Oh well, good luck I guess xDD.

If everything goes right, you should now have a completely capable Arch system ready to go.

Get out of the root my pressing Ctrl+d, and then unmount everything
```
# umount -l /mnt
# shutdown now
```
Restart your system, and you should be able to login.

{{< figure src="/images/baremetal/16.png" title="" >}}

Finally, we need to reconnect to the Internet. If you are using wifi instead of ethernet, since now we are using NetworkManager, we now have different way of connecting to the internet (fucking tedious, ikr). It's pretty similiar to what we have done though, list all available network, connect to the SSID.
```
$ nmcli device wifi list
$ nmcli device wifi connect SSID password password_here
```
And uh... that's pretty much it :3

# Post-Install

Hopefully by now, you now have an machine runnning Arch. If you do, congratulations! Otherwise, I'm very dissapointed Anyway, time to get two things configured now. Auto login without pass and stopping your laptop from suspending when lid is closed. 

We would want the device to automatically login whenever we power it on. This saves us the pain from having to relogin manually everytime you have a blackout in your home. Obviously you dont want to enable this if you have another person who knew how to use the command line in the vicinity. Plus systemd will run services at boot before you even login. For those reason, this step might be unnecessary for you, but it's handy nonetheless.

Next, if you are using laptop, you might prefer to have it running with the lid closed. 

If you are using a Desktop computer, you could just let it run headless, a.k.a. unplug the monitor and the keyboard. But don't do that yet, we still haven't installed SSH and configure it. In the next part we will go through the process of getting ssh ready so you can conveniently access your server from your main machine.
