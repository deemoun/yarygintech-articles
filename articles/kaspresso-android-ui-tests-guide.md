---
title: "Kaspresso: Android UI Tests Without Espresso Pain"
description: "A practical guide to using Kaspresso for Android UI automation, when to pick it over Espresso, Appium, or Robolectric, and how to actually write and run tests."
pubDate: 2026-06-11
tags: ["android", "qa", "testing", "automation", "kaspresso", "espresso", "appium", "kotlin"]
---
Kaspresso is one of those Android testing tools that sounds like another wrapper around Espresso.

And technically, yes, it is built on top of Espresso and UI Automator.

**But that is also the point.**

Espresso is powerful, but writing real test automation with plain Espresso can get annoying very fast. You end up with long matchers, repeated waits, flaky clicks, weak logs, screenshots bolted on later, and test code that looks like someone lost a fight with `onView()`.

Kaspresso tries to fix that.

It gives you a cleaner Kotlin DSL, better logging, steps, screenshots, flaky-safety mechanisms, device controls, and access to both app UI and system UI. So you still run native Android instrumentation tests, but the test code becomes more readable and more useful for actual QA work.

Not magic. Not a silver bullet. But definitely less painful than raw Espresso.

---

## What Kaspresso actually is

Kaspresso is a native Android UI testing framework.

Under the hood it uses:

| Piece | What it does |
|---|---|
| Espresso | Main interaction with views inside your app |
| UI Automator | Interaction with system UI and other apps |
| Kakao | Kotlin DSL over Espresso |
| Kautomator | Kotlin DSL over UI Automator |
| Kaspresso | Test structure, steps, logging, screenshots, flaky safety, device actions |

So you can think of it like this:

```text
Espresso = raw engine
Kakao = nicer syntax for Espresso
UI Automator = system/outside-app control
Kautomator = nicer syntax for UI Automator
Kaspresso = test framework around all of that
```

That is why Kaspresso feels closer to real QA automation than plain Espresso.

Espresso gives you the tools. Kaspresso gives you a workflow.

---

## When I would use Kaspresso

I would use Kaspresso when I test an Android app and I want native UI tests that are readable, stable enough for CI, and not completely horrible to maintain.

Good use cases:

| Use case | Why Kaspresso fits |
|---|---|
| Login flows | Easy screen objects, steps, assertions |
| Checkout / order flows | Better logs and screenshots when something breaks |
| App navigation | Clean DSL makes tests readable |
| Permission dialogs | UI Automator/Kautomator can help with system UI |
| Flaky UI flows | Kaspresso has flaky-safety tools |
| Regression smoke tests | Good fit for CI on emulator/device |
| Native Android apps | It runs close to the app and Android framework |

Bad use cases:

| Use case | Better tool |
|---|---|
| Testing iOS and Android from one framework | Appium |
| Fast JVM-only unit tests | Robolectric or plain JUnit |
| Business logic without UI | JUnit/MockK |
| Testing backend/API only | RestAssured/Postman/Newman/etc. |
| WebView-heavy cross-platform app | Maybe Appium or Playwright, depends on the app |

Kaspresso is not trying to replace every testing tool.

It is for Android UI/instrumentation tests.

---

## Why use Kaspresso instead of plain Espresso?

Plain Espresso works. I am not saying it is bad.

But if you have written enough Espresso tests, you know the usual pain:

- noisy `onView(withId(...))` code everywhere
- repeated matchers
- weak reporting
- screenshots are not built into your normal flow
- logs are not friendly for debugging
- flaky waits become your problem
- test readability gets worse as the suite grows

Kaspresso improves the working experience.

| Reason | Espresso | Kaspresso |
|---|---|---|
| Syntax | Raw `onView()` / matchers | Kotlin DSL via Kakao |
| Readability | Can become noisy | Screen objects are cleaner |
| Logging | Basic | Better structured logs |
| Screenshots | Manual setup | Built-in support |
| Flaky handling | Mostly your job | Flaky-safety helpers |
| System UI | Need UI Automator separately | UI Automator is part of the ecosystem |
| Test steps | Manual convention | Built into test style |
| QA friendliness | More developer-style | More test-automation-style |

Example of Espresso style:

```kotlin
onView(withId(R.id.email))
    .perform(typeText("demo@test.com"), closeSoftKeyboard())

onView(withId(R.id.password))
    .perform(typeText("password"), closeSoftKeyboard())

onView(withId(R.id.loginButton))
    .perform(click())

onView(withId(R.id.homeTitle))
    .check(matches(isDisplayed()))
```

Same idea with Kaspresso/Kakao screen objects:

```kotlin
LoginScreen {
    email.replaceText("demo@test.com")
    password.replaceText("password")
    loginButton.click()
}

HomeScreen {
    title.isDisplayed()
}
```

This is not just prettier. It matters when you have 100+ tests and need to understand failures fast.

---

## Why use Kaspresso instead of Appium?

Appium is useful when you need cross-platform automation or black-box testing from outside the app.

But for native Android apps, Appium is often heavier than you need.

| Reason | Appium | Kaspresso |
|---|---|---|
| Test type | Black-box, external driver | Native Android instrumentation |
| Speed | Usually slower | Usually faster |
| Setup | Appium server, capabilities, drivers | Android test dependency + Gradle |
| Language | Many languages | Kotlin/Java Android test code |
| Cross-platform | Android + iOS | Android only |
| Access to app internals | Limited | Better, because tests run inside Android instrumentation |
| CI complexity | Higher | Lower for Android-only projects |
| Best for | Cross-platform E2E | Native Android regression/UI tests |

My practical opinion:

Use Appium if the company wants one automation stack for Android and iOS.

Use Kaspresso if you are serious about Android and want tests that Android developers can also run and debug.

For Android-only UI tests, Kaspresso is usually the more sane choice.

---

## Why use Kaspresso instead of Robolectric?

Robolectric is not the same kind of tool.

Robolectric runs Android-ish tests on the JVM. It is fast and useful for testing logic that touches Android classes without launching a real emulator.

Kaspresso runs on a real device or emulator as an instrumentation test.

| Reason | Robolectric | Kaspresso |
|---|---|---|
| Runs on | JVM | Emulator/real device |
| Speed | Fast | Slower |
| UI realism | Limited | Real Android UI |
| Good for | ViewModel-ish logic, Android framework interactions | Real user flows |
| System dialogs | Not the real thing | Can interact with real system UI |
| Flakiness | Lower, because no real device | Higher risk, but more realistic |
| CI cost | Cheap | More expensive |
| Confidence | Medium | Higher for actual UI behavior |

Use Robolectric for fast feedback.

Use Kaspresso when you need to know the app actually works on Android.

They can live together.

A healthy Android test pyramid could look like this:

```text
Many unit tests
Some Robolectric tests
Some Kaspresso UI tests
A few Appium E2E tests only if cross-platform is needed
```

---

## Installing Kaspresso

In your app module, add the dependency to `build.gradle.kts`.

```kotlin
dependencies {
    androidTestImplementation("com.kaspersky.android-components:kaspresso:1.6.1")
}
```

Make sure you have an instrumentation runner:

```kotlin
android {
    defaultConfig {
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }
}
```

You will also usually have AndroidX test dependencies already:

```kotlin
dependencies {
    androidTestImplementation("androidx.test.ext:junit:1.2.1")
    androidTestImplementation("androidx.test:runner:1.6.2")
    androidTestImplementation("androidx.test:rules:1.6.1")
}
```

Version numbers change. Check Maven Central before copying this into a real project six months from now.

---

## Basic project structure

Kaspresso tests live in:

```text
app/src/androidTest/java/your/package/name/
```

A simple structure:

```text
androidTest/
└── java/
    └── com/example/app/
        ├── screens/
        │   ├── LoginScreen.kt
        │   └── HomeScreen.kt
        └── tests/
            └── LoginTest.kt
```

The important idea: keep UI locators in screen classes, not directly inside every test.

Bad:

```kotlin
@Test
fun loginWorks() {
    onView(withId(R.id.email)).perform(typeText("demo@test.com"))
    onView(withId(R.id.password)).perform(typeText("password"))
    onView(withId(R.id.loginButton)).perform(click())
    onView(withId(R.id.homeTitle)).check(matches(isDisplayed()))
}
```

Better:

```kotlin
@Test
fun loginWorks() = run {
    step("Enter login credentials") {
        LoginScreen {
            email.replaceText("demo@test.com")
            password.replaceText("password")
        }
    }

    step("Tap login") {
        LoginScreen {
            loginButton.click()
        }
    }

    step("Check home screen") {
        HomeScreen {
            title.isDisplayed()
        }
    }
}
```

That is the whole point: tests should read like actions, not like matcher soup.

---

## Creating a screen object

Example `LoginScreen.kt`:

```kotlin
package com.example.app.screens

import com.example.app.R
import com.example.app.ui.LoginActivity
import com.kaspersky.kaspresso.screens.KScreen
import io.github.kakaocup.kakao.edit.KEditText
import io.github.kakaocup.kakao.text.KButton

object LoginScreen : KScreen<LoginScreen>() {

    override val layoutId: Int = R.layout.activity_login
    override val viewClass: Class<*> = LoginActivity::class.java

    val email = KEditText { withId(R.id.email) }
    val password = KEditText { withId(R.id.password) }
    val loginButton = KButton { withId(R.id.loginButton) }
}
```

Example `HomeScreen.kt`:

```kotlin
package com.example.app.screens

import com.example.app.R
import com.example.app.ui.HomeActivity
import com.kaspersky.kaspresso.screens.KScreen
import io.github.kakaocup.kakao.text.KTextView

object HomeScreen : KScreen<HomeScreen>() {

    override val layoutId: Int = R.layout.activity_home
    override val viewClass: Class<*> = HomeActivity::class.java

    val title = KTextView { withId(R.id.homeTitle) }
}
```

The exact imports can differ depending on your version and app setup, but the pattern is what matters.

Each screen class describes what exists on the screen.

The test describes what the user does.

Do not mix those two unless you want a maintenance mess later.

---

## Writing a Kaspresso test

Example `LoginTest.kt`:

```kotlin
package com.example.app.tests

import androidx.test.ext.junit.rules.ActivityScenarioRule
import androidx.test.ext.junit.runners.AndroidJUnit4
import com.example.app.screens.HomeScreen
import com.example.app.screens.LoginScreen
import com.example.app.ui.LoginActivity
import com.kaspersky.kaspresso.testcases.api.testcase.TestCase
import org.junit.Rule
import org.junit.Test
import org.junit.runner.RunWith

@RunWith(AndroidJUnit4::class)
class LoginTest : TestCase() {

    @get:Rule
    val activityRule = ActivityScenarioRule(LoginActivity::class.java)

    @Test
    fun userCanLogin() = run {
        step("Enter credentials") {
            LoginScreen {
                email.replaceText("demo@test.com")
                password.replaceText("password")
            }
        }

        step("Submit login form") {
            LoginScreen {
                loginButton.click()
            }
        }

        step("Verify home screen is opened") {
            HomeScreen {
                title.isDisplayed()
            }
        }
    }
}
```

This is the style I like for UI tests.

Not too abstract. Not too clever. Not a page object religion.

Just enough structure so the test is readable.

---

## Common commands

Here are the commands you will actually use.

| Task | Command |
|---|---|
| Run all connected Android tests | `./gradlew connectedDebugAndroidTest` |
| Run tests for a specific build variant | `./gradlew connectedFreeDebugAndroidTest` |
| Run one test class | `./gradlew connectedDebugAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=com.example.app.tests.LoginTest` |
| Run one test method | `./gradlew connectedDebugAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=com.example.app.tests.LoginTest#userCanLogin` |
| Install debug APK | `./gradlew installDebug` |
| Install test APK | `./gradlew installDebugAndroidTest` |
| List connected devices | `adb devices` |
| Clear app data | `adb shell pm clear com.example.app` |
| Run instrumentation manually | `adb shell am instrument -w com.example.app.test/androidx.test.runner.AndroidJUnitRunner` |
| Run instrumentation with one class | `adb shell am instrument -w -e class com.example.app.tests.LoginTest com.example.app.test/androidx.test.runner.AndroidJUnitRunner` |
| Pull test artifacts from device | `adb pull /sdcard/Android/data/com.example.app/files/ ./test-artifacts` |
| Check device logs | `adb logcat` |
| Run on CI emulator | Start emulator, then run `./gradlew connectedDebugAndroidTest` |

The Gradle command is what you will use most of the time.

The `adb shell am instrument` command is useful when debugging or when your CI setup is custom.

---

## Common Kaspresso/Kakao actions

This is the stuff you use every day.

| Action | Example |
|---|---|
| Tap a button | `loginButton.click()` |
| Type text | `email.typeText("demo@test.com")` |
| Replace text | `email.replaceText("demo@test.com")` |
| Clear text | `email.clearText()` |
| Check visible | `title.isDisplayed()` |
| Check not visible | `errorText.isNotDisplayed()` |
| Check text | `title.hasText("Welcome")` |
| Check hint | `email.hasHint("Email")` |
| Scroll to element | `someView.scrollTo()` |
| Assert enabled | `loginButton.isEnabled()` |
| Assert disabled | `loginButton.isDisabled()` |
| Assert checked | `checkbox.isChecked()` |
| Assert not checked | `checkbox.isNotChecked()` |

Example:

```kotlin
LoginScreen {
    email.replaceText("demo@test.com")
    password.replaceText("password")
    loginButton.click()
}
```

Simple. Readable. No ceremony.

---

## Using steps properly

Kaspresso steps are not just decoration.

They make logs readable.

Bad:

```kotlin
@Test
fun checkoutWorks() = run {
    CatalogScreen { item.click() }
    CartScreen { checkout.click() }
    PaymentScreen { cardNumber.replaceText("4111111111111111") }
    PaymentScreen { pay.click() }
    SuccessScreen { title.isDisplayed() }
}
```

Better:

```kotlin
@Test
fun checkoutWorks() = run {
    step("Open product from catalog") {
        CatalogScreen {
            item.click()
        }
    }

    step("Go to checkout") {
        CartScreen {
            checkout.click()
        }
    }

    step("Enter payment details") {
        PaymentScreen {
            cardNumber.replaceText("4111111111111111")
            pay.click()
        }
    }

    step("Verify successful order") {
        SuccessScreen {
            title.isDisplayed()
        }
    }
}
```

When this fails in CI, you do not want to guess where it died.

Steps give you a readable story.

---

## Using Kautomator for system UI

Sometimes your test leaves the app.

Examples:

- Android permission dialogs
- notification shade
- system settings
- another app
- file picker
- camera/gallery picker

This is where UI Automator comes in. Kaspresso includes Kautomator for a nicer DSL.

Example idea:

```kotlin
import com.kaspersky.components.kautomator.component.common.views.UiView

object PermissionDialog {
    val allowButton = UiView { withText("Allow") }
}
```

Then in a test:

```kotlin
step("Allow notification permission") {
    PermissionDialog.allowButton.click()
}
```

Text can change by Android version and language, so be careful. System UI tests are always more fragile than normal app UI tests.

If you can avoid system dialogs in most tests, avoid them.

For one smoke test, fine.

For every test, pain.

---

## Handling flaky tests

Kaspresso has flaky-safety mechanisms that retry actions/assertions for a period of time instead of instantly failing on small UI timing issues.

That helps with real Android UI automation because apps are not perfectly synchronous.

But here is the important part:

Do not use flaky safety as an excuse to write bad tests.

If the app needs 10 seconds to show a button because the backend is slow, fix the test setup. Mock the backend, seed data, disable animations, or control the state.

Flaky safety should protect against small UI timing issues.

It should not hide broken architecture.

Practical anti-flake checklist:

| Problem | Better fix |
|---|---|
| Animation makes click unstable | Disable animations on emulator |
| Backend data is random | Use mock server or seeded test data |
| Test depends on previous test | Clear app data before test |
| Same account reused by parallel tests | Generate isolated test users |
| System dialog appears randomly | Handle it in setup |
| Element selector is weak | Use stable resource IDs |
| Test waits for text from network | Mock or control the response |

Useful emulator setup:

```bash
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0
```

This alone removes a lot of nonsense.

---

## Recommended test style

Use this simple pattern:

```text
Arrange
Act
Assert
```

For UI tests:

```text
Prepare app state
Open screen
Perform user action
Check result
```

Example:

```kotlin
@Test
fun wrongPasswordShowsError() = run {
    before {
        // clear user/session state if needed
    }.after {
        // cleanup if needed
    }.run {
        step("Open login screen") {
            // activity rule already opened it
        }

        step("Enter invalid credentials") {
            LoginScreen {
                email.replaceText("demo@test.com")
                password.replaceText("wrong-password")
                loginButton.click()
            }
        }

        step("Verify error message") {
            LoginScreen {
                errorText.hasText("Invalid email or password")
            }
        }
    }
}
```

The point is not to make it fancy.

The point is to make it obvious.

A test should be boring to read.

Boring tests are usually good tests.

---

## What to test with Kaspresso

Do not automate everything through the UI.

That is how teams create a slow and flaky monster.

Good Kaspresso test candidates:

| Test | Why |
|---|---|
| Login success | Critical path |
| Login error | Common failure state |
| Main navigation | App health check |
| Checkout/order flow | Business-critical |
| Settings change | Real UI state |
| Permission-dependent flow | Android-specific behavior |
| Offline state | Important user scenario |
| Deep link opening | Android integration point |

Bad Kaspresso test candidates:

| Test | Why not |
|---|---|
| Every validation rule | Better as unit tests |
| Every API error | Better as API/contract tests |
| Every tiny UI label | Too brittle |
| Visual pixel-perfect checks | Use screenshot testing tools |
| Complex backend workflows | Mock or test lower layers |
| Pure ViewModel logic | Unit test it |

The UI suite should be small and valuable.

Not huge and miserable.

---

## Kaspresso vs Espresso vs Appium vs Robolectric

Here is the practical comparison.

| Tool | Best for | Runs on | Main advantage | Main downside |
|---|---|---|---|---|
| Kaspresso | Native Android UI regression tests | Emulator/device | Readable, stable-ish, great Android workflow | Android only |
| Espresso | Native Android UI tests | Emulator/device | Official, powerful, direct | Verbose, less QA-friendly |
| Appium | Cross-platform E2E | Emulator/device/real device | Android + iOS from one stack | Slower, heavier, more moving parts |
| Robolectric | JVM Android-ish tests | Local JVM | Fast, cheap CI feedback | Not real UI/device behavior |

My rule:

| Situation | Pick |
|---|---|
| Android-only UI automation | Kaspresso |
| Small official Android UI test with no framework overhead | Espresso |
| Android + iOS E2E from one suite | Appium |
| Fast tests without emulator | Robolectric |
| Business logic | JUnit |
| API testing | RestAssured |
| Web UI testing | Playwright |

Tools are not religions.

Use the tool that matches the test layer.

---

## CI setup idea

A basic CI flow:

```yaml
name: Android UI Tests

on:
  pull_request:
  push:
    branches: [ main ]

jobs:
  ui-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Run Android tests
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 35
          target: google_apis
          arch: x86_64
          script: ./gradlew connectedDebugAndroidTest
```

This is enough to start.

Later you can add:

- screenshots as artifacts
- Allure report
- test retries
- sharding
- managed devices
- Firebase Test Lab
- different Android API levels

But do not start with a monster setup.

Make one reliable test run first.

Then expand.

---

## Common mistakes

| Mistake | Why it hurts |
|---|---|
| Putting all locators inside test methods | Test becomes impossible to maintain |
| Testing too much through UI | Suite becomes slow and flaky |
| Depending on real backend | Random failures |
| Reusing one test account everywhere | Parallel runs break |
| Ignoring test data setup | Tests become order-dependent |
| Using text selectors everywhere | Breaks with copy changes/localization |
| Not disabling animations | Random UI timing failures |
| No screenshots/logs in CI | Debugging becomes archaeology |
| Making page objects too abstract | Nobody understands the test anymore |

The worst automation suites are not bad because of the tool.

They are bad because nobody controlled the scope.

Kaspresso helps, but it will not save a chaotic test strategy.

---

## My practical setup recommendation

If I were adding Kaspresso to a real Android project, I would start like this:

| Step | What I would do |
|---|---|
| 1 | Add Kaspresso dependency |
| 2 | Create 2-3 screen objects |
| 3 | Write one happy-path smoke test |
| 4 | Add one negative test |
| 5 | Run locally from Gradle |
| 6 | Run in CI on one emulator |
| 7 | Save screenshots/logs as artifacts |
| 8 | Add only business-critical flows |

Start small.

One stable test is better than twenty flaky ones.

For the first batch, I would automate:

| Priority | Test |
|---|---|
| P0 | App launches |
| P0 | User can log in |
| P0 | Main screen opens |
| P1 | Wrong password shows error |
| P1 | User can log out |
| P1 | One important business flow works |

After that, stop and evaluate.

Are the tests stable?
Are failures easy to debug?
Is CI too slow?
Are selectors good?
Do developers actually run them?

If the answer is no, fix the foundation before writing more tests.

---

## Final take

Kaspresso is what I would use when plain Espresso starts feeling too raw, but Appium feels too heavy.

It keeps you inside the Android ecosystem, gives you readable Kotlin test code, and adds the kind of things QA engineers actually need: steps, logs, screenshots, flaky handling, and system UI support.

It will not magically make bad tests good.

But if your Android app needs a real UI regression suite, Kaspresso is a very reasonable choice.

My honest recommendation:

Use Kaspresso for a small number of high-value Android UI tests.

Use unit tests and Robolectric for the lower layers.

Use Appium only when cross-platform coverage is actually required.

That setup gives you coverage without turning the test suite into a slow circus.