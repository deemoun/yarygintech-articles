---
title: "XDG Portals on Linux: The Invisible Layer Between Apps, Sandboxes, and Your Desktop"
description: "A practical Linux article explaining what XDG Desktop Portals are, how they work with Flatpak and Wayland, and how to debug them from the terminal."
pubDate: 2026-06-24
tags: ["linux", "xdg-portals", "flatpak", "wayland", "dbus", "desktop-linux", "sandboxing", "gnome", "kde", "terminal"]
draft: false
---
XDG Portals are one of those Linux desktop technologies that most users never notice until something breaks.

- Your file picker does not open.  
- Screen sharing in the browser stops working.  
- OBS cannot see your screen on Wayland.  
- A Flatpak app cannot access a folder.  
- A screenshot tool works in GNOME but fails in Sway or Hyprland.

Very often, the problem is not the app itself. The problem is somewhere in the portal stack.

XDG Desktop Portals are a standard way for applications to ask the desktop environment for controlled access to files, cameras, screens, notifications, secrets, and other desktop features.

They matter most for:

- Flatpak apps
- Wayland screen sharing
- Sandboxed applications
- Browser permissions
- File pickers
- Desktop notifications
- Modern Linux security

In simple terms, XDG Portals are the permission bridge between an application and the real desktop.

## The Problem XDG Portals Solve

Traditional Linux desktop apps were usually trusted completely.

If an app ran as your user, it could often read your home folder, inspect many config files, talk to devices, and interact with desktop services directly.

That model is simple, but it is not very secure.

Modern Linux desktops are moving toward a better model:

```text
App asks for access
Desktop asks the user
User approves or denies
App receives limited access
```

This is how mobile operating systems have worked for years. Linux desktops are slowly catching up.

Instead of giving apps direct access to everything, portals allow apps to request specific operations through a trusted system component.

For example:

```text
Bad old model:
App directly opens ~/Documents, ~/Downloads, ~/.ssh, etc.

Portal model:
App asks for a file chooser.
User selects one file.
App gets access only to that file.
```

That is a huge security improvement.

## Basic Architecture

The main service is:

```bash
xdg-desktop-portal
```

This is the generic front door. Applications talk to it over D-Bus.

But `xdg-desktop-portal` does not usually implement every desktop-specific behavior by itself. It delegates to a backend.

Common backends include:

```text
xdg-desktop-portal-gnome
xdg-desktop-portal-gtk
xdg-desktop-portal-kde
xdg-desktop-portal-wlr
xdg-desktop-portal-hyprland
```

A simplified architecture looks like this:

```text
Application
   |
   | D-Bus request
   v
xdg-desktop-portal
   |
   | delegates request
   v
Portal backend
   |
   v
GNOME / KDE / wlroots / Hyprland / etc.
```

The app does not need to know whether you are using GNOME, KDE Plasma, Sway, or Hyprland. It talks to the same portal interface.

## Why This Matters More on Wayland

On X11, applications could do many things too freely.

An X11 app could potentially:

- Watch keyboard input
- Capture the screen
- Inspect windows
- Mess with other apps in ugly ways

Wayland intentionally blocks much of that.

That is good for security, but it creates a practical problem:

How does OBS record the screen?  
How does Firefox share a browser window in a video call?  
How does a screenshot app work?

The answer is: through portals.

On Wayland, screen capture typically flows through the ScreenCast portal.

```text
Browser / OBS
   |
   v
xdg-desktop-portal ScreenCast API
   |
   v
Desktop backend
   |
   v
PipeWire stream
```

PipeWire is usually involved in screen capture and audio/video routing.

So if screen sharing breaks on Wayland, you often need to check:

- xdg-desktop-portal
- the correct portal backend
- PipeWire
- the compositor
- browser/app support

## Common Portal Interfaces

Portals expose several APIs. Some of the most important ones are:

## File Chooser Portal

Used when an app needs the user to select a file or folder.

Example use cases:

- Opening a document in a Flatpak editor
- Uploading a file in a browser
- Saving a file from a sandboxed app

Instead of giving the app full filesystem access, the user picks a file and the app receives limited permission.

## OpenURI Portal

Used to open links safely.

For example, a chat app can request:

```text
Open https://example.com in the default browser
```

The app does not need to know which browser you use.

## ScreenCast Portal

Used for screen sharing and recording, especially on Wayland.

Common users:

- OBS Studio
- Firefox
- Chromium
- Discord
- Zoom
- Electron apps

This is one of the most common portal pain points.

## Screenshot Portal

Used for taking screenshots in a controlled way.

This matters because Wayland compositors do not allow random apps to scrape the screen freely.

## Camera Portal

Used to request camera access.

Not every app uses it perfectly yet, but it is part of the broader direction Linux desktops are moving toward.

## Notification Portal

Used to show desktop notifications through a standard interface.

This helps apps behave consistently across desktop environments.

## Secret Portal

Used for access to secret storage, depending on environment support.

This can connect to systems like GNOME Keyring or KWallet.

## Location Portal

Used when an app requests location data.

The exact behavior depends heavily on desktop environment and service support.

## Checking Whether Portals Are Running

Start with systemd user services:

```bash
systemctl --user status xdg-desktop-portal
```

You should see whether the service is active.

Example:

```text
Active: active (running)
```

Also check the backend:

```bash
systemctl --user status xdg-desktop-portal-gnome
systemctl --user status xdg-desktop-portal-kde
systemctl --user status xdg-desktop-portal-gtk
systemctl --user status xdg-desktop-portal-wlr
systemctl --user status xdg-desktop-portal-hyprland
```

Not all of these will exist on your system. That is normal.

To list portal-related services:

```bash
systemctl --user list-units | grep portal
```

## Checking Installed Portal Packages

On Debian, Ubuntu, MX Linux, Linux Mint:

```bash
dpkg -l | grep xdg-desktop-portal
```

On Fedora:

```bash
rpm -qa | grep xdg-desktop-portal
```

On Arch Linux:

```bash
pacman -Qs xdg-desktop-portal
```

Typical packages might look like:

```text
xdg-desktop-portal
xdg-desktop-portal-gtk
xdg-desktop-portal-gnome
xdg-desktop-portal-kde
xdg-desktop-portal-wlr
xdg-desktop-portal-hyprland
```

## Inspecting the Portal D-Bus Service

Portals communicate over the session D-Bus.

You can inspect the portal service with:

```bash
busctl --user tree org.freedesktop.portal.Desktop
```

You can also introspect the main object:

```bash
busctl --user introspect org.freedesktop.portal.Desktop /org/freedesktop/portal/desktop
```

You should see interfaces like:

```text
org.freedesktop.portal.FileChooser
org.freedesktop.portal.OpenURI
org.freedesktop.portal.ScreenCast
org.freedesktop.portal.Screenshot
org.freedesktop.portal.Notification
org.freedesktop.portal.Settings
```

This is a useful sanity check.

If the D-Bus service does not exist, your portal service may not be running.

## Watching Portal Logs

The most useful debugging command is:

```bash
journalctl --user -u xdg-desktop-portal -f
```

Then reproduce the problem.

For example:

1. Start the log command.
2. Open Firefox.
3. Try screen sharing.
4. Watch what appears in the journal.

Also check backend logs:

```bash
journalctl --user | grep xdg-desktop-portal
```

Or more aggressively:

```bash
journalctl --user -f | grep -i portal
```

## Restarting Portals

Sometimes the portal stack gets confused after desktop upgrades, backend changes, or switching sessions.

Restart it:

```bash
systemctl --user restart xdg-desktop-portal
```

You can also restart a backend:

```bash
systemctl --user restart xdg-desktop-portal-gtk
```

or:

```bash
systemctl --user restart xdg-desktop-portal-kde
```

or:

```bash
systemctl --user restart xdg-desktop-portal-wlr
```

Then retry the broken app.

## Checking Which Desktop Session You Are In

Portal backend selection depends partly on your desktop environment.

Check:

```bash
echo $XDG_CURRENT_DESKTOP
```

Also check:

```bash
echo $XDG_SESSION_TYPE
```

Typical output:

```text
wayland
```

or:

```text
x11
```

Check desktop session:

```bash
loginctl show-session "$XDG_SESSION_ID" -p Type -p Desktop
```

If this outputs weird or empty values, portal backend selection can break.

## Backend Configuration

Portal backend preference is often controlled by files under:

```bash
/usr/share/xdg-desktop-portal/
```

Look for:

```bash
ls /usr/share/xdg-desktop-portal/
```

You may see files like:

```text
portals.conf
gnome-portals.conf
kde-portals.conf
hyprland-portals.conf
```

Inspect them:

```bash
cat /usr/share/xdg-desktop-portal/portals.conf
```

or:

```bash
grep -R "default" /usr/share/xdg-desktop-portal/
```

On some systems, custom overrides may exist under:

```bash
~/.config/xdg-desktop-portal/
```

Check with:

```bash
ls ~/.config/xdg-desktop-portal/
```

## Flatpak and Portals

Flatpak apps heavily depend on portals.

List installed Flatpak apps:

```bash
flatpak list --app
```

Show permissions for one app:

```bash
flatpak info --show-permissions com.example.App
```

Run a Flatpak app with verbose output:

```bash
flatpak run -v com.example.App
```

You can also override permissions:

```bash
flatpak override --user --filesystem=xdg-download com.example.App
```

Remove the override:

```bash
flatpak override --user --reset com.example.App
```

But here is the important point:

If an app supports portals properly, you should not need to give it broad filesystem access.

This is worse:

```bash
flatpak override --user --filesystem=home com.example.App
```

This is better:

```text
Use the portal file chooser and grant only the selected file.
```

## Testing File Chooser Behavior

A good practical test is to open a Flatpak app and try opening a file outside its sandbox.

For example:

```bash
flatpak run org.gnome.TextEditor
```

Then try to open a file from a folder the app normally cannot access.

If the portal file chooser works, the selected file should open.

If the file picker does not appear, check:

```bash
journalctl --user -u xdg-desktop-portal -f
```

## Testing Screen Sharing on Wayland

First check session type:

```bash
echo $XDG_SESSION_TYPE
```

If it says:

```text
wayland
```

then screen sharing likely depends on portals.

Check PipeWire:

```bash
systemctl --user status pipewire
systemctl --user status wireplumber
```

Some distros may use `pipewire-media-session` instead of WirePlumber:

```bash
systemctl --user status pipewire-media-session
```

Try screen sharing in Firefox or Chromium and watch logs:

```bash
journalctl --user -f | grep -Ei "portal|pipewire|screencast"
```

For OBS, make sure the PipeWire screen capture source is available.

On many systems the flow is:

```text
OBS -> xdg-desktop-portal -> backend -> compositor -> PipeWire stream
```

If one link is broken, screen capture fails.

## GNOME Notes

GNOME usually uses:

```text
xdg-desktop-portal-gnome
```

You may also see:

```text
xdg-desktop-portal-gtk
```

GNOME has strong portal integration, especially for file chooser, screenshots, settings, and screen sharing.

Useful checks:

```bash
echo $XDG_CURRENT_DESKTOP
```

Expected:

```text
GNOME
```

Check packages:

```bash
dpkg -l | grep xdg-desktop-portal-gnome
```

or:

```bash
rpm -qa | grep xdg-desktop-portal-gnome
```

## KDE Plasma Notes

KDE Plasma usually uses:

```text
xdg-desktop-portal-kde
```

This backend integrates portals with KDE dialogs, KWin, and Plasma behavior.

Useful check:

```bash
echo $XDG_CURRENT_DESKTOP
```

Expected values may include:

```text
KDE
```

Check package:

```bash
pacman -Qs xdg-desktop-portal-kde
```

or your distro equivalent.

## Sway and wlroots Notes

Sway commonly uses:

```text
xdg-desktop-portal-wlr
```

For screen sharing, wlroots-based compositors often need this backend.

Check package:

```bash
pacman -Qs xdg-desktop-portal-wlr
```

Screen sharing issues on Sway are commonly related to:

- Missing `xdg-desktop-portal-wlr`
- Broken PipeWire setup
- Wrong `XDG_CURRENT_DESKTOP`
- Missing compositor configuration
- App not using the portal correctly

## Hyprland Notes

Hyprland commonly uses:

```text
xdg-desktop-portal-hyprland
```

For Hyprland, do not blindly install every portal backend and hope it works. Too many conflicting backends can make behavior worse.

Check:

```bash
pacman -Qs xdg-desktop-portal-hyprland
```

Watch logs:

```bash
journalctl --user -f | grep -i hyprland
```

and:

```bash
journalctl --user -f | grep -i portal
```

## Common Breakage Pattern: Too Many Backends

A common Linux problem:

```text
You install GNOME.
Then KDE.
Then Hyprland.
Then random Flatpak tools.
Now you have several portal backends installed.
```

The system may choose the wrong backend for some interfaces.

Check installed backends:

```bash
dpkg -l | grep xdg-desktop-portal
```

or:

```bash
pacman -Qs xdg-desktop-portal
```

Then check the portal config:

```bash
grep -R . /usr/share/xdg-desktop-portal/
```

The fix is usually not “install more stuff.”  
The fix is usually “install the correct backend and make sure it is selected.”

## Common Problem: File Picker Does Not Open

Debug checklist:

```bash
systemctl --user status xdg-desktop-portal
systemctl --user list-units | grep portal
echo $XDG_CURRENT_DESKTOP
echo $XDG_SESSION_TYPE
journalctl --user -u xdg-desktop-portal -f
```

Then try opening the file picker again.

Possible causes:

- No backend installed
- Wrong backend selected
- Broken D-Bus session
- App bug
- Flatpak permission issue
- Mixed desktop environment configuration

## Common Problem: Screen Sharing Broken in Browser

Debug checklist:

```bash
echo $XDG_SESSION_TYPE
systemctl --user status xdg-desktop-portal
systemctl --user status pipewire
systemctl --user status wireplumber
journalctl --user -f | grep -Ei "portal|pipewire|screencast"
```

In Chromium-based browsers, also check whether PipeWire/WebRTC screen capture support is enabled. Most modern builds support it, but old builds and weird distro packages can still cause problems.

In Firefox, Wayland screen sharing generally expects a working portal and PipeWire setup.

## Common Problem: Flatpak App Cannot See Files

First inspect permissions:

```bash
flatpak info --show-permissions APP_ID
```

Example:

```bash
flatpak info --show-permissions org.mozilla.firefox
```

Then check overrides:

```bash
flatpak override --show org.mozilla.firefox
```

Reset bad overrides:

```bash
flatpak override --user --reset org.mozilla.firefox
```

If the app supports portals, prefer using the file picker instead of granting full home directory access.

## Useful One-Liner Debug Script

Here is a quick command bundle:

```bash
echo "Desktop: $XDG_CURRENT_DESKTOP"
echo "Session: $XDG_SESSION_TYPE"
echo "Session ID: $XDG_SESSION_ID"
echo
systemctl --user --no-pager status xdg-desktop-portal | sed -n '1,12p'
echo
systemctl --user list-units | grep portal
echo
busctl --user tree org.freedesktop.portal.Desktop 2>/dev/null || echo "Portal D-Bus service not available"
```

This gives you a quick overview of the portal state.

## Practical Mental Model

Think of portals like this:

```text
The app is not allowed to touch something directly.
So it asks the desktop to do it.
The desktop shows UI or applies policy.
The app receives limited access.
```

That is the entire idea.

Portals are not just a Flatpak detail. They are now part of the modern Linux desktop security model.

## Why Linux Needed This

Linux desktop security historically relied too much on user trust.

If you installed an app, it often had broad access to your user account.

That is not good enough anymore.

Today, many apps are:

- Electron apps
- Browser-based apps
- Third-party Flatpaks
- Closed-source communication tools
- Random utilities from GitHub
- AI tools
- Screen recorders
- File converters

Giving all of them full access to your home directory is a bad idea.

Portals help move Linux toward a more controlled desktop permission model.

## Developer Notes

If you are building a Linux desktop app, you should avoid hardcoding desktop-specific behavior.

Do not assume:

```text
GNOME file picker
KDE file picker
X11 screen capture
Direct filesystem access
```

Instead, use toolkit APIs that already integrate with portals.

GTK, Qt, Electron, browsers, and Flatpak-aware libraries often use portals automatically when running in the right environment.

For Flatpak packaging, test your app without broad filesystem permissions. If the app only works with:

```bash
--filesystem=home
```

then your sandbox story is probably weak.

## Final Thoughts

XDG Portals are not flashy. They are plumbing.

But they are important plumbing.

They make Flatpak usable.  
They make Wayland screen sharing possible.  
They make sandboxed apps less annoying.  
They make Linux desktop security more realistic.

If you use Linux in 2026, especially with Wayland and Flatpak, portals are no longer optional background trivia. They are one of the main layers that decide whether your desktop apps feel modern or broken.

And when something does break, the terminal commands in this article give you a practical place to start.