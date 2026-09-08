# Running Playwright Without the Playwright Test Runner

## Mental Map

**No Test Runner → No automatic test lifecycle → Your code controls the browser lifecycle**

## 1. The Scenario

Suppose you are **not using Playwright Test Runner** and do not want your script to depend on a Playwright Test configuration.

You can still use Playwright directly from a normal JavaScript or TypeScript file.

For example:

```text
login.js
```

You can explicitly launch a browser and perform actions.

---

## 2. What the Test Runner Normally Does

When using `@playwright/test`, the Test Runner provides many conveniences.

It can:

1. discover test files
2. read the tests
3. load Playwright configuration
4. create and manage fixtures
5. provide objects such as `page`
6. execute tests
7. handle retries and projects
8. collect results
9. generate reports
10. manage browser/context/page lifecycle according to the test model

---

## 3. What Changes Without the Test Runner?

With a plain Playwright script, you are responsible for the lifecycle.

You explicitly decide when to:

- launch the browser
- create a context
- create a page
- navigate
- perform actions
- validate results
- close the browser

Example:

```ts
import { chromium, expect } from '@playwright/test';

(async () => {
  const browser = await chromium.launch({
    headless: false,
    channel: 'chrome'
  });

  const page = await browser.newPage();

  await page.goto(
    'https://naveenautomationlabs.com/opencart/index.php?route=account/login'
  );

  console.log(page.url());

  await expect(page).toHaveTitle('Account Login');

  await browser.close();
})();
```

---

## 4. Important Import Distinction

If you use:

```ts
import { test, expect } from '@playwright/test';
```

you are using the Playwright Test package.

If your goal is a standalone Playwright automation script, a clearer approach is to use the core Playwright package:

```ts
import { chromium } from 'playwright';
```

For standalone assertions, use a normal assertion library or your own validation approach.

For example:

```ts
import { chromium } from 'playwright';
import assert from 'node:assert/strict';

(async () => {
  const browser = await chromium.launch({
    headless: false,
    channel: 'chrome'
  });

  const page = await browser.newPage();

  await page.goto('https://example.com');

  assert.match(await page.title(), /Example/);

  await browser.close();
})();
```

---

## 5. Do Not Say ".spec Files Are Mandatory"

A common misunderstanding is:

> ".spec files must always be created under the tests folder."

That is true only as a typical Playwright Test project convention, not as a general requirement for Playwright itself.

For the Test Runner:

- the directory is configurable using `testDir`
- test files must match the configured test-file patterns
- `.spec.ts` is a common naming convention
- `.js` and `.ts` test files can be used when they match the configured patterns

For a standalone script:

```text
login.js
```

can simply be executed as a normal Node.js program.

---

## 6. Test Runner vs Plain Playwright

| Area | Playwright Test Runner | Plain Playwright Script |
|---|---|---|
| Test discovery | Automatic | You control execution |
| `playwright.config.ts` | Used by Test Runner | Not required |
| Fixtures | Built-in | You create/manage objects |
| Browser lifecycle | Managed through test lifecycle | You manage it |
| Test execution | Test Runner | Normal program execution |
| Retries | Built-in | You implement if needed |
| Projects | Built-in | You implement your own logic |
| Reports | Built-in reporter system | You need your own reporting approach |
| `.spec` naming | Common/used for discovery | Not required |
| Parallel execution | Supported by Test Runner | You must design it yourself |

---

## 7. Simple Comparison

### With Test Runner

```text
npx playwright test
        ↓
Test Runner
        ↓
Reads config
        ↓
Discovers tests
        ↓
Creates fixtures
        ↓
Runs tests
        ↓
Collects results
        ↓
Generates report
```

### Without Test Runner

```text
node login.js
        ↓
Your JavaScript program
        ↓
Launch browser
        ↓
Create page
        ↓
Perform actions
        ↓
Validate
        ↓
Close browser
```

## Interview Answer

> If I do not use the Playwright Test Runner, I can still use the Playwright library directly from a normal JavaScript or TypeScript file. In that case, I am responsible for launching the browser, creating the context and page, performing actions and validations, handling errors, and closing the browser. I also do not automatically get the Test Runner features such as test discovery, fixtures, projects, retries, and built-in reporting.

## Memory Trick

**Test Runner = Framework manages the test lifecycle**

**Plain Playwright = I manage the lifecycle**
