---
title: "How to Make Your Mac Feel Faster: Free Cleanup Tools and Habits"
description: "A practical Mac cleanup guide based on built-in macOS tools, manual folder checks, AppCleaner, OnyX, Homebrew cleanup, and Xcode storage maintenance."
pubDate: 2026-05-27
tags: ["Mac", "macOS", "Cleanup", "AppCleaner", "OnyX", "Homebrew", "Xcode", "Storage", "Maintenance"]
draft: false
---

**A friend once asked me if it was worth updating macOS to the latest version.**

The question itself made me think.

*In the long run, updates are usually necessary. Not because every update is exciting, but because security updates, browser support, app compatibility, and developer tools eventually move forward.*

The only macOS version I personally avoided updating for a long time was Mojave. It was the last version with 32-bit app support, and I used it for years because some older apps still mattered to me.

But eventually, even Mojave became too old.

The bigger reason people avoid macOS updates is usually simple:

```text
Not enough free space.
```

The update needs space. Xcode needs space. iPhone backups need space. Caches grow. Downloads pile up. Old apps leave files behind.

And then the Mac starts feeling slow.

This article is not about fake “one-click speed booster” nonsense. It is about simple cleanup habits and a few free tools that actually help.

## Start With Built-in macOS Storage Tools

Before installing anything, check what macOS already shows you.

On modern macOS versions:

```text
System Settings -> General -> Storage
```

Apple has a short official guide here:  
https://support.apple.com/102624

This view helps you understand what is taking space:

- Applications
- Documents
- Photos
- iOS backups
- macOS system data
- Developer tools
- Trash

Do not blindly delete everything. Just use this screen to understand where the problem is.

If the Mac has only a few gigabytes free, it can feel slow. macOS needs free space for updates, swap, caches, app data, and temporary files.

My personal rule is simple:

```text
Keep at least 20-30% of the internal drive free if possible.
```

On a 256 GB Mac, that means you should not live permanently with 5 GB free.

## Clean Up Manually First

Before using third-party tools, check the obvious folders.

### Downloads

```text
~/Downloads
```

This is the first place I always check.

Installers, ZIP archives, exported videos, random screenshots, PDFs, disk images — everything ends up here.

Sort by size and delete what you no longer need.

### Documents

```text
~/Documents
```

Some apps store data here. Sometimes old projects, exports, and temporary folders stay there forever.

Do not delete blindly, but check it.

### Movies

```text
~/Movies
```

This folder can become huge if you use iMovie, Final Cut, screen recording, or video export tools.

If you use iMovie, the library can take a lot of space:

```text
iMovie Library.imovielibrary
```

You can move it to an external drive if needed.

### Pictures

```text
~/Pictures
```

This is where the Photos library usually lives.

If you use Apple Photos, check how large the library is. If it is massive, consider moving it to external storage or using iCloud Photos carefully.

### Applications

```text
/Applications
```

This is where most installed apps live.

But do not just drag every app to Trash and call it clean. Many apps leave support files, preferences, caches, and helper files behind.

For removing apps properly, I prefer AppCleaner.

## Remove Apps With AppCleaner

AppCleaner is one of those small Mac utilities that I still like.

AppCleaner:  
https://freemacsoft.net/appcleaner/

The idea is simple:

```text
Drag app into AppCleaner -> review related files -> remove
```

It finds related files like:

- *preferences;*
- *caches;*
- *application support files;*
- *launch agents;*
- *saved state files.*

This is better than just deleting the `.app` bundle from `/Applications`.

Of course, still be careful. If you are deleting a complex app that contains your projects or local data, check what AppCleaner wants to remove before confirming.

For normal apps, it works great.

## Use OnyX for Maintenance

OnyX is another free Mac utility I like.

OnyX:  
https://www.titanium-software.fr/en/onyx.html

It can run maintenance tasks, clean caches, rebuild indexes, and access some hidden macOS settings.

But here is the important part:

```text
Do not treat OnyX like magic.
```

It will not turn an old Mac into a new one. It will not fix weak hardware. It will not solve every slowdown.

But it can help when:

- Spotlight search behaves weirdly;
- caches are huge or broken;
- the system feels sluggish after months of use;
- you want to run maintenance tasks from one place.

Before using tools like this, make sure you have a backup.

Use Time Machine or at least copy important files somewhere safe.

Time Machine guide:  
https://support.apple.com/104984

In most cases, OnyX works fine. But maintenance tools still modify system-related data, so do not click every option randomly.

## Check iPhone and iPad Backups

If you back up iPhone or iPad to your Mac, the backups can become huge.

The folder is usually here:

```text
~/Library/Application Support/MobileSync/Backup
```

Do not manually delete random files inside if you are not sure.

The better way is to manage device backups through Finder:

```text
Connect device -> Finder -> Manage Backups
```

Old iPhone backups can easily take tens of gigabytes.

If you no longer need them, delete them properly.

## Check Homebrew

If you are a developer or technical user, Homebrew can also take space.

On Apple Silicon Macs, Homebrew usually lives here:

```text
/opt/homebrew
```

On Intel Macs, it is usually here:

```text
/usr/local
```

Basic commands:

```bash
brew update
brew upgrade
brew cleanup
```

Homebrew documentation:  
https://docs.brew.sh/Manpage

Homebrew also performs some cleanup automatically, but running `brew cleanup` manually can still remove old downloads and outdated versions.

To see installed packages:

```bash
brew list
```

To remove something:

```bash
brew uninstall package_name
```

Be careful with dependencies. If you do not know what a package does, search first.

## Xcode Is a Monster (No,seriously!)

If you have Xcode installed, check it right now.

Xcode can take a ridiculous amount of space.

The app itself lives here:

```text
/Applications/Xcode.app
```

But the real problem is often not only the app. It is also:

- old iOS simulators;
- DerivedData;
- device support files;
- archives;
- caches.

Useful folders to check:

```text
~/Library/Developer/Xcode/DerivedData
~/Library/Developer/Xcode/Archives
~/Library/Developer/Xcode/iOS DeviceSupport
```

If you work with iOS development or automation, these folders can grow fast.

You can delete `DerivedData` safely in most cases. Xcode will recreate it when needed.

```bash
rm -rf ~/Library/Developer/Xcode/DerivedData
```

Do not run commands like this if you are not sure what they do. The command above deletes the entire DerivedData folder.

For simulators, it is better to remove old runtimes from Xcode settings.

You can also check simulator data with:

```bash
xcrun simctl list
```

And delete unavailable simulators:

```bash
xcrun simctl delete unavailable
```

This can recover a lot of space if you test iOS apps.

## Check Large Files From Terminal

Sometimes the fastest way to find the problem is Terminal.

To check disk usage in your home folder:

```bash
du -sh ~/*
```

To sort folders by size:

```bash
du -sh ~/* 2>/dev/null | sort -h
```

To check free disk space:

```bash
df -h
```

These commands are simple and useful.

They do not delete anything. They just show what is taking space.

## Things I Would Not Do

There are a few things I would avoid.

Do not randomly delete files from:

```text
/System
/Library
/private
/usr
/bin
/sbin
```

Do not install five different “Mac cleaner” apps.

Do not pay for a subscription cleaner before checking the basics.

Do not delete caches every day. Caches exist for a reason. They can make apps faster. You clean them when they become too large or problematic.

Do not remove files from `~/Library` unless you understand what they belong to.

## My Simple Cleanup Routine

Here is what I would personally do:

1. Check macOS Storage settings.
2. Empty Downloads.
3. Remove large old videos and installers.
4. Delete apps I no longer use with AppCleaner.
5. Check iPhone/iPad backups.
6. Clean old Xcode data if I use Xcode.
7. Run `brew cleanup` if I use Homebrew.
8. Use OnyX occasionally, not every day. And use it carefully
9. Reboot the Mac after heavy cleanup.
10. Keep enough free space for macOS to breathe.

That is enough for most people.

## Conclusion

The best way to keep a Mac fast is boring:

```text
Do not fill the disk.
Do not install random junk.
Remove apps properly.
Keep macOS updated.
Have backups.
```

Free tools like AppCleaner and OnyX are useful, but they are not magic. They are just helpers.

Most of the real work is understanding where your space went and cleaning it carefully.

Your Mac is still one of your main daily tools. Treat it well, keep enough free storage, and do not wait until the update fails because the disk is completely full.