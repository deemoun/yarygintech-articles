---
title: "VulnBankLab: Android APK Security Walkthrough for QA, Pentest and AppSec Training"
description: "A practical and personal look at teaching software testing, why one-size-fits-all QA courses often fail, and why mentorship works better when the student is actually ready."
pubDate: 2026-06-01
tags: ["Android Security", "Frida", "Objection", "Apktool", "App Analysis", "Jadx", "Appsec"]
draft: false
---

There is a very good way to learn Android security: take a small intentionally vulnerable application and stop pretending that security is magic.

Look at the APK. Look at the manifest. Look at the code. Look at the logs. Trigger the app through `adb`. Check what is stored locally. Then connect every finding to a real-world mistake developers can make.

That is the point of this article.

Today we are looking at **VulnBankLab**, an intentionally vulnerable Android training app built with Jetpack Compose.

**Project link:**

- [VulnerableBankApp on GitHub]([https://github.com/deemoun/VulnerableBankApp](https://github.com/deemoun/Vulnerable-Bank-App-Demo))

The app imitates a small fake banking workflow: login, dashboard, transactions, transfer, settings, and a security information screen. It is not supposed to be secure. It is a lab. The whole idea is to make unsafe Android patterns visible and easy to explain.

You can use this article as a blog post, a workshop handout, or as a video script.

> Educational note: only test apps you own, apps designed for training, or apps where you have explicit permission. This walkthrough is based on a deliberately vulnerable lab project.

---

## What We Are Going To Analyze

The repository already documents the intended weak points. The app contains unsafe defaults on purpose:

| Area | What is intentionally vulnerable | Why it is useful for training |
|---|---|---|
| Manifest | Activities are exported, app is debuggable, deep link is exposed | Teaches Android attack surface review |
| Authentication | Default credentials are hardcoded | Shows how secrets can be recovered from APKs |
| Local storage | Username, password and token are stored in `SharedPreferences` | Demonstrates insecure local persistence |
| Logging | Credentials, tokens and transfer data are written to Logcat | Shows why debug logging matters |
| Deep links | Transfer screen is reachable via `vuln://transfer` | Demonstrates auth bypass through external entry points |
| Validation | Transfer form accepts weak input paths | Shows why UI validation and backend validation both matter |
| Root / emulator checks | Detection is intentionally weak | Shows why client-side checks are not a security boundary |

This is small enough to explain in one video and large enough to demonstrate a real Android testing workflow.

---

## Suggested Toolset

You do not need an enormous lab to start. One emulator or test phone, Android Studio, and a few APK analysis tools are enough.

| Tool | Link | Use in this walkthrough |
|---|---|---|
| Android Studio | [developer.android.com/studio](https://developer.android.com/studio) | Build, install, run the app, inspect Logcat |
| ADB | Included with Android SDK Platform Tools | Launch activities, trigger deep links, inspect app data |
| JADX | [github.com/skylot/jadx](https://github.com/skylot/jadx) | Decompile APK to Java-like source, inspect strings, manifest and code |
| Apktool | [ibotpeaches.github.io/Apktool](https://ibotpeaches.github.io/Apktool/) | Decode resources, manifest and Smali; rebuild APKs when needed |
| PulseAPK Core | [github.com/deemoun/PulseAPK-Core](https://github.com/deemoun/PulseAPK-Core) | GUI workflow for APK decompilation, Smali analysis, building and patching |
| PulseAPK | [github.com/deemoun/PulseAPK](https://github.com/deemoun/PulseAPK) | Windows GUI frontend for apktool, analysis and APK rebuild/sign workflow |
| MobSF | [github.com/MobSF/Mobile-Security-Framework-MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) | Automated static and dynamic mobile security assessment |
| Frida | [frida.re](https://frida.re/) | Runtime instrumentation and method hooking |
| Objection | [github.com/sensepost/objection](https://github.com/sensepost/objection) | Runtime mobile exploration powered by Frida |

For this particular application, I would start with the simplest flow:

1. Build or download the APK.
2. Open it in JADX or decompile it with Apktool / PulseAPK Core.
3. Review `AndroidManifest.xml`.
4. Search for credentials, tokens and API URLs.
5. Run the app and watch Logcat.
6. Trigger the exported screens with `adb`.
7. Inspect local storage after login.
8. Discuss fixes.

That is already enough for a strong educational video.

---

## Project Overview

The app package is:

```text
com.training.vulnerablebank
```

The repository describes the app as a deliberately vulnerable Android training app built with Jetpack Compose. The current screens include:

| Screen | Purpose | Security angle |
|---|---|---|
| `LoginActivity` | Login form with pre-filled credentials | Hardcoded credentials, logging, weak session assumptions |
| `DashboardActivity` | Fake balance and navigation | Session validation assumptions |
| `TransactionsActivity` | Hardcoded/sample transaction list | Local data handling discussion |
| `TransferActivity` | Transfer form and deep link target | Auth bypass, input validation, logging |
| `SecurityInfoActivity` | Lists embedded issues | Good for explaining the lab intentionally |
| `SettingsActivity` | Settings screen | Also exported, so part of attack surface |
| `HiddenFeatureActivity` | Hidden feature screen | Good for direct activity launch demo |

The important part: this app is deliberately built as a learning target, not a production template.

---

## Finding 1: Debuggable Build

The manifest contains:

```xml
<application
    android:debuggable="true"
    ...>
```

For a lab, this is useful. For production, it is a serious mistake.

A debuggable Android application is easier to inspect and manipulate. It can make runtime analysis simpler and can also make it easier to use tools that attach to the app process.

| Check | Expected result in this app |
|---|---|
| Open `AndroidManifest.xml` | `android:debuggable="true"` is present |
| Install and run debug build | App behaves like an intentionally inspectable lab target |
| Discuss production fix | Make sure release builds are not debuggable |

### Video demo idea

Show the manifest in Android Studio or JADX. Zoom into `android:debuggable="true"`, then explain:

> This is fine for training. But when this lands in production, it becomes a gift to anyone trying to inspect the application.

### How to fix in a real app

In production:

```xml
android:debuggable="false"
```

Better: do not manually set it in the manifest at all unless you have a very clear reason. Let Gradle build types control it.

Also make sure CI/CD never signs and distributes a debug build as a release build.

---

## Finding 2: All Activities Are Exported

The manifest declares multiple activities with:

```xml
android:exported="true"
```

That includes `LoginActivity`, `DashboardActivity`, `TransactionsActivity`, `TransferActivity`, `SecurityInfoActivity`, `SettingsActivity`, and `HiddenFeatureActivity`.

Exported activities can be launched by other apps or through ADB. Some activities must be exported, for example the launcher activity or legitimate external entry points. But exporting every internal screen is usually a bad sign.

| Activity | Exported? | Why it matters |
|---|---:|---|
| `LoginActivity` | Yes | Expected for launcher, but still should be controlled |
| `DashboardActivity` | Yes | Internal screen can be opened directly |
| `TransactionsActivity` | Yes | Internal data screen can be opened directly |
| `TransferActivity` | Yes | Transfer flow becomes externally reachable |
| `SecurityInfoActivity` | Yes | Internal security info screen is externally reachable |
| `SettingsActivity` | Yes | Settings surface is externally reachable |
| `HiddenFeatureActivity` | Yes | Hidden screen can be discovered and opened |

### ADB activity launch examples

Use only in your own lab environment:

```bash
adb shell am start -n com.training.vulnerablebank/.DashboardActivity
adb shell am start -n com.training.vulnerablebank/.TransactionsActivity
adb shell am start -n com.training.vulnerablebank/.TransferActivity
adb shell am start -n com.training.vulnerablebank/.SecurityInfoActivity
adb shell am start -n com.training.vulnerablebank/.HiddenFeatureActivity
```

The key question is not only “does it open?”

The real question is:

> Does the screen enforce its own authorization check when opened directly?

If the screen assumes the user came from the correct previous screen, that is fragile. Attackers and testers do not have to follow your intended UI path.

### How to fix in a real app

| Problem | Better approach |
|---|---|
| Internal activities exported | Set `android:exported="false"` unless external access is required |
| Sensitive screens rely on navigation flow | Enforce session checks inside each sensitive screen |
| Deep links open privileged flows | Validate authentication and intent parameters before showing sensitive UI |
| Hidden screens rely on obscurity | Remove them or protect them properly |

---

## Finding 3: Deep Link Authentication Bypass

The transfer screen has an intent filter for:

```text
vuln://transfer
```

That means the transfer activity can be opened through a deep link.

Command:

```bash
adb shell am start -W -a android.intent.action.VIEW -d "vuln://transfer"
```

This is one of the best demo points in the app because it is visual and easy to understand. You can show the user flow like this:

1. Start from a clean app state.
2. Do not log in.
3. Trigger the deep link from ADB.
4. Watch the transfer screen open.
5. Explain why sensitive screens must validate the session themselves.

| Test | Expected lab behavior |
|---|---|
| Trigger `vuln://transfer` without login | Transfer screen opens |
| Check Logcat | App logs that transfer activity opened without auth checks |
| Try submitting data | Transfer flow attempts to process input |

### Why this matters

Deep links are not just navigation shortcuts. They are external entry points.

If an app exposes deep links, each deep link must be treated like an API endpoint:

- Is the user authenticated?
- Is the user allowed to perform this action?
- Are input parameters validated?
- Can another app trigger the same flow?
- Is the destination screen safe to open directly?

### How to fix in a real app

For a real banking-style app, the transfer screen should do something like this:

```kotlin
if (!sessionManager.isAuthenticated()) {
    startActivity(Intent(this, LoginActivity::class.java))
    finish()
    return
}
```

But do not stop there. Client-side session checks are a UX and local safety layer. Real transfer authorization must also happen server-side.

---

## Finding 4: Hardcoded Credentials and API Secrets

The app contains hardcoded values:

```kotlin
const val HARDCODED_USERNAME = "admin"
const val HARDCODED_PASSWORD = "password123"
const val DEFAULT_AUTH_TOKEN = "dev-session-token-abc123"
const val HARDCODED_API_URL = "https://api.vulnbanklab.training/internal"
const val HARDCODED_API_KEY = "sk_test_vulnbanklab_unsafe_key"
```

Again, this is intentional for the lab. It gives us an easy static analysis target.

| Value | Type | Risk in a real app |
|---|---|---|
| `admin` | Username | Predictable default account |
| `password123` | Password | Static credential recoverable from APK |
| `dev-session-token-abc123` | Token | Fake session secret stored in client code |
| `https://api.vulnbanklab.training/internal` | API URL | Environment and internal endpoint disclosure |
| `sk_test_vulnbanklab_unsafe_key` | API key | Static API key recoverable from APK |

### How to discover this

With JADX:

1. Open the APK.
2. Search for `password123`.
3. Search for `sk_test`.
4. Search for `api.` or `token`.
5. Open the class where the values are defined.

With command-line tools, you can also use strings-style checks after unpacking or decompiling:

```bash
grep -R "password123\|sk_test\|auth_token\|api" ./decompiled-app
```

### Why this matters

Anything shipped inside an APK should be treated as recoverable.

Obfuscation can slow someone down, but it does not turn a client-side secret into a safe secret. If the app needs the value to run, a motivated tester can usually recover it.

### How to fix in a real app

| Bad pattern | Better pattern |
|---|---|
| Static admin credentials in the app | Use real authentication with server-side verification |
| API keys embedded in the APK | Use backend-mediated access or scoped public keys only |
| Long-lived token shipped with app | Issue short-lived tokens after authentication |
| Trusting client-held secrets | Move trust decisions to the server |

---

## Finding 5: Credentials and Tokens Stored in Plaintext SharedPreferences

The app saves the username, password and token into `SharedPreferences`:

```kotlin
preferences.edit()
    .putString(KEY_USERNAME, user.username)
    .putString(KEY_PASSWORD, user.password)
    .putString(KEY_AUTH_TOKEN, user.authToken)
    .apply()
```

The preferences file name is:

```kotlin
private const val PREFS_NAME = "vuln_bank_prefs"
```

And the keys are:

```kotlin
private const val KEY_USERNAME = "username"
private const val KEY_PASSWORD = "password"
private const val KEY_AUTH_TOKEN = "auth_token"
```

This is a classic Android security teaching point.

### Lab workflow

1. Install the app.
2. Log in.
3. Inspect the app data directory.

If the app is debuggable, you can often use `run-as`:

```bash
adb shell run-as com.training.vulnerablebank
cd shared_prefs
ls
cat vuln_bank_prefs.xml
```

Expected finding:

| Stored item | Expected weakness |
|---|---|
| Username | Stored as plaintext |
| Password | Stored as plaintext |
| Auth token | Stored as plaintext |
| Balances / transactions | Stored locally as JSON-like data |

### Why this matters

Local storage is not automatically safe. On rooted devices, compromised devices, debug builds, backups, and misconfigured environments, local app data can become accessible.

Also, storing passwords locally is usually a design smell. Most apps should store a session token or refresh token, not the user's raw password. Even then, sensitive tokens should be protected properly.

### How to fix in a real app

| Problem | Better approach |
|---|---|
| Password stored locally | Do not store raw passwords |
| Token stored in plaintext | Use encrypted storage where appropriate |
| Sensitive data included in backups | Review backup rules and data extraction rules |
| App trusts local account balance | Treat local values as display/cache only; verify server-side |

For Android, look at Jetpack Security and `EncryptedSharedPreferences` where appropriate. But remember: encrypted local storage is not a replacement for server-side authorization.

---

## Finding 6: Sensitive Data Written To Logcat

The app writes sensitive values to Logcat.

Login flow:

```kotlin
Log.d("VulnBankLab", "Logging in with username=$username password=$password")
```

Preferences manager:

```kotlin
Log.d(TAG, "Saving plaintext credentials username=${user.username} password=${user.password}")
Log.d(TAG, "Saving auth token=${user.authToken}")
```

Transfer flow:

```kotlin
Log.d("VulnBankLab", "Transfer activity opened without authentication checks")
Log.d("VulnBankLab", "Transfer requested to=$recipient amount=$amount")
```

### Logcat demo

```bash
adb logcat | grep -i "VulnBankLab\|PreferencesManager"
```

Then perform:

1. Login.
2. Transfer screen open.
3. Transfer submission.

Expected visible evidence:

| Action | Log output risk |
|---|---|
| Login | Username and password appear in logs |
| Save user | Credentials and auth token appear in logs |
| Open transfer | Auth bypass hint appears in logs |
| Submit transfer | Recipient and amount appear in logs |

### Why this matters

Logs are useful during development. But logs should not contain credentials, tokens, session IDs, personal data, payment data, or anything that would be dangerous if exposed.

Even when modern Android restricts log access between apps, Logcat leakage is still a real problem in debug builds, internal builds, rooted environments, shared test devices, crash reports, and careless diagnostics.

### How to fix in a real app

| Bad logging | Better logging |
|---|---|
| `password=$password` | Never log passwords |
| `token=$token` | Never log tokens |
| Full transfer details | Log event IDs or sanitized metadata only |
| Debug logs in release | Strip or disable debug logs in production |

A safer log might look like:

```kotlin
Log.d("BankApp", "Login attempt submitted")
```

Not perfect, but much better than dumping credentials.

---

## Finding 7: Weak Root and Emulator Detection

The app contains a weak root/emulator check. It checks a list of common `su` paths and some emulator indicators.

Example paths:

```kotlin
"/system/bin/su"
"/system/xbin/su"
"/sbin/su"
"/data/adb/magisk"
```

Example emulator indicators:

```kotlin
"generic"
"sdk_gphone"
"emulator"
"goldfish"
"ranchu"
"genymotion"
"simulator"
```

The code itself says this is intentionally weak for demo purposes.

| Check type | Weakness |
|---|---|
| File path checks | Can miss modern hiding techniques or alternative paths |
| Build string checks | Can be spoofed or changed |
| Client-side result | Can be patched or hooked |
| UI warning only | Does not enforce real security controls |

### Defensive lesson

Root detection can be part of risk scoring, anti-tampering, fraud detection, or policy enforcement. But sensitive operations must still be protected server-side.

For example:

| Risk | Better control |
|---|---|
| Device is rooted | Add risk score, require stronger verification, limit risky actions |
| App is instrumented | Detect suspicious runtime conditions, but do not rely only on local checks |
| Transfer is sensitive | Validate authorization and transaction rules on the backend |

---

## Finding 8: Missing or Weak Input Validation In Transfer Flow

The README says the transfer flow is intentionally missing validation for arbitrary recipient and amount values. In the current implementation, the storage layer rejects `amount <= 0`, but the UI still accepts weak input and the activity itself is externally reachable.

The transfer screen accepts free-form text:

```kotlin
var recipient by rememberSaveable { mutableStateOf("") }
var amount by rememberSaveable { mutableStateOf("") }
```

And submits raw strings:

```kotlin
onClick = { onSubmit(recipient, amount) }
```

The submit function parses the amount:

```kotlin
val parsedAmount = amount.toDoubleOrNull()
```

This gives you a good teaching point: validation may exist in one layer, but the flow can still be poorly designed.

### Inputs to try

| Input | What to observe |
|---|---|
| Empty recipient | Should be rejected cleanly |
| Unknown recipient | Should not leak too much detail |
| Non-numeric amount | Should show proper validation error |
| Negative amount | Should be rejected |
| Very large amount | Should be rejected if balance is insufficient |
| Decimal edge cases | Should be handled consistently |

### Better validation model

For a real app, validation should happen in layers:

| Layer | Responsibility |
|---|---|
| UI | Friendly feedback and basic formatting |
| ViewModel/domain layer | Business rule validation |
| Backend | Final authorization and transaction validation |
| Audit layer | Record safe, non-sensitive event metadata |

Never trust the client alone for money movement.

---

## Suggested Static Analysis Workflow

Here is a simple workflow you can show on screen.

### Option A: JADX

```bash
jadx-gui vulnerable-bank.apk
```

Then search for:

```text
password123
sk_test
SharedPreferences
Log.d
android:exported
vuln://transfer
su
Magisk
```

### Option B: Apktool

```bash
apktool d vulnerable-bank.apk -o vulnerable-bank-decoded
```

Then inspect:

```bash
cat vulnerable-bank-decoded/AndroidManifest.xml
 grep -R "password123\|sk_test\|Log.d\|SharedPreferences\|vuln" vulnerable-bank-decoded/
```

### Option C: PulseAPK Core

PulseAPK Core is useful when you want a GUI workflow around APK analysis.

Project:

- [PulseAPK Core](https://github.com/deemoun/PulseAPK-Core)

PulseAPK Core is especially useful for workshops because it reduces the number of terminal commands beginners need to remember.

---

## Suggested Dynamic Testing Workflow

Dynamic testing means we run the app and observe behavior.

| Step | Command / action | What to prove |
|---|---|---|
| Install app | `adb install app-debug.apk` | App is ready on test device |
| Watch logs | `adb logcat | grep -i "VulnBankLab\|PreferencesManager"` | Sensitive data appears in logs |
| Trigger deep link | `adb shell am start -W -a android.intent.action.VIEW -d "vuln://transfer"` | Transfer screen can be opened externally |
| Launch activity | `adb shell am start -n com.training.vulnerablebank/.DashboardActivity` | Exported internal screens can be opened directly |
| Inspect storage | `adb shell run-as com.training.vulnerablebank` | Plaintext preferences can be inspected in debug context |

This is a strong bridge for QA engineers moving toward AppSec. It feels like testing, because it is testing. The difference is the test objective: instead of asking “does it work?”, you ask “can this be abused?”

---

## Vulnerability Summary Table

| # | Finding | Evidence | Impact | Real-world fix |
|---:|---|---|---|---|
| 1 | Debuggable build | `android:debuggable="true"` | Easier runtime inspection and tampering | Ensure release builds are not debuggable |
| 2 | Exported activities | All activities use `android:exported="true"` | Internal screens can be launched externally | Export only required entry points |
| 3 | Deep link auth bypass | `vuln://transfer` opens transfer screen | Sensitive flow reachable without normal navigation | Validate session in destination activity |
| 4 | Hardcoded credentials | `admin` / `password123` | Credentials recoverable from APK | Server-side auth, no static secrets |
| 5 | Hardcoded API key/token | Static token and `sk_test...` key | Secret extraction from client | Backend-mediated secrets, scoped keys |
| 6 | Plaintext SharedPreferences | Password/token saved as strings | Local data exposure | Avoid storing passwords; encrypt sensitive tokens |
| 7 | Sensitive Logcat output | Credentials and token logged | Data leakage in debug/log pipelines | Sanitize logs, strip debug logs in release |
| 8 | Weak root/emulator checks | Path and build-string checks only | Easy bypass or false confidence | Treat as risk signal, not security boundary |
| 9 | Weak transfer validation | Free-form transfer input and external entry | Bad transaction handling patterns | Validate in UI, domain layer and backend |

---

## What This Teaches QA Engineers

This lab is especially useful for QA engineers because the workflow is not alien.

You already know how to:

- read requirements,
- inspect behavior,
- test edge cases,
- write bug reports,
- use logs,
- think in flows,
- question assumptions.

Mobile security testing adds a different angle:

| Normal QA question | Security testing version |
|---|---|
| Does login work? | Can login be bypassed? |
| Does transfer work? | Can transfer be opened directly? |
| Does the app save state? | Does it save sensitive data unsafely? |
| Are errors logged? | Are secrets logged? |
| Does the screen validate input? | Can malformed input reach sensitive logic? |
| Does the app detect root? | Does it rely too much on root detection? |

That is why QA people can move into AppSec faster than they think. The mindset is already close. You just need different targets and different tools.

---

## Safe Bug Report Examples

If you want to turn this into training tasks, here are example bug report titles.

| Bug title | Severity for lab | Notes |
|---|---:|---|
| Internal activities are exported and can be launched directly | High | Especially dashboard, transfer and hidden feature screens |
| Transfer screen can be opened through deep link without authentication | High | Clear auth bypass demo |
| Application is built with `android:debuggable=true` | Medium / High | Depends on release context |
| Credentials are hardcoded in client code | High | Easy static extraction |
| API key and auth token are hardcoded in client code | High | Static secret leakage |
| Password and token are stored in plaintext SharedPreferences | High | Local sensitive data exposure |
| Credentials and auth token are printed to Logcat | High | Sensitive logging issue |
| Root detection is weak and implemented only client-side | Medium | Good anti-tampering discussion |
| Transfer form lacks strong validation and authorization model | High | Important for financial flows |

---

## Final Thoughts

VulnBankLab is not trying to be subtle. That is a good thing.

For teaching Android security, subtle examples are often worse. Beginners need to see the pattern clearly first:

- The manifest exposes too much.
- The app logs too much.
- The app stores too much.
- The app trusts the client too much.
- The app assumes navigation equals authorization.

Once those ideas click, you can move to harder targets, obfuscation, Frida hooks, SSL pinning, native libraries, anti-tampering, and real-world mobile testing methodology.

But the basics matter.

A surprising number of serious mobile findings still start with boring questions:

- What is exported?
- What is logged?
- What is stored?
- What is hardcoded?
- What can be opened directly?
- What does the app trust that it should not trust?

That is why a small vulnerable banking app is a good training target.

Start there. Build the habit. Then go deeper.

---

## Links

- VulnerableBankApp: [[https://github.com/deemoun/VulnerableBankApp](https://github.com/deemoun/Vulnerable-Bank-App-Demo)]([https://github.com/deemoun/VulnerableBankApp](https://github.com/deemoun/Vulnerable-Bank-App-Demo))
- PulseAPK Core: [https://github.com/deemoun/PulseAPK-Core](https://github.com/deemoun/PulseAPK-Core)
- Apktool: [https://ibotpeaches.github.io/Apktool/](https://ibotpeaches.github.io/Apktool/)
- JADX: [https://github.com/skylot/jadx](https://github.com/skylot/jadx)
- MobSF: [https://github.com/MobSF/Mobile-Security-Framework-MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF)
- Frida: [https://frida.re/](https://frida.re/)
- Objection: [https://github.com/sensepost/objection](https://github.com/sensepost/objection)
- Android Studio: [https://developer.android.com/studio](https://developer.android.com/studio)
