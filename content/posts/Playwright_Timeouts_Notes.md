# Playwright Timeouts — Complete Notes, Revision & Interview Guide

> **Purpose:** Understand Playwright timeouts by category, know what belongs in `playwright.config.ts`, understand precedence, and avoid the common mistake of treating every timeout as the same thing.

---

# 1. The Big Picture

Playwright has **multiple independent timeout mechanisms**.

The most important ones are:

```text
                    PLAYWRIGHT TIMEOUTS
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
     TEST TIMEOUT      ACTION TIMEOUT      EXPECT TIMEOUT
        │                  │                  │
   whole test          click/fill/etc.    expect(...)
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                 NAVIGATION TIMEOUT
                           │
                     goto/reload/etc.

Additional:
- GLOBAL TIMEOUT
- FIXTURE TIMEOUT
- beforeAll / afterAll timeout
```

The key idea:

> **Do not ask "What is the Playwright timeout?"**
>
> Ask **"Which timeout category is controlling this operation?"**

---

# 2. Timeout Categories — Quick Reference

| Timeout | Config location | Default | Controls |
|---|---|---:|---|
| **Test timeout** | `timeout` | 30 sec | One test + applicable setup/hook time |
| **Expect timeout** | `expect.timeout` | 5 sec | Auto-retrying assertions |
| **Action timeout** | `use.actionTimeout` | 0 / no timeout | Playwright actions |
| **Navigation timeout** | `use.navigationTimeout` | 0 / no timeout | Navigation operations |
| **Global timeout** | `globalTimeout` | 0 / disabled | Entire test run |
| **Fixture timeout** | Fixture definition | No separate timeout by default | Individual fixture |
| **beforeAll / afterAll timeout** | Hook/test timeout setting | 30 sec | Individual hook |
| **Per-operation timeout** | `{ timeout: ... }` | — | One specific operation/assertion |

The official Playwright Test documentation separates these timeout types. citeturn0search0turn0search1

---

# 3. The Most Important Mental Model

Imagine your test is:

```ts
test("Create customer", async ({ page }) => {

    await page.goto("/login");

    await page.getByLabel("Username").fill("karthik");

    await page.getByRole("button", {
        name: "Login"
    }).click();

    await expect(
        page.getByText("Dashboard")
    ).toBeVisible();

});
```

Different operations can be controlled by **different timeout categories**:

```text
test("Create customer")
│
├── TEST TIMEOUT
│      └── Overall test budget
│
├── page.goto()
│      └── NAVIGATION TIMEOUT
│
├── fill()
│      └── ACTION TIMEOUT
│
├── click()
│      └── ACTION TIMEOUT
│
└── expect(...)
       └── EXPECT TIMEOUT
```

This is the most important mental map.

---

# 4. `playwright.config.ts` — Main Timeout Configuration

A typical configuration can look like:

```ts
import { defineConfig } from "@playwright/test";

export default defineConfig({

    // 1. Maximum time for each test
    timeout: 30_000,

    // 2. Assertion timeout
    expect: {
        timeout: 5_000
    },

    // 3. Entire test-run timeout
    globalTimeout: 60 * 60 * 1000,

    use: {

        // 4. Default timeout for actions
        actionTimeout: 10_000,

        // 5. Default timeout for navigation
        navigationTimeout: 30_000
    }
});
```

These are **different settings for different jobs**. citeturn0search0turn0search4

---

# 5. Test Timeout

## Configuration

```ts
export default defineConfig({
    timeout: 30_000
});
```

Default:

```text
30 seconds
```

This is the timeout for **each test**.

It includes time spent by:

- test function
- test-scoped fixture setup
- `beforeEach`

There is a separate timeout period for fixture teardown and `afterEach` after the test function finishes. `beforeAll` and `afterAll` have their own hook timeout. citeturn0search0turn0search2

---

## Example

```ts
test("Create customer", async ({ page }) => {

    await page.goto("/customers");

    // test operations...

});
```

If the test's applicable test timeout expires:

```text
Test timeout exceeded
```

---

# 6. Changing Test Timeout

## In config

```ts
export default defineConfig({
    timeout: 60_000
});
```

Now each test gets:

```text
60 seconds
```

---

## For one test

```ts
test("Slow report test", async ({ page }) => {

    test.setTimeout(120_000);

    // test...
});
```

Now this test gets:

```text
120 seconds
```

---

## Using `test.slow()`

```ts
test("Slow test", async ({ page }) => {

    test.slow();

});
```

`test.slow()` is a convenient way to triple the default test timeout. citeturn0search0turn0search3

---

# 7. Test Timeout Precedence

Think:

```text
playwright.config.ts
        ↓
   base test timeout
        ↓
test.describe.configure(...)
        ↓
test.setTimeout(...)
```

A more specific test/group setting can change the timeout for the relevant tests.

Example:

```ts
export default defineConfig({
    timeout: 30_000
});
```

Then:

```ts
test.describe("Reports", () => {

    test.describe.configure({
        timeout: 60_000
    });

    test("Generate report", async () => {
        // gets 60 seconds
    });
});
```

And a specific test can change its own timeout:

```ts
test("Very slow report", async () => {

    test.setTimeout(120_000);

});
```

Playwright documents `test.setTimeout()` and `test.describe.configure({ timeout })` as ways to change test timeout at narrower scopes. citeturn0search3turn0search5

### Easy memory

```text
Config
  ↓
Describe/group
  ↓
Individual test
```

**More specific test-level configuration wins for that test.**

---

# 8. Expect Timeout

This is one of the most important timeouts for Playwright.

## Configuration

```ts
export default defineConfig({

    expect: {
        timeout: 5_000
    }

});
```

Default:

```text
5 seconds
```

It controls **auto-retrying async Web-first assertions**. citeturn0search0turn0search9

---

## Example

```ts
await expect(
    page.getByText("Order Created")
).toBeVisible();
```

Playwright retries the assertion until:

```text
Condition becomes true
        OR
Expect timeout expires
```

---

# 9. Per-Assertion Expect Timeout

You can override it for one assertion:

```ts
await expect(
    page.getByText("Report Ready")
).toBeVisible({
    timeout: 30_000
});
```

Now:

```text
Global expect config = 5 sec

This assertion = 30 sec
```

The per-assertion timeout is more specific.

---

# 10. Expect Timeout Precedence

Think:

```text
playwright.config.ts
expect.timeout
       ↓
default for assertions
       ↓
expect(..., { timeout: xxx })
       ↓
specific assertion wins
```

Example:

```ts
export default defineConfig({
    expect: {
        timeout: 5000
    }
});
```

Then:

```ts
await expect(locator).toBeVisible({
    timeout: 15000
});
```

This assertion uses:

```text
15 seconds
```

not 5 seconds.

---

# 11. Action Timeout

Configuration:

```ts
export default defineConfig({

    use: {
        actionTimeout: 10_000
    }

});
```

This sets the default timeout for Playwright actions.

Examples include operations such as:

```ts
click()
fill()
check()
uncheck()
hover()
press()
selectOption()
setInputFiles()
```

The official API describes `actionTimeout` as the default timeout for each Playwright action. Its default is `0`, meaning no action timeout. citeturn0search4turn1search0

---

# 12. Action Timeout vs Per-Action Timeout

Config:

```ts
use: {
    actionTimeout: 10_000
}
```

Then:

```ts
await page.getByRole("button", {
    name: "Login"
}).click();
```

The click uses the configured action timeout.

But:

```ts
await page.getByRole("button", {
    name: "Login"
}).click({
    timeout: 20_000
});
```

This specific click uses:

```text
20 seconds
```

not:

```text
10 seconds
```

### Mental model

```text
actionTimeout
      ↓
Default for actions

click({ timeout: 20000 })
      ↓
Override for this click
```

---

# 13. Action Timeout Precedence

For a normal page action, think:

```text
Per-action timeout
        ↓
page.setDefaultTimeout()
        ↓
browserContext.setDefaultTimeout()
        ↓
config use.actionTimeout
```

The important point from the API is that `page.setDefaultTimeout()` and `browserContext.setDefaultTimeout()` can change the default timeout for methods accepting a timeout. Page-level settings take precedence over context-level settings. citeturn1search0turn1search2

Example:

```ts
// config
use: {
    actionTimeout: 10000
}
```

Then at runtime:

```ts
page.setDefaultTimeout(20000);
```

Then:

```ts
await button.click();
```

uses the page-level default:

```text
20 seconds
```

And:

```ts
await button.click({
    timeout: 5000
});
```

uses:

```text
5 seconds
```

because the operation-specific timeout is most specific.

---

# 14. Navigation Timeout

Configuration:

```ts
export default defineConfig({

    use: {
        navigationTimeout: 30_000
    }

});
```

This controls navigation operations.

Common examples:

```ts
page.goto()
page.reload()
page.goBack()
page.goForward()
page.setContent()
page.waitForNavigation()
page.waitForURL()
```

The exact methods affected depend on the API, but the key concept is:

> **Navigation has its own timeout category.**

The default `navigationTimeout` is `0`, meaning no timeout. citeturn0search4turn1search0

---

# 15. Navigation Timeout vs Action Timeout

This is an important distinction.

```ts
use: {
    actionTimeout: 10_000,
    navigationTimeout: 30_000
}
```

Then:

```ts
await page.getByRole("button", {
    name: "Login"
}).click();
```

The click itself is primarily controlled by the **action timeout**.

But:

```ts
await page.goto("/dashboard");
```

uses the **navigation timeout**.

Mental model:

```text
click / fill / check / hover
          ↓
    ACTION TIMEOUT

goto / reload / back / forward
          ↓
 NAVIGATION TIMEOUT
```

---

# 16. Per-Navigation Timeout

You can override navigation timeout for one operation:

```ts
await page.goto("/reports", {
    timeout: 60_000
});
```

This means:

```text
Config navigationTimeout = 30 sec

This goto = 60 sec
```

The per-call timeout is more specific.

---

# 17. Navigation Timeout Runtime Precedence

Playwright's API explicitly documents the relationship between these settings.

For navigation operations:

```text
Per-navigation { timeout }
        ↓
page.setDefaultNavigationTimeout()
        ↓
page.setDefaultTimeout()
        ↓
browserContext.setDefaultNavigationTimeout()
        ↓
browserContext.setDefaultTimeout()
        ↓
config use.navigationTimeout
```

Important API detail:

> `page.setDefaultNavigationTimeout()` takes priority over `page.setDefaultTimeout()`, `browserContext.setDefaultTimeout()`, and `browserContext.setDefaultNavigationTimeout()`.

Also, page-level defaults take priority over context-level defaults. citeturn1search0turn1search2

Do not memorize every line mechanically. Remember:

```text
Specific operation
      ↓
Page-level navigation default
      ↓
Context-level navigation/default timeout
      ↓
Config default
```

---

# 18. Global Timeout

Configuration:

```ts
export default defineConfig({

    globalTimeout: 60 * 60 * 1000

});
```

This controls the **whole Playwright test run**.

Example:

```text
100 tests
   ↓
All workers running
   ↓
Global timeout = 1 hour
   ↓
Whole run cannot continue beyond that global limit
```

Default:

```text
0
```

which means the global timeout is disabled. citeturn0search0turn0search1

---

# 19. Global Timeout Is NOT Test Timeout

Very important.

Suppose:

```ts
export default defineConfig({

    timeout: 30_000,

    globalTimeout: 3_600_000

});
```

These mean:

```text
timeout: 30 sec
    ↓
Maximum time for an individual test

globalTimeout: 1 hour
    ↓
Maximum time for the entire test run
```

Mental picture:

```text
WHOLE TEST RUN
┌───────────────────────────────────────┐
│          globalTimeout = 1h           │
│                                       │
│  Test 1 → 30s max                     │
│  Test 2 → 30s max                     │
│  Test 3 → 30s max                     │
│  Test 4 → 30s max                     │
│  ...                                  │
└───────────────────────────────────────┘
```

They are **not competing timeouts for the same operation**.

---

# 20. Fixture Timeout

Fixtures can have their own timeout.

Example:

```ts
const test = base.extend<{
    slowFixture: string;
}>({

    slowFixture: [async ({}, use) => {

        // slow setup

        await use("hello");

    }, {
        timeout: 60_000
    }]

});
```

This gives that fixture its own timeout.

By default, a fixture's setup/teardown time is part of the test timeout. A separate fixture timeout is useful when a fixture needs more time. citeturn0search0turn0search2

---

# 21. `beforeEach` / `afterEach`

`beforeEach` participates in the test timeout.

So if:

```ts
timeout: 30_000
```

and the test has:

```ts
test.beforeEach(async ({ page }) => {
    // setup
});
```

the applicable `beforeEach` time counts toward the test timeout.

For `afterEach`, Playwright documents a separate timeout period after the test function finishes. citeturn0search0

---

# 22. `beforeAll` / `afterAll`

These hooks have their own timeout.

By default, their timeout is the test timeout value.

Example:

```ts
test.beforeAll(async () => {

    test.setTimeout(60_000);

    // slow setup

});
```

This changes the timeout for that hook rather than changing the timeout of every individual test.

Playwright documents `beforeAll` and `afterAll` as having their own timeout; they do not share time with a test. citeturn0search0turn0search3

---

# 23. `testInfo.setTimeout()`

You can also change the currently running test timeout through `testInfo`.

Example:

```ts
test.beforeEach(async ({ page }, testInfo) => {

    testInfo.setTimeout(
        testInfo.timeout + 30_000
    );

});
```

This changes the timeout for the currently running test/hook context as documented by Playwright. citeturn0search6

---

# 24. The Complete Precedence Map

This is the section you should remember for interviews.

## A. Test timeout

```text
playwright.config.ts
timeout
       ↓
test.describe.configure({ timeout })
       ↓
test.setTimeout(...)
```

More specific test/group configuration can override the broader configuration.

---

## B. Assertion timeout

```text
playwright.config.ts
expect: { timeout: ... }
       ↓
expect(...).matcher({ timeout: ... })
```

Example:

```ts
expect: {
    timeout: 5000
}
```

then:

```ts
await expect(locator).toBeVisible({
    timeout: 15000
});
```

The assertion uses 15 seconds.

---

## C. Action timeout

```text
playwright.config.ts
use.actionTimeout
       ↓
browserContext.setDefaultTimeout()
       ↓
page.setDefaultTimeout()
       ↓
locator.click({ timeout: ... })
```

For the runtime API, page-level defaults take priority over context-level defaults, and an explicit operation timeout is the most specific choice. citeturn1search0turn1search2

---

## D. Navigation timeout

```text
playwright.config.ts
use.navigationTimeout
       ↓
browserContext.setDefaultNavigationTimeout()
       ↓
page.setDefaultNavigationTimeout()
       ↓
page.goto({ timeout: ... })
```

There is an important additional rule:

```text
page.setDefaultNavigationTimeout()
```

takes priority over:

```text
page.setDefaultTimeout()
browserContext.setDefaultTimeout()
browserContext.setDefaultNavigationTimeout()
```

Playwright explicitly documents this priority. citeturn1search0turn1search2

---

# 25. One Example — See All Timeouts Together

```ts
import { defineConfig } from "@playwright/test";

export default defineConfig({

    // Test timeout
    timeout: 30_000,

    // Assertion timeout
    expect: {
        timeout: 5_000
    },

    // Entire test run
    globalTimeout: 60 * 60 * 1000,

    use: {

        // UI action timeout
        actionTimeout: 10_000,

        // Navigation timeout
        navigationTimeout: 30_000
    }
});
```

Then:

```ts
test("Create customer", async ({ page }) => {

    // Navigation timeout
    await page.goto("/login");

    // Action timeout
    await page.getByLabel("Username")
        .fill("Karthik");

    // Action timeout
    await page.getByRole("button", {
        name: "Login"
    }).click();

    // Expect timeout
    await expect(
        page.getByText("Dashboard")
    ).toBeVisible();

});
```

The test itself is still under:

```text
30-second test timeout
```

while the individual operations have their own timeout categories.

---

# 26. Critical Concept: These Timeouts Are Not Additive

Do NOT think:

```text
30 sec test
+
10 sec action
+
5 sec expect
=
45 sec test
```

That is the wrong mental model.

Instead:

```text
TEST
└── has its own test-time budget

ACTION
└── has an action-level timeout

ASSERTION
└── has an assertion-level timeout
```

The test timeout is an enclosing test-level limit; the other timeout settings govern their respective operations. The exact failure can therefore depend on which timeout is reached first.

---

# 27. `waitForTimeout()` Is Different

This:

```ts
await page.waitForTimeout(5000);
```

means:

> **Pause for 5 seconds.**

This:

```ts
await button.click({
    timeout: 5000
});
```

means:

> **Allow up to 5 seconds for the click operation to succeed.**

Remember:

```text
waitForTimeout(5000)
       ↓
fixed sleep

{ timeout: 5000 }
       ↓
maximum allowed operation/assertion time
```

---

# 28. Locator Timeout

Common examples:

```ts
await button.click({
    timeout: 5000
});

await textbox.fill("Hello", {
    timeout: 5000
});

await checkbox.check({
    timeout: 5000
});

await locator.waitFor({
    timeout: 5000
});
```

These are **per-operation overrides**.

They do not modify the project-wide configuration.

---

# 29. Locator Creation Is Different

Do not confuse:

```ts
const button = page.getByRole("button", {
    name: "Login"
});
```

with:

```ts
await button.click({
    timeout: 5000
});
```

Mental model:

```text
getByRole()
     ↓
Build Locator

click()
     ↓
Perform operation
     ↓
Timeout applies here
```

Similarly:

```ts
page.locator()
page.getByText()
page.getByRole()
locator.nth()
locator.filter()
```

are locator-building/refinement operations.

---

# 30. Event Timeout

For event waits:

```ts
const popupPromise = page.waitForEvent("popup", {
    timeout: 10_000
});
```

Similarly:

```ts
page.waitForEvent("download", {
    timeout: 10_000
});

page.waitForEvent("filechooser", {
    timeout: 10_000
});

context.waitForEvent("page", {
    timeout: 10_000
});
```

These are per-event wait timeouts.

---

# 31. API Timeout

For Playwright API testing:

```ts
await request.get("/users", {
    timeout: 10_000
});

await request.post("/users", {
    timeout: 10_000
});

await request.put("/users/1", {
    timeout: 10_000
});

await request.patch("/users/1", {
    timeout: 10_000
});

await request.delete("/users/1", {
    timeout: 10_000
});
```

Here the timeout belongs to the **API request operation**.

Do not confuse it with:

```text
test timeout
expect timeout
action timeout
navigation timeout
```

---

# 32. What Should Normally Go in `playwright.config.ts`?

A practical framework configuration might start with:

```ts
export default defineConfig({

    timeout: 30_000,

    expect: {
        timeout: 5_000
    },

    use: {
        actionTimeout: 10_000,
        navigationTimeout: 30_000
    }

});
```

But these values are **project decisions**, not mandatory Playwright values.

Do not blindly copy numbers from someone else's framework.

---

# 33. What Should You Override at Test Level?

Use test-level overrides for exceptional tests.

Example:

```ts
test("Generate large report", async ({ page }) => {

    test.setTimeout(120_000);

});
```

Use assertion-level overrides for exceptional assertions:

```ts
await expect(
    page.getByText("Report Generated")
).toBeVisible({
    timeout: 60_000
});
```

Use action/navigation overrides when one specific operation needs longer:

```ts
await page.getByRole("button", {
    name: "Generate"
}).click({
    timeout: 20_000
});
```

---

# 34. Common Mistakes

## Mistake 1 — Increasing test timeout to fix an assertion

Bad thinking:

```text
Assertion failed after 5 sec
        ↓
Increase test timeout to 2 minutes
```

If the problem is specifically the assertion, consider:

```ts
expect: {
    timeout: 10000
}
```

or a per-assertion timeout.

---

## Mistake 2 — Increasing action timeout for a navigation problem

If:

```ts
page.goto()
```

is timing out, understand the navigation timeout separately.

---

## Mistake 3 — Adding `waitForTimeout()` everywhere

Bad:

```ts
await page.waitForTimeout(5000);
await button.click();
```

Prefer Playwright's automatic waiting and web-first assertions.

---

## Mistake 4 — Using 60 seconds for everything

```ts
timeout: 60000
actionTimeout: 60000
navigationTimeout: 60000
expect: {
    timeout: 60000
}
```

This may hide synchronization or application problems and can make failures slow.

---

## Mistake 5 — Thinking all timeouts are one hierarchy

They are not.

Remember:

```text
TEST TIMEOUT
ACTION TIMEOUT
NAVIGATION TIMEOUT
EXPECT TIMEOUT
GLOBAL TIMEOUT
FIXTURE TIMEOUT
HOOK TIMEOUT
```

They serve different purposes.

---

# 35. Interview Scenario

### Question

You have:

```ts
export default defineConfig({

    timeout: 30_000,

    expect: {
        timeout: 5_000
    },

    use: {
        actionTimeout: 10_000,
        navigationTimeout: 20_000
    }
});
```

And:

```ts
await page.goto("/dashboard");

await page.getByRole("button", {
    name: "Generate"
}).click();

await expect(
    page.getByText("Report Ready")
).toBeVisible();
```

### Which timeout applies?

```text
page.goto()
    ↓
navigationTimeout = 20 sec

click()
    ↓
actionTimeout = 10 sec

expect(...).toBeVisible()
    ↓
expect.timeout = 5 sec

Whole test
    ↓
test timeout = 30 sec
```

This is the mental model interviewers want you to understand.

---

# 36. Senior SDET Mental Model

When a timeout occurs, first identify **what timed out**.

```text
Did the TEST exceed its overall limit?
        ↓
Test timeout

Did CLICK/FILL/CHECK/etc. exceed its limit?
        ↓
Action timeout

Did GOTO/RELOAD/etc. exceed its limit?
        ↓
Navigation timeout

Did EXPECT keep retrying too long?
        ↓
Expect timeout

Did the entire suite run too long?
        ↓
Global timeout

Did a fixture take too long?
        ↓
Fixture timeout

Did beforeAll/afterAll take too long?
        ↓
Hook timeout
```

This is much more useful than memorizing numbers.

---

# 37. Final Timeout Cheat Sheet

```text
┌──────────────────────┬───────────────────────────────┐
│ TEST                 │ timeout                       │
│                      │ One test's overall timeout    │
├──────────────────────┼───────────────────────────────┤
│ EXPECT               │ expect.timeout                │
│                      │ Assertion retry timeout       │
├──────────────────────┼───────────────────────────────┤
│ ACTION               │ use.actionTimeout             │
│                      │ click/fill/check/etc.         │
├──────────────────────┼───────────────────────────────┤
│ NAVIGATION           │ use.navigationTimeout         │
│                      │ goto/reload/back/etc.         │
├──────────────────────┼───────────────────────────────┤
│ GLOBAL               │ globalTimeout                  │
│                      │ Whole test run                │
├──────────────────────┼───────────────────────────────┤
│ FIXTURE              │ fixture timeout                │
│                      │ Individual fixture            │
├──────────────────────┼───────────────────────────────┤
│ HOOK                 │ test.setTimeout()             │
│                      │ beforeAll/afterAll             │
└──────────────────────┴───────────────────────────────┘
```

---

# 38. The Precedence Cheat Sheet

## Test

```text
config timeout
      ↓
describe.configure({ timeout })
      ↓
test.setTimeout(...)
```

## Expect

```text
config expect.timeout
      ↓
expect(..., { timeout })
```

## Action

```text
config use.actionTimeout
      ↓
context/page default timeout
      ↓
per-action { timeout }
```

## Navigation

```text
config use.navigationTimeout
      ↓
context/page navigation default
      ↓
per-navigation { timeout }
```

Remember the runtime page/context priority:

```text
page navigation default
        >
page general default
        >
context navigation default
        >
context general default
```

as documented by Playwright's Page and BrowserContext APIs. citeturn1search0turn1search2

---

# 39. The 10 Things You Should Remember Forever

```text
1. Playwright does NOT have one universal timeout.

2. timeout in config = timeout for each test.

3. expect.timeout = timeout for auto-retrying assertions.

4. use.actionTimeout = default timeout for Playwright actions.

5. use.navigationTimeout = default timeout for navigation operations.

6. globalTimeout = maximum time for the entire test run.

7. Fixture timeout is separate when you explicitly configure a fixture timeout.

8. beforeAll/afterAll have their own hook timeout.

9. A per-operation { timeout: xxx } is more specific than its configured default.

10. waitForTimeout() is a fixed sleep — it is NOT the same thing as { timeout }.
```

---

# 40. One Final Mental Map

```text
                    PLAYWRIGHT TEST
                           │
             ┌─────────────┴─────────────┐
             │                           │
        TEST TIMEOUT                GLOBAL TIMEOUT
        "one test"                  "whole run"
             │
     ┌───────┼────────┐
     ↓       ↓        ↓
  ACTION  NAVIGATION EXPECT
     │       │        │
 click()   goto()   toBeVisible()
 fill()    reload() toHaveText()
 check()   goBack() toHaveValue()
     │       │        │
     ↓       ↓        ↓
 action    navigation expect
 timeout   timeout    timeout
```

Then remember:

```text
CONFIG
  ↓
provides DEFAULTS

PER-OPERATION
  ↓
can OVERRIDE the relevant default

TEST TIMEOUT
  ↓
controls the test-level budget

GLOBAL TIMEOUT
  ↓
controls the whole run
```

---

# Official Playwright References

- Timeouts: https://playwright.dev/docs/test-timeouts
- TestConfig: https://playwright.dev/docs/api/class-testconfig
- TestOptions: https://playwright.dev/docs/api/class-testoptions
- Page: https://playwright.dev/docs/api/class-page
- BrowserContext: https://playwright.dev/docs/api/class-browsercontext
- Test: https://playwright.dev/docs/api/class-test
- TestInfo: https://playwright.dev/docs/api/class-testinfo
- Fixtures: https://playwright.dev/docs/test-fixtures
- Assertions: https://playwright.dev/docs/test-assertions

> **Accuracy note:** Playwright timeout behavior and API signatures can change between releases. For exact signatures, verify against the current official API documentation.
