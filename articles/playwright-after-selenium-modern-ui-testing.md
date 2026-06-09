---
title: "Playwright After Selenium: Why Modern UI Testing Finally Feels Less Painful"
description: "A practical, slightly opinionated guide to Playwright from the point of view of someone who spent years with Selenium, waits, drivers, flaky tests, and browser drama."
pubDate: 2026-06-08
tags: ["Playwright", "Selenium", "Test Automation", "QA Automation", "JavaScript", "TypeScript", "Web Testing", "E2E Testing", "Automation Frameworks", "CI/CD"]
draft: false
---
I spent a lot of time with Selenium.

Not just "I installed Selenium once and clicked a button" time. I mean real Selenium time. WebDriver setup, driver versions, explicit waits, random `StaleElementReferenceException`, strange CI failures, browser updates breaking things, tests passing locally and dying in the pipeline like they had a personal problem with Jenkins or GitHub Actions.

Selenium was not useless. Far from it. Selenium basically carried web automation for years. A lot of QA automation careers were built on top of Selenium, mine included. It gave us a standard, it gave us WebDriver, it gave companies a way to stop clicking the same forms manually like it was 2008 forever.

But after working with modern tools, especially Playwright, it becomes hard not to notice something:

Selenium often feels like a tool from the old web.

Playwright feels like a tool built for the web we actually test now.

And that difference matters.

## The Selenium Mindset

When you come from Selenium, you usually think in this pattern:

1. Open browser.
2. Find element.
3. Wait.
4. Click.
5. Wait again.
6. Hope the page did what you think it did.
7. Add another wait because CI is slower.
8. Pretend this is engineering.

That sounds harsh, but let's be honest. A huge amount of Selenium code in real projects is not clean automation. It is survival automation.

You see things like this:

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement loginButton = wait.until(
    ExpectedConditions.elementToBeClickable(By.id("login"))
);
loginButton.click();
```

This is not terrible code. It is normal Selenium code.

But this is also the point. You are constantly managing the timing problem yourself. Is the element visible? Is it clickable? Is it attached to the DOM? Did React re-render it? Did the animation finish? Did the browser actually navigate? Did the network request complete? Did the frontend framework decide to replace the entire component because someone changed one state variable?

Selenium can handle a lot of this, but it often makes you build the reliability layer around it.

Playwright puts much more of that reliability layer into the framework itself.

## Playwright Feels Like Less Plumbing

The first thing that hits you with Playwright is that a lot of boring setup disappears.

With Selenium, especially in older projects, you think about browsers, drivers, driver versions, remote grids, capabilities, and all that ceremony. It is powerful, but it is also a lot of glue.

With Playwright, you install it and it brings a serious amount of the testing stack with it:

```bash
npm init playwright@latest
npx playwright test
```

You get a test runner. You get browser support. You get parallel execution. You get traces. You get screenshots and videos if configured. You get reports. You get a debugging story that does not feel like you are reading tea leaves from a CI log.

The official Playwright documentation describes Playwright Test as a full-featured test runner with auto-waiting, assertions, tracing, and parallelism across Chromium, Firefox, and WebKit. That is important because it means Playwright is not just a browser automation library. It is more like a complete E2E testing toolkit.

And that is probably the biggest difference after Selenium.

Selenium gives you the engine.

Playwright gives you the engine, dashboard, black box recorder, and mechanic tools in the trunk.

## Auto-Waiting Is Not Magic, But It Is a Big Deal

Playwright's auto-waiting is one of those features that sounds small until you go back to a flaky Selenium suite and start crying internally.

In Playwright, when you do this:

```ts
await page.getByRole('button', { name: 'Login' }).click();
```

Playwright does not just immediately throw a click at whatever random DOM node it found.

It performs actionability checks. It waits until the element is visible, stable, enabled, and able to receive events. If those checks do not pass before the timeout, the action fails.

That is not magic. You still need to write good tests. You can still write garbage Playwright code. Trust me, any tool can be abused by a motivated team with bad deadlines.

But auto-waiting removes a massive amount of manual waiting code.

In Selenium, the question is often:

> What should I wait for before doing this?

In Playwright, the question becomes:

> What does the user actually see and do?

That is a healthier question.

## Locators Are a Different Philosophy

Selenium people often think in selectors:

```java
driver.findElement(By.cssSelector(".btn-primary")).click();
```

Playwright pushes you toward locators:

```ts
await page.getByRole('button', { name: 'Submit' }).click();
```

This looks like a small syntax difference. It is not.

Playwright locators are lazy. They are resolved when the action happens, not when you define them. They also come with retry-ability and strictness. If your locator matches multiple elements when it should match one, Playwright will complain. Selenium will often just grab something and then you get to debug why it clicked the wrong button at 2 AM.

Playwright also encourages user-facing locators:

```ts
await page.getByLabel('Email').fill('test@example.com');
await page.getByPlaceholder('Password').fill('secret');
await page.getByRole('button', { name: 'Sign in' }).click();
```

This is closer to how a user experiences the page.

Not always perfect, of course. Sometimes you still need `data-testid`. Sometimes the app is a div soup nightmare made by people who think accessibility is a browser plugin. But the default direction is better.

In Selenium, many teams build their own discipline around selectors.

In Playwright, the framework tries to push you into better discipline from the start.

## The Trace Viewer Is Where Playwright Starts Feeling Unfair

One of the worst parts of UI automation is debugging failed tests in CI.

The failure says:

```text
Element not found
```

Very helpful. Thank you, machine.

Now you open logs. Maybe there is a screenshot. Maybe the screenshot is from the wrong moment. Maybe the page was still loading. Maybe the element was behind a modal. Maybe the test user was not created. Maybe the frontend returned a 500 and your test is just the messenger getting blamed.

Playwright's trace viewer changes this experience a lot.

You can record a trace and inspect what happened step by step: actions, snapshots, network, console logs, screenshots, timings. It feels like having a flight recorder for your test.

Example config:

```ts
export default defineConfig({
  use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
});
```

This is the kind of thing that makes automation practical in a real team.

Because test automation is not only about writing tests. That is the easy part.

The hard part is maintaining them when they fail, when the app changes, when CI is unstable, when developers say "it works on my machine", and when product people want new features yesterday.

A tool that makes failure analysis faster is not a luxury. It saves your life slowly.

## Browser Contexts Are Cleaner Than Browser Gymnastics

Selenium can run multiple browser sessions, of course. It can do a lot. But Playwright's browser context model is just cleaner for many modern test scenarios.

A browser context is like an isolated browser profile. Cookies, local storage, sessions — all separated.

That means you can test multiple users without launching a completely new browser for each one:

```ts
const adminContext = await browser.newContext();
const userContext = await browser.newContext();

const adminPage = await adminContext.newPage();
const userPage = await userContext.newPage();
```

This is very useful for apps with roles, permissions, collaboration, dashboards, admin panels, and all the usual enterprise web app stuff.

Instead of fighting shared browser state, you can create isolated contexts quickly.

For automation architecture, this is a very nice idea.

## Playwright Is Great, But It Is Not a Religion

Now, because tech people love turning tools into religions, we need to say this clearly:

Playwright does not make Selenium obsolete in every possible case.

Selenium is still useful. It has huge language support. It has massive ecosystem history. It works with many vendors, grids, cloud providers, old enterprise stacks, and strange corporate environments where the browser version is controlled by a committee of ghosts.

If your company already has a stable Selenium framework, hundreds of tests, Java-based infrastructure, reporting, custom utilities, and people who understand it, you do not automatically throw everything away because Playwright is cooler.

That is childish engineering.

But if you are starting a new web UI automation project today, especially for a modern frontend, Playwright is very hard to ignore.

It gives you a better default experience.

And defaults matter.

Bad defaults create bad frameworks. Good defaults save average teams from themselves.

## The Java Problem

Here is where it gets interesting for people like me, because I usually like Java more than the whole JavaScript ecosystem circus.

Playwright does support Java. You can write Playwright tests in Java.

Example:

```java
try (Playwright playwright = Playwright.create()) {
    Browser browser = playwright.chromium().launch();
    Page page = browser.newPage();
    page.navigate("https://example.com");
    System.out.println(page.title());
}
```

So yes, if your background is Java, you are not locked out.

But the main Playwright experience is clearly strongest in the TypeScript and JavaScript world. The test runner, examples, docs, community snippets, and ecosystem usually feel most natural there.

That can be annoying if you spent years building Selenium + Java + Maven + JUnit/TestNG + Selenide + Allure pipelines.

But it is also a useful discomfort.

The modern web is JavaScript-heavy anyway. If you test web apps, understanding TypeScript is not wasted time. You do not need to become a frontend developer with 47 npm packages and emotional damage from React state management. But knowing enough TypeScript to write clean Playwright tests is a good investment.

And honestly, Playwright's syntax is not that scary.

A simple test looks like this:

```ts
import { test, expect } from '@playwright/test';

test('user can open the homepage', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle(/Example/);
});
```

That is readable.

No factory. No driver manager. No page load strategy debate. No ritual sacrifice to ChromeDriver.

Just test the page.

## What About Page Object Model?

People coming from Selenium often ask:

> Do we still use Page Object Model in Playwright?

Yes, but do not overbuild it.

Selenium frameworks often become huge architecture monuments. BasePage, BaseTest, DriverFactory, WaitUtils, ElementUtils, ConfigManager, BrowserManager, ScreenshotManager, ReportManager, and then somewhere underneath all that concrete there is one actual click.

Playwright needs less of that.

You can still use page objects:

```ts
export class LoginPage {
  constructor(private page: Page) {}

  emailInput = this.page.getByLabel('Email');
  passwordInput = this.page.getByLabel('Password');
  loginButton = this.page.getByRole('button', { name: 'Login' });

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}
```

That is fine.

But do not recreate your old Selenium framework just because you are emotionally attached to framework folders.

Playwright already gives you many things that Selenium frameworks had to build manually.

So the better question is not:

> How do I copy my Selenium framework into Playwright?

The better question is:

> Which parts of my Selenium framework are no longer needed?

That is where the cleanup begins.

## What Playwright Does Better Than Selenium

Here is my practical list.

### 1. Less waiting pain

Auto-waiting and web-first assertions reduce a lot of timing garbage.

You still need to understand async behavior, but you write fewer custom waits.

### 2. Better debugging

Trace viewer, screenshots, videos, console logs, network inspection — this is a major advantage.

A failed test without good artifacts is just a mystery bill someone has to pay.

### 3. Cleaner multi-browser setup

Playwright supports Chromium, Firefox, and WebKit from one API. It can also run against branded browsers like Chrome and Edge.

This is useful because WebKit testing gives you a closer signal for Safari-like behavior than pretending Chrome is the entire internet.

### 4. Better isolation

Browser contexts are fast and clean.

Testing different users, roles, permissions, and session states becomes less annoying.

### 5. Modern test runner

Parallel execution, fixtures, retries, projects, reporters, traces — it feels like a testing platform, not a pile of dependencies duct-taped together.

### 6. Better API testing integration

Playwright can also do API requests. That means you can set up data through API calls and verify UI behavior without clicking through five setup screens like a medieval monk.

Example:

```ts
const response = await request.post('/api/login', {
  data: {
    email: 'user@example.com',
    password: 'password'
  }
});
expect(response.ok()).toBeTruthy();
```

This is not a replacement for real API testing frameworks in every case, but for E2E setup and hybrid tests it is very useful.

## What Selenium Still Does Better

Playwright fans sometimes pretend Selenium has no reason to exist. That is not serious.

Selenium still has advantages.

### 1. Bigger legacy ecosystem

If your company is deep into Java, Selenium Grid, cloud providers, old reporting systems, and custom test infrastructure, Selenium may fit better.

### 2. More language maturity in some teams

Java Selenium is still very common. A lot of QA engineers know it. A lot of training material exists. A lot of old frameworks are based on it.

### 3. WebDriver standard

Selenium is built around the W3C WebDriver protocol. That gives it a different kind of standardization and compatibility story.

Playwright uses its own architecture and browser control model. That gives it power, but also makes it less universal in some enterprise situations.

### 4. Some corporate environments move slowly

In a normal modern project, Playwright may be better.

In a weird corporate project with old browsers, locked-down machines, remote grids, and ten years of QA infrastructure, "better" is not always the same as "possible."

This is why tool choice should be practical, not emotional.

## Migration From Selenium To Playwright

If you are moving from Selenium to Playwright, do not try to convert everything line by line.

That is the wrong way.

A Selenium test like this:

```java
driver.get("https://example.com/login");
driver.findElement(By.id("email")).sendKeys("test@example.com");
driver.findElement(By.id("password")).sendKeys("secret");
driver.findElement(By.cssSelector(".login-button")).click();
```

Should not simply become:

```ts
await page.goto('https://example.com/login');
await page.locator('#email').fill('test@example.com');
await page.locator('#password').fill('secret');
await page.locator('.login-button').click();
```

That works, but it misses the point.

A better Playwright version is closer to the user:

```ts
await page.goto('/login');
await page.getByLabel('Email').fill('test@example.com');
await page.getByLabel('Password').fill('secret');
await page.getByRole('button', { name: 'Login' }).click();

await expect(page.getByText('Dashboard')).toBeVisible();
```

The point is not syntax migration.

The point is mindset migration.

## My Recommended Playwright Project Structure

For a small or medium project, I would start simple:

```text
tests/
  auth/
    login.spec.ts
    logout.spec.ts
  dashboard/
    dashboard.spec.ts

pages/
  LoginPage.ts
  DashboardPage.ts

fixtures/
  users.ts
  test-data.ts

playwright.config.ts
```

Do not build a monster framework on day one.

Start with tests. Add page objects only when duplication becomes real. Add helpers when there is a repeated pattern. Add fixtures when setup becomes messy.

A lot of automation engineers over-engineer because it feels professional.

But a framework is not good because it has many folders.

A framework is good if failed tests are easy to understand, easy to fix, and hard to write incorrectly.

## CI/CD Is Where Playwright Makes Sense Fast

Playwright works nicely in CI because it already understands headless execution, browser installation, parallelism, retries, and artifacts.

A basic GitHub Actions setup can be pretty straightforward:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

This is clean enough that students and junior QA engineers can actually understand it.

That matters.

A framework nobody understands is not a framework. It is a haunted house.

## The Main Thing Selenium People Need To Unlearn

The biggest thing to unlearn is the obsession with manual waiting and low-level element control.

In Selenium, you often write tests like you are controlling the browser from the outside with a stick.

In Playwright, you write closer to how the page behaves.

Use locators. Use assertions. Use test isolation. Use traces. Use fixtures. Do not sleep for three seconds like it is a solution. It is not. It is just giving your flaky test a coffee break.

Bad:

```ts
await page.waitForTimeout(3000);
await page.locator('.success').isVisible();
```

Better:

```ts
await expect(page.getByText('Success')).toBeVisible();
```

This is the whole vibe.

Stop waiting for time.

Start waiting for meaning.

## Should You Learn Playwright In 2026?

Yes.

Especially if you already know Selenium.

Actually, knowing Selenium makes Playwright more valuable because you understand the pain Playwright is trying to remove.

If you are a QA engineer, test automation engineer, SDET, developer who owns frontend quality, or someone building a modern testing portfolio, Playwright is one of the most useful tools to learn right now.

Not because Selenium is dead.

Not because every LinkedIn automation guru says "Playwright is the future" while posting the same carousel for the 45th time.

But because Playwright solves real daily automation problems with fewer excuses.

It makes tests easier to write.

It makes failures easier to debug.

It makes parallel and cross-browser runs easier to set up.

It makes modern frontend testing feel less like duct tape engineering.

And after years of Selenium pain, that feels pretty good.

## Final Opinion

Selenium taught the industry how to automate browsers.

Playwright teaches the industry that browser automation does not have to feel like punishment.

If you are maintaining a huge Selenium framework, do not panic and rewrite everything just to look modern.

But if you are starting fresh, building a portfolio project, teaching automation, or creating a modern QA course, Playwright deserves serious attention.

My honest take:

Selenium is still important.

But Playwright is the tool I would rather start with today.

Because at some point, test automation should stop being a fight against the framework and start being a fight against actual product bugs.

And that is already enough pain.
