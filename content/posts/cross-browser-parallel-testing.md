# Playwright Parallel & Cross-Browser Testing

## Mental Map

**`playwright.config.ts` → `projects[]` → Test Runner → Browser combinations → Parallel execution**

## 1. Cross-Browser Testing

Playwright can run the same test against multiple browser configurations using the `projects` array in `playwright.config.ts`.

Example:

```ts
projects: [
  {
    name: 'chromium',
    use: { browserName: 'chromium' }
  },
  {
    name: 'firefox',
    use: { browserName: 'firefox' }
  },
  {
    name: 'webkit',
    use: { browserName: 'webkit' }
  }
]
```

The same test can therefore be executed once for each enabled project.

### Important

The number of browser/project configurations determines the **test-browser combinations** that the Test Runner schedules. It does not, by itself, determine the number of parallel workers.

For example:

```text
1 test
×
3 projects
=
3 test executions
```

Whether those executions run at the same time depends on the configured workers and other execution settings.

---

## 2. Parallel Testing

Playwright Test supports parallel execution.

If multiple workers are available, independent tests can be distributed across workers.

```text
Worker 1 → Login test
Worker 2 → Search test
Worker 3 → Checkout test
```

With multiple projects:

```text
                 ┌→ Chromium
Test ────────────┼→ Firefox
                 └→ WebKit
```

The actual scheduling depends on the number of workers and Playwright's execution model.

---

## 3. `channel` Property

The `channel` option is used when you want Playwright to launch a particular browser channel.

Common examples include:

```ts
channel: 'chrome'
channel: 'msedge'
```

`channel` is primarily relevant to Chromium-based branded browser channels.

Do not think of `channel` as a generic way to select any browser.

For Firefox and WebKit, use:

```ts
browserName: 'firefox'
```

or:

```ts
browserName: 'webkit'
```

---

## 4. Running Browsers Such as Opera

Opera is Chromium-based, but it is not selected by simply writing:

```ts
channel: 'opera'
```

Instead, you can launch a locally installed Opera executable using `executablePath`.

Example:

```ts
{
  name: 'opera',
  use: {
    browserName: 'chromium',
    launchOptions: {
      executablePath: 'C:\Path\To\Opera\opera.exe'
    }
  }
}
```

### Important Correction

The property is:

```ts
launchOptions
```

not:

```ts
launchOption
```

Also, `executablePath` should be used carefully because Playwright documents that custom browser executables are not guaranteed to work with every browser/version.

---

## 5. Simple Enterprise Example

```ts
projects: [
  {
    name: 'chrome',
    use: {
      browserName: 'chromium',
      channel: 'chrome'
    }
  },
  {
    name: 'firefox',
    use: {
      browserName: 'firefox'
    }
  },
  {
    name: 'webkit',
    use: {
      browserName: 'webkit'
    }
  }
]
```

One test can become:

```text
Login Test
   │
   ├── Chrome
   ├── Firefox
   └── WebKit
```

## Interview Answer

> Playwright supports cross-browser testing through projects in `playwright.config.ts`. Each project can define a different browser or browser configuration. Playwright Test also supports parallel execution through workers, so independent test executions can run concurrently. The number of projects controls the browser/configuration combinations, while the worker configuration controls how much work can execute in parallel.

## Memory Trick

**Projects = WHAT configurations to test**

**Workers = HOW MUCH can run at the same time**

**Cross-browser = Same test, different browser configurations**

**Parallel = Multiple independent executions at the same time**
