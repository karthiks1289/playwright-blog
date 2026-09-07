# Playwright Multi-Browser Support — Complete Guide

## 1. Overview

Playwright can automate applications across three main browser engines:

- **Chromium**
- **Firefox**
- **WebKit**

Playwright can also launch certain **branded Chromium-based browsers**, such as Google Chrome and Microsoft Edge.

### Mental Map

```text
                         PLAYWRIGHT
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          Chromium         Firefox          WebKit
              |               |               |
       +------+------+         |               |
       |             |         |               |
       v             v         v               v
    Chrome         Edge    Playwright      Safari-like
                           Firefox          testing
```

> **Important:** WebKit is the browser engine used by Safari. Playwright's WebKit is **not Safari itself**.

---

# 2. Playwright Browser Binaries

When we install Playwright and install its browsers, Playwright downloads its supported browser binaries.

| Binary | Purpose |
|---|---|
| Chromium | Playwright's bundled Chromium browser |
| Firefox | Playwright's patched Firefox build |
| WebKit | Playwright's bundled WebKit build |

## Important Correction

Do **not** describe the Firefox binary as a "Nightly Build."

A better description is:

> Playwright uses a Playwright-specific patched Firefox build.

Similarly, do **not** say that Playwright copies Safari's source code and creates WebKit.

A better description is:

> Playwright uses its own bundled WebKit build for Safari-like browser testing.

---

# 3. Chromium, Chrome and Edge — What Is the Difference?

This is an important concept.

### Playwright Chromium

```typescript
const browser = await chromium.launch({
  headless: false
});
```

This launches the Chromium browser bundled with Playwright.

### Google Chrome

```typescript
const browser = await chromium.launch({
  headless: false,
  channel: 'chrome'
});
```

This launches the installed branded Google Chrome.

### Microsoft Edge

```typescript
const browser = await chromium.launch({
  headless: false,
  channel: 'msedge'
});
```

This launches the installed Microsoft Edge.

### Mental Map

```text
chromium.launch()
       |
       +----> Playwright bundled Chromium


chromium.launch({ channel: 'chrome' })
       |
       +----> Installed Google Chrome


chromium.launch({ channel: 'msedge' })
       |
       +----> Installed Microsoft Edge
```

---

# 4. Firefox

Firefox is not Chromium-based.

Playwright provides a Playwright-specific Firefox build.

```typescript
import { firefox, Browser, Page, test } from '@playwright/test';

test('Launch Firefox', async () => {
  const browser: Browser = await firefox.launch({
    headless: false
  });

  const page: Page = await browser.newPage();

  await page.goto(
    'https://naveenautomationlabs.com/opencart/index.php?route=account/login'
  );

  await page.pause();

  await browser.close();
});
```

### Remember

```text
Firefox
   ↓
Not Chromium
   ↓
firefox.launch()
   ↓
Playwright Firefox build
```

---

# 5. WebKit

WebKit is not Safari.

WebKit is the browser engine associated with Safari.

Playwright provides its own WebKit build for cross-browser testing.

```typescript
import { webkit, Browser, Page, test } from '@playwright/test';

test('Launch WebKit', async () => {
  const browser: Browser = await webkit.launch({
    headless: false
  });

  const page: Page = await browser.newPage();

  await page.goto(
    'https://naveenautomationlabs.com/opencart/index.php?route=account/login'
  );

  await page.pause();

  await browser.close();
});
```

### Remember

```text
Safari
   ↓
Uses WebKit engine

Playwright
   ↓
Provides its own WebKit build
   ↓
Used for Safari-like cross-browser testing
```

---

# 6. Ways to Interact With Browsers

There are two common approaches.

## Way 1 — Configure Browsers in `playwright.config.ts`

This is generally the preferred approach for a real Playwright test framework.

Example:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome']
      }
    },

    {
      name: 'firefox',
      use: {
        ...devices['Desktop Firefox']
      }
    },

    {
      name: 'webkit',
      use: {
        ...devices['Desktop Safari']
      }
    }
  ]
});
```

### Mental Map

```text
playwright.config.ts
        |
        v
     projects
        |
   +----+----+----+
   |         |    |
   v         v    v
Chromium  Firefox WebKit
   |         |    |
   +---------+----+
            |
            v
       Same tests
```

The same test can be executed against multiple browser projects.

---

# 7. Way 2 — Launch Browser Programmatically

We can also launch a browser directly from the test code.

### Basic Flow

```text
Create Browser
      ↓
Create Page
      ↓
Navigate
      ↓
Interact with Application
      ↓
Close Browser
```

Example:

```typescript
import { Browser, chromium, Page, test } from '@playwright/test';

test('Launch Chromium', async () => {
  const browser: Browser = await chromium.launch({
    headless: false
  });

  const page: Page = await browser.newPage();

  await page.goto(
    'https://naveenautomationlabs.com/opencart/index.php?route=account/login'
  );

  await page.pause();

  await browser.close();
});
```

---

# 8. Complete Program — Multiple Browsers

The following example demonstrates Chromium, Chrome, Edge, Firefox and WebKit.

```typescript
import {
  Browser,
  chromium,
  firefox,
  Page,
  test,
  webkit
} from '@playwright/test';

const applicationUrl =
  'https://naveenautomationlabs.com/opencart/index.php?route=account/login';

test('Launch Playwright Chromium', async () => {
  const browser: Browser = await chromium.launch({
    headless: false
  });

  const page: Page = await browser.newPage();

  await page.goto(applicationUrl);
  await page.pause();

  await browser.close();
});

test('Launch Google Chrome', async () => {
  const browser: Browser = await chromium.launch({
    headless: false,
    channel: 'chrome'
  });

  const page: Page = await browser.newPage();

  await page.goto(applicationUrl);
  await page.pause();

  await browser.close();
});

test('Launch Microsoft Edge', async () => {
  const browser: Browser = await chromium.launch({
    headless: false,
    channel: 'msedge'
  });

  const page: Page = await browser.newPage();

  await page.goto(applicationUrl);
  await page.pause();

  await browser.close();
});

test('Launch Firefox', async () => {
  const browser: Browser = await firefox.launch({
    headless: false
  });

  const page: Page = await browser.newPage();

  await page.goto(applicationUrl);
  await page.pause();

  await browser.close();
});

test('Launch WebKit', async () => {
  const browser: Browser = await webkit.launch({
    headless: false
  });

  const page: Page = await browser.newPage();

  await page.goto(applicationUrl);
  await page.pause();

  await browser.close();
});
```

---

# 9. `channel` vs `executablePath`

This is an important interview and real-project concept.

## `channel`

Use a supported browser channel when Playwright provides one.

```typescript
const browser = await chromium.launch({
  headless: false,
  channel: 'chrome'
});
```

or:

```typescript
const browser = await chromium.launch({
  headless: false,
  channel: 'msedge'
});
```

### Mental Model

```text
channel
   ↓
"Playwright knows this browser channel"
   ↓
Launch the installed browser
```

---

# 10. `executablePath`

`executablePath` allows us to explicitly specify the browser executable.

Example:

```typescript
const browser = await chromium.launch({
  headless: false,
  executablePath: 'C:\\Path\\To\\browser.exe'
});
```

For example, if a Chromium-based browser does not have a convenient supported Playwright channel, an explicit executable path can be used where appropriate.

### Mental Model

```text
executablePath
      ↓
"Here is the exact browser executable"
      ↓
Playwright launches that executable
```

## Important

Do not write:

> Opera and Brave can never use `channel`.

That is too absolute.

Use:

> For a Chromium-based browser without a suitable Playwright browser channel, `executablePath` can be used to point Playwright to the browser executable.

---

# 11. Browser Launch Cheat Sheet

| Requirement | Playwright code |
|---|---|
| Playwright Chromium | `chromium.launch()` |
| Google Chrome | `chromium.launch({ channel: 'chrome' })` |
| Microsoft Edge | `chromium.launch({ channel: 'msedge' })` |
| Playwright Firefox | `firefox.launch()` |
| Playwright WebKit | `webkit.launch()` |
| Specific browser executable | `chromium.launch({ executablePath: '...' })` |

---

# 12. Important Terminology

| Incorrect / Misleading | Better wording |
|---|---|
| CFT is every Chromium browser Playwright installs | Playwright provides a bundled Chromium build |
| Nightly Build = Playwright Firefox | Playwright provides a patched Firefox build |
| WebKit = Safari | WebKit is the engine associated with Safari; Playwright provides its own WebKit build |
| Playwright copies Safari source code | Playwright provides a bundled WebKit build |
| Playwright cannot automate Chrome/Edge | Playwright can automate supported branded Chromium browsers |
| Opera/Brave can never use channels | Use a supported channel if available; otherwise `executablePath` can be used |

---

# 13. Why Multiple Browsers Matter in Testing

A web application can behave differently across browser engines.

For example:

```text
Your Web Application
        |
        +---- Chromium → Test
        |
        +---- Firefox  → Test
        |
        +---- WebKit   → Test
```

A test passing in Chromium does not automatically prove that the application behaves identically in Firefox or WebKit.

Cross-browser testing helps identify browser-specific issues.

---

# 14. Real Project Recommendation

For an enterprise Playwright framework, prefer browser configuration through `playwright.config.ts`.

Example:

```typescript
projects: [
  {
    name: 'chromium',
    use: { ...devices['Desktop Chrome'] }
  },
  {
    name: 'firefox',
    use: { ...devices['Desktop Firefox'] }
  },
  {
    name: 'webkit',
    use: { ...devices['Desktop Safari'] }
  }
]
```

Then tests remain browser-independent.

```text
                 Test
                  |
        +---------+---------+
        |         |         |
        v         v         v
    Chromium   Firefox    WebKit
```

This is cleaner than manually launching a different browser inside every test.

---

# 15. Commands to Run the Tests

Run all tests:

```bash
npx playwright test
```

Run a specific test file:

```bash
npx playwright test tests/browser.spec.ts
```

Run a specific test by title:

```bash
npx playwright test -g "Launch Microsoft Edge"
```

Run a specific project:

```bash
npx playwright test --project=chromium
```

```bash
npx playwright test --project=firefox
```

```bash
npx playwright test --project=webkit
```

Run with the browser visible:

```bash
npx playwright test --headed
```

Open Playwright UI mode:

```bash
npx playwright test --ui
```

---

# 16. Common Mistake — Test File Not Found

If you run:

```bash
npx playwright test tests/browser.spec.ts -g "Launching msedge"
```

and receive:

```text
Error: No tests found.
```

Check:

1. Does the file actually exist?
2. Is the test title exactly `"Launching msedge"`?
3. Is the file included by `testDir` / test matching rules?
4. Are you using the correct filename and path?

For example, if your test is:

```typescript
test('Launch Microsoft Edge', async () => {
```

then run:

```bash
npx playwright test tests/browser.spec.ts -g "Launch Microsoft Edge"
```

The `-g` value is matched against the test title.

---

# 17. Final Mental Map

```text
                         PLAYWRIGHT
                              |
                +-------------+-------------+
                |             |             |
                v             v             v
             Chromium       Firefox       WebKit
                |             |             |
        +-------+-------+     |             |
        |               |     |             |
        v               v     v             v
     Chrome           Edge  Firefox      Safari-like
                                  testing
```

### Launching

```text
Chromium
   → chromium.launch()

Chrome
   → chromium.launch({ channel: 'chrome' })

Edge
   → chromium.launch({ channel: 'msedge' })

Firefox
   → firefox.launch()

WebKit
   → webkit.launch()

Specific executable
   → chromium.launch({ executablePath: '...' })
```

### Configuration

```text
playwright.config.ts
        ↓
     projects
        ↓
Chromium / Firefox / WebKit
        ↓
     Same tests
```

---

# 18. Interview-Ready Answer

### Question: How does Playwright support multiple browsers?

> Playwright supports three browser engines: Chromium, Firefox and WebKit. Playwright provides its own browser builds for these engines. For Chromium-based branded browsers such as Chrome and Microsoft Edge, Playwright can launch supported installed browser channels using the `channel` option. Where an appropriate channel is not available, `executablePath` can be used to point to a specific browser executable. In a real automation framework, I would normally configure Chromium, Firefox and WebKit as projects in `playwright.config.ts` so the same test suite can run across multiple browsers.

---

# 19. One-Line Memory Trick

> **3 engines: Chromium + Firefox + WebKit | Branded Chromium: channel | Specific `.exe`: executablePath | Framework: projects in config**

