---
title: "Android Reverse Engineering Toolkit: Repos Worth Using"
description: "A practical guide to Android-focused reverse engineering and mobile security tools from my GitHub star lists: static analysis, dynamic analysis, APK patching, traffic inspection, Frida workflows, emulators, and vulnerable labs."
pubDate: 2026-06-04
tags: ["Android", "Reverse Engineering", "Mobile Security", "APK", "Frida", "JADX", "MobSF", "Burp Suite", "mitmproxy", "Dynamic Analysis", "Static Analysis", "AppSec"]
draft: false
---
There are many reverse engineering and cybersecurity repositories on GitHub, but not all of them are useful for Android work.

Some are too generic. Some are iOS-only. Some are Windows malware tooling. Some are interesting, but they do not really help when your actual task is simple: take an APK, understand what it does, inspect its traffic, bypass annoying lab protections, run it in a controlled environment, and produce useful findings.

This article is focused only on Android-related tools from these two GitHub star lists:

- Reverse Engineering: https://github.com/stars/deemoun/lists/reverse-engineering
- Cybersecurity: https://github.com/stars/deemoun/lists/cybersecurity

The goal is not to list everything. The goal is to build a practical Android reverse engineering toolkit and explain when each tool makes sense.

## The Android reverse engineering workflow

For Android, the workflow usually looks like this:

1. Get the APK.
2. Decompile it.
3. Read the manifest, resources, Java/Kotlin code, native libraries, and configs.
4. Look for endpoints, secrets, hardcoded values, exported components, insecure storage, weak crypto, and certificate pinning.
5. Run the app on an emulator or a test device.
6. Intercept traffic.
7. Use dynamic instrumentation when static analysis is not enough.
8. Patch, rebuild, resign, and test the APK again.
9. Write findings clearly.

No single tool does all of this perfectly. That is why a good toolkit is a combination of static analysis, dynamic analysis, patching, traffic inspection, and lab automation.

## Static analysis: understand the APK before running it

Static analysis is where I usually start. Before touching Frida or Burp, I want to know what is inside the APK.

### JADX AI MCP

Repo: https://github.com/zinja-coder/jadx-ai-mcp

JADX is already one of the main tools for Android reverse engineering. The interesting part here is not just decompilation itself, but the MCP integration around JADX.

Use it when you want to analyze a bigger APK and need help navigating code faster. Android apps can be annoying to inspect manually because you have many activities, fragments, services, receivers, obfuscated classes, generated code, and SDK noise. A JADX MCP-style workflow can help you ask better questions about the codebase instead of manually jumping through hundreds of files.

Use it for:

- finding suspicious code paths;
- understanding login or authorization logic;
- locating certificate pinning checks;
- tracing API endpoints;
- explaining strange obfuscated flows;
- speeding up repetitive code navigation.

It does not replace understanding Android internals. It just makes the analysis faster.

### APKDeepLens

Repo: https://github.com/d78ui98/APKDeepLens

APKDeepLens is useful when you want a wider security-oriented view of an APK. It is not just about opening code and reading it manually. It helps collect security insights from the app package.

Use it when you need a fast first pass over an APK and want to understand what deserves attention. This is useful for QA engineers moving into mobile security because it gives a structured starting point.

Use it for:

- initial APK triage;
- finding suspicious permissions;
- checking exposed components;
- collecting package metadata;
- spotting common Android security issues;
- preparing notes before deeper manual analysis.

The important thing: do not blindly trust automated output. Treat it as a map. You still need to verify the findings manually.

### mobsfscan

Repo: https://github.com/MobSF/mobsfscan

mobsfscan is a static analysis tool based on MobSF rules. It supports Android source code written in Java and Kotlin, so it is useful when you have source code, not only a compiled APK.

Use it when you are testing an Android project as part of development or CI/CD. If you work as QA, SDET, or AppSec, this is a good bridge between classic testing and security testing.

Use it for:

- scanning Android source code;
- checking insecure coding patterns;
- adding security checks to CI;
- catching obvious issues before the APK reaches manual testing.

This is not a full mobile pentest. It is a static code scanner. But that is exactly why it is useful: it is simple, repeatable, and easy to automate.

### BFScan

Repo: https://github.com/BlackFan/BFScan

BFScan is useful for extracting URLs, paths, secrets, raw HTTP requests, and OpenAPI-like information from JAR, WAR, and APK applications.

For Android work, this is useful when you want to quickly answer a very practical question: “Where does this app talk to?”

Use it for:

- finding backend endpoints;
- extracting URLs from APKs;
- finding hidden API paths;
- collecting interesting strings before traffic interception;
- preparing Burp or mitmproxy testing.

This kind of tool is necessary because many Android bugs start from backend interaction. You may not find much by only clicking through the app. Extracting endpoints gives you a better attack surface.

### get_schemas

Repo: https://github.com/teknogeek/get_schemas

This is a small but useful tool: it prints URL schemes from an Android app.

Use it when you are testing deep links, custom schemes, intent handling, and app-to-app communication. For Android security, this matters because bad deep link validation can lead to account takeover flows, token leakage, unwanted navigation, or broken authorization assumptions.

Use it for:

- discovering custom URL schemes;
- preparing deep link test cases;
- checking app entry points;
- mapping external attack surface.

It is not a big framework. It is a focused helper. That is good.

## APK decompilation, patching, rebuilding, and signing

At some point, static reading is not enough. You need to change the APK, rebuild it, resign it, or inject something into it.

### PulseAPK-Core

Repo: https://github.com/deemoun/PulseAPK-Core

PulseAPK-Core is the cross-platform core for APK work: decompilation, analysis, and building.

Use it if you want the logic separated from the Windows UI. This matters if you want to build tooling around APK analysis, automate workflows, or eventually support Linux/macOS better.

Use it for:

- custom APK tooling;
- scripting repeatable APK analysis;
- building your own frontend;
- separating UI from APK processing logic.

For a long-term tool, this is the more important architectural part. The UI is nice, but the core is what makes it reusable.

### apk-mitm

Repo: https://github.com/niklashigi/apk-mitm

apk-mitm automatically prepares Android APK files for HTTPS inspection.

Use it when you need to inspect app traffic and the app does not cooperate with your proxy setup. In many Android apps, simply installing a user certificate is not enough because of Network Security Config, certificate pinning, or other hardening.

Use it for:

- preparing APKs for HTTPS traffic inspection;
- debugging apps where proxying fails;
- quickly testing whether traffic can be made visible;
- learning how APK patching affects TLS inspection.

Do not treat it as magic. If the app has serious custom pinning or native checks, you may still need Frida or manual patching.

### frida-gadget

Repo: https://github.com/ksg97031/frida-gadget

This tool patches APKs to enable Frida Gadget usage by downloading the library and injecting code into the app.

Use it when you cannot or do not want to run frida-server on the device. Frida Gadget is useful because the instrumentation library is embedded into the app itself.

Use it for:

- non-root dynamic analysis;
- instrumenting apps without a traditional frida-server setup;
- testing apps in environments where root is not available;
- building repeatable patched APK labs.

This is one of those tools that becomes important when the normal “rooted emulator + frida-server” path is not enough.

## Dynamic analysis with Frida

Static analysis tells you what the code looks like. Dynamic analysis tells you what the app actually does at runtime.

For Android reverse engineering, Frida is one of the most important tools because it lets you hook Java methods, native functions, crypto calls, network logic, root checks, emulator checks, and certificate pinning logic.

### Frida-Script-Runner

Repo: https://github.com/z3n70/Frida-Script-Runner

Frida-Script-Runner is a web-based Frida framework and toolkit for Android and iOS penetration testing, mobile security, and dynamic analysis.

Use it when you want a more organized Frida workflow instead of running random scripts manually from the terminal.

Use it for:

- managing Frida scripts;
- testing hooks faster;
- working with multiple bypass scripts;
- dynamic app analysis;
- building a more visual mobile pentest workflow.

This is useful because Frida work can get messy quickly. You start with one script, then another, then root bypass, then SSL bypass, then logging, then class enumeration. A runner helps keep this under control.

### frida-script-gen

Repo: https://github.com/thecybersandeep/frida-script-gen

frida-script-gen generates Frida bypass scripts for Android APK root and SSL checks.

Use it when you need a starting point for common bypass scripts. This is useful in labs, CTF-style apps, vulnerable Android apps, and quick experiments.

Use it for:

- root detection bypass templates;
- SSL pinning bypass templates;
- learning how Frida scripts are structured;
- speeding up repetitive Android lab work.

But do not become lazy with generated scripts. The real skill is understanding why a hook works and how to adjust it when the app uses a different check.

### magisk-frida

Repo: https://github.com/ViRb3/magisk-frida

magisk-frida runs frida-server on boot with Magisk and keeps it up to date.

Use it when you have a rooted Android device or emulator with Magisk and you want Frida to be available automatically.

Use it for:

- rooted test devices;
- persistent Frida setup;
- mobile security labs;
- avoiding manual frida-server setup every time.

This is necessary because Android dynamic analysis often becomes annoying because of setup friction. If Frida is always available, you spend more time testing and less time fighting the environment.

### ZygiskFrida

Repo: https://github.com/lico-n/ZygiskFrida

ZygiskFrida injects Frida Gadget using Zygisk to bypass anti-tamper checks.

Use it when the app detects normal Frida setups or when you need a more advanced injection approach.

Use it for:

- apps with anti-Frida checks;
- apps that detect frida-server;
- more advanced Android instrumentation;
- testing hardened apps in a controlled lab.

This is not the first tool I would give to a beginner. Start with normal Frida. Use Zygisk-based approaches when basic instrumentation fails.

### phantom-frida

Repo: https://github.com/TheQmaks/phantom-frida

phantom-frida builds anti-detection Frida server versions from source with many patches against common detection vectors.

Use it when the app detects Frida by process names, symbols, ports, maps, threads, or other runtime artifacts.

Use it for:

- anti-Frida bypass experiments;
- testing apps with stronger runtime protection;
- research into detection and evasion;
- labs where normal frida-server gets blocked.

This is powerful, but it can also turn into a rabbit hole. Use it when you actually hit Frida detection, not as the default for every APK.

### frida-android-helper2

Repo: https://github.com/secuworm2/frida-android-helper2

This is a utility-style repository for Android Frida work.

Use it when you already understand Frida basics and want helper scripts or automation around common Android instrumentation tasks.

Use it for:

- speeding up Frida setup;
- collecting helper logic;
- experimenting with Android runtime hooks;
- reducing repetitive terminal work.

For beginners, I would still recommend learning raw Frida first. Helpers make sense after you understand the manual workflow.

### kahlo-mcp

Repo: https://github.com/FuzzySecurity/kahlo-mcp

kahlo-mcp is a Frida MCP server for AI-assisted Android instrumentation.

Use it if you are experimenting with AI-assisted reversing workflows. The idea is interesting: connect Frida instrumentation with an assistant-style workflow that can help inspect, hook, and reason about runtime behavior.

Use it for:

- experimental Android instrumentation;
- AI-assisted Frida workflows;
- faster exploration of runtime behavior;
- research, not beginner production workflows.

This is not necessary for everyone, but it fits the direction mobile reversing is going: less manual glue code, more assisted workflows.

## Traffic inspection and API testing

Most Android apps are just clients for backend APIs. If you cannot inspect traffic, you miss half of the story.

### mitmproxy

Repo: https://github.com/mitmproxy/mitmproxy

mitmproxy is an interactive TLS-capable HTTP proxy for penetration testers and developers.

Use it when you want a scriptable, terminal-friendly alternative to Burp Suite or Charles Proxy.

Use it for:

- HTTP/HTTPS interception;
- inspecting API requests;
- modifying traffic;
- scripting traffic manipulation;
- debugging mobile app backend communication.

For Android security work, mitmproxy is necessary because traffic inspection shows you what the app really sends: tokens, headers, device identifiers, API paths, request bodies, and backend behavior.

### Atlantis Android

Repo: https://github.com/ProxymanApp/atlantis-android

Atlantis Android captures HTTP/HTTPS traffic from Android apps and sends it to Proxyman for debugging.

Use it when you are working with Proxyman and want better in-app network debugging support.

Use it for:

- debugging Android network calls;
- inspecting traffic during development;
- QA workflows where developers can integrate debugging helpers;
- teams already using Proxyman.

This is more developer/QA friendly than pure offensive tooling. That is fine. Android security is not only about bypassing protections. Sometimes you need clean observability.

### noxen

Repo: https://github.com/frankheat/noxen

noxen is an Android interception tool for component communication and attack-surface mapping.

Use it when you care about more than HTTP traffic. Android apps communicate through activities, services, broadcast receivers, content providers, intents, and internal component flows.

Use it for:

- mapping Android components;
- inspecting app communication paths;
- finding exposed attack surface;
- understanding how an app behaves internally.

This is important because Android security is not just TLS and APIs. Exported components and intent handling can be just as dangerous.

## Emulator and lab environment

A good Android security workflow needs a controlled environment. You need reproducible devices, snapshots, root when needed, proxy setup, Frida setup, and sometimes many test runs.

### docker-android

Repo: https://github.com/HQarroum/docker-android

 docker-android provides a minimal and customizable Docker image running the Android emulator as a service.

Use it when you want Android emulators in a more automated environment.

Use it for:

- CI-style Android testing;
- repeatable emulator environments;
- automated mobile security checks;
- running test devices as services.

For manual reversing, a local emulator is often enough. But when you want repeatability, Docker-based Android becomes interesting.

### BrutDroid

Repo: https://github.com/Brut-Security/BrutDroid

BrutDroid automates Android Studio pentest setup with emulator rooting, Frida, and Burp Suite integration.

Use it when you want to reduce the setup pain of Android mobile pentesting.

Use it for:

- preparing a mobile pentest lab;
- emulator rooting;
- Frida setup;
- Burp integration;
- teaching or repeating Android security workflows.

This kind of tool is necessary because mobile security setup is often more painful than the actual test. A student can lose hours on certificates, emulator images, root, ADB, and proxy configuration before even touching the APK.

## Android SELinux and lower-level system work

Most APK reverse engineering does not require deep SELinux work. But if you are researching Android internals, rooted devices, Magisk modules, or security boundaries, SELinux becomes relevant.

### setools-android

Repo: https://github.com/xmikos/setools-android

setools-android is an unofficial Android port of SETools with sepolicy-inject included.

Use it when you are inspecting Android SELinux policies or experimenting with policy changes in a research environment.

Use it for:

- Android SELinux policy inspection;
- rooted device research;
- understanding restrictions around processes and services;
- advanced Android platform security learning.

This is not a daily APK reversing tool. It is more for Android system security research.

### selinux_permissive

Repo: https://github.com/evdenis/selinux_permissive

selinux_permissive is a Magisk module that switches SELinux to permissive mode.

Use it only in a lab when you clearly understand why you need it.

Use it for:

- debugging rooted lab devices;
- temporary Android system experiments;
- testing whether SELinux is blocking your research setup.

Do not use this as a normal setup recommendation. SELinux exists for a reason. Turning it permissive weakens the security model, so this belongs in controlled research environments only.

## Vulnerable apps and practice targets

Tools are useless without practice targets. You need intentionally vulnerable apps where you can break things legally and safely.

### Vulnerable-Bank-App-Demo

Repo: https://github.com/deemoun/Vulnerable-Bank-App-Demo

This is an intentionally vulnerable demo bank app for security analysts and test engineers.

Use it when you want to teach or practice mobile security without touching real apps.

Use it for:

- Android security demos;
- QA-to-AppSec training;
- Frida labs;
- traffic interception practice;
- insecure storage and API testing examples;
- writing tutorials.

This kind of app is necessary because beginners need a legal playground. Real apps are messy, protected, and legally risky. Vulnerable demo apps let you learn the workflow safely.

## AI-assisted Android reversing

AI-assisted tooling is clearly becoming part of reverse engineering. I would not build the whole workflow around it yet, but I would not ignore it either.

### areclaw

Repo: https://github.com/TheQmaks/areclaw

areclaw is an Android Reverse Engineering Command-Line Automation Workspace with AI-driven security analysis.

Use it when you want to experiment with AI-assisted Android reversing from the command line.

Use it for:

- APK analysis automation;
- assisted security review;
- faster triage;
- repeatable command-line reversing workflows.

This is promising, but I would still verify everything manually. AI can help you navigate, summarize, and connect clues. It can also confidently miss obvious context.

### android-reverse-engineering-skill

Repo: https://github.com/SimoneAvogadro/android-reverse-engineering-skill

This is a Claude Code skill for Android app reverse engineering.

Use it if you are already experimenting with AI coding agents and want Android-specific reversing help.

Use it for:

- guided APK analysis;
- codebase navigation;
- summarizing Android app structure;
- creating repeatable reversing prompts and workflows.

Again, this is not a replacement for fundamentals. It is a productivity layer.

## My practical Android security stack

If I had to keep the workflow simple, I would organize it like this:

### For static analysis

- JADX + JADX AI MCP
- APKDeepLens
- mobsfscan
- BFScan
- get_schemas

### For APK modification

- PulseAPK
- PulseAPK-Core
- pulse-gadget
- apk-mitm
- frida-gadget

### For runtime instrumentation

- Frida-Script-Runner
- frida-script-gen
- magisk-frida
- ZygiskFrida
- phantom-frida

### For traffic and attack surface

- mitmproxy
- Atlantis Android
- noxen

### For lab setup

- docker-android
- BrutDroid
- Vulnerable-Bank-App-Demo

### For advanced Android system research

- setools-android
- selinux_permissive

### For AI-assisted experiments

- areclaw
- kahlo-mcp
- android-reverse-engineering-skill

## Final thoughts

Android reverse engineering is not about one magical tool. It is about combining tools correctly.

Use static analysis to understand the APK. Use traffic inspection to understand the backend behavior. Use Frida when the app hides behavior at runtime. Use patching tools when you need to modify the APK. Use vulnerable apps to practice legally. Use AI-assisted tools as helpers, not as your brain replacement.

The best toolkit is not the biggest one. The best toolkit is the one you can actually use to move from APK to evidence.

**That is the real goal: not just “I reversed something”, but “I found something, verified it, and can explain it clearly.**