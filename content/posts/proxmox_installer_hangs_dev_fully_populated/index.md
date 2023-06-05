---
title: "Proxmox installer hangs on 'Waiting for /dev to be fully populated'."
slug: "proxmox-installer-hangs-dev-fully-populated"
date: 2023-06-05
draft: false
weight: 0
tags: []
summary: ""
cover:
    image: "img/header.jpeg"
    alt: ""
    caption: ""
    relative: true
---

## Nothing ever comes easy, does it?

When I installed my home server the system went through its first POST correctly after a short period of self-discovery (at least I think that's what rebooting thrice means). And Virtualization support was enabled out of the box.  
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
```bash
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


{{< callout icon="fa-solid fa-heart fa-3x" bg_colour="#faf4ed" >}}
**Thanks for reading!** If you have any questions or feedback feel free to contact me through the means listed [on my main site](https://dylanmaassen.nl). Sharing my posts is also really appreciated!
{{< /callout >}}