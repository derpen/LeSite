---
title: "Turning Your Old Laptop into a Bare Metal Server Part 1 - Hardening SSH"
tags: ["linux"]
date: 2023-07-14T18:58:50+07:00
draft: false
featured_image: ""
---

In this part, we will harden our ssh. That is, we will make accessing our server harder for unauthorized user.

# Generating SSH keypair

First, we will generate a keypair. Instead of using a password, we will configure ssh to use a file containing a secret key. This is more secure than using password since they can't just brute force their way in.

To generate keypair, run this command on your **main machine**. You can even run this command on windows!
```
$ ssh-keygen
```
Optionally, **ssh-keygen** has some options you can use to toughen your key. The most commonly used is the -t and -b options.
```
$ ssh-keygen -t rsa -b 4096
```
What this does is to use rsa for the key algorithm, and to have 4096 bits in the key. 

It will then prompt you on where to save the file. You can change the file name at this point if you want (Will be useful if you are planning to connect to multiple servers later). It will also prompt you for a passphrase for that key. You can leave it empty if you want.

After this, your key should be in your home directory, specifically in **.ssh/**. If you check the inside of that folder, you'll see two files that is generated with the name you just chosen earlier.

{{< figure src="/images/baremetal/19.png" title="" >}}

The .pub file will be stored in your server, while the one without the .pub is the one that you keep secret in your main machine. Absolutely do NOT share this key to anyone you don't trust.

To copy this file to your server the hot way, you can do something like this from your main machine. Again, this command works on all platform.
```
$ scp .ssh/id_rsa.pub derpen@192.168.100.7:
```
Make sure you don't forget the colon at the end. Obviously, change the id_rsa to the file name, the user, and the ip to the one you had. This will copy the .pub key to the home directory of that user.
# The Hardening part

Now let's ssh to our server.
```
$ ssh derpen@192.168.100.7
```

Take a look at the your home directory using **ls**.

{{< figure src="/images/baremetal/20.png" title="" >}}

Our key has succesfully been copied. Next, we need to take the content of this file in the proper place. The proper place is **.ssh/authorized_keys**. By default, the .ssh folder is not created yet. So let's do that, and afterwards, we move and rename our **id_rsa.pub** into **authorized_keys**.
```
$ mkdir .ssh
$ mv ~/id_rsa.pub ~/.ssh/authorized_keys
```
We are now ready to tell ssh to use that key.

```
$ sudo nano /etc/ssh/sshd_config
```

There are couple of line you want to change
- Set **PermitRootLogin** to no
- Set **PubkeyAuthentication** to yes
- Set **PasswordAuthentication** to no

**Remember, if any of these lines is commented, remove the comments**

Optionally, you can change your port. This makes it harder for people to figure out which port your ssh is running on.

Now, you can restart ssh service
```
$ sudo systemctl restart sshd
```

Log out of your ssh connection by pressing Ctrl+d. Now, try to login again. Depending on what you do, it might result in an error.
```
$ ssh derpen@192.168.100.7
```
First, if you rename your ssh key to anything other than id_rsa, you have to specify it's directory in your command.
```
$ ssh -i ~/.ssh/keyname derpen@192.168.100.7
```
in Windows, its
```
$ ssh -i %HOMEPATH%/.ssh/keyname derpen@192.168.100.7
```
**Note: From this point onwards, I will keep using "~", Windows user needs to make sure they subtitute tilde to %HOMEPATH%**

Second, since we disabled root login, make sure you are not trying to connect as root.
```
// This won't work
$ ssh -i ~/.ssh/keyname root@192.168.100.7 
```

Thirldy, if you decided to change your ports, you also need to specify that port. For example if you change the port to 6969,
```
$ ssh -i ~/.ssh/keyname -p 6969 derpen@192.168.100.7
```

The end. Your ssh is now gajilion times more secure. Nobody can brute force a password attack anymore, and even if they did, they won't logged in as root. Again, make sure you keep your private key very very very safe.
