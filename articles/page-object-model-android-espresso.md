---
title: "Page Object Model in Android UI Testing: A Simple Espresso Framework"
description: "A practical guide to cleaning up Android Espresso UI tests with a basic Page Object Model structure, reusable actions, and readable test classes."
pubDate: 2026-05-28
tags: ["Android", "Espresso", "UI Testing", "Page Object Model", "QA Automation", "Android Testing", "Java"]
draft: false
---
Page Object Model is not a new idea in test automation. Some teams use it everywhere. Some teams avoid it because it feels like extra architecture for simple tests.

Both sides are right in some way.

If your test project has three small checks, you probably do not need a framework around it. But if your UI test suite keeps growing, repeated Espresso calls quickly become annoying. You start copying the same `onView()`, `withId()`, `perform(click())`, and `check(matches(...))` lines across many tests. Then one screen changes, one locator changes, or one action needs better synchronization — and suddenly you are editing the same logic in ten different places.

That is where Page Object Model starts to make sense.

In this article, we will refactor a simple Android Espresso test setup into a basic Page Object style structure. Nothing over-engineered. Just enough structure to make the tests easier to read and maintain.

## What is Page Object Model?

The main idea is simple: each screen, page, or major UI area gets its own class.

That class knows where its UI elements are and exposes actions that can be performed on that screen. Instead of writing low-level Espresso code directly inside every test, the test calls readable methods.

So instead of this:

```java
onView(withId(R.id.navigation_dashboard))
    .perform(click())
    .check(matches(isDisplayed()));
```

We can move the reusable logic into a page or base class and make the test read more like this:

```java
DashboardPage.clickDashboardButton();
```

The test becomes easier to read. The low-level implementation is still there, but it is no longer scattered everywhere.

## Why use it?

Page Object Model helps when your test suite starts becoming repetitive.

It gives you:

- cleaner test classes;
- reusable UI actions;
- one place to update locators;
- easier onboarding for new team members;
- better separation between test logic and UI interaction details.

The important thing is not to turn POM into a religion. For a tiny demo project, it may look like unnecessary work. For a real test suite with dozens or hundreds of UI tests, it can save a lot of time.

## The original Espresso test

Let’s say we have a simple Android app with bottom navigation. It has three main sections:

- Home
- Dashboard
- Notifications

The direct Espresso version might look like this:

```java
@Test
public void clickButtonHome() {
    onView(withId(R.id.navigation_home))
            .perform(click())
            .check(matches(isDisplayed()));
}

@Test
public void clickButtonDashboard() {
    onView(withId(R.id.navigation_dashboard))
            .perform(click())
            .check(matches(isDisplayed()));
}

@Test
public void clickButtonNotification() {
    onView(withId(R.id.navigation_notifications))
            .perform(click())
            .check(matches(isDisplayed()));
}
```

This is not bad code. It is simple and readable.

The problem starts later.

If the same pattern is repeated across 50 or 300 tests, the project becomes harder to maintain. Every test knows too much about implementation details. Every test directly touches resource IDs. Every test repeats the same interaction code.

So let’s move this into a small framework structure.

## Project setup

For a modern Android project, use AndroidX Test dependencies instead of the old support library dependencies.

A basic setup can look like this:

```gradle
dependencies {
    androidTestImplementation "androidx.test:runner:1.7.0"
    androidTestImplementation "androidx.test:rules:1.7.0"
    androidTestImplementation "androidx.test.ext:junit:1.3.0"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.7.0"
}
```

And make sure the test runner is configured:

```gradle
android {
    defaultConfig {
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
}
```

Espresso tests live in:

```text
src/androidTest/java
```

These are instrumented UI tests. They run on a real device or emulator, not only on your local JVM.

## Basic framework structure

For this simple example, we can create the following classes:

```text
androidTest/java/your/package/
├── base/
│   └── EspressoBaseTest.java
├── pages/
│   ├── DashboardPage.java
│   ├── HomePage.java
│   └── NotificationsPage.java
├── util/
│   └── TestLog.java
└── TestPlan.java
```

This is still very small. No heavy framework. No magic.

Each class has a clear job:

`EspressoBaseTest` contains reusable Espresso actions.

`HomePage`, `DashboardPage`, and `NotificationsPage` represent app screens or UI areas.

`TestLog` contains small helper logic.

`TestPlan` contains the actual test cases.

## Base class for common Espresso actions

First, let’s create a simple base class:

```java
package your.package.base;

import static androidx.test.espresso.Espresso.onView;
import static androidx.test.espresso.action.ViewActions.click;
import static androidx.test.espresso.assertion.ViewAssertions.matches;
import static androidx.test.espresso.matcher.ViewMatchers.isDisplayed;
import static androidx.test.espresso.matcher.ViewMatchers.withId;

public class EspressoBaseTest {

    protected static void clickButton(int resourceId) {
        onView(withId(resourceId))
                .perform(click())
                .check(matches(isDisplayed()));
    }
}
```

This method takes a resource ID, finds the view, clicks it, and checks that it is displayed.

It is still the same Espresso code. We just moved it into one reusable place.

## Dashboard page

Now we can create a page class for Dashboard:

```java
package your.package.pages;

import your.package.R;
import your.package.base.EspressoBaseTest;

public class DashboardPage extends EspressoBaseTest {

    private static int dashboardButton() {
        return R.id.navigation_dashboard;
    }

    public static void clickDashboardButton() {
        clickButton(dashboardButton());
    }
}
```

This class now owns the Dashboard-related UI locator and action.

The test does not need to know the exact resource ID anymore.

## Home page

The Home page follows the same idea:

```java
package your.package.pages;

import your.package.R;
import your.package.base.EspressoBaseTest;

public class HomePage extends EspressoBaseTest {

    private static int homeButton() {
        return R.id.navigation_home;
    }

    public static void clickHomeButton() {
        clickButton(homeButton());
    }
}
```

## Notifications page

And the Notifications page:

```java
package your.package.pages;

import your.package.R;
import your.package.base.EspressoBaseTest;

public class NotificationsPage extends EspressoBaseTest {

    private static int notificationsButton() {
        return R.id.navigation_notifications;
    }

    public static void clickNotificationsButton() {
        clickButton(notificationsButton());
    }
}
```

Again, nothing complicated. We are not trying to impress anyone with architecture. We are just moving repeated details into classes where they belong.

## Simple logging utility

A small helper class can be useful when you want to print debug information during test execution:

```java
package your.package.util;

import android.util.Log;

public class TestLog {

    private static final boolean LOG_ENABLED = true;
    private static final String TAG = "Espresso Test";

    public static void log(String message) {
        if (LOG_ENABLED) {
            Log.v(TAG, message);
        }
    }
}
```

In a real project, this could later be expanded with screenshots, test artifacts, or custom reporting logic.

For now, it is just a simple example.

## Test plan

Now the test class becomes much cleaner:

```java
package your.package;

import static your.package.pages.DashboardPage.clickDashboardButton;
import static your.package.pages.HomePage.clickHomeButton;
import static your.package.pages.NotificationsPage.clickNotificationsButton;

import androidx.test.ext.junit.rules.ActivityScenarioRule;
import androidx.test.ext.junit.runners.AndroidJUnit4;

import your.package.util.TestLog;

import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(AndroidJUnit4.class)
public class TestPlan {

    @Rule
    public ActivityScenarioRule<MainActivity> activityRule =
            new ActivityScenarioRule<>(MainActivity.class);

    @Test
    public void clickDashboardPageButton() {
        clickDashboardButton();
        TestLog.log("Dashboard button pressed");
    }

    @Test
    public void clickHomePageButton() {
        clickHomeButton();
        TestLog.log("Home button pressed");
    }

    @Test
    public void clickNotificationsPageButton() {
        clickNotificationsButton();
        TestLog.log("Notifications button pressed");
    }
}
```

Now the test class describes the scenario instead of exposing every low-level Espresso call.

That is the main point.

## What changed?

Compared to the original version:

1. The click implementation is hidden inside `clickButton()`.
2. Resource IDs are stored inside page classes.
3. Tests use readable page-level methods.
4. The structure is easier to extend later.
5. If a locator changes, we update the page class instead of editing every test.

This is the kind of refactoring that looks small at first, but becomes valuable when the test suite grows.

## Should every project use Page Object Model?

**No.**

If the project is tiny, direct Espresso tests may be enough. There is no reason to build a framework just to click three buttons.

But when the test suite grows, POM gives you a cleaner way to organize UI automation. It does not solve all problems, but it reduces duplication and makes the project easier to maintain.

The main rule is simple:

Use Page Object Model when it makes your tests easier to read and cheaper to change.

Do not use it just because everyone says “framework” sounds more professional.

## Final thoughts

This is a very basic Page Object Model example for Android Espresso tests. In a real project, you would probably add better naming, waits or idling resources, screenshot handling, reporting, test data management, and maybe split common actions by screen type.

But the foundation stays the same.

Keep test cases readable. Keep UI details in page classes. Keep repeated Espresso code in reusable methods.

That alone already makes your Android UI tests more maintainable.