# Playwright Test Runner, Test Discovery & CLI

## Mental Map

**Command → Test Runner → `testDir` → Test Files → Tests → Execution → Results**

## 1. What Is Playwright Test Runner?

The Playwright Test Runner is responsible for running Playwright tests written using `@playwright/test`.

It can:

- discover test files
- read the tests inside them
- prepare fixtures such as `browser`, `context`, and `page`
- execute tests
- handle retries, projects, workers, and other configuration
- collect test results
- generate reports

---

## 2. How Does `npx playwright test` Know Where the Tests Are?

When you run:

```bash
npx playwright test
```

the Playwright Test Runner loads the Playwright configuration and uses the configured `testDir` to discover tests.

Example:

```ts
testDir: './tests'
```

This tells Playwright:

> Look for test files under the `tests` directory.

The exact files selected also depend on Playwright's test-file matching rules and any CLI filters.

### Important Correction

It is not mandatory that test files always live in a folder named `tests`.

You can configure another directory:

```ts
testDir: './automation-tests'
```

Also, Playwright Test can run both JavaScript and TypeScript test files when they match the configured test-file patterns.

---

## 3. Typical Test Structure

```text
project
│
├── playwright.config.ts
│
└── tests
    ├── login.spec.ts
    ├── search.spec.ts
    └── checkout.spec.ts
```

Example:

```ts
import { test, expect } from '@playwright/test';

test('Login test', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle(/Example/);
});
```

---

## 4. Run All Tests

```bash
npx playwright test
```

This runs the tests discovered by the Test Runner according to the configuration and CLI options.

---

## 5. Useful CLI Commands

| Requirement | Command |
|---|---|
| Run all tests | `npx playwright test` |
| Run specific spec | `npx playwright test tests/login.spec.ts` |
| Run specific test by title | `npx playwright test -g "test name"` |
| Run test from a line | `npx playwright test tests/login.spec.ts:25` |
| Run directory | `npx playwright test tests/login` |
| Run multiple specs | `npx playwright test tests/login.spec.ts tests/signup.spec.ts` |
| Run last failed | `npx playwright test --last-failed` |
| Run headed | `npx playwright test --headed` |
| Run headless | `npx playwright test` |
| Open UI Mode | `npx playwright test --ui` |
| Debug | `npx playwright test --debug` |

---

## 6. API Logs

To enable Playwright API debug logging in PowerShell:

```powershell
$env:DEBUG="pw:api"
npx playwright test
```

This is useful when you want to understand what Playwright is doing during execution.

---

## Interview Answer

> When we run `npx playwright test`, the Playwright Test Runner loads the configuration, uses `testDir` and test-file matching rules to discover the tests, and then executes them according to the configured projects, workers, fixtures, and other settings.

## Memory Trick

**`testDir` = Where should Playwright look?**

**CLI path/filter = Which tests should Playwright run?**

**Test Runner = Who discovers and executes them?**
