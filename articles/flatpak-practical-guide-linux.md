---
title: "Flatpak on Linux: The Practical Guide"
description: "How to install, search, update, remove, and manage Flatpak apps on Linux without turning your system into a mess."
pubDate: 2026-06-11
tags: ["linux", "flatpak", "flathub", "desktop-linux", "apps", "permissions"]
draft: false
---
Flatpak is one of those Linux things that sounds boring until you actually need it.

You install some distro. The repo has an old version of an app. The `.deb` package is broken. The AppImage wants random libraries. The official website gives you some weird tarball. And then you remember: okay, maybe Flatpak exists for a reason.

**Flatpak is basically a way to install desktop apps on Linux in a more universal way. It does not replace your package manager completely. I would not install my kernel, drivers, shell tools, or system services through Flatpak.**

But for desktop apps? Browsers, messengers, media tools, editors, emulators, note apps, random GUI utilities? Flatpak is often the cleanest option.

This article is the practical version. How to install it, how to search, how to update, how to remove apps properly, and what small tricks are actually useful.

---

## Flatpak vs your normal package manager

On Debian/Ubuntu/MX Linux, you probably use:

```bash
sudo apt install appname
```

On Fedora:

```bash
sudo dnf install appname
```

On Arch:

```bash
sudo pacman -S appname
```

Flatpak is different. It installs apps from Flatpak repositories, usually Flathub.

The main idea:

| Thing | Native package | Flatpak |
|---|---|---|
| Comes from | Your distro repo | Flathub or another Flatpak remote |
| App version | Often older on stable distros | Often newer |
| Dependencies | Shared with the system | Bundled through runtimes |
| Permissions | Normal Linux app access | Sandboxed-ish, configurable |
| Best for | System tools, drivers, CLI tools | Desktop GUI apps |
| Worst for | New desktop apps on old distros | Deep system integration |

Flatpak is not magic security dust. It is not a VM. It is not Android-level isolation. But it does give apps a more controlled environment, and that alone is useful.

---

## Install Flatpak

Most distros already have Flatpak in their repo.

### Debian, Ubuntu, MX Linux, Linux Mint

```bash
sudo apt update
sudo apt install flatpak
```

On GNOME-based systems, you may also want the software center plugin:

```bash
sudo apt install gnome-software-plugin-flatpak
```

On KDE:

```bash
sudo apt install plasma-discover-backend-flatpak
```

### Fedora

Fedora usually has Flatpak already installed:

```bash
flatpak --version
```

If not:

```bash
sudo dnf install flatpak
```

### Arch / EndeavourOS / Manjaro

```bash
sudo pacman -S flatpak
```

### openSUSE

```bash
sudo zypper install flatpak
```

After installing Flatpak, reboot or log out/in. Not always required, but it avoids stupid PATH/session weirdness.

---

## Add Flathub

Flatpak needs a repository. The main one is Flathub.

Add it system-wide:

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Check that it worked:

```bash
flatpak remotes
```

You should see something like:

```text
flathub system
```

If you see no Flathub, you did not add the remote correctly.

---

## System vs user installs

This part confuses people.

Flatpak can install apps system-wide or only for your user.

System install:

```bash
flatpak install flathub org.mozilla.firefox
```

User-only install:

```bash
flatpak install --user flathub org.mozilla.firefox
```

System install usually needs admin privileges. User install does not.

My simple recommendation:

| Situation | Use |
|---|---|
| Personal laptop, one user | Either is fine |
| Shared machine | System install |
| No sudo access | `--user` |
| You want everything clean per-user | `--user` |
| You do not care | Default system install |

The annoying bit: if you add Flathub system-wide, then try to install with `--user`, Flatpak may complain because your user installation does not have the Flathub remote.

To add Flathub only for your user:

```bash
flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Check user remotes:

```bash
flatpak remotes --user
```

Check system remotes:

```bash
flatpak remotes --system
```

---

## Search for apps

The easiest way is the Flathub website:

```text
https://flathub.org
```

But from terminal:

```bash
flatpak search firefox
```

Example:

```bash
flatpak search telegram
```

Flatpak app IDs look like this:

```text
org.mozilla.firefox
org.telegram.desktop
com.spotify.Client
org.gimp.GIMP
```

Yes, they look like Android package names. Get used to it.

---

## Install apps

Basic format:

```bash
flatpak install flathub APP_ID
```

Example:

```bash
flatpak install flathub org.mozilla.firefox
flatpak install flathub org.gimp.GIMP
flatpak install flathub org.telegram.desktop
```

Flatpak may ask you to confirm runtimes. That is normal.

A runtime is a shared base environment used by apps. For example, GNOME runtime, KDE runtime, Freedesktop runtime, etc.

At first Flatpak may look huge because it downloads runtimes. After that, multiple apps reuse them.

---

## Run Flatpak apps

Normally they appear in your app menu.

From terminal:

```bash
flatpak run org.mozilla.firefox
```

If you do not know the app ID:

```bash
flatpak list --app
```

Run with verbose output when debugging:

```bash
flatpak run -v org.mozilla.firefox
```

---

## List installed Flatpaks

List apps:

```bash
flatpak list --app
```

List runtimes:

```bash
flatpak list --runtime
```

List everything:

```bash
flatpak list
```

Show more columns:

```bash
flatpak list --columns=name,application,version,branch,installation
```

This is useful when you have both system and user installs.

---

## Update Flatpak apps

Update everything:

```bash
flatpak update
```

Update one app:

```bash
flatpak update org.mozilla.firefox
```

Check what would update:

```bash
flatpak update --assumeno
```

Flatpak updates are usually safe. But yes, they can be large. If you are on mobile data, do not blindly run updates.

---

## Remove apps

Remove one app:

```bash
flatpak uninstall org.mozilla.firefox
```

Remove unused runtimes:

```bash
flatpak uninstall --unused
```

This command is important. Flatpak apps often leave runtimes behind after removal. Not because Flatpak is evil, but because other apps might still need them.

So after removing a bunch of apps, run:

```bash
flatpak uninstall --unused
```

Then check disk usage:

```bash
du -sh ~/.var/app 2>/dev/null
du -sh /var/lib/flatpak 2>/dev/null
```

User Flatpaks usually live around:

```text
~/.local/share/flatpak
~/.var/app
```

System Flatpaks usually live around:

```text
/var/lib/flatpak
```

---

## See app info

```bash
flatpak info org.mozilla.firefox
```

Show permissions:

```bash
flatpak info --show-permissions org.mozilla.firefox
```

This is one of the best Flatpak commands because you can see what the app can actually access.

Example things you may see:

```text
[Context]
network=true
sockets=x11;wayland;pulseaudio;
devices=dri;
filesystems=xdg-download;
```

Translation:

| Permission | Meaning |
|---|---|
| `network=true` | App can use the network |
| `sockets=wayland` | App can talk to Wayland |
| `sockets=x11` | App can talk to X11 |
| `devices=dri` | GPU acceleration |
| `filesystems=home` | Access to home folder |
| `filesystems=xdg-download` | Access to Downloads |
| `filesystem=host` | Basically broad host file access |

When you see `filesystem=host`, pay attention. Sometimes it is needed. Sometimes it is lazy packaging.

---

## Change permissions with Flatseal

Install Flatseal:

```bash
flatpak install flathub com.github.tchx84.Flatseal
```

Run it:

```bash
flatpak run com.github.tchx84.Flatseal
```

Flatseal is the friendly GUI for Flatpak permissions.

Use it when an app needs access to:

- Downloads
- Documents
- external drives
- camera
- microphone
- USB devices
- Wayland/X11
- GPU acceleration

My rule: do not give every app full home access just because one file picker annoyed you once.

Give the smallest permission that solves the problem.

Bad lazy fix:

```text
filesystem=home
```

Better fix:

```text
xdg-download
xdg-documents
/path/to/specific/folder
```

---

## Change permissions from terminal

Give an app access to Downloads:

```bash
flatpak override --user --filesystem=xdg-download org.example.App
```

Give access to a specific folder:

```bash
flatpak override --user --filesystem=/home/$USER/Projects org.example.App
```

Remove a filesystem override:

```bash
flatpak override --user --nofilesystem=/home/$USER/Projects org.example.App
```

Reset all overrides for one app:

```bash
flatpak override --user --reset org.example.App
```

Show current overrides:

```bash
flatpak override --user --show org.example.App
```

Show global overrides:

```bash
flatpak override --show
```

Be careful with global overrides. It is easy to forget you changed something for every Flatpak app.

---

## Portals: why file access is weird

Flatpak apps often use portals.

That means the app does not directly browse your whole filesystem. Instead, your desktop environment opens a file picker and gives the app access to the file you selected.

That is why some Flatpak apps can open a file from your home folder without having full home folder access.

This is good. It means the app gets access to what you choose, not everything.

But not every app handles portals perfectly. Some older apps expect normal filesystem access and behave weirdly as Flatpaks.

When that happens, check permissions before blaming the app.

---

## Install from a `.flatpakref` file

Sometimes Flathub gives you a `.flatpakref` file.

Install it like this:

```bash
flatpak install --from app.flatpakref
```

Or directly from URL:

```bash
flatpak install --from https://example.com/app.flatpakref
```

Most of the time you will not need this. Normal Flathub install is cleaner.

---

## Remove or inspect remotes

List remotes:

```bash
flatpak remotes
```

Show apps available in a remote:

```bash
flatpak remote-ls flathub
```

Search inside a remote:

```bash
flatpak remote-ls flathub | grep -i gimp
```

Remove a remote:

```bash
flatpak remote-delete NAME
```

Do not remove Flathub unless you know what you are doing.

---

## Useful maintenance commands

Update everything:

```bash
flatpak update
```

Remove unused runtimes:

```bash
flatpak uninstall --unused
```

Repair Flatpak installation:

```bash
flatpak repair
```

Repair only user install:

```bash
flatpak repair --user
```

Kill a running Flatpak app:

```bash
flatpak kill org.example.App
```

Enter the app sandbox:

```bash
flatpak run --command=sh org.example.App
```

That last one is useful for debugging. You can see what the app sees inside the sandbox.

---

## Where Flatpak apps store config

Flatpak apps usually store user data here:

```text
~/.var/app/APP_ID/
```

Example:

```text
~/.var/app/org.mozilla.firefox/
~/.var/app/org.gimp.GIMP/
```

This is useful when you want to backup or reset an app.

Reset an app brutally:

```bash
flatpak uninstall org.example.App
rm -rf ~/.var/app/org.example.App
```

Do not run that blindly. That deletes the app’s user data.

Better: move it first.

```bash
mv ~/.var/app/org.example.App ~/.var/app/org.example.App.backup
```

Then launch the app again.

---

## Good apps to install as Flatpak

These usually make sense as Flatpaks:

| App type | Examples |
|---|---|
| Browsers | Firefox, LibreWolf, Chromium |
| Chat apps | Telegram, Discord, Slack |
| Media | VLC, OBS, Audacity |
| Graphics | GIMP, Krita, Inkscape |
| Writing | Apostrophe, Marker, Obsidian |
| Emulation | RetroArch, Bottles |
| Dev helpers | Dev Toolbox, DBeaver, Insomnia |

I especially like Flatpak for apps that are annoying to install natively or outdated in distro repos.

For example, on Debian stable or MX Linux, Flatpak can give you newer desktop apps without turning the whole system into a Frankenstein machine.

---

## Apps I would not install as Flatpak

I would avoid Flatpak for:

| App type | Why |
|---|---|
| Kernel tools | They belong to the system |
| Drivers | Same |
| CLI utilities | Usually better from distro repo |
| Shells and terminals | Can work, but weird |
| VPN clients | Need deeper system integration |
| Password managers | Case-by-case |
| IDEs | Depends on workflow |
| Android tools / ADB | Usually better native |
| Docker / Podman tools | Better native |

For IDEs, it depends.

VS Code Flatpak can work, but if you need SDKs, compilers, Android tools, Docker, device access, and weird integrations, native install may be less annoying.

Same with Android Studio. Can be done, but I would rather install it normally.

---

## Flatpak and themes

This is where Linux becomes Linux.

Sometimes Flatpak apps do not follow your GTK/KDE theme perfectly.

You can install matching Flatpak theme packages:

```bash
flatpak search theme
```

Then install the one matching your system theme.

For GTK themes, you may also need to give apps access to theme folders:

```bash
flatpak override --user --filesystem=~/.themes
flatpak override --user --filesystem=~/.icons
```

But again: do not throw random global permissions everywhere unless you know why.

On GNOME and KDE, theme handling is usually better. On XFCE with custom macOS-like themes, expect some rough edges.

---

## Flatpak and external drives

If a Flatpak app cannot see your external drive, it is probably a permission issue.

Example: give access to `/media`:

```bash
flatpak override --user --filesystem=/media org.example.App
```

Or `/run/media` on some distros:

```bash
flatpak override --user --filesystem=/run/media org.example.App
```

For a specific mounted drive:

```bash
flatpak override --user --filesystem=/media/$USER/MyDrive org.example.App
```

Specific folder is better than the entire host.

---

## Flatpak and USB devices

Flatpak can access some device types through permissions, but device access is still where sandboxing gets annoying.

For cameras and microphones, portals usually help.

For USB serial devices, hardware flashers, Android debugging, SDR tools, 3D printers, and similar stuff, Flatpak may not be enough by itself.

You may need:

1. A Flatpak device permission.
2. Correct Linux group membership.
3. A udev rule.
4. Replugging the device.
5. Restarting the app.

For example, for serial devices:

```bash
ls -l /dev/ttyUSB*
ls -l /dev/ttyACM*
```

If the device belongs to `dialout`, add yourself:

```bash
sudo usermod -aG dialout $USER
```

Then log out and back in.

For some USB devices, you may need udev rules under:

```text
/etc/udev/rules.d/
```

Reload rules:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

This is the part where Flatpak is not the whole story. Linux permissions still exist underneath.

---

## Common problems

### App does not start

Run it from terminal:

```bash
flatpak run APP_ID
```

Then read the error.

Also try:

```bash
flatpak update
flatpak repair
```

### App cannot access files

Check permissions:

```bash
flatpak info --show-permissions APP_ID
```

Then use Flatseal or:

```bash
flatpak override --user --filesystem=xdg-download APP_ID
```

### App cannot see external drive

Try:

```bash
flatpak override --user --filesystem=/media APP_ID
flatpak override --user --filesystem=/run/media APP_ID
```

### App looks ugly

Search for theme packages:

```bash
flatpak search gtk theme
```

Or give theme access:

```bash
flatpak override --user --filesystem=~/.themes
flatpak override --user --filesystem=~/.icons
```

### Disk usage is too high

Remove unused runtimes:

```bash
flatpak uninstall --unused
```

Check size:

```bash
du -sh ~/.local/share/flatpak ~/.var/app /var/lib/flatpak 2>/dev/null
```

### You installed the same app twice

Check installations:

```bash
flatpak list --columns=name,application,installation
```

You may have one system install and one user install.

Remove the one you do not want:

```bash
flatpak uninstall --user APP_ID
```

or:

```bash
sudo flatpak uninstall --system APP_ID
```

---

## My basic Flatpak workflow

On a fresh Linux desktop:

```bash
sudo apt install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak update
```

Then install apps:

```bash
flatpak install flathub org.mozilla.firefox
flatpak install flathub org.telegram.desktop
flatpak install flathub org.gimp.GIMP
flatpak install flathub com.github.tchx84.Flatseal
```

After removing apps:

```bash
flatpak uninstall --unused
```

When permissions are weird:

```bash
flatpak info --show-permissions APP_ID
flatpak run APP_ID
```

Then fix with Flatseal.

Simple.

---

## Flatpak is not perfect

The annoying parts:

- apps can be larger
- first install pulls runtimes
- theme integration can be inconsistent
- permissions can confuse normal users
- CLI integration is awkward
- device access can be painful
- some apps are packaged better than others

But it solves a real problem.

Linux has too many distros, too many package formats, too many repo policies, and too many old versions of desktop apps.

Flatpak gives you a boring practical answer:

> Install the app from Flathub and move on.

That is not always the right answer. But for many desktop apps, it is the least annoying one.

---

## Final take

Use your distro package manager for the system.

Use Flatpak for desktop apps.

Use Flatseal when permissions get weird.

Run `flatpak uninstall --unused` sometimes.

Do not give every app full filesystem access just because you are lazy.

And when USB devices are involved, remember: Flatpak permissions are only one layer. Linux groups and udev rules still matter.