---
title: "Mockito: A Practical Guide to Mock Objects in Java Tests"
description: "A modern, simple guide to Mockito: what mock objects are, why they matter, how to add Mockito to a Java or Android project, and how to write clean unit tests without overcomplicating things."
pubDate: 2026-05-28
tags: ["Java", "Mockito", "Unit Testing", "Test Automation", "JUnit 5", "Android", "QA Automation"]
draft: false
---
Let's talk about Mockito - a popular and one of the most useful tools when you want to write clean unit tests without dragging the whole application into the test.

## What Is a Mock Object?

A mock object is a fake version of a real object.

It behaves the way we tell it to behave, but it does not execute the real implementation.

That is the whole point.

Instead of creating a real database, real API client, real repository, real payment service, or real Android dependency, we create a controlled test double and tell it:

> When this method is called, return this value.

This makes the test faster, simpler, and easier to control.

## Why Do We Need Mocks?

Mocks are useful when the real dependency is inconvenient for a unit test.

Typical examples:

- the real object talks to a database;
- the real object sends network requests;
- the real object depends on Android framework classes;
- the real object is slow;
- the real object is unstable;
- the real object is not finished yet;
- the dependency is not the thing we actually want to test.

A good unit test should focus on one piece of logic.

If I am testing a `TransferService`, I do not want my test to depend on a real server, real database, real login session, or real bank backend. I want to test the logic inside `TransferService`.

Mockito helps with exactly that.

## Mockito in Simple Words

Mockito is a Java mocking framework.

It allows you to:

- create mock objects;
- define what mocked methods should return;
- verify that methods were called;
- test logic without depending on real external systems.

The core idea usually looks like this:

```java
UserRepository repository = Mockito.mock(UserRepository.class);

Mockito.when(repository.findUserNameById(1))
        .thenReturn("Dmitry");
```

Now the mocked repository will return `"Dmitry"` when `findUserNameById(1)` is called.

No database. No backend. No setup hell.

## Modern Mockito Dependency

For a regular Java project with Gradle, you can add Mockito like this:

```gradle
dependencies {
    testImplementation "org.mockito:mockito-core:5.23.0"
    testImplementation "org.junit.jupiter:junit-jupiter:5.11.4"
}
```

For JUnit 5 integration, also add:

```gradle
dependencies {
    testImplementation "org.mockito:mockito-junit-jupiter:5.23.0"
}
```

And make sure JUnit 5 is enabled:

```gradle
test {
    useJUnitPlatform()
}
```

For Maven:

```xml
<dependencies>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.23.0</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>5.23.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

If you are working inside an Android project, you normally put Mockito into `testImplementation`, not `androidTestImplementation`, when you are writing local JVM unit tests.

```gradle
dependencies {
    testImplementation "org.mockito:mockito-core:5.23.0"
    testImplementation "org.mockito:mockito-junit-jupiter:5.23.0"
}
```

Instrumented Android UI tests are a different topic. Mockito is mostly a unit testing tool. For UI tests you will usually think about Espresso, UIAutomator, Appium, fake servers, test data, and dependency injection.

## A Better Example Than Adding Two Numbers

The old version of this article used a calculator-like example:

```java
add(10, 20)
```

That is fine for showing syntax, but it does not really explain why Mockito is useful.

Mockito becomes useful when we have one class depending on another class.

Let’s use a simple service.

```java
public interface UserRepository {
    String findUserNameById(int userId);
}
```

And now a service that depends on this repository:

```java
public class GreetingService {

    private final UserRepository userRepository;

    public GreetingService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public String greetUser(int userId) {
        String userName = userRepository.findUserNameById(userId);
        return "Hello, " + userName + "!";
    }
}
```

In real life, `UserRepository` might read from a database or call an API.

But the `GreetingService` does not need a real database in a unit test. We only want to check that it builds the correct greeting.

## Writing a Simple Mockito Test

Here is a clean JUnit 5 test:

```java
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.junit.jupiter.api.Assertions.assertEquals;

class GreetingServiceTest {

    @Test
    void shouldReturnGreetingForUser() {
        UserRepository repository = Mockito.mock(UserRepository.class);

        Mockito.when(repository.findUserNameById(1))
                .thenReturn("Dmitry");

        GreetingService greetingService = new GreetingService(repository);

        String result = greetingService.greetUser(1);

        assertEquals("Hello, Dmitry!", result);
    }
}
```

What happens here?

1. We create a mock `UserRepository`.
2. We tell Mockito what to return when `findUserNameById(1)` is called.
3. We inject the mock into `GreetingService`.
4. We call the method under test.
5. We check the result.

This is the standard shape of many unit tests.

## Using `@Mock` and `@ExtendWith`

Mockito can also create mocks automatically with annotations.

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class GreetingServiceTest {

    @Mock
    private UserRepository repository;

    @Test
    void shouldReturnGreetingForUser() {
        when(repository.findUserNameById(1))
                .thenReturn("Dmitry");

        GreetingService greetingService = new GreetingService(repository);

        String result = greetingService.greetUser(1);

        assertEquals("Hello, Dmitry!", result);
    }
}
```

This version is cleaner when your test has several mocks.

Instead of manually writing:

```java
UserRepository repository = Mockito.mock(UserRepository.class);
```

Mockito creates the mock for you.

## Verifying Calls

Sometimes you do not only care about the returned value.

Sometimes you want to check that a dependency was called.

Example:

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.verify;

class NotificationServiceTest {

    @Test
    void shouldSendNotification() {
        EmailClient emailClient = mock(EmailClient.class);

        NotificationService service = new NotificationService(emailClient);

        service.notifyUser("user@example.com");

        verify(emailClient).send("user@example.com", "Your notification is ready");
    }
}
```

`verify()` checks that a method was actually called.

This is useful, but do not overuse it. If every test verifies ten internal calls, your tests become too fragile. You change implementation details, and suddenly half of the tests break even though the behavior is still correct.

Verify important interactions, not every tiny implementation step.

## Mock vs Stub vs Fake

People love to argue about test terminology.

In practical work, here is the simple version:

A **mock** is an object where you usually verify interactions.

A **stub** returns predefined data.

A **fake** is a simplified working implementation.

Example:

- mock: “Check that `emailClient.send()` was called.”
- stub: “When `repository.findUser()` is called, return this user.”
- fake: “Use an in-memory repository instead of a real database.”

Mockito can be used for both mocking and stubbing, even if people use the word “mock” for everything.

## What Mockito Should Not Replace

Mockito is not a replacement for all testing.

It does not replace:

- integration tests;
- API tests;
- UI tests;
- end-to-end tests;
- manual exploratory testing;
- common sense.

This matters even more now, with AI-generated code and AI-assisted testing everywhere.

AI can generate test templates. It can suggest edge cases. It can even write Mockito tests quickly.

But AI still does not magically understand your product risk, business logic, weird production behavior, or the way users actually break things.

Mockito is a tool. AI is a tool. Neither replaces a tester who understands the system.

## A Common Mistake: Mocking the Class You Are Testing

This is one of the biggest beginner mistakes.

Bad idea:

```java
GreetingService service = Mockito.mock(GreetingService.class);

Mockito.when(service.greetUser(1))
        .thenReturn("Hello, Dmitry!");

assertEquals("Hello, Dmitry!", service.greetUser(1));
```

This test proves almost nothing.

You are mocking the class you are supposed to test. You told the mock what to return, then checked that it returned it.

That is not useful testing. That is testing Mockito itself.

Usually, you should mock dependencies, not the class under test.

Better:

```java
UserRepository repository = Mockito.mock(UserRepository.class);

Mockito.when(repository.findUserNameById(1))
        .thenReturn("Dmitry");

GreetingService service = new GreetingService(repository);

assertEquals("Hello, Dmitry!", service.greetUser(1));
```

Now we are testing real logic inside `GreetingService`.

## Mockito and Android

Mockito can be useful in Android projects, but mostly for local unit tests.

Good candidates:

- ViewModel logic;
- UseCase classes;
- repositories;
- validators;
- mappers;
- formatters;
- business logic that does not need a real device.

Example:

```java
class LoginViewModelTest {

    @Test
    void shouldShowSuccessWhenLoginIsValid() {
        AuthRepository authRepository = Mockito.mock(AuthRepository.class);

        Mockito.when(authRepository.login("demo", "password"))
                .thenReturn(true);

        LoginViewModel viewModel = new LoginViewModel(authRepository);

        viewModel.login("demo", "password");

        assertEquals("Success", viewModel.getState());
    }
}
```

For Android UI automation, Mockito is usually not the main tool. If you test screens, buttons, navigation, and real UI behavior, you will probably use Espresso, UIAutomator, or Appium.

But if you test pure logic behind the screen, Mockito fits perfectly.

## When Not to Use Mockito

Do not use Mockito just because you can.

Avoid mocking when:

- a simple real object is easier;
- the class has no external dependencies;
- the test becomes harder to read than the production code;
- you are mocking every internal method;
- you are only testing implementation details.

For example, if you have a small `PriceCalculator` class with no database, no network, and no external services, just create the real object and test it directly.

Simple code should have simple tests.

## Final Thoughts

Mockito is still one of the most useful tools in Java test automation.

It helps you isolate logic, remove slow dependencies, and write fast unit tests. But the goal is not to mock everything. The goal is to test the right thing at the right level.

Use Mockito when it makes the test clearer. Do not use it to hide bad architecture.

And definitely do not use it to create fake confidence.

A good Mockito test should answer one simple question:

> Does this piece of logic work correctly when its dependencies behave in a known way?

If the answer is yes, Mockito is doing its job.
