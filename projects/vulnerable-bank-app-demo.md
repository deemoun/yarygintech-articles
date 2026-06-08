---
title: Vulnerable Bank App Demo
description: Intentionally vulnerable Android banking app for QA, Appium, Espresso, and mobile security training
link: https://github.com/deemoun/Vulnerable-Bank-App-Demo
applicationCategory: EducationalApplication
operatingSystem: Android
sameAs: https://github.com/deemoun/Vulnerable-Bank-App-Demo
---
Vulnerable Bank App Demo is an intentionally vulnerable Android banking application built for **QA engineers, test automation students, and mobile security analysts**.

The project is designed as a realistic training playground for:

- Android UI automation
- Appium test practice
- Espresso/instrumented testing
- ADB and deep link workflows
- Mobile security demonstrations
- Reverse engineering and traffic analysis labs

The app intentionally includes vulnerable and test-friendly patterns so engineers can practice automation, analysis, and security workflows in a controlled environment.

---

## 🔧 Main Workflow Areas

### 1) Build the App

Compile the Android project and generate a debug APK for local testing or CI usage.

**Typical flow:**  
Clone repository → Run Gradle build → Install debug APK on emulator/device

---

### 2) Manual QA Testing

Use the app as a banking demo environment for exploratory testing and mobile QA practice.

**Typical flow:**  
Launch app → Log in → Navigate through banking screens → Verify behavior and edge cases

---

### 3) Appium Automation

Run Appium-style flows against the app using stable QA-friendly entry points and UI identifiers.

**Typical flow:**  
Start emulator/device → Start Appium server → Install APK → Run automation tests

---

### 4) Espresso / Instrumented Testing

Use Android instrumented tests to validate UI behavior directly on emulator or device.

**Typical flow:**  
Start emulator → Run connected Android tests → Review results

---

### 5) Security Training

Use the app for mobile security demonstrations such as insecure flows, deep links, reverse engineering, and traffic analysis.

**Typical flow:**  
Install APK → Inspect behavior → Analyze app logic → Test vulnerable patterns safely

---

## ⭐ Key Features

- **Android Banking Demo App**
  - Login flow
  - Dashboard screen
  - Transfer flow
  - Transactions screen
  - Bank-style navigation for realistic QA practice

- **Built for Test Automation**
  - Appium-friendly workflows
  - Espresso/instrumented test support
  - Local JUnit test structure
  - Stable package and activity entry points

- **Security Education Playground**
  - Intentionally vulnerable patterns
  - Useful for Android reverse engineering labs
  - Good target app for Frida, Objection, ADB, and traffic analysis demos
  - Safe environment for teaching mobile security concepts

- **CI-Friendly Project Structure**
  - Gradle wrapper included
  - GitHub Actions workflows
  - APK build automation
  - Lint and test commands for repeatable validation

- **Developer-Friendly Setup**
  - Kotlin/Java Android project
  - JDK 17 support
  - Android SDK / API 36 target environment
  - Helper scripts for local and CI builds

---

## 📁 Important Project Files

- `README.md` — project overview, setup, commands, and QA entry points
- `app/` — main Android application module
- `app/src/main/java/com/training/vulnerablebank/` — activities, app logic, and utilities
- `app/src/main/res/` — Android resources
- `app/src/test/java/com/training/vulnerablebank/` — local/unit tests
- `app/src/androidTest/java/com/training/vulnerablebank/` — instrumented/Espresso tests
- `.github/workflows/` — CI workflows for APK builds and tests
- `scripts/` — helper scripts for local and CI usage
- `build.gradle.kts` — root Gradle build configuration
- `settings.gradle.kts` — Gradle project settings

---

## ⚙️ Useful QA Entry Points

Package name:

```bash
com.training.vulnerablebank
```

Launch login screen:

```bash
adb shell am start -n com.training.vulnerablebank/.LoginActivity
```

Launch dashboard screen:

```bash
adb shell am start -n com.training.vulnerablebank/.DashboardActivity
```

Launch transfer screen:

```bash
adb shell am start -n com.training.vulnerablebank/.TransferActivity
```

Deep link example:

```bash
adb shell am start -a android.intent.action.VIEW -d 'vuln://transfer'
```

---

## 🧪 Common Commands

Build debug APK:

```bash
./gradlew clean assembleDebug
```

Run lint:

```bash
./gradlew lint
```

Run local/unit tests:

```bash
./gradlew testDebugUnitTest
```

Run instrumented tests on emulator/device:

```bash
./gradlew connectedDebugAndroidTest
```

Run Gradle managed device test:

```bash
./gradlew :app:headlessApi36DebugAndroidTest
```

Build and copy APK to artifacts:

```bash
./scripts/ci-build-apk.sh
```

---

## 🎯 Project Objectives

- Provide a realistic Android banking app for QA and automation training
- Give students a safe target for Appium, Espresso, ADB, and CI practice
- Support mobile security lessons without relying on random third-party apps
- Make Android reverse engineering and traffic analysis demos repeatable
- Keep the project practical, simple to build, and easy to use in courses
- Serve as a stable demo app for articles, videos, workshops, and classroom exercises

---

## ✅ Best Use Cases

- QA automation courses
- Appium lessons
- Android testing workshops
- Mobile security demonstrations
- Reverse engineering practice
- CI/CD examples for Android testing
- ADB and deep link training
- Vulnerable app demos for safe educational use
