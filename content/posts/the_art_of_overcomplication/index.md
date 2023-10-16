---
title: "The Art of Overcomplication"
date: 2023-10-16T23:13:09+02:00
draft: true
weight: 0
tags: ["Personal", "Mental health"]
summary: ""
cover:
    image: "img/header.jpg"
    alt: "Header image alt text"
    relative: true
thanks:
    icon: "fa-solid fa-heart fa-3x"
    title: "Thanks for reading"
    text: "If you have any questions or feedback feel free to contact me through the means listed [on my main site](https://dylanmaassen.nl). Sharing my posts is also really appreciated!"
---

"Overcomplication" doesn't appear to actually be a word, if you ask any of the popular dictionaries, that is.
Is that going to stop me from using it anyway? *I don't think so.* 

![A prompt in my editor to add "Overcomplicated" to the spell check dictionary](dictionary.png)

It is merely the process of overcomplicating something. This is something that I catch myself doing all the time. Every new project I start, every single thing I try to do to 'improve' my workflows, ends up wasting a lot of time when I often end up on a road very well traveled and documented. Which incidentally is also why I stopped updating this blog. Well that and a severe lack of motivation, possibly related to aforementioned overcomplication.  

To be fair, I do start of every overcomplication with the feeling that I can absolutely, **100%** do better than whatever has been done before. And often times my ideas are pretty good. But it's lacking either in available software, my own skills or simply the technical possibilities at the time.

### An ongoing saga
For a practical example, take my development environment. I have settled on using a virtual machine running the server component for Visual Studio Code which is all fine and dandy. I picked this because I wanted to be able to connect to it from any machine. However, I ran into a snag doing this. The extension used to use the environment over SSH can also forward ports. However, on Windows I've found this to be less than stable. 

So I wanted a Linux development environment that ideally I could use from my Windows desktop. Not wanting to kludge around with local VMs I set up a VM on my Proxmox server, one with a graphical environment. And to connect to this I've tried a plethora of tools, each with their complications and/or high cost for an individual. I spent weeks trying these tools, trying to work with them and figuring out the latency is simply not comfortable to work with. And I know these tools can perform better, but my network uses a wireless backhaul and using something network based simply wasn't working out for me.

You know what I ended up going with? Laptop I already had, an HDMI cable, and the program Synergy to share a keyboard and mouse between Windows and Linux. No high costs spare for the initial purchase of the laptop. And of course 0 latency as opposed to the remote access tools I had tried. 

And as it turns out port forwarding through VS Code is pretty rubbish on Linux as well.
