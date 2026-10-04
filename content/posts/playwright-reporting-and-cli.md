# Playwright Reporting & Reporting CLI

## Mental Map

**Test Execution → Results → Reporter → Report Format**

## 1. What Is a Reporter?

A Playwright reporter controls **how test results are presented or generated**.

You can configure reporters in `playwright.config.ts` or select a reporter from the CLI.

---

## 2. Configure Reporters in `playwright.config.ts`

Example:

```ts
reporter: [
  ['list'],
  ['html']
]
```

This configures multiple reporters.

### Important

The reporter configuration is an **array of reporter entries**. Each entry can contain the reporter name and optional configuration.

For example:

```ts
reporter: [
  ['html', { open: 'never' }],
  ['list']
]
```

---

## 3. Configure Reporter from CLI

You can override the reporter for a particular run:

```bash
npx playwright test --reporter=html
```

Examples:

```bash
npx playwright test --reporter=list
npx playwright test --reporter=json
npx playwright test --reporter=junit
```

Multiple reporters can be configured in the configuration file.

---

## 4. Common Reporter Types

### HTML

```text
html
```

Generates an interactive HTML report.

Useful when testers want to visually inspect:

- passed tests
- failed tests
- skipped tests
- duration
- errors
- test details
- attachments

### List

```text
list
```

Displays test execution information in the terminal.

### JSON

```text
json
```

Produces machine-readable JSON test results.

This can be consumed by custom reporting or dashboard systems.

### JUnit

```text
junit
```

Produces XML results commonly consumed by CI/CD and reporting systems.

---

## 5. Example Configuration

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    ['list'],
    ['html']
  ]
});
```

Now one execution can provide terminal output and an HTML report.

---

## Interview Answer

> Playwright reporters determine how test execution results are presented. We can configure reporters in `playwright.config.ts` or specify a reporter from the CLI. Common reporters include HTML, list, JSON, and JUnit. Different reporters are useful for different consumers such as testers, CI/CD systems, or custom dashboards.

## Memory Trick

**Reporter = How do I want to see or consume the results?**

**HTML = Human-friendly**

**List = Console**

**JSON = Machine/custom processing**

**JUnit = CI/reporting integration**
