---
title: "Automation: Web + Mobile"
description: "Principles, locators, simple examples, and unit tests for practical QA automation."
pubDate: 2026-05-24
tags: ["Automation", "Web", "Mobile", "Locators", "JUnit", "Gradle"]
draft: false
---
## Principles, Locators, Simple Examples, and Unit Tests

**Element -> Action -> Assertion**  
**CSS / XPath / ID**  
**JUnit / Gradle**

## What Is Test Automation?

Test automation is not just a click recorder. It is code that verifies a product faster and more consistently than a human can.

Automation allows tests to run without manual involvement. Regression checks can run at night, in CI, or before a release.

Automation also checks system behavior through code. A test captures the expected result and quickly shows when something is broken.

The main value for the team is fast feedback. Developers learn earlier when an important scenario has been broken.

## Why Do We Need Automation?

The main goal is not to have “more tests”. The goal is a stable release process with a fast signal about risk.

Automation is useful for:

1. Fast regression testing  
   These are checks that are long and boring to repeat manually.

2. Critical scenarios  
   Login, payment, registration, profile updates, checkout, and other business-critical flows.

3. CI/CD  
   Tests can run automatically on a Pull Request or before deployment.

4. Release confidence  
   A good automated test suite reduces unpleasant surprises before a release.

Automation is especially useful when a scenario is repeated often, is important for the business, and has a clear expected result.

## When You Should Not Automate

Automated tests can save time, but they can also waste time if they are written too early.

It is usually a bad idea to automate when the UI is unstable and markup changes every day.

It is also risky to automate during the very early stage of a project, when it is still unclear what exactly should be tested.

If requirements change constantly, automated tests may break faster than they provide value.

Another common problem is lack of maintenance time. Automated tests are code, and code must be supported.

A practical rule: first stabilize the product and the manual scenario, then move that scenario into code.

## Web vs Mobile Automation

Web automation is usually easier to start with. Mobile automation requires more environment setup and more discipline.

For web automation, common tools include Selenium, Selenide, and Playwright. Web tests are usually easier to run locally and in CI. Web applications often have stable CSS selectors or data-testid attributes, and test debugging is usually faster.

For mobile automation, common tools include Appium, Espresso, XCUITest, and Maestro. Mobile tests often require emulators or real devices. They also deal with permissions, system dialogs, network conditions, and slower execution.

This does not mean mobile automation is bad. It means the setup has to be treated seriously.

## The Test Pyramid

Not everything should be tested through the UI. The lower the level of the test, the faster and more stable it usually is.

Unit tests are numerous, fast, and cheap.

API or integration tests are fewer and verify connections between parts of the system.

UI end-to-end tests should be limited to critical user paths.

A good strategy is not to move every check into the browser or Appium if the same logic can be verified reliably at the method, service, or API level.

## The Basic Principle of a UI Test

Almost every good UI test follows a simple chain:

1. Find an element  
   Use a locator such as id, CSS selector, XPath, accessibility id, or another supported locator strategy.

2. Check the element state  
   Make sure the element is visible, enabled, available, or contains the expected text.

3. Perform an action  
   Click, type text, select an option, scroll, or swipe.

4. Check the result  
   Use assertions to verify the URL, text, state, response, or application behavior.

A minimal example:

```java
WebElement loginButton = driver.findElement(By.id("login"));
loginButton.click();
assertEquals("/profile", currentUrl);
```

The important idea is simple: find something, interact with it, then verify the result.

## Locators: Choosing a Strategy

A locator is a contract between the test and the interface. The more stable this contract is, the less maintenance the test suite needs.

The best options are usually:

1. data-testid or test id  
   This is often the best option for automated tests.

2. id or accessibility id  
   These are good when the value is stable.

3. CSS selector by class or attribute  
   This is acceptable when the selector is not tightly coupled to the visual layout.

4. XPath by structure  
   This should be used carefully.

The main question is: will this locator break during a normal redesign?

If the answer is yes, the locator is probably weak.

## Web Locators: What to Choose

In Selenium and Selenide, the most common locator strategies are id, CSS selector, and XPath.

Examples:

```java
By.id("email")
```

This is fast, readable, and stable.

```java
By.cssSelector("[data-testid=login]")
```

This is one of the best options when the application has test-specific attributes.

```java
By.cssSelector("button[type=submit]")
```

This is acceptable if the semantic meaning of the button is stable.

```java
By.xpath("//button[text()='Login']")
```

This can work, but visible text may change.

```java
By.xpath("/html/body/div[2]/div[3]/button")
```

This is almost always a bad idea because it depends too much on page structure.

## CSS Selectors: Simple Examples

CSS selectors are usually easier to read than XPath and work well for web automation.

Examples:

```css
.login-form input[name="email"]
button[type="submit"]
[data-testid="checkout-button"]
.profile-card .user-name
a[href*="/settings"]
```

How to read them:

`.login-form` means an element with the class `login-form`.

`input[name="email"]` means an input element with the attribute `name="email"`.

`[data-testid="checkout-button"]` means an element with a stable testing attribute.

`a[href*="/settings"]` means a link whose href contains `/settings`.

For teaching, it is useful to show students not only the final locator, but also the logic of reading it.

## XPath: When It Is Useful

XPath is not evil. Bad XPath is evil.

Good examples:

```xpath
//button[text()="Login"]
//input[@placeholder="Email"]
//label[text()="Email"]/following::input[1]
//*[contains(@class,"error")]
```

Bad examples:

```xpath
/html/body/div[3]/div/div[2]/form/div[4]/button
//*[@id="root"]/div/div/div[2]/span[1]
```

XPath is useful when you need to find an element by text, by a nearby label, or through a more complex relationship.

Do not rely on div numbers and absolute paths. These locators break easily.

## Mobile Locators

In mobile automation, locator stability is even more important because mobile tests are usually slower than web tests.

Good mobile locator strategies include:

1. accessibility id  
   This is usually the best option for Appium and is also useful for accessibility.

2. resource-id  
   This is often a good choice on Android.

3. iOS predicate or class chain  
   These are native strategies for iOS.

4. XPath  
   XPath works, but it is often slower and more fragile.

In real projects, ask developers to add accessibility ids, content-desc values, or test tags for critical elements.

## A Simple Selenium Test

For beginners, it is useful to show the smallest possible scenario: open a page, find elements, perform actions, and check the result.

Example with Selenium and JUnit:

```java
@Test
void loginTest() {
    WebDriver driver = new ChromeDriver();

    driver.get("http://localhost:5000/login");
    driver.findElement(By.id("username")).sendKeys("student");
    driver.findElement(By.id("password")).sendKeys("test123");
    driver.findElement(By.cssSelector("button[type=submit]")).click();

    assertTrue(driver.getCurrentUrl().contains("/profile"));

    driver.quit();
}
```

What matters here:

The fields are found by id.

The button is found with a CSS selector.

The assertion is at the end.

`quit()` closes the browser after the test.

Later, setup and cleanup should be moved into `setUp` and `tearDown` methods.

## What Are Unit Tests?

A unit test checks a small unit of logic without a browser, Appium, or an external system.

Unit tests are fast. They usually run in milliseconds or seconds.

They are isolated. They do not require UI, network, database, or a real device.

They are cheap. They are easy to maintain and quick to debug.

For a QA automation engineer, unit tests are not a replacement for UI tests. They are a companion tool. They can verify utilities, data formatting, validation, parsing, API clients, and helper logic.

## Why Unit Tests Matter in a Test Project

The automated test framework itself can also break. It should be tested too.

Useful targets for unit testing include:

TestUtils: username generation, email generation, random data.

ApiUtils: request body generation and response processing.

Page Objects: some logic can be extracted and tested separately.

Parsers and mappers: JSON, dates, money, statuses.

The point is simple: do not wait until one helper bug breaks 50 UI tests. Catch the issue with a small unit test first.

## A Very Simple Unit Test

JUnit can test ordinary Java code without a browser. This makes it useful for explaining assertion logic.

Example:

```java
public class PriceCalculator {
    public static double total(double price, int count) {
        return price * count;
    }
}

@Test
void totalShouldMultiplyPriceByCount() {
    double result = PriceCalculator.total(10.0, 3);

    assertEquals(30.0, result);
}
```

This test follows the Arrange, Act, Assert pattern.

Arrange: prepare the data.

Act: call the method.

Assert: check the result.

There is no WebDriver and no flaky UI.

## A Unit Test for a Helper Class

This is a good example for QA engineers: test a utility that is used in UI or API tests.

Example:

```java
public class TestUtils {
    public static String generateUsername(String prefix) {
        return prefix + "_" + System.currentTimeMillis();
    }
}

@Test
void usernameShouldStartWithPrefix() {
    String username = TestUtils.generateUsername("student");

    assertTrue(username.startsWith("student_"));
    assertFalse(username.isBlank());
}
```

Why this is useful:

It checks logic without UI.

It catches mistakes in test code quickly.

It helps students learn assertions.

It prepares the foundation for Page Objects and API helpers.

## Test Architecture

The goal of test architecture is to make the test read like a scenario while hiding technical details in separate layers.

A simple structure may include:

1. Test  
   The scenario and assertions.

2. Page Object  
   Actions and elements for a page or screen.

3. Utils / API  
   Test data, preparation, and helper logic.

4. Driver Factory  
   Browser or driver creation.

Do not turn a test into a long wall of `findElement` calls. The larger the project becomes, the more important Page Object Model and code reuse become.

## Common Mistakes

It is easier to prevent these mistakes during a course than to fix flaky tests later.

Common problems include:

1. Thread.sleep  
   Replace it with explicit waits.

2. Fragile XPath  
   Do not depend on `div[3]` and absolute paths.

3. Too much dependence on UI  
   Move some checks to API or unit tests.

4. No architecture  
   Use Page Objects, Utils, and Driver Factory.

5. No test data strategy  
   Create data through API, fixtures, or stable preparation logic.

6. No CI  
   Tests quickly stop being useful when nobody runs them regularly.

## CI/CD

Without remote execution, automated tests often remain a local toy.

A basic CI/CD flow looks like this:

1. Pull Request  
   The code changes.

2. Build  
   The project is compiled and assembled.

3. Tests  
   Unit, API, and UI tests are executed.

4. Report  
   The team sees the result.

Common tools include GitHub Actions, Jenkins, and GitLab CI.

For UI tests, it is important to think separately about browsers, drivers, artifacts, screenshots, videos, and reports.

## What to Automate First

Start not with the number of tests, but with the value of the scenarios.

Good first candidates:

Login: used often and critical for most products.

Checkout or payment: directly affects money.

API scenarios: fast and stable checks.

Critical user flow: the path from user entry to the final result.

Smoke suite: the smallest useful set of checks before release.

Regression core: the parts of the product that break most often.

## Conclusion

Automation is programming, not just recording clicks.

Good automated tests need stable locators such as data-testid, id, and accessibility id.

They need clear architecture: Page Object, Utils, and Driver Factory.

They need the right testing level: Unit, API, or UI depending on the task.

They need regular execution locally and in CI/CD.

Good tests are easy to read, easy to maintain, and easy to run.
