---
title: "Linux Network Monitoring Tools That Are Actually Worth Installing"
description: "A practical list of Linux terminal utilities for monitoring network usage, Wi-Fi quality, bandwidth, processes, and system information without turning your setup into a bloated mess."
tags: ["linux", "networking", "terminal", "sysadmin", "privacy", "tools"]
pubDate: 2026-06-09
draft: false
---
Linux has this funny problem.

There are too many tools.

You search for something simple like “how to monitor network usage on Linux” and suddenly people throw 40 utilities at you like you are building a data center in your kitchen.

I do not need that.

Most of the time I just want to know a few basic things:

- Is my internet actually working?
- Which app is eating my bandwidth?
- Is my Wi-Fi signal trash?
- How much traffic did this machine use today?
- Is my old laptop still doing fine or slowly dying in silence?

So this is not a “complete list of every Linux networking tool ever made”.

This is the useful list.

The kind of stuff I would actually install on a normal Linux laptop, especially on something like MX Linux, Debian, Ubuntu, or an old MacBook revived with Linux.

## Fastfetch: because the terminal needs some personality

Let’s start with the most important useless utility.

`fastfetch`.

Yes, technically it does not monitor your network. It does not fix anything. It does not debug packets. It does not make your Linux machine faster.

But it looks good.

And sometimes that is enough.

Fastfetch shows your system information in the terminal: distro, kernel, desktop environment, CPU, GPU, memory, disk, uptime, shell, resolution, and all that nice nerd stuff.

It is basically the modern replacement for `neofetch`.

Install it:

```bash
sudo apt install fastfetch
```

Run it:

```bash
fastfetch
```

I would not recommend forcing it to run every time you open the terminal. That sounds cool for two days and then becomes annoying.

Better make an alias:

```bash
echo "alias ff='fastfetch'" >> ~/.bashrc
source ~/.bashrc
```

Now you can just type:

```bash
ff
```

Small thing, but it makes your Linux setup feel more yours.

## btop: the best terminal system monitor

If you install only one system monitor, install `btop`.

It shows CPU, RAM, disks, processes, temperatures if available, and network activity. It looks clean, works fast, and is actually pleasant to use.

Install it:

```bash
sudo apt install btop
```

Run it:

```bash
btop
```

This is one of those tools that makes Linux feel powerful without making it feel ancient.

I still like `htop`, but `btop` is better for daily use.

## htop: still useful, still classic

`htop` is the classic process viewer.

It is not as pretty as `btop`, but it is everywhere, lightweight, and easy to understand.

Install it:

```bash
sudo apt install htop
```

Run it:

```bash
htop
```

If you are on some minimal system, server, old laptop, or rescue environment, `htop` is still great.

It is boring in a good way.

## nethogs: find the app eating your internet

This one is very useful.

`nethogs` shows network usage by process.

Not just “your machine is using 2 MB/s”.

It tells you which process is doing it.

Firefox?
Telegram?
Docker?
Some random updater?
A suspicious process you forgot about?

Install it:

```bash
sudo apt install nethogs
```

Run it:

```bash
sudo nethogs
```

This is probably one of the most practical network tools for normal desktop Linux.

Because when internet feels weird, you usually do not care about abstract interface statistics. You want to know what app is being greedy.

## nload: simple internet speedometer

`nload` is extremely simple.

It shows incoming and outgoing traffic in real time.

Install it:

```bash
sudo apt install nload
```

Run it:

```bash
nload
```

That is it.

No drama. No complicated UI. Just current traffic.

Good tool when you want to quickly check if your connection is idle or actively moving data.

## iftop: see network connections by host

`iftop` is like `top`, but for network connections.

It shows which hosts your machine is talking to and how much bandwidth they are using.

Install it:

```bash
sudo apt install iftop
```

Run it:

```bash
sudo iftop
```

This is more technical than `nethogs`.

`nethogs` says: this app is using internet.

`iftop` says: your machine is talking to this IP or host and using this much bandwidth.

Both are useful, just for different questions.

## vnstat: traffic history without the nonsense

Sometimes real-time monitoring is not enough.

You want to know how much traffic your machine used today, this week, or this month.

That is where `vnstat` is good.

Install it:

```bash
sudo apt install vnstat
```

Enable it:

```bash
sudo systemctl enable --now vnstat
```

Check usage:

```bash
vnstat
```

Daily usage:

```bash
vnstat -d
```

Monthly usage:

```bash
vnstat -m
```

This is useful if you are on mobile hotspot, limited internet, travel router, or just curious how much data your Linux machine burns.

It is not flashy. It just quietly keeps stats.

That is exactly what I want from a background utility.

## wavemon: Wi-Fi signal reality check

If you use Wi-Fi, install `wavemon`.

It shows signal level, link quality, bitrate, frequency, and other wireless information.

Install it:

```bash
sudo apt install wavemon
```

Run it:

```bash
wavemon
```

This is great when your internet feels slow but you are not sure if the problem is your ISP, router, Wi-Fi signal, or the laptop itself.

Sometimes the answer is very simple: your Wi-Fi signal is garbage.

Move the laptop two meters and suddenly everything works better.

Linux makes you feel like a hacker, but half of troubleshooting is still “move closer to the router”.

## bmon: clean interface monitoring

`bmon` gives you bandwidth monitoring per network interface.

Install it:

```bash
sudo apt install bmon
```

Run it:

```bash
bmon
```

It is useful when you have multiple interfaces: Wi-Fi, Ethernet, Docker, VPN, virtual adapters, maybe a phone tether.

You can see what is actually moving traffic.

Not my first tool, but good to have.

## iperf3: test your local network speed

`iperf3` is not for monitoring daily traffic.

It is for testing speed between two devices on your local network.

For example, you want to know if your Wi-Fi is actually fast between your laptop and another machine.

Install it:

```bash
sudo apt install iperf3
```

On one machine:

```bash
iperf3 -s
```

On another machine:

```bash
iperf3 -c 192.168.1.50
```

Replace the IP with the server machine IP.

This is useful because internet speed tests do not tell the whole story.

Your ISP can be fine while your local Wi-Fi is trash.

## tcpdump: when you want to get serious

`tcpdump` is the classic packet capture tool.

It is not friendly. It is not pretty. But it is powerful.

Install it:

```bash
sudo apt install tcpdump
```

Capture packets from all interfaces:

```bash
sudo tcpdump -i any
```

Capture only DNS traffic:

```bash
sudo tcpdump -i any port 53
```

Capture traffic to a file:

```bash
sudo tcpdump -i any -w capture.pcap
```

Then you can open that file later in Wireshark.

This is not something you need every day, but if you are doing QA, security testing, Android debugging, proxy troubleshooting, or weird network investigation, it is worth knowing.

## curl and jq: boring but essential

These two should be installed on every machine.

`curl` lets you make HTTP requests from the terminal.

`jq` lets you read and format JSON.

Install them:

```bash
sudo apt install curl jq
```

Example:

```bash
curl https://api.github.com | jq
```

For web testing, API testing, debugging, automation, and general Linux life, these are essential.

Not sexy. Just useful.

## traceroute and whois: old-school network debugging

Install them:

```bash
sudo apt install traceroute whois
```

Check network path:

```bash
traceroute example.com
```

Check domain/IP registration info:

```bash
whois example.com
```

You will not use these every day, but when you need them, you need them.

## My recommended install command

If I was setting up a fresh Linux laptop, I would install this:

```bash
sudo apt update
sudo apt install fastfetch btop htop nethogs nload iftop vnstat wavemon bmon iperf3 tcpdump curl jq traceroute whois
```

That gives you a nice mix:

- system info
- process monitoring
- bandwidth monitoring
- Wi-Fi diagnostics
- traffic history
- local network testing
- basic packet capture
- API/debugging tools

Not too much. Not too little.

## My actual top five

If you do not want to install everything, start with this:

```bash
sudo apt install fastfetch btop nethogs wavemon vnstat
```

That is the sweet spot.

`fastfetch` makes the machine feel nice.

`btop` shows system health.

`nethogs` catches bandwidth-hungry apps.

`wavemon` helps with Wi-Fi issues.

`vnstat` tracks traffic history.

For a desktop Linux user, that already covers most real-life situations.

## Final thoughts

Linux does not need to be complicated.

You do not need a giant monitoring stack just to understand what your laptop is doing.

A few good terminal tools are enough.

And honestly, this is one of the reasons I still like Linux on old machines.

You install some lightweight utilities, open the terminal, and suddenly a 10-year-old laptop feels like a real workstation again.

Not because it became new.

But because you can actually see what is happening.

That is the nice part.

Linux gives you control.

Sometimes too much control.

But with the right tools, it becomes useful instead of annoying.
