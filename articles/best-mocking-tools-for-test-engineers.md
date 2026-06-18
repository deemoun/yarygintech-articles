---
title: "Best Mocking Tools for Test Engineers: What to Use for Unit, API, UI, and Service Testing"
description: "A practical QA-focused guide to mocking in 2026: when to use Jest, Vitest, Sinon, MSW, Nock, WireMock, Mockoon, Cypress intercepts, Pact, and Testcontainers."
pubDate: 2026-06-18
tags: ["Testing", "Mocking", "QA", "Automation"]
draft: false
---
Mocking is one of those topics where people quickly start sounding smarter than they actually are.

You will hear words like mocks, stubs, spies, fakes, service virtualization, contract testing, synthetic backends, simulated dependencies, and whatever new term the tool vendors are pushing this year.

But for a test engineer, the real question is simpler:

> How do I make tests stable, fast, useful, and still close enough to reality?

That is the whole game.

Mocking is not about avoiding real testing. It is about controlling the parts of the system that are slow, expensive, random, unavailable, dangerous, or hard to reproduce.

A good mock helps you test a specific behavior clearly.  
A bad mock turns your test suite into fan fiction.

So let's talk about the tools that are actually worth knowing.

## First, Know What You Are Mocking

Before picking tools, separate the problem by layer.

There are several different kinds of mocking:

| Layer | What You Mock | Typical Tools |
|---|---|---|
| Function/unit level | Functions, modules, timers, dates, classes | Jest, Vitest, Sinon |
| HTTP/API level | REST/GraphQL responses | MSW, Nock, WireMock, Mockoon |
| Browser/UI level | Network calls from the browser | Cypress `cy.intercept`, Playwright route mocking, MSW |
| Service level | Whole external services | WireMock, Hoverfly, Mountebank |
| Contract level | Expected provider/consumer behavior | Pact |
| Environment level | Databases, queues, Redis, S3-like storage | Testcontainers, LocalStack, Docker Compose |

This distinction matters.

If you use a unit-test mock tool to simulate a whole distributed system, the tests will become fake.  
If you use a heavy service virtualization stack just to mock one helper function, you are overengineering.

A test engineer should know both directions: small mocks and big mocks.

## Mock, Stub, Spy, Fake: Quick Practical Difference

People argue about definitions. In real projects, use this mental model.

A **spy** watches what happened.

```ts
const sendSpy = vi.spyOn(emailService, "send");

await userSignup("test@example.com");

expect(sendSpy).toHaveBeenCalledWith("test@example.com");
```

A **stub** returns controlled data.

```ts
vi.spyOn(userRepository, "findById").mockResolvedValue({
  id: "123",
  name: "Dmitry",
});
```

A **mock** usually means a fake object with expectations.

```ts
const paymentGateway = {
  charge: vi.fn().mockResolvedValue({ status: "approved" }),
};

await checkout(paymentGateway);

expect(paymentGateway.charge).toHaveBeenCalledOnce();
```

A **fake** is a simplified working implementation.

```ts
class FakeUserStore {
  private users = new Map();

  save(user) {
    this.users.set(user.id, user);
  }

  findById(id) {
    return this.users.get(id);
  }
}
```

The fake is often underrated. It can be more maintainable than a pile of mocks.

## 1. Jest Mocks: Still Good for Mature JavaScript Projects

Jest is still one of the default answers for JavaScript and TypeScript testing, especially in older React, Node, and enterprise projects.

The mocking tools are built in:

- `jest.fn()`
- `jest.mock()`
- `jest.spyOn()`
- fake timers
- manual mocks
- module mocks

Basic example:

```ts
import { sendWelcomeEmail } from "./email";
import { registerUser } from "./register";

jest.mock("./email", () => ({
  sendWelcomeEmail: jest.fn(),
}));

test("sends welcome email after registration", async () => {
  await registerUser({
    email: "qa@example.com",
    password: "secret123",
  });

  expect(sendWelcomeEmail).toHaveBeenCalledWith("qa@example.com");
});
```

This is the boring but useful kind of mocking.

You are not testing the real email provider.  
You are testing that your registration flow triggers the email dependency.

Good use cases for Jest mocks:

- unit tests
- service tests
- checking function calls
- replacing slow dependencies
- controlling timers
- testing error branches
- old projects already using Jest

Bad use cases:

- full API simulation
- browser network mocking
- testing how real services behave
- replacing every dependency just because you can

Jest mocks are powerful, but they are also easy to abuse. If every test has twenty `jest.mock()` calls, the test probably knows too much about implementation details.

## 2. Vitest Mocks: The Modern Jest-Like Option

Vitest feels like Jest's younger, faster cousin for modern Vite/ESM projects.

If your frontend stack is Vite, Vue, React, Svelte, Solid, or a newer TypeScript setup, Vitest is often the nicer choice.

Mocking looks familiar:

```ts
import { describe, expect, it, vi } from "vitest";
import { calculateInvoice } from "./invoice";
import { getTaxRate } from "./tax";

vi.mock("./tax", () => ({
  getTaxRate: vi.fn(() => 0.2),
}));

describe("calculateInvoice", () => {
  it("calculates invoice with mocked tax rate", () => {
    const result = calculateInvoice(100);

    expect(result.total).toBe(120);
    expect(getTaxRate).toHaveBeenCalled();
  });
});
```

Vitest also gives you fake timers:

```ts
import { afterEach, expect, it, vi } from "vitest";

afterEach(() => {
  vi.useRealTimers();
});

it("shows timeout message after 5 seconds", () => {
  vi.useFakeTimers();

  const callback = vi.fn();

  setTimeout(callback, 5000);

  vi.advanceTimersByTime(5000);

  expect(callback).toHaveBeenCalledOnce();
});
```

Where Vitest shines:

- Vite projects
- modern TypeScript
- frontend component tests
- fast watch mode
- Jest-like API with less friction in modern stacks

Where I would be careful:

- older CommonJS-heavy projects
- complex Jest migration projects
- teams with huge existing Jest infrastructure

For a test engineer in 2026, Vitest is absolutely worth knowing. Not because it is fashionable, but because a lot of modern frontend projects have already moved toward Vite-based tooling.

## 3. Sinon: Old-School, Still Useful, Framework-Agnostic

Sinon is not the shiny tool anymore, but it is still a very useful library.

It gives you:

- spies
- stubs
- mocks
- fake timers

And it works with almost any test runner.

That means Sinon is still useful when you are not using Jest or Vitest, or when you are in a Mocha/Chai stack.

Example:

```ts
import sinon from "sinon";
import { expect } from "chai";
import { createOrder } from "./orders";

it("calls payment provider", async () => {
  const paymentProvider = {
    charge: sinon.stub().resolves({ id: "pay_123", status: "paid" }),
  };

  await createOrder({
    productId: "book",
    paymentProvider,
  });

  expect(paymentProvider.charge.calledOnce).to.equal(true);
});
```

Sinon is good when:

- you are working in Mocha/Chai
- your framework has no strong built-in mocking
- you want a standalone mocking library
- you work with legacy JavaScript test stacks

I would not add Sinon to a modern Jest or Vitest project unless there is a real reason. Jest and Vitest already cover most of this.

## 4. MSW: Probably the Best API Mocking Tool for Frontend Testing

MSW stands for Mock Service Worker.

This is one of the most useful mocking tools for frontend test engineers because it mocks at the network layer, not by replacing your API client function.

That is a big difference.

Bad approach:

```ts
vi.mock("./apiClient");
```

Better approach in many frontend tests:

```ts
// Let the app call the API normally.
// Intercept the HTTP request and return controlled data.
```

Example MSW handler:

```ts
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/user", () => {
    return HttpResponse.json({
      id: "123",
      name: "QA User",
      plan: "pro",
    });
  }),

  http.post("/api/login", async ({ request }) => {
    const body = await request.json();

    if (body.password === "wrong") {
      return HttpResponse.json(
        { message: "Invalid credentials" },
        { status: 401 }
      );
    }

    return HttpResponse.json({
      token: "fake-token",
    });
  }),
];
```

Why MSW is so good:

- your app still uses `fetch`, Axios, React Query, Apollo, etc.
- mocks can work in browser and Node test environments
- frontend and QA can share mock scenarios
- you can mock REST and GraphQL
- it works well with component tests
- it is closer to real behavior than mocking internal functions

Use MSW for:

- React/Vue/Svelte component tests
- frontend integration tests
- Storybook scenarios
- local development without a backend
- error-state testing
- loading-state testing
- empty-state testing

Example test idea:

```ts
it("shows empty state when user has no projects", async () => {
  server.use(
    http.get("/api/projects", () => {
      return HttpResponse.json([]);
    })
  );

  render(<ProjectsPage />);

  expect(await screen.findByText("No projects yet")).toBeInTheDocument();
});
```

This is exactly the kind of mock that helps a test engineer.

You are not faking the component's internal logic.  
You are faking the backend response.

That is a cleaner boundary.

## 5. Nock: Good for Node.js HTTP Mocking

Nock is a Node.js library for mocking HTTP requests.

It is very useful when you are testing backend code that calls external APIs.

Example:

```ts
import nock from "nock";
import { getCustomer } from "./crmClient";

test("loads customer from CRM", async () => {
  nock("https://crm.example.com")
    .get("/customers/123")
    .reply(200, {
      id: "123",
      name: "Alice",
    });

  const customer = await getCustomer("123");

  expect(customer.name).toBe("Alice");
});
```

Nock is good for:

- Node.js services
- backend integration-ish tests
- testing API clients
- simulating third-party APIs
- forcing 500, timeout, malformed JSON, rate limit responses

Example error test:

```ts
nock("https://payments.example.com")
  .post("/charge")
  .reply(429, {
    error: "rate_limited",
  });
```

Where Nock is not ideal:

- browser tests
- frontend component tests
- non-Node environments
- complex service virtualization across a team

For Node API client tests, Nock is still a practical tool.

For frontend, I would usually pick MSW instead.

## 6. Cypress `cy.intercept`: Great for UI-Level Network Control

Cypress has `cy.intercept()` for spying on and stubbing network requests in browser tests.

Example:

```ts
cy.intercept("GET", "/api/products", {
  statusCode: 200,
  body: [
    { id: "1", name: "Keyboard" },
    { id: "2", name: "Mouse" },
  ],
}).as("getProducts");

cy.visit("/products");

cy.wait("@getProducts");

cy.contains("Keyboard").should("be.visible");
```

This is useful because it controls the UI test environment.

You can test:

- empty lists
- server errors
- slow responses
- unauthorized responses
- edge cases that are hard to set up in real backend data

Example slow response:

```ts
cy.intercept("GET", "/api/report", {
  delay: 3000,
  statusCode: 200,
  body: { status: "ready" },
}).as("getReport");

cy.visit("/report");

cy.contains("Loading").should("be.visible");
cy.wait("@getReport");
cy.contains("ready").should("be.visible");
```

Good use cases:

- frontend E2E tests
- UI error states
- avoiding dependency on unstable staging APIs
- making UI tests deterministic
- checking whether a request was made

Bad use cases:

- replacing all backend testing
- testing payment, auth, permissions only with fake responses
- pretending mocked E2E is the same as real E2E

This is where QA discipline matters.

Some Cypress tests should hit the real backend.  
Some should use intercepts.

If every UI test is fully mocked, you are no longer testing the system. You are testing a theater version of the system.

## 7. WireMock: The Serious API Mocking and Service Virtualization Tool

WireMock is the tool I would expect a serious test engineer to know, especially in backend, microservices, enterprise, or API-heavy systems.

It lets you simulate external HTTP services.

Basic WireMock stub:

```json
{
  "request": {
    "method": "GET",
    "url": "/users/123"
  },
  "response": {
    "status": 200,
    "jsonBody": {
      "id": "123",
      "name": "Test User"
    },
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```

Then your application talks to WireMock instead of the real dependency.

Why WireMock is useful:

- works outside JavaScript
- good for microservices
- can run in Docker
- supports request matching
- supports fault simulation
- can simulate delays
- can be used in CI
- good for testing services that depend on other services

Example scenarios:

- payment provider returns `500`
- KYC provider returns pending status
- shipping provider times out
- partner API returns malformed payload
- auth service returns expired token
- search service returns empty results

WireMock becomes especially valuable when your system depends on APIs you do not control.

For a test engineer, WireMock is a career-level tool. It is not just a library. It teaches you how to think about dependencies, contracts, and controlled environments.

Use it when:

- you test backend services
- you test microservices
- you need stable CI tests
- external dependencies are flaky
- backend teams need to develop in parallel
- staging is always broken

Do not use it for:

- simple unit tests
- small frontend-only projects
- one tiny API response where MSW is enough

## 8. Mockoon: Best Simple GUI Mock Server

Mockoon is great when you need a mock API fast and do not want to build a whole framework around it.

It has a desktop app and CLI, so it is useful for manual QA, frontend demos, local testing, and basic automation support.

Typical use cases:

- frontend team needs backend-like responses
- QA needs to simulate error states
- PM wants a demo before backend is done
- you need a fake REST API locally
- you want a mock server without writing much code

Mockoon is not the most advanced service virtualization tool, but that is the point. Sometimes you just need a simple fake API server.

Example route:

```json
{
  "method": "GET",
  "endpoint": "/api/orders",
  "response": {
    "statusCode": 200,
    "body": [
      {
        "id": "ord_1",
        "status": "paid"
      },
      {
        "id": "ord_2",
        "status": "pending"
      }
    ]
  }
}
```

Where Mockoon fits:

- manual QA
- demos
- local frontend development
- quick fake APIs
- junior-friendly mocking
- small teams

Where it may not be enough:

- complex microservice simulation
- strict contract testing
- advanced CI pipelines
- stateful backend behavior
- large enterprise virtualization

I would happily use Mockoon for fast local testing.  
I would not build a whole backend test strategy only around it.

## 9. Pact: Not Just Mocking, But Contract Testing

Pact is often mentioned with mocks because it gives the consumer a mock provider during tests.

But Pact's real value is not the mock server. The real value is the contract.

The consumer says:

> This is the request I will send, and this is the response I expect.

The provider then verifies:

> Yes, I can actually satisfy that contract.

That solves a real QA problem in microservices.

Without contract testing, you can have this situation:

- frontend tests pass with mocks
- backend tests pass alone
- staging breaks because the response shape changed

Pact helps catch that earlier.

Very simplified idea:

```ts
await provider.addInteraction({
  states: [{ description: "user 123 exists" }],
  uponReceiving: "a request for user 123",
  withRequest: {
    method: "GET",
    path: "/users/123",
  },
  willRespondWith: {
    status: 200,
    headers: {
      "Content-Type": "application/json",
    },
    body: {
      id: "123",
      name: "Alice",
    },
  },
});
```

Use Pact when:

- multiple teams own different services
- frontend/backend contracts break often
- microservices change independently
- API compatibility matters
- you want more trust than static mocks

Do not use Pact just to stub everything. That defeats the point.

Pact is not a replacement for API tests.  
It is a way to stop consumer and provider expectations from drifting apart.

## 10. Testcontainers: The Anti-Mock Tool You Still Need

A mocking article should mention when not to mock.

Testcontainers lets you run real dependencies in Docker during tests:

- PostgreSQL
- MySQL
- Redis
- Kafka
- RabbitMQ
- MongoDB
- local cloud-like services
- Selenium/browser dependencies
- custom containers

This is not mocking. This is controlled reality.

Example mental model:

```ts
// Instead of mocking PostgreSQL behavior,
// start a real PostgreSQL container for the test run.
```

Why this matters:

Mocks are bad at simulating complex systems.

A mocked database will not catch:

- wrong SQL
- transaction issues
- migration problems
- index-related behavior
- encoding problems
- constraint violations
- real connection behavior

So the correct strategy is not "mock everything".

A better strategy:

- mock third-party APIs you cannot control
- use fake timers for time
- use MSW/Nock/WireMock for network boundaries
- use Testcontainers for real databases and infrastructure when possible

This gives you much stronger tests.

## My Practical Mocking Stack in 2026

If I joined a project as a test engineer, this is how I would think.

### Frontend React/Vue/Svelte Project

Use:

- Vitest or Jest for test runner
- Testing Library for component behavior
- MSW for API mocking
- Cypress or Playwright for browser-level tests
- `cy.intercept()` or Playwright routing only for specific E2E scenarios

Avoid:

- mocking every component child
- mocking API client internals too much
- snapshot-testing giant mocked HTML trees
- fully mocked E2E suite with no real backend coverage

Best setup:

```txt
Unit/component tests:
Vitest + Testing Library + MSW

Browser smoke tests:
Playwright/Cypress against real backend or stable test env

UI edge cases:
Browser test + network intercept
```

### Node.js Backend Project

Use:

- Jest or Vitest
- Nock for external HTTP APIs
- Testcontainers for database/Redis/Kafka
- WireMock for bigger dependency simulation
- Pact if multiple services depend on your API

Avoid:

- mocking the database in all integration tests
- mocking internal modules just to make tests easy
- testing only happy-path API calls

Best setup:

```txt
Unit tests:
Vitest/Jest mocks

API client tests:
Nock or MSW Node

Service integration tests:
Testcontainers

External dependencies:
WireMock

Provider/consumer safety:
Pact
```

### Microservices / Enterprise Backend

Use:

- WireMock
- Pact
- Testcontainers
- Docker Compose
- real API tests in CI
- some mocks for rare error states

Avoid:

- fragile shared staging dependency
- mocks that are not versioned
- mocks that nobody owns
- mocks copied from outdated API docs

Best setup:

```txt
Service tests:
Real service + mocked dependencies through WireMock

Contract tests:
Pact

Infrastructure tests:
Testcontainers

End-to-end:
Small number, real environment, high-value flows only
```

### Manual QA / Exploratory Testing

Use:

- Mockoon
- WireMock standalone
- browser devtools overrides
- proxy tools like Charles or mitmproxy when needed
- feature flags and test data tools

Manual QA benefits a lot from mocking.

You can test:

- 401 unauthorized
- 403 forbidden
- 500 server error
- slow API
- empty list
- corrupted response
- huge response
- weird Unicode
- expired session
- payment declined
- user with no permissions
- user with too many permissions

A good QA does not just click the happy path. Mocking gives you the power to create ugly situations on demand.

## Bad Mocking Smells

Here are signs that your mocking strategy is going wrong.

### 1. The Test Knows Too Much

Bad:

```ts
expect(userService.repository.db.client.query).toHaveBeenCalled();
```

This test is married to the implementation.

Better:

```ts
expect(result.status).toBe("created");
```

Test behavior, not plumbing.

### 2. The Mock Duplicates Production Logic

If your mock has the same business rules as production code, you now have two implementations.

That is dangerous.

Bad mock:

```ts
function calculateDiscountMock(user) {
  if (user.plan === "pro" && user.region === "EU" && user.age > 30) {
    return 0.17;
  }

  return 0.03;
}
```

This is not a mock anymore. It is a second business engine.

### 3. Every Test Mocks Everything

If your test file starts with ten module mocks, stop and think.

Maybe the code is too coupled.  
Maybe the test is too low-level.  
Maybe you need a better boundary.

### 4. No Test Ever Talks to the Real Dependency

Mocks are useful, but some test somewhere must verify reality.

If you mock Stripe, S3, email, auth, database, queue, and search in every single test, you have no idea whether production integration works.

At minimum, you need:

- contract tests
- integration tests
- smoke tests
- sandbox tests
- scheduled checks against real third-party APIs when possible

### 5. Mocks Are Shared and Dirty

Shared mock state creates flaky tests.

Always reset mocks:

```ts
afterEach(() => {
  vi.clearAllMocks();
  vi.useRealTimers();
});
```

Or in Jest:

```ts
afterEach(() => {
  jest.clearAllMocks();
  jest.useRealTimers();
});
```

Dirty mocks are one of those boring reasons why test suites become flaky and nobody trusts them.

## What Should a Test Engineer Learn First?

If you are building your mocking skill set, I would learn in this order.

### 1. Built-in Test Runner Mocks

Learn Jest or Vitest mocks first.

You need to understand:

- spy
- stub
- fake timer
- module mock
- mock reset
- async mock
- rejected promise mock

Example:

```ts
vi.fn().mockResolvedValue({ ok: true });
vi.fn().mockRejectedValue(new Error("Network error"));
```

This is basic automation literacy.

### 2. HTTP Mocking

Then learn MSW and Nock.

You need to simulate:

- `200 OK`
- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `429 Rate Limited`
- `500 Internal Server Error`
- timeout
- slow response
- malformed body

This is where test engineers become useful. Developers often test the happy path. QA should be good at forcing the ugly paths.

### 3. Browser Network Mocking

Learn `cy.intercept()` or Playwright route mocking.

Even if you prefer Playwright, Cypress intercepts are worth understanding because a lot of companies still use Cypress.

You should be able to answer:

- Did the request happen?
- What payload did the UI send?
- What happens if the backend returns empty data?
- What happens if the response is delayed?
- What happens if auth expires?

### 4. Service Virtualization

Learn WireMock.

This is the step from "I can write tests" to "I can design test environments."

You should understand:

- stub mappings
- request matching
- dynamic responses
- delays
- faults
- Docker usage
- CI usage
- test data ownership

### 5. Contract Testing

Learn Pact after you understand API mocking.

Contract testing is harder to appreciate if you have never seen frontend and backend teams break each other with small API changes.

Once you have seen that pain, Pact makes sense.

## Tool Ranking by Practical Value

Here is my honest ranking for a test engineer.

### Must Know

- Jest mocks or Vitest mocks
- MSW
- Nock
- Cypress `cy.intercept()` or equivalent browser network mocking
- WireMock

### Strong Career Boost

- Pact
- Testcontainers
- Mockoon
- Docker Compose test environments

### Useful in Specific Projects

- Sinon
- Hoverfly
- Mountebank
- LocalStack
- custom fake services

### Usually Overkill for Small Projects

- full enterprise service virtualization platform
- giant mock management portals
- complex stateful mocks for simple CRUD apps

The best mocking tool is not always the most powerful one. It is the one your team will actually use correctly.

## A Good Mocking Strategy

Here is a sane default strategy.

### Unit Tests

Mock direct dependencies.

Use:

- Jest
- Vitest
- Sinon

Goal:

- fast feedback
- edge cases
- business logic
- error branches

### Component Tests

Mock network, not components.

Use:

- MSW
- Testing Library
- Vitest/Jest

Goal:

- test UI behavior with realistic API responses

### API Client Tests

Mock HTTP server.

Use:

- Nock
- MSW Node
- WireMock

Goal:

- test request shape
- response parsing
- error handling
- retry behavior

### Service Tests

Run the real service with fake external services.

Use:

- WireMock
- Testcontainers
- Docker Compose

Goal:

- test your service seriously without depending on unstable external systems

### E2E Tests

Use fewer mocks.

Use real systems for critical flows. Use mocks only for:

- rare edge cases
- expensive third-party calls
- impossible states
- destructive actions
- flaky dependencies

Goal:

- confidence, not theater

## Example: Testing a Checkout Flow

Bad strategy:

```txt
Mock everything:
- fake product service
- fake cart service
- fake payment service
- fake database
- fake auth
- fake email
- fake UI state

Result:
The test passes, but nobody knows if checkout works.
```

Better strategy:

```txt
Unit tests:
- mock payment gateway client
- test price calculation
- test validation

API tests:
- real checkout service
- real test database
- WireMock payment provider

Contract tests:
- Pact between checkout service and payment provider wrapper

E2E smoke:
- one real checkout flow in staging/sandbox

UI edge cases:
- intercept payment declined response
- verify error message
```

That is a real strategy.

Not too pure.  
Not too fake.  
Not too slow.

## Common Mock Scenarios Every QA Should Cover

For any API-driven app, build reusable mock scenarios.

### Authentication

- valid token
- expired token
- missing token
- malformed token
- user disabled
- user has no role
- user has multiple roles

### Payments

- approved
- declined
- insufficient funds
- 3D Secure required
- provider timeout
- duplicate charge
- refund failed
- webhook delayed

### Files

- upload success
- file too large
- unsupported type
- corrupted file
- virus scan pending
- storage unavailable

### Search

- results found
- no results
- too many results
- slow search
- special characters
- Unicode input
- pagination bug

### AI / LLM Features

- empty response
- slow response
- hallucinated format
- invalid JSON
- safety refusal
- rate limit
- partial response
- streaming interrupted

Mocking is very useful for AI features because real AI output is nondeterministic. You still need real model checks, but deterministic mocks help you test UI and backend behavior around the model.

## Final Take

The best mocking tool depends on the boundary.

For function-level tests, use Jest or Vitest.  
For frontend API mocking, use MSW.  
For Node HTTP clients, use Nock.  
For browser tests, use Cypress intercepts or Playwright routing.  
For backend dependency simulation, use WireMock.  
For quick manual mock APIs, use Mockoon.  
For contracts between services, use Pact.  
For databases and infrastructure, do not mock too much — use Testcontainers.

The main skill is not memorizing tools.

The main skill is knowing what should be fake and what should stay real.

That is where a test engineer becomes valuable.

Bad QA asks: "How do I mock this?"

Good QA asks:

> "Should this be mocked at all, and if yes, at what boundary?"
