---
title: "The Art of Overcomplication + Mad About Convenience"
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

It is merely the process of overcomplicating something. This is something that I catch myself doing all the time. Every new project I start, every single thing I try to do to 'improve' my workflows, ends up wasting a lot of time when I often end up on a road very well travelled and documented. Which incidentally is also why I stopped updating this blog. Well that and a severe lack of motivation, possibly related to aforementioned overcomplication.  

To be fair, I do start of every overcomplication with the feeling that I can absolutely, **100%** do better than whatever has been done before. And often times my ideas are pretty good. But it's lacking either in available software, my own skills or simply the technical possibilities at the time.

### An ongoing saga
For a practical example, take my development environment. I have settled on using a virtual machine running the server component for Visual Studio Code which is all fine and dandy. I picked this because I wanted to be able to connect to it from any machine. However, I ran into a snag doing this. The extension used to use the environment over SSH can also forward ports. However, on Windows I've found this to be less than stable. 

So I wanted a Linux development environment that ideally I could use from my Windows desktop. Not wanting to kludge around with local VMs I set up a VM on my Proxmox server, one with a graphical environment. And to connect to this I've tried a plethora of tools, each with their complications and/or high cost for an individual. I spent weeks trying these tools, trying to work with them and figuring out the latency is simply not comfortable to work with. And I know these tools can perform better, but my network uses a wireless backhaul and using something network based simply wasn't working out for me.

New plan: Laptop I already had, an HDMI cable, and the program Synergy to share a keyboard and mouse between Windows and Linux. No high costs spare for the initial purchase of the laptop. And of course 0 latency as opposed to the remote access tools I had tried. 

As it turns out port forwarding through VS Code is pretty rubbish on Linux as well. So is trying to use a laptop exclusively with an external monitor. You gotta open it up to reach the power button, decrypt your hard drive and then let it boot completely before you can switch to the external monitor. All in all less than a minute of work but it's the kind of effort that makes me think twice about booting up. 

### And finally, bliss
So, turns out one of the more well-known Chinese brands of mini PCs has a unit that pretty much has exactly the specs of my laptop. That's perfect since my programming usually doesn't end up being very resource intensive. It's not like I'm actually compiling or anything. Oh, and it was on sale, not weird for a chip that is now 3 years old, but it'll do nicely. Now, when I first tried to boot a Kubuntu Live CD I ended up with green screen. That usually points to the GPU being a little silly goofy. And the Windows 11 install it unexpectedly came with (I think they shipped me the more expensive version) wouldn't boot any more. 

I was ready to register a return but decided to follow the manual's suggestion of clearing the CMOS, for which the unit has a pinhole up front. I fully expected this to not work because the unit was factory new. But lo and behold, Windows booted again and Kubuntu would load properly 🥳 Since then I've also learnt that the little HDMI switcher I have is trash (well I was asking it to do something above spec, so that's my bad.) and either the mini-pc or my USB-C dongle supports either display or USB, not both. But hey, I got a pretty stellar little Linux machine for under 300 euros, so I'm not complaining.  

### Experiences over possessions
Starting 2020 I decided I wanted to experience more, starting with concerts. Well Broadway musical would be another one if they do actually perform them here at some point. Of course, we needn't dwell on what happened in 2020. I got exactly 1 concert in and the others got cancelled. Although, seeing Cage the Elephant was probably the best first concert experience I could've had, and I was in luck because it was the last show of the tour the managed to do.

Then came 2022, and I got to see Modest Mouse live. One of my all-time favourite bands. I did have to stay in a wildly overpriced hotel in Amsterdam because it was also TwitchCon at the same time. That did not detract from the experience, the band was on fire, the venue was nice. There was a guy next to me full-body rocking out to every song and god I wish I had that confidence.

While writing the last paragraph I stopped in the middle because I had to leave for my first concert of the year (in October, so there won't be many to follow lmao). I went to go see Spinvis which is one of the few Dutch musicians I listen to. 
