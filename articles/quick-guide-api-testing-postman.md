---
title: "Quick Guide: API Testing for Test Engineers via Postman"
description: "A short practical guide for test engineers who want to understand REST API testing with Postman: requests, parameters, authorization, assertions, and running collections from the command line."
pubDate: 2026-05-27
tags: ["API Testing", "Postman", "QA", "Test Engineering", "REST API", "Newman", "Automation"]
draft: false
---

When we talk about testing, people often imagine only UI testing.

*Click a button. Open a screen. Verify a message. Check that the app did not crash.*

That is still part of the job, of course. But modern QA work is much wider than that.

A typical QA engineer may need to:

- test a new build;
- verify old app versions still work;
- check the app against different environments;
- debug backend issues;
- write automated checks;
- understand what happens between the app and the server.

**This is where API testing becomes useful.**

It is not exactly manual testing, and it is not always full automation either. It sits somewhere in the middle. You send a request directly to the backend and check the response without clicking through the UI.

For many bugs, this is faster, cleaner, and easier to debug.

## What We Need

For this quick example, we need only two things:

- a tool for sending API requests;
- an API endpoint to test.

I will use Postman, because it is still one of the easiest tools for this job.

You can use the desktop app or the web version.

Postman documentation:  
https://learning.postman.com/docs/

For a public test API, I will use 7Timer, a simple weather forecast API.

7Timer documentation:  
https://www.7timer.info/doc.php?lang=en

## Create a Request in Postman

Open Postman and create a new collection.

Call it something simple, for example:

```text
API Testing Practice
```

Inside that collection, create a new request:

```text
Get Weather
```

Use the `GET` method and paste this URL:

```text
https://www.7timer.info/bin/astro.php?lon=19.818699&lat=41.327545&ac=0&unit=metric&output=json
```

Press **Send**.

If everything works, you should see a response from the server in the bottom part of Postman.

That is the basic idea of API testing:

```text
Request -> Server -> Response
```

You send something to the backend, and the backend returns something back.

## What Are Query Parameters?

Look at the URL again:

```text
https://www.7timer.info/bin/astro.php?lon=19.818699&lat=41.327545&ac=0&unit=metric&output=json
```

The part after `?` contains query parameters:

```text
lon=19.818699
lat=41.327545
ac=0
unit=metric
output=json
```

These parameters change what the API returns.

In this example:

- `lon` is longitude;
- `lat` is latitude;
- `unit=metric` asks for metric units;
- `output=json` asks the API to return JSON.

In Postman, you can edit those parameters in the **Params** tab. This is more convenient than editing the full URL manually.

Try disabling `output=json`.

The response may change, because the server no longer receives the same instruction.

That is already a useful test idea: change parameters and check how the backend behaves.

## Why JSON Matters

Most modern APIs return JSON.

JSON is just structured text that is easy for apps and services to read.

A response may look like this:

```json
{
  "product": "astro",
  "init": "2026052712",
  "dataseries": []
}
```

The exact response can be different, but the idea is the same.

As a tester, you usually care about a few things:

- Did the request succeed?
- Did the server return the expected status code?
- Is the response valid JSON?
- Are important fields present?
- Are the values correct?
- Is the response fast enough?
- What happens when parameters are missing or invalid?

That is already real API testing.

## Status Codes Are Important

When you send a request, the server returns an HTTP status code.

The most common ones are:

| Status Code | Meaning |
|---|---|
| 200 | OK. The request worked. |
| 201 | Created. Something was created. |
| 400 | Bad request. The client sent something wrong. |
| 401 | Unauthorized. Authentication is missing or invalid. |
| 403 | Forbidden. You are authenticated, but not allowed. |
| 404 | Not found. The resource does not exist. |
| 500 | Server error. Something broke on the backend. |

A basic API test usually starts with checking the status code.

If you expect `200`, but the server returns `500`, that is already a problem.

## Authorization

Some APIs are public. The weather endpoint above can be called without an account.

But real projects usually require authorization.

In Postman, this is usually configured in the **Authorization** tab.

Common authorization types include:

- Bearer Token;
- API Key;
- Basic Auth;
- OAuth 2.0.

For example, many APIs expect a token in the header:

```text
Authorization: Bearer your_token_here
```

Do not hardcode real tokens inside examples, screenshots, or public repositories.

Use Postman variables instead.

For example:

```text
{{access_token}}
```

This keeps the request reusable and safer.

## Writing Simple Tests in Postman

Postman can do more than just send requests.

You can also write small JavaScript checks after the response is received.

In Postman, open the request and go to:

```text
Scripts -> Post-response
```

Then add a simple status code check:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

Press **Send** again.

Now Postman will send the request and run this check automatically.

If the response status is `200`, the test passes.

## Checking the Response Body

You can also check that the response contains some expected data.

For example:

```javascript
pm.test("Body contains product field", function () {
    pm.expect(pm.response.text()).to.include("product");
});
```

This is simple, but it is not the best long-term approach.

A better option is to parse JSON and check fields directly:

```javascript
pm.test("Response has product field", function () {
    const json = pm.response.json();

    pm.expect(json).to.have.property("product");
});
```

This is cleaner because you are checking the structure, not just searching for text.

You can also check the value:

```javascript
pm.test("Product is astro", function () {
    const json = pm.response.json();

    pm.expect(json.product).to.eql("astro");
});
```

Now the test verifies that the field exists and has the expected value.

## Checking Response Time

API testing is not only about correctness.

Performance matters too.

A request can return correct data, but still be too slow.

Here is a simple response time check:

```javascript
pm.test("Response time is below 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

This checks that the response time is under one second.

Do not make this too strict for unstable public APIs. In real projects, agree on realistic limits with the team.

## A Better Test Example

Here is a more complete example for this request:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response is valid JSON", function () {
    pm.response.json();
});

pm.test("Response has product field", function () {
    const json = pm.response.json();

    pm.expect(json).to.have.property("product");
});

pm.test("Product is astro", function () {
    const json = pm.response.json();

    pm.expect(json.product).to.eql("astro");
});

pm.test("Response time is below 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

This is still short, but already useful.

It checks:

- status code;
- JSON format;
- required field;
- expected value;
- response time.

That is much better than just pressing **Send** and looking at the response manually.

## What QA Engineers Should Actually Test

For real API testing, I usually think in these categories:

### Positive Checks

The request is valid and should work.

Examples:

- valid parameters;
- valid token;
- existing user;
- correct request body;
- expected response structure.

### Negative Checks

The request is wrong and should fail correctly.

Examples:

- missing required field;
- invalid token;
- invalid ID;
- wrong data type;
- empty body;
- unsupported method.

A good backend should not only work when everything is correct. It should also fail in a predictable way.

### Contract Checks

The API response should keep the expected structure.

For example, if the app expects this:

```json
{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

Then removing `email` or changing `id` from number to string can break the frontend or mobile app.

This is why API response structure matters.

### Environment Checks

Many bugs happen because environments are different.

For example:

- staging works, production fails;
- QA environment has old data;
- test user exists in one environment but not another;
- backend config is different.

Postman environments are useful here.

You can create variables like:

```text
{{base_url}}
{{access_token}}
{{user_id}}
```

Then switch between environments without rewriting requests manually.

## Running Postman Tests from Command Line

Postman is useful as a GUI tool, but you can also run collections from the command line using Newman.

Newman is Postman’s command-line collection runner.

Install it with npm:

```bash
npm install -g newman
```

Then export your Postman collection and run it:

```bash
newman run api-testing-practice.postman_collection.json
```

This is useful when you want to run API checks in CI.

For example:

```bash
newman run api-testing-practice.postman_collection.json \
  --environment qa.postman_environment.json
```

The important part: Newman returns an exit code.

If tests fail, the CI job can fail too.

That turns your Postman collection into a lightweight automated API test suite.

Newman documentation:  
https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/

## Common Mistakes

Here are a few mistakes I often see when people start API testing:

- checking only status code and ignoring response body;
- using real production tokens in saved collections;
- testing only positive cases;
- hardcoding URLs instead of using environments;
- not checking error messages;
- not checking response structure;
- forgetting that public APIs may be unstable or slow;
- mixing test data with real user data.

API testing is simple to start, but it becomes powerful when you treat it like real testing, not just clicking **Send**.

## Conclusion

Postman is a good starting point for API testing.

You can send requests, change parameters, add authorization, inspect JSON responses, and write small assertions directly inside the request.

For a QA engineer, this is a very practical skill.

You do not need to become a backend developer to test APIs. But you should understand how requests work, what status codes mean, how JSON responses are structured, and how to verify the important parts automatically.

Start with one request.

Then add assertions.

Then add negative cases.

Then run the collection from the command line.

That is already a solid API testing workflow.