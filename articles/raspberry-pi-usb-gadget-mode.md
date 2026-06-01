---
title: "Quick Guide: Setting up Raspberry Pi in USB Gadget Mode"
description: "A short practical guide for debugging Raspberry Pi connection issues in a gadget mode."
pubDate: 2026-05-27
tags: ["Raspberry Pi", "Gadget Mode", "Raspberry Pi Debug", "Raspbian", "Kali Linux"]
draft: false
---

USB gadget mode is one of the most useful Raspberry Pi tricks, especially on Raspberry Pi Zero, Zero W, Zero 2 W, and some other models that support USB OTG/device mode. Instead of connecting the Pi to a monitor, keyboard, Wi-Fi, or Ethernet adapter, you connect it to your computer with a USB cable and make it appear as a USB Ethernet device.

In the ideal case, you plug the Pi into your computer and SSH into it through a virtual network interface. In the real world, it often half-works: the USB device appears, but SSH does not connect; the interface gets a `169.254.x.x` address; `ping` fails; or Linux shows `NO-CARRIER`.

This guide is a practical checklist for enabling, testing, and debugging Raspberry Pi USB Ethernet gadget mode.

---

## What USB Gadget Mode Does

In normal USB mode, your Raspberry Pi acts as a **USB host**. You plug keyboards, mice, storage devices, or network adapters into it.

In USB gadget mode, the Raspberry Pi acts as a **USB device**. Your computer sees it as something like:

```text
Linux-USB Ethernet/RNDIS Gadget
USB Ethernet
CDC Ethernet
RNDIS/Ethernet Gadget
```

That virtual USB network card can then be used for:

- SSH access without Wi-Fi
- Headless setup
- Direct file transfer
- Internet sharing from the host computer
- Portable field/lab setups
- Network experiments and packet capture labs

---

## Supported Raspberry Pi Models

USB Ethernet gadget mode is most commonly used with:

- Raspberry Pi Zero
- Raspberry Pi Zero W
- Raspberry Pi Zero 2 W

It can also work on some other models with USB-C or USB OTG/device support, but the setup may differ. On a Pi Zero/Zero 2 W, make sure you use the **data USB port**, not the power-only port.

Typical Pi Zero layout:

```text
[mini HDMI] [USB] [PWR IN]
```

Use the port labeled:

```text
USB
```

Do **not** use:

```text
PWR IN
```

The power port can power the board, but it will not create a USB Ethernet device.

---

## Cable Warning: Many Micro-USB Cables Are Charge-Only

This is one of the most common failures.

A charge-only cable can power the Pi but cannot carry data. If the Pi powers on but the computer never detects a new USB device, suspect the cable first.

On Linux, run this on the host computer:

```bash
lsusb
```

After connecting the Pi, you should see a new device, often similar to:

```text
Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget
```

If nothing new appears in `lsusb`, check:

- wrong port on the Pi
- charge-only cable
- bad cable
- Pi not booting
- boot configuration issue

---

## Basic Manual Setup

After flashing your Raspberry Pi image, open the boot partition on your computer.

Depending on the operating system image, the boot files may be in one of these locations:

```text
/boot
/boot/firmware
```

When editing from another computer, you usually see the boot partition directly as a small FAT partition.

### 1. Edit `config.txt`

Add this line:

```ini
dtoverlay=dwc2
```

If the file already has another overlay, such as:

```ini
dtoverlay=vc4-kms-v3d
```

do not replace it. Add `dwc2` on a separate line:

```ini
dtoverlay=vc4-kms-v3d
dtoverlay=dwc2
```

### 2. Edit `cmdline.txt`

Find `cmdline.txt`.

This file must stay as **one single line**.

Add this after `rootwait`:

```text
modules-load=dwc2,g_ether
```

Example:

```text
console=serial0,115200 console=tty1 root=PARTUUID=xxxxxxxx-02 rootfstype=ext4 fsck.repair=yes rootwait modules-load=dwc2,g_ether
```

Do not add line breaks.

---

## First Boot and SSH

Insert the microSD card into the Raspberry Pi.

Connect the Pi to your computer using the **USB data port**.

Wait for the Pi to boot. First boot can take longer than expected.

Try:

```bash
ssh user@raspberrypi.local
```

or, depending on the image:

```bash
ssh user@kali.local
ssh user@hostname.local
```

The username and password depend on the image you installed. Raspberry Pi OS, Kali, DietPi, and other images do not always use the same defaults.

---

## What Should Happen on the Host Computer

On Linux, check interfaces:

```bash
ip a
```

You should see something like:

```text
usb0
enx...
```

The name depends on your distro and NetworkManager settings.

A healthy physical USB link may look like:

```text
usb0: <BROADCAST,MULTICAST,UP,LOWER_UP>
```

A broken or not-yet-ready link may look like:

```text
usb0: <NO-CARRIER,BROADCAST,MULTICAST,UP> state DOWN
```

`LOWER_UP` means the physical USB network link is active.

`NO-CARRIER` means the interface exists, but the link is not active.

---

## Understanding the Common IP Problem

USB gadget mode creates a network link, but that does not automatically mean both sides have usable IP addresses.

You need:

```text
Host computer: some IP address
Raspberry Pi: another IP address in the same subnet
```

For example:

```text
Host: 10.42.0.1/24
Pi:   10.42.0.2/24
```

or:

```text
Host: 192.168.7.1/24
Pi:   192.168.7.2/24
```

The exact numbers do not matter. What matters is that both devices are in the same subnet.

---

## If You See `169.254.x.x`

Example:

```text
inet 169.254.87.112/16
```

This is a link-local address. It usually means the interface is up, but DHCP did not provide an address.

This is not necessarily fatal, but it often explains why SSH does not work.

You can either:

1. use `.local` hostname resolution, if mDNS works;
2. configure static IP addresses;
3. use NetworkManager internet sharing on the host;
4. configure DHCP on one side.

---

## Linux Host Method 1: NetworkManager Shared Mode

On many Linux desktops, the simplest host-side setup is NetworkManager shared mode.

First check the interface name:

```bash
nmcli dev status
```

Assume the USB interface is `usb0`.

Create a shared connection:

```bash
sudo nmcli con add type ethernet ifname usb0 con-name pi-usb ipv4.method shared ipv6.method ignore
sudo nmcli con up pi-usb
```

This usually gives the host an address like:

```text
10.42.0.1/24
```

Then search for the Pi:

```bash
ip neigh show dev usb0
```

or:

```bash
sudo nmap -sn 10.42.0.0/24
```

If the Pi gets an address, SSH into it:

```bash
ssh user@10.42.0.x
```

To remove this connection later:

```bash
sudo nmcli con down pi-usb
sudo nmcli con delete pi-usb
```

This should not break your main Wi-Fi or Ethernet internet connection if you only apply it to `usb0`.

---

## Linux Host Method 2: Manual Static IP on the Host

You can manually assign an address to the host side:

```bash
sudo ip addr flush dev usb0
sudo ip addr add 192.168.7.1/24 dev usb0
sudo ip link set usb0 up
```

Then try the Pi address you configured:

```bash
ping 192.168.7.2
ssh user@192.168.7.2
```

If you did not configure the Pi to use `192.168.7.2`, this will fail. The host and Pi both need addresses in the same subnet.

---

## Configuring a Static IP on the Raspberry Pi Side

Sometimes the USB Ethernet gadget appears on the host, but the Pi never gets an IP address. In that case, configure the Pi side manually.

Mount the root filesystem partition of the microSD card on another Linux machine.

Do not edit the small boot partition for this part. You need the main Linux root filesystem.

Depending on the image, the root filesystem may mount as something like:

```text
/media/username/rootfs
/media/username/kali-root
/run/media/username/rootfs
```

### Option A: NetworkManager Connection File

For systems that use NetworkManager, create:

```text
/etc/NetworkManager/system-connections/usb0.nmconnection
```

Example:

```ini
[connection]
id=usb0
type=ethernet
interface-name=usb0
autoconnect=true

[ipv4]
method=manual
address1=10.42.0.2/24,10.42.0.1
dns=1.1.1.1;8.8.8.8;

[ipv6]
method=ignore
```

Set permissions:

```bash
sudo chmod 600 /path/to/rootfs/etc/NetworkManager/system-connections/usb0.nmconnection
sudo chown root:root /path/to/rootfs/etc/NetworkManager/system-connections/usb0.nmconnection
sync
```

Then boot the Pi and try:

```bash
ping 10.42.0.2
ssh user@10.42.0.2
```

### Option B: `/etc/network/interfaces.d/usb0`

Some images may use classic Debian networking.

Create:

```text
/etc/network/interfaces.d/usb0
```

Example:

```ini
auto usb0
allow-hotplug usb0
iface usb0 inet static
    address 192.168.7.2
    netmask 255.255.255.0
    gateway 192.168.7.1
```

Then configure the host side as:

```bash
sudo ip addr flush dev usb0
sudo ip addr add 192.168.7.1/24 dev usb0
sudo ip link set usb0 up
```

Now test:

```bash
ping 192.168.7.2
ssh user@192.168.7.2
```

---

## Debugging Checklist

### 1. Does the host see the USB device?

```bash
lsusb
```

Good sign:

```text
Linux-USB Ethernet/RNDIS Gadget
```

If no new USB device appears:

- check the cable
- check the port
- check power
- check `dtoverlay=dwc2`
- check `modules-load=dwc2,g_ether`
- make sure `cmdline.txt` is one line

---

### 2. Does the host have a USB network interface?

```bash
ip a
```

Look for:

```text
usb0
enx...
```

If no interface appears, the USB gadget driver may not have loaded correctly, or the host may not recognize the gadget type.

---

### 3. Is the physical link up?

```bash
ip a show usb0
```

Good:

```text
LOWER_UP
```

Bad or not ready:

```text
NO-CARRIER
state DOWN
```

If you see `NO-CARRIER`:

- wait longer for boot
- reconnect the cable
- use the data USB port
- test another cable
- check `dmesg -w`

---

### 4. Does the host have an IP?

```bash
ip a show usb0
```

Possible results:

```text
inet 10.42.0.1/24
inet 192.168.7.1/24
inet 169.254.x.x/16
```

`169.254.x.x` often means DHCP failed.

---

### 5. Can the host see the Pi?

```bash
ip neigh show dev usb0
```

If you see:

```text
192.168.7.2 FAILED
```

that means the host tried to reach that IP but the Pi did not answer there.

This often means:

- the Pi has no IP on USB
- the Pi has a different IP
- the Pi is not fully booted
- the host and Pi are in different subnets
- the USB link dropped

---

### 6. Scan the USB subnet

If your host has:

```text
10.42.0.1/24
```

scan:

```bash
sudo nmap -sn 10.42.0.0/24
```

If your host has:

```text
192.168.7.1/24
```

scan:

```bash
sudo nmap -sn 192.168.7.0/24
```

---

### 7. Watch kernel logs while reconnecting

On the host:

```bash
sudo dmesg -w
```

Now unplug and reconnect the Pi.

You want to see the USB device appear and the network driver attach.

---

## Windows Notes

On Windows, the Pi should appear as a USB Ethernet/RNDIS network adapter.

Check:

```text
Device Manager → Network adapters
```

If it appears as a serial device or unknown device, Windows may not have selected the right driver.

Common Windows-side fixes include:

- update the driver manually
- choose an RNDIS-compatible driver
- check whether it appears under Network adapters
- disable/re-enable the adapter
- check whether the network profile is created

Windows Internet Connection Sharing can also be used, but the assigned IP range may not match the examples in this article.

---

## macOS Notes

On macOS, the Pi may appear as a new network service.

Check:

```text
System Settings → Network
```

You may need to approve or activate the new interface.

Try:

```bash
ifconfig
```

and:

```bash
ping raspberrypi.local
ssh user@raspberrypi.local
```

---

## Common Failure Patterns

### The Pi powers on, but nothing appears on the host

Likely causes:

- charge-only cable
- wrong USB port
- Pi not booting
- bad image
- bad `config.txt` / `cmdline.txt`

### USB device appears, but no `usb0`

Likely causes:

- host driver issue
- gadget driver mismatch
- Windows RNDIS problem
- OS image differences

### `usb0` exists but shows `NO-CARRIER`

Likely causes:

- Pi still booting
- cable issue
- link dropped
- wrong port
- gadget not fully initialized

### `usb0` has `169.254.x.x`

Likely causes:

- DHCP did not work
- no static IP configured
- host and Pi are not negotiating addresses

### `ping` fails and `ip neigh` shows `FAILED`

Likely causes:

- wrong target IP
- Pi has no IP on USB
- Pi has a different IP
- subnet mismatch
- firewall or network manager interference

### SSH says `No route to host`

Likely causes:

- host has no route to that subnet
- USB interface is down
- target IP is wrong
- Pi is not reachable at that address

---

## A Practical Known-Good Static Setup

Use this when you want fewer moving parts.

Host computer:

```text
192.168.7.1/24
```

Raspberry Pi:

```text
192.168.7.2/24
```

Host commands:

```bash
sudo ip addr flush dev usb0
sudo ip addr add 192.168.7.1/24 dev usb0
sudo ip link set usb0 up
```

Pi `/etc/network/interfaces.d/usb0`:

```ini
auto usb0
allow-hotplug usb0
iface usb0 inet static
    address 192.168.7.2
    netmask 255.255.255.0
    gateway 192.168.7.1
```

Test:

```bash
ping 192.168.7.2
ssh user@192.168.7.2
```

---

## Another Practical Setup: NetworkManager Shared Mode

Use this when you want the host computer to share internet with the Pi.

Host:

```bash
sudo nmcli con add type ethernet ifname usb0 con-name pi-usb ipv4.method shared ipv6.method ignore
sudo nmcli con up pi-usb
```

Host will commonly become:

```text
10.42.0.1/24
```

Pi NetworkManager file:

```ini
[connection]
id=usb0
type=ethernet
interface-name=usb0
autoconnect=true

[ipv4]
method=manual
address1=10.42.0.2/24,10.42.0.1
dns=1.1.1.1;8.8.8.8;

[ipv6]
method=ignore
```

Test:

```bash
ping 10.42.0.2
ssh user@10.42.0.2
```

---

## Final Advice

USB gadget mode is simple in theory but annoying in practice because there are three separate layers:

1. USB detection
2. virtual Ethernet interface creation
3. IP addressing and SSH

Do not debug all three at once.

Follow this order:

```text
lsusb
ip a
LOWER_UP vs NO-CARRIER
host IP
Pi IP
ping
ssh
```

If the USB device is not visible, do not waste time on IP addresses.

If the interface has `NO-CARRIER`, do not waste time on SSH.

If the interface has only `169.254.x.x`, fix addressing.

If `ip neigh` says `FAILED`, the Pi is not answering at that IP.

Once you separate these layers, Raspberry Pi gadget mode becomes much easier to troubleshoot.
