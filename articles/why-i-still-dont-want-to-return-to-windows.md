---
title: "Why I Still Don't Want to Return to Windows"
description: "A personal and practical look at why Windows no longer feels like the default choice, especially when Linux and macOS cover most of my real work better."
pubDate: 2026-05-27
tags: ["Windows", "Linux", "macOS", "Operating Systems", "Desktop Linux", "Privacy", "Development", "Open Source"]
draft: false
---

I wrote the [first version of this article back in 2022](https://medium.com/@nomadic-dmitry/5-reasons-ill-never-return-back-to-windows-f46f19adb634).

At that time, I had not used Windows as my main operating system for more than 10 years. I still had occasional thoughts about installing it on a separate partition, mostly out of curiosity.

Years later, I still do not have a strong reason to return. The situation changed a bit, though.

Back then, the comparison in my head was mostly:

```text
Windows vs macOS
```

Now it is more like:

```text
Windows vs Linux vs macOS
```

And honestly, Windows looks weaker to me than before.

Not because it is unusable. It is usable.

But it no longer feels like the default operating system for people who want control over their own computer. And telemetry is out of control as well.

## 1. Windows Became Too Busy

The biggest problem with Windows is not one single bug or one bad feature.

It is the general direction. Windows feels busy.

Too many popups. Too many panels. Too many services. Too many background processes. Too many “suggestions”. Too many cloud hooks. Too many features I did not ask for.

The operating system should help me use the computer.

It should not constantly try to become a platform, assistant, store, ad surface, Microsoft account manager, cloud sync layer, widget board, news feed, gaming hub, and AI playground at the same time.

This is where Linux feels refreshing. On Linux, I can install a clean system and add what I need.

On Windows, I often feel like I am removing things first before I can even use it.

## 2. The Microsoft Store Never Became the Center of Anything

Microsoft tried to make the Store important. It never really became important to me.

I know it exists. I know some people use it. I know some apps are there.

But when I think about installing software on Windows, I still think about downloading `.exe` installers from random websites.

That is funny, because Windows is supposed to be the mainstream desktop system.

On Linux, software installation often feels more logical:

```bash
sudo apt install app
sudo dnf install app
flatpak install flathub app
```

Or you use a software center connected to repositories.

It is not perfect. Linux packaging can also be messy. There are deb packages, RPMs, Flatpak, AppImage, Snap, AUR, and more.

But at least the model makes sense to me.

The system package manager is part of the system.

On Windows, package management still feels like something added later because the old model was never fixed properly.

*Yes, there is winget now. That is good.*

But for me, it does not change the feeling that Windows still carries decades of old habits on top of new layers.

## 3. Windows 11 Solved Problems I Did Not Have

Most people were comfortable enough with Windows 7.

Then Windows 8 arrived and tried to reinvent the desktop for touch screens. It did not work well for normal desktop users.

Then Windows 10 tried to fix the mess.

Then Windows 11 changed things again.

Centered taskbar. New Start menu. New context menus. Different Settings app. More visual redesigns. More “modern” surfaces. Still old Control Panel parts underneath.

That is Windows in one sentence:

```text
New UI on top of old UI on top of older UI.
```

I do not hate change. I like change when it makes the system better.

But with Windows, I often feel like the UI changes because Microsoft needs the OS to look new, not because the workflow actually got better.

Linux is not immune to this either.

GNOME changes workflows. KDE has endless settings. Some distros make strange decisions.

But the difference is simple:

```text
On Linux, I can choose another desktop.
```

If I do not like GNOME, I can use KDE.

If KDE is too much, I can use XFCE.

If I want something traditional, I can use Cinnamon. On Windows, the UI is mostly what Microsoft gives you. And they keep removing the UI that was working perfectly well. That's what I call "the classic Windows experience"

You can patch it with third-party tools, but that already tells you the story.

## 4. Windows Hardware Requirements Became Annoying

Windows 11 also introduced stricter hardware requirements.

TPM 2.0. Secure Boot. Supported CPUs. Modern firmware expectations.

I understand the security argument. But from a user perspective, it also means many machines that are still perfectly usable became “unsupported” for no good practical reason.

This is where Linux wins hard. An old laptop that Windows considers outdated can still run Linux really well.

Maybe not with GNOME on full animations, but with XFCE, LXQt, Cinnamon, or a lightweight KDE setup, it can still be useful.

I like installing a modern system on a laptop that someone else would throw away.

Linux lets me do that. Windows never cared about that.

## 5. Linux Is Better Than It Used to Be

This is the biggest difference compared to 2022.

Linux desktop got better. Not perfect. But Better.

Gaming improved thanks to Steam and Proton.

Flatpak made desktop apps easier to install across distros.

Wayland is finally becoming normal.

PipeWire improved audio and screen sharing.

KDE Plasma became more polished.

GNOME became more consistent.

Fedora, Linux Mint, Ubuntu, Pop!_OS, Debian, openSUSE, MX Linux — there are many serious options now.

You can pick what fits your machine instead of trying to force one system everywhere.

For my kind of usage, Linux covers a lot:

- writing;
- browsing;
- coding;
- testing;
- Android tooling;
- Docker;
- servers;
- scripting;
- video tools;
- retro computing;
- privacy-focused workflows.

And if something breaks, at least I can usually understand what is happening.

Windows often hides things behind layers of “something went wrong”.

## 6. macOS Is Still Good, But Not the Same

I used to think macOS was the obvious better alternative to Windows. In many ways, it still is.

The UI is cleaner. The hardware integration is excellent. The terminal experience is better. The Unix-like base is useful. For many developers, macOS still feels comfortable.

But I am less romantic about macOS now.

Apple Silicon is impressive, but Apple also made the Mac feel more like a controlled appliance.

Linux is not as polished. It is not as smooth. It will not always hold your hand.

But it feels like a real computer.

That is why I no longer see this as “Mac is better than Windows”.

For me, it is more like:

```text
macOS is polished.
Linux is flexible.
Windows is noisy.and doesn't care about me
```

And I do not want noisy.

## 7. WSL Is Useful, But It Does Not Replace Linux

Windows Subsystem for Linux is a good tool.

I am not going to pretend otherwise.

If you are stuck on Windows and need Linux tools, WSL can be useful. It gives you a Linux environment without dual booting or running a full VM manually.

But for me, WSL always felt like a workaround.

It is Linux inside Windows.

I prefer Linux as the actual operating system.

When I use Linux directly, the terminal, filesystem, package manager, permissions, services, Docker setup, SSH workflow, scripts, and desktop all belong to the same world.

With WSL, I always feel the border that it creates. It is impressive technology.

But it is still not the same as using Linux.

## 8. Windows Still Has Its Place

To be fair, Windows is not useless.

It is still the best choice for some people.

If you need specific Windows-only software, use Windows.

If you play games with aggressive anti-cheat systems, Windows may still be the safest option.

If your company requires Windows, use Windows.

If you rely on Adobe, some CAD software, certain audio tools, or enterprise apps, Windows can be the practical choice.

I am not against using the right tool. I am against pretending Windows is still the obvious default for everyone.

For many people, Linux is good enough.

For many developers, Linux is better.

For many casual users, a good Linux Mint install is less annoying than Windows 11.

For people who like polished hardware and do not mind Apple’s restrictions, macOS is still great.

Windows is just one option now.

Not the center of the universe.

## 9. My Current Setup Preference

If I had to choose today, my personal order would be:

```text
Linux first.
macOS second.
Windows only if necessary.
```

Linux gives me the most control.

macOS gives me polish when I want a clean commercial system.

Windows gives me compatibility when there is no alternative.

## Conclusion

Linux became good enough for real daily use, and Windows became annoying enough that I do not miss it.

That is just my experience.

If Windows works for you, use it.

If macOS works for you, use it.

But if you are tired of being managed by your operating system, try Linux.
