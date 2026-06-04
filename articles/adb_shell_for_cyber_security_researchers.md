---
title: "ADB Shell for QA Engineers, Android Enthusiasts, and Cyber Researchers"
description: "A practical Android guide explaining how adb shell helps with app testing, debugging, emulator control, logs, packages, permissions, network checks, screenshots, file access, and mobile security research."
pubDate: 2026-06-03
tags: ["Android", "ADB", "Android Emulator", "QA", "Mobile Testing", "Cybersecurity", "Mobile Security", "AppSec", "Logcat", "Android Studio", "Reverse Engineering", "Android Testing"]
draft: false
---

`adb shell` is one of the most useful tools in the Android ecosystem. It is not only for Android developers. A QA Engineer can use it to test apps faster, collect logs, reproduce bugs, change device state, clear app data, inspect packages, test permissions, simulate bad conditions, and prepare better bug reports. An Android enthusiast can use it to understand how the device works under the hood. A cyber researcher can use it as the basic entry point for mobile app analysis.

ADB means **Android Debug Bridge**. It lets your computer communicate with an Android device or emulator. The `adb shell` part opens a Linux-like command shell inside Android. From there, you can inspect the system, call Android services, control apps, read logs, work with files, and run many diagnostic commands.

This article is a practical field guide. It focuses on commands that are actually useful during mobile QA, Android tinkering, and security research.

## Why ADB Shell Matters

A normal tester can click through the UI. A better tester can also control the device from the command line.

With ADB you can:

- install and uninstall APK files;
- start and stop apps directly;
- clear app data before a clean test;
- collect logs during a crash;
- inspect installed packages and app paths;
- test permissions;
- simulate low battery, unplugged charging state, rotation, network changes, and app standby;
- take screenshots and screen recordings;
- copy files from and to the device;
- inspect app storage on debug builds;
- run an emulator in different modes;
- connect tools like Appium, Frida, Objection, Burp Suite, Charles Proxy, and Android Studio to the same test device.

For QA work, this is not “hacker magic”. This is normal professional tooling.

## Basic Setup

ADB comes with Android SDK Platform Tools. If Android Studio is installed, you usually already have it.

Common paths:

```bash
# macOS / Linux, common SDK path
~/Library/Android/sdk/platform-tools/adb
~/Android/Sdk/platform-tools/adb

# Windows, common SDK path
C:\Users\YourUser\AppData\Local\Android\Sdk\platform-tools\adb.exe
```

You can add `platform-tools` to your `PATH`, so you can run `adb` from any terminal.

Check that ADB works:

```bash
adb version
```

List connected devices:

```bash
adb devices
```

Typical output:

```text
List of devices attached
emulator-5554   device
R58M123456A     device
```

If you see `unauthorized`, unlock the phone and accept the USB debugging prompt.

## Physical Device vs Emulator

ADB works with both:

```bash
adb devices
```

A physical device usually appears with a serial number. An emulator usually appears as something like:

```text
emulator-5554
```

If you have multiple devices connected, specify the target:

```bash
adb -s emulator-5554 shell
adb -s R58M123456A shell
```

This matters when you run tests, install builds, collect logs, or debug a bug on a specific Android version.

## Entering the Android Shell

Open a shell:

```bash
adb shell
```

Run one command without entering interactive mode:

```bash
adb shell getprop ro.build.version.release
adb shell getprop ro.product.model
```

Useful device information:

```bash
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk
adb shell getprop ro.product.manufacturer
adb shell getprop ro.product.model
adb shell getprop ro.build.fingerprint
```

These commands are useful for bug reports. A vague bug report says “crashes on Android”. A better bug report says:

```text
Device: Pixel 8 emulator
Android: 15
API level: 35
Build fingerprint: google/sdk_gphone64_x86_64/...
App version: 1.4.2-debug
```

## Running the Android Emulator from the Command Line

You do not have to start the emulator from Android Studio. You can run it directly from the terminal.

First, list available virtual devices:

```bash
emulator -list-avds
```

Example output:

```text
Pixel_8_API_35
Pixel_9_Rooted
Medium_Phone_API_36
```

Start an emulator:

```bash
emulator -avd Pixel_8_API_35
```

Alternative syntax:

```bash
emulator @Pixel_8_API_35
```

On many systems, the `emulator` binary is inside:

```bash
~/Android/Sdk/emulator/emulator
```

or on Windows:

```powershell
C:\Users\YourUser\AppData\Local\Android\Sdk\emulator\emulator.exe
```

## Useful Emulator Startup Modes

The emulator has many command-line flags. These are the ones that are genuinely useful for QA and research.

### Cold Boot Mode

Use this when the emulator behaves weirdly, hangs on boot, or loads a broken snapshot.

```bash
emulator -avd Pixel_8_API_35 -no-snapshot-load
```

This tells the emulator not to load the previous snapshot. It starts cleaner.

### Wipe Data Mode

Use this when you want a completely fresh virtual device.

```bash
emulator -avd Pixel_8_API_35 -wipe-data
```

This deletes emulator user data. It is useful before testing onboarding, login, first launch, permissions, and fresh install flows.

Do not use this if you need to preserve test accounts, app data, downloaded files, or proxy certificates inside the emulator.

### No Snapshot Save

Use this when you do not want the emulator to save its current state after closing.

```bash
emulator -avd Pixel_8_API_35 -no-snapshot-save
```

Good for repeatable test sessions where you want to avoid polluted emulator state.

### No Snapshot Load and No Snapshot Save

A good clean testing mode:

```bash
emulator -avd Pixel_8_API_35 -no-snapshot-load -no-snapshot-save
```

This avoids both loading old state and saving new state.

### Headless Mode

Useful for CI, automation, or running an emulator without a visible window.

```bash
emulator -avd Pixel_8_API_35 -no-window
```

Often combined with:

```bash
emulator -avd Pixel_8_API_35 -no-window -no-audio -no-boot-anim
```

This is useful for automated test environments, but for manual QA it is usually better to keep the emulator window visible.

### No Boot Animation

Speeds up emulator startup a little:

```bash
emulator -avd Pixel_8_API_35 -no-boot-anim
```

### No Audio

Useful on servers, CI machines, or unstable desktop environments:

```bash
emulator -avd Pixel_8_API_35 -no-audio
```

### GPU Modes

For graphics issues, rendering bugs, black screens, or emulator crashes, GPU mode matters.

Common options:

```bash
emulator -avd Pixel_8_API_35 -gpu host
emulator -avd Pixel_8_API_35 -gpu swiftshader_indirect
emulator -avd Pixel_8_API_35 -gpu auto
```

Typical usage:

- `-gpu host` uses the host GPU and is usually faster.
- `-gpu swiftshader_indirect` uses software rendering and can be more stable on problematic graphics drivers.
- `-gpu auto` lets the emulator decide.

If an emulator crashes with hardware rendering, try:

```bash
emulator -avd Pixel_8_API_35 -gpu swiftshader_indirect
```

### Read-Only Mode

Useful when running multiple instances from the same AVD image:

```bash
emulator -avd Pixel_8_API_35 -read-only
```

For normal manual testing, you usually do not need it.

### Choose a Specific Port

Useful when running several emulators:

```bash
emulator -avd Pixel_8_API_35 -port 5556
```

Then target it with:

```bash
adb -s emulator-5556 shell
```

### Network Delay and Speed Simulation

You can start the emulator with simulated network conditions:

```bash
emulator -avd Pixel_8_API_35 -netdelay gsm -netspeed edge
```

Another example:

```bash
emulator -avd Pixel_8_API_35 -netdelay none -netspeed full
```

This helps test loading states, retries, timeouts, bad network UX, and offline-ish behavior.

### DNS Server Override

Useful for network debugging:

```bash
emulator -avd Pixel_8_API_35 -dns-server 8.8.8.8
```

For security testing, proxying, and DNS experiments, this can help isolate whether a problem is caused by DNS, proxy configuration, or the app itself.

## Waiting for the Emulator to Boot

After starting an emulator, you can wait for it:

```bash
adb wait-for-device
```

Check boot completion:

```bash
adb shell getprop sys.boot_completed
```

A simple loop:

```bash
while [ "$(adb shell getprop sys.boot_completed | tr -d '\r')" != "1" ]; do
  sleep 1
done
```

Then unlock the screen:

```bash
adb shell input keyevent 82
```

This is useful in scripts that install APKs and run automated tests.

## Installing and Removing APKs

Install an APK:

```bash
adb install app-debug.apk
```

Reinstall while keeping app data:

```bash
adb install -r app-debug.apk
```

Install and allow version downgrade:

```bash
adb install -r -d app-debug.apk
```

Install fresh by uninstalling first:

```bash
adb uninstall com.example.app
adb install app-debug.apk
```

Uninstall but keep app data:

```bash
adb uninstall -k com.example.app
```

For QA, the difference between reinstalling and clean installing is important. Many bugs only happen on a fresh install, after an upgrade, or after corrupted old data is kept.

## Clearing App Data

Clear app data:

```bash
adb shell pm clear com.example.app
```

This resets the app like it was freshly installed, but keeps the APK installed.

Use it before testing:

- onboarding;
- login;
- permissions;
- first launch popups;
- feature flags stored locally;
- local cache behavior;
- corrupted app state.

This is one of the most useful QA commands.

## Starting and Stopping Apps

Start an app by package and activity:

```bash
adb shell am start -n com.example.app/.MainActivity
```

If you do not know the activity, inspect the APK or use:

```bash
adb shell cmd package resolve-activity --brief com.example.app
```

Force-stop an app:

```bash
adb shell am force-stop com.example.app
```

Start a URL intent:

```bash
adb shell am start -a android.intent.action.VIEW -d "https://example.com"
```

Test deep links:

```bash
adb shell am start -a android.intent.action.VIEW -d "myapp://profile/123"
```

This is very useful for testing routing, app links, broken deep links, password reset links, payment callbacks, and marketing links.

## Package Inspection

List installed packages:

```bash
adb shell pm list packages
```

Filter by name:

```bash
adb shell pm list packages | grep example
```

Show APK path:

```bash
adb shell pm path com.example.app
```

Show detailed package info:

```bash
adb shell dumpsys package com.example.app
```

Find app version:

```bash
adb shell dumpsys package com.example.app | grep versionName
adb shell dumpsys package com.example.app | grep versionCode
```

Check requested permissions:

```bash
adb shell dumpsys package com.example.app | grep permission
```

This is useful when a bug depends on app version, flavor, build type, permissions, or a wrong APK being installed.

## Working with Permissions

Grant a runtime permission:

```bash
adb shell pm grant com.example.app android.permission.CAMERA
```

Revoke a runtime permission:

```bash
adb shell pm revoke com.example.app android.permission.CAMERA
```

Examples:

```bash
adb shell pm grant com.example.app android.permission.ACCESS_FINE_LOCATION
adb shell pm revoke com.example.app android.permission.POST_NOTIFICATIONS
```

Reset all runtime permissions:

```bash
adb shell pm reset-permissions
```

For QA, this is faster than clicking through settings. It also allows repeatable test scenarios.

Test cases:

- camera allowed;
- camera denied;
- location approximate vs precise;
- notifications denied;
- permission denied forever;
- permission requested at the wrong time;
- app crash after permission revocation.

## Logs: Logcat for Real Bug Reports

Start reading logs:

```bash
adb logcat
```

Clear logs before reproducing a bug:

```bash
adb logcat -c
```

Save logs to a file:

```bash
adb logcat -d > bug-log.txt
```

Collect logs while reproducing:

```bash
adb logcat > bug-live-log.txt
```

Filter by package PID:

```bash
adb shell pidof com.example.app
adb logcat --pid=$(adb shell pidof com.example.app)
```

Filter crashes:

```bash
adb logcat | grep -i "fatal exception"
```

Filter Android runtime crashes:

```bash
adb logcat AndroidRuntime:E *:S
```

Good QA flow:

```bash
adb logcat -c
adb shell am force-stop com.example.app
adb shell monkey -p com.example.app 1
# reproduce the bug manually
adb logcat -d > crash-report-log.txt
```

A bug report with logs is dramatically better than “app crashed”.

## Screenshots and Screen Recording

Take a screenshot:

```bash
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png .
```

Record the screen:

```bash
adb shell screenrecord /sdcard/recording.mp4
```

Stop recording with `Ctrl+C`, then pull it:

```bash
adb pull /sdcard/recording.mp4 .
```

This is useful for visual bugs, animation issues, flickering, wrong transitions, and “hard to explain” bugs.

## Copying Files

Push a file to the device:

```bash
adb push test.json /sdcard/Download/test.json
```

Pull a file from the device:

```bash
adb pull /sdcard/Download/result.json .
```

List files:

```bash
adb shell ls -la /sdcard/Download
```

This helps test import/export flows, media upload, file picker behavior, backups, logs, and generated reports.

## App Data and `run-as`

For debuggable apps, you can inspect private app storage with `run-as`:

```bash
adb shell run-as com.example.app ls -la
```

Copy a database from app storage:

```bash
adb shell run-as com.example.app cp databases/app.db /sdcard/Download/app.db
adb pull /sdcard/Download/app.db .
```

This usually works only for apps built with `android:debuggable="true"`. For production apps, Android protects private app data.

For QA, this is useful for checking:

- local SQLite databases;
- shared preferences;
- cached API responses;
- feature flags;
- migration bugs;
- broken local state after app upgrade.

For cyber research, it helps you understand what the app stores locally and whether sensitive data is stored insecurely in debug builds or test builds.

## Testing App Links and Intents

Open a normal web URL:

```bash
adb shell am start -a android.intent.action.VIEW -d "https://example.com/path"
```

Open a custom scheme:

```bash
adb shell am start -a android.intent.action.VIEW -d "myapp://login/reset?token=test"
```

Send text to another app:

```bash
adb shell am start \
  -a android.intent.action.SEND \
  -t "text/plain" \
  --es android.intent.extra.TEXT "Hello from ADB"
```

Deep links are common sources of bugs:

- app opens the wrong screen;
- app crashes on malformed parameters;
- authentication is bypassed incorrectly;
- expired tokens are accepted;
- links work on one Android version but fail on another;
- web fallback opens instead of the app.

ADB makes these cases easy to repeat.

## Input Simulation

Tap coordinates:

```bash
adb shell input tap 500 1200
```

Type text:

```bash
adb shell input text "hello"
```

Press keys:

```bash
adb shell input keyevent KEYCODE_BACK
adb shell input keyevent KEYCODE_HOME
adb shell input keyevent KEYCODE_APP_SWITCH
adb shell input keyevent KEYCODE_ENTER
```

Swipe:

```bash
adb shell input swipe 500 1600 500 300
```

This is not a replacement for Appium or Espresso, but it is great for quick checks and simple scripts.

## Device State Testing

### Battery State

Unplug device from charging logically:

```bash
adb shell dumpsys battery unplug
```

Set battery level:

```bash
adb shell dumpsys battery set level 15
```

Set charging state:

```bash
adb shell dumpsys battery set status 2
```

Reset battery state:

```bash
adb shell dumpsys battery reset
```

Useful for testing low-battery warnings, background restrictions, sync behavior, and power-related edge cases.

### Doze and App Standby

Force idle mode:

```bash
adb shell dumpsys deviceidle force-idle
```

Exit idle mode:

```bash
adb shell dumpsys deviceidle unforce
```

Put an app into standby-like inactive state:

```bash
adb shell am set-inactive com.example.app true
```

Make it active again:

```bash
adb shell am set-inactive com.example.app false
```

This is useful for testing push notifications, background work, alarms, sync, and delayed jobs.

## Network and Proxy Checks

Check Wi-Fi/network interfaces:

```bash
adb shell ip addr
```

Check routes:

```bash
adb shell ip route
```

Check DNS-related properties:

```bash
adb shell getprop | grep dns
```

Set global HTTP proxy:

```bash
adb shell settings put global http_proxy 192.168.1.10:8080
```

Remove global proxy:

```bash
adb shell settings put global http_proxy :0
```

This is useful with Burp Suite or Charles Proxy. Remember: proxying traffic is not enough if the app uses certificate pinning or does not trust user-installed certificates. In that case, you need to understand Android certificate storage, Network Security Config, and pinning behavior.

## Security Research Use Cases

For mobile security research, ADB is the base layer. Before Frida, Objection, JADX, apktool, or Burp, you usually need ADB.

Common tasks:

```bash
# Install a test APK
adb install target.apk

# Find package name
adb shell pm list packages | grep target

# Find APK path
adb shell pm path com.target.app

# Pull APK from device
adb pull /data/app/~~example/base.apk .

# Start the app
adb shell monkey -p com.target.app 1

# Watch logs
adb logcat

# Check app process
adb shell pidof com.target.app

# Forward a port for tooling
adb forward tcp:27042 tcp:27042
```

ADB helps with:

- APK extraction;
- debug build analysis;
- log inspection;
- checking exported activities indirectly through intent testing;
- testing deep links;
- observing local storage in debuggable builds;
- preparing a device for Frida;
- proxy setup;
- collecting evidence for a security report.

Important line: ADB is powerful, but it does not magically bypass Android security. On a normal non-rooted production device, app private data is still protected. For deeper research, you may need a rooted emulator, a test build, a vulnerable training app, or a lab device.

## QA Bug Report Workflow with ADB

A strong Android bug report can be prepared like this:

```bash
adb devices
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk
adb shell dumpsys package com.example.app | grep versionName
adb logcat -c
```

Then reproduce the bug.

After reproduction:

```bash
adb logcat -d > bug-log.txt
adb shell screencap -p /sdcard/bug.png
adb pull /sdcard/bug.png .
```

Attach:

- steps to reproduce;
- actual result;
- expected result;
- app version;
- Android version;
- device model;
- screenshot or screen recording;
- logcat file;
- network/proxy notes if relevant.

This is how you move from “I found something weird” to a report developers can actually fix.

## Useful ADB Commands Cheat Sheet

```bash
# Devices
adb devices
adb -s emulator-5554 shell

# Device info
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk

# Install / uninstall
adb install app.apk
adb install -r app.apk
adb uninstall com.example.app
adb shell pm clear com.example.app

# Start / stop app
adb shell monkey -p com.example.app 1
adb shell am force-stop com.example.app
adb shell am start -n com.example.app/.MainActivity

# Packages
adb shell pm list packages
adb shell pm path com.example.app
adb shell dumpsys package com.example.app

# Logs
adb logcat
adb logcat -c
adb logcat -d > log.txt
adb logcat AndroidRuntime:E *:S

# Screenshots / video
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png .
adb shell screenrecord /sdcard/video.mp4
adb pull /sdcard/video.mp4 .

# Files
adb push local.txt /sdcard/Download/local.txt
adb pull /sdcard/Download/file.txt .

# Permissions
adb shell pm grant com.example.app android.permission.CAMERA
adb shell pm revoke com.example.app android.permission.CAMERA

# Proxy
adb shell settings put global http_proxy 192.168.1.10:8080
adb shell settings put global http_proxy :0

# Battery
adb shell dumpsys battery unplug
adb shell dumpsys battery set level 10
adb shell dumpsys battery reset

# Emulator
emulator -list-avds
emulator -avd Pixel_8_API_35
emulator -avd Pixel_8_API_35 -no-snapshot-load
emulator -avd Pixel_8_API_35 -wipe-data
emulator -avd Pixel_8_API_35 -no-window -no-audio -no-boot-anim
emulator -avd Pixel_8_API_35 -gpu swiftshader_indirect
emulator -avd Pixel_8_API_35 -netdelay gsm -netspeed edge
```

## Common Problems

### `adb: command not found`

ADB is not in your `PATH`. Use the full path to `adb` or add `platform-tools` to your shell configuration.

### Device Shows as `unauthorized`

Unlock the phone and accept the USB debugging prompt. If it still fails:

```bash
adb kill-server
adb start-server
adb devices
```

You can also revoke USB debugging authorizations in Developer Options and reconnect.

### Emulator Snapshot Fails to Load

Start with a clean boot:

```bash
emulator -avd Pixel_8_API_35 -no-snapshot-load
```

If it is still broken:

```bash
emulator -avd Pixel_8_API_35 -wipe-data
```

### Emulator Is Slow

Check hardware acceleration, GPU mode, RAM, disk speed, and whether another emulator is already running.

Try:

```bash
emulator -avd Pixel_8_API_35 -gpu host
```

If graphics drivers are unstable, try:

```bash
emulator -avd Pixel_8_API_35 -gpu swiftshader_indirect
```

### More Than One Device Connected

Always specify the device:

```bash
adb -s emulator-5554 shell
adb -s R58M123456A install app.apk
```

## Final Thoughts

`adb shell` is one of those tools that separates casual Android testing from serious Android testing. You do not need to memorize every command. You need to understand the main patterns:

- inspect the device;
- control the app;
- reset state;
- collect logs;
- simulate conditions;
- automate boring setup steps;
- make better bug reports;
- prepare the device for deeper security research.

For a QA Engineer, ADB makes testing faster and bug reports stronger. For an Android enthusiast, it opens the system and makes Android less mysterious. For a cyber researcher, it is the basic bridge between your computer, your lab device, and the app you are analyzing.

Learn the boring commands first. They are the ones you will use every day.