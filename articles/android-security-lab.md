---
title: "Android Security Lab"
description: "A practical Android reverse-engineering playlist covering APKTool, Jadx, Frida, smali patching, traffic interception, emulator workflows, and Android internals."
pubDate: 2026-05-24
tags: ["Android", "Security", "Reverse Engineering", "Frida", "APKTool", "Jadx", "Traffic Interception", "Mobile Security"]
draft: false
---

This is a practical video playlist about Android reverse engineering, mobile security testing, and application analysis.

The series covers smali patching, root detection bypasses, Frida instrumentation, traffic interception, emulator workflows, Android filesystem internals, APKTool, Jadx, and general security research workflows.

The materials are intended for educational use, security research, lab environments, and authorized testing only.

---

## 🎬 Videos

### 1. **[How to Bypass Android Root Checks via Smali Patching — No Frida. Part 2](https://youtu.be/WC_FdEfaFug?si=1wsm0p8-GSzoCrvd)**

Static root detection bypass using smali modifications and APKTool.

This video focuses on modifying the application directly instead of using dynamic instrumentation tools like Frida.

---

### 2. **[Android Root Detection Bypass — Reverse Engineering. Part 1](https://www.youtube.com/watch?v=ymE1U5iC6F0)**

An entry-level reverse-engineering workflow for finding and patching Android root checks.

Tools used include Jadx for reading decompiled Java/Kotlin code and APKTool for working with smali.

---

### 3. **[Frida on Fire — Dynamic Analysis for Android & iOS](https://youtu.be/xGKLkzQCyGU?si=Ak1xBe83QvDjYr2e)**

Introduction to dynamic instrumentation with Frida.

The video covers basic hooking, tracing, runtime inspection, and how Frida helps analyze application behavior without rebuilding the APK.

---

### 4. **[Android Under the Hood — Where Do Apps Live?](https://youtu.be/sJF0aVaV4YM?si=qBjrxpyd1bNtJLgt)**

A practical overview of Android’s filesystem layout.

You will learn where applications are installed, where private app data is stored, and how Android organizes internal application directories.

---

### 5. **[Working with Android Emulator — Terminal, ADB Commands](https://youtu.be/IMz_oW0k4oQ?si=_-4945juN5L4ugH9)**

A hands-on guide to working with the Android emulator from the terminal.

Topics include ADB commands, emulator configuration, shell access, and command-line workflows useful for testing and analysis.

---

### 6. **[HACKING Android Applications — Real Examples](https://youtu.be/Ai1agob7q_k?si=HcPXUDzr4w5w_jsN)**

Hands-on examples of Android application manipulation and vulnerability exploration.

This video demonstrates practical techniques used when analyzing intentionally vulnerable or authorized test applications.

---

### 7. **[Reverse Engineering Android Apps for Beginners — APKTool, Jadx](https://youtu.be/OwVO3Hk5y9s?si=fCuYFiAq34YGSIvg)**

A beginner-friendly introduction to Android static analysis.

The video explains how to inspect APK files, decompile code, analyze resources, and understand the basic structure of Android applications.

---

### 8. **[Intercepting Android App Traffic — Charles Proxy + Frida Tutorial](https://youtu.be/HCKXLWbrF8k?si=i-4jXYVCiwdDpygr)**

A practical traffic interception workflow for Android applications.

This video combines proxy-based analysis with Frida hooks to inspect application network behavior more deeply.

---

### 9. **[Interception of Traffic on Android — Setting Up an Emulator](https://youtu.be/yWsBhp-Fg3k?si=INFFphetSuWUygu4)**

A focused emulator setup guide for mobile traffic analysis.

Topics include emulator networking, proxy configuration, HTTPS interception, and common setup issues.

---

### 10. **[Android Reverse Engineering Setup — Part 1. Tools Review](https://youtu.be/jwHrBa3f5i0)**

The first episode of the Android Reverse Engineering setup series.

This video reviews the core tools used for Android pentesting, security research, and malware analysis, including Frida, Objection, Drozer, Jadx, APKTool, MobSF, and related utilities.

---

### 11. **[Android Reverse Engineering Setup — Part 2. Installing Tools](https://youtu.be/NfGq5BOB70M)**

A practical installation walkthrough for the Android reverse-engineering toolkit.

Tools covered include Frida, Android Emulator, Android SDK Tools, Android Platform Tools, and Ghidra.

---

### 12. **[Android App Development from Scratch — Live Coding](https://www.youtube.com/playlist?list=PL0zZBw8Dq429NOr4MPDZHolp8UoFVuNPI)**

A practical live-coding series where we build Android apps from scratch using Android Studio, Jetpack Compose, and AI-assisted development.

Each stream focuses on implementing new features, improving app architecture, adding security layers, and exploring modern Android development workflows.

This playlist is useful for beginners and mid-level developers who want to understand Android applications from the developer side, not only from the reverse-engineering side.

---

## 🔧 Full Android Reverse Engineering Workflow

Below is a compact end-to-end workflow for unpacking, patching, rebuilding, installing, and analyzing Android apps using APKTool, ADB, and Frida on a rooted emulator.

Use this workflow only with applications you own, intentionally vulnerable apps, lab targets, or software you are authorized to test.

---

## 📦 1. Unpack and Rebuild APK with APKTool

Use APKTool to decode an APK into readable resources and smali code:

```bash
apktool d app.apk -o unpacked
```

After making changes, rebuild the APK:

```bash
apktool b unpacked -o app_patched.apk
```

This is useful when you need to inspect resources, modify smali code, patch checks, or understand the internal structure of an Android application.

---

## 🔐 2. Start a Rooted Emulator

If your emulator supports root access, enable root mode:

```bash
adb root
```

Root access is often useful for security research because it allows deeper inspection of processes, files, certificates, and runtime behavior.

---

## 🧩 3. Push and Run Frida Server

Copy the Frida server binary to the emulator:

```bash
adb push frida-server /data/local/tmp/
```

Make it executable:

```bash
adb shell chmod +x /data/local/tmp/frida-server
```

Start Frida server:

```bash
adb shell /data/local/tmp/frida-server &
```

Frida server allows your host machine to dynamically instrument applications running inside the emulator.

---

## 📲 4. Install the Target APK on the Emulator

Install the APK:

```bash
adb install fdroid.apk
```

If you rebuilt or patched an APK, make sure it is properly signed before installing it.

---

## 🧰 5. Install Frida Tools on the Host Machine

Create a Python virtual environment:

```bash
python -m venv new_venv
```

Activate it:

```bash
source new_venv/bin/activate
```

Install Frida tools:

```bash
pip3 install frida-tools
```

Check that Frida can see the connected device:

```bash
frida-ps -U
```

---

## 🔍 6. Find the Target Process

You can inspect running processes from the Android shell:

```bash
adb shell
adb top
```

Or use Frida directly:

```bash
frida-ps -U
```

Find the package name or process ID of the application you want to analyze.

---

## 🎯 7. Run a Frida Script

Run a local Frida script against a process ID:

```bash
frida -U -p <process_id> -l ssl-pin.js
```

Or attach by package name:

```bash
frida -U -n com.example.app -l script.js
```

This approach is commonly used for runtime inspection, method hooking, bypass experiments, logging, and behavior analysis.

---

## ▶️ 8. Run a Script from Frida CodeShare

You can also run a public Frida CodeShare script:

```bash
frida -U -n com.example.app -c codeshare/<script_name>
```

CodeShare can be useful for quick experiments, but always review scripts before running them.

Do not blindly execute third-party instrumentation scripts in sensitive environments.

---

## 🧠 Recommended Learning Path

If you are new to Android reverse engineering, start with the basics:

1. Learn how Android apps are structured.
2. Inspect APKs with Jadx.
3. Decode and rebuild APKs with APKTool.
4. Practice simple smali modifications.
5. Learn ADB and emulator workflows.
6. Add Frida for dynamic analysis.
7. Combine traffic interception with runtime hooks.
8. Build your own Android apps to understand how real apps are designed internally.

The best way to learn Android security is to move between both sides: development and reverse engineering.

When you understand how apps are built, it becomes much easier to understand how they can break.

---

## 🧪 Suggested Lab Targets

For safe practice, use intentionally vulnerable apps, demo applications, or your own projects.

Good lab categories include:

- Demo login applications
- Intentionally vulnerable Android apps
- Open-source APKs
- Self-built Jetpack Compose apps
- Local test backends
- Emulator-only security labs

Avoid testing real applications without permission.

---

## ✅ Summary

This playlist provides a practical Android security lab path:

- Static analysis with Jadx
- APK unpacking and rebuilding with APKTool
- Smali patching
- Root detection bypass experiments
- Dynamic instrumentation with Frida
- Traffic interception with Charles Proxy
- Emulator setup and ADB workflows
- Android filesystem and internals
- Building Android apps to better understand app security

The goal is not just to “hack APKs,” but to understand how Android applications work internally, how security checks are implemented, and how testers can analyze mobile apps in a controlled and responsible way.
