---
title: "Trying Out AutoMLs - The Horrors of 'It Just Works' Automation Library"
date: 2023-07-23T13:01:19+07:00
draft: true
---

Wasted my entire weekend trying to get automls to work. It's even worse knowing that every library advertises themself as "even noobs can make ML models". I'm using python 3.9.

Lots of frustration attempting to get things to work in kaggle and colab. Gave up, installed on local instead, won't work anyways. Fking cunts

"Skill Issue" you might say, well if it does not work on any system, including mine, and takes some finiggling before it actually performs the bare minimum, then it defeats the point of being an "easy" and "quick" solution. Not to mention there is too much variable to troubleshoot. Error might come from pip, it might come from python version, it might come from the package itself, or it might come from their dependencies. Not worth the time in my opinion, but who knows, it might work on your machine. My advice is just to git gud and know which algorithm to use from which library. Or create your own wrapper script to handle the automation rather than trying to rely on these "just works" library.

Main take:
- Why tf every package wants to download to my home dir instead of custom/current directory ??????? In my opinion, by default, it should switch to my current env folder. Change your pip cache directory immediately.
- By default they want to do their own data preprocessing, which feels weird tbh because every library have their own format for the "raw" data. This means unironically I have to preprocess it again each time I want to use different library.

PyCaret:
- Dogshit documentation, lots of deadlinks. New link actually have outdated information.
- Support for NLP actually dropped. No reason given, yet their documentation still implies that NLP is indeed possible.
- Might still be good for any other ML case though, but for NLP? Don't bother.

for NLP, 0/10

FLAML:
- Had to download huggingface, torch, and nvidia cudnn. This cudnn from pip might clash with my currently installed cudnn from pacman. Nevermind, didn't even work. CUDA ran out of memory error. Okay, then I'll just turn off the GPU option right? False, it still tries to use CUDA. I immediately gave up at this point because I'm already pissed that this library had me installed gigabytes of dependencies just for it to tell me CUDA ran out of memory when I specifically says not to use GPU. T
- Documentation kinda sucks too. I can't find all the options I can use for fit function. Because of this, I can't find if there are other ways to disable using gpu completely. There is no forums online explaining how to do this either. Unlucky.

Immediate 0/10 because it depends on billion gillion zillion amount of libraries. You think internet is free in third world country??

TPOT:
- It runs I guess, so an Instant 9/10. It needs to run for 311 minutes though, and it only gets around 0.47 score at its best. Problem probably came from my dataset, so whatever. Documentation is rather lacking, but since it's still a pretty new project, I will also let it slide.

Auto-Sklearn:
- Using a way too outdated scikit-learn version. Will conflict with other libraries and cause dependecies nightmare where you had to unintall and reinstall. 
- It spent 10 minutes trying to find the perfect model, did not find any. Could be my fault for not waiting a bit longer.
- Unlike FLAML or (older version of) Pycaret, this one does not have its own NLP classification helper. It does have classic classification you'd usually use for iris dataset, which is what I tried. But I was a bit worried that I'll be waiting for nothing. Maybe I should give it more time, but I'm at the point where I just want to be done with this. At least it does not give me unsolvable errors. Might try this again in the future.
7/10 because It's at least runable without issues. There is no progress bar though, you could let it run for an entire day and you wouldn't know if it's near ending or not. Documentation is also pretty good. Although outdated scikit gave a little bit of headache.

Lazypredict:
- The fastest. Took 7 minutes using data that TPOT took 311 mins. Tbf, this one did not try every single parameter like TPOT did. But this is still lightning fast nontheless.
- No errors on first run either, actually pretty cool
10/10, will use this again in the future.

Before releasing this blog, I will make sure to retry everything again, on different environment (running locally right now), and with different dataset. Still NLP case, but we shall see.

Oh, btw, google is so shit nowadays. I lost count on how many AI generated articles I stumbled upon that is basically just chatgpt answering "explain me the X library" without giving me actual example on how to use said library. I might switch to duckduckgo from now on.

In the end, I think I'll go back to tensorflow. I could have spent my weekend at learning pytorch instead, but oh well, wcyd..
