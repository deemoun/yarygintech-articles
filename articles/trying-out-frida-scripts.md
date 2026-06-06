---
title: "Trying Out Frida Scripts: Root Bypass and App Crash Experiments"
description: "A practical Android security lab article showing how Frida can hook Java methods at runtime, bypass simple root detection, and deliberately crash a test app for learning purposes."
pubDate: 2026-06-06
tags: ["Android", "Frida", "Mobile Security", "Reverse Engineering", "Root Detection", "Dynamic Analysis", "Cybersecurity"]
draft: false
---
Frida is one of those tools that looks almost magical the first time you use it.

You launch an Android app, attach a script, and suddenly you can change what methods return, block suspicious checks, print internal values, or even force the app to crash at a specific point.

That is not magic, of course. It is runtime instrumentation.

**In this article, we will try two small Frida scripts:**

1. **A script that bypasses simple root detection checks.**
2. **A script that intentionally crashes an Android app.**

The point is not to attack random apps. The point is to understand how Android apps behave at runtime and how fragile many defensive checks can be when they only run inside the app process.

Use this only in your own lab apps, intentionally vulnerable apps, CTF apps, or apps where you have permission to test.

---

## What Frida Actually Does

Frida lets you inject JavaScript into a running process and hook functions while the app is running.

On Android, this usually means you can hook Java classes and methods such as:

- `java.io.File.exists()`
- `java.lang.Runtime.exec()`
- `android.os.SystemProperties.get()`
- `android.app.Application.onCreate()`
- `android.app.Activity.onCreate()`

That means you can observe what the app is doing, change return values, block calls, or throw exceptions.

For QA and security testing, this is extremely useful because you can test how the app behaves under strange or hostile runtime conditions without rebuilding the APK every time.

---

## Lab Setup

For this kind of experiment, you usually need:

- An Android emulator or rooted test device.
- Frida installed on your computer.
- `frida-server` running on the Android side if you are using a rooted setup.
- A test APK that you are allowed to analyze.
- Basic ADB access.

Example commands:

```bash
adb devices
frida-ps -U
```

If Frida sees the device, you should get a process list from the Android device.

To spawn an app with a script:

```bash
frida -U -f com.example.app -l script.js
```

To attach to an already running app:

```bash
frida -U -n com.example.app -l script.js
```

Replace `com.example.app` with the actual package name of your lab app.

---

## Example 1: Bypassing Simple Root Detection

A lot of Android apps try to detect root by checking for common files, properties, or shell commands.

Typical checks include:

- Looking for `/system/xbin/su`
- Looking for `busybox`
- Looking for `Superuser.apk`
- Checking whether `Build.TAGS` contains `test-keys`
- Running commands like `su`
- Reading system properties through `getprop`

The following Frida script tries to interfere with those checks at runtime.

Create a file named `root-bypass.js`:

```javascript
Java.perform(() => {
    const String = Java.use("java.lang.String");
    const Runtime = Java.use("java.lang.Runtime");
    const SystemProperties = Java.use("android.os.SystemProperties");

    // Bypass file existence checks (su, busybox, etc.)
    const File = Java.use("java.io.File");
    File.exists.implementation = function () {
        const path = this.getAbsolutePath();
        if (path.includes("su") || path.includes("busybox") || path.includes("Superuser.apk")) {
            console.log("Bypassing file.exists() for:", path);
            return false;
        }
        return this.exists();
    };

    // Bypass Build.TAGS check
    const Build = Java.use("android.os.Build");
    Build.TAGS.value = "release-keys";

    // Bypass Runtime.exec("su") or similar
    Runtime.exec.overload("[Ljava.lang.String;").implementation = function (cmdArray) {
        const cmd = cmdArray.join(" ");
        if (cmd.includes("su")) {
            console.log("Blocking Runtime.exec:", cmd);
            throw Java.use("java.io.IOException").$new("Permission denied");
        }
        return this.exec(cmdArray);
    };

    Runtime.exec.overload("java.lang.String").implementation = function (cmd) {
        if (cmd.includes("su")) {
            console.log("Blocking Runtime.exec:", cmd);
            throw Java.use("java.io.IOException").$new("Permission denied");
        }
        return this.exec(cmd);
    };

    // Bypass getprop-based root checks
    SystemProperties.get.overload("java.lang.String").implementation = function (key) {
        console.log("getprop called with:", key);
        return "";
    };
});
```

Run it like this:

```bash
frida -U -f com.example.app -l root-bypass.js
```

If the app performs root checks during startup, spawning the app with `-f` is usually better than attaching later. That gives your script a chance to hook methods before the app finishes its first security checks.

---

## What This Script Hooks

### 1. `File.exists()`

The script hooks `java.io.File.exists()` and checks the path being tested.

If the path contains suspicious root-related names such as `su`, `busybox`, or `Superuser.apk`, the script returns `false`.

So if the app asks:

```java
new File("/system/xbin/su").exists()
```

Frida can force the answer to be:

```java
false
```

Even if the file exists.

This is why file-based root detection is weak by itself.

---

### 2. `Build.TAGS`

Some apps check this value:

```java
android.os.Build.TAGS
```

On some rooted or custom systems, it may contain `test-keys`. Some apps treat that as a root or tampering signal.

The script changes it to:

```text
release-keys
```

Again, this does not make the device magically clean. It only changes what the app sees at runtime.

---

### 3. `Runtime.exec()`

Some apps try to execute `su` directly:

```java
Runtime.getRuntime().exec("su")
```

The script hooks common `Runtime.exec()` overloads and throws an `IOException` when the command contains `su`.

From the app's point of view, it looks like the command failed.

That is often enough to bypass simple checks.

---

### 4. `SystemProperties.get()`

Some root checks read Android system properties, similar to running `getprop` from the shell.

The script hooks:

```java
android.os.SystemProperties.get(String key)
```

and returns an empty string.

This is a very aggressive approach. In a real analysis, you may want to return clean-looking values only for specific keys instead of blanking every property request.

For a first lab experiment, though, it makes the behavior easy to see.

---

## Important Weakness in This Example

This script is useful for learning, but it is not a universal root bypass.

Real apps may also use:

- Native C/C++ checks through JNI.
- Direct syscalls.
- `/proc` checks.
- SELinux context checks.
- Magisk detection.
- Frida detection.
- Play Integrity API or server-side validation.
- Obfuscated custom checks.

So do not treat this as a magic script. Treat it as a clean first example of how runtime method hooking works.

That is the correct mindset.

---

## Example 2: Crashing an Android App with Frida

Now let's look at the opposite idea.

Instead of bypassing something, we intentionally break the app.

Why would this be useful?

Because controlled crashes are useful in testing. You can use them to check:

- Whether crash reporting works.
- Whether the app restarts cleanly.
- Whether sensitive data is written to logs during a crash.
- Whether the app handles lifecycle failures safely.
- Whether the backend receives crash telemetry.

Again, this should be done only against your own app, a lab app, or an app you are allowed to test.

Create a file named `crash-app.js`:

```javascript
Java.perform(() => {
    const Application = Java.use("android.app.Application");
    const Activity = Java.use("android.app.Activity");

    // Sabotage Application.onCreate()
    Application.onCreate.implementation = function () {
        console.log("Crashing Application.onCreate()");
        throw Java.use("java.lang.RuntimeException").$new("Crashed by Frida in Application.onCreate()");
    };

    // Optionally sabotage Activity.onCreate()
    Activity.onCreate.overload("android.os.Bundle").implementation = function (bundle) {
        console.log("Crashing Activity.onCreate()");
        throw Java.use("java.lang.RuntimeException").$new("Crashed by Frida in Activity.onCreate()");
    };
});
```

Run it like this:

```bash
frida -U -f com.example.app -l crash-app.js
```

If the hook works, the app should crash during startup.

---

## What This Crash Script Does

Android apps usually start from the `Application` class and then launch an `Activity`.

This script hooks two lifecycle methods:

```java
Application.onCreate()
```

and:

```java
Activity.onCreate(Bundle bundle)
```

Instead of letting those methods run normally, the script throws a `RuntimeException`.

That is enough to kill the app process in most normal cases.

This is not subtle. It is a blunt-force test.

And that is fine. Sometimes blunt-force testing is exactly what you want in a lab.

---

## Watching the Crash in Logcat

Open another terminal and run:

```bash
adb logcat | grep -i "Crashed by Frida"
```

Or use a broader crash filter:

```bash
adb logcat | grep -i "RuntimeException"
```

You should see the exception message from your Frida script.

This is a good way to confirm that your hook actually executed.

---

## When to Spawn and When to Attach

Use spawn mode when the target behavior happens during startup:

```bash
frida -U -f com.example.app -l script.js
```

Use attach mode when the app is already running and you want to hook behavior that happens later:

```bash
frida -U -n com.example.app -l script.js
```

For root detection, spawn mode is usually better because many apps check the environment immediately.

For button clicks, API calls, or later screen actions, attach mode can be enough.

---

## Common Problems

### Frida cannot find the app

Check the package name:

```bash
adb shell pm list packages | grep example
```

Then use the exact package name with Frida.

---

### Script loads too late

Use `-f` spawn mode instead of attaching to an already running process.

---

### The app still detects root

The app may be using checks that this script does not cover.

Try to identify the actual check first. Do not randomly paste bigger bypass scripts and hope for the best. That is how you learn nothing.

Use tools like:

- JADX for reading Java/Kotlin code.
- apktool for smali-level inspection.
- Logcat for runtime clues.
- Frida tracing for method discovery.
- Objection for interactive exploration.

---

### The app uses native checks

Java hooks will not catch everything.

If the app checks root from native code, you may need to hook native functions such as file access, process execution, or string comparisons. That is a more advanced topic.

---

## A Better Learning Workflow

A good workflow looks like this:

1. Run the app normally.
2. Observe the behavior.
3. Read the APK in JADX.
4. Find the method responsible for the check.
5. Write a tiny Frida hook for that exact method.
6. Run the app again.
7. Confirm the behavior changed.
8. Document what happened.

This is much better than using a huge copy-pasted script that hooks everything.

Small scripts teach you more.

---

## Defensive Lesson for Developers

If you are developing Android apps, the lesson is simple:

Client-side checks can be bypassed.

Root detection, emulator detection, anti-debug checks, and Frida detection can raise the cost of tampering, but they should not be your only protection.

Do not put your real security boundary only inside the APK.

For serious protection, combine multiple layers:

- Server-side validation.
- Proper authentication and authorization.
- Short-lived tokens.
- Integrity checks where appropriate.
- Secure storage.
- Least privilege API design.
- Logging and anomaly detection.
- No hardcoded secrets in the APK.

Frida is a reminder that the user's device is not your trusted backend.

---

## Final Thoughts

These two scripts are simple, but they show the core idea behind Frida very well.

The root bypass script changes what the app sees.

The crash script changes how the app behaves.

That is the power of runtime instrumentation.

For QA engineers, mobile security learners, and Android reverse engineering beginners, this is one of the best ways to understand what really happens inside an app after it starts running.

Do not just watch the app from the outside.

Hook it. Break it. Observe it. Then explain what happened.

That is where the learning starts.