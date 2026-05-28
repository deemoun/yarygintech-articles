---
title: "Retro Gaming on Apple Silicon: What Works and What Doesn’t"
description: "A practical and personal look at retro gaming on Apple Silicon Macs using DOSBox, CrossOver, UTM, and a few realistic expectations."
pubDate: 2026-05-27
tags: ["Apple Silicon", "Mac", "Retro Gaming", "DOSBox", "CrossOver", "UTM", "macOS", "Emulation", "Windows Games", "DOS Games"]
draft: false
---

One of the things I missed after moving to Apple Silicon was the ability to run old x86 virtual machines easily.

On older Intel Macs, it was simple. You could use VirtualBox, VMware, Parallels, or Boot Camp and run old Windows systems directly. It was not always perfect, but it felt natural.

Apple Silicon changed that.

The hardware is fast, quiet, and efficient. But it is also ARM-based, which means old x86 software does not run natively. That includes many old Windows games, DOS tools, and random retro software from the 90s and early 2000s.

So if you want to play retro games on an Apple Silicon Mac, you need to think a bit differently.

It is still possible. Just do not expect it to work like an old x86 PC.

## The Main Problem

Apple Silicon Macs use ARM processors.

Most old Windows and DOS games were made for x86 processors.

That means you usually have three options:

- emulate the old machine;
- translate Windows API calls;
- use native or modern source ports when available.

Each option has its own trade-offs.

Some games work beautifully. Some need tweaking. Some are not worth the pain.

But for casual retro gaming, Apple Silicon Macs are actually good enough.

## DOS Games: Use DOSBox or DOSBox-X

The easiest place to start is DOS gaming.

For DOS games, I would use either DOSBox or DOSBox-X.

DOSBox is simple and classic:  
https://www.dosbox.com/

DOSBox-X is more advanced and flexible:  
https://dosbox-x.com/

If you just want to play old DOS games, DOSBox is usually enough.

If you want more control, better configuration options, Windows 3.x support, or a more complete retro PC environment, DOSBox-X is worth checking out.

Typical use case:

```text
Old DOS games
Windows 3.x games
Early 90s software
Classic shareware titles
```

This is probably the cleanest retro gaming experience on Apple Silicon.

You do not need to emulate a full modern Windows system. You just run the DOS environment and play.

It also feels right. DOS games were never about heavy 3D graphics anyway.

## Windows Games: CrossOver Is Often Better Than a VM

For old Windows games, I would try CrossOver first.

CrossOver:  
https://www.codeweavers.com/crossover

CrossOver is based on Wine. It does not emulate a full Windows machine. Instead, it translates Windows API calls into macOS-compatible calls.

That sounds boring, but in practice it can be very useful.

You can install Windows applications, Steam, GOG installers, and many older games without installing Windows itself.

This is important on Apple Silicon because running old x86 Windows inside a virtual machine can be slow and annoying. CrossOver can often be faster because it avoids emulating the whole operating system.

It is not magic, though.

Some games work great. Some need special settings. Some do not work at all.

CrossOver uses "bottles", which are separate environments for different apps or games. That is useful because one game may need a specific DirectX version, another may need fonts, and another may need different Windows compatibility settings.

For retro gaming, this can be enough:

```text
Install CrossOver
Create a bottle
Install Steam, GOG game, or standalone installer
Try the game
Adjust settings if needed
```

This is not as clean as running the game on a real Windows XP machine, but it is convenient.

And convenience matters.

## Steam on CrossOver

One useful trick is installing Steam inside CrossOver.

This gives you access to older Windows games from your Steam library.

It will not work with everything, but many older and lighter games are playable. The funny part is that the game may be going through several layers:

```text
Windows game -> Wine/CrossOver -> macOS -> Apple Silicon
```

And yet, in many cases, it still runs surprisingly well.

That is the part I like about Apple Silicon. The machines are fast enough that even weird compatibility layers can still feel usable.

## UTM: Good for Some Things, Not for Everything

UTM is another useful option.

UTM:  
https://mac.getutm.app/

UTM can run virtual machines and emulated systems on macOS. It is based on QEMU and gives you a nice UI around it.

For Apple Silicon, UTM is great when you run ARM operating systems. For example, ARM Linux or Windows on ARM can be quite usable.

But if you want to run old x86 Windows versions, that is emulation, not native virtualization.

That means slower performance.

From my experience, old Windows versions like Windows XP can run, but not always fast enough to feel great. Windows 7 and newer are usually not worth it for retro gaming this way.

Still, UTM has its place.

It is useful for:

```text
Experimenting with old systems
Running lightweight OS images
Trying Mac OS 9 / PowerPC setups
Learning how old environments worked
```

For actual gaming, I would not start with UTM unless the game or system specifically needs it.

## Mac OS 9 and PowerPC Stuff

One interesting use case for UTM is old Mac software.

If you want to play with Mac OS 9 or PowerPC-era software, UTM can be fun.

This is not something every retro gamer needs, but it is a cool option if you are into old Mac history.

Apple Silicon Macs are obviously not PowerPC machines, but with emulation you can still explore that era.

It is not perfect, but it is usable enough for experiments.

And honestly, seeing old Mac OS running on a modern Apple Silicon machine is just fun.

## Sinclair QL and Other Weird Retro Systems

Retro gaming is not only about DOS and Windows.

There are also smaller and stranger systems worth exploring.

For example, QPC2 is a Sinclair QL emulator:  
https://www.kilgus.net/qpc/

There are also other Sinclair QL emulators and resources:  
https://www.rwapsoftware.co.uk/emulators.html

This is where CrossOver can sometimes help too, especially when an old emulator only exists as a Windows program.

That is one of the good things about this setup. Even if Apple Silicon is not ideal for classic x86 virtualization, you can still combine different tools and make a lot of old software run.

## Native Ports Are Usually the Best Option

Before trying emulation, always check whether the game has a modern source port.

For example, many classic games have modern engines or ports:

```text
Doom source ports
Quake source ports
ScummVM for adventure games
OpenRA for classic RTS-style games
OpenTTD for Transport Tycoon-like gameplay
```

ScummVM:  
https://www.scummvm.org/

OpenRA:  
https://www.openra.net/

OpenTTD:  
https://www.openttd.org/

This is often better than trying to recreate an old Windows setup.

Native or modern ports usually give you:

- better resolution support;
- easier controls;
- fewer compatibility problems;
- better performance;
- less setup pain.

Yes, it is not always the same as playing on original hardware. But for casual gaming, it is often the best experience.

## What I Would Use First

If I had to build a simple retro gaming setup on an Apple Silicon Mac today, I would start like this:

### For DOS games

Use [DOSBox](https://www.dosbox.com) or [DOSBox-X](https://dosbox-x.com/).

This is the easiest and most reliable path.

### For old Windows games

Try CrossOver first. This is where I had the best experience overall.

### For old Mac systems

Try UTM.

Especially for Mac OS 9 or other experiments.

### For classic games with modern engines

Use native source ports.

This is often the cleanest way.

### For the full authentic experience

Buy an old Windows laptop or desktop.

**Seriously.**

## Apple Silicon Is Good, But Not Perfect for Retro Gaming

Apple Silicon Macs are not the first thing I would recommend for serious retro gaming.

They are powerful machines, but they are not old x86 computers. Some things are easy, some things are annoying, and some things are just not worth forcing.

If you want the full early-2000s experience, an actual old Windows PC is still better.

A real Windows XP-era laptop or desktop gives you:

- proper x86 compatibility;
- real old drivers;
- old DirectX behavior;
- fewer translation layers;
- more authentic weirdness.

And sometimes that weirdness is part of the fun.

But if you already have an Apple Silicon Mac, you can still do a lot.

DOSBox covers DOS games.

CrossOver covers many Windows games.

UTM is useful for experiments.

Native ports cover many classics.

That is enough for occasional retro gaming.

## Conclusion

Apple Silicon is not the perfect retro gaming platform, but it is good enough for fun.

You will not get the same freedom as on an old x86 PC, and you should not expect every Windows game from 1998 or 2003 to work perfectly.

Sometimes buying another machine (especially when old Thinkpads are that cheap) is a good option.

**Have fun playing your favorite Retro games.**