---
title: "iOS Simulator for Testing: Practical Command-Line Guide"
description: "A practical guide for QA engineers and mobile testers: creating, booting, resetting, installing apps, launching apps, setting permissions, simulating location, recording video, and managing iOS Simulator devices from the command line."
pubDate: 2026-06-16
tags: ["iOS", "Testing", "Device Simulator", "QA Engineer"]
draft: false
---
Android has the Android Emulator. iOS has the iOS Simulator.

That sounds like a small naming difference, but it matters.

The Android Emulator emulates an Android device environment. The iOS Simulator runs iOS, iPadOS, watchOS, tvOS, and visionOS simulator runtimes on your Mac. It is not the same thing as a real iPhone. It is not a perfect hardware replacement. It is a fast testing tool built into Xcode.

For QA work, that is still extremely useful.

You can create clean simulator devices, boot them from the terminal, install simulator builds, launch apps with arguments, reset permissions, fake location, send test push notifications, take screenshots, record videos, and prepare repeatable test environments without clicking around Xcode all day.

This article is focused only on the Simulator side: devices, runtimes, commands, and practical testing workflow.

No Appium deep dive.  
No XCTest tutorial.  
No real-device provisioning pain.  
Just the iOS Simulator and the command line.

---

## Simulator vs Emulator: the first thing to understand

In casual conversation people say “iOS emulator,” but Apple’s official tool is **Simulator**.

The distinction is useful:

- Simulator is great for fast UI and functional testing.
- Simulator is good for testing multiple screen sizes.
- Simulator is good for checking app behavior with clean state.
- Simulator is good for permissions, screenshots, location, deep links, and basic push notification simulation.
- Simulator is not a real iPhone.
- Simulator is not a perfect performance test environment.
- Simulator does not fully reproduce real hardware, modem, camera, Bluetooth, sensors, Secure Enclave, thermal behavior, or App Store installation.

So the mindset should be simple:

Use Simulator to move fast.  
Use real devices before you trust the release.

---

## What you need

You need a Mac with Xcode installed.

The main tools are:

```bash
xcrun
simctl
xcodebuild
```

You normally do not call `simctl` directly from some random location. You call it through `xcrun`:

```bash
xcrun simctl
```

`xcrun` finds the correct developer tool inside the currently selected Xcode installation.

Check which Xcode your command line is using:

```bash
xcode-select -p
```

Example output:

```bash
/Applications/Xcode.app/Contents/Developer
```

If you have multiple Xcode versions installed, select the one you want:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

Check Xcode version:

```bash
xcodebuild -version
```

Run the first-launch setup if Xcode was just installed:

```bash
sudo xcodebuild -runFirstLaunch
```

If `xcrun simctl` fails, the problem is usually one of these:

```bash
xcode-select -p
xcodebuild -version
xcrun simctl help
```

If the active developer directory points to the wrong Xcode, fix it with `xcode-select`.

---

## Install or check iOS Simulator runtimes

Xcode usually installs the current iOS Simulator runtime, but not always every older runtime.

List installed runtimes:

```bash
xcrun simctl list runtimes
```

List only available iOS runtimes:

```bash
xcrun simctl list runtimes | grep iOS
```

You may see something like this:

```bash
iOS 18.5 (18.5 - 22F76) - com.apple.CoreSimulator.SimRuntime.iOS-18-5
```

The important part is the runtime identifier:

```bash
com.apple.CoreSimulator.SimRuntime.iOS-18-5
```

You use that identifier when creating a simulator from the command line.

Modern Xcode versions can also download platform components from the command line:

```bash
xcodebuild -downloadPlatform iOS
```

For most manual testers, the easier route is:

```text
Xcode → Settings → Components / Platforms
```

Then install the iOS runtime you need.

For CI machines, prefer command-line installation and script the setup.

---

## List available simulator devices

Show everything:

```bash
xcrun simctl list
```

Show only devices:

```bash
xcrun simctl list devices
```

Show only available devices:

```bash
xcrun simctl list devices available
```

Show device types:

```bash
xcrun simctl list devicetypes
```

Show runtimes:

```bash
xcrun simctl list runtimes
```

Device types are models like:

```bash
iPhone 15
iPhone 15 Pro
iPhone 16
iPad Pro 11-inch
```

But the command line needs the internal identifier, which looks like this:

```bash
com.apple.CoreSimulator.SimDeviceType.iPhone-16
```

Runtimes also have internal identifiers:

```bash
com.apple.CoreSimulator.SimRuntime.iOS-18-5
```

Do not blindly copy old examples from the internet. Device type IDs and runtime IDs change as Xcode changes. Always list what is installed on your machine.

---

## Create a simulator device

The command format is:

```bash
xcrun simctl create "<Name>" "<DeviceTypeID>" "<RuntimeID>"
```

Example:

```bash
xcrun simctl create "QA iPhone"   "com.apple.CoreSimulator.SimDeviceType.iPhone-16"   "com.apple.CoreSimulator.SimRuntime.iOS-18-5"
```

If it succeeds, the command prints a simulator UDID:

```bash
A1B2C3D4-1111-2222-3333-ABCDEF123456
```

That UDID is the safest way to refer to the simulator later.

Names are convenient, but names can collide. UDIDs do not.

A more reliable workflow:

```bash
DEVICE_TYPE="com.apple.CoreSimulator.SimDeviceType.iPhone-16"
RUNTIME="com.apple.CoreSimulator.SimRuntime.iOS-18-5"

UDID=$(xcrun simctl create "QA iPhone" "$DEVICE_TYPE" "$RUNTIME")

echo "$UDID"
```

Now you can use `$UDID` in the rest of your script.

---

## Boot a simulator

Boot by name:

```bash
xcrun simctl boot "QA iPhone"
```

Boot by UDID:

```bash
xcrun simctl boot A1B2C3D4-1111-2222-3333-ABCDEF123456
```

Open the Simulator app UI:

```bash
open -a Simulator
```

Wait until the simulator is fully booted:

```bash
xcrun simctl bootstatus "QA iPhone" -b
```

Or with UDID:

```bash
xcrun simctl bootstatus "$UDID" -b
```

This is very useful in scripts. Without `bootstatus`, your next command may run too early.

A practical boot script:

```bash
UDID="A1B2C3D4-1111-2222-3333-ABCDEF123456"

xcrun simctl boot "$UDID"
xcrun simctl bootstatus "$UDID" -b
open -a Simulator
```

---

## Use the `booted` shortcut

If only one simulator is running, you can use:

```bash
booted
```

Example:

```bash
xcrun simctl io booted screenshot screenshot.png
```

That means: run the command against the currently booted simulator.

This is convenient for manual work.

For scripts and CI, prefer UDID. If multiple simulators are booted, `booted` can become ambiguous.

---

## Shutdown, erase, and delete simulators

Shutdown one simulator:

```bash
xcrun simctl shutdown "QA iPhone"
```

Shutdown all simulators:

```bash
xcrun simctl shutdown all
```

Erase a simulator:

```bash
xcrun simctl shutdown "QA iPhone"
xcrun simctl erase "QA iPhone"
```

Erase all simulators:

```bash
xcrun simctl shutdown all
xcrun simctl erase all
```

Delete a simulator:

```bash
xcrun simctl delete "QA iPhone"
```

Delete unavailable simulators:

```bash
xcrun simctl delete unavailable
```

That last command is useful after Xcode upgrades. Old simulators can remain tied to runtimes you no longer have.

For QA, there are three levels of cleanup:

```bash
# light cleanup: close running simulators
xcrun simctl shutdown all

# clean device state, but keep devices
xcrun simctl erase all

# remove broken/unavailable old devices
xcrun simctl delete unavailable
```

Do not delete everything unless you actually want to recreate your simulator set.

---

## Install an app into Simulator

Important: you install a **Simulator build**, usually a `.app` bundle built for the simulator.

You normally cannot install a normal App Store `.ipa` into Simulator. Real-device builds and simulator builds are different.

Install an app:

```bash
xcrun simctl install booted /path/to/MyApp.app
```

Install into a specific simulator:

```bash
xcrun simctl install "$UDID" /path/to/MyApp.app
```

Uninstall:

```bash
xcrun simctl uninstall booted com.example.myapp
```

The last argument is the app bundle ID.

Example bundle ID:

```bash
com.yourcompany.MyApp
```

If you do not know the bundle ID, ask the developer or inspect the app’s `Info.plist`.

---

## Launch an app

Launch by bundle ID:

```bash
xcrun simctl launch booted com.example.myapp
```

Launch and show console output:

```bash
xcrun simctl launch --console booted com.example.myapp
```

Launch and terminate the already running process first:

```bash
xcrun simctl launch --terminate-running-process booted com.example.myapp
```

Launch with environment variables:

```bash
xcrun simctl launch   --console   --env QA_MODE=1   booted   com.example.myapp
```

Pass launch arguments to the app:

```bash
xcrun simctl launch booted com.example.myapp --uitest-mode --mock-api
```

Everything after the bundle ID is passed to the app as process arguments.

This is useful when developers add test switches like:

```text
--reset-on-start
--use-staging
--mock-location
--disable-animations
--uitest-mode
```

A good QA/dev agreement is to define a few stable launch arguments for testing. It makes the app easier to automate and easier to debug.

---

## Open URLs and deep links

Open a normal URL:

```bash
xcrun simctl openurl booted "https://example.com"
```

Open a deep link:

```bash
xcrun simctl openurl booted "myapp://settings/profile"
```

Open a universal link:

```bash
xcrun simctl openurl booted "https://example.com/product/123"
```

This is excellent for testing:

- onboarding links
- password reset links
- payment return links
- magic login links
- marketing campaign links
- universal link routing
- custom scheme routing

If a link works in a browser but not through `simctl openurl`, that is already useful information.

---

## Take screenshots

Take a screenshot from the booted simulator:

```bash
xcrun simctl io booted screenshot screenshot.png
```

Save it with a timestamp:

```bash
xcrun simctl io booted screenshot "screenshot-$(date +%Y%m%d-%H%M%S).png"
```

For bug reports, this is faster than clicking around the Simulator UI.

For content and App Store-style screenshots, combine screenshots with status bar overrides.

---

## Record video

Record video:

```bash
xcrun simctl io booted recordVideo video.mp4
```

Stop recording with:

```text
Control-C
```

Overwrite existing file:

```bash
xcrun simctl io booted recordVideo --force video.mp4
```

Use H.264 if you need broader compatibility:

```bash
xcrun simctl io booted recordVideo --codec=h264 video.mp4
```

This is useful for:

- bug reports
- UI glitches
- animation problems
- onboarding flows
- flaky test reproduction
- short demos for developers

A video often explains a bug better than five paragraphs.

---

## Override the status bar

The status bar can make screenshots messy. Random time, random battery, random Wi-Fi state. For clean screenshots, override it.

Example:

```bash
xcrun simctl status_bar booted override   --time "9:41"   --batteryState charged   --batteryLevel 100   --wifiBars 3   --cellularBars 4
```

Clear the override:

```bash
xcrun simctl status_bar booted clear
```

This is useful for:

- App Store screenshots
- blog screenshots
- tutorial screenshots
- consistent QA documentation
- visual comparison

If a specific status bar option does not work on a specific iOS runtime, check:

```bash
xcrun simctl status_bar help
```

Simulator behavior can change between Xcode and iOS runtime versions.

---

## Manage permissions from the command line

Permissions are one of the best reasons to use `simctl`.

You can test what happens when the user grants, denies, or resets access.

Grant camera permission:

```bash
xcrun simctl privacy booted grant camera com.example.myapp
```

Revoke camera permission:

```bash
xcrun simctl privacy booted revoke camera com.example.myapp
```

Grant location permission:

```bash
xcrun simctl privacy booted grant location com.example.myapp
```

Revoke photos permission:

```bash
xcrun simctl privacy booted revoke photos com.example.myapp
```

Reset all permissions for one app:

```bash
xcrun simctl privacy booted reset all com.example.myapp
```

Check available services:

```bash
xcrun simctl privacy help
```

Useful permission test cases:

- first launch with no permissions
- user grants permission
- user denies permission
- user revokes permission later in Settings
- app explains why permission is needed
- app does not crash when permission is missing
- app recovers when permission is granted later

Do not only test the happy path. Permission denial is a normal user behavior.

---

## Simulate location

For many years, location simulation was mostly done through Xcode menus or GPX files. Modern Xcode versions also support location control through `simctl`.

Set a location:

```bash
xcrun simctl location booted set 37.7749,-122.4194
```

That example is San Francisco.

Set Mexico City:

```bash
xcrun simctl location booted set 19.4326,-99.1332
```

Set New York:

```bash
xcrun simctl location booted set 40.7128,-74.0060
```

Clear location:

```bash
xcrun simctl location booted clear
```

Check help:

```bash
xcrun simctl location help
```

This is useful for testing:

- maps
- delivery apps
- weather apps
- region restrictions
- nearby search
- geofencing
- location permission flows
- city-specific content

For movement and routes, GPX files are still useful, especially inside Xcode test plans.

---

## Add photos and videos to Simulator

Add media:

```bash
xcrun simctl addmedia booted ~/Desktop/photo.jpg
```

Add multiple files:

```bash
xcrun simctl addmedia booted ~/Desktop/test-media/*.jpg
```

Add a video:

```bash
xcrun simctl addmedia booted ~/Desktop/sample.mp4
```

This is useful for testing:

- avatar upload
- document/photo upload
- image picker
- video picker
- camera-roll permission behavior
- compression
- media metadata handling

Combine it with permission testing:

```bash
xcrun simctl privacy booted grant photos com.example.myapp
xcrun simctl addmedia booted ~/Desktop/test-media/*.jpg
```

---

## Send a simulated push notification

Simulator can simulate remote push notifications for development and testing.

Create a file called `payload.apns`:

```json
{
  "aps": {
    "alert": {
      "title": "QA Push",
      "body": "Hello from Simulator"
    },
    "sound": "default"
  }
}
```

Send it:

```bash
xcrun simctl push booted com.example.myapp payload.apns
```

You can also target a specific simulator:

```bash
xcrun simctl push "$UDID" com.example.myapp payload.apns
```

This is good for testing:

- push notification display
- notification tap behavior
- deep links from notifications
- foreground/background behavior
- notification permissions
- basic payload handling

But be careful: this is simulated. You are not testing the full APNs production pipeline.

Real push notification testing still needs real devices and real APNs conditions.

---

## Add a root certificate to Simulator

For HTTPS debugging with Charles, Proxyman, Burp Suite, or mitmproxy, you may need to trust a custom root certificate inside Simulator.

Add a root certificate:

```bash
xcrun simctl keychain booted add-root-cert /path/to/certificate.pem
```

Example:

```bash
xcrun simctl keychain booted add-root-cert ~/Desktop/proxy-ca.pem
```

This can help with:

- proxy testing
- HTTPS inspection
- staging environments with internal certificates
- local development APIs

But do not confuse Simulator with real iOS device behavior. Certificate handling can differ, especially when testing production-like security behavior.

For real release testing, also verify on a physical iPhone.

---

## Get app container paths

Sometimes you need to inspect app files.

Get the app container:

```bash
xcrun simctl get_app_container booted com.example.myapp app
```

Get the data container:

```bash
xcrun simctl get_app_container booted com.example.myapp data
```

Open the data container in Finder:

```bash
open "$(xcrun simctl get_app_container booted com.example.myapp data)"
```

This is useful for checking:

- cached files
- downloaded assets
- local databases
- logs
- user defaults
- temporary files
- whether logout actually cleans up local data

Do not abuse this as a replacement for proper app observability. But for local debugging, it is very handy.

---

## Spawn commands inside the simulator environment

`simctl spawn` lets you run certain commands in the simulator context.

Example: stream logs:

```bash
xcrun simctl spawn booted log stream
```

Filter logs:

```bash
xcrun simctl spawn booted log stream --predicate 'process == "MyApp"'
```

This can get noisy fast, but it is useful when you need to see what the simulated device is doing.

You can also combine this with app launch:

```bash
xcrun simctl launch --console booted com.example.myapp
```

For normal QA work, `--console` is usually simpler.

---

## Run tests against a specific simulator

This article is not a full `xcodebuild` guide, but one command is worth knowing.

Run tests on a specific Simulator destination:

```bash
xcodebuild test   -scheme MyApp   -destination 'platform=iOS Simulator,name=iPhone 16'
```

Use a specific OS version:

```bash
xcodebuild test   -scheme MyApp   -destination 'platform=iOS Simulator,name=iPhone 16,OS=18.5'
```

Use a specific UDID:

```bash
xcodebuild test   -scheme MyApp   -destination "platform=iOS Simulator,id=$UDID"
```

The UDID version is the most stable for CI.

For QA automation, a common setup is:

```bash
xcrun simctl shutdown all
xcrun simctl erase "$UDID"
xcrun simctl boot "$UDID"
xcrun simctl bootstatus "$UDID" -b

xcodebuild test   -scheme MyApp   -destination "platform=iOS Simulator,id=$UDID"
```

That gives you a cleaner test start.

---

## Useful mini workflows

### Clean manual testing session

```bash
UDID="A1B2C3D4-1111-2222-3333-ABCDEF123456"

xcrun simctl shutdown "$UDID"
xcrun simctl erase "$UDID"
xcrun simctl boot "$UDID"
xcrun simctl bootstatus "$UDID" -b
open -a Simulator
```

Use this when you want to test onboarding or first-run behavior.

---

### Install and launch a fresh build

```bash
APP="/path/to/MyApp.app"
BUNDLE_ID="com.example.myapp"

xcrun simctl install booted "$APP"
xcrun simctl launch --console booted "$BUNDLE_ID"
```

Use this when a developer gives you a simulator build.

---

### Test denied permissions

```bash
BUNDLE_ID="com.example.myapp"

xcrun simctl privacy booted reset all "$BUNDLE_ID"
xcrun simctl privacy booted revoke camera "$BUNDLE_ID"
xcrun simctl privacy booted revoke photos "$BUNDLE_ID"
xcrun simctl launch --terminate-running-process booted "$BUNDLE_ID"
```

Now check how the app behaves without expected permissions.

---

### Prepare clean screenshots

```bash
xcrun simctl status_bar booted override   --time "9:41"   --batteryState charged   --batteryLevel 100   --wifiBars 3

xcrun simctl io booted screenshot screenshot.png

xcrun simctl status_bar booted clear
```

Useful for blog posts, QA docs, release notes, and App Store-style screenshots.

---

### Test location-specific behavior

```bash
BUNDLE_ID="com.example.myapp"

xcrun simctl privacy booted grant location "$BUNDLE_ID"
xcrun simctl location booted set 19.4326,-99.1332
xcrun simctl launch --terminate-running-process booted "$BUNDLE_ID"
```

Now the app should see Mexico City as the simulated location.

---

### Test deep link behavior

```bash
xcrun simctl openurl booted "myapp://product/123"
xcrun simctl openurl booted "https://example.com/product/123"
```

You should test both custom scheme links and universal links if the app supports both.

---

## Where Simulator is weak

Simulator is useful, but it is not magic.

Do not fully trust Simulator for:

- real device performance
- battery drain
- thermal behavior
- memory pressure
- camera quality
- Bluetooth
- NFC
- Face ID / Touch ID edge cases
- cellular network behavior
- real APNs delivery
- App Store install/update flow
- device-only entitlements
- hardware security behavior
- real crash patterns from production devices

Simulator is best used as a fast feedback tool.

Real devices are still required before release.

A good mobile QA strategy is not “Simulator or real device.”

It is:

```text
Simulator for speed.
Real devices for confidence.
```

---

## Basic command cheat sheet

```bash
# check selected Xcode
xcode-select -p

# set selected Xcode
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer

# show simctl help
xcrun simctl help

# list everything
xcrun simctl list

# list devices
xcrun simctl list devices

# list available devices
xcrun simctl list devices available

# list device types
xcrun simctl list devicetypes

# list runtimes
xcrun simctl list runtimes

# create simulator
xcrun simctl create "QA iPhone" "<DeviceTypeID>" "<RuntimeID>"

# boot simulator
xcrun simctl boot "QA iPhone"

# wait until booted
xcrun simctl bootstatus "QA iPhone" -b

# open Simulator UI
open -a Simulator

# shutdown simulator
xcrun simctl shutdown "QA iPhone"

# shutdown all
xcrun simctl shutdown all

# erase simulator
xcrun simctl shutdown "QA iPhone"
xcrun simctl erase "QA iPhone"

# erase all
xcrun simctl shutdown all
xcrun simctl erase all

# delete simulator
xcrun simctl delete "QA iPhone"

# delete unavailable old simulators
xcrun simctl delete unavailable

# install app
xcrun simctl install booted /path/to/MyApp.app

# uninstall app
xcrun simctl uninstall booted com.example.myapp

# launch app
xcrun simctl launch booted com.example.myapp

# launch with console
xcrun simctl launch --console booted com.example.myapp

# launch with env variable
xcrun simctl launch --env QA_MODE=1 booted com.example.myapp

# launch with app arguments
xcrun simctl launch booted com.example.myapp --uitest-mode --mock-api

# open URL
xcrun simctl openurl booted "https://example.com"

# open deep link
xcrun simctl openurl booted "myapp://settings"

# screenshot
xcrun simctl io booted screenshot screenshot.png

# record video
xcrun simctl io booted recordVideo video.mp4

# grant permission
xcrun simctl privacy booted grant camera com.example.myapp

# revoke permission
xcrun simctl privacy booted revoke camera com.example.myapp

# reset permissions
xcrun simctl privacy booted reset all com.example.myapp

# set location
xcrun simctl location booted set 37.7749,-122.4194

# clear location
xcrun simctl location booted clear

# add media
xcrun simctl addmedia booted ~/Desktop/photo.jpg

# send push
xcrun simctl push booted com.example.myapp payload.apns

# add root certificate
xcrun simctl keychain booted add-root-cert /path/to/certificate.pem

# get app data container
xcrun simctl get_app_container booted com.example.myapp data

# stream logs
xcrun simctl spawn booted log stream
```

---

## Final thought

The iOS Simulator is not as “open” as the Android Emulator. You do not get the same low-level control. You do not boot arbitrary system images. You do not install App Store apps. You do not test everything a real iPhone can do.

But for daily QA work, it is still powerful.

If you learn `xcrun simctl`, Simulator stops being just a window launched by Xcode. It becomes a controllable test environment.

You can create devices.  
You can reset state.  
You can install builds.  
You can launch apps with arguments.  
You can fake location.  
You can manage permissions.  
You can generate screenshots and videos.  
You can prepare repeatable testing sessions.

That is exactly what a tester needs.

Not magic.  
Not a real device.  
But a very useful workhorse.
