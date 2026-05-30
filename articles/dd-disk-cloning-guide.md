---
title: "The Art of Cloning Disk Drives with dd: Powerful, Simple, and Dangerous"
description: "A practical guide to using the dd command for cloning disks, creating images, restoring backups, wiping drives, and avoiding painful data loss mistakes."
pubDate: 2026-05-29
tags: ["Linux", "dd", "Disk Cloning", "Backup", "Command Line", "macOS", "Storage", "Data Recovery"]
draft: false
---
What is the best way to learn something technical?

By doing it.

**And sometimes by breaking something first.**

That is exactly how I learned the `dd` command. I wanted to clone a drive, made one small mistake, confused the input and output parameters, and destroyed the contents of a disk.

That is the command line for you.

It gives you power, but it does not babysit you. If you tell it to overwrite a drive, it will overwrite the drive. It will not ask whether you are emotionally ready for the consequences.

The `dd` command is one of those tools that looks simple, but can do serious damage if used without attention.

At the same time, it is extremely useful.

You can use it to:

- Clone a full disk
- Clone a partition
- Create a disk image
- Restore an image to a drive
- Wipe a drive with zeros
- Create compressed backups
- Work with old machines, Linux systems, macOS, rescue environments, and external drives

So let’s talk about it properly.

---

## What Is the dd Command?

`dd` is a classic command-line utility used for low-level copying and conversion of data.

In simple words, it can copy data from one place to another.

But unlike a normal file copy, `dd` can work directly with disks, partitions, images, and special device files.

That makes it powerful.

It also makes it dangerous.

The most common use cases are:

- Byte-to-byte disk cloning
- Partition cloning
- Creating disk images
- Restoring disk images
- Wiping drives
- Creating compressed backups

Some people jokingly call `dd` a “data destroyer.”

That joke exists for a reason.

If you provide the wrong destination, the data on that destination can be gone.

---

## Is dd Installed by Default?

On most Linux systems, yes.

On macOS, yes.

On Windows, not usually in the same native way, but there are similar tools and ports. You can also use Linux environments, live USB systems, WSL for some tasks, or dedicated disk imaging tools depending on what you need.

In this article, I will mostly use Linux-style examples.

The general logic is similar on other Unix-like systems, but device names and some options may differ.

On Linux, disks usually look like this:

```bash
/dev/sda
/dev/sdb
/dev/nvme0n1
```

On macOS, disks usually look like this:

```bash
/dev/disk0
/dev/disk1
/dev/rdisk1
```

This difference is important.

Never blindly copy commands from an article and execute them without checking your actual disk names.

That is how accidents happen.

---

## The Basic dd Syntax

The basic syntax looks like this:

```bash
sudo dd if=/path/to/source of=/path/to/destination
```

The two most important parameters are:

```bash
if=
```

Input file. This is the source.

```bash
of=
```

Output file. This is the destination.

For example:

```bash
sudo dd if=/dev/sda of=/dev/sdb
```

This means:

- Read from `/dev/sda`
- Write to `/dev/sdb`

If `/dev/sdb` contains important data, it will be overwritten.

`dd` does not care.

It simply does what you asked.

---

## Important Warning Before You Continue

Before using `dd`, understand this clearly:

**The destination will be overwritten.**

If you run this:

```bash
sudo dd if=/dev/sda of=/dev/sdb
```

Then `/dev/sdb` is not receiving “some files.”

It is receiving a raw copy of `/dev/sda`.

The filesystem, partition table, bootloader, empty space, deleted file remnants, everything goes as raw data.

That is exactly why `dd` is useful.

And that is exactly why it is dangerous.

Before running a command, check:

- What is the source?
- What is the destination?
- Are you sure the destination is the drive you want to erase?
- Are any important drives connected?
- Do you have a backup?
- Did device names change after reboot or reconnecting USB drives?

If you are tired, distracted, or in a hurry, do not use `dd`.

Seriously.

---

## How to Check Your Drives on Linux

On Linux, the easiest command is:

```bash
lsblk
```

It shows disks, partitions, sizes, and mount points.

Example output may look like this:

```bash
NAME        SIZE TYPE MOUNTPOINT
sda       465.8G disk
├─sda1      512M part /boot
└─sda2    465.3G part /
sdb       931.5G disk
└─sdb1    931.5G part /media/backup
nvme0n1   476.9G disk
├─nvme0n1p1 512M part
└─nvme0n1p2 476G part
```

This is the moment where you slow down and read carefully.

Do not guess.

Look at the size. Look at the mount point. Look at the drive model if needed.

You can also use:

```bash
sudo fdisk -l
```

Or:

```bash
sudo parted -l
```

On macOS, use:

```bash
diskutil list
```

Again, the goal is simple: know exactly what you are copying from and what you are copying to.

---

## Cloning a Full Disk

The most classic use case is cloning one entire disk to another disk.

Example:

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=16M status=progress
```

Let’s break this down:

```bash
sudo
```

Runs the command with administrator privileges.

```bash
dd
```

Runs the disk copy utility.

```bash
if=/dev/sda
```

Source disk.

```bash
of=/dev/sdb
```

Destination disk.

```bash
bs=16M
```

Block size. This can make the process faster.

```bash
status=progress
```

Shows progress while copying.

This command creates a raw clone of `/dev/sda` onto `/dev/sdb`.

The destination drive should usually be the same size or larger than the source drive.

If the destination is smaller, the copy may fail or produce an unusable result.

If the destination is larger, the cloned partition layout may not use all available space immediately. You may need to expand the partition later using tools like GParted.

---

## Cloning a Single Partition

You do not always need to clone the entire disk.

Sometimes you only want one partition.

Example:

```bash
sudo dd if=/dev/sda1 of=/dev/sdb1 bs=16M status=progress
```

This copies partition `/dev/sda1` to partition `/dev/sdb1`.

This is useful when you only care about one partition, not the bootloader or full disk layout.

But be careful: copying a partition is not always enough to make a system bootable. Bootloaders, EFI partitions, partition tables, and UUIDs may matter.

For simple data partitions, this works well.

For bootable systems, think twice.

---

## Creating a Disk Image

You can create an image file from a disk.

Example:

```bash
sudo dd if=/dev/sda of=backup.img bs=16M status=progress
```

This creates a raw image named `backup.img`.

The image contains a byte-level copy of the disk.

You can store it on another drive and restore it later.

You can also create an image from a partition:

```bash
sudo dd if=/dev/sda1 of=partition-backup.img bs=16M status=progress
```

This is useful for:

- Backing up old drives
- Preserving retro systems
- Creating restore points
- Making images before risky experiments
- Saving a known working system state

For retro computers and old operating systems, this can be extremely useful. If you are playing with Windows 98, Windows 2000, Linux experiments, or old laptop drives, having an image can save you from painful reinstallations.

---

## Restoring a Disk Image

Restoring is the reverse operation.

Example:

```bash
sudo dd if=backup.img of=/dev/sda bs=16M status=progress
```

This writes the image back to the disk.

Again, warning:

**Everything on `/dev/sda` will be overwritten.**

You can also restore a partition image:

```bash
sudo dd if=partition-backup.img of=/dev/sda1 bs=16M status=progress
```

This is simple, but dangerous.

Before restoring, run:

```bash
lsblk
```

Check the destination again.

Then check it again.

Then run the command.

---

## How to Make dd Faster

By default, `dd` may use a small block size, which can make copying slower.

You can specify a bigger block size with `bs`.

Example:

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=16M status=progress
```

I often use `16M` as a reasonable value on modern machines.

Other common values are:

```bash
bs=4M
bs=8M
bs=16M
bs=64M
```

Bigger does not always mean better.

Too small can be slow.

Too large may not give you any benefit and may even behave worse depending on hardware, memory, USB adapters, and storage type.

For most modern systems, `4M` or `16M` is a good starting point.

---

## Showing Progress

Old versions of `dd` were famously silent.

You could run a huge copy operation and stare at an empty terminal, wondering whether anything was happening.

Modern GNU `dd` supports:

```bash
status=progress
```

Example:

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=16M status=progress
```

This shows progress while the copy is running.

At the end, you will also see summary information.

If your version does not support `status=progress`, there are other methods, but on modern Linux systems this option is usually available.

---

## Making Sure Data Is Written to Disk

After `dd` finishes, it is a good idea to run:

```bash
sync
```

This makes sure pending writes are flushed to disk.

Example:

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=16M status=progress
sync
```

This matters especially when working with USB drives or external disks.

Do not just unplug the drive immediately because the command “looks finished.”

Let the system finish writing.

---

## Creating a Compressed Backup

Raw disk images can be huge.

If you create an image of a 500 GB drive, the image may be 500 GB, even if only 80 GB is actually used.

You can compress the output using `gzip`.

Example:

```bash
sudo dd if=/dev/sda bs=16M status=progress | gzip -c > backup.img.gz
```

This reads the disk and compresses the image into `backup.img.gz`.

To restore it:

```bash
gunzip -c backup.img.gz | sudo dd of=/dev/sda bs=16M status=progress
```

This is useful when storing backups, especially if the disk contains a lot of empty space.

But remember: compression takes CPU time, so the process may be slower depending on the machine.

---

## Using dd with pv for Better Progress

Another nice option is using `pv`, which shows progress in a more pleasant way.

First install it if needed:

```bash
sudo apt install pv
```

Then use it like this:

```bash
sudo dd if=/dev/sda bs=16M | pv | sudo dd of=/dev/sdb bs=16M
```

Or when creating a compressed image:

```bash
sudo dd if=/dev/sda bs=16M | pv | gzip -c > backup.img.gz
```

`pv` is not required, but it makes long operations more comfortable.

If you use `status=progress`, you may not need `pv`.

But it is good to know both options.

---

## Wiping a Drive with Zeros

You can use `dd` to overwrite a drive with zeros.

Example:

```bash
sudo dd if=/dev/zero of=/dev/sdb bs=16M status=progress
```

Here:

```bash
/dev/zero
```

is a special device that produces zero bytes.

This command writes zeros to the destination drive.

It can be useful when:

- Cleaning a disk before reuse
- Removing old partition tables
- Preparing a drive for a fresh setup
- Destroying old data in a basic way

But do not confuse this with advanced secure erase for all situations.

For SSDs, modern storage, and sensitive data, you may need proper secure erase tools, encryption, or manufacturer-specific utilities.

For a normal “wipe this old test drive” scenario, zeroing can be enough.

For serious security, think deeper.

---

## Copying Only Part of a Disk

Sometimes you may want to copy only a certain amount of data.

The `count` parameter can help.

Example:

```bash
sudo dd if=/dev/sda of=mbr-backup.img bs=512 count=1
```

This copies only the first 512 bytes.

Historically, this could be used for backing up the MBR on older systems.

Another example:

```bash
sudo dd if=/dev/sda of=first-gigabyte.img bs=1M count=1024
```

This copies the first 1 GB of the disk.

This is more advanced, but useful in specific forensic, recovery, or troubleshooting scenarios.

If you are not sure why you need this, you probably do not need it.

---

## Skipping Errors with conv=noerror,sync

Sometimes you may be working with a damaged disk.

By default, read errors can interrupt the process.

You may see examples like this:

```bash
sudo dd if=/dev/sda of=backup.img bs=16M conv=noerror,sync status=progress
```

The options mean:

```bash
conv=noerror
```

Continue after read errors.

```bash
conv=sync
```

Pad blocks so the output stays aligned.

This can help when trying to copy from a failing drive.

But if the disk is physically dying and the data is important, `dd` is not always the best tool.

For damaged drives, `ddrescue` is usually a better choice.

It is designed specifically for data recovery scenarios and can retry problem areas more intelligently.

So the practical rule is:

- Healthy disk: `dd` can be fine
- Damaged disk: consider `ddrescue`

---

## dd vs Clonezilla vs GParted

It is worth saying this directly.

`dd` is not always the best tool.

If you want a more user-friendly cloning experience, Clonezilla may be better.

If you want to resize partitions, GParted is better.

If you want to rescue data from a failing disk, `ddrescue` is better.

So why use `dd`?

Because it is simple, universal, and available almost everywhere.

It is excellent when you understand exactly what you want:

- Copy this disk to that disk
- Create a raw image
- Restore this image
- Wipe this device
- Backup this partition

But it is not friendly.

It will not protect you from yourself.

---

## Mistakes to Avoid

Here are the mistakes that matter most.

### 1. Confusing if and of

This is the classic disaster.

Wrong:

```bash
sudo dd if=/dev/empty-drive of=/dev/important-drive
```

Correct idea:

```bash
sudo dd if=/dev/source of=/dev/destination
```

Always remember:

- `if` = source
- `of` = destination

### 2. Trusting device names blindly

Today your USB drive is `/dev/sdb`.

Tomorrow it may be `/dev/sdc`.

After rebooting or reconnecting drives, names can change.

Always check with:

```bash
lsblk
```

### 3. Keeping unnecessary drives connected

If possible, disconnect drives you do not need.

This reduces the chance of overwriting the wrong disk.

### 4. Running old commands from terminal history

Do not press the up arrow and run an old `dd` command without checking it.

That is a lazy way to create a disaster.

### 5. Using dd while tired

This sounds funny, but it is serious.

Disk cloning is not the best task to do while tired, distracted, or after a few drinks.

The command is short.

The damage can be large.

---

## A Safer Workflow

Here is a practical workflow I like:

1. Connect only the drives you need.
2. Run `lsblk`.
3. Identify source and destination by size and mount point.
4. Unmount mounted partitions if needed.
5. Write the `dd` command manually.
6. Re-read the command before pressing Enter.
7. Use `status=progress`.
8. Wait for completion.
9. Run `sync`.
10. Only then disconnect the drive.

Example:

```bash
lsblk
sudo dd if=/dev/sda of=/dev/sdb bs=16M status=progress
sync
```

Simple.

But do not confuse simple with harmless.

---

## Practical Examples

### Clone one disk to another

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=16M status=progress
sync
```

### Clone one partition to another

```bash
sudo dd if=/dev/sda1 of=/dev/sdb1 bs=16M status=progress
sync
```

### Create a disk image

```bash
sudo dd if=/dev/sda of=backup.img bs=16M status=progress
sync
```

### Restore a disk image

```bash
sudo dd if=backup.img of=/dev/sda bs=16M status=progress
sync
```

### Create a compressed image

```bash
sudo dd if=/dev/sda bs=16M status=progress | gzip -c > backup.img.gz
```

### Restore a compressed image

```bash
gunzip -c backup.img.gz | sudo dd of=/dev/sda bs=16M status=progress
sync
```

### Wipe a disk with zeros

```bash
sudo dd if=/dev/zero of=/dev/sdb bs=16M status=progress
sync
```

### Backup the first 512 bytes

```bash
sudo dd if=/dev/sda of=mbr-backup.img bs=512 count=1
```

---

## When I Would Use dd

I would use `dd` when I need a raw, direct, predictable copy.

For example:

- Cloning an old laptop drive
- Creating an image before experimenting
- Backing up a retro OS installation
- Copying a USB drive
- Wiping a test disk
- Restoring a known working image

I would not use it blindly for every backup task.

For normal personal backups, file-level backup tools can be better. They are easier to inspect, easier to restore partially, and usually safer.

`dd` is great when you need a disk-level copy.

It is not always great when you just need your documents backed up.

---

## Conclusion

The `dd` command is one of the most powerful old-school Unix tools.

It can clone disks, create images, restore backups, wipe drives, and help you preserve entire systems exactly as they are.

But it is also unforgiving.

One wrong parameter can destroy data.

That does not mean you should avoid it forever. It means you should respect it.

Use `lsblk`. Check the source. Check the destination. Disconnect unnecessary drives. Use `status=progress`. Run `sync`. Do not rush.

The command itself is simple.

The responsibility is the hard part.

And that is probably the best description of `dd`: simple, powerful, and dangerous enough to teach you discipline.
