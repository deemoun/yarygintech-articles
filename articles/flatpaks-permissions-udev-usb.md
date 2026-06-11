---
title: "Flatpaks, Permissions, and USB Devices"
description: "A practical Linux guide to Flatpak sandbox permissions, Flatseal, and udev rules for USB devices when the sandbox is not the real problem."
pubDate: "2026-06-11"
tags: ["linux", "flatpak", "flatseal", "udev", "usb", "security", "desktop-linux"]
draft: false
---
Flatpak is one of those Linux things that sounds simple until it is not.

You install an app from Flathub. It runs. Nice.

Then you try to open a folder outside your home directory. Or connect a USB device. Or use a serial adapter. Or make some hardware tool see your phone, Arduino, SDR dongle, printer, scanner, MIDI keyboard, whatever.

And suddenly the app acts like the device does not exist.

The usual first reaction is:

```bash
sudo something something permissions
```

Or worse:

```bash
flatpak override --user --device=all com.some.App
```

Sometimes that works. Sometimes it does nothing. Sometimes the problem was not Flatpak at all.

This article is about the practical split:

- Flatpak permissions control what the sandboxed app can see.
- udev rules control what your normal Linux user is allowed to access on the host.
- If the host user cannot access the device, the Flatpak app probably cannot magically access it either.

That is the part people miss.

## The mental model

Flatpak apps run in a sandbox. Not a perfect magic security bubble, but still a sandbox.

By default, the app does not automatically get full access to your home folder, host system, hardware devices, session bus, system bus, and every random socket on your machine.

That is the point.

A normal Linux package usually runs as your user and sees almost everything your user can see. A Flatpak app sees a filtered view.

So when something does not work, ask two questions:

| Question | What it means |
|---|---|
| Can my normal Linux user access this thing outside Flatpak? | Host permission problem. Think groups, udev, `/dev`, `lsusb`, `dmesg`. |
| Can the Flatpak app see this thing inside the sandbox? | Flatpak permission problem. Think Flatseal, `flatpak override`, portals. |

Do not mix them together immediately. Debug one layer at a time.

## Portals: the nice path

Flatpak wants apps to use portals when possible.

A portal is basically a controlled bridge between the sandbox and the host. The classic example is the file picker. The app does not need your entire home folder forever. You pick one file, and the portal gives the app access to that file.

That is why some Flatpak apps can open a file outside their sandbox even when you did not manually give them full filesystem access.

This is good design.

The problem is that not everything has a nice portal path. USB devices, weird hardware, serial tools, old apps, dev tools, emulators, Android tools, SDR apps, flashing utilities — these often need more direct access.

That is where permissions and udev rules come in.

## Useful Flatpak commands

First, list installed apps:

```bash
flatpak list --app
```

Find the app ID. It looks like this:

```text
org.mozilla.firefox
com.github.tchx84.Flatseal
com.valvesoftware.Steam
org.gnome.Boxes
```

Show app permissions:

```bash
flatpak info --show-permissions com.some.App
```

Show your overrides:

```bash
flatpak override --show com.some.App
```

Reset overrides for one app:

```bash
flatpak override --user --reset com.some.App
```

Run a shell inside the app sandbox:

```bash
flatpak run --command=sh com.some.App
```

Then look around:

```bash
ls /dev
ls /run/host
ls ~/.var/app
```

This is useful because you stop guessing. You see what the app actually sees.

## Flatseal: the practical GUI

You can manage Flatpak permissions from the terminal, but Flatseal is usually faster.

Install it:

```bash
flatpak install flathub com.github.tchx84.Flatseal
```

Run it:

```bash
flatpak run com.github.tchx84.Flatseal
```

Flatseal lets you click an app and change permissions: filesystem access, devices, sockets, environment variables, Wayland/X11, network, and so on.

This is the tool I would recommend for normal desktop use.

Terminal is better for scripts and exact notes. Flatseal is better when you are experimenting.

## Filesystem permissions

A common problem: the app cannot see a folder.

Example: you have media on another drive:

```text
/mnt/storage/Videos
```

Give one app access:

```bash
flatpak override --user --filesystem=/mnt/storage/Videos com.some.App
```

Read-only access:

```bash
flatpak override --user --filesystem=/mnt/storage/Videos:ro com.some.App
```

Give access to Downloads:

```bash
flatpak override --user --filesystem=xdg-download com.some.App
```

Take it away:

```bash
flatpak override --user --nofilesystem=/mnt/storage/Videos com.some.App
```

Or reset all overrides for that app:

```bash
flatpak override --user --reset com.some.App
```

Do not give `--filesystem=host` just because one folder is missing.

That is the Linux desktop equivalent of removing your front door because you lost one key.

## Device permissions in Flatpak

This is where things get messy.

Some common device permissions:

| Permission | What it is usually for |
|---|---|
| `--device=dri` | GPU/OpenGL acceleration. Very common. |
| `--device=input` | Input devices. Depends on Flatpak version/support. |
| `--device=usb` | USB device access. Newer and more specific than `all`. |
| `--device=all` | Broad device access. Works around many issues, but weakens the sandbox a lot. |

Example:

```bash
flatpak override --user --device=usb com.some.App
```

If that does not work, check your Flatpak version:

```bash
flatpak --version
```

On older systems, newer permissions may not be supported. Some guides still use:

```bash
flatpak override --user --device=all com.some.App
```

That is the big hammer.

It may be acceptable for a trusted hardware utility, emulator, or dev tool. But do not pretend it is clean sandboxing. You are giving the app broad access to host devices.

Use the smallest permission that actually fixes the problem.

## When Flatpak permission is not enough

Here is the important part.

Imagine you installed a Flatpak app that talks to a USB serial adapter.

You give it USB/device access.

Still broken.

Why?

Because the device node on the host may be owned by `root`, or by a group your user is not in.

Example serial device:

```bash
ls -l /dev/ttyUSB0
```

You might see something like:

```text
crw-rw---- 1 root dialout 188, 0 Jun 11 12:00 /dev/ttyUSB0
```

That means only `root` and users in the `dialout` group can access it.

Check your groups:

```bash
groups
```

Add yourself to the group:

```bash
sudo usermod -aG dialout $USER
```

Then log out and log back in.

Not close the terminal. Not open a new tab. Actually log out and back in.

For Arch-based distros, the group may be `uucp` instead:

```bash
sudo usermod -aG uucp $USER
```

For some devices, group membership is not enough. That is when you write a udev rule.

## What udev does

udev is the Linux device manager in userspace.

When you plug in a device, the kernel detects it, and udev applies rules. Those rules can set permissions, create stable symlinks, assign groups, tag devices for desktop access, and run actions.

Local custom rules usually go here:

```text
/etc/udev/rules.d/
```

Rule files end with:

```text
.rules
```

Example:

```text
/etc/udev/rules.d/99-my-usb-device.rules
```

## Finding your USB device IDs

Plug in the device.

Run:

```bash
lsusb
```

Example output:

```text
Bus 001 Device 008: ID 1a86:7523 QinHeng Electronics CH340 serial converter
```

Here:

```text
idVendor  = 1a86
idProduct = 7523
```

You can also monitor udev events live:

```bash
udevadm monitor --environment --udev
```

Then unplug and plug the device back in.

For a specific device node:

```bash
udevadm info -a -n /dev/ttyUSB0
```

For HID devices:

```bash
ls -l /dev/hidraw*
udevadm info -a -n /dev/hidraw0
```

Yes, the output is ugly. Welcome to Linux hardware debugging.

## A basic udev rule for a serial USB device

For a CH340 serial adapter:

```bash
sudo nano /etc/udev/rules.d/99-ch340.rules
```

Add:

```udev
SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE="0660", GROUP="dialout", TAG+="uaccess", SYMLINK+="ch340"
```

Reload rules:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Unplug and plug the device back in.

Check:

```bash
ls -l /dev/ttyUSB0
ls -l /dev/ch340
```

The symlink is optional, but it is nice. Instead of guessing whether the device is `/dev/ttyUSB0` or `/dev/ttyUSB1`, you can use:

```text
/dev/ch340
```

## A udev rule for hidraw devices

Some USB devices show up as `hidraw`, not serial.

Example rule:

```udev
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1234", ATTRS{idProduct}=="abcd", MODE="0660", TAG+="uaccess"
```

Replace the vendor and product IDs with your real ones.

Do not blindly copy random IDs from Stack Overflow and then wonder why nothing works.

## `TAG+="uaccess"` vs `MODE="0666"`

You will see a lot of old rules like this:

```udev
MODE="0666"
```

That means everyone on the system can read/write the device.

On a single-user laptop, it may not be the end of the world. But it is still sloppy.

A better desktop-style rule often uses:

```udev
TAG+="uaccess"
```

That tells systemd/logind to give the active local user access to the device.

For serial devices, using a proper group like `dialout` is also normal:

```udev
GROUP="dialout", MODE="0660"
```

My usual order:

1. Try the distro group first (`dialout`, `uucp`, etc.).
2. Use `TAG+="uaccess"` for desktop hotplug access.
3. Avoid `MODE="0666"` unless you understand the tradeoff and do not care.

## Debug flow for USB + Flatpak

Use this order.

Do not randomly change five things at once.

### 1. Check the device on the host

```bash
lsusb
```

If the device does not appear there, Flatpak is not your problem.

Check cable, hub, power, kernel support, `dmesg`:

```bash
sudo dmesg -w
```

Plug the device in and watch the logs.

### 2. Check the device node

For serial:

```bash
ls -l /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
```

For HID:

```bash
ls -l /dev/hidraw* 2>/dev/null
```

For generic USB bus nodes:

```bash
ls -l /dev/bus/usb/*/*
```

### 3. Check host access

Try the same operation outside Flatpak if possible.

For serial:

```bash
screen /dev/ttyUSB0 115200
```

Or:

```bash
python3 -m serial.tools.list_ports
```

If the host user cannot access it, fix that first.

### 4. Add group or udev rule

For serial:

```bash
sudo usermod -aG dialout $USER
```

Or write a custom udev rule if the device needs one.

### 5. Give the Flatpak app the minimum useful permission

Try USB access:

```bash
flatpak override --user --device=usb com.some.App
```

If your Flatpak version/app still needs the broad hammer:

```bash
flatpak override --user --device=all com.some.App
```

Then restart the app.

### 6. Check inside the sandbox

```bash
flatpak run --command=sh com.some.App
```

Inside:

```bash
ls /dev
ls /dev/bus/usb
```

If it is not visible there, the app cannot use it.

## Common fixes table

| Problem | Likely fix |
|---|---|
| App cannot open a folder on another drive | Add `--filesystem=/path` or use Flatseal. |
| App can open selected files but not scan a folder | Portal access is temporary/specific; add filesystem permission. |
| USB device appears in `lsusb`, but app does not see it | Add Flatpak device permission. |
| USB serial device exists as `/dev/ttyUSB0`, but permission denied | Add user to `dialout`/`uucp` or create udev rule. |
| HID device exists as `/dev/hidrawX`, but permission denied | Add udev rule with `TAG+="uaccess"`. |
| Device works as root but not as user | Host permission problem, not only Flatpak. |
| Device works outside Flatpak but not inside | Flatpak sandbox permission problem. |
| Device path keeps changing | Add udev `SYMLINK+=` rule. |

## Example: making a Flatpak app talk to a USB serial adapter

Let’s say you have a CH340 adapter.

Find it:

```bash
lsusb
```

Output:

```text
ID 1a86:7523 QinHeng Electronics CH340 serial converter
```

Check device:

```bash
ls -l /dev/ttyUSB0
```

Add yourself to `dialout`:

```bash
sudo usermod -aG dialout $USER
```

Log out and back in.

Create a stable udev rule:

```bash
sudo nano /etc/udev/rules.d/99-ch340.rules
```

Add:

```udev
SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE="0660", GROUP="dialout", TAG+="uaccess", SYMLINK+="ch340"
```

Reload:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Replug the device.

Give the Flatpak app USB/device access:

```bash
flatpak override --user --device=usb com.some.App
```

If that fails because your setup is older or the app needs broader access:

```bash
flatpak override --user --device=all com.some.App
```

Now point the app to:

```text
/dev/ch340
```

or:

```text
/dev/ttyUSB0
```

## Do not use `sudo flatpak run`

This is almost always the wrong fix:

```bash
sudo flatpak run com.some.App
```

You are changing the user, environment, permissions, config paths, and sandbox behavior all at once.

It may appear to fix something, but it teaches you nothing. Worse, it can create root-owned config files in places that later break normal app usage.

Fix the real permissions.

## Security reality check

Flatpak is useful, but permissions matter.

If you give an app:

```bash
--filesystem=host
--device=all
--socket=session-bus
--socket=system-bus
```

then do not brag about sandboxing.

You basically turned the Flatpak app into a very complicated normal package with extra steps.

Sometimes you need broad access. Development tools, emulators, hardware tools, flashing tools, and virtualization apps can be ugly like that.

But make it a conscious decision.

Give broad access only to apps you trust and only when narrower permissions do not work.

## My practical rule

For normal desktop apps:

- Prefer portals.
- Use Flatseal for small tweaks.
- Give specific folder access instead of `host`.
- Avoid broad device access.

For hardware/dev tools:

- First make the device work on the host.
- Fix Linux permissions with groups or udev.
- Then give the Flatpak app the minimum device access it needs.
- Use `--device=all` only when you actually need it.

Flatpak is not the enemy here.

The real problem is that Linux hardware access is split across layers: kernel, udev, user groups, desktop session, portals, and Flatpak sandbox permissions.

Once you separate those layers, debugging becomes much less stupid.