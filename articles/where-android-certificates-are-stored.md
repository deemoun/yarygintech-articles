---
title: "Where Android Certificates Are Stored"
description: "A practical Android security guide explaining system CAs, user-installed certificates, Network Security Config, TLS trust, certificate pinning, and what this means for mobile app testing."
pubDate: 2026-06-03
tags: ["Android", "Cybersecurity", "Mobile Security", "Certificates", "TLS", "HTTPS", "Burp Suite", "Charles Proxy", "Network Security Config", "AppSec"]
draft: false
---

When you test Android applications, sooner or later you hit the same wall: HTTPS traffic does not show up correctly in Burp Suite, Charles Proxy, mitmproxy, or another interception tool. The proxy is configured. The device is connected. The certificate is installed. But the app still refuses to connect.

This usually happens because Android certificate trust is not one simple place. Android has system certificate authorities, user-installed certificates, app-specific trust configuration, and sometimes certificate pinning inside the app itself.

This article explains where Android certificates are stored, why this matters, and how the theory connects to real mobile security testing.

## First, what certificates are we talking about?

In Android security testing, the word "certificate" can mean several different things.

The most common one is a CA certificate, or Certificate Authority certificate. This is the kind of certificate you install when you want Android or an app to trust your proxy. Burp Suite, Charles Proxy, and similar tools generate a local CA certificate. When your device trusts that CA, the proxy can generate fake certificates for websites and apps during testing.

There are also server certificates. These are presented by real HTTPS servers during a TLS handshake. For example, when an app connects to `api.example.com`, that server presents a certificate proving its identity.

There are also client certificates. These are less common in normal apps but may be used in enterprise or banking environments for mutual TLS, where both server and client authenticate each other.

In this article, the main focus is CA certificates because they are the most important for Android traffic interception and trust analysis.

## The basic Android trust model

Android uses a trust store to decide whether a TLS certificate chain is valid.

A simplified HTTPS connection looks like this:

1. The app connects to a server.
2. The server sends its certificate.
3. Android or the app checks whether the certificate was issued by a trusted CA.
4. If the CA is trusted and the certificate matches the hostname, the connection continues.
5. If not, the connection fails.

That sounds simple, but Android has more than one source of trust.

There are three major places to think about:

- System trusted CAs
- User-installed CAs
- App-bundled or app-configured CAs

And then there is a separate topic: certificate pinning, where the app may reject traffic even when the CA is technically trusted.

## System CA store

The system CA store contains root certificates trusted by the Android platform.

Historically, system certificates are commonly found here:

```bash
/system/etc/security/cacerts/
```

On many Android versions, this directory contains certificate files with names like this:

```text
00673b5b.0
1d58a078.0
2d9dafe4.0
```

These filenames are based on OpenSSL-style subject hashes. The name is not random. It helps Android and Conscrypt find certificates efficiently without scanning every file.

On older Android versions, guides often tell you to copy a proxy CA certificate into:

```bash
/system/etc/security/cacerts/
```

That idea is historically important, but it is not the whole story anymore.

Modern Android is more locked down. The system partition is usually read-only. Verified Boot protects system integrity. And from newer Android versions, especially Android 14 and later, certificate handling is also affected by Conscrypt and APEX module changes. So the old "just remount `/system` and copy the cert" advice is often outdated or incomplete.

For normal users, the system CA store is not meant to be modified. For security researchers, it is still relevant, but it depends heavily on Android version, root method, emulator/device type, and whether you are testing in an authorized lab environment.

## User CA store

When you install a CA certificate through Android settings, it normally goes into the user credential store, not the system store.

The user-facing flow is usually something like:

```text
Settings -> Security -> Encryption & credentials -> Install a certificate
```

The exact menu names vary by Android version and vendor skin.

User-installed CA certificates are stored separately from system certificates. On Android devices, user certificate files are commonly associated with locations under `/data/misc/`, for example:

```bash
/data/misc/user/0/cacerts-added/
```

There can also be removed or disabled certificate records, for example:

```bash
/data/misc/user/0/cacerts-removed/
```

The `0` in the path refers to the primary Android user. Android is a multi-user operating system, so paths can differ for secondary users or work profiles.

This matters because installing a certificate as a user CA does not automatically mean every app will trust it.

## The Android 7+ change: user CAs are not trusted by default by many apps

A major change happened with Android 7.0 Nougat.

Apps targeting Android 7.0 / API 24 and higher do not trust user-installed CAs by default. They trust the system CA store unless the developer explicitly opts into trusting user CAs.

This was an intentional security improvement. Before this change, if a user or attacker installed a CA certificate on a device, many apps would automatically trust it. That made man-in-the-middle attacks easier.

For normal users, this is good.

For testers, this is where frustration starts.

You can install the Burp or Charles certificate on the device, open Chrome, and HTTPS may work. But the target app may still reject the connection because it does not trust user CAs.

So when someone says:

> I installed the Burp certificate but the Android app still does not work.

The correct answer is often:

> The certificate is probably installed in the user store, but the app does not trust user CAs.

## Network Security Config

Android provides a clean way for app developers to define certificate trust rules without writing custom TLS code. This is called Network Security Config.

The app declares it in `AndroidManifest.xml`:

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
</application>
```

Then the configuration lives in:

```text
res/xml/network_security_config.xml
```

A very common debug configuration looks like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

This means user-installed CAs are trusted only when the app is built as debuggable.

For security testing, that is a clean approach when you control the app source code. You can intercept traffic in debug builds without weakening the release build.

Another configuration trusts user CAs for all app traffic:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

This is convenient for testing, but it is usually not something you want in a production app unless there is a strong enterprise reason.

A more careful version limits user CA trust to a specific domain:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">internal.example.com</domain>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </domain-config>
</network-security-config>
```

This is better because it reduces the trust scope.

## App-bundled certificates

An app can also bundle its own CA certificate in resources.

For example:

```text
app/src/main/res/raw/my_ca.pem
```

Then the app can reference it from Network Security Config:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <trust-anchors>
            <certificates src="@raw/my_ca" />
        </trust-anchors>
    </domain-config>
</network-security-config>
```

This means the app trusts the CA included inside the APK for that domain.

From a reverse engineering point of view, this is interesting because you can inspect the APK and look for:

```text
res/xml/network_security_config.xml
res/raw/*.cer
res/raw/*.crt
res/raw/*.pem
res/raw/*.der
```

You can also decompile the app with tools like JADX and search for:

```text
networkSecurityConfig
trust-anchors
certificates
pin-set
X509TrustManager
TrustManager
CertificatePinner
HostnameVerifier
```

That gives you a quick idea of how the app handles TLS trust.

## Certificate pinning

Certificate pinning is different from normal CA trust.

With normal CA trust, the app accepts a server certificate if it chains to a trusted CA and matches the hostname.

With pinning, the app says something stricter:

> I only trust this specific certificate or this specific public key for this domain.

So even if your Burp or Charles certificate is installed correctly, the app can still reject the connection because the proxy-generated certificate does not match the pinned certificate or public key.

Network Security Config supports pinning:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2027-01-01">
            <pin digest="SHA-256">base64PinValueHere=</pin>
            <pin digest="SHA-256">backupBase64PinValueHere=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

Apps can also implement pinning in code. For example, OkHttp has `CertificatePinner`.

When testing an app, certificate pinning is one of the main reasons HTTPS interception fails even after the proxy CA is installed.

## Where to look during Android app analysis

When analyzing an APK, start with the manifest:

```bash
AndroidManifest.xml
```

Look for:

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

Then inspect the referenced XML file:

```text
res/xml/network_security_config.xml
```

Search for these patterns:

```text
src="system"
src="user"
src="@raw/"
debug-overrides
domain-config
base-config
pin-set
cleartextTrafficPermitted
```

Then inspect raw resources:

```text
res/raw/
```

Look for certificate-like files:

```text
*.cer
*.crt
*.pem
*.der
```

Then inspect Java/Kotlin code for custom TLS behavior:

```text
X509TrustManager
TrustManagerFactory
SSLContext
HostnameVerifier
CertificatePinner
checkServerTrusted
setHostnameVerifier
```

A suspicious pattern is a custom `TrustManager` that accepts everything. That can make testing easier, but it is a real security vulnerability in production.

Another suspicious pattern is a permissive `HostnameVerifier` that always returns `true`.

## Practical testing: why browser traffic works but app traffic does not

A very common situation:

1. You configure Android Wi-Fi proxy.
2. You install the Burp or Charles certificate.
3. Chrome traffic works.
4. The target app traffic fails.

This does not mean your whole setup is broken.

It usually means one of these is true:

- The app does not trust user-installed CAs.
- The app uses certificate pinning.
- The app uses its own network stack.
- The app ignores the system proxy.
- The app uses native code for TLS.
- The app uses gRPC, WebSockets, HTTP/2, or another protocol that needs extra proxy handling.
- The app uses a VPN/proxy detection mechanism.
- The app talks to a domain not covered by your current rules.

The certificate store is only one part of the full traffic interception puzzle.

## Emulator vs physical device

For Android security labs, an emulator is often easier than a physical phone.

On an emulator, especially a test image, you can more easily control root access, proxy settings, snapshots, and system behavior.

On a physical device, things are more complicated:

- Bootloader unlocking may wipe the device.
- Verified Boot can block system modification.
- Rooting depends on device and firmware.
- Vendor Android builds can behave differently.
- Work profiles and device management can add extra certificate stores.
- Banking and DRM-heavy apps may detect tampering.

For normal app testing, a debug build plus Network Security Config is usually the cleanest route.

For black-box security testing, you need to understand the difference between user CA trust, system CA trust, and pinning.

## Android certificate storage summary

Here is the practical mental model:

| Certificate type | Typical location or source | Who controls it | Trusted by default? |
|---|---|---|---|
| System CAs | `/system/etc/security/cacerts/` and modern Conscrypt/APEX-managed locations | Android platform / OEM / system image | Yes, by most apps |
| User CAs | User credential store, commonly under `/data/misc/user/0/cacerts-added/` | Device user / profile owner | Not by default for Android 7+ target apps |
| App-bundled CAs | `res/raw/` inside the APK | App developer | Only if app config/code uses them |
| Debug CAs | Network Security Config `debug-overrides` | App developer | Only in debuggable builds |
| Pinned certs/keys | XML pin-set or app code | App developer | App-specific strict trust |

## What this means for security

Android's certificate model is designed to reduce accidental trust.

From a security point of view, this is good. A random user-installed CA should not automatically gain visibility into every app's encrypted traffic.

From a testing point of view, this means you need to know what you are actually testing:

- Are you testing your own debug build?
- Are you testing a third-party app with authorization?
- Are you analyzing whether an app trusts user CAs?
- Are you checking whether pinning is implemented?
- Are you checking for unsafe custom TLS code?
- Are you trying to intercept browser traffic or app traffic?

These are different tasks.

A beginner mistake is thinking:

> I installed the certificate, so HTTPS interception should work everywhere.

A better model is:

> I installed a user CA. Now I need to know whether this specific app trusts user CAs, uses system CAs only, bundles its own CAs, or pins certificates.

## Good security testing checklist

When HTTPS interception fails, use this checklist:

1. Confirm the device proxy is set correctly.
2. Confirm the proxy CA is installed.
3. Confirm browser HTTPS traffic works.
4. Check the Android version.
5. Check the app target SDK if possible.
6. Decompile the APK and inspect `AndroidManifest.xml`.
7. Look for `networkSecurityConfig`.
8. Inspect `res/xml/network_security_config.xml`.
9. Search for bundled certificates in `res/raw/`.
10. Search code for `CertificatePinner`, `X509TrustManager`, and `HostnameVerifier`.
11. Check whether the app ignores system proxy settings.
12. Check whether the app uses native libraries for network traffic.
13. Document what failed and why.

This is how you move from random guessing to actual mobile security analysis.

## Common commands and paths

Useful paths to remember:

```bash
/system/etc/security/cacerts/
```

Common historical system CA directory.

```bash
/data/misc/user/0/cacerts-added/
```

Common user-installed CA storage path for the primary user.

```bash
/data/misc/user/0/cacerts-removed/
```

Common location related to removed or disabled certificates.

```bash
/apex/com.android.conscrypt/
```

Relevant on newer Android versions because Conscrypt can be delivered as an APEX module.

Useful APK locations to inspect:

```text
AndroidManifest.xml
res/xml/network_security_config.xml
res/raw/
lib/
classes.dex
```

Useful search terms inside decompiled code:

```text
networkSecurityConfig
CertificatePinner
X509TrustManager
TrustManager
HostnameVerifier
SSLContext
checkServerTrusted
cleartextTrafficPermitted
```

## Final thoughts

Android certificates are not stored in one magical place.

The system CA store, user CA store, app resources, Network Security Config, and certificate pinning all affect whether HTTPS connections succeed or fail.

For Android cybersecurity work, this theory is not optional. It explains why Burp or Charles sometimes works instantly in the browser but fails completely in a real app.

Once you understand where certificates live and how Android decides what to trust, HTTPS interception becomes less mysterious. You stop clicking random settings and start analyzing the app's actual trust model.