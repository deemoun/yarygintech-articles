---
title: "Package Managers for macOS That Are Still Worth Using"
description: "A practical guide to active macOS package managers: Homebrew, MacPorts, Nix, pkgx, Conda/Mamba, and mas-cli."
pubDate: 2026-06-11
tags: ["Apple", "NixOS", "Homebrew", "Pacckages", "DevTools"]
draft: false
---
macOS is Unix-ish enough that you *want* a package manager, but Apple does not ship a real general-purpose one.

Yes, there is the App Store. Yes, there are `.dmg` files. Yes, you can drag apps into `/Applications` like it is still 2009.

**But if you do any kind of development, QA work, automation, security testing, scripting, or just want a sane way to install command-line tools, you need something better.**

**The good news: macOS has several active package managers.**

**The bad news: they are not all trying to solve the same problem.**

So this is not a “which one is objectively best” article. It is more like: which tool should you use, and when?

## Quick Answer

For most people:

| Use case | Pick |
|---|---|
| Normal macOS dev machine | Homebrew |
| More isolated Unix-style package system | MacPorts |
| Reproducible/dev environment nerd stuff | Nix |
| Run tools without thinking too much | pkgx |
| Python/data science/ML environments | Conda or Mamba |
| Script Mac App Store installs | mas-cli |

If you just want one: install **Homebrew**.

If you are building reproducible dev environments or want to go deep: learn **Nix**.

If you are doing Python/data science work: use **Conda/Mamba**.

If you want App Store automation: use **mas**, usually installed through Homebrew.

---

## 1. Homebrew

Official site: https://brew.sh/  

Homebrew is the default answer for macOS package management.

It installs command-line tools, libraries, services, and GUI apps through **casks**.

Example:

```bash
brew install wget
brew install git
brew install nmap
brew install --cask firefox
brew install --cask visual-studio-code
```

Homebrew is popular because it feels simple. You search, install, update, remove. Done.

```bash
brew search ffmpeg
brew install ffmpeg
brew upgrade
brew cleanup
```

### Pros

- Biggest ecosystem on macOS.
- Very easy to use.
- Installs both CLI tools and GUI apps.
- Tons of documentation and Stack Overflow answers.
- Works well on Apple Silicon and Intel Macs.
- Great for setting up a new Mac quickly.
- Good enough for most developers.

### Cons

- It can get messy if you install everything through it and never clean up.
- Formula changes can occasionally break old workflows.
- Not as reproducible as Nix.
- It relies on community-maintained formulae and taps, so you should not blindly trust random taps.
- Some GUI apps installed as casks are basically just automated `.dmg` downloads.

### Best For

Use Homebrew if you want the practical default.

It is what I would install first on a new Mac.

### Basic Commands

| Task | Command |
|---|---|
| Install package | `brew install package` |
| Install GUI app | `brew install --cask app-name` |
| Search | `brew search name` |
| Update package metadata | `brew update` |
| Upgrade installed packages | `brew upgrade` |
| Remove package | `brew uninstall package` |
| Clean old files | `brew cleanup` |
| List installed packages | `brew list` |

---

## 2. MacPorts

Official site: https://www.macports.org/  

MacPorts is older than Homebrew and still active.

It has a more traditional Unix package manager feel. Packages are called **ports**. It usually installs into `/opt/local`, which keeps it separate from a lot of Apple and Homebrew stuff.

Example:

```bash
sudo port selfupdate
sudo port install wget
sudo port install git
```

### Pros

- Clean separation under `/opt/local`.
- Good for people who like a more classic ports-style system.
- Can be more predictable in some cases because it builds and manages its own dependency tree.
- Supports a wide range of older macOS versions compared with many modern tools.
- Good for Unix/X11/open-source software.

### Cons

- Usually slower than Homebrew, especially when building.
- Uses `sudo` more often.
- Smaller mindshare than Homebrew.
- Some guides and scripts assume Homebrew instead.
- Mixing MacPorts and Homebrew can create PATH and dependency confusion if you are careless.

### Best For

Use MacPorts if you want a more isolated Unix package tree and do not care that Homebrew is more popular.

Also good if you are using older Macs or older macOS versions and Homebrew support is not ideal.

### Basic Commands

| Task | Command |
|---|---|
| Update MacPorts | `sudo port selfupdate` |
| Install package | `sudo port install package` |
| Search | `port search name` |
| Upgrade installed packages | `sudo port upgrade outdated` |
| Remove package | `sudo port uninstall package` |
| List installed ports | `port installed` |

---

## 3. Nix

Official site: https://nixos.org/  

Nix is not just a package manager. It is almost a different philosophy.

Instead of “install a bunch of stuff into the system and hope it keeps working,” Nix tries to make packages and environments reproducible.

You can install tools, create isolated shells, pin versions, define dev environments, and even manage macOS configuration with extra projects like `nix-darwin`.

Example:

```bash
nix shell nixpkgs#git
nix shell nixpkgs#python312
```

A simple dev shell can be defined in a file and shared with a project.

### Pros

- Excellent for reproducible development environments.
- Lets you pin versions.
- Multiple versions of tools can coexist.
- Powerful for teams and CI.
- Great when you want Linux/macOS environment parity.
- Can be used declaratively.

### Cons

- Steep learning curve.
- Documentation and ecosystem can feel fragmented.
- Error messages can be rough.
- The Nix store takes disk space.
- Overkill if you only want `wget`, `git`, and a few desktop apps.
- On macOS, it can feel less native than Homebrew.

### Best For

Use Nix if you care about reproducibility.

For example:

- “I want this project to have the same tools on every machine.”
- “I want my dev environment defined in code.”
- “I want to avoid random dependency drift.”
- “I use Linux and macOS and want similar workflows.”

Do not start with Nix if you only want a casual package manager. It is powerful, but it is not casual.

### Basic Commands

| Task | Command |
|---|---|
| Run a package temporarily | `nix shell nixpkgs#package` |
| Search packages online | https://search.nixos.org/packages |
| Collect garbage | `nix-collect-garbage` |
| Show Nix version | `nix --version` |

---

## 4. pkgx

Official site: https://pkgx.dev/  

`pkgx` is a newer tool from the creator of Homebrew. It is more of a package runner/environment tool than a classic system package manager.

The idea is simple: run tools without manually installing a bunch of stuff first.

Example:

```bash
pkgx node --version
pkgx python --version
pkgx ffmpeg -version
```

It can also be used in scripts and development environments.

### Pros

- Lightweight.
- Nice for running tools on demand.
- Good for scripts.
- Avoids permanently installing everything.
- Interesting for clean dev workflows.
- Less ceremony than Nix for some use cases.

### Cons

- Much smaller ecosystem mindshare than Homebrew.
- Newer, so fewer tutorials and fewer people using it in companies.
- Not a full replacement for Homebrew if you install lots of GUI apps.
- Some users may find the model less obvious than `brew install`.

### Best For

Use pkgx if you like the idea of running tools on demand.

It is especially nice for scripts, temporary tooling, and avoiding a messy global install list.

For a normal Mac setup, I would still start with Homebrew. But pkgx is worth knowing.

### Basic Commands

| Task | Command |
|---|---|
| Install pkgx using Homebrew | `brew install pkgx` |
| Install with shell script | `curl https://pkgx.sh | sh` |
| Run a tool | `pkgx tool-name` |

---

## 5. Conda and Mamba

Conda docs: https://docs.conda.io/projects/conda/en/latest/user-guide/install/  

Conda is a package and environment manager mostly used for Python, data science, machine learning, scientific computing, and tools with annoying native dependencies.

Mamba is a faster Conda-compatible package manager. In practice, many people use Conda/Miniforge/Mambaforge style environments and then use `mamba` because dependency solving is faster.

Example:

```bash
conda create -n qa-lab python=3.12
conda activate qa-lab
conda install requests pytest
```

With mamba:

```bash
mamba create -n data-lab python=3.12 pandas numpy
mamba activate data-lab
```

### Pros

- Great for Python environments.
- Good for scientific/data packages.
- Handles native dependencies better than plain `pip` in many cases.
- Easy to create and delete isolated environments.
- Mamba is usually faster at resolving packages.

### Cons

- Not a general macOS package manager.
- Can conflict mentally with `pip`, Homebrew Python, system Python, and IDE settings.
- Environments can become huge.
- Base environment pollution is common if you are careless.
- Not the best tool for installing random macOS CLI utilities.

### Best For

Use Conda/Mamba if you work with:

- Python
- data science
- ML
- scientific packages
- projects that need isolated Python environments
- packages that are painful to build from source

Do not use Conda as your main Mac package manager. Use it for environments.

### Basic Commands

| Task | Conda | Mamba |
|---|---|---|
| Create env | `conda create -n env python=3.12` | `mamba create -n env python=3.12` |
| Activate env | `conda activate env` | `mamba activate env` |
| Install package | `conda install package` | `mamba install package` |
| Remove env | `conda env remove -n env` | `mamba env remove -n env` |
| List envs | `conda env list` | `mamba env list` |

---

## 6. mas-cli

GitHub: https://github.com/mas-cli/mas  

`mas` is not a general package manager. It is a command-line interface for the Mac App Store.

It is useful when you want to script Mac App Store installs.

Example:

```bash
brew install mas
mas search xcode
mas install 497799835
mas list
mas upgrade
```

The annoying part is that App Store apps use numeric IDs. So `mas` is great for automation, but not as pleasant as Homebrew for discovery.

### Pros

- Lets you automate App Store installs.
- Useful for new Mac setup scripts.
- Works well together with Homebrew.
- Can list and update App Store apps from the terminal.

### Cons

- Only works with Mac App Store apps.
- You still need to be signed in to the App Store.
- App IDs are clunky.
- Not a replacement for Homebrew, MacPorts, or Nix.

### Best For

Use `mas` if you want a script that installs App Store apps on a fresh Mac.

For example, your setup script can install CLI tools with Homebrew and App Store apps with `mas`.

### Basic Commands

| Task | Command |
|---|---|
| Install mas | `brew install mas` |
| Search App Store | `mas search app-name` |
| Install app by ID | `mas install app-id` |
| List installed apps | `mas list` |
| Upgrade App Store apps | `mas upgrade` |

---

## What I Would Actually Use

My practical setup would be:

```bash
# Main package manager
Homebrew

# Optional: App Store automation
mas-cli

# Optional: Python/data environments
Conda or Mamba

# Optional: reproducible project environments
Nix

# Optional: cleaner Unix-style isolated tree
MacPorts
```

But I would not install all of them just because they exist.

That is how you create a dependency soup.

For most people:

```bash
Homebrew + mas + maybe Conda/Mamba
```

That is already enough.

For serious reproducible dev environments:

```bash
Homebrew + Nix
```

For old-school Unix/macOS package management:

```bash
MacPorts
```

For temporary tool execution:

```bash
pkgx
```

---

## Should You Mix Package Managers?

You *can*, but be careful.

The biggest problem is PATH order.

For example, you might have:

```bash
/usr/bin/python3
/opt/homebrew/bin/python3
/opt/local/bin/python3
~/miniconda3/bin/python3
/nix/store/.../bin/python3
```

Then you run:

```bash
python3 --version
which python3
```

And suddenly you are not using the Python you thought you were using.

Same story with:

```bash
git
openssl
curl
node
java
ruby
```

So if you mix package managers, have a reason.

Good combinations:

| Combination | Makes Sense? | Why |
|---|---:|---|
| Homebrew + mas | Yes | Homebrew for normal packages, mas for App Store apps |
| Homebrew + Conda/Mamba | Yes | Homebrew for system tools, Conda/Mamba for Python envs |
| Homebrew + Nix | Yes, if disciplined | Homebrew for Mac-native convenience, Nix for reproducible projects |
| MacPorts + Homebrew | Risky | Possible, but PATH/dependency confusion is common |
| Conda as everything | No | Conda is not a general macOS package manager |
| Nix as everything | Maybe | Powerful, but not beginner-friendly on macOS |

---

## What About Fink?

Fink is historically important, but I would not recommend it for a modern macOS setup in 2026.

The project still exists, but current macOS support looks behind modern releases. If you are maintaining an old Mac OS X environment, maybe it matters. For a normal modern Mac, I would pick Homebrew, MacPorts, or Nix instead.

---

## Final Recommendation

Use **Homebrew** as the default.

Add **mas-cli** if you want to automate App Store apps.

Use **Conda/Mamba** only for Python/data/ML environments.

Use **Nix** when reproducibility matters enough to justify the learning curve.

Try **pkgx** if you like lightweight “run this tool now” workflows.

Use **MacPorts** if you want a more traditional isolated Unix package tree, especially on older Macs.

Do not install every package manager. Pick the ones that solve your actual problem.

The boring setup that works:

```bash
Homebrew + mas + Conda/Mamba when needed
```

The nerd setup that scales:

```bash
Homebrew + Nix
```

The old-school clean Unix setup:

```bash
MacPorts
```

And honestly, for most Mac users, Homebrew alone is already 80% of the win.