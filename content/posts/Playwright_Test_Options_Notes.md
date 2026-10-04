# Playwright Test Options — Practical Notes

## 1. Big Picture

Think of Playwright configuration in layers:

```text
playwright.config.ts
        ↓
Global defaults / projects / workers

test.describe.configure()
        ↓
Group execution settings

test.use()
        ↓
Browser / Browser Context settings

test(...)
        ↓
Individual test

expect(...)
        ↓
Assertion behavior
```

The goal is **not** to memorize every option.  
Understand **which level controls what**.

---

# 2. `test.use()` — MUST KNOW

Used to override Playwright configuration for a test or group of tests.

```ts
test.use({
  browserName: "chromium",
  viewport: { width: 1280, height: 720 }
});
```

### Common options

| Option | Purpose |
|---|---|
| `browserName` | Select Chromium, Firefox, or WebKit |
| `viewport` | Browser viewport size |
| `baseURL` | Base URL used by navigation |
| `storageState` | Reuse authentication/session state |
| `locale` | Browser locale |
| `timezoneId` | Browser timezone |
| `geolocation` | Browser location |
| `permissions` | Browser permissions |
| `colorScheme` | Light/dark color scheme |
| `userAgent` | Custom browser user agent |
| `extraHTTPHeaders` | Additional HTTP headers |
| `httpCredentials` | HTTP authentication credentials |
| `ignoreHTTPSErrors` | Ignore HTTPS certificate errors |
| `serviceWorkers` | Configure service-worker behavior |

### Mental model

```text
test.use()
    ↓
Controls the test's browser/context environment
```

---

# 3. `test.describe()` — MUST KNOW

Groups related tests.

```ts
test.describe("Login Tests", () => {

  test("Valid login", async ({ page }) => {});

  test("Invalid login", async ({ page }) => {});

});
```

It can also contain common configuration and hooks.

```ts
test.describe("Mobile Tests", () => {

  test.use({
    viewport: { width: 375, height: 667 }
  });

  test("Test 1", async ({ page }) => {});

});
```

### Remember

> `test.describe()` = group related tests.

---

# 4. `test.describe.configure()` — MUST KNOW

Used to configure execution behavior for a `describe` group.

```ts
test.describe.configure({
  mode: "parallel",
  retries: 2,
  timeout: 60_000
});
```

Important properties:

```text
mode
retries
timeout
```

### Mental model

```text
test.describe.configure()
        ↓
Controls HOW a group of tests executes
```

---

# 5. `mode` — MUST KNOW

Controls execution mode for tests in a `describe` group.

## Default

```ts
test.describe.configure({
  mode: "default"
});
```

Tests normally execute sequentially within the worker.

## Parallel

```ts
test.describe.configure({
  mode: "parallel"
});
```

Tests in the group can run in parallel.

## Serial

```ts
test.describe.configure({
  mode: "serial"
});
```

Tests are run serially and are treated as a dependent group.

### Do NOT confuse `mode` and `workers`

```text
workers
   ↓
How many worker processes are available?

mode: parallel
   ↓
Whether tests in a describe group can execute in parallel
```

---

# 6. `test.skip()` — MUST KNOW

Skips a test.

```ts
test.skip("Feature not ready", async ({ page }) => {
});
```

It can also be conditional:

```ts
test.skip(browserName === "webkit", "Not supported");
```

### Remember

```text
skip → Do not run this test
```

---

# 7. `test.fixme()` — MUST KNOW

Marks a test as known to be broken and needing a fix.

```ts
test.fixme("Known application bug", async ({ page }) => {
});
```

### Remember

```text
fixme → This test is broken / needs fixing
```

---

# 8. `test.fail()` — MUST KNOW

Marks a test as expected to fail.

```ts
test.fail("Known bug", async ({ page }) => {
});
```

### Important difference

```text
skip
  → Don't run

fixme
  → Don't run because it is known to be broken

fail
  → Run it, but failure is expected
```

---

# 9. `test.slow()` — MUST KNOW

Marks a test as slow.

```ts
test.slow();
```

You can also provide a description:

```ts
test.slow(true, "This test is slow");
```

Playwright increases the test timeout for a slow test.

### Remember

```text
slow → Tell Playwright this test needs more time
```

---

# 10. Test Timeout — MUST KNOW

A test timeout controls how long the **test as a whole** is allowed to run.

### Global

```ts
// playwright.config.ts

export default defineConfig({
  timeout: 30_000
});
```

### Individual test

```ts
test.setTimeout(60_000);
```

### Describe group

```ts
test.describe.configure({
  timeout: 60_000
});
```

### Important

Do not confuse:

```text
Test timeout
    ↓
Entire test

Action timeout
    ↓
click(), fill(), etc.

Assertion timeout
    ↓
expect(...)
```

---

# 11. `retries` — MUST KNOW

Controls how many times Playwright retries a failed test.

### Global

```ts
export default defineConfig({
  retries: 2
});
```

### Describe group

```ts
test.describe.configure({
  retries: 2
});
```

Mental model:

```text
Test fails
   ↓
Retry 1
   ↓
Retry 2
```

---

# 12. `test.step()` — SHOULD KNOW

Creates a named step inside a test.

```ts
await test.step("Login", async () => {
  await page.getByLabel("Username").fill("admin");
  await page.getByLabel("Password").fill("password");
  await page.getByRole("button", { name: "Login" }).click();
});
```

Useful for:

- Readable reports
- Debugging
- Organizing large tests

### Remember

```text
test.step()
    ↓
Break one test into meaningful reportable steps
```

---

# 13. `workers` — MUST KNOW

Configured mainly in `playwright.config.ts`.

```ts
export default defineConfig({
  workers: 4
});
```

Workers are **separate worker processes used by Playwright Test** to execute tests.

### Mental model

```text
workers: 1
    ↓
One worker process

workers: 4
    ↓
Up to four worker processes
```

### Important correction

Do not remember:

> Worker = Thread

For Playwright Test, remember:

> **Worker = process used by Playwright Test for test execution.**

---

# 14. `fullyParallel` — MUST KNOW

Configured in `playwright.config.ts`.

```ts
export default defineConfig({
  fullyParallel: true
});
```

It controls whether tests across files can be run fully in parallel.

### Mental model

```text
workers
   ↓
How many worker processes?

fullyParallel
   ↓
How freely can tests be distributed for parallel execution?
```

---

# 15. Project Configuration — SHOULD KNOW

A project represents a particular test configuration/environment.

```ts
projects: [
  {
    name: "chromium",
    use: {
      browserName: "chromium"
    }
  },

  {
    name: "firefox",
    use: {
      browserName: "firefox"
    }
  }
]
```

Mental model:

```text
Project
   ↓
A named test configuration
   ↓
Chromium / Firefox / Mobile / etc.
```

---

# 16. `test.only()` — MUST KNOW

Runs only the selected test(s).

```ts
test.only("Debug this test", async ({ page }) => {
});
```

Useful while debugging locally.

### Important

Do not leave `test.only()` in committed code.

---

# 17. `test.describe.only()` — SHOULD KNOW

Runs only the selected `describe` block.

```ts
test.describe.only("Login Tests", () => {

  test("Test 1", async ({ page }) => {});

  test("Test 2", async ({ page }) => {});

});
```

Useful during local debugging.

---

# 18. `test.describe.skip()` — SHOULD KNOW

Skips the entire `describe` group.

```ts
test.describe.skip("Not ready", () => {

  test("Test 1", async ({ page }) => {});

  test("Test 2", async ({ page }) => {});

});
```

---

# 19. `test.describe.serial()` — SHOULD KNOW

Creates a serial group.

```ts
test.describe.serial("Checkout", () => {

  test("Login", async ({ page }) => {});

  test("Add product", async ({ page }) => {});

  test("Checkout", async ({ page }) => {});

});
```

Use serial execution carefully. Ideally, tests should be independent.

---

# 20. Five Buckets to Remember

```text
                    PLAYWRIGHT TEST OPTIONS
                             |
       +---------------------+---------------------+
       |                     |                     |
   EXECUTION             TEST CONTROL          ENVIRONMENT
       |                     |                     |
   workers               timeout               test.use()
   fullyParallel         retries               browserName
   mode                   skip                  viewport
   serial                 fixme                 storageState
                          fail                  baseURL
                          slow

       +---------------------+
       |
    ORGANIZATION / DEBUGGING
       |
   test.describe
   test.step
   test.only
   describe.only
   describe.skip
```

---

# 21. Priority for Interviews

## MUST KNOW

1. `test.use()`
2. `test.describe()`
3. `test.describe.configure()`
4. `test.skip()`
5. `test.fixme()`
6. `test.fail()`
7. `test.slow()`
8. `test.setTimeout()`
9. `retries`
10. `workers`
11. `fullyParallel`
12. `mode: "default" | "parallel" | "serial"`
13. `test.only()`

## SHOULD KNOW

- `test.step()`
- `test.describe.serial()`
- `test.describe.skip()`
- `test.describe.only()`
- Project configuration
- `storageState`
- `baseURL`
- Browser/context options

---

# 22. Final Mental Map

Remember this instead of memorizing everything:

```text
PLAYWRIGHT
   |
   +-- CONFIG
   |     |
   |     +-- workers
   |     +-- fullyParallel
   |     +-- retries
   |     +-- timeout
   |     +-- projects
   |
   +-- GROUP
   |     |
   |     +-- test.describe()
   |     +-- describe.configure()
   |     +-- mode
   |
   +-- ENVIRONMENT
   |     |
   |     +-- test.use()
   |     +-- browserName
   |     +-- viewport
   |     +-- storageState
   |     +-- baseURL
   |
   +-- TEST CONTROL
   |     |
   |     +-- skip
   |     +-- fixme
   |     +-- fail
   |     +-- slow
   |     +-- only
   |
   +-- REPORTING / STRUCTURE
         |
         +-- test.step()
