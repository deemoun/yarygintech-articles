---
title: "Debloating the Lenovo Idea Tab (TB336FU): A Practical ADB Guide"
description: A practical guide to safely debloating the Lenovo Idea Tab
  TB336FU and modern Lenovo tablets using ADB, without root or
  bootloader unlock.
pubDate: 2026-07-02
tags: ["Android", "ADB", "Android Debloat", "Bloatware", "Lenovo"]
draft: false
---
One of the first things I do with a new Android device is remove
everything I don't actually use. Modern Lenovo tablets are surprisingly
clean compared to many competitors, but there is still plenty of
software that can safely disappear.

This guide documents the process I used on the Lenovo Idea Tab TB336FU
(Android 15 / ZUI 17). The same workflow is applicable to many recent
Lenovo tablets.

> No root required.
>
> No custom ROM required.
>
> No bootloader unlock required.

## Before You Start

Enable:

-   Developer Options
-   USB Debugging

Verify ADB:

``` bash
adb devices
```

## Backup Package Lists

Always save the package list before touching anything.

``` bash
adb shell pm list packages | sort > remaining_packages.txt
adb shell pm list packages -s | sort > system_packages.txt
adb shell pm list packages -3 | sort > user_packages.txt
adb shell pm list packages -f | sort > packages_with_paths.txt
adb shell pm list packages -d | sort > disabled_packages.txt
```

## Safe Removal

These applications were removed with:

``` bash
adb shell pm uninstall --user 0 PACKAGE_NAME
```

Examples:

-   Facebook services
-   Opera
-   Instagram
-   YouTube Music
-   Google TV
-   Lenovo What's New
-   Google Feedback
-   Traceur
-   Google Restore
-   Gmail
-   Google Calendar
-   Google Maps

## Safe Disable

These were disabled first instead of being removed:

``` bash
adb shell pm disable-user --user 0 PACKAGE_NAME
```

Examples:

-   Lenovo User Experience
-   Lenovo Service Framework
-   Lenovo ID
-   Lenovo Tablet Center
-   Lenovo Weather
-   Google Assistant
-   Google Search
-   Meet
-   Drive
-   Photos
-   Digital Wellbeing
-   Motorola Ready For components

## Packages You Should Leave Alone

If you use the Lenovo Pen, keep these enabled:

-   com.lenovo.penservice
-   com.zui.pengen
-   com.zui.notes
-   com.myscript.nebo.lenovo

Also avoid removing:

-   com.android.systemui
-   com.android.settings
-   com.google.android.gms
-   com.google.android.gsf
-   com.lenovo.ota

## Restoring Packages

Packages removed with `--user 0` are not permanently deleted.

Restore one:

``` bash
adb shell cmd package install-existing PACKAGE_NAME
```

Re-enable:

``` bash
adb shell pm enable PACKAGE_NAME
```

## Debugging Running Services

List Lenovo services:

``` bash
adb shell dumpsys activity services | grep -i lenovo
```

Inspect a package:

``` bash
adb shell dumpsys package PACKAGE_NAME
```

Show APK path:

``` bash
adb shell pm path PACKAGE_NAME
```

Show package permissions:

``` bash
adb shell dumpsys package PACKAGE_NAME | grep permission
```

Memory usage:

``` bash
adb shell dumpsys meminfo PACKAGE_NAME
```

Running processes:

``` bash
adb shell ps -A
```

## My Workflow

1.  Backup package lists.
2.  Remove obvious bloat.
3.  Disable questionable packages.
4.  Reboot.
5.  Verify pen, camera, OTA, launcher and Play Store.
6.  Only then consider permanent removal for the current user.

## Final Thoughts

The TB336FU turned out to be much cleaner than expected. Lenovo
separates many features into standalone packages, making ADB debloating
relatively safe if approached carefully.

The biggest recommendation is simple:

Disable first. Use the tablet for a few days.

Only remove packages after you're confident they are unnecessary.

That approach dramatically reduces the chance of breaking useful
features while still ending up with a much cleaner Android experience.