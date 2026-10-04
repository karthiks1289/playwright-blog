# Playwright `playwright.config.ts` — Complete Beginner Notes & Revision

> Based on the current Playwright Test documentation.  
> Goal: understand what the config file controls, what you MUST know for real projects/interviews, and what you can learn later.

---

# 1. What is `playwright.config.ts`?

`playwright.config.ts` is the **central configuration file for Playwright Test**.

Think:

```text
playwright.config.ts
        |
        +-- Where are my tests?
        +-- How should tests execute?
        +-- How long can tests wait?
        +-- Should tests retry?
        +-- Which browser?
        +-- What browser/context settings?
        +-- How should failures be captured?
        +-- Which projects/environments?
        +-- How should results be reported?
```

Instead of repeating the same settings in every test, we configure them once.

Example:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  timeout: 30_000,

  use: {
    baseURL: 'https://myapp.com',
    browserName: 'chromium',
    headless: true,
  },

  reporter: 'html',
});
```

---

# 2. First understand the structure

A Playwright config normally looks like this:

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

  // TEST RUNNER SETTINGS
  testDir: './tests',
  timeout: 30_000,
  retries: 2,
  workers: 4,
  reporter: 'html',

  // BROWSER / CONTEXT DEFAULTS
  use: {
    baseURL: 'https://myapp.com',
    browserName: 'chromium',
    headless: true,
    screenshot: 'only-on-failure',
    trace: 'on-first-retry',
    video: 'on-first-retry',
  },

  // DIFFERENT EXECUTION CONFIGURATIONS
  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
      },
    },
  ],

});
```

## The most important distinction

### Top level

Controls **how Playwright Test runs**.

Examples:

```ts
timeout
retries
workers
fullyParallel
reporter
testDir
testMatch
projects
```

### `use`

Controls the **browser/browser-context environment used by tests**.

Examples:

```ts
use: {
  browserName
  headless
  baseURL
  viewport
  storageState
  screenshot
  video
  trace
}
```

### `projects`

Defines **separate test configurations**.

Examples:

```text
Chromium
Firefox
WebKit
Mobile Chrome
Smoke
Regression
QA environment
Staging environment
```

---

# 3. Mental Map

Remember this:

```text
CONFIG
│
├── 1. FIND TESTS
│     ├── testDir
│     ├── testMatch
│     └── testIgnore
│
├── 2. RUN TESTS
│     ├── workers
│     ├── fullyParallel
│     ├── retries
│     ├── repeatEach
│     └── maxFailures
│
├── 3. TIMEOUTS
│     ├── timeout
│     ├── expect.timeout
│     ├── globalTimeout
│     ├── use.actionTimeout
│     └── use.navigationTimeout
│
├── 4. BROWSER / CONTEXT
│     └── use
│          ├── browserName
│          ├── headless
│          ├── baseURL
│          ├── viewport
│          ├── storageState
│          ├── screenshot
│          ├── video
│          ├── trace
│          └── many other context options
│
├── 5. DIFFERENT CONFIGURATIONS
│     └── projects
│
├── 6. RESULTS
│     ├── reporter
│     └── outputDir
│
└── 7. ENVIRONMENT / SETUP
      ├── webServer
      ├── globalSetup
      └── globalTeardown
```

---

# 4. `defineConfig()`

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // configuration
});
```

`defineConfig()` is Playwright's helper for defining the configuration.

The important thing for a beginner:

```text
defineConfig(...)
      ↓
contains your Playwright configuration
```

You do not need to treat it as a Playwright feature that executes tests.

---

# 5. Test Discovery — How does Playwright find tests?

## 5.1 `testDir`

```ts
testDir: './tests'
```

Tells Playwright where to search for test files.

Example:

```text
project
│
├── playwright.config.ts
└── tests
    ├── login.spec.ts
    ├── cart.spec.ts
    └── checkout.spec.ts
```

```ts
testDir: './tests'
```

means:

> Search recursively inside `tests`.

### MUST REMEMBER

`testDir` = **Where should Playwright look for tests?**

---

# 6. `testMatch`

Controls which files are considered test files.

Example:

```ts
testMatch: '**/*.spec.ts'
```

Only matching files are collected.

You can also use a regular expression:

```ts
testMatch: /.*smoke\.spec\.ts/
```

### Default

Playwright's default matching includes JavaScript/TypeScript test files with `.test` or `.spec` naming patterns.

### MUST REMEMBER

```text
testDir   = WHERE to search
testMatch = WHICH files to include
```

---

# 7. `testIgnore`

Exclude matching files.

```ts
testIgnore: '**/experimental/**'
```

Mental model:

```text
testDir
   ↓
testMatch → include these
   ↓
testIgnore → exclude these
```

### MUST REMEMBER

`testIgnore` = **Which matching test files should NOT run?**

---

# 8. Test Timeout — `timeout`

```ts
timeout: 30_000
```

Maximum time allowed for an individual test.

Default:

```text
30 seconds
```

Important:

The test timeout includes the test function and relevant fixtures/hooks that share the test timeout.

Example:

```ts
test('login', async ({ page }) => {
  // entire test has the configured test timeout
});
```

### Mental model

```text
ONE TEST
└── maximum allowed time = timeout
```

### MUST REMEMBER

`timeout` ≠ `expect.timeout`

They solve different problems.

---

# 9. `expect.timeout`

```ts
expect: {
  timeout: 5_000,
}
```

Controls how long Playwright's async `expect()` matchers wait for their condition.

Default:

```text
5 seconds
```

Example:

```ts
await expect(page.getByText('Welcome')).toBeVisible();
```

The assertion can wait according to the expect timeout.

### Mental model

```text
timeout
   = how long the TEST can run

expect.timeout
   = how long an EXPECTATION waits
```

---

# 10. `globalTimeout`

```ts
globalTimeout: 60 * 60 * 1000
```

Maximum time for the **entire test suite/run**.

Default:

```text
No global timeout
```

Mental model:

```text
Entire Playwright run
└── globalTimeout
```

This is especially useful in CI to stop a broken test run from consuming resources indefinitely.

---

# 11. Action timeout

Configured inside `use`:

```ts
use: {
  actionTimeout: 10_000,
}
```

Controls the timeout for individual Playwright actions such as:

```ts
click()
fill()
check()
selectOption()
```

Default:

```text
0 = no action timeout
```

Example:

```ts
await page.getByRole('button', { name: 'Login' }).click();
```

If the click does not complete within the configured action timeout, it fails.

### Important

This is NOT the same as test timeout.

```text
test timeout
    ↓
whole test

action timeout
    ↓
individual Playwright action
```

---

# 12. Navigation timeout

Configured inside `use`:

```ts
use: {
  navigationTimeout: 30_000,
}
```

Controls navigation-related operations.

Example:

```ts
await page.goto('/login');
```

Default:

```text
0 = no navigation timeout
```

### Mental model

```text
actionTimeout
    → click/fill/check/etc.

navigationTimeout
    → navigation operations

timeout
    → whole test

expect.timeout
    → expect assertion waiting

globalTimeout
    → entire test run
```

---

# 13. Timeout Cheat Sheet

| Timeout | Where | Controls |
|---|---|---|
| `timeout` | top level | Individual test |
| `expect.timeout` | `expect` | Async expect assertions |
| `globalTimeout` | top level | Entire test run |
| `use.actionTimeout` | `use` | Individual Playwright actions |
| `use.navigationTimeout` | `use` | Navigation actions |

Do not mix them up.

---

# 14. Retries — `retries`

```ts
retries: 2
```

Maximum retry attempts for failed tests.

Example:

```text
Original run
   ↓
FAILED
   ↓
Retry #1
   ↓
FAILED
   ↓
Retry #2
```

`retries: 2` means **2 retry attempts after the initial attempt**.

Common CI pattern:

```ts
retries: process.env.CI ? 2 : 0
```

Meaning:

```text
Local → no retries
CI    → retry failed tests up to 2 times
```

### MUST REMEMBER

Retries are useful for handling intermittent failures, but they should not be used to hide real product/test defects.

---

# 15. Workers

```ts
workers: 4
```

Controls the maximum number of concurrent worker processes for the test run.

Example:

```text
workers = 4

Worker 1 → test A
Worker 2 → test B
Worker 3 → test C
Worker 4 → test D
```

More workers can increase parallel execution, but resource-heavy tests or shared test data may require fewer workers.

Current Playwright terminology refers to **worker processes**, not JavaScript threads.

### MUST REMEMBER

```text
workers = how many worker processes may execute concurrently
```

---

# 16. `fullyParallel`

```ts
fullyParallel: true
```

By default, Playwright runs test files in parallel, while tests within one file run in order.

`fullyParallel: true` allows all tests in all files to run in parallel.

Conceptually:

```text
Default:

File A
  test 1 → test 2 → test 3

File B
  test 1 → test 2 → test 3

Files can run in parallel.


fullyParallel: true

test 1 ─┐
test 2 ─┼→ can execute concurrently
test 3 ─┤
test 4 ─┘
```

Actual concurrency is still constrained by available workers.

### Important

`fullyParallel` does NOT mean infinite parallelism.

Workers still control the maximum concurrency.

---

# 17. `repeatEach`

```ts
repeatEach: 3
```

Repeats each test multiple times.

Useful for investigating flaky behavior.

Example:

```text
testLogin

Run 1
Run 2
Run 3
```

Do not confuse:

```text
retries
    = repeat only after failure

repeatEach
    = intentionally repeat each test
```

---

# 18. `maxFailures`

Limits how many test failures can occur before Playwright stops the test run.

Example:

```ts
maxFailures: 10
```

Useful when a large suite is running and you do not want to continue after many failures.

---

# 19. `forbidOnly`

Very useful in CI.

```ts
forbidOnly: !!process.env.CI
```

If someone accidentally leaves:

```ts
test.only(...)
```

in the code, CI fails instead of silently running only that test.

Mental model:

```text
Local:
test.only → allowed

CI:
test.only → fail the run
```

### MUST KNOW FOR INTERVIEWS

This is a common CI safety setting.

---

# 20. `use` — The Most Important Section

This is where you configure browser/browser-context behavior shared by tests.

Example:

```ts
use: {
  browserName: 'chromium',
  headless: true,
  baseURL: 'https://myapp.com',
  viewport: { width: 1280, height: 720 },
  screenshot: 'only-on-failure',
  trace: 'on-first-retry',
  video: 'on-first-retry',
}
```

Think:

```text
use
 ↓
How should my test browser/context behave?
```

---

# 21. `browserName`

```ts
use: {
  browserName: 'chromium',
}
```

Supported browser names include:

```text
chromium
firefox
webkit
```

Default:

```text
chromium
```

Important:

`browserName` chooses the browser engine.

---

# 22. `headless`

```ts
use: {
  headless: true,
}
```

Controls whether the browser UI is visible.

```text
true  → browser UI is not shown
false → browser UI is shown
```

Default:

```text
true
```

Common:

```ts
headless: false
```

while debugging locally.

---

# 23. `baseURL`

```ts
use: {
  baseURL: 'https://myapp.com',
}
```

Then instead of:

```ts
await page.goto('https://myapp.com/login');
```

you can write:

```ts
await page.goto('/login');
```

Playwright resolves the relative URL using `baseURL`.

### MUST REMEMBER

```text
baseURL = common application root URL
```

Very common in real frameworks.

---

# 24. `viewport`

```ts
use: {
  viewport: {
    width: 1280,
    height: 720,
  },
}
```

Controls the browser page viewport.

Default is:

```text
1280 × 720
```

This is useful for consistent test execution.

---

# 25. `storageState`

```ts
use: {
  storageState: 'auth.json',
}
```

Loads saved browser storage state such as authentication-related cookies/local storage.

Common real-world use:

```text
Login once
   ↓
Save authentication state
   ↓
Tests reuse authenticated state
```

This can reduce repeated login work.

### MUST KNOW

`storageState` is heavily used in authentication frameworks.

---

# 26. `screenshot`

```ts
use: {
  screenshot: 'only-on-failure',
}
```

Automatic screenshot behavior.

Common modes:

```text
'off'
'on'
'only-on-failure'
'on-first-failure'
```

Typical framework choice:

```ts
screenshot: 'only-on-failure'
```

---

# 27. `video`

```ts
use: {
  video: 'on-first-retry',
}
```

Controls automatic video recording.

Common modes include:

```text
'off'
'on'
'retain-on-failure'
'on-first-retry'
'on-all-retries'
```

A common choice:

```ts
video: 'on-first-retry'
```

---

# 28. `trace`

```ts
use: {
  trace: 'on-first-retry',
}
```

Controls Playwright Trace recording.

Trace is extremely useful for debugging failures because it can provide a timeline of test execution, screenshots/DOM snapshots and other execution details.

Common modes:

```text
'off'
'on'
'retain-on-failure'
'on-first-retry'
```

Typical choice:

```ts
trace: 'on-first-retry'
```

### MUST KNOW

For debugging Playwright failures:

```text
Trace > Video > Screenshot
```

This is not a ranking of test quality; it is a practical way to remember that Trace generally gives the richest Playwright-specific debugging information.

---

# 29. `testIdAttribute`

By default, Playwright's `getByTestId()` uses:

```html
data-testid
```

You can configure another attribute:

```ts
use: {
  testIdAttribute: 'data-test',
}
```

Then:

```ts
page.getByTestId('login-button')
```

looks for:

```html
data-test="login-button"
```

Useful when an application uses a custom test-id attribute.

---

# 30. Other `use` options you should recognize

You do not need to memorize every option initially, but you should recognize these:

```ts
use: {
  locale: 'en-US',
  timezoneId: 'Asia/Kolkata',
  colorScheme: 'dark',

  geolocation: {
    latitude: 17.4,
    longitude: 78.4,
  },

  permissions: ['geolocation'],

  ignoreHTTPSErrors: true,

  extraHTTPHeaders: {
    'x-test-user': 'qa',
  },

  userAgent: 'custom-user-agent',

  offline: false,
}
```

### Mental grouping

```text
locale          → language/locale
timezoneId      → timezone
colorScheme     → light/dark
geolocation     → location
permissions     → browser permissions
ignoreHTTPSErrors → ignore certificate errors
extraHTTPHeaders  → headers for requests
offline         → simulate offline mode
```

---

# 31. `channel`

Example:

```ts
use: {
  channel: 'chrome',
}
```

Selects a browser channel such as Chrome or Edge where supported.

Do not confuse:

```text
browserName = browser engine
channel     = specific browser channel/distribution
```

---

# 32. `devices`

Playwright provides predefined device configurations.

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'Mobile Chrome',
      use: {
        ...devices['Pixel 5'],
      },
    },
  ],
});
```

A device preset can configure multiple browser/context characteristics together.

This is commonly used with projects.

---

# 33. Projects — One of the Most Important Config Concepts

`projects` lets you run the same or different tests using different configurations.

Example:

```ts
projects: [
  {
    name: 'chromium',
    use: {
      browserName: 'chromium',
    },
  },

  {
    name: 'firefox',
    use: {
      browserName: 'firefox',
    },
  },

  {
    name: 'webkit',
    use: {
      browserName: 'webkit',
    },
  },
]
```

Conceptually:

```text
                 SAME TESTS
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      Chromium    Firefox     WebKit
```

---

# 34. Why projects are useful

Projects are not only for browsers.

You can use them for:

```text
Browser
Mobile/Desktop
Smoke/Regression
Different environments
Different test directories
Different authentication states
Different configurations
```

Example:

```ts
projects: [
  {
    name: 'Smoke',
    testMatch: '**/*.smoke.spec.ts',
  },

  {
    name: 'Regression',
    testMatch: '**/*.regression.spec.ts',
  },
]
```

---

# 35. Project-specific `use`

Global:

```ts
use: {
  baseURL: 'https://qa.myapp.com',
}
```

Project-specific:

```ts
projects: [
  {
    name: 'Firefox',
    use: {
      browserName: 'firefox',
    },
  },
]
```

Think:

```text
Global config
     ↓
shared defaults

Project config
     ↓
project-specific overrides
```

---

# 36. Project-specific settings

A project can have its own:

```ts
{
  name: 'Smoke',

  testDir: './smoke-tests',

  testMatch: '**/*.smoke.spec.ts',

  retries: 1,

  workers: 2,

  use: {
    browserName: 'chromium',
  },
}
```

This is very important for real framework design.

---

# 37. Project Dependencies

Example:

```ts
projects: [
  {
    name: 'setup',
    testMatch: /global\.setup\.ts/,
  },

  {
    name: 'chromium',
    dependencies: ['setup'],
  },
]
```

Meaning:

```text
setup
  ↓
chromium tests
```

The dependent project waits for its dependency.

This is useful when setup itself is represented as a Playwright project/test.

---

# 38. Project Teardown

A project can specify a teardown project.

Conceptually:

```text
setup
  ↓
tests
  ↓
teardown
```

Example:

```ts
{
  name: 'setup',
  testMatch: /global\.setup\.ts/,
  teardown: 'teardown',
}
```

This is different from simply having an `afterAll()` hook inside a test file.

---

# 39. `reporter`

Controls how test results are reported.

Example:

```ts
reporter: 'html'
```

Common built-in reporters include:

```text
list
dot
line
html
json
junit
blob
github
```

You can also configure multiple reporters.

Example:

```ts
reporter: [
  ['list'],
  ['html', { open: 'never' }],
]
```

---

# 40. HTML reporter

Typical:

```ts
reporter: [
  ['html', { open: 'never' }],
]
```

After execution:

```bash
npx playwright show-report
```

HTML reports are commonly useful for local debugging and CI artifacts.

---

# 41. `outputDir`

```ts
outputDir: 'test-results'
```

Controls where Playwright stores test execution artifacts.

Examples:

```text
screenshots
videos
traces
attachments
other test output
```

Do not confuse:

```text
outputDir
   = test execution artifacts

HTML reporter outputFolder
   = HTML report files
```

They are related to test results but are not the same setting.

---

# 42. `webServer`

Useful when your application must be started before tests.

Example:

```ts
webServer: {
  command: 'npm run start',
  url: 'http://localhost:3000',
  reuseExistingServer: !process.env.CI,
}
```

Conceptually:

```text
Playwright starts app
       ↓
app becomes available
       ↓
tests start
```

Common for local applications.

Usually paired with:

```ts
use: {
  baseURL: 'http://localhost:3000',
}
```

---

# 43. `globalSetup` and `globalTeardown`

Example:

```ts
globalSetup: require.resolve('./global-setup'),
globalTeardown: require.resolve('./global-teardown'),
```

These configure functions that run before/after the test suite.

### Important distinction

Do not confuse:

```text
globalSetup
    = suite-level setup

test.beforeAll()
    = setup for a test file/scope
```

Also remember that Playwright now supports **project dependencies**, which can be preferable when setup needs to behave like a test project and produce normal test/report/trace artifacts.

---

# 44. `grep`

Run only tests whose titles/tags match a pattern.

Example:

```ts
grep: /smoke/
```

Then tests matching the pattern are selected.

Useful for test tagging/filtering.

---

# 45. `grepInvert`

Opposite of `grep`.

```ts
grepInvert: /slow/
```

Means matching tests are excluded.

---

# 46. `shard`

Used to split a test suite across multiple machines/jobs.

Example:

```ts
shard: {
  total: 4,
  current: 1,
}
```

Conceptually:

```text
1000 tests
     ↓
 ┌───┬───┬───┬───┐
 J1  J2  J3  J4
```

Each shard handles part of the test suite.

Very useful in CI at scale.

---

# 47. `preserveOutput`

Controls preservation of test output.

Possible values include:

```text
always
never
failures-only
```

This is useful when managing test artifacts/output behavior.

Not a first-day setting.

---

# 48. Configuration vs Test Code

A very important distinction:

### Config

```ts
use: {
  baseURL: 'https://qa.myapp.com',
  screenshot: 'only-on-failure',
}
```

### Test

```ts
test('login', async ({ page }) => {
  await page.goto('/login');
});
```

Config defines **defaults and execution rules**.

Test code defines **what the test actually does**.

---

# 49. Configuration Scope / Override Mental Model

Think:

```text
GLOBAL CONFIG
      ↓
PROJECT CONFIG
      ↓
FILE / test.use()
      ↓
INDIVIDUAL ACTION / ASSERTION OPTIONS
```

Example:

```ts
// config
use: {
  baseURL: 'https://qa.myapp.com'
}
```

Project:

```ts
projects: [
  {
    name: 'staging',
    use: {
      baseURL: 'https://staging.myapp.com',
    },
  },
]
```

File:

```ts
test.use({
  baseURL: 'https://special.myapp.com',
});
```

Individual operation can also have its own timeout:

```ts
await page.goto('/login', {
  timeout: 10_000,
});
```

The exact precedence depends on the option, so do not memorize one universal "highest wins" rule for every Playwright setting.

---

# 50. A Practical Real-Project Config

A reasonable framework-style example:

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

  // -------------------------
  // TEST DISCOVERY
  // -------------------------
  testDir: './tests',

  // -------------------------
  // EXECUTION
  // -------------------------
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 2 : undefined,

  // -------------------------
  // TIMEOUT
  // -------------------------
  timeout: 30_000,

  expect: {
    timeout: 5_000,
  },

  // -------------------------
  // REPORTING
  // -------------------------
  reporter: [
    ['list'],
    ['html', { open: 'never' }],
  ],

  // -------------------------
  // BROWSER / CONTEXT
  // -------------------------
  use: {

    baseURL: 'https://qa.myapp.com',

    browserName: 'chromium',

    headless: true,

    screenshot: 'only-on-failure',

    trace: 'on-first-retry',

    video: 'on-first-retry',

  },

  // -------------------------
  // PROJECTS
  // -------------------------
  projects: [

    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
      },
    },

    {
      name: 'firefox',
      use: {
        ...devices['Desktop Firefox'],
      },
    },

  ],

});
```

Do not memorize this entire file.

Understand **why each section exists**.

---

# 51. MUST Know vs SHOULD Know vs LATER

## MUST KNOW

These should become automatic knowledge:

```text
testDir
testMatch
testIgnore

timeout
expect.timeout
retries
workers
fullyParallel

use
  browserName
  headless
  baseURL
  viewport
  storageState
  screenshot
  video
  trace

projects
reporter
outputDir
webServer

forbidOnly
```

---

## SHOULD KNOW

Know the purpose and recognize them in a framework:

```text
repeatEach
maxFailures
globalTimeout

testIdAttribute
channel

locale
timezoneId
colorScheme
geolocation
permissions
ignoreHTTPSErrors
extraHTTPHeaders
offline

globalSetup
globalTeardown

grep
grepInvert

dependencies
teardown

shard
```

---

## LEARN LATER

Do not spend learning time memorizing these now:

```text
snapshotPathTemplate
captureGitInfo
metadata
tag
build
preserveOutput
connectOptions
launchOptions
proxy
serviceWorkers
advanced tracing configuration
custom reporter implementation
```

You should recognize them when you see them and know where to look them up when needed.

---

# 52. Most Important Interview Questions

## Q1. What is `playwright.config.ts`?

**Answer:**

> `playwright.config.ts` is the central Playwright Test configuration file. It controls test discovery, execution, timeouts, retries, browser/context defaults, projects, reporting and other test-runner behavior.

---

## Q2. What is the difference between top-level config and `use`?

**Answer:**

> Top-level options configure the Playwright Test runner, while `use` configures browser and browser-context defaults used by tests.

---

## Q3. What is `baseURL`?

**Answer:**

> `baseURL` defines the common application URL so tests can use relative URLs such as `page.goto('/login')`.

---

## Q4. What is the difference between `timeout` and `expect.timeout`?

**Answer:**

> `timeout` controls the maximum duration of a test, while `expect.timeout` controls how long an async Playwright assertion waits for its expected condition.

---

## Q5. What is the difference between `retries` and `repeatEach`?

**Answer:**

> `retries` reruns a failed test, while `repeatEach` intentionally runs every test multiple times.

---

## Q6. What is `workers`?

**Answer:**

> `workers` controls the maximum number of worker processes that can execute tests concurrently.

---

## Q7. What is `fullyParallel`?

**Answer:**

> `fullyParallel` allows tests within test files to run in parallel as well, subject to the available workers.

---

## Q8. Why use `projects`?

**Answer:**

> Projects allow the same test suite or different subsets of tests to run with different configurations, such as Chromium, Firefox, WebKit, mobile devices, or different test groups.

---

## Q9. Why use `trace: 'on-first-retry'`?

**Answer:**

> It records a Playwright trace for the first retry of a failed test, providing detailed execution information useful for debugging without recording traces for every normal run.

---

## Q10. What does `forbidOnly` do?

**Answer:**

> It prevents a CI run from accidentally succeeding with only a `test.only()` test being executed.

---

# 53. Common Beginner Mistakes

### Mistake 1

Putting runner settings inside `use`:

```ts
use: {
  workers: 4 // ❌
}
```

`workers` is a test-runner configuration option, not a normal `use` browser/context option.

---

### Mistake 2

Thinking `timeout` means every Playwright operation gets that timeout.

Not exactly.

```text
timeout
    → test timeout

expect.timeout
    → expect timeout

actionTimeout
    → action timeout

navigationTimeout
    → navigation timeout
```

---

### Mistake 3

Thinking `workers: 4` means four tests always run simultaneously.

Not necessarily.

The actual parallel execution also depends on how tests are scheduled and whether the suite has enough parallelizable work.

---

### Mistake 4

Thinking `fullyParallel: true` means unlimited parallelism.

Wrong.

Workers still limit concurrency.

---

### Mistake 5

Using retries to hide flaky tests.

Retries can help diagnose/intermittent failures, but a repeatedly failing test still needs investigation.

---

### Mistake 6

Putting every option into `playwright.config.ts`.

Do not.

Configure only what your framework actually needs.

---

# 54. The 10 Things to Remember Forever

If you forget everything else, remember this:

```text
1. testDir
   → WHERE Playwright searches for tests.

2. testMatch
   → WHICH files to include.

3. testIgnore
   → WHICH matching files to exclude.

4. timeout
   → MAX time for one test.

5. expect.timeout
   → MAX time an async expect waits.

6. workers
   → HOW MANY worker processes can run concurrently.

7. use
   → HOW browser/context should behave.

8. projects
   → DIFFERENT test configurations.

9. reporter
   → HOW test results are reported.

10. webServer
    → START the application before tests when needed.
```

---

# 55. Final Mental Map

```text
                 playwright.config.ts
                          |
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   FIND TESTS         RUN TESTS         BROWSER
        |                 |                 |
   testDir           workers             use
   testMatch         retries               |
   testIgnore        timeout         ┌──────┼─────────┐
                      |               ↓      ↓         ↓
                 fullyParallel    browser  baseURL   trace
                                   headless storage   video
                                      |               screenshot
                                      |
                                  PROJECTS
                                      |
                           ┌──────────┼──────────┐
                           ↓          ↓          ↓
                       Chromium    Firefox    WebKit
                           |
                       REPORTING
                           |
                       reporter
                       outputDir
                           |
                       ENVIRONMENT
                           |
                       webServer
```

---

# 56. One-Line Memory Trick

> **Config decides WHERE tests are, HOW they run, HOW LONG they can run, WHICH browser/environment they use, and HOW results are reported.**

---

# 57. What You Should Practice Next

Create a small config and change only one option at a time:

1. `testDir`
2. `testMatch`
3. `timeout`
4. `expect.timeout`
5. `workers`
6. `fullyParallel`
7. `retries`
8. `baseURL`
9. `headless`
10. `screenshot`
11. `trace`
12. `video`
13. `projects`
14. `reporter`
15. `webServer`

Do not memorize syntax first.

For every option ask:

```text
WHAT does it control?
WHERE is it configured?
WHY would a real project need it?
```

That is the fastest way to make `playwright.config.ts` understandable rather than memorized.

---

## Official references

- Playwright Configuration: https://playwright.dev/docs/test-configuration
- Playwright `use` options: https://playwright.dev/docs/test-use-options
- Playwright TestConfig API: https://playwright.dev/docs/api/class-testconfig
- Playwright TestProject API: https://playwright.dev/docs/api/class-testproject
- Playwright TestOptions API: https://playwright.dev/docs/api/class-testoptions
- Playwright Timeouts: https://playwright.dev/docs/test-timeouts
- Playwright Reporters: https://playwright.dev/docs/test-reporters
- Playwright Projects: https://playwright.dev/docs/test-projects
