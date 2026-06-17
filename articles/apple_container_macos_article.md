---
title: "Apple Container: Linux Containers on macOS, but Done the Apple Way"
description: "A practical look at Apple's open-source container CLI and Containerization framework: what it is, how it works, basic commands, where it fits, and why it is not simply Docker Desktop with an Apple logo."
pubDate: 2026-06-16
tags: ["Apple Silicon", "Virtualization", "Virtual Image", "Mac OS"]
draft: false
---
Apple quietly did something interesting for developers: it released an open-source container tool for macOS.

Not a fancy app.  
Not another Docker Desktop skin.  
Not a random wrapper around existing Linux tools.

The project is literally called `container`.

Very Apple name, honestly. Almost too Apple.

At first glance, it sounds boring: “Apple made a container tool.” But under the hood, this is actually a pretty important piece of infrastructure for macOS developers, especially on Apple Silicon.

The short version:

Apple’s `container` lets you create, build, and run Linux containers on a Mac. It uses OCI-compatible images, so it can work with the same general container image ecosystem people already know from Docker and registries. But instead of running all containers inside one big shared Linux VM, Apple’s design runs each container inside its own lightweight virtual machine.

That is the interesting part.

This is not just “Docker but from Apple.”  
It is Apple applying its usual philosophy to containers:

> isolate more, expose less, integrate tightly with macOS, and hide the weird parts until you need them.

Whether that is good or annoying depends on what you are trying to do.

---

## The usual macOS container problem

Containers are a Linux thing.

Yes, people use Docker on macOS every day. But the actual Linux container model depends on Linux kernel features: namespaces, cgroups, filesystems, networking, capabilities, and so on.

macOS does not magically become Linux just because you install Docker Desktop.

So on a Mac, the usual trick is this:

```text
macOS host
  → Linux VM
    → containers inside that VM
```

That works. It is practical. It is why Docker Desktop, OrbStack, Colima, Lima, Rancher Desktop, and other tools exist.

But it also means that “running containers on macOS” has always been a bit of a lie. You are normally running Linux containers inside a Linux virtual machine that is running on your Mac.

Apple’s `container` does not remove the need for Linux. It still needs Linux. The difference is how Apple structures the isolation.

Instead of this:

```text
One Linux VM
  → container A
  → container B
  → container C
```

Apple’s approach is closer to this:

```text
container A → lightweight Linux VM A
container B → lightweight Linux VM B
container C → lightweight Linux VM C
```

Each container gets its own lightweight VM boundary.

That is not a small design choice.

---

## What is Apple `container`?

`container` is an open-source command-line tool from Apple for creating and running Linux containers on macOS.

The official project says it is:

- written in Swift
- optimized for Apple Silicon
- based on the `Containerization` Swift package
- designed to consume and produce OCI-compatible container images
- able to pull from and push to standard container registries
- built around lightweight virtual machines

In normal-person terms:

```text
It is a command-line container tool for Mac users.
It understands normal container images.
It runs Linux containers using Apple’s virtualization stack.
```

You can do familiar things:

```bash
container run -it ubuntu:latest /bin/bash
container run -d --name web -p 8080:80 nginx:latest
container build --tag my-app .
container image list
container exec web sh
container stop web
container delete web
```

The commands look intentionally familiar. If you have used Docker, you will not feel completely lost.

But the architecture is different enough that you should not assume every Docker habit maps perfectly.

---

## What is Containerization?

There are two related projects:

```text
container
Containerization
```

`container` is the command-line tool.

`Containerization` is the lower-level Swift package that powers the tool. It provides APIs for running Linux containers on macOS using Apple’s `Virtualization.framework`.

So the stack roughly looks like this:

```text
container CLI
  → Containerization Swift package
    → Apple Virtualization.framework
      → lightweight Linux VM
        → containerized process
```

This is very Apple.

Instead of saying, “Here is a random daemon and a pile of Linux-ish glue,” Apple built a Swift framework and a CLI around its own platform APIs.

That matters because it makes this more than just a command-line toy. If Apple keeps pushing this, other Mac developer tools could eventually build container features using the same lower-level framework.

That is probably the long-term point.

---

## Basic requirements

This is where the hype needs to calm down a bit.

Apple `container` is mainly for Apple Silicon Macs. It is not designed as a universal retro Mac container solution. Your old Intel MacBook is not the target here.

The current project documentation focuses on:

- Apple Silicon Mac
- modern macOS support
- Apple’s virtualization stack
- Swift implementation
- OCI images

If you are on an M1, M2, M3, or M4 Mac with a supported macOS version, this is relevant.

If you are on Intel macOS, this is probably not the tool you should be building your workflow around.

If you are on Linux, just use normal Linux container tools. That is where containers are native anyway.

---

## Install and start the service

Apple distributes signed installer packages through the GitHub releases page.

After installing, start the service:

```bash
container system start
```

Check that it works:

```bash
container list --all
```

Or shorter:

```bash
container ls -a
```

If nothing is running yet, an empty list is normal.

Check version information:

```bash
container system version
```

Get general help:

```bash
container --help
```

Get help for a subcommand:

```bash
container run --help
container build --help
container image --help
container system --help
```

This is one of those tools where you should actually use `--help`. The project is moving, and command details can change between releases.

---

## First useful command: run a shell

The simplest test:

```bash
container run -it ubuntu:latest /bin/bash
```

Or with Alpine:

```bash
container run -it alpine:latest sh
```

Inside the container:

```bash
uname -a
cat /etc/os-release
exit
```

You are not in macOS anymore. You are in a Linux environment running through Apple’s container stack.

That is the whole trick.

For quick testing, add `--rm` so the container is removed after it stops:

```bash
container run -it --rm alpine:latest sh
```

This is the equivalent of saying:

```text
I need a temporary Linux shell.
Do not keep the container around after I exit.
```

---

## Run a background web server

A common test:

```bash
container run -d --name web -p 8080:80 nginx:latest
```

Then open:

```bash
open http://localhost:8080
```

Or test with curl:

```bash
curl http://localhost:8080
```

List running containers:

```bash
container ls
```

Show all containers, including stopped ones:

```bash
container ls -a
```

Stop it:

```bash
container stop web
```

Delete it:

```bash
container delete web
```

Or use the short alias if available in your version:

```bash
container rm web
```

The workflow feels familiar because it has to. Nobody wants to relearn basic container operations from zero just because Apple wrote the tool in Swift.

---

## Build an image

Create a small test project:

```bash
mkdir apple-container-test
cd apple-container-test
```

Create a `Dockerfile`:

```Dockerfile
FROM docker.io/python:alpine
WORKDIR /app
RUN echo '<h1>Hello from Apple container</h1>' > index.html
CMD ["python3", "-m", "http.server", "80", "--bind", "0.0.0.0"]
```

Build it:

```bash
container build --tag apple-web-test .
```

List images:

```bash
container image list
```

Run it:

```bash
container run -d --name apple-web -p 8080:80 apple-web-test
```

Test it:

```bash
curl http://localhost:8080
```

Stop it:

```bash
container stop apple-web
```

Delete it:

```bash
container delete apple-web
```

This is the point where the tool becomes practical. You are not just pulling random images; you can build your own image locally.

For QA and dev work, that matters.

---

## Use environment variables

Run with an environment variable:

```bash
container run --rm -e ENV_NAME=local alpine env
```

Example with Node:

```bash
container run --rm -e NODE_ENV=production node:22 node -e 'console.log(process.env.NODE_ENV)'
```

This is useful for testing simple backend services, fake APIs, scripts, build tools, or local test environments.

For example:

```bash
container run --rm \
  -e API_BASE_URL=http://localhost:8080 \
  -e FEATURE_FLAG_NEW_LOGIN=true \
  my-test-tool
```

Nothing exotic here. Just normal container workflow.

---

## Limit CPU and memory

Because Apple’s `container` runs each container as a lightweight VM, resource limits matter.

Example:

```bash
container run --rm --cpus 2 --memory 1g alpine sh
```

For heavier workloads:

```bash
container run --rm --cpus 8 --memory 8g ubuntu:latest bash
```

This is not just about politeness. On a MacBook, a runaway container build can make the whole machine feel awful.

For local QA or dev work, I would not run random containers with unlimited resources unless I had a good reason.

A boring but practical default:

```bash
container run --rm --cpus 2 --memory 2g image-name
```

Then increase only when needed.

---

## Mount host folders

Share a local folder with the container:

```bash
container run --rm \
  --volume "$PWD:/workspace" \
  alpine ls -la /workspace
```

Same idea using `--mount`:

```bash
container run --rm \
  --mount source="$PWD",target=/workspace \
  alpine ls -la /workspace
```

This is useful for:

- build scripts
- local test data
- static site generation
- running Linux CLI tools against local files
- quick reproducible environments
- QA helper scripts

Example:

```bash
container run --rm \
  --volume "$PWD:/work" \
  node:22 \
  sh -c "cd /work && npm test"
```

This is the kind of use case where containers are actually useful on a Mac. You do not necessarily want to install every runtime and dependency locally. You just want to run a project in a clean environment.

---

## Execute commands inside a running container

Start a container:

```bash
container run -d --name web -p 8080:80 nginx:latest
```

Run a command inside it:

```bash
container exec web ls /
```

Open an interactive shell:

```bash
container exec -it web sh
```

Check logs:

```bash
container logs web
```

Stop it:

```bash
container stop web
```

This is normal container life:

```text
run
inspect
exec
logs
stop
delete
```

If you know those verbs, you can survive most container tools.

---

## Images and registries

List local images:

```bash
container image list
```

Pull an image:

```bash
container image pull alpine:latest
```

Tag an image:

```bash
container image tag apple-web-test registry.example.com/user/apple-web-test:latest
```

Login to a registry:

```bash
container registry login registry.example.com
```

Push an image:

```bash
container image push registry.example.com/user/apple-web-test:latest
```

The important part is OCI compatibility. Apple is not trying to create a weird Apple-only image format. That would be classic platform-lock-in nonsense, and thankfully that is not the direction here.

The tool works with standard container image concepts.

That means the image you build locally can still make sense outside Apple’s tool, assuming the architecture and dependencies line up.

---

## Arm64 vs amd64

Apple Silicon is Arm.

Most modern images have Arm builds now, but not all of them.

A lot of pain in Apple Silicon container workflows comes from architecture mismatch:

```text
Your Mac: arm64
Some old image: amd64 only
Production server: maybe amd64
CI runner: maybe amd64
```

Apple’s tooling can support multi-architecture workflows and Rosetta-based amd64 execution in some cases, but you should still be aware of what you are running.

Check inside a container:

```bash
container run --rm alpine uname -m
```

Expected on Arm:

```text
aarch64
```

Build for multiple architectures:

```bash
container build \
  --arch arm64 \
  --arch amd64 \
  --tag registry.example.com/user/my-app:latest \
  .
```

Run a specific architecture when supported:

```bash
container run --arch arm64 --rm image-name uname -m
container run --arch amd64 --rm image-name uname -m
```

This is one of the big practical testing points.

Do not assume “it works on my M-series Mac” means it will work on an x86 Linux production machine. Also do not assume x86 images will always feel good through translation.

For QA, architecture is part of the environment. Treat it that way.

---

## Networking: direct IPs, ports, and DNS

The nice thing about Apple’s design is that containers can get their own IP addresses.

List running containers:

```bash
container ls
```

You may see an IP in the output.

For many normal use cases, port publishing is still the clean workflow:

```bash
container run -d --name web -p 8080:80 nginx:latest
```

Then:

```bash
curl http://localhost:8080
```

You can also use a local DNS domain if configured:

```bash
sudo container system dns create test
```

Then a container named `my-web-server` can be reachable as something like:

```text
my-web-server.test
```

That is nicer than remembering container IP addresses.

For test environments, this is useful. You can have predictable local service names instead of random ports everywhere.

Example idea:

```text
api.test
auth.test
mock-payment.test
web.test
```

Of course, the moment local DNS enters the picture, you should document it. Otherwise, two months later you will forget why your Mac resolves some magic `.test` domain.

---

## Accessing a host service from a container

Sometimes you need the container to call something running on the Mac host.

Example:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Then you want a container to call that service.

Apple’s docs show a DNS-based approach where you create a domain that maps back to a host address:

```bash
sudo container system dns create host.container.internal --localhost 203.0.113.113
```

Then from the container:

```bash
container run -it --rm alpine/curl curl http://host.container.internal:8000
```

This is the kind of thing that always becomes annoying in local dev setups. Containers need to talk to the host. The host needs to talk to containers. Containers need to talk to each other. Then you change Wi-Fi networks and suddenly everything breaks.

So for QA/dev purposes, keep your local networking boring:

- use predictable ports
- use predictable local DNS only when needed
- document the setup
- avoid magic if a simple `localhost:8080` works

---

## Volumes and persistent data

For persistent data, use volumes.

Create a volume:

```bash
container volume create mydata
```

List volumes:

```bash
container volume list
```

Use it:

```bash
container run --rm \
  --volume mydata:/data \
  alpine sh -c "echo hello > /data/test.txt"
```

Read it later:

```bash
container run --rm \
  --volume mydata:/data \
  alpine cat /data/test.txt
```

Remove it:

```bash
container volume delete mydata
```

This matters for testing databases.

Example:

```bash
container run -d \
  --name postgres-test \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16
```

Now your database data survives container restarts.

That can be good or bad.

For repeatable QA, persistent state is often the enemy. You want to know whether the app works from a clean state, not from some cursed local volume you created last week.

So use volumes intentionally.

---

## Cleanup commands

Container tools always collect junk.

Images, stopped containers, volumes, build cache, old networks — it all piles up.

Basic cleanup:

```bash
container ls -a
container image list
container volume list
```

Stop a container:

```bash
container stop name
```

Delete it:

```bash
container delete name
```

Delete an image:

```bash
container image delete image-name
```

Delete a volume:

```bash
container volume delete volume-name
```

Stop the whole system service:

```bash
container system stop
```

Start it again:

```bash
container system start
```

Show disk usage if your version supports it:

```bash
container system df
```

For QA machines, cleanup is not optional. Local test environments rot fast.

---

## The security angle: one VM per container

The big philosophical difference is isolation.

Traditional local Mac container setups often use one shared Linux VM. Inside that VM, multiple containers run side by side.

Apple’s approach runs each container in its own lightweight VM.

That gives you a stronger boundary between containers. It also fits Apple’s general style:

```text
Sandbox the app.
Limit the permission.
Give it only the files it needs.
Keep boundaries clear.
```

You can see the same philosophy elsewhere in Apple platforms:

- app sandboxing
- TCC permissions
- per-app containers in `~/Library/Containers`
- iOS app data isolation
- notarization
- entitlements
- privacy prompts
- system extensions instead of old kernel extensions

Apple likes controlled boundaries. Sometimes that protects users. Sometimes it makes developers want to throw the laptop out the window. Both can be true.

With `container`, the per-container VM approach is not just security theater. It changes the trust model.

If I am running random Linux images on my Mac, I would rather have better isolation by default.

But there is a tradeoff.

More isolation can mean more complexity. Networking can be different. Mounts can be more explicit. Compatibility can be less “it just works like Docker Desktop.” Some workflows that assume one shared Linux machine may not map perfectly.

That is the Apple tax.

---

## What about macOS app containers?

There is another meaning of “Apple containers” that people can confuse with this.

macOS sandboxed apps often store data here:

```text
~/Library/Containers/
```

Example:

```text
~/Library/Containers/com.example.app/
```

That is not the same thing as Linux containers.

A macOS app container is an app sandbox storage area. It is about limiting where a macOS app can read and write user data.

Apple’s `container` CLI is about running Linux container images.

They share the same general philosophy — isolation — but they are different technologies.

So when someone says “Apple container,” ask what they mean:

```text
Apple app sandbox container?
Linux containers on macOS using Apple's container CLI?
Containerization Swift framework?
iCloud container?
Core Data persistent container?
```

Apple reuses simple words until everything becomes confusing. Very elegant. Very annoying.

---

## Is this a Docker replacement?

Not fully.

At least not yet.

For some workflows, yes, it can replace Docker Desktop:

- running a Linux shell
- running a local web server
- building simple images
- testing CLI tools
- running small services
- pulling OCI images
- pushing images to registries
- local dev experiments
- isolated QA helper environments

For other workflows, Docker Desktop or OrbStack may still be better:

- complex Compose setups
- Kubernetes-heavy local development
- GUI management
- existing team workflows
- Docker Desktop extensions
- enterprise policy integration
- mature troubleshooting docs
- wider community examples
- compatibility with scripts that assume Docker exactly

This is the important practical answer:

```text
Apple container is exciting infrastructure.
It is not automatically the best daily tool for every developer today.
```

If your company already has a giant Docker Compose workflow, you probably will not casually switch tomorrow.

If you are an individual developer or QA engineer who wants a clean native-feeling way to run Linux containers on an Apple Silicon Mac, this is worth watching and trying.

---

## Where this is useful for QA engineers

For QA, this is not just “DevOps toy of the week.”

It can be useful in normal testing work.

### 1. Run fake APIs locally

You can run a mock backend:

```bash
container run -d \
  --name mock-api \
  -p 8080:80 \
  my-mock-api
```

Then point the mobile app or web app to:

```text
http://localhost:8080
```

Or to your Mac’s local network IP if testing from a phone.

### 2. Run test databases

```bash
container run -d \
  --name postgres-test \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:16
```

Useful for backend testing and local integration tests.

### 3. Run CLI tools without installing them

```bash
container run --rm -v "$PWD:/work" node:22 sh -c "cd /work && npm test"
```

Or:

```bash
container run --rm -v "$PWD:/work" python:3.12 sh -c "cd /work && pytest"
```

Your Mac stays cleaner.

### 4. Reproduce Linux-only bugs

If something fails only in Linux CI, containers help you reproduce it locally.

Not always perfectly. But often enough.

### 5. Test architecture assumptions

Run arm64 and amd64 variants when possible:

```bash
container run --arch arm64 --rm image uname -m
container run --arch amd64 --rm image uname -m
```

This is useful when your laptop is Apple Silicon but production is still x86.

### 6. Build disposable test environments

For testing, disposable environments are gold.

```bash
container run --rm image-name
```

Run it. Test it. Destroy it.

No local mess. No mystery packages installed through Homebrew at 2 AM.

---

## Where this is weak

I like the direction, but let’s not pretend this solves everything.

### It is Apple Silicon-first

That is fine in 2026, but it still means the tool is not for every Mac.

### It is young compared to Docker

Docker has years of ecosystem weight, tutorials, Stack Overflow answers, Compose examples, CI integrations, and painful bug reports already solved by somebody else.

Apple’s tool is newer. You will hit rough edges.

### It is not Linux-native Docker

Linux is still the natural home for Linux containers.

macOS will always be a host that needs virtualization for this. Apple can make that nicer, but it cannot change what Linux containers are.

### Team compatibility matters

A tool can be technically cool and still be wrong for your team.

If everyone uses Docker Compose, CI expects Docker, documentation says Docker, and onboarding assumes Docker Desktop, switching to Apple `container` might create more confusion than value.

### GUI users may not care

This is a command-line tool. If someone wants a polished visual container dashboard, this is not the main selling point.

---

## Basic command cheat sheet

```bash
# start the service
container system start

# stop the service
container system stop

# show version
container system version

# general help
container --help

# command help
container run --help
container build --help
container image --help

# list running containers
container ls

# list all containers
container ls -a

# run an interactive shell
container run -it --rm alpine:latest sh

# run Ubuntu bash
container run -it ubuntu:latest /bin/bash

# run a web server
container run -d --name web -p 8080:80 nginx:latest

# show logs
container logs web

# exec into a running container
container exec -it web sh

# stop a container
container stop web

# delete a container
container delete web

# build an image
container build --tag my-app .

# list images
container image list

# pull image
container image pull alpine:latest

# delete image
container image delete alpine:latest

# run with env variable
container run --rm -e NODE_ENV=test node:22 node -e 'console.log(process.env.NODE_ENV)'

# run with CPU and memory limits
container run --rm --cpus 2 --memory 2g alpine sh

# mount current folder
container run --rm -v "$PWD:/work" alpine ls -la /work

# create volume
container volume create mydata

# list volumes
container volume list

# use volume
container run --rm -v mydata:/data alpine sh -c "echo hello > /data/test.txt"

# inspect volume data
container run --rm -v mydata:/data alpine cat /data/test.txt

# delete volume
container volume delete mydata

# login to registry
container registry login registry.example.com

# tag image
container image tag my-app registry.example.com/user/my-app:latest

# push image
container image push registry.example.com/user/my-app:latest
```

---

## My take

Apple `container` is one of those tools that looks boring until you think about the architecture.

A normal user will not care. A lot of developers will continue using Docker Desktop or OrbStack because their workflows already work. That is reasonable.

But for Apple Silicon Macs, this is a serious signal.

Apple is not saying:

```text
Here is a toy container CLI.
```

Apple is saying:

```text
We want a native macOS container stack built on Swift, Virtualization.framework, OCI images, and stronger isolation boundaries.
```

That matters.

For me, the interesting part is not even the CLI itself. The interesting part is the platform direction. Apple clearly understands that modern development needs Linux containers, even on macOS. Instead of pretending macOS is Linux, Apple is building a Mac-native way to run Linux container workloads.

Very Apple.  
A bit controlled.  
Probably annoying in places.  
Also probably the right architectural direction for the Mac.

I would not delete Docker Desktop tomorrow just because this exists.

But I would absolutely test it, learn the commands, and keep an eye on where it goes.

Especially if you are on Apple Silicon and you like clean, isolated, command-line-driven development environments.