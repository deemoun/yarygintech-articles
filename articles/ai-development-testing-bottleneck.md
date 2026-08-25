---
title: "AI Made Development Faster. Testing Became the Bottleneck"
description: "The analysis of what happened to the development in the era of AI."
pubDate: "2026-08-24"
tags: ["AI", "Software Development", "QA Testing", "Test Engineer"]
draft: false
---
For years, one of the standard complaints in software development was:

> Development is done. Why is testing taking so long?

In the AI era this question becomes much more interesting.

Because development really **can** be done surprisingly fast now.

A developer can describe a feature to an AI coding agent, let it inspect
the repository, generate several files, update tests, fix compilation
errors, and produce a pull request while somebody else is still reading
the Jira ticket.

Sometimes that is genuinely impressive.

But there is a small problem.

The code being produced faster does not mean that our ability to
determine whether the code is **correct** improved at the same speed.

In many projects, we are entering a strange situation:

``` text
Feature generation: 10 minutes
        ↓
Code review: 20 minutes
        ↓
Testing: 2 hours
```

And this is not necessarily because QA is inefficient.

It is because **creating a possible implementation and proving that an
implementation behaves correctly are fundamentally different jobs.**

AI dramatically reduced the cost of producing code.

It did not reduce the cost of reality.

------------------------------------------------------------------------

# Code Generation Is Cheap Now

Imagine a relatively ordinary feature request:

``` text
Add an option to export user reports as CSV.
```

A few years ago, a developer might need to:

1.  inspect the reporting code;
2.  understand the data model;
3.  implement CSV serialization;
4.  add the UI control;
5.  handle permissions;
6.  add tests;
7.  fix formatting issues;
8.  open a pull request.

Maybe this takes half a day.

Today, an AI coding agent can often generate a plausible implementation
in minutes.

That changes the economics of software development.

The expensive part is increasingly not:

``` text
Can we produce code that implements this?
```

The expensive part becomes:

``` text
Can we prove that this implementation is actually correct?
```

Those are very different questions.

------------------------------------------------------------------------

# AI Produces Plausible Software

This word is important:

**plausible.**

Modern coding models are extremely good at producing code that *looks*
correct.

The architecture looks reasonable.

The variable names look reasonable.

The tests look reasonable.

The error handling looks reasonable.

You run:

``` bash
npm test
```

and get:

``` text
247 tests passed
```

Great.

But then somebody exports a report containing:

``` text
Smith, John
```

and discovers that the generated CSV is malformed because the comma was
not escaped correctly.

Or somebody exports:

``` text
=HYPERLINK(...)
```

and now you have a CSV injection problem.

Or the export works perfectly for 500 records and consumes 3 GB of
memory for 500,000 records.

Or timestamps are generated in the server timezone instead of the user's
timezone.

Or Unicode characters become corrupted on Windows Excel.

The AI did not necessarily write obviously bad code.

It wrote code that handled the obvious interpretation of the task.

Testing is where we discover all the interpretations nobody wrote down.

------------------------------------------------------------------------

# Development Can Be Parallelized More Easily Than Validation

This creates another interesting problem.

AI agents can generate work extremely quickly and in parallel.

You can theoretically have:

``` text
Agent 1 → Feature A
Agent 2 → Feature B
Agent 3 → Refactor C
Agent 4 → Fix bug D
Agent 5 → Update dependency E
```

Five pull requests may appear before lunch.

But somebody still needs to understand what changed.

And more importantly, somebody needs to understand how those changes
interact.

The amount of code produced can scale faster than the human ability to
validate it.

This creates what I would call a **verification backlog**.

Instead of waiting for developers to finish implementation, QA may
increasingly be waiting for enough time to understand and validate a
constant stream of generated changes.

The bottleneck moves.

------------------------------------------------------------------------

# AI Can Generate Tests Too

The obvious response is:

> Fine. Let AI generate the tests.

And it absolutely should.

AI is already useful for generating:

-   unit tests;
-   API tests;
-   Playwright tests;
-   test data;
-   mocks;
-   boundary-value cases;
-   basic regression scenarios.

But there is a subtle problem.

If the same reasoning produces both the implementation and the tests,
both can share the same misunderstanding.

Suppose the requirement says:

``` text
Users can delete their account.
```

AI generates:

``` text
DELETE /api/account
```

Then generates a test:

``` javascript
expect(response.status()).toBe(200);
```

The test passes.

Excellent.

But what does "delete account" actually mean?

Should it:

-   revoke all sessions?
-   delete uploaded files?
-   remove billing information?
-   cancel subscriptions?
-   anonymize audit records?
-   remove the account from search?
-   invalidate API keys?
-   prevent password reset?
-   disappear from another user's cached contacts?

The generated test may perfectly validate the generated implementation
while both completely miss the product requirement.

A passing test is evidence.

It is not proof that you tested the right thing.

------------------------------------------------------------------------

# The Oracle Problem Becomes More Important

Testing has an old problem usually described as the **test oracle
problem**.

A test needs some mechanism for deciding:

``` text
Correct
or
Incorrect
```

For simple behavior this is easy.

``` python
assert add(2, 2) == 4
```

But real applications contain much fuzzier expectations.

Is this recommendation good?

Is this AI summary accurate?

Is this page confusing?

Is this fraud detection result acceptable?

Is this response too slow?

Did this migration preserve everything users care about?

Does this application behave correctly after upgrading from a version
released three years ago?

AI can generate an enormous number of tests.

But somebody still has to define what **correct behavior** means.

In the AI era, this may become one of the most valuable parts of QA
engineering.

------------------------------------------------------------------------

# More Code Means More States

There is another uncomfortable effect.

When producing software becomes cheaper, teams can simply produce **more
software**.

More features.

More configuration.

More experiments.

More integrations.

More generated UI.

More API endpoints.

More conditional behavior.

Which means more states.

Suppose a feature behaves differently depending on:

``` text
Operating system
Browser
Account type
Feature flag
Locale
Subscription
Permission state
Network state
Application version
Backend version
```

You do not have ten test cases.

You have a state space.

AI can help explore it.

But AI can also make the state space larger by making it economically
possible to add even more behavior.

Development velocity can therefore increase the amount of testing
required rather than reduce it.

------------------------------------------------------------------------

# AI Bugs Can Look Surprisingly Normal

People sometimes imagine AI-generated bugs as bizarre hallucinations:

``` text
model invented nonexistent API
```

Those certainly happen.

But I think the more dangerous category is much less exciting.

AI-generated code can contain completely ordinary software bugs:

-   race conditions;
-   incorrect assumptions;
-   missing cleanup;
-   weak error handling;
-   timezone problems;
-   bad authorization checks;
-   pagination errors;
-   caching bugs;
-   null handling;
-   concurrency problems;
-   incompatible migrations;
-   accessibility regressions.

In other words:

**AI did not eliminate software engineering mistakes. It industrialized
the production of plausible implementations that can contain them.**

That makes validation more important, not less.

------------------------------------------------------------------------

# A One-Line Prompt Can Create a Huge Test Surface

Consider this prompt:

``` text
Add Google login.
```

Very short development instruction.

But testing it properly may involve:

``` text
New user
Existing user
Existing email with password login
Revoked Google permission
Expired token
Cancelled login
Multiple Google accounts
Offline device
Slow network
Backend unavailable
Account deletion
Session expiration
Logout
Token refresh
Different browsers
Mobile deep links
Existing sessions
```

The amount of text required to request a feature has almost no
relationship to the amount of behavior that needs validation.

AI makes this imbalance much more visible.

A product manager can generate a specification quickly.

An agent can generate the implementation quickly.

Reality still contains all the edge cases.

------------------------------------------------------------------------

# The Fastest Bug Fix Can Still Require Slow Testing

Imagine a production bug:

``` text
Users occasionally receive duplicate notifications.
```

An AI agent inspects the code and proposes a fix in three minutes.

Maybe it changes:

``` python
send_notification(event)
```

to something involving an idempotency key.

The diff is six lines.

How long does testing take?

Possibly much longer.

You need to reproduce the original condition.

Maybe it requires:

``` text
Two workers
Concurrent events
Retry behavior
Specific timing
Queue redelivery
Network interruption
```

The implementation may take three minutes.

Building confidence in the fix may take three hours.

This is not inefficiency.

The **complexity of verification is not proportional to the size of the
diff.**

That was always true.

AI just makes it impossible to ignore.

------------------------------------------------------------------------

# Test Automation Does Not Automatically Solve This

I am a big supporter of automation.

But automation has a specific job.

It lets us repeatedly check known expectations.

For example:

``` javascript
await page.getByRole('button', { name: 'Login' }).click();

await expect(page.getByText('Dashboard')).toBeVisible();
```

Useful.

Fast.

Repeatable.

But automation is strongest when we already know:

``` text
What to execute
What to observe
What result to expect
```

The difficult part of testing new AI-generated functionality is often
discovering those things in the first place.

That requires:

-   understanding the feature;
-   understanding the architecture;
-   identifying risks;
-   thinking about state;
-   exploring unexpected behavior;
-   deciding what deserves regression coverage.

AI can assist with all of this.

But simply generating another 5,000 automated tests is not necessarily
the answer.

You can very quickly create a test suite nobody understands and
everybody is afraid to delete.

------------------------------------------------------------------------

# False Confidence Can Become the Real Problem

Imagine an AI agent reports:

``` text
Implementation complete.

✓ 34 unit tests passed
✓ 12 integration tests passed
✓ lint passed
✓ build successful
```

That output feels authoritative.

Humans like green checkmarks.

But those checks only tell us that the system satisfied the conditions
we asked it to check.

They do not tell us whether important conditions are missing.

This is where AI can create a new type of risk:

**high-confidence-looking incomplete validation.**

Bad code that crashes immediately is easy to distrust.

Code accompanied by 46 generated tests looks much safer.

Sometimes it is.

Sometimes the tests simply document the same incorrect assumptions as
the implementation.

------------------------------------------------------------------------

# QA Should Move Earlier --- But Also Higher

For years we have talked about **shift left**:

``` text
Test earlier.
```

That still makes sense.

But AI development probably requires another shift:

``` text
Think higher.
```

QA engineers should increasingly spend less time manually checking every
obvious implementation detail and more time asking:

-   What assumptions did the agent make?
-   What did the requirement fail to specify?
-   Which states were probably ignored?
-   What can interact with this change?
-   What happens when dependencies fail?
-   What happens to existing users?
-   What would an attacker try?
-   What happens at scale?
-   Which generated tests are actually meaningful?

This is closer to **quality engineering and risk analysis** than
traditional "execute test case, mark Pass."

------------------------------------------------------------------------

# QA Can Use AI Too

None of this means testers should sit there manually clicking buttons
while developers use sophisticated agents.

That would obviously be a terrible workflow.

QA should use the same acceleration.

For example, AI can help:

``` text
Read the pull request
        ↓
Summarize behavioral changes
        ↓
Identify affected components
        ↓
Generate candidate test scenarios
        ↓
Generate test data
        ↓
Create automation
        ↓
Analyze logs
        ↓
Compare API responses
```

You can give an AI a diff and ask:

> What existing behavior could this accidentally affect?

You can provide an OpenAPI specification and ask it to generate boundary
cases.

You can feed it logs from a failed test and ask it to correlate events.

You can use it to create Playwright scaffolding.

You can use it to inspect database migrations.

This absolutely makes testing faster.

But notice what the human is increasingly doing.

The human is deciding:

``` text
What deserves attention?
```

That is a much harder thing to automate completely.

------------------------------------------------------------------------

# Testing Becomes a Search Problem

I think this is one of the more interesting ways to think about modern
QA.

Testing is increasingly a search through an enormous behavioral space.

You have:

``` text
Inputs
States
Environments
Timing
Dependencies
User behavior
Historical data
Permissions
Failures
```

Somewhere inside that space are important bugs.

You cannot test everything.

You never could.

The QA engineer's job is therefore not to maximize the number of tests.

It is to maximize the probability of discovering important failures with
the time available.

AI is extremely useful here because it can generate candidate scenarios
cheaply.

But prioritization still matters.

Ten intelligent tests can be more valuable than 10,000 generated
assertions.

------------------------------------------------------------------------

# The New Bottleneck Is Confidence

Traditionally we measured engineering velocity using things like:

``` text
Stories completed
Pull requests merged
Deployment frequency
Lead time
```

AI can make several of those numbers look fantastic.

But there is another metric hiding underneath them:

``` text
How quickly can we become confident enough to ship?
```

That is not the same as:

``` text
How quickly can we produce the implementation?
```

If AI reduces implementation from two days to two hours but validation
still takes one day, testing suddenly appears extremely slow.

Testing did not necessarily get slower.

**Development got faster around it.**

That distinction matters.

Otherwise organizations may respond by simply reducing validation until
the numbers look balanced again.

That is one way to make deployment faster.

It is not necessarily a good way to make software.

------------------------------------------------------------------------

# This Changes the Value of a Tester

The least valuable testing work in the AI era will probably be highly
mechanical work where the expected behavior is already completely
defined.

For example:

``` text
Open page.
Click button.
Check text.
Repeat 400 times.
```

Machines are increasingly good at this.

The valuable part moves toward:

``` text
Understanding systems
Finding ambiguity
Modeling risk
Designing experiments
Investigating failures
Understanding users
Connecting behavior across components
Questioning assumptions
```

In other words, the tester becomes less of a test executor and more of
an **investigator**.

I think that is actually a much more interesting job.

------------------------------------------------------------------------

# Developers Will Test More Too

The separation between:

``` text
Developer writes code
QA tests code
```

also makes less sense when agents can perform both activities.

Developers can ask an agent to generate tests immediately.

QA can ask an agent to inspect implementation details.

Both roles gain access to tools that previously required more
specialized effort.

The interesting distinction becomes less about who presses which button
and more about perspective.

A developer naturally asks:

> How do I make this work?

A tester should naturally ask:

> How can this fail?

A security specialist asks:

> How can this be abused?

Those perspectives remain valuable even if all three people use the same
AI model.

------------------------------------------------------------------------

# AI Features Make Testing Even Harder

There is also an ironic twist.

AI is not only writing applications.

Applications themselves increasingly contain AI.

Now the expected output may be nondeterministic.

Instead of:

``` text
Input A → Output B
```

we may have:

``` text
Input A → Usually acceptable output
```

That creates entirely new testing problems:

-   hallucinations;
-   prompt injection;
-   inconsistent responses;
-   unsafe outputs;
-   model upgrades;
-   latency;
-   token cost;
-   context-window behavior;
-   retrieval quality;
-   evaluation datasets;
-   probabilistic regressions.

Traditional assertions become harder.

This:

``` python
assert response == expected
```

may no longer make sense.

Now you may need statistical evaluation, semantic checks, human review,
or model-based evaluation.

So while AI accelerates development, it is simultaneously introducing
products that are **harder to test deterministically**.

That is a pretty funny outcome.

------------------------------------------------------------------------

# A Better AI-Era Workflow

A practical workflow could look something like this:

``` text
Requirement
    ↓
AI-assisted implementation
    ↓
AI-generated developer tests
    ↓
Risk analysis
    ↓
Exploratory testing
    ↓
Targeted integration/E2E automation
    ↓
Security/performance checks where relevant
    ↓
Production observability
```

The important part is that every layer answers a different question.

Unit tests:

> Does this piece behave as expected?

Integration tests:

> Do these components work together?

E2E tests:

> Can the important user workflow succeed?

Exploratory testing:

> What did we fail to anticipate?

Security testing:

> What happens when somebody intentionally abuses the system?

Production monitoring:

> What happens with real users and real data?

AI can assist with every layer.

It cannot make the layers identical.

------------------------------------------------------------------------

# Do Not Measure QA Against Code Generation Speed

This may become an important management lesson.

If an agent creates a feature in 20 minutes, it does not follow that QA
should validate it in 20 minutes.

The implementation time is almost irrelevant.

A five-line authorization change can require more careful testing than a
2,000-line UI refactor.

The right question is not:

> Why did testing take longer than coding?

The right question is:

> What level of evidence do we need before we are comfortable shipping
> this change?

Sometimes the answer will be five minutes.

Sometimes it will be two days.

That depends on risk, not typing speed.

------------------------------------------------------------------------

# Final Thoughts

AI is making software development dramatically faster.

That is real.

But generating software and validating software are asymmetric
activities.

To create a feature, you need to find **one implementation that appears
to work**.

To test it, you need to search for the many conditions under which it
might not.

AI helps both sides.

It can generate tests, analyze diffs, produce test data, inspect logs,
create automation, and suggest edge cases.

But faster code generation also means:

``` text
More code
More changes
More experiments
More states
More interactions
More things requiring validation
```

So testing may increasingly look like the slow part of development.

That does not necessarily mean testing failed to keep up.

It may mean that **confidence became the expensive part of software
engineering.**

And in the AI era, that is probably exactly where good QA engineers
should be working.

Not competing with an agent to see who can click through a checklist
faster.

But figuring out what nobody --- including the AI --- thought to check.