---
title: "Installing System-Level Certificates on Android"
description: "A short practical guide for installing a custom CA certificate into the Android system trust store for traffic interception and security testing."
pubDate: 2026-05-25
tags: ["Android", "Security", "Certificates", "MITM", "Charles Proxy", "Burp Suite", "Frida", "ADB"]
draft: false
---

# Installing System-Level Certificates on Android

This guide shows how to install a custom CA certificate as a **system-level trusted certificate** on Android.

This is useful when you want to inspect HTTPS traffic from Android apps with tools like Charles Proxy, Burp Suite, mitmproxy, or similar proxy tools.

Use this only on your own devices, lab emulators, test apps, or systems where you have permission.

---

## Why System Certificates Matter

Android has two certificate stores:

- **User certificates**
- **System certificates**

Installing a certificate through Android Settings usually puts it into the **user store**.

That works for browsers and some apps, but many Android apps ignore user-installed certificates unless the app explicitly allows them in its `network_security_config`.

For deeper mobile testing, the proxy CA certificate often needs to be placed into the **system certificate store**.

---

## What You Need

You need:

- A rooted Android emulator or rooted Android device
- ADB installed on your computer
- Your proxy CA certificate exported in `.cer`, `.crt`, or `.pem` format
- Basic terminal access

This guide uses a certificate named:

```bash
proxy-ca.pem
```

Rename your certificate if needed.

---

## Step 1 — Convert the Certificate to PEM

If your certificate is already PEM, skip this step.

For DER / `.cer` format:

```bash
openssl x509 -inform DER -in proxy-ca.cer -out proxy-ca.pem
```

For CRT format, this often works:

```bash
openssl x509 -in proxy-ca.crt -out proxy-ca.pem
```

Check the certificate:

```bash
openssl x509 -in proxy-ca.pem -text -noout
```

If it prints certificate details, the file is fine.

---

## Step 2 — Get the Android Certificate Hash

Android system certificates need a special filename based on the certificate subject hash.

Run:

```bash
openssl x509 -inform PEM -subject_hash_old -in proxy-ca.pem | head -1
```

Example output:

```bash
9a5ba575
```

Now rename the certificate using that hash:

```bash
cp proxy-ca.pem 9a5ba575.0
```

Replace `9a5ba575` with your actual hash.

---

## Step 3 — Start the Emulator with Writable System

For Android Emulator, start it with writable system support:

```bash
emulator -avd YOUR_AVD_NAME -writable-system
```

Then restart ADB as root:

```bash
adb root
adb remount
```

Check that the device is connected:

```bash
adb devices
```

---

## Step 4 — Push the Certificate to the Device

Push the certificate to a temporary location first:

```bash
adb push 9a5ba575.0 /sdcard/
```

Then open a root shell:

```bash
adb shell
su
```

Copy the certificate into the system certificate store:

```bash
cp /sdcard/9a5ba575.0 /system/etc/security/cacerts/
```

Set the correct permissions:

```bash
chmod 644 /system/etc/security/cacerts/9a5ba575.0
chown root:root /system/etc/security/cacerts/9a5ba575.0
```

Exit the shell:

```bash
exit
exit
```

Reboot the device:

```bash
adb reboot
```

---

## Step 5 — Verify the Certificate

After reboot, open:

```text
Settings → Security → Encryption & credentials → Trusted credentials → System
```

Your certificate should appear in the system list.

You can also check from ADB:

```bash
adb shell ls -l /system/etc/security/cacerts/ | grep 9a5ba575
```

Again, replace `9a5ba575` with your own hash.

---

## Step 6 — Configure Your Proxy

Set the Android device or emulator to use your proxy.

For emulator traffic, you can usually configure proxy settings in Android Wi-Fi settings:

```text
Wi-Fi → Current network → Edit → Proxy → Manual
```

Use:

```text
Proxy host: your computer IP
Proxy port: 8888 for Charles, 8080 for Burp, or your custom port
```

For an emulator, `10.0.2.2` usually points to the host machine.

Example:

```text
Proxy host: 10.0.2.2
Proxy port: 8888
```

Now open the app and check if HTTPS traffic appears in your proxy tool.

---

## Android 14 and Android 15 Note

On Android 14 and newer, certificate handling changed.

Many devices no longer rely only on:

```bash
/system/etc/security/cacerts/
```

Modern Android versions may use the Conscrypt APEX certificate store:

```bash
/apex/com.android.conscrypt/cacerts/
```

This path is not as simple to modify as the old system certificate directory.

For Android 14 and 15, the easier practical options are:

- Use an older Android emulator image for traffic labs, for example Android 11, 12, or 13
- Use a rooted emulator image where system certificate injection still works
- Use a Magisk or KernelSU module that moves user certificates into the system trust store
- Use Frida-based SSL pinning bypass when the problem is app-level pinning, not certificate trust

For YouTube demos and training labs, Android 11–13 is usually the smoothest choice.

---

## Common Problems

### Certificate appears under User, not System

That means it was installed through Android Settings.

For many apps, this is not enough.

You need to copy it into the system certificate store or use a root/Magisk-based method.

---

### `adb remount` fails

Possible reasons:

- Emulator was not started with `-writable-system`
- Device is not rooted
- Bootloader is locked
- You are using a production phone image
- Android version blocks direct system modification

For labs, use a rooted Android emulator image.

---

### HTTPS still does not decrypt

Possible reasons:

- The app uses SSL pinning
- Proxy is not configured correctly
- Certificate was installed into the user store only
- Certificate filename is wrong
- Permissions are wrong
- The app uses native networking or custom TLS logic

Check the basics first:

```bash
adb shell settings get global http_proxy
adb shell ls -l /system/etc/security/cacerts/
```

Then test with a browser before testing a real app.

---

### Browser works, but the app does not

That usually means the app has stricter network security rules or SSL pinning.

In that case, system certificates alone may not be enough.

You may need Frida, Objection, or a custom bypass script.

---

## Quick Command Summary

Replace `proxy-ca.pem` and `9a5ba575` with your real filenames.

```bash
openssl x509 -inform PEM -subject_hash_old -in proxy-ca.pem | head -1
cp proxy-ca.pem 9a5ba575.0

adb root
adb remount
adb push 9a5ba575.0 /sdcard/

adb shell
su
cp /sdcard/9a5ba575.0 /system/etc/security/cacerts/
chmod 644 /system/etc/security/cacerts/9a5ba575.0
chown root:root /system/etc/security/cacerts/9a5ba575.0
exit
exit

adb reboot
```

---

## Final Check

Before recording or testing a real app, confirm these three things:

1. The certificate is visible under **System trusted credentials**
2. Android proxy settings point to your proxy tool
3. A browser request appears decrypted in Charles, Burp, or mitmproxy

If the browser works but the app does not, the issue is probably inside the app: certificate pinning, custom trust manager, native networking, or a strict network security config.
