---
title: "Apple Silicon’s Dark Side: The Reality of a Closed Ecosystem and How To Get Around It"
description: "A personal story about installing Asahi Linux on an M1 Mac, breaking the boot setup, and restoring macOS through DFU mode using Linux instead of another Mac."
pubDate: 2026-05-27
tags: ["Apple Silicon", "Mac", "Asahi Linux", "Linux", "macOS", "DFU", "Apple", "M1", "Repair"]
draft: false
---

*For many years, the Mac was my go-to machine.*

It was the device I used for entertainment, coding, writing, experimenting, and doing the kind of computer stuff I actually enjoy. I was an Apple user for more than 15 years, and for a long time, the Mac felt like the perfect balance between a polished consumer product and a real computer.

**Then Apple Silicon happened.**

To be fair, Apple Silicon brought a lot of good things: great battery life, quiet machines, strong performance, and impressive efficiency. My MacBook Air with the M1 chip was a lovely laptop in many ways.

But at the same time, I started feeling that the Mac was becoming less of a computer I owned and more of a sealed appliance I was allowed to use.

And that feeling eventually pushed me to try something different.

## Why I Tried Asahi Linux

After about three years with my MacBook Air, I decided to install [Asahi Linux](https://asahilinux.org/).

The idea sounded great: Linux running natively on Apple Silicon hardware. Not inside a virtual machine. Not through some slow workaround. A real Linux system on an M1 Mac.

The reason was simple: I was getting tired of macOS limitations.

With every new macOS release, I felt like the system was becoming more locked down. More controlled. More opinionated. More focused on pushing Apple’s own vision of how I should use the machine.

I still liked the hardware, but I didn’t like the direction of the operating system as much anymore.

Even tools like Homebrew started feeling like a patch over a bigger problem. Yes, they make macOS more usable for developers, but at some point it feels like you are plugging holes in a system that does not really want to be that flexible anymore.

So I thought: why not try Linux?

## Installing Asahi Linux

Technically, installing Asahi Linux is not complicated.

You run a script:

```bash
curl https://alx.sh | sh
```

The installer shrinks your existing macOS volume, creates space for Linux, sets up the boot environment, installs the system, and reboots the machine.

In theory, it is simple.

In practice, you are still doing this on Apple hardware, where dual booting is not treated as a first-class feature. Apple does not give you the kind of boot customization that old PC users are used to. You are working around the system, not with it.

The installer lets you choose between GNOME, KDE, or a minimal CLI setup.

I picked GNOME first.

The installation failed.

Thankfully, it did not brick the system. I tried again with KDE, and this time the installation completed successfully.

And honestly, the first boot felt exciting.

There I was, running Linux natively on an M1 Mac. Not in a VM. Not through emulation. Real Linux on Apple Silicon. GPU acceleration was working. The system felt smooth. I could still dual boot back into macOS when needed.

For a moment, it felt like I had unlocked the machine.

But the experience was not as perfect as it looked at first.

## The Reality of Linux on Apple Silicon

Linux itself ran well.

Asahi Linux is an impressive project, and what the developers have achieved is genuinely great. The system was stable enough for normal use, KDE worked fine, and the machine felt fast.

But the problem was not the desktop environment.

**The problem was the ecosystem.**

A lot of proprietary Linux software still does not provide ARM builds. For example, I could not run Android Studio the way I needed. That was a big issue for me because Android development and testing are part of my normal workflow.

Some VPN clients were also problematic. You can sometimes compile things manually, search for workarounds, or try alternative packages, but that gets old quickly when you just need your tools to work.

Most open-source packages are available for ARM, but not everything is there. And even when something technically runs, it does not always mean the experience is smooth.

Then there is virtualization.

Forget about normal x86 virtualization. Apple Silicon is ARM, so ARM virtual machines are the realistic path. Anything x86-related either does not work the way you want or becomes painfully slow.

Hardware support also depends on the exact Apple Silicon model you own. Asahi Linux has made huge progress, but not every feature is supported on every Mac.

So yes, Linux on the M1 Mac worked.

But it did not replace macOS for me.

It felt more like a very impressive technical achievement than a practical daily-driver setup for my needs. I wanted a system that could handle everything I do. Asahi was close enough to be interesting, but not close enough to become my main OS.

Eventually, I decided to remove it.

That is where the real trouble started.

## How I Almost Bricked the Mac

After using Asahi Linux for a while, I decided to return the MacBook to a clean macOS-only setup.

I booted into macOS and opened Disk Utility, expecting to delete the Linux volumes.

That did not work.

Disk Utility showed the partitions, but it did not give me a clean way to remove everything. The layout was more complicated than I expected, and Apple’s own tools were not very helpful.

So I booted back into Linux and opened a partitioning tool there.

That was my mistake.

I removed one of the Asahi-related volumes, thinking I was only deleting the Linux side of the setup.

But that volume turned out to be important not only for booting Asahi Linux, but also for the overall Apple Silicon boot process. I had not erased macOS itself, but I had broken the path needed to load it.

The Mac no longer booted properly.

I held the Power button and managed to get into the system recovery environment. That was a relief, at least for a few minutes.

But Recovery did not really save me.

Disk Utility still refused to remove all the partitions. I tried using the command-line version of Disk Utility to wipe the disk properly, but some partitions remained. I was stuck in a weird broken state:

- I could not boot macOS.
- I could not fully erase the internal storage.
- I could not cleanly reinstall macOS from Recovery.
- I could not access the old-style Internet Recovery I was used to from Intel Macs.

Every key combination brought me back to a recovery environment that could not complete the job because the partition layout was broken.

At that point, the machine was not completely dead, but it was not usable either.

It was stuck.

## Apple Silicon Feels More Like an iPad Than an Old Mac

This was the moment when Apple Silicon really started to annoy me.

On old Intel Macs, I was used to a different kind of freedom. You could wipe the disk, boot from USB, install macOS from Internet Recovery, install Linux, install Windows through Boot Camp, replace things, break things, and usually recover without needing another Mac.

Apple Silicon is different.

It is technically powerful, but the recovery model feels much more like an iPhone or iPad. If something goes wrong deeply enough, you may need another Apple device to restore it properly.

That is not how I want a computer to behave.

A laptop should not require another laptop from the same company just to recover from a broken system state.

But complaining would not fix the Mac.

I needed DFU mode.

## What DFU Mode Does

DFU stands for Device Firmware Update.

It is a special recovery state used by Apple devices when normal recovery is not enough. In DFU mode, the device can receive a fresh firmware and operating system restore from another machine.

On Apple Silicon Macs, DFU mode is often the last-resort option when the machine cannot boot or reinstall macOS normally.

The official Apple-style path is simple in theory:

1. Take another Mac.
2. Install Apple Configurator.
3. Connect the broken Mac using a USB-C cable.
4. Put the broken Mac into DFU mode.
5. Restore it from the working Mac.

Nice.

Except I did not have another Mac.

I had a Windows machine.

And Apple does not provide a normal Windows tool for restoring a broken Apple Silicon Mac in this situation.

That part felt insane to me.

If Apple sells these machines as computers, there should be a straightforward way to restore them without owning a second Mac. But that is not how the ecosystem works.

So I started looking for another route.

## Restoring the Mac Using Linux

Eventually, I found a way to restore the Mac using Linux.

Since I had another computer, I booted Fedora Linux on it and connected it to the broken Mac with a USB-C cable.

The reason I used Fedora was simple: the DFU restore tools had convenient prebuilt packages there.

I installed the required tools:

```bash
sudo dnf install -y idevicerestore usbmuxd usbutils
```

Then I put the Mac into DFU mode and checked whether the Linux machine could see it:

```bash
lsusb
```

After that, I started `usbmuxd`:

```bash
usbmuxd -f &
```

`usbmuxd` is a daemon that proxies USB connections from Apple devices into TCP sockets. Tools like `idevicerestore` can then communicate with the device through it.

Finally, I restored macOS using an IPSW image:

```bash
sudo idevicerestore /path/to/UniversalMac_*.ipsw
```

And it worked.

The Mac came back to life.

The only catch was that, because of tooling limitations at the time, I could only flash a specific macOS version this way. After the restore finished, I updated macOS normally through the system updater.

But the important part was done: the machine was alive again.

## What I Learned

I was happy to see the Mac boot again, but the whole experience changed how I looked at Apple Silicon Macs.

Asahi Linux itself was not the villain here. It is an impressive project, and I respect the amount of reverse engineering and engineering work behind it.

The real issue is the Apple Silicon recovery model and the lack of flexibility.

On Intel Macs, I felt like I owned the machine more directly. On Apple Silicon, I felt like I was borrowing access to a highly locked-down device.

Yes, the hardware is excellent.

Yes, the performance-per-watt is great.

Yes, macOS is polished.

But once you step outside Apple’s expected path, things can get ugly fast.

And if the machine reaches a broken state, Apple expects you to have another Mac available to fix it.

That is a very Apple solution.

## Final Thoughts

After this experience, I stopped experimenting with Asahi Linux on that machine.

It was not because Asahi was bad. It simply did not fit my workflow well enough, and the risk of breaking the Apple Silicon boot setup was not worth it for me.

I wanted the M1 Mac to become a flexible Swiss Army knife: macOS when I needed polish, Linux when I wanted control, and maybe some virtualization for everything else.

But that is not really what Apple Silicon Macs are.

They are great machines if you stay inside the Apple-approved lane. The moment you want the kind of freedom that older Intel Macs had, the limitations become obvious.

Eventually, I sold that Mac and returned to a PC.

After more than 15 years as an Apple fan, that felt strange.

But maybe that was the real lesson: Apple Silicon is technically brilliant, but brilliance does not automatically mean freedom.

It was a nice ride.
