---
title: "Turning Your Old Laptop into a Bare Metal Server Part 4 - Port Forwarding"
date: 2023-07-14T22:50:05+07:00
draft: false
featured_image: ""
tags: ["linux"]
---

# Configuring your router

My router is a Huawei HG8245A that I got from my ISP. It's pretty shit, connection randomly disconnects frequently, packet loss issue is a common thing, but hey, what can you do. We are going to need our router. Usually, your home router ip is 192.168.100.1

{{< figure src="/images/baremetal/28.png" title="">}}

By default, if you haven't set it up already, the default credential should be root and admin for user accountand password respectively. There is also the admintelecom trick (if you know you know (ytta)).

If you want to see your public ip here, you can go to status page, see the ip address for the 2nd WAN entry. If the ip start with a 10, for example 10.10.10.153, it means you have a private ip. A quick fix is to restart your router until your ip does not start with 10.

{{< figure src="/images/baremetal/29.png" title="">}}

But, what we want here is the DMZ configuration. Go ahead and go to **Forward Rules**, and **DMZ configuration**. Here, add your ip, or choose the hostname that is already registered in your router. Check Enable DMZ, and make sure to choose the WAN Name to the second entry that we have just saw earlier.

{{< figure src="/images/baremetal/30.png" title="">}}

Next, we need to configure which port to expose. Under **DMZ Configuration**, is **Port Mapping Configuration**. Go ahead and create a new entry.

{{< figure src="/images/baremetal/31.png" title="">}}

This basically means, route any connections coming on port 80 from the outside, to the port 80 of our target machine, which is our baremetal server. You can ignore some fields like what I did, what matters are the Start port and End port for both external and internal. 

In the image above I have opened Port 80, but go ahead and do the same thing 3 more times to open more ports. We need our ssh port (default is 22), 443 for HTTPS, and go ahead and also open port 8080 temporarily. We gonna need this port so we can use Jenkins later.

That's pretty much it! That's all we need to do to expose our ip to the Internet! Just like how my imaginary gf described it, "quick and dissapointing", but now you can put your public ip address to the browser and see your website running! 

There is also problem with private ip and public ip depending on your ISP, something like [this.](https://www.uplotify.id/cara-dapat-ip-public-indihome/). One solution is to keep restarting your router until you got a public ip. But I have restarted my router for like 52738942349 time, and I never once get a private ip (Nevermind, it immediately happens right when I write this). You can try to write a script to restart your router until it detects a public ip (I'm not sure how, good luck!).
