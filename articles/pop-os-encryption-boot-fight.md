---
title: "Pop!_OS, Encryption, and the Boot Fight: Fixing a Linux System That Refused to Start"
description: "A practical story about troubleshooting Pop!_OS with encrypted root, Btrfs subvolumes, systemd-boot, kernelstub, EFI entries, and the infamous missing rootflags=subvol=@ boot option."
pubDate: 2026-06-04
tags: ["Linux", "Pop!_OS", "Btrfs", "LUKS", "Encryption", "systemd-boot", "kernelstub", "UEFI", "Bootloader", "Troubleshooting"]
draft: false
---

Pop!_OS is one of those Linux distributions that looks friendly at first. The installer is polished, NVIDIA support is usually better than on many other distributions, and full-disk encryption is not hidden behind some “advanced expert mode” screen.

But the moment you stop using the default layout and start doing something more serious — encrypted root, Btrfs subvolumes, separate EFI partitions, dual boot, recovery entries, systemd-boot, and kernel updates — the friendly surface disappears very quickly.

This article is based on my own fight with Pop!_OS: the system was installed, encrypted, **technically alive, but sometimes it simply refused to boot correctly**. The problem looked scary at first, but the actual cause was more specific and more interesting.

It was not “Linux encryption is broken”.

It was not “Btrfs is bad”.

It was not even “Pop!_OS cannot boot”.

The real problem was that the boot entry did not always know how to mount the correct Btrfs subvolume.

In my case, the missing detail was:

```bash
rootflags=subvol=@
```

That one option was the difference between a working encrypted Pop!_OS system and a boot failure.

---

## The Setup

The machine was using Pop!_OS 24.04 with a more advanced disk layout than a basic beginner install.

The important pieces were:

- Pop!_OS 24.04
- encrypted root using LUKS
- Btrfs filesystem
- root moved to the `@` subvolume
- `systemd-boot` instead of GRUB
- Pop!_OS `kernelstub`
- EFI system partition mounted at `/boot/efi`
- Windows also present on another EFI path / boot option

This kind of setup is not exotic for Linux enthusiasts, but it is already beyond the “just click next” world.

A simple Pop!_OS encrypted install is usually fine. The installer knows what it created. The bootloader knows where to look. The initramfs knows how to unlock the encrypted root.

The trouble starts when you customize the filesystem layout after installation, especially with Btrfs subvolumes.

---

## Why Encryption Makes Boot Problems Feel Worse

When a normal unencrypted Linux install fails to boot, you often get direct access to the filesystem from a live USB. You mount the partition, edit a config file, update the bootloader, and move on.

With encryption, there is one more gate in front of everything.

Before you can fix anything, you need to unlock the encrypted volume:

```bash
sudo cryptsetup luksOpen /dev/nvme1n1p6 cryptroot
```

Then you mount the root filesystem from the decrypted mapper device:

```bash
sudo mount /dev/mapper/cryptroot /mnt
```

But with Btrfs subvolumes, that may still not be enough.

If your real root is inside the `@` subvolume, you need to mount it explicitly:

```bash
sudo mount -o subvol=@ /dev/mapper/cryptroot /mnt
```

That distinction matters a lot.

Without the correct subvolume, you may be looking at the top-level Btrfs volume instead of the actual installed system. It feels like the install is broken or missing, but it may simply be mounted from the wrong place.

---

## The Real Problem: Btrfs Subvolume Was Not Passed to the Kernel

After moving root to a Btrfs subvolume called `@`, the system needs to be told about it during boot.

The kernel command line needs this option:

```bash
rootflags=subvol=@
```

Without it, the kernel and initramfs may unlock the encrypted disk correctly, find the Btrfs filesystem, and still fail to mount the right root.

That is the nasty part.

You can have the correct passphrase.

You can have a healthy LUKS container.

You can have a healthy Btrfs filesystem.

You can have a valid EFI entry.

You can have `systemd-boot` installed.

And the system can still fail because it is trying to mount the wrong Btrfs root.

This is why the issue felt like a bigger disaster than it really was.

---

## Pop!_OS Does Not Use GRUB by Default

A lot of Linux boot repair guides assume GRUB.

They tell you to run:

```bash
sudo update-grub
```

or edit:

```bash
/etc/default/grub
```

But on modern UEFI Pop!_OS installs, that is usually not the main path.

Pop!_OS normally uses:

```bash
systemd-boot
```

and manages kernel boot options through:

```bash
kernelstub
```

That means the important files are often in the EFI loader entries:

```bash
/boot/efi/loader/entries/
```

A typical boot entry may look like this:

```text
title Pop!_OS
linux /EFI/Pop_OS-xxxx/vmlinuz.efi
initrd /EFI/Pop_OS-xxxx/initrd.img
options root=UUID=... ro quiet splash loglevel=0 systemd.show_status=false
```

For my Btrfs subvolume setup, the options line also needed:

```text
rootflags=subvol=@
```

So the entry should contain something closer to:

```text
options root=UUID=... ro quiet splash rootflags=subvol=@
```

The exact line can vary, but the idea is the same: the boot entry must tell the kernel which Btrfs subvolume is the real root.

---

## The Annoying Part: Kernel Updates Can Create New Entries

The system may boot after you fix one loader entry.

Then a kernel update happens.

A new Pop!_OS boot entry appears.

And the new entry may not contain the option you manually added before.

So you boot again, and suddenly the system is broken again.

That was the part that made the problem feel cursed.

The fix was not just “edit one file once”. The real fix was understanding where Pop!_OS stores boot options and making sure the correct option survives kernel updates.

The command that matters here is:

```bash
sudo kernelstub --add-options "rootflags=subvol=@"
```

This tells Pop!_OS to add that boot option properly instead of relying only on manual edits.

After that, it is also reasonable to rebuild initramfs:

```bash
sudo update-initramfs -u -k all
```

And check the current boot command line after a successful boot:

```bash
cat /proc/cmdline
```

If the running system was booted correctly, you should see:

```text
rootflags=subvol=@
```

somewhere in the output.

---

## Recovery From a Live USB

The recovery workflow is not beautiful, but it is logical.

Boot from a Pop!_OS or Ubuntu live USB.

Unlock the encrypted root:

```bash
sudo cryptsetup luksOpen /dev/nvme1n1p6 cryptroot
```

Mount the real root subvolume:

```bash
sudo mount -o subvol=@ /dev/mapper/cryptroot /mnt
```

Mount the EFI partition:

```bash
sudo mount /dev/nvme1n1p3 /mnt/boot/efi
```

Bind system directories:

```bash
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run
```

Enter the installed system:

```bash
sudo chroot /mnt
```

Add the missing boot option:

```bash
kernelstub --add-options "rootflags=subvol=@"
```

Rebuild initramfs:

```bash
update-initramfs -u -k all
```

Check systemd-boot:

```bash
bootctl status
```

If needed, reinstall or update systemd-boot:

```bash
bootctl install
bootctl update
```

Then exit and reboot:

```bash
exit
sudo reboot
```

This is the clean version of the fight. In reality, it can involve a lot of confusion: wrong mounts, missing `/dev/pts` in chroot, EFI paths that do not look obvious, and boot entries that point to recovery instead of the main system.

---

## The `/dev/pts` Problem in Chroot

One of the more annoying live USB problems is this kind of error:

```text
sudo: unable to allocate pty: No such device
```

This usually means the chroot environment is incomplete. You mounted some system directories, but not everything required for pseudo-terminals.

The fix is usually to make sure `/dev`, `/proc`, `/sys`, and `/run` are mounted before chrooting:

```bash
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run
```

If `/dev/pts` is still missing, mount it directly:

```bash
sudo mount -t devpts devpts /mnt/dev/pts
```

This is the kind of small Linux detail that can waste a lot of time because it looks unrelated to the original boot problem.

---

## EFI Confusion: Windows, Pop!_OS, and Old Bootloaders

Another layer of the fight was EFI confusion.

On a dual-boot system, you may have:

- Windows Boot Manager
- Linux Boot Manager
- Pop!_OS boot entry
- old Ubuntu or GRUB folders
- recovery entries
- fallback EFI paths

On some machines, the firmware keeps preferring Windows. On some HP laptops, the firmware may even rewrite the boot order back to Windows.

You can inspect boot entries with:

```bash
sudo efibootmgr
```

You can inspect systemd-boot with:

```bash
sudo bootctl status
```

And if Pop!_OS keeps booting into recovery by default, you can check the loader entries in:

```bash
/boot/efi/loader/entries/
```

Sometimes the fix is not reinstalling Linux. Sometimes it is simply choosing the correct default entry.

For example:

```bash
sudo bootctl set-default Pop_OS-current.conf
```

The exact filename may differ, so always list the entries first:

```bash
ls /boot/efi/loader/entries/
```

The important lesson: do not blindly reinstall GRUB on a Pop!_OS system unless you really mean to switch bootloader strategies. Pop!_OS is designed around systemd-boot on UEFI systems.

---

## Why This Is Not Really a “Pop!_OS Is Bad” Problem

It is tempting to blame Pop!_OS completely.

And yes, Pop!_OS can be frustrating. The combination of systemd-boot, kernelstub, recovery entries, NVIDIA handling, COSMIC changes, and custom disk layouts can make troubleshooting feel less standard than on Ubuntu or Debian.

But the deeper issue is this:

Linux boot is a chain.

Every part has to agree with the next part.

The chain looks roughly like this:

```text
UEFI firmware
  ↓
EFI boot entry
  ↓
systemd-boot
  ↓
loader entry
  ↓
kernel + initramfs
  ↓
LUKS unlock
  ↓
Btrfs mount
  ↓
correct subvolume
  ↓
system starts
```

If any piece has wrong information, boot fails.

In my case, the missing piece was not the passphrase, not the disk, not the kernel, not Btrfs itself. It was the instruction that says:

```text
Mount the @ subvolume as root.
```

That instruction is:

```bash
rootflags=subvol=@
```

---

## What I Would Do Differently Next Time

After fighting with this setup, I would be more careful with a few things.

First, I would decide on the filesystem layout before installation. Moving root into a Btrfs subvolume after the fact is possible, but it creates more room for bootloader mismatch.

Second, I would document the exact disk layout immediately:

```bash
lsblk -f
```

```bash
findmnt /
```

```bash
findmnt /home
```

```bash
sudo btrfs subvolume list /
```

Third, I would save the current kernel command line:

```bash
cat /proc/cmdline
```

Fourth, I would inspect Pop!_OS boot entries after every major kernel or bootloader change:

```bash
ls /boot/efi/loader/entries/
```

```bash
sudo bootctl status
```

Fifth, I would avoid mixing GRUB advice into a systemd-boot setup unless I intentionally want to migrate to GRUB.

Random bootloader commands from random Linux guides can make the situation worse.

---

## Quick Checklist for Pop!_OS + LUKS + Btrfs Boot Problems

If Pop!_OS does not boot after an encrypted Btrfs setup, check these things first.

### 1. Can you unlock the encrypted disk?

```bash
sudo cryptsetup luksOpen /dev/nvme1n1p6 cryptroot
```

If this fails, the problem is before Btrfs and before systemd-boot.

### 2. Can you mount the Btrfs root subvolume?

```bash
sudo mount -o subvol=@ /dev/mapper/cryptroot /mnt
```

If mounting without `subvol=@` shows a different layout, that is a major clue.

### 3. Does the boot entry contain `rootflags=subvol=@`?

```bash
grep -R "rootflags" /boot/efi/loader/entries/
```

If it is missing, add it through kernelstub:

```bash
sudo kernelstub --add-options "rootflags=subvol=@"
```

### 4. Does the running system really use the option?

After booting:

```bash
cat /proc/cmdline
```

Look for:

```text
rootflags=subvol=@
```

### 5. Is Pop!_OS booting the recovery entry by default?

Check:

```bash
sudo bootctl status
```

List entries:

```bash
ls /boot/efi/loader/entries/
```

Set the correct default if needed:

```bash
sudo bootctl set-default Pop_OS-current.conf
```

### 6. Did the firmware choose the wrong EFI entry?

Check:

```bash
sudo efibootmgr
```

If the firmware always prefers Windows, the issue may be UEFI boot order, not the Linux install itself.

---

## Final Thoughts

This kind of problem is why Linux is both powerful and annoying.

The system gives you the tools to build almost any layout you want: encrypted root, Btrfs subvolumes, separate EFI partitions, dual boot, custom boot options, snapshots, recovery entries, everything.

But the price is that you are responsible for the chain.

Pop!_OS makes the default path easy, but once you step outside that path, you need to understand how the boot process actually works.

My biggest takeaway from this fight is simple:

When encrypted Pop!_OS with Btrfs refuses to boot, do not panic and do not immediately reinstall.

Check the boot chain.

Check the EFI entry.

Check systemd-boot.

Check `kernelstub`.

Check the initramfs.

Check whether LUKS unlocks.

Check whether Btrfs mounts.

And if your real root lives in `@`, check for the one option that can make or break the whole system:

```bash
rootflags=subvol=@
```

Sometimes the system is not dead.

Sometimes it is just looking in the wrong place.
