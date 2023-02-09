---
title: "Homelabbing: Part 1"
date: 2023-02-09
draft: false
weight: 0
tags: ['Homelab', 'Proxmox', 'Linux', 'Hardware']
summary: "In this x part series Dylan explores the wonder of trying to run server workloads on consumer hardware. This is part 1 in which he struggles to install Proxmox VE"
cover:
    image: "img/server.webp"
    alt: "A (human) server serving a croissant to a patron at a restaurant."
    caption: "Haha they're a server, get it? (please laugh) - [Credit](https://www.pexels.com/photo/woman-in-black-long-sleeve-shirt-sitting-on-chair-3907080/)"
    relative: true
---

In the past I have spent copious amounts of money on cloud computing services and I have decided that enough is enough.
So now I'm spending copious amounts of money on hardware to hopefully make something that can pass as a server.

## Specifications

At the moment the rig consists of the following:
- Intel Core i5 12600
- 32 GB of 3200Mhz Corsair Vengeance LPX RAM
- MSI PRO Z690-P DDR4 motherboard
- Fractal Design ION+ 2 560W Platinum PSU
- Seagate FireCuda 530 500 GB M.2 SSD
- Samsung 870 EVO 1 TB SSD
- Samsung 970 EVO Plus 500 GB M.2 SSD (already had one lying around)

Oh! And it's all wrapped in a be quiet! Pure Base 500 case but that doesn't make it serve any faster.  
I did replace the case fans with be quiet! Silent Wings 3 and boy are they silent. 
Like I haven't measured, but I'd be hard-pressed to hear the system at all.

## Dawn of the first snag

Nothing ever comes easy, does it?

The system went through its first POST correctly after a short period of self-discovery (at least I think that's what rebooting thrice means). And Virtualization support was enabled out of the box.  
I flashed the Proxmox VE 7.2 ISO to a flash drive, and it booted from the flash drive straight away. All is happy in the world right? Right.

The installer for Proxmox (based on Debian) halted on a very descriptive `Waiting for /dev to be fully populated`.

Now, the cause for this issue was pretty easy to find on Google, it was Linux trying to wrap its little head around getting the graphics to work. 
Apparently this particular version of the kernel doesn't quite support the integrated graphics on a 12600 yet.  

The solution came through these two posts on the Proxmox forums:

{{< extlink url="https://forum.proxmox.com/threads/how-to-do-nomodeset-booting-up-7-2-installer.110307/post-483024" text="Part 1" >}},
{{< extlink url="https://forum.proxmox.com/threads/generic-solution-when-install-gets-framebuffer-mode-fails.111577/" text="Part 2" >}}.

If those posts ever disappear, the steps are as follows:

Change the boot options in Grub from `quiet splash = silent` to `nomodeset` to cause all boot messages to come through and to stop trying to load graphics drivers.  
Next your installer might exit with the message `Cannot run in framebuffer mode. Please specify busIDs for all framebuffer devices`.

{{< callout type="info" >}}
Note: From here 'X' refers to the X Window System, if future versions of Proxmox ship with Wayland, god help us all.
{{< /callout >}}

Hit `ctrl+alt+F2` to go to a separate console that should be display X errors and allow you to perform some sneaky maintenance.  
Use the command `lspci | grep -i vga` to list PCI devices with 'vga' in them.

It should list something like 
```
00:01.0 VGA compatible controller: Intel Corporation HD Graphics 530 (rev 06)
```

Time to add a description for this device to your X config.

Create a file in `/usr/share/X11/xorg.conf.d` called something ending in `.conf`. In my case I called it `driver-intel.conf`.

Add the following content:

```bash
Section "Device"
    Identifier "Card0"
    Driver "fbdev"
    BusID "pci0:01:0:0:"
EndSection
```

The BusID has to match with information you got from lspci, fbdev tells X we're dealing with a Frame Buffer device.

Then restart X with `xinit -- -dpi 96 >/dev/tty2 2>&1` and you should be golden.

## Clear skies

After this Proxmox worked brilliantly, I hope they do release a non-graphical installer one day. 
You administer Proxmox through a web interface so actually requiring the system running it to have graphics is quite dumb.

{{< lightbox img="img/proxmox_overview.webp" group="none" caption="yes" alt="Proxmox main screen" >}}



