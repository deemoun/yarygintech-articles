---
title: "I Built Real Apps with AI, and Here Is What You Can Learn from It"
description: "A practical look at using AI tools to build real scripts, desktop apps, file managers, and browser extensions without turning the process into magic or hype."
pubDate: 2026-05-29
tags: ["App Development", "AI", "Linux"]
draft: false
---
*AI-assisted development is moving fast.*

I tried to stay away from it at first. It looked like another tech fad: loud, overhyped, and full of people pretending that prompting a chatbot made them senior engineers overnight.

But then I kept seeing how much people were building with so-called vibe coding. Some of it was garbage, sure. But some of it was genuinely useful. At some point, I stopped watching from the side and decided to test it properly myself.

And honestly, once you build a few real tools with AI, it becomes very hard to ignore what is happening.

A few years ago, even creating a simple Notepad clone or a To-Do app required a decent amount of programming effort. You had to think about the UI, file storage, event handling, packaging, small bugs, and all the boring glue code around the actual idea.

Now you can prototype something like that in twenty minutes and have a usable version by the next day.

Will it be the best app ever written? Of course not.

But you immediately feel the power. You get from idea to working prototype so quickly that it changes the way you think about small software projects.

---

## Where to Start

The best way to start is not by asking AI to build the next billion-dollar startup.

Start with tools that solve your own problems.

That sounds boring, but it is exactly where AI-assisted development makes the most sense. You need a real pain point. Something specific. Something annoying enough that you actually care if the tool works.

My first example was simple: I was moving away from the iOS ecosystem and needed a script that could sort my files and move all pictures into a separate folder. I wanted my exported photos and random images to be easier to manage.

That is how my first small script was born.

One prompt, one useful output.

Could I have written it myself? Yes, absolutely. But that was not the point. I wanted it fast. I wanted it without sitting there for an hour thinking about file extensions, folder paths, edge cases, and boring cleanup.

And I got exactly that.

![Image Relocator Script](image-relocator-script.png)

I also created a repository where I collect small useful scripts like this.

The next script I built listed available Android emulators on macOS and launched the selected one. That one became especially handy for testing.

![Script that scans for available emulators on Mac](mac-emulator-script.png)

Then I made another script that converts Markdown files into PDFs. Again, nothing revolutionary. But it worked perfectly for my workflow.

This is where the value becomes obvious.

You are not replacing engineering. You are removing friction.

Small tools that used to sit in the “maybe someday” pile can now become real in one evening.

But after a few scripts, I wanted more.

---

## Ramping Up to Real UI Work

Building scripts is nice, but I wanted to see what would happen if I tried to build an actual application with a proper UI.

My first candidate was obvious: a YouTube downloader.

That is how I created **ZV Tube** — basically a simple GUI around `yt-dlp`.

![ZV Tube interface](zv-tube-interface.png)

The idea was not to reinvent YouTube clients or create some giant media platform. I just wanted a clean desktop app that could search videos, download them, or play only the audio.

No bloat.

No ads.

No account system.

No modern app nonsense.

Just a small tool that does what I need.

The app worked perfectly for my use case. My friends liked it too. And that is important: I was not trying to create something abstract. I built a tool around my own actual workflow.

Technically, it is a WPF .NET application. Under the hood, it uses existing command-line tools. The value is not in pretending that I invented video downloading. The value is in wrapping something powerful into a small interface that feels comfortable to use.

That is one of the best use cases for AI-assisted development: build a personal front end for things you already use.

---

## Dual-Pane File Manager

After ZV Tube, I wanted to build something more desktop-like.

Sure, there are already tons of file managers for every operating system. But I wanted one for myself.

And my reason was extremely specific.

I wanted a quick way to create an empty file by pressing **Shift + F7**.

Sounds ridiculous? Maybe. But that was my real pain point with modern dual-pane file managers. They often have too many features, too much legacy behavior, or they make simple actions weirdly annoying.

Could I have modified an existing file manager? Probably.

But I did not want a giant project. I wanted my own tiny tool with the behavior I needed.

So I built **Damn Simple File Manager**.

![Damn Simple File Manager Interface](damn-simple-file-manager-interface.png)

It took me about three days to build the full functionality I wanted.

The app is intentionally simple. Two panels, basic navigation, file operations, keyboard shortcuts, and the small conveniences that matter to me.

Later, I also added the ability to bookmark specific paths and links, with export and import support for URLs.

![Link and Bookmarks Manager](link-and-bookmarks-manager.png)

No telemetry.

No online accounts.

No subscription.

No “AI-powered productivity dashboard.”

Just a lightweight, self-contained desktop tool.

And honestly, that is still one of the best feelings in software: using something you built because it solves your own problem exactly the way you wanted.

If you like the idea, feel free to try it and suggest improvements.

---

## Chrome Extension Challenge

Let’s be honest: dual-pane file managers are a niche category.

I wanted to try something different after that. Something smaller, browser-based, and useful for website work.

For some reason, I assumed Chrome extensions were kind of dead as a category. But then I realized I actually needed a simple tool for detecting broken links.

So I built one.

That is how **404 Doctor** was born.

![404 Doctor Chrome Extension](404-doctor-chrome-extension.png)

It was built in a couple of hours, and it works well for my needs.

Again, this is not magic. It is not some massive SaaS product. It is a small utility that does a clear job: helps detect broken links.

But this is exactly why AI-assisted development is interesting. It lets you create these small practical tools without turning every idea into a massive engineering project.

Maybe it will be useful for you too.

---

## What I Used for All of This

My “tech stack” was very simple.

It was basically five ingredients:

- **ChatGPT** for brainstorming ideas, planning features, and generating UI/image concepts.
- **Cursor** for refining drafts and improving code.
- **Codex** for pull requests, small fixes, and code tweaks.
- **My brain and experience** for writing better prompts and not accepting nonsense blindly.
- **My QA/testing background** for checking multiple scenarios instead of trusting the first happy path.

That last part matters a lot.

AI can generate code very quickly, but it does not magically understand your real workflow. It does not know what you forgot to test. It does not know which edge case will annoy you tomorrow.

This is where experience still matters.

You need to review the output. You need to run the app. You need to break it. You need to check what happens when the input is empty, when the folder is missing, when the command-line tool is not installed, when the file name is weird, when the user clicks the wrong button.

AI can speed up the build process. It does not remove responsibility.

And no, this did not make me a ton of money.

Actually, it made me zero money.

Not a cent. Not even a single donation so far.

But that is fine. The point of these experiments was not immediate profit. The point was to understand the workflow and see what I could build with it.

And after doing it several times, I am convinced that this approach can eventually lead to something people genuinely love.

---

## What You Can Learn from It

The biggest lesson is simple: stop thinking about AI only as a toy for generating fake startup ideas.

Use it to build tools.

Use it to remove annoying steps from your own workflow.

Use it to create small utilities you would never have had the energy to build before.

The best projects are often not huge. They are specific.

A script that sorts files.

A launcher for emulators.

A Markdown-to-PDF converter.

A desktop wrapper around a command-line tool.

A small file manager with exactly the shortcut you want.

A browser extension that checks broken links.

None of these ideas need to change the world. They just need to be useful.

That is where AI-assisted development becomes real.

---

## Conclusion

I do not believe everyone should become a programmer.

I also do not believe AI will solve every problem or magically turn bad ideas into good products.

But having this kind of power at your fingertips changes things.

Before, I could not imagine creating a usable desktop app in two or three days without burning out or getting stuck in boring glue code. It would usually take too much time for one person, especially when the project was just a personal tool.

Now it feels possible.

Not effortless. Not automatic. But possible.

That is the important shift.

AI-assisted development gives individuals more leverage. It lets you build useful, personal, strange, niche, lightweight tools that would previously stay forever in your notes.

And I think we are going to see a lot more of those.

I just hope not everything ends up hidden behind a paywall.