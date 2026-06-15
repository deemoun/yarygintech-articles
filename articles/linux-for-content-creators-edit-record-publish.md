---
title: "Linux for Content Creators: Can You Actually Edit, Record, and Publish on Linux?"
description: "A practical look at using Linux for YouTube, blogging, screen recording, video editing, audio work, thumbnails, and publishing without pretending everything is perfect."
pubDate: "2026-06-15"
tags: ["linux", "content-creators", "youtube", "video-editing", "obs-studio", "kdenlive", "davinci-resolve", "flatpak", "open-source", "desktop-linux", "creator-tools"]
draft: false
---
Linux for content creation is one of those topics where people usually lie in two opposite directions.

One group says Linux is perfect, everything works, and if something does not work then you are just lazy or not “technical enough.”

**The other group says Linux is useless for creators because it does not have Adobe Premiere Pro, Final Cut Pro, Photoshop, or some trendy AI video editor with a monthly subscription and a startup landing page.**

Both views are too simple.

The real answer is more interesting:

**Yes, you can create serious content on Linux.**

You can record videos. You can edit videos. You can make thumbnails. You can write articles. You can publish to YouTube, Medium, your own website, Substack, GitHub Pages, WordPress, or whatever else you use.

But there are trade-offs.

Linux can be a great creator workstation if your workflow fits the tools. It can also be annoying if your entire workflow depends on proprietary apps that simply do not exist on Linux.

So let’s talk about it like normal people.

Not as Linux marketing.

Not as anti-Linux drama.

Just the practical reality of making content on Linux in 2026.

## The Short Answer

Can you edit, record, and publish on Linux?

Yes.

Should every creator switch to Linux?

No.

Linux is good for creators who:

- record tutorials
- make programming or cybersecurity videos
- create Linux/tech content
- edit normal YouTube videos
- write articles and blog posts
- make thumbnails with open-source tools
- use OBS heavily
- like stable, scriptable workflows
- do not depend on Adobe tools
- do not need every trendy AI creator app

Linux is not ideal for creators who:

- live inside Adobe Creative Cloud
- need Final Cut Pro
- depend on Descript, CapCut desktop, ScreenFlow, or Camtasia-style workflows
- collaborate with teams using Premiere project files
- need very specific commercial plugins
- work with professional color pipelines and client delivery standards
- want everything to behave like macOS without touching settings

That is the honest answer.

Linux can absolutely be a creator system.

But it is not a magical replacement for every commercial creative workflow.

## Why Linux Is Actually Interesting for Creators

The funny thing is that Linux is not “behind” in every creator category.

For some workflows, it is genuinely excellent.

If you create technical content, Linux is almost perfect:

- terminal recording
- programming tutorials
- cybersecurity labs
- Android testing
- Docker demos
- networking tools
- server work
- web development
- automation
- open-source software reviews
- retro computing experiments

This is where Linux feels natural.

You are already using the terminal. You already care about files, scripts, packages, tools, and reproducible setup. You are probably not trying to make a cinematic wedding video with 500 LUTs and a paid transition pack called “Epic Zoom Glitch Pack 9.”

For tech creators, Linux is not just usable. It can be the best environment.

You can record the same system you are teaching. You can show commands directly. You can run servers locally. You can automate parts of your workflow. You can write scripts to process videos, compress files, upload assets, rename screenshots, generate thumbnails, or publish articles.

That is real value.

Linux is not just a cheaper macOS replacement.

It is a different kind of workstation.

## Recording Video: OBS Is the King

For screen recording and streaming, Linux is in a pretty good place because OBS Studio exists.

OBS is one of the most important creator tools on any operating system, not just Linux. It works on Windows, macOS, and Linux. It can record your screen, capture windows, use webcams, manage scenes, mix audio sources, apply filters, stream to YouTube or Twitch, and do most of what a technical creator needs.

On Linux, OBS is usually the first tool I would install.

For YouTube tutorials, it is enough for:

- screen recording
- webcam overlay
- microphone capture
- scene switching
- browser capture
- intro/outro scenes
- recording terminal sessions
- recording app demos
- live streaming

If you are making Linux tutorials, cybersecurity videos, coding videos, QA automation lessons, or Android testing content, OBS is not just enough. It is probably the main tool.

The modern Linux desktop also improved a lot because of PipeWire. Screen capture on Wayland used to be a disaster zone. It is still not perfect everywhere, but it is much better than it used to be.

Still, there are things to watch for:

- Wayland screen sharing may behave differently depending on your desktop environment
- Nvidia drivers can still be annoying
- audio routing can be confusing
- some OBS plugins are easier on Windows
- virtual camera support may require extra setup
- Flatpak OBS plugin support can be different from native package installs

This is the Linux creator pattern in one sentence:

**The main tool works, but the edges can be weird.**

That is not a reason to avoid Linux. It is just something to know before you promise yourself a “simple setup.”

## Video Editing: Kdenlive Is the Practical Default

For open-source video editing on Linux, Kdenlive is the practical default.

It is not perfect. It can have bugs. It can crash. It can sometimes feel like a very powerful tool assembled by people who care deeply about video editing but occasionally forget that normal humans also use computers.

But it is useful.

For normal YouTube editing, Kdenlive can handle a lot:

- cutting clips
- arranging timelines
- adding titles
- adding music
- adding voiceover
- basic effects
- transitions
- speed changes
- proxy clips
- color correction
- audio adjustments
- subtitles
- rendering/exporting

For tech videos, tutorials, talking-head content, screen recordings, and basic documentary-style editing, Kdenlive is good enough.

And “good enough” is not an insult.

For content creation, good enough means:

- you can finish the video
- you can export it
- you can repeat the workflow
- you do not fight the tool every single day
- the final result looks normal to viewers

Most viewers do not care if you edited in Premiere, Final Cut, DaVinci Resolve, Kdenlive, or a toaster.

They care if the video is useful, watchable, and not painful.

Kdenlive is especially nice if you already like KDE apps or if you want something powerful without paying for a commercial editor.

But you need to test your own workflow.

Do not move your entire channel to Linux because someone online said Kdenlive is “basically Premiere.”

It is not.

It is Kdenlive.

That can be enough. But it is not the same thing.

## What About DaVinci Resolve on Linux?

DaVinci Resolve is the big serious option.

It is professional. It is powerful. It is used in real production environments. It has editing, color grading, Fusion effects, Fairlight audio, and a much more polished professional feel than most open-source editors.

And yes, it exists for Linux.

But here is the catch: DaVinci Resolve on Linux is not the same casual experience as installing Kdenlive from Flathub and moving on.

Resolve on Linux is more picky.

It officially targets specific enterprise-style Linux environments, especially Rocky Linux. It expects proper GPU drivers. It wants a serious workstation. It may not behave nicely on your random old laptop with Intel graphics and a distro you installed because it had a cool wallpaper.

This is where people get disappointed.

They hear:

> DaVinci Resolve supports Linux.

Then they imagine:

> Great, I will install it on any Linux distro like a normal app.

Not exactly.

Resolve on Linux is best if you are building a dedicated editing workstation with compatible hardware. If you have a strong desktop, a good GPU, and you are willing to follow the supported environment, it can be amazing.

If you are on a random ThinkPad, old MacBook, or lightweight Linux laptop, Kdenlive is probably the saner choice.

My practical view:

- casual YouTube editing: Kdenlive first
- serious color work: DaVinci Resolve if your hardware and distro fit
- lightweight edits: Shotcut or OpenShot can work
- quick cuts and conversion: FFmpeg and LosslessCut are useful
- old hardware: keep expectations low

DaVinci Resolve on Linux is real.

But it is not the beginner-friendly path.

## Shotcut, OpenShot, Olive, and the Other Editors

Kdenlive and Resolve get most of the attention, but they are not the only options.

Shotcut is a solid cross-platform editor. It is often simpler than Kdenlive and can be enough for basic editing. Some people prefer it because it feels less tied to the KDE ecosystem.

OpenShot exists and is easy to understand, but I personally would not build a serious YouTube workflow around it unless your editing needs are very basic.

Olive has always been interesting, but I would treat it carefully for production work. Cool project, not always the tool you want when you need to publish today.

Then there are command-line tools.

FFmpeg is boring, ugly, and incredibly powerful. If you create content on Linux and never learn even basic FFmpeg, you are missing out.

You can use it to:

- convert files
- compress videos
- extract audio
- merge clips
- cut sections
- generate thumbnails
- normalize formats
- prepare uploads
- batch process recordings

FFmpeg is not a replacement for a timeline editor.

It is a power tool for all the annoying tasks around editing.

For example, converting a video to a YouTube-friendly MP4:

```bash
ffmpeg -i input.mov -c:v libx264 -crf 20 -preset medium -c:a aac -b:a 192k output.mp4
```

Or extracting audio:

```bash
ffmpeg -i video.mp4 -vn audio.wav
```

A creator on Linux should not be afraid of this stuff.

This is one of the reasons Linux can be very strong: you can combine GUI apps with command-line automation.

## Audio Recording and Editing

Audio matters more than video.

People will forgive a slightly ugly screen recording. They will not forgive painful audio for very long.

On Linux, audio is much better than it used to be.

The old PulseAudio/JACK pain is slowly being replaced by PipeWire, which is a more modern audio and video routing system. For normal users, this means audio routing can be less horrible than before, especially on modern distributions.

For audio tools, you have several good options:

- Audacity for simple recording and editing
- Ardour for more serious audio production
- REAPER if you want a commercial DAW with Linux support
- EasyEffects for noise reduction, EQ, compression, and live audio processing
- Tenacity if you prefer a community fork style tool
- Helvum or qpwgraph for routing PipeWire audio

For YouTube voiceover work, Audacity is still the simple default.

Record voice. Clean noise. Normalize. Compress. Export.

That is enough for many creators.

For real music production, Linux is possible but more complicated. You need to care about audio interfaces, latency, plugins, JACK/PipeWire routing, VST support, and project compatibility.

For a YouTuber doing tech content?

You can absolutely make good audio on Linux.

The basic setup:

- decent USB microphone
- OBS or Audacity
- EasyEffects if needed
- quiet room
- consistent recording distance
- some basic compression and noise reduction

Do not overcomplicate it.

A $70 microphone used correctly beats a $400 microphone used in a room that sounds like a bathroom.

## Thumbnails and Graphics

This is where some creators start to miss Photoshop.

But Linux is not helpless.

For thumbnails and graphics, the main tools are:

- GIMP
- Krita
- Inkscape
- Blender
- Photopea in the browser
- Canva in the browser
- Figma in the browser
- Penpot
- Darktable and RawTherapee for photo work

GIMP is the obvious Photoshop-like option. It is powerful, but the workflow is different. If you expect it to behave exactly like Photoshop, you will hate it. If you learn it as its own tool, it is usable.

Krita is excellent for painting and illustration. It can also be useful for thumbnail work if you like a more drawing-oriented workflow.

Inkscape is great for vector graphics, logos, diagrams, arrows, icons, and clean visual elements.

For YouTube thumbnails, you can absolutely use Linux.

The question is not “can Linux make thumbnails?”

The question is:

> Are you willing to build a thumbnail workflow around tools that are not Photoshop?

For many people, the easiest answer is actually browser-based tools.

Use Photopea or Canva in the browser if that gets the job done faster.

There is no moral prize for doing everything in native open-source apps.

The goal is to publish.

## Writing Articles and Scripts

Linux is excellent for writing.

This is one of the easiest parts.

You can use:

- VS Code
- VSCodium
- Obsidian
- Zettlr
- Ghostwriter
- Typora
- LibreOffice
- OnlyOffice
- Vim
- Emacs
- any browser-based CMS

If your website is based on Markdown files, Linux is almost perfect.

You can write the article in a text editor, preview it locally, commit it to GitHub, deploy through Netlify, and automate parts of the process.

This is where Linux feels cleaner than macOS or Windows.

Not because the apps are magically better, but because the whole system is comfortable for file-based workflows.

For example:

```bash
git add content/articles/linux-for-content-creators.md
git commit -m "Add Linux for content creators article"
git push
```

That is a nice publishing workflow.

It is boring in the best way.

## Publishing to YouTube and the Web

Publishing itself is mostly operating-system independent.

YouTube Studio works in the browser.

WordPress works in the browser.

Medium works in the browser.

Substack works in the browser.

Netlify works from GitHub.

Most modern publishing platforms do not care if you are on Linux, Windows, or macOS.

The problems are usually around the files before upload:

- did the editor export correctly?
- is the codec accepted?
- is the file too large?
- is audio synced?
- did the browser upload fail?
- is your thumbnail the right size?
- did you keep the original project files?

For final video export, HandBrake is very useful. It can convert and compress video files into modern formats. If your editor exports something too large or weird, HandBrake can clean it up.

For batch jobs, FFmpeg is even better.

For thumbnails, browser tools or GIMP/Krita/Inkscape are enough.

So yes, publishing from Linux is not the problem.

The bigger question is whether your creation pipeline before publishing is comfortable.

## The Biggest Problems for Creators on Linux

Now let’s stop pretending everything is perfect.

Linux has real problems for creators.

### No Adobe Creative Cloud

This is the obvious one.

No native Premiere Pro. No After Effects. No Photoshop. No Lightroom Classic. No Illustrator.

There are alternatives, but they are not drop-in replacements.

If your entire professional life is built around Adobe files and client workflows, Linux is probably not your main creative OS.

### No Final Cut Pro

Final Cut is macOS-only.

If you like Final Cut, use a Mac.

Do not waste your life trying to recreate that exact workflow on Linux.

### AI Creator Apps Are Often Missing

Many newer creator tools focus on macOS, Windows, iOS, or web first.

Linux support is usually not the priority.

This matters if you use tools like:

- AI transcription editors
- AI video clipping tools
- social media repurposing apps
- eye contact correction
- automatic podcast editors
- commercial teleprompter suites

Some of this works in the browser. Some does not.

If your workflow depends on one specific app, check before switching.

### Hardware Acceleration Can Be Annoying

Video editing loves hardware acceleration.

Linux can use GPU acceleration, but it depends on:

- GPU brand
- driver version
- distro
- app
- codec
- Wayland vs X11
- Flatpak vs native package
- export settings

This is where macOS often feels easier.

Apple controls the hardware and software. Linux is more flexible, but also more chaotic.

### Color Management Is Not Always Simple

If you do serious photography, print work, or color grading, Linux can be more work.

For casual YouTube content, this is probably fine.

For professional color-critical workflows, test carefully.

### Plugin Ecosystem Is Smaller

A lot of commercial video/audio plugins are built for Windows and macOS first.

Linux has plugins, but not always the ones people expect.

Again: it depends on your workflow.

### The “One Annoying Thing” Problem

Linux often works until one very specific thing becomes annoying.

Maybe your capture card works differently.

Maybe your Bluetooth microphone is weird.

Maybe your webcam has the wrong exposure.

Maybe your editor exports slowly.

Maybe your thumbnail font is missing.

Maybe your Flatpak app cannot see the folder you want.

None of these problems are impossible.

But they cost time.

And creators need to protect their publishing rhythm.

## The Best Linux Creator Setup I Would Actually Use

For a practical creator workstation, I would not overcomplicate it.

I would use a mainstream distro.

Not some ultra-minimal Arch rice setup with 900 custom scripts and a window manager that requires a PhD in dotfiles.

For a creator machine, boring is good.

Good distro choices:

- Fedora Workstation
- Ubuntu LTS
- Linux Mint
- Pop!_OS
- openSUSE Tumbleweed if you like newer packages
- Debian Stable if you know what you are doing and use Flatpak for newer apps

Then install the basic creator stack:

```bash
flatpak install flathub com.obsproject.Studio
flatpak install flathub org.kde.kdenlive
flatpak install flathub org.gimp.GIMP
flatpak install flathub org.kde.krita
flatpak install flathub org.inkscape.Inkscape
flatpak install flathub fr.handbrake.ghb
flatpak install flathub org.audacityteam.Audacity
flatpak install flathub com.github.tchx84.Flatseal
```

Then keep system/dev tools native:

```bash
sudo apt install git ffmpeg curl wget
```

On Fedora:

```bash
sudo dnf install git ffmpeg curl wget
```

Depending on the distro, FFmpeg codec availability may vary. Some distros are more strict about patents and repositories. That is not Linux being broken; that is distribution policy.

For writing:

- VS Code or VSCodium
- Obsidian
- a Markdown editor
- Git

For browser publishing:

- Firefox
- Chrome or Chromium
- maybe Brave if that is your thing

For audio:

- Audacity
- EasyEffects
- qpwgraph if you need routing

For advanced video:

- DaVinci Resolve, but only if you are willing to build around its requirements

This setup is enough for many creators.

Not every creator.

But many.

## Linux for YouTubers

For YouTube, Linux can work very well.

Especially for these channel types:

- programming
- Linux tutorials
- cybersecurity
- Android testing
- QA automation
- retro computing
- open-source reviews
- software tutorials
- server/devops content
- privacy and security content
- educational screen recordings

For these, Linux is not a limitation. It is part of the content.

Your OS becomes the studio.

You can record the exact environment you are teaching. You can show real commands. You can run virtual machines. You can test tools. You can break stuff and fix it on camera.

This is much harder to fake on macOS or Windows.

But if your channel is more about cinematic vlogs, travel films, heavy motion graphics, or commercial client work, Linux may be less attractive.

Not because it cannot edit video.

Because the surrounding ecosystem is weaker.

## Linux for Bloggers and Writers

Linux is excellent for blogging.

If your writing workflow is Markdown, Git, static sites, or plain text, Linux is almost ideal.

You can:

- write in Markdown
- preview locally
- manage images
- optimize screenshots
- commit to Git
- deploy with Netlify
- automate slugs and frontmatter
- use shell scripts for repetitive tasks

This is the kind of creator workflow where Linux is genuinely better than people expect.

A writer does not need a MacBook Pro to type Markdown into a Git repo.

A fast old ThinkPad with Linux can be enough.

Even old MacBooks running Linux can become surprisingly good writing and publishing machines.

The key is not chasing fancy apps.

The key is building a repeatable publishing system.

## Linux for Podcasting

Linux can work for podcasting too.

The basic workflow is fine:

- record audio
- clean it up
- edit tracks
- export WAV/MP3
- publish through browser

Audacity is enough for simple podcasts.

Ardour or REAPER is better for more complex multi-track production.

The main risk is hardware.

Before committing, test your audio interface, microphone, headphones, and recording chain.

Do not discover five minutes before an interview that your interface has crackling audio because the buffer settings are wrong.

Linux podcasting is possible.

But test the setup before real work.

## Linux for Designers and Photographers

This is more complicated.

For basic graphic work, Linux is fine.

For thumbnails, diagrams, blog images, simple logos, and web graphics, GIMP, Krita, Inkscape, and browser tools are enough.

For serious photography, Darktable and RawTherapee are powerful. But if you live in Lightroom, switching will hurt.

For professional design work, lack of Adobe apps is a serious issue. Yes, alternatives exist. No, they are not always enough.

This is where the honest recommendation matters:

- hobbyist thumbnails: Linux is fine
- tech diagrams: Linux is great
- web graphics: Linux is fine
- professional Photoshop/Illustrator client workflow: maybe not
- Lightroom-based photo business: test before switching
- print/color-critical design: be careful

Linux can be a creative OS.

It is not automatically the best OS for every creative job.

## Laptop vs Desktop

For Linux content creation, hardware matters.

A desktop is easier.

You can choose a GPU, add storage, fix cooling, use multiple monitors, plug in audio gear, and avoid battery life drama.

A laptop is more convenient, but can be more annoying:

- sleep issues
- Nvidia hybrid graphics
- webcam quality
- microphone quality
- thermal throttling
- battery drain
- display scaling
- external monitor quirks

If you want a Linux creator laptop, I would prefer:

- Intel or AMD CPU
- 16 GB RAM minimum, 32 GB better
- good SSD
- decent cooling
- known Linux compatibility
- no exotic hardware
- USB-C ports that actually behave
- AMD or Intel graphics for simplicity, Nvidia if you really need it and accept driver work

Old laptops can be fine for writing, blogging, and light editing.

For serious video editing, do not romanticize weak hardware.

A 10-year-old laptop can write articles.

It may not enjoy 4K video timelines.

## Flatpak Is Very Useful Here

For creators, Flatpak makes a lot of sense.

Many creator apps are graphical desktop apps. That is exactly where Flatpak shines.

You can install newer versions without waiting for your distribution repository.

This is useful for:

- OBS
- Kdenlive
- GIMP
- Krita
- Inkscape
- HandBrake
- Audacity
- VLC
- Blender
- Bottles

But remember the rule from the Flatpak article:

Use Flatpak for desktop apps. Keep system tools native when needed.

For example, I would not rely on Flatpak for everything around Android development, Docker, command-line automation, or low-level system tools.

For creator apps, yes.

For the whole workstation, no.

## A Realistic Linux Creator Workflow

Here is what a realistic Linux content workflow could look like.

### Step 1: Plan and Write

Use Obsidian, VS Code, or a Markdown editor.

Write the script, article, or outline.

### Step 2: Record

Use OBS for screen and camera.

Record microphone audio directly in OBS or separately in Audacity.

### Step 3: Edit

Use Kdenlive for normal YouTube edits.

Use DaVinci Resolve only if your system supports it properly.

### Step 4: Audio Cleanup

Use Audacity or EasyEffects.

Keep it simple: noise reduction, compression, normalization.

### Step 5: Thumbnail

Use GIMP, Krita, Inkscape, Photopea, or Canva.

Nobody cares if your thumbnail was made in Photoshop if it gets clicked.

### Step 6: Export and Compress

Export from the editor.

Use HandBrake or FFmpeg if needed.

### Step 7: Publish

Upload through the browser.

Commit the article to GitHub if your site is static.

Post the supporting content on socials.

### Step 8: Archive

Keep project files organized.

This is not glamorous, but it works.

Content creation is usually not about having the fanciest software.

It is about having a repeatable process.

## Where Linux Is Better Than Windows or macOS

Linux has some real advantages.

### It Is Scriptable

You can automate boring parts of the workflow.

Rename files, compress videos, move exports, generate folders, resize images, create thumbnails from frames, publish Markdown, sync backups.

Linux is great at this.

### It Is Lightweight

You can build a fast writing and editing environment without running a huge amount of background nonsense.

This matters on older machines.

### It Is Stable If You Keep It Boring

A boring Linux setup can be very reliable.

The problem is that Linux users often refuse to keep it boring.

They install 400 themes, switch kernels, add random repositories, test experimental desktops, and then blame Linux when something breaks.

For a creator machine, boring wins.

### It Is Great for Technical Content

If your content is about software, security, servers, QA, Android, Linux, or development, Linux gives you a natural lab.

You are not just using the OS.

You are creating inside the same environment you are explaining.

## Where Linux Is Worse

Linux is worse when you need the commercial creative ecosystem.

No serious person should deny this.

macOS is better for:

- Final Cut Pro
- Logic Pro
- polished audio/video workflows
- Apple hardware acceleration
- creator apps with nice UI
- color-managed creative workflows
- “it just works” laptop experience

Windows is better for:

- broad commercial software support
- many plugins
- gaming plus editing
- Nvidia-centric workflows
- more capture card/vendor support
- random creator tools from small companies

Linux is better for:

- technical creation
- scripting
- open-source workflows
- stable writing/publishing systems
- development-heavy channels
- privacy-focused workflows
- reviving older hardware
- avoiding subscriptions

Choose based on the work.

Not based on ideology.

## My Honest Recommendation

If you are a creator and curious about Linux, do not switch everything at once.

Start with a test workflow.

Install Linux on a second machine or external SSD.

Try to make one complete piece of content:

- write the script
- record the screen
- record audio
- edit the video
- make the thumbnail
- export
- upload
- publish the article

If you can finish one real project, Linux is viable for you.

If you spend the whole day fighting missing apps and weird exports, maybe Linux should stay as your secondary machine.

That is fine.

The goal is not to prove loyalty to Linux.

The goal is to publish content.

## Final Thoughts

Linux for content creators is not a fantasy anymore.

You can absolutely record, edit, write, design, and publish on Linux.

OBS is strong. Kdenlive is usable. DaVinci Resolve exists for serious setups. GIMP, Krita, Inkscape, Audacity, HandBrake, Blender, and FFmpeg cover a lot of ground. Browser-based tools fill many gaps.

For tech creators, Linux can be excellent.

For writers and bloggers, it can be almost perfect.

For YouTubers making tutorials, screen recordings, software reviews, cybersecurity labs, or programming content, Linux is more than enough.

But Linux is not the best answer for every creator.

If your work depends on Adobe, Final Cut, Descript, heavy motion graphics, commercial plugins, or very specific client workflows, be careful. You may be able to replace some tools, but you may not replace the whole ecosystem.

The realistic conclusion is simple:

**Linux is a good creator OS if you build your workflow around its strengths instead of pretending it is macOS with a penguin wallpaper.**

Use it where it makes sense.

Keep the setup boring.

Automate the boring parts.

Do not chase perfect tools.

And most importantly, actually publish.

That matters more than the operating system.

## Useful Links

- [OBS Studio](https://obsproject.com/)
- [OBS Studio on Flathub](https://flathub.org/apps/com.obsproject.Studio)
- [Kdenlive](https://kdenlive.org/)
- [Kdenlive Downloads](https://kdenlive.org/download/)
- [DaVinci Resolve](https://www.blackmagicdesign.com/products/davinciresolve)
- [GIMP](https://www.gimp.org/)
- [Krita](https://krita.org/)
- [Inkscape](https://inkscape.org/)
- [Audacity](https://www.audacityteam.org/)
- [HandBrake](https://handbrake.fr/)
- [FFmpeg](https://ffmpeg.org/)
- [Flatpak](https://flatpak.org/)
- [Flathub](https://flathub.org/)
