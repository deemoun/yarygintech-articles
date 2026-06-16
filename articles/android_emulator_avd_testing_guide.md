---
title: "Android Emulator for Testing: A Practical AVD Command-Line Guide"
description: "A practical Android testing guide focused only on AVD and emulator commands: creating devices, launching them, resetting state, tuning performance, testing networks, using snapshots, and keeping emulator setups repeatable."
pubDate: "2026-06-16"
tags: ["Android", "Android Testing", "Android Emulator", "AVD", "QA", "Mobile Testing", "Command Line"]
draft: false
---
Most Android testers use the emulator through Android Studio, click the green play button, and move on. That is fine for casual work. But if you test Android apps seriously, sooner or later you need to control the emulator directly.

**You want to create a repeatable device. You want to start it clean. You want to simulate a slow network. You want to test with a specific API level. You want to run it without the Android Studio UI. You want to know why the emulator is slow, broken, or secretly using some old snapshot from three days ago.**

That is where the command line becomes useful.

This article is focused only on AVD and emulator work:

- creating Android Virtual Devices
- listing and deleting AVDs
- launching emulators from the terminal
- useful emulator startup flags
- snapshots and clean boots
- RAM, GPU, network, ports, proxy, and storage options
- basic troubleshooting commands

This is not an ADB article. This is not an Appium article. This is not an Espresso article. This is the layer underneath all of that: the virtual Android device itself.

## What Is an AVD?

AVD means Android Virtual Device.

It is basically a device configuration used by the Android Emulator. It defines things like:

- Android version / API level
- system image type
- CPU architecture
- device profile
- screen size
- RAM
- storage
- camera behavior
- graphics mode
- snapshots

The important thing to understand: the AVD is not the emulator binary itself.

The pieces are usually split like this:

```bash
sdkmanager   # installs SDK packages, system images, emulator package
avdmanager   # creates, lists, moves, and deletes AVDs
emulator     # launches an AVD as a running virtual device
```

So the usual flow is:

```bash
# 1. Install the needed SDK pieces
sdkmanager --install "emulator" "platforms;android-36" "system-images;android-36;google_apis;x86_64"

# 2. Create an AVD from a system image
avdmanager create avd -n Pixel_API_36 -k "system-images;android-36;google_apis;x86_64"

# 3. Start it
emulator @Pixel_API_36
```

That is the basic idea.

## Setting Up the Command-Line Paths

Your Android SDK is usually here:

### Linux

```bash
$HOME/Android/Sdk
```

### macOS

```bash
$HOME/Library/Android/sdk
```

### Windows

```powershell
%LOCALAPPDATA%\Android\Sdk
```

On Linux or macOS, add the main tools to your shell config:

```bash
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin"
export PATH="$PATH:$ANDROID_HOME/emulator"
```

On macOS, adjust the SDK path:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin"
export PATH="$PATH:$ANDROID_HOME/emulator"
```

Then check that the tools exist:

```bash
sdkmanager --version
avdmanager --help
emulator -version
```

If `avdmanager` is missing, you probably do not have the Android SDK Command-line Tools package installed correctly.

## Install Emulator and System Images

Before you can create an AVD, you need a system image.

List available packages:

```bash
sdkmanager --list
```

On Linux/macOS, filter system images:

```bash
sdkmanager --list | grep "system-images"
```

On Windows PowerShell or Command Prompt, use:

```powershell
sdkmanager --list | findstr system-images
```

A system image package name usually looks like this:

```text
system-images;android-36;google_apis;x86_64
```

Breakdown:

```text
android-36       # API level
              
google_apis      # image type / tag
x86_64           # CPU architecture
```

Common image types:

```text
default                # plain Android image
google_apis            # Android image with Google APIs
google_apis_playstore  # image with Play Store support, when available
```

Common architectures:

```text
x86_64      # common for Intel/AMD machines
arm64-v8a   # common for Apple Silicon and ARM hosts
```

Install the emulator and one image:

```bash
sdkmanager --install \
  "emulator" \
  "platforms;android-36" \
  "system-images;android-36;google_apis;x86_64"
```

Accept licenses if needed:

```bash
sdkmanager --licenses
```

For Apple Silicon, you will probably use an ARM64 image instead:

```bash
sdkmanager --install \
  "emulator" \
  "platforms;android-36" \
  "system-images;android-36;google_apis;arm64-v8a"
```

The exact package depends on what `sdkmanager --list` shows on your machine.

## List Available Device Profiles

Device profiles define the shape of the virtual device: phone, tablet, foldable, Wear OS device, Android TV device, and so on.

List device profiles:

```bash
avdmanager list device
```

The output contains device IDs and names. You can use one of them when creating an AVD.

Example:

```bash
avdmanager list device | grep -i pixel
```

On Windows:

```powershell
avdmanager list device | findstr /i pixel
```

Do not blindly copy random device IDs from the internet. Device profile names can differ between SDK versions. Always list what is actually available locally.

## Create a Basic AVD

The simplest AVD creation command looks like this:

```bash
avdmanager create avd \
  -n Pixel_API_36 \
  -k "system-images;android-36;google_apis;x86_64"
```

When it asks whether you want to create a custom hardware profile, you can usually answer `no` for a normal testing emulator.

For scripts, you can pipe the answer:

```bash
echo "no" | avdmanager create avd \
  -n Pixel_API_36 \
  -k "system-images;android-36;google_apis;x86_64"
```

If your local `avdmanager` supports the `--device` or `-d` option, you can also specify a hardware profile:

```bash
echo "no" | avdmanager create avd \
  -n Pixel_8_API_36 \
  -k "system-images;android-36;google_apis;x86_64" \
  -d "pixel_8"
```

If that fails, run:

```bash
avdmanager create avd --help
```

Then use the options supported by your installed version. The Android tools change over time, and the help command is often more useful than old blog posts.

## Create Multiple AVDs for Testing

For real QA work, one emulator is usually not enough.

A practical local set could look like this:

```bash
# Modern Google APIs image
avdmanager create avd \
  -n Pixel_API_36 \
  -k "system-images;android-36;google_apis;x86_64"

# Older supported Android version
avdmanager create avd \
  -n Pixel_API_30 \
  -k "system-images;android-30;google_apis;x86_64"

# Play Store image, if available
avdmanager create avd \
  -n Pixel_Play_API_36 \
  -k "system-images;android-36;google_apis_playstore;x86_64"
```

Why keep more than one?

Because bugs often hide in differences:

- new Android permission behavior
- old WebView behavior
- Play Store vs non-Play Store image
- older target devices
- different screen densities
- different storage states

For daily work, I would not create 20 emulators. That becomes a mess. Keep a small, intentional matrix.

## List Existing AVDs

List AVDs through `avdmanager`:

```bash
avdmanager list avd
```

List AVDs through the emulator tool:

```bash
emulator -list-avds
```

The second command is especially useful when you just want the names you can launch:

```bash
emulator @Pixel_API_36
```

## Launch an Emulator

There are two common launch styles:

```bash
emulator -avd Pixel_API_36
```

or:

```bash
emulator @Pixel_API_36
```

I usually prefer the `@name` version because it is shorter and easy to read in scripts.

Example:

```bash
emulator @Pixel_API_36
```

Cold boot:

```bash
emulator @Pixel_API_36 -no-snapshot-load
```

Fast boot without saving new state on exit:

```bash
emulator @Pixel_API_36 -no-snapshot-save
```

Full no-snapshot mode:

```bash
emulator @Pixel_API_36 -no-snapshot
```

Clean device reset:

```bash
emulator @Pixel_API_36 -wipe-data
```

For testing, `-wipe-data` is one of the most useful flags. It removes installed apps, settings, and user data from that virtual device and brings it back to a clean state.

## Useful Startup Flags for QA

Here are the emulator flags I would actually care about as a tester.

### Start Clean

```bash
emulator @Pixel_API_36 -wipe-data
```

Good for:

- onboarding tests
- first launch tests
- permission flow tests
- login state tests
- making sure a bug is not caused by old app data

### Force Cold Boot

```bash
emulator @Pixel_API_36 -no-snapshot-load
```

Good when you do not trust the current emulator state.

Snapshots are useful, but they can also hide bugs. If an app behaves weirdly, do a cold boot before blaming the app.

### Do Not Save State on Exit

```bash
emulator @Pixel_API_36 -no-snapshot-save
```

Good when you want to start from a snapshot but avoid overwriting it with a dirty session.

### Disable Snapshots Completely

```bash
emulator @Pixel_API_36 -no-snapshot
```

Good for repeatable test runs where you want a full boot and no saved state.

### Skip Boot Animation

```bash
emulator @Pixel_API_36 -no-boot-anim
```

Good for faster startup, especially on slower machines or CI boxes.

### Run Without a Window

```bash
emulator @Pixel_API_36 -no-window
```

Good for headless environments.

This is useful in automation pipelines, but it is not very friendly for manual testing because you cannot see the device UI.

### Disable Audio

```bash
emulator @Pixel_API_36 -no-audio
```

Good when audio drivers are causing emulator startup problems or when you do not care about audio in the test session.

### Set RAM

```bash
emulator @Pixel_API_36 -memory 2048
```

Good when you want to quickly override the AVD RAM setting.

Do not just throw 8 GB at every emulator. That is usually not necessary. For many QA tasks, 2-4 GB is enough.

### Set Time Zone

```bash
emulator @Pixel_API_36 -timezone Europe/Paris
```

Good for:

- date formatting bugs
- calendar bugs
- travel apps
- booking flows
- notification timing
- day boundary bugs

Another example:

```bash
emulator @Pixel_API_36 -timezone America/Los_Angeles
```

### Show Verbose Startup Logs

```bash
emulator @Pixel_API_36 -verbose
```

Good when the emulator fails to launch and you need to see which files, images, and settings it is actually using.

### Check Emulator Acceleration

```bash
emulator -accel-check
```

Good when the emulator is painfully slow.

If acceleration is broken, Android Emulator becomes miserable. On Linux you usually want KVM. On macOS it uses Apple's Hypervisor Framework. On Windows, the modern recommended path is Windows Hypervisor Platform.

## GPU and Graphics Options

The default is usually fine:

```bash
emulator @Pixel_API_36 -gpu auto
```

Use host GPU acceleration:

```bash
emulator @Pixel_API_36 -gpu host
```

Use software rendering:

```bash
emulator @Pixel_API_36 -gpu software
```

When would you change this?

Use `-gpu host` when you want better performance and your graphics drivers behave correctly.

Use `-gpu software` when the emulator launches but renders weirdly, flickers, shows black screens, crashes, or behaves badly under GPU acceleration.

For practical testing, I would start with:

```bash
emulator @Pixel_API_36 -gpu auto
```

Then only change it if there is a real problem.

## Network Testing Flags

The emulator can simulate slower mobile networks directly from startup flags.

### Full Speed

```bash
emulator @Pixel_API_36 -netspeed full -netdelay none
```

This is basically the normal fast mode.

### EDGE-like Network

```bash
emulator @Pixel_API_36 -netspeed edge -netdelay gsm
```

Good for checking whether the app becomes unusable on bad connections.

### 3G-like Network

```bash
emulator @Pixel_API_36 -netspeed umts -netdelay umts
```

Good for a more realistic slow-ish mobile network test.

### LTE-like Network

```bash
emulator @Pixel_API_36 -netspeed lte -netdelay lte
```

Good when you want mobile conditions without making the app painfully slow.

### Disable Network Throttling

```bash
emulator @Pixel_API_36 -netfast
```

This is equivalent to full speed with no delay.

## Proxy From Emulator Startup

If you use traffic inspection tools like Charles Proxy, Burp Suite, or mitmproxy, you can start the emulator with an HTTP proxy:

```bash
emulator @Pixel_API_36 -http-proxy 127.0.0.1:8888
```

Or with an explicit scheme:

```bash
emulator @Pixel_API_36 -http-proxy http://127.0.0.1:8888
```

This only routes traffic through the proxy. It does not magically solve certificate trust issues. HTTPS interception still requires certificate setup inside the emulator or system image.

But for emulator startup, this flag is nice because it makes the proxy setting repeatable.

## Capture Emulator Network Traffic

You can capture emulator traffic into a file:

```bash
emulator @Pixel_API_36 -tcpdump /tmp/emulator_traffic.cap
```

Then open the capture in Wireshark:

```bash
wireshark /tmp/emulator_traffic.cap
```

This is useful when you want a lower-level network view and do not want to rely only on a proxy tool.

Be realistic though: most modern app traffic is encrypted. Packet capture is still useful for seeing hosts, timing, DNS behavior, connection attempts, and weird network-level problems.

## DNS Testing

Use custom DNS servers:

```bash
emulator @Pixel_API_36 -dns-server 1.1.1.1,8.8.8.8
```

Good for debugging cases where DNS behaves differently across networks.

For example, an app might work on your normal network but fail on a corporate VPN, hotel Wi-Fi, or a broken ISP DNS setup.

## Ports and Multiple Emulators

Every emulator instance uses console and ADB-related ports. If you run several emulators, port handling matters.

Start one emulator on a specific port:

```bash
emulator @Pixel_API_36 -port 5556
```

Start another one on a different port:

```bash
emulator @Pixel_API_30 -port 5558
```

Keep ports even-numbered and avoid random values. The emulator has a valid port range, and it expects specific port behavior.

For normal manual testing, you rarely need this. For scripts and parallel testing, you eventually will.

## Data, Cache, and Storage

The AVD keeps writable user data in its own data directory.

Default AVD data locations:

### Linux/macOS

```bash
~/.android/avd/AVD_NAME.avd/
```

### Windows

```powershell
C:\Users\YOUR_USER\.android\avd\AVD_NAME.avd\
```

Important files you may see there:

```text
config.ini          # AVD configuration
userdata-qemu.img   # writable user data
cache.img           # cache partition
sdcard.img          # optional emulated SD card
snapshots.img       # snapshot storage
```

For most testers, the file to know is:

```text
config.ini
```

You can inspect it:

```bash
cat ~/.android/avd/Pixel_API_36.avd/config.ini
```

You can also use:

```bash
emulator @Pixel_API_36 -verbose
```

That will show what paths and images the emulator is using.

## Move or Rename an AVD

Rename an AVD:

```bash
avdmanager move avd -n Pixel_API_36 -r Pixel_API_36_Clean
```

Move it to another path:

```bash
avdmanager move avd \
  -n Pixel_API_36 \
  -p /mnt/fastssd/android-avds/Pixel_API_36.avd
```

This can be useful if your home directory is small and emulator images are eating your disk.

## Delete an AVD

Delete an AVD:

```bash
avdmanager delete avd -n Pixel_API_36
```

Before deleting, list your AVDs:

```bash
avdmanager list avd
```

This sounds obvious, but it is very easy to delete the wrong emulator if your names are lazy.

Use names that explain the purpose:

```text
Pixel_API_36_GoogleAPIs
Pixel_API_30_Regression
Pixel_API_36_PlayStore
Tablet_API_35
```

Bad names:

```text
test
new
android
pixel
copy2
```

You will hate yourself later.

## Snapshots: Useful, but Dangerous

Snapshots are great when you want fast startup.

But they are dangerous when you need reliable test results.

A snapshot can preserve:

- app state
- login sessions
- weird cached data
- old permissions
- broken network state
- old system time behavior
- half-finished setup

For manual exploratory testing, snapshots are fine.

For clean regression testing, be careful.

Useful snapshot-related flags:

```bash
# Do not load a snapshot; cold boot instead
emulator @Pixel_API_36 -no-snapshot-load

# Do not save the current session on exit
emulator @Pixel_API_36 -no-snapshot-save

# Disable snapshot load/save completely
emulator @Pixel_API_36 -no-snapshot

# List snapshots
emulator @Pixel_API_36 -snapshot-list

# Start from a named snapshot
emulator @Pixel_API_36 -snapshot clean_login_state
```

My practical rule:

Use snapshots for speed. Use clean boots for trust.

## Headless Emulator Recipe

For CI-style use:

```bash
emulator @Pixel_API_36 \
  -no-window \
  -no-audio \
  -no-boot-anim \
  -no-snapshot \
  -gpu software
```

This is not the only possible setup, but it is a sane starting point.

For local manual testing, I would not use this. You want to see the device.

## Clean Manual Testing Recipe

When I want a clean emulator for manual QA:

```bash
emulator @Pixel_API_36 \
  -wipe-data \
  -no-snapshot \
  -no-boot-anim \
  -gpu auto
```

This gives you a clean state and avoids snapshot weirdness.

Yes, it boots slower. But for debugging suspicious issues, that is a good trade.

## Fast Daily Testing Recipe

When I just want to work quickly:

```bash
emulator @Pixel_API_36 \
  -no-boot-anim \
  -gpu auto
```

If your snapshot state is clean, this is fine.

If your emulator starts acting possessed, stop being lazy and run:

```bash
emulator @Pixel_API_36 -wipe-data -no-snapshot
```

## Bad Network Testing Recipe

```bash
emulator @Pixel_API_36 \
  -netspeed edge \
  -netdelay gsm
```

Use this for:

- loading states
- retry logic
- timeout behavior
- offline-ish user experience
- progress indicators
- flaky API behavior

A lot of apps look good only on perfect Wi-Fi. That is not real life.

## Proxy Testing Recipe

```bash
emulator @Pixel_API_36 \
  -http-proxy 127.0.0.1:8888 \
  -no-snapshot-load
```

Good for starting the emulator already pointed at your proxy tool.

Again: proxy is not certificate trust. For HTTPS inspection, certificate setup is a separate topic.

## Debug Startup Problems

If the emulator does not start, do not guess immediately. Run it loudly:

```bash
emulator @Pixel_API_36 -verbose
```

Check acceleration:

```bash
emulator -accel-check
```

Check help for the option you are using:

```bash
emulator -help
emulator -help-all
emulator -help-netspeed
emulator -help-datadir
```

Common causes of emulator pain:

- wrong system image for your CPU architecture
- broken or disabled virtualization
- bad GPU driver behavior
- corrupted snapshot
- not enough disk space
- old emulator package
- Unicode or weird characters in paths on Windows
- stale AVD config copied from another machine

The lazy fix is deleting and recreating the AVD. Sometimes that is honestly the right fix. But if the problem keeps coming back, check acceleration, GPU mode, SDK paths, and snapshots.

## A Small Emulator Command Cheat Sheet

```bash
# List installed/available SDK packages
sdkmanager --list

# Install emulator and system image
sdkmanager --install "emulator" "platforms;android-36" "system-images;android-36;google_apis;x86_64"

# Accept SDK licenses
sdkmanager --licenses

# List device profiles
avdmanager list device

# List existing AVDs
avdmanager list avd
emulator -list-avds

# Create AVD
avdmanager create avd -n Pixel_API_36 -k "system-images;android-36;google_apis;x86_64"

# Create AVD non-interactively
echo "no" | avdmanager create avd -n Pixel_API_36 -k "system-images;android-36;google_apis;x86_64"

# Start emulator
emulator @Pixel_API_36

# Cold boot
emulator @Pixel_API_36 -no-snapshot-load

# Wipe data
emulator @Pixel_API_36 -wipe-data

# No snapshots
emulator @Pixel_API_36 -no-snapshot

# Faster boot
emulator @Pixel_API_36 -no-boot-anim

# Headless
emulator @Pixel_API_36 -no-window -no-audio -no-boot-anim -no-snapshot

# Slow network
emulator @Pixel_API_36 -netspeed edge -netdelay gsm

# Proxy
emulator @Pixel_API_36 -http-proxy 127.0.0.1:8888

# Packet capture
emulator @Pixel_API_36 -tcpdump /tmp/emulator.cap

# Custom DNS
emulator @Pixel_API_36 -dns-server 1.1.1.1,8.8.8.8

# GPU options
emulator @Pixel_API_36 -gpu auto
emulator @Pixel_API_36 -gpu host
emulator @Pixel_API_36 -gpu software

# Check acceleration
emulator -accel-check

# Verbose startup
emulator @Pixel_API_36 -verbose

# Rename AVD
avdmanager move avd -n Pixel_API_36 -r Pixel_API_36_Clean

# Delete AVD
avdmanager delete avd -n Pixel_API_36
```

## Final Thoughts

The Android Emulator is not just a thing Android Studio opens for you. It is a proper testing tool by itself.

If you know the AVD and emulator commands, you can build cleaner test environments, reproduce bugs more reliably, simulate bad networks, run headless devices, and stop blaming the app when the real problem is a dirty emulator snapshot.

For QA work, the most important commands are not exotic.

Learn these first:

```bash
avdmanager list avd
emulator -list-avds
emulator @AVD_NAME
emulator @AVD_NAME -wipe-data
emulator @AVD_NAME -no-snapshot-load
emulator @AVD_NAME -no-snapshot
emulator @AVD_NAME -netspeed edge -netdelay gsm
emulator @AVD_NAME -verbose
emulator -accel-check
```

That alone already gives you much better control than clicking random buttons in the IDE and hoping the virtual device behaves.