---
title: "Static vs Dynamic Analysis: Understanding Mobile App Behavior for AppSec"
description: "How to analyze your mobile applications statically and dynamically"
pubDate: "2026-08-24"
tags: ["mobile security", "static analysis", "mobile development", "appsec", "security specialist", "android development"]
draft: false
---
When you start looking at mobile application security, one distinction
appears almost immediately: **static analysis** and **dynamic
analysis**.

The names make them sound more complicated than they are.

Static analysis means examining an application **without actually
running it**.

Dynamic analysis means examining **what the application does while it is
running**.

Both approaches can reveal security problems. Both also have blind
spots.

And if your goal is to understand the actual behavior of a mobile
application rather than simply run a scanner and export a report, you
normally need both.

In this article, we'll look at the practical workflow from an
AppSec/security specialist's point of view, primarily using Android
examples. Most of the methodology also applies to iOS, although the
tooling and platform restrictions are different.

> **Important:** Only analyze applications and systems you own or have
> permission to test.

------------------------------------------------------------------------

## The APK Is Already Evidence

One useful thing about mobile security is that the application itself
gives you a surprisingly large amount of information.

An Android APK is not some completely opaque executable.

It is essentially a package containing things such as:

-   compiled application code;
-   `AndroidManifest.xml`;
-   resources;
-   certificates;
-   native libraries;
-   configuration files;
-   assets.

Before launching the application, we can already start answering
security questions.

For example:

-   What permissions does the app request?
-   What components are exported?
-   Does it allow cleartext traffic?
-   Are there hardcoded API endpoints?
-   Are API keys or tokens embedded in resources?
-   Does it contain native libraries?
-   Does the application attempt to detect root?
-   Does it contain certificate-pinning logic?
-   Are debug features accidentally enabled?
-   What third-party SDKs are included?

This is the territory of **static analysis**.

------------------------------------------------------------------------

# Static Analysis

Static analysis is the process of examining an application without
executing it.

For Android applications, the first step is usually obtaining the APK
and unpacking or decompiling it.

A very simple toolkit might include:

``` bash
jadx
apktool
apkanalyzer
apksigner
grep
strings
```

Different tools answer different questions.

There is no single "reverse engineering button."

------------------------------------------------------------------------

## Start With the Manifest

The Android manifest is one of the highest-value files in an initial
review.

With `apktool`:

``` bash
apktool d application.apk -o application_decoded
```

Then inspect:

``` text
application_decoded/AndroidManifest.xml
```

You might find permissions such as:

``` xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

Permissions alone are not vulnerabilities.

The interesting question is whether they make sense for the
application's functionality and how the application actually uses them.

The manifest can also reveal application components:

``` xml
<activity>
<service>
<receiver>
<provider>
```

Pay particular attention to components exposed outside the application.

For example:

``` xml
android:exported="true"
```

An exported activity is not automatically vulnerable either.

But now you have something worth testing dynamically:

> Can another application or ADB invoke this component in a way the
> developer did not expect?

That is a good example of static analysis generating a hypothesis for
dynamic analysis.

------------------------------------------------------------------------

## Decompile the Code

For Android, `jadx` is usually my first choice for quickly understanding
application logic.

``` bash
jadx-gui application.apk
```

Or:

``` bash
jadx -d source application.apk
```

You can now search for interesting strings and APIs.

Useful search terms often include:

``` text
http://
https://
api
token
secret
password
Authorization
Bearer
WebView
setJavaScriptEnabled
TrustManager
HostnameVerifier
CertificatePinner
root
debug
SharedPreferences
getExternalStorage
```

Do not treat this as a magic vulnerability keyword list.

The goal is to understand the application's architecture.

For example, you may discover:

``` text
https://api.example.com/v2/
```

Then find the networking client responsible for communicating with it.

Then find an authorization interceptor.

Then discover how the access token is obtained.

That is already much more useful than a generic scanner telling you that
the application "uses network communication."

------------------------------------------------------------------------

## Look for Hardcoded Secrets --- Carefully

One of the easiest static checks is searching for interesting strings.

For example:

``` bash
grep -RniE "api[_-]?key|secret|token|password" application_decoded/
```

You can also run:

``` bash
strings application.apk | grep -i token
```

But this produces a lot of noise.

Not every API key is a secret.

Some identifiers are intentionally shipped with the client. Some keys
are restricted by package name, signing certificate, API scope,
server-side policy, or quota.

So finding:

``` text
AIza...
```

does not automatically mean:

> Critical vulnerability! Production credentials exposed!

The real question is:

**What can somebody actually do with this value?**

Security analysis is about impact, not keyword matching.

------------------------------------------------------------------------

## Native Libraries Matter Too

Not all application logic lives in Java or Kotlin.

Check:

``` text
lib/arm64-v8a/
lib/armeabi-v7a/
lib/x86_64/
```

You may find `.so` libraries containing:

-   cryptographic logic;
-   anti-tampering checks;
-   root detection;
-   proprietary protocols;
-   embedded configuration;
-   JNI implementations.

A quick first pass:

``` bash
strings libsomething.so
```

For deeper work, tools such as Ghidra can help analyze native code.

This is also where static analysis becomes significantly more difficult.

Decompiled Java is often relatively readable.

Optimized ARM64 assembly is a different experience entirely.

------------------------------------------------------------------------

## Static Analysis Has a Major Problem

Static analysis tells you what the application **can potentially do**.

It does not necessarily tell you what it **actually does**.

Imagine finding this endpoint:

``` text
https://analytics.example.com/upload
```

Interesting.

But several questions remain:

-   When is it called?
-   What triggers it?
-   What data is sent?
-   Is authentication required?
-   Does it happen before login?
-   Does it happen in the background?
-   Does the app send device identifiers?
-   Is the payload encrypted at the application layer?
-   Does behavior change depending on account state?

Reading code may answer some of these questions.

Running the application can answer them much faster.

Now we move to dynamic analysis.

------------------------------------------------------------------------

# Dynamic Analysis

Dynamic analysis means observing and manipulating the application while
it executes.

This is where the app stops being a collection of files and starts
behaving like a system.

A basic Android lab might contain:

-   an emulator or test device;
-   ADB;
-   Burp Suite, mitmproxy, or Charles;
-   Frida;
-   Objection;
-   `logcat`;
-   optionally a rooted emulator/device.

ADB alone already gives you a lot.

Check the connected device:

``` bash
adb devices
```

Find the application:

``` bash
adb shell pm list packages | grep example
```

Inspect package information:

``` bash
adb shell dumpsys package com.example.app
```

Launch an activity:

``` bash
adb shell am start -n com.example.app/.MainActivity
```

Now we can compare what we saw statically with what actually happens at
runtime.

------------------------------------------------------------------------

## Watch the Logs

One of the simplest dynamic-analysis tools is:

``` bash
adb logcat
```

You would be surprised how often applications leak useful information
into logs.

Look for:

-   URLs;
-   authentication errors;
-   stack traces;
-   internal object values;
-   tokens;
-   user identifiers;
-   API responses;
-   debug messages.

You can filter output:

``` bash
adb logcat | grep -i example
```

or:

``` bash
adb logcat | grep -iE "token|auth|http|error"
```

Again, the point is not just to grep random security words.

Perform an action in the application and observe what changes.

For example:

1.  clear the logs;
2.  open the login screen;
3.  authenticate;
4.  perform one specific action;
5.  inspect the resulting logs.

This makes the result reproducible.

That matters enormously when reporting security findings.

------------------------------------------------------------------------

## Observe Network Traffic

Network inspection is probably one of the most useful parts of dynamic
mobile analysis.

Configure the device or emulator to use an interception proxy such as
Burp Suite or mitmproxy.

Now perform normal application actions:

``` text
Launch app
    ↓
Login
    ↓
Open profile
    ↓
Change setting
    ↓
Upload file
```

Watch the corresponding requests.

You may discover:

-   undocumented API endpoints;
-   excessive information returned by the server;
-   insecure authorization;
-   sensitive information in requests;
-   weak session handling;
-   unnecessary analytics;
-   unexpected third-party services.

This is where application behavior becomes much easier to understand.

Instead of guessing how the networking layer works from decompiled code,
you can observe:

``` http
POST /api/v1/profile
Authorization: Bearer ...
Content-Type: application/json
```

and examine the actual request and response.

------------------------------------------------------------------------

## And Then Certificate Pinning Appears

Sometimes you configure your proxy and everything works.

Sometimes the app immediately refuses to connect.

One common reason is **certificate pinning**.

Static analysis may already have given you clues.

For example, you might find references to:

``` text
CertificatePinner
TrustManager
X509Certificate
checkServerTrusted
```

Dynamic instrumentation tools such as Frida are useful here because they
allow you to inspect or alter application behavior at runtime in an
authorized test environment.

This introduces an important difference between the two approaches.

Static analysis asks:

> Where is this security control implemented?

Dynamic analysis asks:

> What happens if I interact with this control while the application is
> running?

------------------------------------------------------------------------

# Runtime Instrumentation With Frida

Frida is one of the most useful tools for dynamic mobile application
analysis.

It allows you to inject JavaScript into a running process and hook
functions.

For example, suppose the application contains:

``` java
boolean isDeviceRooted()
```

Static analysis might show you the implementation.

Dynamic instrumentation can let you observe when the method is called
and, in a test environment, change its return value.

Conceptually:

``` javascript
SomeClass.isDeviceRooted.implementation = function () {
    console.log("isDeviceRooted() called");
    return false;
};
```

This can help answer questions such as:

-   Is root detection actually executed?
-   Which check causes the application to stop?
-   Does behavior change after bypassing the check?
-   Is the control client-side only?
-   What code path becomes reachable afterward?

This is much more interesting than simply declaring:

> The application has root detection.

We want to understand its security impact.

------------------------------------------------------------------------

## Objection: Faster Interactive Exploration

Objection builds on Frida and provides an interactive interface for many
common mobile-testing tasks.

For example, during an authorized Android assessment you can use it to
explore:

-   activities;
-   services;
-   application classes;
-   file storage;
-   runtime methods;
-   common platform protections.

Frida gives you flexibility.

Objection often gives you speed.

I would not treat them as competing tools.

Use whichever gets you to the answer faster.

------------------------------------------------------------------------

# Behavior Is More Than Network Traffic

A common mistake is to think dynamic analysis means only intercepting
HTTP requests.

Mobile applications interact with the operating system constantly.

You should also observe things such as:

-   files created by the application;
-   databases;
-   SharedPreferences;
-   clipboard usage;
-   screenshots;
-   notifications;
-   deep links;
-   intents;
-   background services;
-   WebViews;
-   external storage;
-   IPC between applications;
-   runtime permissions.

For example, after logging in, inspect the application's data directory
on an appropriate test device.

You might discover:

``` text
shared_prefs/
databases/
files/
cache/
```

Then ask:

**What changed after authentication?**

Maybe a token appears in SharedPreferences.

Maybe a SQLite database contains user information.

Maybe cached API responses remain after logout.

Maybe a supposedly temporary file never gets deleted.

These are behavioral findings even though no network attack is involved.

------------------------------------------------------------------------

# Build Tests Around Actions

Randomly clicking through an application while Burp and `logcat` are
running is better than nothing.

But a structured workflow is much more useful.

Treat application behavior almost like a test case.

For example:

## Scenario: User Logout

### Before logout

Check:

``` text
Authentication token
Cookies
SharedPreferences
Database
Cached files
Running services
```

### Perform

``` text
Tap Logout
```

### After logout

Check the same things again.

Then try:

-   replaying an old request;
-   reopening a protected activity;
-   using the Back button;
-   restarting the application;
-   invoking a deep link;
-   restoring the application from background.

Now you are not simply "doing security testing."

You are testing a specific security property:

> Does logout actually terminate the user's authenticated state?

This is where QA-style thinking becomes extremely useful in AppSec.

------------------------------------------------------------------------

# Static + Dynamic Is the Real Workflow

The strongest workflow is not:

``` text
Static analysis
OR
Dynamic analysis
```

It is:

``` text
Static analysis
        ↓
Create hypotheses
        ↓
Dynamic analysis
        ↓
Observe behavior
        ↓
Return to code
        ↓
Explain why behavior occurs
        ↓
Verify impact
```

Suppose static analysis reveals:

``` xml
<activity
    android:name=".AdminActivity"
    android:exported="true" />
```

That is interesting.

Now dynamically test:

``` bash
adb shell am start -n com.example.app/.AdminActivity
```

What happens?

Possibility 1:

``` text
The activity opens, but immediately verifies authentication.
```

Probably fine.

Possibility 2:

``` text
The activity opens and exposes administrative functionality.
```

Now you may have a real security issue.

The manifest gave you the lead.

Runtime behavior gave you the evidence.

------------------------------------------------------------------------

# Another Example: Deep Links

Static analysis might reveal:

``` xml
<data
    android:scheme="example"
    android:host="payment" />
```

Now you know the application accepts something like:

``` text
example://payment
```

But that alone tells you very little.

Dynamic testing can answer:

-   What parameters does the handler accept?
-   Does it require authentication?
-   Can parameters alter sensitive actions?
-   Can another application trigger it?
-   Does it load arbitrary URLs?
-   Does it expose internal application screens?

You can invoke a test deep link with ADB:

``` bash
adb shell am start \
  -a android.intent.action.VIEW \
  -d "example://payment"
```

This is the recurring pattern:

**Static analysis tells you where to look. Dynamic analysis tells you
what happens.**

------------------------------------------------------------------------

# Automated Scanners Are Useful --- But They Are Not the Analysis

Tools such as MobSF can automate a significant part of the initial
inspection.

That is useful.

You can quickly get information about:

-   permissions;
-   exported components;
-   certificates;
-   URLs;
-   trackers;
-   insecure configurations;
-   code patterns.

But scanner output should be treated as **input to an investigation**,
not the final result.

A scanner might say:

``` text
Application allows cleartext traffic.
```

That sounds bad.

But does the production application actually send sensitive data over
HTTP?

Maybe yes.

Maybe the configuration exists for a local development endpoint that is
never reachable in production.

The difference matters.

Likewise, a scanner may find a hardcoded key.

Your job is to determine whether the key has meaningful privileges.

Security findings need context.

------------------------------------------------------------------------

# What Static Analysis Is Good At

Static analysis is particularly useful for discovering:

-   application attack surface;
-   exported components;
-   permissions;
-   deep-link handlers;
-   embedded URLs;
-   hardcoded configuration;
-   suspicious API usage;
-   cryptographic implementations;
-   third-party libraries;
-   root/emulator detection;
-   certificate-pinning implementations;
-   hidden or unused functionality.

It is also excellent for answering:

> Where in the code is this behavior implemented?

------------------------------------------------------------------------

# What Dynamic Analysis Is Good At

Dynamic analysis is particularly useful for understanding:

-   actual network communication;
-   runtime authentication behavior;
-   session handling;
-   local data changes;
-   application state transitions;
-   runtime protections;
-   WebView behavior;
-   API interaction;
-   component exposure;
-   behavior under manipulated conditions.

It answers a different question:

> What actually happens when I do this?

------------------------------------------------------------------------

# Static Analysis Can Lie to You

Not literally, of course.

But decompiled code can easily lead you toward the wrong conclusion.

You may see:

``` java
if (isRooted()) {
    blockApplication();
}
```

and assume rooted devices are blocked.

Then you run the application on a rooted test device and discover the
code path is never executed in the production build.

Or perhaps it only executes for a specific feature.

Or the decompiler reconstructed the code poorly.

Or the relevant method belongs to an unused library.

Code presence is not proof of behavior.

------------------------------------------------------------------------

# Dynamic Analysis Can Lie to You Too

Runtime testing has the opposite problem.

You only observe the paths you execute.

If you never trigger a particular feature, you may never see its
behavior.

Maybe an insecure endpoint is called only:

-   after seven days;
-   when push notification data arrives;
-   for premium accounts;
-   on a specific Android version;
-   when the device enters a certain state;
-   after a remote feature flag changes.

Dynamic analysis shows reality --- but only the reality you managed to
trigger.

Static analysis can expose paths you did not know existed.

That is why the approaches complement each other so well.

------------------------------------------------------------------------

# A Practical Mobile AppSec Workflow

For a new Android application, I would roughly approach it like this.

## 1. Identify the package

Collect:

``` text
APK
Package name
Version
Version code
Signing certificate
Target SDK
Minimum SDK
```

Check the signature:

``` bash
apksigner verify --print-certs application.apk
```

## 2. Inspect the manifest

Look at:

``` text
Permissions
Exported activities
Services
Receivers
Providers
Deep links
Backup settings
Network security configuration
Debug settings
```

## 3. Decompile

Open the APK in `jadx`.

Search for:

``` text
Endpoints
Authentication logic
Storage
WebViews
Crypto
Root detection
Pinning
Interesting SDKs
```

## 4. Run the application

Use a controlled emulator or device.

Watch:

``` bash
adb logcat
```

Inspect package state and interact with exposed components.

## 5. Proxy the traffic

Observe what the application actually sends and receives.

Map UI actions to API calls.

## 6. Inspect local state

Compare files and application state before and after important actions.

## 7. Instrument interesting code

Use Frida or Objection when runtime behavior needs deeper inspection.

## 8. Verify findings

Do not report:

``` text
Potentially insecure exported activity.
```

when you can test the activity and explain exactly what an attacker
could achieve.

------------------------------------------------------------------------

# Think in Hypotheses, Not Tools

This is probably the biggest difference between simply learning security
tools and actually doing useful application security work.

Do not start with:

> What can I do with Frida today?

Start with:

> I think the application stores the session locally after logout. How
> can I prove or disprove that?

Then choose the tool.

Maybe the answer requires:

``` text
ADB + filesystem inspection
```

Another hypothesis may require:

``` text
jadx + Burp
```

Another:

``` text
apktool + ADB intents
```

Another:

``` text
Frida
```

Tools are implementation details.

The investigation is the important part.

------------------------------------------------------------------------

# Evidence Matters

A useful AppSec finding should be reproducible.

Instead of writing:

> The app may expose sensitive functionality through an exported
> activity.

Show:

``` bash
adb shell am start -n com.example.app/.InternalActivity
```

Then explain:

``` text
1. No authentication is required.
2. The activity displays account information.
3. It can be launched by another application.
4. The behavior reproduces after a clean installation.
```

Now developers have something they can understand, reproduce, and fix.

This is one area where software-testing experience translates extremely
well into security.

A good security report looks surprisingly similar to a good bug report.

You still need:

``` text
Preconditions
Steps
Actual result
Expected security behavior
Evidence
Impact
```

The difference is that you also need to think about an attacker.

------------------------------------------------------------------------

# Final Thoughts

Static and dynamic analysis are not two competing ways to inspect a
mobile application.

They answer different questions.

Static analysis tells you:

> **What exists inside the application?**

Dynamic analysis tells you:

> **What does the application actually do?**

The interesting work happens when you connect them.

Find an exported component statically.

Invoke it dynamically.

Find certificate-pinning code statically.

Observe the network behavior dynamically.

Find token-storage logic statically.

Log in, log out, restart the app, and inspect what survives dynamically.

Find a suspicious WebView configuration statically.

Then determine what content can actually reach that WebView at runtime.

That loop is what turns reverse engineering into application security
analysis.

And you do not need twenty tools to start.

For Android, a very capable initial toolkit is already:

``` text
ADB
jadx
apktool
Burp Suite or mitmproxy
Frida
Objection
```

Learn what the application is supposed to do.

Form a hypothesis.

Inspect the code.

Run the application.

Manipulate the environment.

Collect evidence.

Then explain the security impact.

That is much closer to real AppSec work than pressing **Scan** and
waiting for a red report.
