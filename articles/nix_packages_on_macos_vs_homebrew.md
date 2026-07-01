---
title: "Nix Packages on macOS: Could It Be Better Than Homebrew?"
description: "Let's compare which package manager works the best for macOS."
pubDate: 2026-07-01
tags: ["NixOS", "macOS", "Linux", "HomeBrew"]
draft: false
---
Homebrew has become the default package manager on macOS, but Nix has
quietly built a loyal following among developers who care about
reproducible environments, isolated dependencies, and rollback support.

The downside? It looks intimidating.

Let's see what it actually does differently.

------------------------------------------------------------------------

## Homebrew vs Nix Philosophy

Homebrew installs software into locations like:

``` text
/opt/homebrew
/usr/local
```

Applications often share libraries and dependencies.

Nix takes the opposite approach.

Every package lives inside:

``` text
/nix/store
```

Each package directory contains a cryptographic hash.

Example:

``` text
/nix/store/9yx9n4xv3l4i-python3-3.13.2
```

Nothing gets overwritten.

Multiple versions happily coexist.

------------------------------------------------------------------------

## What Does the Nix Directory Look Like?

A typical installation contains directories similar to:

``` text
/nix
 ├── store/
 ├── var/
 ├── profiles/
 └── gcroots/
```

**store**

Contains every installed package.

**profiles**

Represents "generations" of installed environments.

**gcroots**

Prevents packages currently in use from being garbage collected.

**var**

Stores metadata used by the package manager.

Unlike Homebrew, binaries aren't scattered throughout the filesystem.

------------------------------------------------------------------------

## Why Are Paths So Weird?

Those long hashes aren't random.

They're calculated from things such as:

-   package version
-   compiler
-   dependencies
-   build options

If anything changes, the hash changes.

That makes every package effectively immutable.

------------------------------------------------------------------------

## Reproducible Development Environments

Instead of writing setup instructions, projects can include a
configuration file.

For example:

``` bash
nix develop
```

creates an isolated shell with the exact compiler, libraries and tools
defined for that project.

Two developers can work on different versions of Python, Rust or Node
without interfering with each other.

------------------------------------------------------------------------

## What Permissions Does Nix Need?

This is one of the biggest differences from Homebrew.

During installation, Nix usually:

-   creates the **/nix** directory
-   installs a background daemon (multi-user installation)
-   creates dedicated build users such as `_nixbld1`, `_nixbld2`, etc.
-   updates shell configuration
-   may enable the daemon through macOS launch services

Because these are system-wide changes, the installer typically asks for
administrator privileges.

After installation, day-to-day package management usually does **not**
require running every command with `sudo`. The daemon performs
privileged operations while regular users interact with it through the
Nix client.

------------------------------------------------------------------------

## Why Does It Create Build Users?

The build users are intentional.

Packages are built inside isolated environments using dedicated system
accounts.

Benefits include:

-   improved isolation
-   fewer accidental writes outside the build
-   better reproducibility
-   reduced impact if a build script behaves unexpectedly

This design is very different from traditional package managers.

------------------------------------------------------------------------

## Does Nix Modify macOS?

Much less than many people expect.

It doesn't replace system libraries.

It doesn't overwrite Apple-provided software.

Most changes are concentrated in:

-   `/nix`
-   shell configuration
-   daemon configuration
-   launchd service registration

Your actual macOS installation remains largely untouched.

------------------------------------------------------------------------

## Rollbacks

Every environment is stored as a generation.

If an update breaks something, switching back is straightforward.

That's a feature many developers don't appreciate until they actually
need it.

------------------------------------------------------------------------

## Can Homebrew and Nix Coexist?

Yes.

A common setup is:

-   Homebrew for GUI apps and miscellaneous utilities
-   Nix for programming languages and project environments

Many experienced macOS developers run both without issues.

------------------------------------------------------------------------

## Downsides

Nix isn't perfect.

Expect to learn concepts like:

-   flakes
-   derivations
-   channels
-   generations
-   garbage collection

The learning curve is significantly steeper than Homebrew.

------------------------------------------------------------------------

## Final Thoughts

If you simply want to install `wget` or `ffmpeg`, Homebrew remains the
easiest choice.

If you frequently switch between projects, maintain multiple language
versions, or want every developer on your team to use identical
environments, Nix offers advantages that are difficult to replicate with
traditional package managers.

It isn't necessarily faster.

It isn't necessarily simpler.

But it is one of the most interesting approaches to package management
available on macOS today.