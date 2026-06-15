---
title: "How Automation Worked Before Selenium and Playwright"
description: "A practical look at old-school test automation: desktop tools, macro recorders, object repositories, Sikuli, AutoIt, WinRunner, QTP, Robot Framework, and why modern web automation feels so different."
tags: ["Software Testing", "QA", "Cybersecurity", "Test Automation", "Sikuli"]
pubDate: "2026-06-15"
draft: false
---
Today, when people say "test automation", they usually mean something like Selenium, Playwright, Cypress, Appium, REST API tests, CI pipelines, Docker containers, and a nice HTML report at the end.

Very clean. Very modern. Very developer-like.

But automation was not always like that.

Before Selenium became the default language of web UI testing, and long before Playwright made browser automation feel almost boringly smooth, a lot of automation was basically this:

- click this button
- wait two seconds
- type this text
- compare this screenshot
- pray the window opens in the same position
- pray the machine is not slow today
- pray nobody moves the mouse

And I am only half joking.

Old-school automation was much closer to controlling a real desktop than controlling an application through a clean API. It was not always beautiful, but it taught testers something very important: software does not live only in code. It lives in windows, dialogs, focus problems, timing issues, installers, printers, Citrix sessions, old ERP systems, and weird native applications that nobody wants to touch but the whole company depends on.

This article is about that world.

Not nostalgia for the sake of nostalgia. More like: how did automation work before modern browser frameworks made everyone think UI testing starts with `page.locator()`?

## The Automation World Before Modern Web Testing

Before web apps took over everything, most business software was desktop software.

Banks had thick clients. Factories had Windows applications. Insurance companies had ancient internal tools. Medical systems had native interfaces. Accounting departments had desktop software with ten thousand fields. Big enterprise products were often Windows-first, sometimes Java Swing, sometimes Delphi, sometimes .NET WinForms, sometimes some nightmare that looked like it had survived three acquisitions.

So automation tools were built around that reality.

They did not start by asking:

> What is the DOM element?

They asked:

> What is on the screen, and can I click it?

That is a completely different mindset.

Modern browser automation usually talks to the browser engine. Selenium uses browser drivers. Playwright controls browsers through modern automation protocols. The test has some real understanding of the page structure.

Older desktop automation often worked through one of these approaches:

1. coordinate-based clicking
2. image recognition
3. Windows control inspection
4. accessibility APIs
5. record-and-playback tools
6. vendor-specific object models
7. scripting languages that simulated a human user

Each approach had strengths. Each approach also had ways to ruin your day.

## Coordinate-Based Automation: The Caveman Method That Still Exists

The most primitive kind of automation is coordinate automation.

Move mouse to X=430, Y=220. Click. Type text. Press Tab. Press Enter.

That sounds stupid now, but it was surprisingly common. And honestly, it still exists in some places, especially in internal tools, remote desktops, virtual machines, and legacy systems where there is no better interface.

The problem is obvious.

If the screen resolution changes, the test breaks. If the window opens slightly shifted, the test breaks. If the toolbar is collapsed, the test breaks. If Windows scaling changes from 100% to 125%, the test becomes modern art.

But coordinate automation had one advantage: it worked on almost anything.

It did not care whether the application was written in C++, Java, Delphi, Visual Basic, or some enterprise framework from 1998. If the user could see it and click it, the script could try to click it too.

This is the ugly ancestor of many later automation tools.

## Macro Recorders: Automation for People Who Did Not Want to Code

A lot of old automation started with record-and-playback.

You clicked through the application manually. The tool recorded your actions. Then you pressed Play and hoped it could repeat them.

For demos, this looked magical.

For real testing, it was often painful.

Recorders produced scripts that were too literal. They captured too much noise. Instead of creating a clean reusable test, they created a long list of low-level actions:

```text
Click button at position
Wait
Set field
Wait
Click menu
Wait
Press Enter
```

Then one dialog changed and half the script died.

Still, recorders were important. They gave non-programmers a way into automation. They also shaped the whole enterprise testing industry. Many commercial tools sold automation as something a manual tester could create by recording business flows.

That promise was never completely fake, but it was oversold.

The hard part was never recording the first test.

The hard part was maintaining 500 recorded tests after the application changed.

## WinRunner, QTP, and the Enterprise Automation Era

Before Selenium became the default answer for web testing, commercial tools ruled many QA departments.

Mercury WinRunner and later QuickTest Professional, better known as QTP, were big names in that world. QTP eventually became HP UFT, then Micro Focus UFT, and now OpenText UFT One. The naming history alone tells you how much enterprise software likes mergers.

These tools were not just simple clickers. They had object repositories, scripting, checkpoints, test management integrations, reports, and support for many types of enterprise applications.

The idea was simple:

Instead of saying "click at coordinates", the tool tried to identify UI objects.

For example:

```text
Window("Login").TextBox("Username").Set "admin"
Window("Login").TextBox("Password").SetSecure "*****"
Window("Login").Button("OK").Click
```

This was already much better than raw coordinates.

But the object repository became its own problem.

You had to maintain names, properties, mappings, and object definitions. If developers changed labels, control IDs, window hierarchy, or component libraries, the automation could break. And because many of these tools were GUI-heavy and proprietary, maintaining them sometimes felt like testing inside a testing tool instead of writing normal code.

Still, for many companies, tools like WinRunner and QTP were the serious automation stack.

Not Selenium. Not Playwright. Not GitHub Actions.

A Windows machine, a licensed commercial automation tool, a test management system, and a QA engineer fighting with object recognition.

That was the job.

## AutoIt: The Small Windows Automation Hammer

AutoIt is one of those tools that looks simple until you realize how useful it can be.

It is a BASIC-like scripting language for automating the Windows GUI. It can send keystrokes, move the mouse, click controls, interact with windows, and automate routine desktop tasks.

In QA, AutoIt was often used for the annoying things that web automation could not handle well.

For example:

- native file upload dialogs
- Windows installers
- system tray applications
- desktop configuration flows
- legacy Windows tools
- small setup utilities
- printing dialogs
- authentication popups

Even in Selenium projects, people used AutoIt as a helper when the browser automation hit a native Windows dialog.

A typical situation looked like this:

Selenium can click the upload button on the web page. But then Windows opens a native file chooser. Selenium does not control that dialog directly. So the test calls an AutoIt script to type the path and press Enter.

That is old-school automation living inside modern automation.

AutoIt is not fashionable, but it represents something important: automation is not always a clean test framework. Sometimes it is a practical script that removes a painful manual step.

## Sikuli: Automation by Looking at the Screen

Sikuli was one of the most interesting ideas in GUI automation.

Instead of identifying elements by IDs, XPath, names, or accessibility properties, Sikuli used screenshots.

You took a screenshot of a button, icon, field, menu item, or visual element. Then the script looked for that image on the screen and interacted with it.

Conceptually, that is very cool.

Instead of writing:

```python
click("submit_button_id")
```

You used something closer to:

```python
click("submit_button.png")
```

For humans, this feels natural. We also recognize UI visually. We do not know the internal ID of a button. We see the button and click it.

Sikuli was especially useful when there was no good automation API:

- desktop applications
- games
- Flash applications
- remote desktops
- Citrix sessions
- virtual machines
- custom UI frameworks
- old enterprise software
- applications with poor accessibility support

But image-based automation has a dark side.

Small visual changes can break tests. Theme changes can break tests. Font rendering can break tests. Different DPI can break tests. Anti-aliasing can break tests. A button changing from blue to gray can break tests. If another window covers the target image, the script has no magic powers.

So Sikuli was powerful, but fragile.

It worked best when the environment was controlled: fixed resolution, fixed theme, stable VM, predictable application state.

In other words, Sikuli was great when you treated the test machine like a lab instrument, not like a normal laptop.

## Robot Framework: Keywords Before Everyone Talked About BDD

Robot Framework is not exactly "before Selenium" in the same way as old recorders or WinRunner, but it belongs to the older style of automation thinking.

Its big idea is keyword-driven testing.

Instead of writing only code, you describe test actions with readable keywords:

```robot
*** Test Cases ***
User Can Login
    Open Login Page
    Enter Username    demo
    Enter Password    password123
    Click Login Button
    User Should See Dashboard
```

This was attractive because it separated test intent from test implementation.

A business tester could read the scenario. A technical automation engineer could implement the keywords underneath. In theory, everyone wins.

In practice, it depends on discipline.

Good Robot Framework projects are clean and readable.

Bad Robot Framework projects become keyword soup:

```robot
Click Button And Wait And Verify Thing And Maybe Retry Login With Special Condition
```

Still, Robot Framework deserves respect. It helped many teams think beyond raw scripting. It supported acceptance testing, ATDD-style workflows, libraries, reports, and integration with many different automation backends.

It also shows that automation was never only about browsers. It was about building a language for testing business flows.

## Desktop Automation Was Often More Honest Than Modern Web Automation

Modern web automation can create a false sense of cleanliness.

You install Playwright, write a locator, run tests headless, and everything looks professional.

Old desktop automation was more brutal.

It exposed the real problem immediately:

- the application is slow
- the UI is inconsistent
- the installer is weird
- the dialog blocks the test
- the field has no stable identifier
- the app behaves differently on another machine
- focus randomly disappears
- the test machine matters
- environment setup is half the battle

That sounds bad, but it made testers more aware of the full system.

A good desktop automation engineer had to understand the operating system, windows, processes, files, registry, services, permissions, screen resolution, keyboard layouts, timing, and environment state.

That is not useless knowledge.

Actually, a lot of modern QA engineers are weak exactly there. They can write a browser test but get lost when the problem is outside the browser.

## The Main Problem Was Stability

The big enemy of old automation was not syntax.

It was stability.

UI automation is always fighting with timing and state. But old desktop automation had fewer safety nets.

Modern Playwright auto-waits for elements. It understands frames, network events, DOM state, visibility, browser contexts, downloads, permissions, and more.

Old tools often needed explicit waits everywhere:

```text
wait 2 seconds
click
wait 5 seconds
if window exists then continue
else fail
```

And fixed waits are poison.

If the machine is fast, tests waste time. If the machine is slow, tests fail. If the application hangs, the test may keep clicking into nothing.

That is why serious automation engineers slowly moved from simple scripts to frameworks.

They added helpers:

- wait until window exists
- wait until control is enabled
- retry click
- check process state
- restart application
- clean test data
- reset environment
- take screenshots on failure
- collect logs

Basically, they had to invent reliability features manually.

Today, Playwright gives you many of these ideas out of the box.

Back then, you built them yourself or suffered.

## Object Recognition vs Visual Recognition

Old automation usually lived between two worlds.

The first world was object recognition.

The tool inspected the application and found controls by properties: name, class, ID, text, hierarchy, accessibility information, or internal metadata.

This was cleaner when it worked.

The second world was visual recognition.

The tool looked at the screen like a human and matched images.

This worked when object recognition was impossible.

Neither approach was perfect.

Object recognition breaks when developers change internal properties or use custom controls that expose nothing useful.

Visual recognition breaks when the UI changes visually.

That is why old automation engineers often mixed approaches.

Use object recognition when possible. Use image recognition when necessary. Use keyboard shortcuts when reliable. Use direct database or API setup when the UI is too slow. Use logs to verify what the UI cannot show clearly.

Good automation was never pure.

It was practical.

## Web Automation Changed the Center of Gravity

Selenium changed things because web applications became the main battlefield.

Instead of automating a random Windows application from the outside, testers could automate the browser. The browser had a page model. HTML had elements. The DOM had structure. Locators existed. The same test could run against different browsers and environments.

It was not perfect. Selenium tests could still be flaky, ugly, slow, and badly written.

But the direction was different.

Automation moved closer to software development.

Tests lived in code repositories. Engineers used normal programming languages. CI became normal. Page Object Model became common. Test code started to look more like production code.

Then Playwright pushed this even further.

It made many painful browser problems feel native: auto-waiting, browser contexts, tracing, screenshots, videos, network control, multiple browser engines, better locators, and a test runner designed for modern web apps.

This is why people who started with Playwright sometimes underestimate how painful UI automation used to be.

They were born after the war.

## What We Lost

Modern automation is better. No question.

But we lost some instincts.

Old automation forced testers to think about the whole machine. Not only the web page.

You had to know what happens when:

- the app is installed
- the app updates
- the OS language changes
- the screen resolution changes
- a modal dialog appears
- a background process crashes
- a file is locked
- a permission prompt appears
- the network drive disappears
- the printer dialog opens
- the user session expires

A lot of modern test automation skips this layer.

It tests the happy web flow and ignores the environment around it.

That is fine for many SaaS products. But it is not enough for desktop apps, mobile apps, embedded systems, enterprise tools, or anything that touches the operating system.

Old-school automation was ugly, but it trained people to respect the environment.

## What Still Matters Today

Desktop automation is not dead.

It just became more niche.

You still see it in:

- banking systems
- medical software
- call center tools
- Windows desktop products
- industrial software
- POS systems
- installers
- internal admin tools
- RPA workflows
- legacy enterprise applications
- virtual desktop environments

Even modern AI agents are weirdly close to old automation ideas.

They look at the screen. They identify UI elements. They click things. They use accessibility trees. They combine visual recognition with structured UI metadata.

That sounds new because now there is AI branding on top.

But the core problem is old:

> How do you make software operate another piece of software through a user interface?

Sikuli people understood that. AutoIt people understood that. WinRunner and QTP people understood that. Desktop automation engineers understood that long before everyone started talking about agents controlling computers.

The tools changed.

The problem did not.

## The Practical Lesson for QA Engineers

If you only know Selenium or Playwright, you know one important slice of automation.

But not the whole thing.

A serious automation engineer should understand at least the basic categories:

- browser automation
- mobile automation
- API automation
- desktop automation
- visual automation
- accessibility-based automation
- system scripting
- test data setup
- CI and environment control

You do not need to become a WinRunner archaeologist. You do not need to write AutoIt every day. You do not need to automate everything through screenshots.

But you should know these things existed.

Because sooner or later, you will hit a problem that does not fit nicely into Playwright.

A file chooser. A system dialog. A desktop installer. A legacy admin tool. A remote machine. A weird certificate popup. A browser extension. A native mobile permission. A virtual desktop. A custom UI that exposes nothing useful.

At that moment, the old world becomes relevant again.

## Conclusion

Before Selenium and Playwright became the normal face of automation, UI testing was much more desktop-oriented, more visual, more fragile, and more dependent on the operating system.

It was macro recorders, object repositories, QTP scripts, WinRunner projects, AutoIt helpers, Sikuli screenshots, Robot Framework keywords, accessibility APIs, Windows dialogs, and a lot of waiting.

Some of it was bad.

Some of it was brilliant.

Most of it was practical.

Modern tools are better, but they did not appear from nowhere. They are a response to years of pain: flaky waits, bad object recognition, unreadable recorded scripts, painful maintenance, and environments that were never as stable as the demo machine.

So when people complain that Playwright is hard, I always want to say:

You have no idea how good you have it.

Try maintaining a screenshot-based desktop automation suite on Windows XP with a shared test machine and random popups.

After that, `page.getByRole()` feels like luxury.
