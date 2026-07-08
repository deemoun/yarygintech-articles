---
title: "How macOS Evolved: From Tiger to Modern macOS"
description: "An engineer guide on how macOS has changed over the years."
pubDate: 2026-07-08
tags: ["MacOS", "Software Engineering", "MacOS Core", "UNIX"]
draft: false
---
When people compare old and new macOS, they usually compare screenshots.

Tiger had brushed metal.
Modern macOS has rounded glassy panels.
Tiger had Dashboard.
Modern macOS has widgets again.
Tiger had blue Aqua buttons.
Modern macOS has a design language that tries to look like iOS, iPadOS and visionOS had a child.

That is the boring comparison.

The real story is not the wallpaper.  
The real story is what happened under the desktop.

Mac OS X Tiger was not just “old macOS”. It was already a serious Unix workstation OS with a consumer-friendly face. But modern macOS is a very different machine:

- different CPU architecture
- different filesystem
- different boot model
- different security model
- different app distribution model
- different service manager
- different graphics stack
- different driver architecture
- different relationship between the user and the system

And yet if you open Terminal, type `ls`, `cd`, `ps`, `top`, or `open .`, you can still feel the same DNA.

That is what makes macOS interesting.

It changed massively, but it did not become a completely different operating system.

---

## 1. The real family tree: Classic Mac OS, NeXTSTEP, BSD, Mach, Darwin

macOS is not a direct continuation of classic Mac OS in the way many casual users imagine.

Classic Mac OS gave Apple the culture, the Finder metaphor, the menu bar, the desktop experience, and a lot of UI expectations.

But the technical foundation of Mac OS X came from NeXTSTEP.

That matters.

NeXTSTEP brought:

- Mach-based kernel ideas
- BSD userland
- Objective-C runtime
- Display PostScript heritage
- application bundles
- property lists
- services menu
- Workspace Manager ideas that later influenced Finder behavior
- development tools that eventually became Xcode
- frameworks that became Cocoa

Classic Mac OS was elegant for users, but architecturally it was aging badly by the late 1990s.

It lacked:

- protected memory
- preemptive multitasking in the modern sense
- a real multi-user security model
- strong Unix-style process isolation
- a modern networking and server foundation

NeXTSTEP solved many of those problems.

So Mac OS X was basically this strange Apple cocktail:

```text
Classic Mac UI ideas
+ NeXTSTEP application model
+ Mach
+ BSD Unix
+ Apple frameworks
+ Apple hardware integration
= Mac OS X
```

You can still see NeXTSTEP fingerprints today.

For example, application bundles:

```bash
ls -la /Applications/Safari.app
```

A `.app` is not a normal executable file. It is a directory bundle.

```bash
find /Applications/Safari.app -maxdepth 3 -type f | head
```

Typical structure:

```text
Safari.app/
  Contents/
    Info.plist
    MacOS/
    Resources/
    Frameworks/
```

That idea feels normal today, but it is very NeXT-like.

Same with property lists:

```bash
plutil -p /Applications/Safari.app/Contents/Info.plist
```

macOS still loves `.plist` files.

App metadata, launch agents, preferences, daemon definitions, sandbox profiles, configuration state — everywhere you look, property lists are still there.

This is one of the most important parts of macOS history:

> macOS did not just inherit a kernel. It inherited an entire object-oriented operating system culture from NeXT.

---

## 2. Tiger was already modern in the places that mattered

Mac OS X 10.4 Tiger shipped in 2005.

If you boot it today, it feels old, but not prehistoric.

Why?

Because the core model was already there:

- Unix filesystem
- Terminal
- `/Applications`
- `/Users`
- `/Library`
- `/System`
- `.app` bundles
- launchd
- Spotlight
- Dashboard
- Automator
- Core Image
- Core Data
- Quartz
- Cocoa
- Carbon
- X11 support

Tiger was not a toy.

You can inspect the system like any Unix machine:

```bash
uname -a
sw_vers
id
whoami
ps aux
top
df -h
mount
```

On Tiger, you would see a Darwin kernel version from the Darwin 8 era.  
On a modern Mac, you see a much newer Darwin/XNU base.

Modern macOS still answers the same kind of questions:

```bash
sw_vers
```

Example:

```text
ProductName:        macOS
ProductVersion:     26.x
BuildVersion:       25xxxx
```

And:

```bash
uname -a
```

Shows the Darwin kernel.

So the user-facing name changed from “Mac OS X” to “OS X” to “macOS”, but the Unix core is still Darwin.

That is why old shell knowledge still works.

Tiger feels old because of the UI and ecosystem.

It does not feel old because of the Unix model.

---

## 3. Darwin and XNU: the thing under the thing

Darwin is the open-source-ish core of macOS.

Not all of macOS is open source. Finder is not open source. AppKit is not open source. Aqua is not open source. The good shiny Apple parts are not open source.

But the low-level foundation historically included Darwin components.

The kernel is XNU.

XNU famously means:

```text
X is Not Unix
```

A very nerdy name.

XNU combines:

- Mach kernel concepts
- BSD process/network/filesystem layer
- I/O Kit driver model

That hybrid architecture matters because macOS is not “just BSD”.

If you ask Linux people what macOS is, they often say:

> “It’s Unix with a weird Apple UI.”

That is close, but not precise.

A better mental model:

```text
Mach-ish kernel foundation
+ BSD personality
+ Apple driver stack
+ Apple frameworks
+ Aqua desktop
```

Try:

```bash
sysctl kern.ostype
sysctl kern.osrelease
sysctl kern.version
```

You can also inspect Mach-related details indirectly:

```bash
hostinfo
```

And kernel extensions / system extensions:

```bash
kmutil showloaded
systemextensionsctl list
```

On Tiger, kernel extensions were a much more normal way of extending the system.

Modern macOS has moved away from old-style third-party kernel extensions because they are dangerous. If a bad kernel extension crashes, the whole machine can crash.

Modern Apple wants drivers and low-level extensions to move into safer models:

- DriverKit
- System Extensions
- Endpoint Security
- Network Extensions

In other words:

```text
Old model:
third-party code in kernel space

New model:
more third-party code in user space with entitlements and approval
```

This is safer, but also more annoying for developers.

That is basically the macOS evolution story in one sentence.

Safer. More locked down. More complex.

---

## 4. From init chaos to launchd

One of the biggest technical shifts happened around Tiger.

Tiger introduced `launchd`.

Before launchd, Unix systems usually had a mix of mechanisms:

- `/etc/rc`
- SystemStarter
- cron
- inetd/xinetd
- startup items
- login hooks
- random shell scripts

It worked, but it was messy.

Apple replaced a lot of that with launchd.

Today launchd is central to macOS.

It manages:

- daemons
- agents
- scheduled jobs
- socket activation
- per-user services
- system services
- background jobs

Check PID 1:

```bash
ps -p 1 -o comm=
```

On modern macOS:

```text
/sbin/launchd
```

List services:

```bash
launchctl list
```

Show a specific service:

```bash
launchctl print system/com.apple.sshd
```

Enable remote login daemon:

```bash
sudo systemsetup -setremotelogin on
```

Then inspect:

```bash
launchctl print system/com.openssh.sshd
```

A launch daemon is usually just a plist file.

System daemons:

```bash
ls /System/Library/LaunchDaemons | head
```

Third-party daemons:

```bash
ls /Library/LaunchDaemons
```

User agents:

```bash
ls ~/Library/LaunchAgents
```

A simple launch agent looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
 "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.example.hello</string>

  <key>ProgramArguments</key>
  <array>
    <string>/usr/bin/say</string>
    <string>Hello from launchd</string>
  </array>

  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

Load it:

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.example.hello.plist
```

Unload it:

```bash
launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.example.hello.plist
```

This is a great thing to show in a video because it explains macOS better than another wallpaper comparison.

Tiger was the beginning of the launchd era.

Modern macOS is launchd everywhere.

---

## 5. NetInfo died, Directory Services survived

Old Mac OS X had NetInfo.

If you used early OS X, you may remember commands like:

```bash
niutil
```

NetInfo was inherited from NeXT.

It handled networked directory information: users, groups, machines, mounts, and other administrative data.

But NetInfo did not survive forever.

Apple moved toward Directory Services and later Open Directory-style infrastructure.

Modern macOS uses commands like:

```bash
dscl . -list /Users
dscl . -read /Users/$(whoami)
id
groups
```

You can inspect local users:

```bash
dscl . -list /Users UniqueID
```

Find your user record:

```bash
dscl . -read /Users/$(whoami)
```

This is one of those invisible changes.

Normal users never noticed.

But system administrators did.

Old Mac OS X felt more like NeXT and classic Unix admin ideas glued together.

Modern macOS feels more like a managed endpoint platform:

- local users
- mobile accounts
- MDM
- profiles
- secure tokens
- FileVault
- iCloud identity
- Apple Account integration
- platform SSO in enterprise contexts

That is a major evolution.

The Mac went from “cool Unix workstation for humans” to “managed corporate endpoint with consumer UX”.

---

## 6. HFS+ to APFS: this was not just a filesystem change

Tiger used HFS+.

HFS+ was old but very Apple.

It had:

- resource fork support
- metadata support
- case-insensitive default behavior
- good compatibility with classic Mac expectations

But HFS+ was designed for another era.

Modern macOS uses APFS.

APFS brought:

- snapshots
- clones
- space sharing
- native encryption
- fast directory sizing in many cases
- better SSD behavior
- volume groups
- sealed system volume support

Check APFS:

```bash
diskutil apfs list
```

Show mounted volumes:

```bash
mount
```

Modern macOS no longer treats the boot disk as one simple writable system volume.

It uses a system/data split.

You can inspect it:

```bash
diskutil list
```

You will usually see something conceptually like:

```text
Macintosh HD
Macintosh HD - Data
Preboot
Recovery
VM
```

The system volume is sealed and mostly read-only.

User data lives separately.

This is a huge philosophical shift.

Tiger model:

```text
The admin owns the machine.
Root can change almost everything.
System files are real files you can modify.
```

Modern model:

```text
The OS protects itself from the admin.
System files are sealed.
Updates replace snapshots.
Security boundaries are part of normal operation.
```

You can see snapshots:

```bash
diskutil apfs listSnapshots /
```

Or:

```bash
tmutil listlocalsnapshots /
```

APFS also enables fast cloning.

Try:

```bash
cp -c largefile.iso clone.iso
```

That `-c` clone behavior can create a copy-on-write clone on APFS.

This is not just faster copying.

It changes what a “copy” means.

On Tiger, a copy was usually a copy.

On APFS, a copy may initially be metadata pointing to shared blocks until one file changes.

That is a massive difference.

---

## 7. The security model: from “Unix admin” to “platform owner”

Tiger trusted the admin much more.

If you had `sudo`, you were close to god.

Modern macOS disagrees.

Modern macOS has layers:

- Gatekeeper
- code signing
- notarization
- sandboxing
- System Integrity Protection
- Transparency, Consent, and Control
- signed system volume
- hardened runtime
- entitlements
- secure boot on Apple Silicon
- sealed system volume
- privacy database
- background item approval

Check SIP:

```bash
csrutil status
```

Check Gatekeeper:

```bash
spctl --status
```

Assess an app:

```bash
spctl --assess --verbose /Applications/SomeApp.app
```

Check code signature:

```bash
codesign -dv --verbose=4 /Applications/Safari.app
```

Verify signature:

```bash
codesign --verify --deep --strict --verbose=2 /Applications/Safari.app
```

Check notarization-ish assessment:

```bash
spctl -a -vvv -t install /Applications/SomeApp.app
```

Modern macOS asks for permission for things that old Mac OS X barely cared about:

- microphone
- camera
- screen recording
- accessibility
- full disk access
- location
- contacts
- calendar
- reminders
- desktop/documents/downloads access in some contexts
- automation control

The privacy database is TCC.

You can see it:

```bash
ls ~/Library/Application\ Support/com.apple.TCC/
```

System database:

```bash
sudo ls /Library/Application\ Support/com.apple.TCC/
```

This is why random apps can no longer simply read everything silently.

From a user perspective, this is good.

From a developer perspective, this can be pain.

From a retro Mac perspective, it is kind of sad because the machine feels less “yours”.

But from a security perspective, it is obviously necessary.

The internet is not 2005 anymore.

---

## 8. Application models: Carbon, Cocoa, Classic, Rosetta, Catalyst, SwiftUI

Tiger lived in a transitional world.

It still had Classic support on PowerPC Macs.

That meant you could run many Mac OS 9 applications inside the Classic environment.

The stack looked roughly like:

```text
Classic Mac apps -> Classic environment
Carbon apps      -> compatibility bridge for old Mac apps
Cocoa apps       -> NeXTSTEP-style modern OS X apps
Unix tools       -> Darwin/BSD layer
Java apps        -> because 2005 was like that
```

Carbon was extremely important.

Without Carbon, Apple could not have moved major Mac software to OS X quickly enough.

Adobe, Microsoft, big professional apps — they needed a migration path.

Cocoa was the “future”.

Carbon was the bridge.

Classic eventually disappeared.

Then PowerPC disappeared.

Then 32-bit apps disappeared.

Then kernel extensions started disappearing.

Then Intel started disappearing.

macOS history is full of bridges that Apple later burns.

A simple timeline:

```text
Classic Environment  -> gone
Carbon UI apps       -> mostly gone / deprecated
PowerPC apps         -> Rosetta 1, then gone
32-bit Intel apps    -> gone
Intel apps on ARM    -> Rosetta 2, now also on the clock
Kernel extensions    -> replaced by system extensions / DriverKit
```

Check app architecture:

```bash
file /Applications/SomeApp.app/Contents/MacOS/SomeApp
```

Universal binary:

```bash
lipo -info /Applications/SomeApp.app/Contents/MacOS/SomeApp
```

Example output:

```text
Architectures in the fat file: SomeApp are: x86_64 arm64
```

Run an Intel shell under Rosetta on Apple Silicon:

```bash
arch -x86_64 zsh
```

Check:

```bash
arch
```

You should see:

```text
i386
```

Then exit:

```bash
exit
```

Modern macOS also has newer application frameworks:

- AppKit still powers serious Mac apps
- SwiftUI is Apple's cross-platform declarative UI model
- Catalyst brings iPad apps to macOS
- Electron brings the web to desktop, for better and worse

But old NeXT DNA is still visible.

`Info.plist` is still there.
Bundles are still there.
Frameworks are still there.
Services are still there.
Objective-C runtime is still there.

Modern macOS is not “pure SwiftUI future”.

It is layers on layers.

---

## 9. Services: one of the most NeXT things still hiding in plain sight

The Services menu is one of the most underrated macOS features.

It came from NeXTSTEP-style inter-application communication ideas.

You select text, go to:

```text
App Menu -> Services
```

And another app can process that selection.

This is not just UI decoration.

It reflects a deeper design philosophy:

> Applications should expose actions to the system, not just live inside their own windows.

Modern macOS also has:

- Share extensions
- Finder Quick Actions
- Automator workflows
- Shortcuts
- Services
- App extensions

These are all related ideas.

Different eras, similar goal.

Tiger had Automator. That was a big deal.

Modern macOS has Shortcuts. More consumer-friendly, more iOS-like.

You can inspect Automator workflows:

```bash
ls ~/Library/Services
```

Create a simple service / quick action and it often lands as a workflow bundle.

Modern Shortcuts are more locked into Apple's current ecosystem, but the idea is old:

```text
Take input
Transform it
Pass it somewhere else
```

This is a very NeXT / Unix / automation-friendly idea, even if Apple keeps changing the UI around it.

---

## 10. Spotlight: from Tiger feature to system database layer

Spotlight was one of Tiger's headline features.

In 2005 it felt like magic:

- instant file search
- metadata indexing
- search inside documents
- smart folders
- system-wide search UI

Modern macOS still uses Spotlight, but it is more than a search box.

It is part of the system's metadata layer.

Check metadata:

```bash
mdls somefile.txt
```

Search from Terminal:

```bash
mdfind "kind:pdf"
```

Find files by name:

```bash
mdfind -name "invoice"
```

Check indexing status:

```bash
mdutil -s /
```

Turn indexing off for a volume:

```bash
sudo mdutil -i off /
```

Turn it back on:

```bash
sudo mdutil -i on /
```

Spotlight also powers parts of Finder search, smart folders, and app launching.

Tiger introduced the basic user-facing concept.

Modern macOS turned metadata indexing into normal infrastructure.

Again, this is a pattern:

```text
Old macOS:
cool feature

Modern macOS:
platform service
```

---

## 11. Networking: from normal Unix networking to controlled platform networking

Tiger had familiar Unix networking tools:

```bash
ifconfig
netstat
ping
traceroute
lsof -i
```

Modern macOS still has many of them, but Apple has added layers.

Modern commands:

```bash
networksetup -listallhardwareports
networksetup -getinfo Wi-Fi
scutil --dns
scutil --proxy
```

Check routes:

```bash
netstat -rn
```

or:

```bash
route -n get default
```

DNS:

```bash
scutil --dns
```

Modern macOS networking is deeply integrated with:

- System Configuration framework
- Network Extension framework
- VPN profiles
- content filters
- per-app VPN
- MDM
- private relay-style features
- application firewall
- packet filter

Application firewall:

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

Packet filter rules:

```bash
sudo pfctl -sr
```

In old Mac OS X, if you were root and knew Unix networking, you felt mostly in control.

In modern macOS, networking is still Unix-like underneath, but user-space frameworks, privacy controls, MDM profiles, and signed system components matter much more.

This is why debugging networking on macOS can be weird.

It is Unix.

But it is Apple Unix.

---

## 12. Logging: from readable logs to unified logging

Old OS X logs were easier to understand.

You could look in:

```bash
/var/log/system.log
```

Use:

```bash
tail -f /var/log/system.log
```

Modern macOS uses unified logging.

The command is:

```bash
log show
```

Recent logs:

```bash
log show --last 10m
```

Stream live logs:

```bash
log stream
```

Filter by process:

```bash
log stream --predicate 'process == "Finder"'
```

Filter by subsystem:

```bash
log show --last 1h --predicate 'subsystem CONTAINS "com.apple"'
```

Unified logging is powerful, but less friendly.

Old model:

```text
Open text file. Read logs.
```

New model:

```text
Query structured logging database with predicates.
```

This is another example of macOS becoming more advanced and less hackable at the same time.

---

## 13. Preferences: defaults, plists, and the old NeXT smell

The `defaults` command is one of those macOS things that feels ancient and modern at the same time.

Read a preference domain:

```bash
defaults read com.apple.finder
```

Change a Finder setting:

```bash
defaults write com.apple.finder AppleShowAllFiles -bool true
killall Finder
```

Revert:

```bash
defaults write com.apple.finder AppleShowAllFiles -bool false
killall Finder
```

Show file extensions:

```bash
defaults write NSGlobalDomain AppleShowAllExtensions -bool true
killall Finder
```

This whole system comes from property-list-based preferences.

Preference files live in places like:

```bash
~/Library/Preferences
/Library/Preferences
```

Example:

```bash
ls ~/Library/Preferences | head
```

This is not how Windows thinks about settings.

This is not how Linux desktops usually think about settings either.

It is very Apple/NeXT.

Modern macOS has System Settings with a big iOS-like interface, but underneath many preferences still exist as plist domains.

The UI changed.

The old machinery is still there.

---

## 14. Finder: surprisingly conservative, but the world around it changed

Finder in Tiger and Finder today are recognizably related.

You still have:

- icon view
- list view
- column view
- Get Info
- sidebar
- aliases
- packages
- network shares
- mounted volumes
- tags / labels concept

But modern Finder lives in a different world.

Tiger Finder managed local files, mounted disks, network shares, and app bundles.

Modern Finder also deals with:

- iCloud Drive
- cloud placeholders
- AirDrop
- file provider extensions
- tags
- gallery view
- quick actions
- SMB improvements
- sandboxed app document access
- external drive privacy prompts
- APFS snapshots
- Time Machine integration

Try:

```bash
ls -lO /
```

Flags matter on macOS.

Example:

```bash
ls -lO /System
```

You may see flags like restricted.

That is SIP territory.

Try extended attributes:

```bash
xattr -l somefile
```

Remove quarantine from a downloaded app:

```bash
xattr -dr com.apple.quarantine SomeApp.app
```

This is important.

Modern Finder is not just showing files.

It is showing files inside a security and metadata universe.

Downloaded apps may carry quarantine attributes.
Cloud files may not fully exist locally.
Bundles are directories pretending to be apps.
APFS copies may not be real full copies yet.
Some folders are protected by TCC.
Some locations are protected by SIP.

So Finder looks similar, but the meaning of “file” became more complicated.

---

## 15. Display and graphics: Quartz to Metal

Tiger's graphics stack was already impressive.

It had:

- Quartz
- Core Graphics
- Core Image
- OpenGL
- PDF-based rendering ideas
- hardware acceleration increasingly becoming important

macOS historically had a strong graphics architecture.

The system could render beautiful UI and typography when Windows still looked rough in many places.

Modern macOS has moved toward Metal.

OpenGL is deprecated.

Quartz is still there, but GPU programming and modern rendering are Metal-first.

You can inspect GPU info:

```bash
system_profiler SPDisplaysDataType
```

On Apple Silicon, GPU is part of the SoC.

That changes the architecture.

Old model:

```text
CPU
separate GPU
separate VRAM
drivers
```

Apple Silicon model:

```text
SoC
unified memory
integrated GPU
Neural Engine
media engines
Secure Enclave
```

This is why modern Macs can feel ridiculously efficient.

The OS and hardware are now designed together in a much deeper way than the Intel era.

PowerPC Macs were also Apple-controlled machines, but Apple did not design the CPU stack like it does now.

Apple Silicon is the closest Apple has come to the “whole widget” dream.

---

## 16. Boot process: from understandable to sealed appliance

Old Macs were easier to reason about.

PowerPC Macs used Open Firmware.

You could boot into Open Firmware and type weird Forth-like commands.

Intel Macs used EFI.

Modern Apple Silicon Macs use a much more Apple-controlled boot chain.

The modern boot process includes:

- secure boot policy
- signed system volume
- recoveryOS
- boot manifests
- APFS volume groups
- startup security settings
- paired system/data volumes

You can inspect boot-related things:

```bash
diskutil apfs list
bless --info
nvram -p
```

On Intel Macs:

```bash
sudo nvram boot-args
```

On modern macOS, messing with boot args and system internals is more restricted.

Apple Silicon Macs are not just PCs with macOS.

They are iPhone-like in some security ideas, but still Mac-like in user workflow.

That tension defines modern macOS.

It is still a computer.

But it is less of a general-purpose “do anything to the OS” box than Tiger was.

---

## 17. Package management: Apple never solved this for nerds, so Homebrew did

Tiger users installed software from:

- `.dmg`
- `.pkg`
- drag-and-drop `.app`
- MacPorts
- Fink
- source builds
- Apple installers

Modern users still install lots of apps from `.dmg`.

This is funny.

Two decades later, normal Mac software distribution still often looks like:

```text
Download DMG
Open DMG
Drag app to Applications
Eject DMG
```

Very Mac.
Very weird.
Still works.

For developers and power users, Homebrew became the unofficial package manager.

```bash
brew install wget
brew install ffmpeg
brew install --cask visual-studio-code
```

On Apple Silicon:

```bash
/opt/homebrew/bin/brew
```

On Intel:

```bash
/usr/local/bin/brew
```

Check:

```bash
which brew
brew --prefix
```

This is an important modern difference.

Tiger-era Unix software on Mac often felt like “porting Unix to Apple”.

Modern Homebrew feels like “Mac is a mainstream developer Unix machine”.

The Mac became one of the default laptops for web, mobile, and cloud development.

That was not guaranteed in the Tiger days.

---

## 18. Virtualization: from Virtual PC to Apple Silicon hypervisor

In the Tiger era, virtualization was a different world.

PowerPC Macs could emulate x86 with Virtual PC, but it was slow.

Intel Macs changed everything.

Boot Camp appeared.
Parallels and VMware became practical.
Running Windows on a Mac became normal.

Then Apple Silicon changed everything again.

No more native x86 Windows via Boot Camp.

Now the modern model is:

- Apple Virtualization framework
- ARM Linux VMs
- ARM Windows VMs through third-party tools
- Rosetta for Linux in some Apple virtualization contexts
- containers through Linux VMs
- UTM
- Docker Desktop
- Colima
- Lima

Modern macOS includes a virtualization framework that developers can use.

Check CPU architecture:

```bash
uname -m
```

On Apple Silicon:

```text
arm64
```

A modern Mac is excellent for ARM development.

It is less convenient if your workflow assumes x86 virtualization.

So the evolution is not simply “newer is better”.

It is more specific:

```text
Tiger PowerPC:
great Mac, bad x86 virtualization

Intel macOS:
best compatibility era

Apple Silicon:
best performance/watt, weaker legacy x86 world
```

That is the honest version.

---

## 19. Developer tools: from Project Builder heritage to Xcode platform machine

Tiger-era development still had strong NeXT roots.

Cocoa development meant Objective-C, Interface Builder, nib files, delegates, AppKit, Foundation.

Modern macOS development has:

- Xcode
- Swift
- SwiftUI
- AppKit
- Catalyst
- notarization
- hardened runtime
- entitlements
- provisioning profiles
- sandboxing
- developer ID signing
- App Store review
- TestFlight for Mac apps

Build a tiny C program:

```bash
cat > hello.c <<'EOF'
#include <stdio.h>

int main(void) {
    printf("Hello from Darwin\n");
    return 0;
}
EOF

clang hello.c -o hello
./hello
```

Check linked libraries:

```bash
otool -L ./hello
```

Inspect binary architecture:

```bash
file ./hello
```

For Swift:

```bash
swift --version
```

Compile:

```bash
cat > hello.swift <<'EOF'
print("Hello from Swift on macOS")
EOF

swiftc hello.swift -o hello-swift
./hello-swift
```

The developer story became more powerful, but also more bureaucratic.

In Tiger, you could build something and run it.

Today, distributing a serious Mac app means understanding:

```text
codesign
notarization
stapling
sandbox entitlements
hardened runtime
Gatekeeper
privacy prompts
```

Example:

```bash
codesign --force --deep --sign "Developer ID Application: Your Name" MyApp.app
xcrun notarytool submit MyApp.zip --keychain-profile "notary-profile" --wait
xcrun stapler staple MyApp.app
```

This is not romantic.

But it is the modern Mac developer reality.

---

## 20. Shell environment: bash to zsh, but Unix survived

Tiger used bash as the default shell.

Modern macOS uses zsh by default.

Check:

```bash
echo $SHELL
```

List available shells:

```bash
cat /etc/shells
```

Apple moved away from shipping newer GPLv3 versions of some tools, so macOS command-line utilities can feel old compared with Linux.

Example:

```bash
bash --version
```

On macOS, system bash is historically old.

Many developers install newer tools with Homebrew:

```bash
brew install bash coreutils findutils gnu-sed grep
```

Then you get GNU-prefixed versions like:

```bash
gdate
gsed
ggrep
```

This is a good video point:

macOS is Unix-certified / Unix-like in daily use, but it is not Linux.

Commands are similar until suddenly they are not.

Example:

```bash
stat file.txt
```

BSD `stat` differs from GNU `stat`.

Same with:

```bash
sed
date
find
```

This is why scripts that work on Linux sometimes break on macOS.

The Mac is a Unix workstation, but it is its own Unix island.

---

## 21. Classic Mac compatibility: Apple keeps moving forward by cutting the past

Apple has a pattern.

They support transitions brilliantly for a while, then they cut.

Examples:

```text
68k -> PowerPC
Classic Mac OS -> Mac OS X
PowerPC -> Intel
32-bit -> 64-bit
Intel -> Apple Silicon
kernel extensions -> system extensions
OpenGL -> Metal
HFS+ -> APFS
```

This is both Apple's strength and Apple's cruelty.

Microsoft keeps ancient compatibility forever and suffers for it.

Apple cuts old things and forces the platform forward.

The result:

- cleaner platform
- less legacy baggage
- angry users
- broken old software
- simpler future for Apple

Tiger could still feel connected to classic Mac history.

Modern macOS feels more like the endpoint of multiple purges.

If you have old software, Tiger can be better than a modern Mac.

If you want security, battery life, performance, and modern frameworks, modern macOS wins easily.

---

## 22. What actually came from NeXTSTEP?

A lot.

Here are the important pieces.

### App bundles

```bash
ls /Applications/*.app | head
```

Apps as directories with structured metadata.

### Property lists

```bash
plutil -p ~/Library/Preferences/com.apple.finder.plist
```

Configuration as structured plist data.

### Objective-C runtime

```bash
otool -L /System/Library/Frameworks/Foundation.framework/Foundation
```

Cocoa and Foundation descend from NeXT frameworks.

The `NS` prefix still means NeXTSTEP.

Examples:

```text
NSString
NSArray
NSDictionary
NSObject
NSView
NSWindow
NSNotificationCenter
```

That is not trivia.  
That is living archaeology.

### Services

The system-wide Services idea is very NeXT.

### Workspace-style app model

Finder is not literally NeXT Workspace Manager, but the concept of app bundles, documents, drag-and-drop, and system services carries that heritage.

### Display / graphics philosophy

NeXT had Display PostScript. macOS moved through Quartz and PDF-like imaging ideas. Not the same implementation, but the philosophy of high-quality resolution-independent rendering is connected.

### Developer culture

Interface Builder came from NeXT.

Xcode inherited ideas from Project Builder and Interface Builder.

Modern Xcode is huge and sometimes annoying, but the lineage is real.

When you write macOS code and see `NS`, you are looking at NeXT's ghost.

---

## 23. What changed the most since Tiger?

Here is the practical answer.

### 1. The OS protects itself now

Tiger trusted the admin.

Modern macOS protects the OS even from admin mistakes.

Commands to show this:

```bash
csrutil status
ls -lO /System
mount
```

### 2. Filesystem became snapshot-based

Tiger:

```text
HFS+
simple boot volume
```

Modern:

```text
APFS
snapshots
system/data split
sealed system volume
```

Commands:

```bash
diskutil apfs list
tmutil listlocalsnapshots /
```

### 3. App compatibility became temporary, not permanent

Tiger still had bridges to the classic world.

Modern macOS keeps bridges only long enough to complete a transition.

Commands:

```bash
file /Applications/App.app/Contents/MacOS/App
lipo -info /Applications/App.app/Contents/MacOS/App
arch
```

### 4. Services moved from Unix-style scripts to managed plists

Commands:

```bash
launchctl list
launchctl print system
ls /Library/LaunchDaemons
ls ~/Library/LaunchAgents
```

### 5. Logging became structured

Old:

```bash
tail -f /var/log/system.log
```

Modern:

```bash
log stream
log show --last 10m
```

### 6. Security became identity-based

Old:

```text
Can the Unix user access it?
```

Modern:

```text
Is the app signed?
Is it notarized?
Does it have entitlement?
Did the user grant TCC permission?
Is it sandboxed?
Is the system volume sealed?
```

Commands:

```bash
codesign -dv --verbose=4 SomeApp.app
spctl --assess --verbose SomeApp.app
xattr -l SomeApp.app
```

### 7. Hardware became part of the OS design

Tiger supported PowerPC Macs.

Modern macOS is increasingly Apple Silicon-first.

Commands:

```bash
system_profiler SPHardwareDataType
system_profiler SPDisplaysDataType
sysctl -n machdep.cpu.brand_string 2>/dev/null || true
uname -m
```

## Final thought: Tiger was open-feeling, modern macOS is integrated

Tiger feels like a computer.

Modern macOS feels like a platform.

That sounds subtle, but it is the whole story.

Tiger:

```text
Unix workstation
Aqua interface
PowerPC/early Intel transition
admin-friendly
hackable
less secure
more local
```

Modern macOS:

```text
sealed OS
APFS snapshots
Apple Silicon
cloud integration
privacy prompts
signed apps
notarization
MDM
system extensions
less hackable
much more secure
```

I do not think modern macOS is simply “better”.

It is better for security, battery life, performance, app sandboxing, and system reliability.

But Tiger was better if you liked the feeling that the machine was fully yours. That is why retro Macs are still fun.

They remind us of a time when a personal computer felt more personal.

Modern macOS is more powerful. Tiger was more understandable.

And somewhere between those two things is the entire history of Apple.