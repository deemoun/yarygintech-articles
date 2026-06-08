---
title: PulseAPK-Core Project
description: Professional-grade APK reverse engineering toolkit
link: https://github.com/deemoun/PulseAPK-Core
applicationCategory: DeveloperApplication
operatingSystem: Windows, macOS, Linux
sameAs: https://github.com/deemoun/PulseAPK-Core
---
PulseAPK-Core is a professional-grade, cross-platform toolkit for Android APK reverse engineering and security analysis.  
Built with **.NET 8 + Avalonia**, it provides an end-to-end workflow for:

- Decompiling APKs
- Static Smali security analysis
- Patching (including Frida gadget workflows)
- Rebuilding and signing APKs

---
{% youtube https://www.youtube.com/watch?v=iu-LOHj5QVM "PulseAPK Application for APK Decompilation" %}

## 🔧 Main Workflow Areas

### 1) Decompile
Decode an input APK into a working project directory.

**Typical flow:**  
Select APK → Set output folder → Run decompile

---

### 2) Build
Rebuild a decompiled project back into an APK.

**Typical flow:**  
Select project folder → Choose output path/name → Build (optionally sign)

---

### 3) Patch APK
Apply patching operations to APK/project sources.

**Typical flow:**  
Load APK/project → Select patch options → Run patch → Verify result

---

### 4) Analyzer
Run static analysis on decompiled Smali/resources to identify security indicators.

**Typical flow:**  
Decompile first → Run analyzer → Review findings → Export report

---

## ⭐ Key Features

- **Static Security Analysis**
  - Root detection checks
  - Emulator detection checks
  - Hardcoded secrets / credentials detection
  - Insecure SQL/HTTP usage indicators

- **Dynamic Rule Engine**
  - Rule-driven analysis via `smali_analysis_rules.json`
  - Customizable detection patterns without major workflow changes

- **Unified APK Lifecycle**
  - Decompile → Analyze → Patch → Rebuild → Sign in one toolchain

- **Configurable Environment**
  - Java / Apktool path management
  - Workspace and analysis configuration options

- **Robust UX**
  - Modern single-window workflow
  - Console-driven feedback during operations

---

## 📁 Important Project Files

- `README.MD` — project overview and usage
- `smali_analysis_rules.json` — analysis rule definitions
- `APK_ANALYSIS_RULES.md` — analysis rules documentation
- `TESTING.md` — testing guidance
- `PulseAPK.sln` — solution entrypoint
- `tests/unit/PulseAPK.Tests` — unit tests

---

## 🎯 Project Objectives

- Improve reliability of decompile/build/sign pipelines
- Expand and refine static analysis rule coverage
- Improve patch automation safety and validation
- Keep cross-platform behavior consistent and predictable
- Maintain strong developer/test quality standards
