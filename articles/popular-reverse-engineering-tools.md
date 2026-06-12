---
title: "Popular Reverse Engineering Tools: IDA Pro, Ghidra, Frida, JADX and More"
description: "A practical overview of popular reverse engineering tools, what each one is good at, and how to choose the right tool for binary, Android, .NET, malware, and low-level analysis."
pubDate: 2026-06-11
tags: ["Cybersecurity", "App Analysis", "Ghidra", "IDA Pro"]
draft: false
---
Reverse engineering tools are not magic. They do not "recover the original source code" in some perfect Hollywood way. Most of the time they give you a better view of something ugly: machine code, bytecode, resources, memory, API calls, strings, control flow, imports, and behavior at runtime.

**That is still extremely useful.**

If you test mobile apps, inspect malware, audit closed-source binaries, analyze old software, or just want to understand what a program actually does, these tools are the basic toolbox. Some are huge professional platforms. Some are small utilities you run before opening the big tools. Some are ugly but powerful. Some are expensive because they save serious time.

This article is a practical overview of the most popular reverse engineering tools: what they are good for, where they are annoying, and when I would actually use them.

Small note before we start: use this stuff legally. Analyze your own apps, client-approved targets, malware samples in a lab, CTF binaries, old software you are allowed to inspect, or apps where you have permission. This is not a "how to crack paid software" article.

---

## The quick map

Reverse engineering usually splits into a few jobs:

| Job | Typical tools |
|---|---|
| Identify file type, compiler, packer, architecture | Detect It Easy, file, PE-bear, readelf, objdump |
| Static binary analysis | IDA Pro, Ghidra, Binary Ninja, Hopper, radare2, Rizin, Cutter |
| Native debugging | x64dbg, WinDbg, GDB, LLDB |
| Dynamic instrumentation | Frida |
| Android APK analysis | JADX, apktool, JEB, Frida |
| .NET / Unity analysis | ILSpy, dnSpyEx |
| Automation / scripting | IDA Python, Ghidra scripts, Binary Ninja API, r2pipe, Frida scripts |

A normal workflow is not "open IDA and win." It is usually:

1. Triage the file.
2. Check strings, imports, metadata, and file format.
3. Open it in a disassembler/decompiler.
4. Rename functions and variables as you understand them.
5. Debug or instrument the interesting parts.
6. Verify guesses with runtime behavior.

Static analysis gives you a map. Dynamic analysis tells you if your map is lying.

---

## IDA Pro

**Official site:** [hex-rays.com/ida-pro](https://hex-rays.com/ida-pro)

IDA Pro is still the classic professional reverse engineering tool. When people say "open it in IDA," they usually mean: load the binary, let IDA analyze it, browse functions, inspect cross-references, rename things, add comments, and slowly turn garbage assembly into something readable.

IDA is a disassembler, decompiler, and debugger. Its biggest strengths are mature analysis, processor support, the plugin ecosystem, and the fact that many experienced reverse engineers already know it.

### Typical usage

Use IDA when you need serious static analysis of a native binary:

- Windows PE malware
- Linux ELF binaries
- macOS Mach-O binaries
- firmware blobs
- old proprietary applications
- weird CPU architectures
- large binaries where navigation matters
- professional workflows where saving time matters more than tool cost

IDA is especially strong when you are spending hours or days with the same binary. You rename functions, create structs, mark data types, comment branches, follow cross-references, and gradually make the database readable.

### Specifics

IDA works around its database. Once you analyze a binary, your work is stored in the IDA database: names, comments, types, function boundaries, patches, enum definitions, struct definitions, and so on.

The decompiler is the famous Hex-Rays decompiler. It turns assembly into C-like pseudocode. That output is not original source code. It is an approximation. Still, for native reversing, good pseudocode can save a ridiculous amount of time.

IDA also supports scripting, mostly through IDAPython. This matters when you have repetitive work: decoding strings, fixing function names, applying types, scanning patterns, or extracting data.

### Where it shines

- Mature static analysis
- Big professional workflows
- Cross-references and navigation
- Large plugin ecosystem
- Lots of training material and community knowledge
- Good decompiler output for many targets

### Where it is annoying

- Expensive
- The UI is powerful but not exactly modern
- The learning curve is real
- Decompiler quality still depends heavily on architecture, compiler, optimizations, and symbols

### My take

If reverse engineering is part of your job, IDA is still one of the tools you need to know. You do not have to start with it, especially because Ghidra exists, but IDA knowledge transfers everywhere. Cross-references, control flow, function boundaries, imports, stack variables, calling conventions — these ideas are the real skill.

---

## Ghidra

**Official repository:** [github.com/NationalSecurityAgency/ghidra](https://github.com/NationalSecurityAgency/ghidra)

Ghidra is a free and open-source software reverse engineering framework created and maintained by the NSA Research Directorate. It includes disassembly, decompilation, graphing, scripting, and analysis tools for Windows, macOS, and Linux.

In plain English: Ghidra is the free tool that made serious decompilation accessible to a lot more people.

### Typical usage

Use Ghidra when you want a strong free platform for:

- native binary analysis
- malware triage
- CTF reversing
- firmware analysis
- learning reverse engineering
- comparing decompiler output against IDA or Binary Ninja
- team or classroom workflows where paid licenses are a problem

Ghidra is a very good default choice if you do not know what to install first.

### Specifics

Ghidra projects can contain multiple binaries. That is useful when you are analyzing related executables, libraries, or firmware components.

The decompiler is built in. You do not need to buy a separate module. The output is often very readable, but again, it is reconstructed pseudocode, not original source.

Ghidra scripting is usually Java or Python/Jython depending on what you are doing. It also has a headless mode, which is useful for batch analysis and automation.

### Where it shines

- Free and open source
- Built-in decompiler
- Good for learning and professional analysis
- Strong type and structure work
- Good automation options
- Cross-platform

### Where it is annoying

- Startup and analysis can feel heavy
- UI feels very Java, because it is
- Some workflows are slower than IDA/Binary Ninja once you get advanced
- You may need to tune analysis options for weird binaries

### My take

Ghidra is the first serious tool I would recommend to most people. It is not a toy. It is not just "free IDA." It has its own workflow, its own strengths, and enough power for real work.

If you teach reverse engineering, Android native analysis, malware basics, or firmware analysis, Ghidra is hard to ignore because students can actually install it without begging for licenses.

---

## Binary Ninja

**Official site:** [binary.ninja](https://binary.ninja/)

Binary Ninja is a commercial reverse engineering platform with a modern UI, decompiler, debugger, and strong API. It is built around intermediate languages: LLIL, MLIL, and HLIL. These representations help make binary analysis scriptable and easier to reason about.

### Typical usage

Use Binary Ninja when you want:

- clean interactive binary analysis
- a modern UI
- fast navigation
- strong Python automation
- intermediate-language-based analysis
- plugin development
- custom architecture or loader work

A lot of people like Binary Ninja because it feels less ancient than traditional tools.

### Specifics

The big idea is Binary Ninja's intermediate language pipeline. Instead of only looking at raw assembly or C-like pseudocode, you can inspect different abstraction layers.

That is useful when writing analysis scripts. Sometimes raw assembly is too low-level, and decompiled C is too high-level or too lossy. An intermediate representation gives you something structured but still close enough to the binary.

Binary Ninja also has a good API. If your workflow involves automation, custom analysis, plugins, or repeated research tasks, that matters.

### Where it shines

- Modern UX
- Strong scripting and API
- Nice graph and decompiler views
- Good for automation-heavy reversing
- Good middle ground between "friendly" and "deep"

### Where it is annoying

- Commercial license
- Smaller historical ecosystem than IDA
- Some niche architectures or old workflows may still be easier in IDA/Ghidra/radare2

### My take

Binary Ninja is great if you care about workflow speed and scripting. It is not always the first tool people learn, but it is one of the tools people grow into after they get tired of fighting older interfaces.

---

## Hopper Disassembler

**Official site:** [hopperapp.com](https://www.hopperapp.com/)

Hopper is a commercial disassembler/decompiler/debugger with a friendlier price point than IDA and a workflow that many macOS users like. It supports Python scripting and can analyze formats like Mach-O and Windows binaries.

### Typical usage

Use Hopper when you want:

- a lighter commercial disassembler
- macOS-focused reversing
- quick static analysis
- a GUI that is less intimidating than IDA
- binary patching and exploration
- learning assembly and executable structure

### Specifics

Hopper is very convenient for "open binary, explore, patch a few bytes, understand what is happening" style work. Its UI is approachable, and the official tutorial clearly frames it as a tool for turning bytes into something readable for humans.

It also supports editing binaries through its hex editor, assembling instructions, and Python scripts.

### Where it shines

- Nice macOS experience
- Lower cost than IDA
- Good for learning and quick analysis
- Python scripting
- Useful for Mach-O work

### Where it is annoying

- Not as dominant as IDA/Ghidra/Binary Ninja in professional reversing
- Smaller ecosystem
- Decompiler and analysis quality can vary depending on target

### My take

Hopper is a good practical tool, especially if you live on macOS and want something lighter than the big platforms. I would not make it my only reverse engineering tool, but I would not dismiss it either.

---

## radare2

**Official site:** [rada.re](https://rada.re/)

radare2 is a free reverse engineering toolkit built around command-line workflows. It can analyze binaries, disassemble code, debug programs, patch files, inspect file formats, and automate tasks through scripts or r2pipe.

It has a reputation: powerful, weird, and not always friendly.

### Typical usage

Use radare2 when you want:

- command-line binary analysis
- scriptable reversing
- quick triage over SSH
- automation pipelines
- CTF work
- file format inspection
- patching and hex-level work

### Specifics

radare2 is not just one GUI application. It is a toolkit with many small commands. You can load a binary, analyze it, search for strings, inspect functions, follow xrefs, debug, and patch.

The syntax can feel cryptic. For example, commands like `aaa`, `afl`, `pdf`, `iz`, and `axt` are normal in radare2 land. Once you learn the vocabulary, it becomes fast. Until then, it feels like the tool is hazing you.

Automation is one of its strengths. r2pipe lets you control radare2 from Python and other languages.

### Where it shines

- Free and open source
- Very scriptable
- Works well in terminals
- Good for automation and remote environments
- Wide file format and architecture support

### Where it is annoying

- Steep learning curve
- Commands are not self-explanatory
- GUI experience is not the main point
- Documentation exists, but you need patience

### My take

radare2 is worth learning if you like terminal tools, CTFs, automation, or low-level work. But do not pretend it is beginner-friendly. It is powerful after the pain.

---

## Rizin and Cutter

**Official sites:** [rizin.re](https://rizin.re/) and [cutter.re](https://cutter.re/)

Rizin is a free reverse engineering framework forked from radare2 with a focus on usability, features, and code cleanliness. Cutter is the official graphical interface powered by Rizin.

### Typical usage

Use Rizin/Cutter when you want:

- a free and open-source reverse engineering platform
- a GUI over a powerful reversing backend
- something in the radare2 family but with a cleaner direction
- graph views, widgets, and visual exploration
- a beginner-friendlier path into open-source binary analysis

### Specifics

Cutter is especially useful if you like the idea of radare2/Rizin but do not want to live fully in command-line mode. It gives you graphs, widgets, integrated terminal access, and a more familiar reverse engineering layout.

Rizin itself can analyze binaries, disassemble code, debug programs, work as a hex editor, and handle forensic-style tasks.

### Where it shines

- Free and open source
- GUI plus terminal workflow
- More approachable than raw radare2 for many people
- Active project direction
- Useful for students and Linux users

### Where it is annoying

- Still not as industry-standard as IDA/Ghidra
- Some workflows can feel rough compared with paid tools
- You may hit plugin or ecosystem limitations

### My take

If you want a free GUI and do not love Ghidra's Java-heavy feel, try Cutter. For learning, it is a nice bridge between "I want a visual tool" and "I want to understand what the backend is doing."

---

## x64dbg

**Official site:** [x64dbg.com](https://x64dbg.com/)

x64dbg is an open-source debugger for Windows x64 and x32 applications. It is popular in malware analysis, unpacking, crackme challenges, and Windows user-mode debugging.

### Typical usage

Use x64dbg when you need to:

- step through a Windows executable
- set breakpoints
- inspect registers and memory
- follow API calls
- observe control flow at runtime
- unpack simple packers in a lab
- compare what static analysis predicted with what actually happens

### Specifics

x64dbg has one interface for x64 and x32 debugging. It supports plugins, themes, scripting, memory maps, references, graphing, and useful integrations. It is much friendlier than old-school command-line debugging if you are learning Windows reversing.

For malware analysis, you would normally run it inside a controlled VM, not on your daily machine.

### Where it shines

- Free and open source
- Great Windows user-mode debugger
- Friendly UI compared with WinDbg
- Plugin ecosystem
- Good for learning dynamic analysis

### Where it is annoying

- Windows-focused
- Not the right tool for kernel debugging
- Malware can detect debuggers, so lab setup matters
- Debugging optimized binaries can still be painful

### My take

If you reverse Windows binaries, install x64dbg. Even if you mostly use IDA/Ghidra for static work, x64dbg is often the easiest way to check runtime behavior.

---

## WinDbg

**Official documentation:** [Microsoft Learn: WinDbg](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)

WinDbg is Microsoft's debugger for Windows. It can analyze crash dumps, debug live user-mode code, debug kernel-mode code, inspect memory and registers, and use Time Travel Debugging in modern versions.

### Typical usage

Use WinDbg when you need:

- crash dump analysis
- Windows internals debugging
- driver debugging
- kernel debugging
- production dump investigation
- memory analysis
- Time Travel Debugging workflows

### Specifics

WinDbg is not as friendly as x64dbg for casual reversing. It is more of an engineering and internals tool. But when you need to understand a Windows crash dump, inspect kernel structures, or work with drivers, WinDbg is the real thing.

Modern WinDbg has a newer UI, scripting support, a debugging data model, and Time Travel Debugging support.

### Where it shines

- Kernel and driver debugging
- Crash dumps
- Windows internals
- Production debugging
- Official Microsoft tooling

### Where it is annoying

- Command syntax can feel brutal
- Learning curve is high
- Not as pleasant for simple crackme-style user-mode reversing
- You need symbols and environment setup to get real value

### My take

x64dbg is the better first Windows reversing debugger. WinDbg is what you learn when you need to go deeper into Windows internals, dump analysis, drivers, or serious debugging.

---

## GDB

**Official site:** [sourceware.org/gdb](https://sourceware.org/gdb/)

GDB is the GNU Project debugger. It lets you inspect what another program is doing while it runs, or what it was doing when it crashed.

### Typical usage

Use GDB when you work with:

- Linux native binaries
- C/C++ programs
- ELF files
- exploit development labs
- embedded targets
- command-line debugging
- remote debugging

### Specifics

GDB is not pretty, but it is everywhere. On Linux, it is one of the default tools you should know. You can run programs, set breakpoints, inspect stack frames, print memory, check registers, disassemble functions, and attach to running processes.

With extensions like pwndbg, gef, or peda, GDB becomes much more comfortable for security research and CTF-style work.

### Where it shines

- Linux and Unix-like debugging
- Remote debugging
- Embedded workflows
- Availability
- Scriptability
- Strong ecosystem of extensions

### Where it is annoying

- CLI-heavy
- Raw GDB is not beginner-friendly
- Debugging optimized code can be confusing
- You often need extensions for a modern reversing workflow

### My take

If you touch Linux binaries, learn basic GDB. You do not need to become a wizard immediately. Learn breakpoints, stepping, registers, memory, backtraces, and disassembly first.

---

## LLDB

**Official site:** [lldb.llvm.org](https://lldb.llvm.org/)

LLDB is the debugger from the LLVM project. It is the default debugger in Xcode and supports C, Objective-C, C++, desktop targets, iOS devices, and the iOS simulator.

### Typical usage

Use LLDB when you work with:

- macOS applications
- iOS apps
- Objective-C / Swift / C / C++
- Xcode debugging
- Apple platform internals
- LLVM-based toolchains

### Specifics

LLDB is conceptually similar to GDB, but the command syntax is different. It has many aliases that make GDB users less miserable, and the official docs include a GDB-to-LLDB command map.

For iOS/macOS reversing, LLDB is an important tool even if you also use Hopper, Ghidra, or IDA.

### Where it shines

- Apple platform debugging
- Xcode integration
- LLVM/Clang ecosystem
- Good expression parsing
- Modern debugger architecture

### Where it is annoying

- Syntax differences if you come from GDB
- Apple platform security restrictions can complicate debugging
- Not a full reverse engineering platform by itself

### My take

If you are doing iOS/macOS security work, LLDB is not optional. Even basic commands are enough to make your static analysis more grounded.

---

## Frida

**Official site:** [frida.re](https://frida.re/)

Frida is a dynamic instrumentation toolkit for developers, reverse engineers, and security researchers. It lets you inject scripts into running processes and hook functions without recompiling or restarting the target in many workflows.

### Typical usage

Use Frida when you want to:

- hook Java methods in Android apps
- hook native functions
- inspect runtime values
- bypass client-side checks in a controlled security test
- trace API calls
- observe crypto usage
- modify behavior temporarily
- test assumptions from static analysis

### Specifics

Frida is not a decompiler. It is runtime instrumentation. You attach to a process, inject JavaScript-based hooks, and observe or modify behavior while the app runs.

For Android security testing, Frida is one of the core tools. You might use JADX to understand the code statically, apktool to inspect resources and smali, then Frida to check what happens at runtime.

A tiny example of the workflow idea:

```bash
frida-ps -Uai
frida -U -f com.example.app -l script.js
```

That does not magically hack anything. It just shows the normal pattern: list apps on a USB device, then spawn or attach with a script.

### Where it shines

- Android runtime analysis
- iOS runtime analysis
- Native function hooks
- Fast experiments
- No rebuild needed
- Great for validating static findings

### Where it is annoying

- App protections may detect it
- Root/jailbreak/emulator setup can be annoying
- Scripts break when app versions change
- It can become a crutch if you skip static understanding

### My take

For mobile app security, Frida is huge. But do not learn only Frida tricks. Learn what the app does first, then use Frida to verify behavior and test controls.

---

## JADX

**Official repository:** [github.com/skylot/jadx](https://github.com/skylot/jadx)

JADX is a Dex-to-Java decompiler. It has GUI and command-line tools for producing Java source code from Android DEX and APK files.

### Typical usage

Use JADX when you want to inspect:

- Android app Java/Kotlin logic
- classes.dex
- APK files
- app package structure
- strings and constants
- API endpoints
- client-side checks
- obfuscated class/method structure

### Specifics

JADX is usually the first Android reversing tool people open because it gives you readable Java-like output quickly.

Example:

```bash
jadx-gui app.apk
jadx -d output app.apk
```

The output is not the original source. Kotlin can look especially strange after decompilation. Obfuscation can turn names into `a`, `b`, `c`, and compiler-generated code can be noisy. Still, JADX is extremely useful for navigation and quick understanding.

### Where it shines

- Fast Android app inspection
- GUI and CLI
- Good search
- Good for Java/Kotlin app logic
- Easy first step for APK review

### Where it is annoying

- Obfuscation makes output ugly
- Decompiled Kotlin may be noisy
- Native libraries are outside its main purpose
- Resources and rebuild workflows are better handled by apktool

### My take

For Android, start with JADX unless you already know you need resources/smali or runtime hooks. It gives you the fastest readable overview.

---

## apktool

**Official site:** [apktool.org](https://apktool.org/)

apktool is a tool for reverse engineering Android APK files. It can decode resources to a nearly original form and rebuild APKs after modifications.

### Typical usage

Use apktool when you need to inspect or work with:

- AndroidManifest.xml
- resources.arsc
- layouts
- strings
- permissions
- smali code
- app rebuilds in lab conditions
- resource-level changes

### Specifics

JADX is better for reading Java-like logic. apktool is better for Android resources and smali-level structure.

Example:

```bash
apktool d app.apk -o app_decoded
apktool b app_decoded -o rebuilt.apk
```

For QA/security education, apktool is excellent because students can see how an APK is structured: manifest, resources, smali, assets, native libraries, META-INF, and so on.

### Where it shines

- Android resource decoding
- Manifest inspection
- Smali-level analysis
- Rebuild workflows
- Understanding APK structure

### Where it is annoying

- Decompiled smali is not beginner-friendly
- Rebuilding/signing can be finicky
- Resource decoding is not always perfect
- Not a high-level Java decompiler

### My take

For Android reversing, JADX and apktool are a pair. JADX shows you readable logic. apktool shows you the real APK structure.

---

## JEB

**Official site:** [pnfsoftware.com](https://www.pnfsoftware.com/)

JEB is a commercial modular reverse engineering platform. It supports disassembly, decompilation, debugging, and analysis of code and document files. It is especially known in Android reversing, but it also supports native code and other targets.

### Typical usage

Use JEB when you need:

- professional Android reversing
- Dalvik/DEX analysis
- native code analysis
- scripting and pipeline workflows
- commercial-grade mobile app analysis
- malware or goodware APK analysis

### Specifics

JEB is not the free beginner path. JADX and apktool cover a lot. But JEB has strong Android-focused workflows, static and dynamic analysis modules, and professional features that matter when APK analysis is your job, not just a weekend experiment.

### Where it shines

- Professional Android analysis
- Modular analysis platform
- Static and dynamic Android workflows
- Scripting and automation
- Good for teams doing repeated mobile reversing

### Where it is annoying

- Commercial license
- Overkill for basic APK inspection
- Smaller casual tutorial ecosystem than JADX/apktool

### My take

If you are just learning Android reversing, start with JADX, apktool, and Frida. If you do Android reversing professionally, JEB is worth knowing about.

---

## ILSpy

**Official repository:** [github.com/icsharpcode/ILSpy](https://github.com/icsharpcode/ILSpy)

ILSpy is an open-source, cross-platform .NET assembly browser and decompiler. It can inspect .NET assemblies and show C#-like decompiled code.

### Typical usage

Use ILSpy when you need to inspect:

- .NET applications
- DLLs
- NuGet packages
- C# assemblies
- metadata
- dependencies
- decompiled C# code
- old internal tools where source is missing

### Specifics

.NET reversing is different from native reversing. Managed metadata gives you much more information than a stripped C binary. Class names, method names, types, properties, and attributes may still be present unless obfuscated.

That makes ILSpy very useful for legitimate debugging, auditing, dependency inspection, and recovering understanding from compiled assemblies.

### Where it shines

- Open source
- Cross-platform
- Great for .NET assemblies
- Easy to use
- Useful for code review when source is unavailable

### Where it is annoying

- Obfuscated .NET can still be painful
- It is not a native binary reversing tool
- Dynamic/runtime behavior may require a debugger

### My take

If you touch .NET, install ILSpy. It is one of those tools that can save hours in five minutes.

---

## dnSpyEx

**Official project:** [github.com/dnSpyEx](https://github.com/dnSpyEx)

The original dnSpy was a popular .NET debugger and assembly editor. dnSpyEx is an unofficial continuation of that idea.

### Typical usage

Use dnSpyEx when you need:

- .NET debugging without source code
- Unity assembly inspection
- C# decompilation
- assembly editing in lab/research contexts
- stepping through managed code

### Specifics

The killer feature is not just decompiling .NET. It is debugging decompiled code. That is extremely convenient when analyzing .NET or Unity applications.

### Where it shines

- .NET debugging
- Unity analysis
- Decompile and debug in one workflow
- Useful for learning managed internals

### Where it is annoying

- Project lineage is messier than ILSpy
- Windows-oriented workflows are common
- Editing assemblies can quickly become fragile
- You need to be careful with legality and ethics

### My take

Use ILSpy for clean .NET browsing and decompilation. Use dnSpyEx when you need debugging and interactive managed-code analysis.

---

## Detect It Easy

**Official repository:** [github.com/horsicq/Detect-It-Easy](https://github.com/horsicq/Detect-It-Easy)

Detect It Easy, usually called DiE, is a file type identification tool. It is popular with malware analysts and reverse engineers because it quickly tells you what kind of file you are dealing with, what compiler or packer might have been used, and what format you should expect.

### Typical usage

Use DiE before opening a binary in a huge tool:

- check PE/ELF/Mach-O format
- identify compiler hints
- detect packer/protector signatures
- inspect entropy
- do quick malware triage
- avoid wasting time in the wrong tool

### Specifics

DiE supports signature-based and heuristic analysis. It is not perfect, but it is fast and useful. If it says something looks packed, encrypted, or weird, you know static analysis may not be straightforward.

### Where it shines

- Fast triage
- Malware analysis starting point
- File type and packer hints
- Cross-platform availability
- Easy UI

### Where it is annoying

- Signatures can be wrong or incomplete
- It tells you hints, not final truth
- You still need deeper analysis after triage

### My take

DiE is the kind of small tool you should run early. It can save you from opening a packed binary in a decompiler and wondering why everything looks like nonsense.

---

## PE-bear

**Official repository:** [github.com/hasherezade/pe-bear](https://github.com/hasherezade/pe-bear)

PE-bear is a multi-platform reversing tool for Windows PE files. It is designed to give malware analysts a fast first view of PE structure and to handle malformed PE files.

### Typical usage

Use PE-bear when you need to inspect:

- PE headers
- sections
- imports
- exports
- resources
- data directories
- suspicious PE structure
- malformed Windows executables

### Specifics

PE-bear is not a replacement for IDA, Ghidra, or x64dbg. It is a PE inspection tool. That is exactly why it is useful. Before you care about pseudocode, you often need to understand the executable structure.

### Where it shines

- PE file structure
- Malware triage
- Header inspection
- Import/export checks
- Fast "first view"

### Where it is annoying

- PE-focused
- Not a general decompiler
- You need PE format knowledge to get full value

### My take

If you analyze Windows executables, learn basic PE structure and use tools like PE-bear. Reverse engineering gets easier when you understand the container before staring at assembly.

---

## How I would choose the tool

There is no single "best" reverse engineering tool. The right tool depends on the target.

### If you are learning native reversing

Start with:

- Ghidra
- x64dbg on Windows
- GDB on Linux
- a few small C programs you compiled yourself

Do not start with a packed malware sample. Start with binaries where you know the source code. Compile them, strip symbols, change optimization flags, and see what changes.

### If you are doing Android app security

Use:

- JADX for Java/Kotlin logic
- apktool for manifest/resources/smali
- Frida for runtime behavior
- Ghidra/IDA/Binary Ninja for native `.so` libraries
- JEB if you need a commercial professional Android platform

A lot of Android analysis is not just "decompile APK." You need to understand app storage, exported components, network behavior, certificate trust, root/emulator checks, native libraries, and backend assumptions.

### If you are doing Windows malware analysis

Use:

- Detect It Easy for initial triage
- PE-bear for PE structure
- Ghidra/IDA/Binary Ninja for static analysis
- x64dbg for user-mode debugging
- WinDbg if you need dumps, drivers, or kernel-level work

And do it in a lab VM. Do not be the person who runs random malware on the main machine because "I just wanted to see what happens."

### If you are doing .NET or Unity

Use:

- ILSpy for clean decompilation and browsing
- dnSpyEx for debugging and interactive work

Managed code is often more readable than native code, but obfuscation can still make it ugly.

### If you are doing macOS/iOS

Use:

- Hopper, Ghidra, IDA, or Binary Ninja for static analysis
- LLDB for debugging
- Frida for runtime instrumentation where applicable

Apple platforms add their own signing, sandboxing, and platform restrictions, so the tooling is only half the problem.

---

## Static analysis vs dynamic analysis

A common beginner mistake is trusting the decompiler too much.

Decompiler output is a guess. A useful guess, but still a guess.

Static analysis can show:

- functions
- strings
- imports
- constants
- possible branches
- control flow
- suspicious APIs
- hardcoded endpoints
- cryptographic calls
- storage paths

Dynamic analysis can show:

- what code actually executes
- runtime values
- real API parameters
- decrypted strings
- network requests
- file writes
- branch decisions
- anti-debugging behavior
- environment checks

You usually need both.

For example, JADX may show a root detection method in an Android app. But Frida can show whether that method actually gets called, what it returns, and what happens if the return value changes. Ghidra may show a suspicious native function. x64dbg or GDB can show whether it runs and what values pass through registers.

Reverse engineering is basically controlled skepticism.

---

## What beginners should not skip

Do not only learn tools. Learn the concepts the tools expose:

- CPU registers
- stack and heap
- calling conventions
- pointers
- endianness
- assembly basics
- PE / ELF / Mach-O structure
- Android APK structure
- DEX and smali basics
- imports and exports
- symbols and debug info
- compiler optimizations
- obfuscation basics
- dynamic linking
- syscalls and APIs

Tools change. These concepts do not change as fast.

Also, do not confuse "the decompiler produced C-like code" with "I understand the program." Rename things. Add comments. Make notes. Trace inputs and outputs. Check runtime behavior. The value is not in opening the tool. The value is in reducing uncertainty.

---

## A simple starter lab

If you want a safe way to learn, create your own tiny C program:

```c
#include <stdio.h>
#include <string.h>

int check_password(const char *input) {
    return strcmp(input, "test123") == 0;
}

int main(int argc, char **argv) {
    if (argc < 2) {
        puts("Usage: ./demo <password>");
        return 1;
    }

    if (check_password(argv[1])) {
        puts("OK");
    } else {
        puts("Nope");
    }

    return 0;
}
```

Compile it with different options:

```bash
gcc demo.c -o demo_O0 -O0
gcc demo.c -o demo_O2 -O2
strip demo_O2
```

Then open the binaries in Ghidra, IDA Free, Binary Ninja trial, Cutter, or radare2. Debug with GDB. Look for:

- `main`
- `check_password`
- string references
- function calls
- branch conditions
- what changed after `strip`
- what changed after `-O2`

This teaches more than downloading random malware and pretending you are doing elite research.

---

## Final recommendations

If I had to keep the list practical:

- **Best free general platform:** Ghidra
- **Best classic professional platform:** IDA Pro
- **Best modern commercial workflow:** Binary Ninja
- **Best Windows user-mode debugger:** x64dbg
- **Best Windows internals debugger:** WinDbg
- **Best Linux debugger:** GDB
- **Best Apple-platform debugger:** LLDB
- **Best Android quick decompiler:** JADX
- **Best Android APK structure tool:** apktool
- **Best Android/runtime instrumentation tool:** Frida
- **Best .NET decompiler:** ILSpy
- **Best .NET debug/decompile workflow:** dnSpyEx
- **Best quick file triage:** Detect It Easy
- **Best PE structure viewer:** PE-bear
- **Best open-source GUI alternative:** Cutter
- **Best terminal-heavy toolkit:** radare2 or Rizin

My actual advice: do not collect tools like Pokemon. Pick one static tool, one debugger, and one target platform. Learn the workflow deeply.

For example:

- Android: JADX + apktool + Frida
- Windows native: Ghidra + x64dbg + DiE
- Linux native: Ghidra + GDB + readelf/objdump
- .NET: ILSpy + dnSpyEx

That is enough to start doing real work instead of just installing tools.

---

## Sources and official links

- [IDA Pro by Hex-Rays](https://hex-rays.com/ida-pro)
- [Ghidra GitHub repository](https://github.com/NationalSecurityAgency/ghidra)
- [Binary Ninja](https://binary.ninja/)
- [Hopper Disassembler](https://www.hopperapp.com/)
- [radare2](https://rada.re/)
- [Rizin](https://rizin.re/)
- [Cutter](https://cutter.re/)
- [x64dbg](https://x64dbg.com/)
- [Microsoft WinDbg documentation](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)
- [GDB official site](https://sourceware.org/gdb/)
- [LLDB official documentation](https://lldb.llvm.org/)
- [Frida](https://frida.re/)
- [JADX GitHub repository](https://github.com/skylot/jadx)
- [apktool](https://apktool.org/)
- [JEB by PNF Software](https://www.pnfsoftware.com/)
- [ILSpy GitHub repository](https://github.com/icsharpcode/ILSpy)
- [dnSpyEx GitHub organization](https://github.com/dnSpyEx)
- [Detect It Easy GitHub repository](https://github.com/horsicq/Detect-It-Easy)
- [PE-bear GitHub repository](https://github.com/hasherezade/pe-bear)
