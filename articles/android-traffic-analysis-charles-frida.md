---
title: "Mobile Traffic Interception Playbook"
description: "A practical workflow for intercepting, inspecting, and modifying mobile app traffic."
pubDate: 2026-05-23
tags: ["Mobile", "Traffic Interception", "Charles Proxy", "Frida", "Objection"]
draft: false
---
When we analyze mobile applications, one of the most useful skills is the ability to intercept and inspect network traffic. This allows us to understand how the app communicates with the backend, what requests it sends, what data it receives, and how the application behaves when the network or server response changes.

Traffic interception is especially useful for QA engineers, mobile testers, security researchers, and developers who need to debug API behavior, reproduce bugs, or test edge cases that are difficult to trigger naturally.

{% youtube PY8TjCS96c8 "Root Detection Bypass" %}

## VPN vs Proxy: What Is the Difference?

A VPN and a proxy may look similar from the outside because both can route traffic through another point in the network. But their goals are different.

A VPN usually works at a lower network level and routes most or all traffic through an encrypted tunnel. The main goal of a VPN is privacy, anonymity, and secure routing.

A proxy works more selectively. It usually handles traffic from specific applications or protocols. For testing, this is exactly what we need. We do not want to hide the traffic. We want to inspect it, modify it, block it, and understand what is happening inside the application.

That is why proxy tools such as Charles Proxy, mitmproxy, Fiddler, or Burp Suite are commonly used for mobile traffic analysis.

## What Is MITM?

MITM stands for “Man-in-the-Middle”. In the context of mobile testing, it means that the proxy server is placed between the mobile app and the real backend server.

The app thinks it is talking directly to the server.  
The server thinks it is talking directly to the app.  
But in reality, the proxy is sitting in the middle and observing the data flow.

This gives the tester control over the communication channel. The proxy can display requests and responses, decrypt HTTPS traffic when configured correctly, block specific requests, or even modify server responses before they reach the app.

## Basic Interception Workflow

The general workflow for mobile traffic interception looks like this:

First, choose the tool that will intercept the traffic. This can be Charles Proxy, mitmproxy, Fiddler, Burp Suite, or another proxy tool.

Second, configure the device or emulator to trust the proxy. For HTTPS traffic, this usually means installing the proxy’s SSL certificate on the device. Without this step, the app or browser may refuse to trust the connection.

Third, start the proxy and route the mobile device traffic through it. Once the traffic is flowing through the proxy, you can begin the analysis.

## Setting Up an Android Emulator with Charles Proxy

A typical setup uses an Android emulator and Charles Proxy running on the host machine.

To route emulator traffic through Charles, you can use ADB:

```bash
adb shell settings put global http_proxy 172.18.0.1:8888
```

The IP address and port may be different depending on your setup. Charles usually listens on port 8888.

To remove the proxy settings and return the emulator to normal network behavior:

```bash
adb shell settings put global http_proxy :0
```

To check the current proxy configuration:

```bash
adb shell settings get global http_proxy
```

After configuring the proxy, open the browser inside the emulator and go to:

`chls.pro/ssl`

This downloads the Charles SSL certificate. Then install it on the Android device or emulator and verify that it appears under user-trusted credentials.

## The SSL Certificate Trust Problem

Installing the SSL certificate is a critical step. Without it, HTTPS traffic cannot be decrypted properly.

The usual process is:

1. Open `chls.pro/ssl` in the emulator browser.
2. Download the Charles certificate.
3. Install the certificate as a CA certificate.
4. Confirm Android’s security warning.
5. Check that the certificate appears in trusted user credentials.

On Android, this is often visible under something like:

Settings → Security → Encryption & credentials → Trusted credentials → User

The exact path depends on the Android version and ROM.

## Enabling SSL Proxying

Even after the certificate is installed, HTTPS traffic may still appear encrypted or unreadable in Charles. A common reason is that SSL Proxying is not enabled for the target host.

In Charles, you need to enable SSL Proxying for the domains you want to inspect.

For broad testing, you can use:

Host: *  
Port: 443

This tells Charles to attempt SSL decryption for HTTPS traffic across all domains.

However, in real testing, it is often better to limit SSL Proxying to the specific API domain you are analyzing. This keeps the session cleaner and reduces unnecessary noise.

## Blocking Requests

One useful feature in Charles is request blocking.

Blocking requests helps test how the mobile application behaves when the backend is unavailable, slow, broken, or returning an error. This is useful for crash testing, resilience testing, and error-state validation.

For example, you can block a specific image URL, API endpoint, or backend request. Then you can observe whether the app crashes, shows an error message, retries the request, or handles the failure gracefully.

In Charles, this can be done through the Block List feature. You can either drop the connection or return a specific response such as 403.

## Using Breakpoints

Breakpoints are another powerful feature.

A breakpoint pauses a request or response in real time before it continues to its destination.

For example, the app sends a request to the backend. Charles pauses it. The tester can inspect or modify request headers, body parameters, or response data. After editing, the tester clicks Execute, and Charles sends the modified data forward.

This is useful when you want to simulate different backend states without changing the real server.

For example, you can test:

- changed response fields;
- missing values;
- error messages;
- different account states;
- modified balances;
- blocked permissions;
- unexpected backend behavior.

Breakpoints allow testers to create controlled scenarios that may be difficult or impossible to reproduce naturally.

## When Charles Is Not Enough: SSL Pinning

Modern mobile applications often use SSL Pinning.

SSL Pinning means the app does not simply trust the system certificate store. Instead, the expected server certificate or public key is embedded directly inside the application. The app compares the real server certificate against the pinned certificate.

If the certificate does not match, the connection fails.

This creates a problem for proxy tools like Charles. Even if the Charles certificate is installed and trusted by Android, the application may still reject it because it does not match the pinned certificate.

Common symptoms include:

SSL handshake failed  
Unknown host  
Connection failed  
Traffic not visible in Charles

This is especially common in banking apps, fintech apps, security-focused apps, and production-grade mobile applications.

## Frida and Objection

When standard proxy setup does not work because of SSL Pinning, dynamic instrumentation tools can help.

Frida is a dynamic instrumentation framework. It allows testers to inject JavaScript into a running mobile application and modify its behavior at runtime.

With Frida, you can hook methods, change return values, bypass checks, inspect runtime behavior, and disable certain protections.

Objection is a toolkit built on top of Frida. It provides ready-made commands for common mobile security testing tasks. One of its useful features is the ability to disable SSL Pinning without writing a custom Frida script.

Frida gives more flexibility and control.  
Objection gives faster workflows and convenient commands.

## Bypassing SSL Pinning in Real Time

A common Objection workflow looks like this:

```bash
objection -g com.training.vulnerablebank explore
```

Then inside Objection:

```bash
android sslpinning disable
```

The result is that Objection injects a runtime patch into the application. The app starts trusting the proxy certificate, and Charles can decrypt the HTTPS traffic.

This is not magic. It is runtime instrumentation. The app is still running, but its certificate validation behavior has been modified while it is running.

## Useful CLI Commands

Here are several useful commands for mobile traffic interception and instrumentation.

ADB root access on emulator:

```bash
adb root
```

Check whether Frida server is running:

```bash
adb shell ps -A | grep frida
```

Check installed certificates:

```bash
adb shell ls /data/misc/user/0/cacerts-added/
```

Launch a deep link from ADB:

```bash
adb shell am start -a android.intent.action.VIEW -d https://example.com
```

Launch an app from the console:

```bash
adb shell monkey -p com.training.vulnerablebank 1
```

Find a running app process with Frida:

```bash
frida-ps -Uai | grep vulnerable
```

Attach Objection to the app:

```bash
objection -g com.training.vulnerablebank explore
```

Run a Frida CodeShare script:

```bash
frida --codeshare dzonerzy/fridantiroot -U -f com.training.vulnerablebank
```

## Full Analysis Cycle

A complete mobile traffic analysis workflow usually looks like this:

First, configure the device or emulator. Set up ADB, install the proxy certificate, and route traffic through Charles Proxy.

Second, use Charles to read, inspect, and modify traffic. This gives visibility into requests, responses, headers, payloads, and backend behavior.

Third, use tools like Block List and Breakpoints to test unusual or broken states. This helps validate how the app behaves when the backend returns errors or unexpected data.

Fourth, if SSL Pinning blocks traffic interception, use Frida or Objection to bypass certificate validation at runtime.

After this, the traffic becomes visible again, and the tester regains control over the analysis process.

## Conclusion

Mobile traffic interception turns a mobile application from a black box into something much more transparent.

With a proxy, we can read and modify traffic.  
With request blocking, we can test failure scenarios.  
With breakpoints, we can simulate different backend responses.  
With Frida and Objection, we can bypass protections that prevent standard proxy-based analysis.

For QA engineers and security testers, this workflow is extremely valuable. It helps find bugs, verify backend behavior, test edge cases, and understand how the app really works under the hood.
