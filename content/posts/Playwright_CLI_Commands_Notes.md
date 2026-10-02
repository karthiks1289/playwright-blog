# Playwright CLI Commands — Reading & Interview Notes

> **Purpose:** Practical Playwright CLI commands you should know for daily automation work and SDET interviews.
>
> **Scope:** This note focuses on the `npx playwright ...` CLI used with Playwright Test. It does **not** mix it up with the newer `playwright-cli` browser-agent CLI.

---

## 1. Mental Map

Think of the Playwright CLI as these buckets:

```text
PLAYWRIGHT CLI
│
├── RUN TESTS
│   └── npx playwright test
│
├── DEBUG / DEVELOP
│   ├── --debug
│   ├── --headed
│   ├── --ui
│   └── codegen
│
├── REPORTS / DEBUG ARTIFACTS
│   ├── show-report
│   ├── show-trace
│   └── merge-reports
│
├── BROWSERS
│   ├── install
│   ├── install-deps
│   └── uninstall
│
└── INFORMATION / MAINTENANCE
    ├── --help
    ├── --version
    └── clear-cache
```

### Memory trick

**Run → Debug → Report → Browser → Help**

If you remember these five areas, most interview questions become easy.

---

# 2. The Most Important Command

## `npx playwright test`

Runs Playwright tests.

```bash
npx playwright test
```

### Run all tests

```bash
npx playwright test
```

### Run one spec file

```bash
npx playwright test tests/login.spec.ts
```

### Run multiple files/directories

```bash
npx playwright test tests/login.spec.ts tests/cart.spec.ts
```

or:

```bash
npx playwright test tests/login/ tests/cart/
```

### Run a test at a specific line

```bash
npx playwright test tests/login.spec.ts:42
```

### Run by test title

```bash
npx playwright test -g "valid login"
```

`-g` is the short form of `--grep`.

### Run headed

```bash
npx playwright test --headed
```

Normally Playwright runs tests headless.

### Run a specific project

```bash
npx playwright test --project=chromium
```

The project name comes from `playwright.config.ts`.

### Run only changed tests

```bash
npx playwright test --only-changed=origin/main
```

This is useful in CI/PR workflows when you want to run tests related to changed files.

---

# 3. Important Test-Run Options

These are worth knowing for interviews.

## `--headed`

Runs the browser with the UI visible.

```bash
npx playwright test --headed
```

**Remember:**

```text
headless = browser UI not visible
headed   = browser UI visible
```

---

## `--project`

Runs tests for a specific configured project.

```bash
npx playwright test --project=chromium
```

Example configuration:

```ts
projects: [
  { name: 'chromium', use: { browserName: 'chromium' } },
  { name: 'firefox', use: { browserName: 'firefox' } }
]
```

Then:

```bash
npx playwright test --project=firefox
```

---

## `--grep` / `-g`

Runs tests whose title matches the supplied pattern.

```bash
npx playwright test --grep "login"
```

Short form:

```bash
npx playwright test -g "login"
```

### Interview point

`--grep` is **test selection**, not a locator.

---

## `--grep-invert`

Runs tests that do **not** match the supplied pattern.

```bash
npx playwright test --grep-invert "slow"
```

Mental model:

```text
--grep         → include matching tests
--grep-invert  → exclude matching tests
```

---

## `--workers`

Controls the number of parallel worker processes used for the test run.

```bash
npx playwright test --workers=1
```

Example:

```bash
npx playwright test --workers=4
```

### Important

This is a **test-run configuration option**.

It is different from writing:

```ts
test.describe.configure({ mode: 'parallel' })
```

That controls how tests are organized/executed within the test suite, while `workers` controls the number of worker processes available to the test run.

---

## `--repeat-each`

Runs each test multiple times.

```bash
npx playwright test --repeat-each=3
```

Example:

```text
Test A
→ run 3 times

Test B
→ run 3 times
```

Useful for reproducing intermittent/flaky behavior.

---

## `--retries`

Specifies the number of retry attempts for failed tests.

```bash
npx playwright test --retries=2
```

Conceptually:

```text
First attempt
     ↓
Failed?
     ↓
Retry 1
     ↓
Failed?
     ↓
Retry 2
```

This can also be configured in `playwright.config.ts`.

---

# 4. Debugging Commands

## `--debug`

Runs the test in Playwright's debugging mode.

```bash
npx playwright test tests/login.spec.ts --debug
```

It is useful when you want to inspect the test interactively.

A common interview example:

> "How would you debug a failing Playwright test locally?"

One practical answer:

```bash
npx playwright test tests/login.spec.ts --debug
```

You can then use the Playwright Inspector/debugging tools to step through the test and inspect locators.

---

## `--ui`

Opens Playwright UI Mode.

```bash
npx playwright test --ui
```

UI Mode is useful for:

- selecting tests
- running tests
- debugging tests
- watching test execution
- inspecting steps
- exploring traces

### Remember

```text
--debug → debugging mode
--ui    → UI Mode
```

Do not treat them as the same feature.

---

# 5. Code Generation

## `npx playwright codegen`

Starts Playwright's test generator.

```bash
npx playwright codegen https://example.com
```

You interact with the application and Playwright generates code based on your actions.

### Very common interview command

```bash
npx playwright codegen https://demo.playwright.dev/todomvc
```

### Load authenticated state

```bash
npx playwright codegen --load-storage=auth.json https://example.com
```

This lets Codegen start with previously saved authentication state.

### Important warning

Authentication state can contain sensitive information.

Do **not** casually commit files such as `auth.json` to Git.

---

# 6. HTML Report

## `npx playwright show-report`

Opens the HTML report from a previous test run.

```bash
npx playwright show-report
```

### Open a specific report directory

```bash
npx playwright show-report playwright-report/
```

### Custom port

```bash
npx playwright show-report --port 8080
```

### Custom host

```bash
npx playwright show-report --host 0.0.0.0
```

### Mental model

```text
npx playwright test
        ↓
test execution
        ↓
HTML report generated
        ↓
npx playwright show-report
        ↓
view report
```

---

# 7. Trace Viewer

## `npx playwright show-trace`

Opens a Playwright trace.

```bash
npx playwright show-trace trace.zip
```

You can inspect things such as:

- actions
- screenshots
- DOM snapshots
- network information
- test timing
- errors

### Useful command

```bash
npx playwright show-trace trace.zip
```

### Custom port

```bash
npx playwright show-trace --port 8080 trace.zip
```

### Mental model

```text
Test
 ↓
Trace recorded
 ↓
trace.zip
 ↓
show-trace
 ↓
investigate failure
```

---

# 8. Browser Installation Commands

## `npx playwright install`

Installs Playwright browser binaries.

```bash
npx playwright install
```

### Install only Chromium

```bash
npx playwright install chromium
```

### Install multiple browsers

```bash
npx playwright install chromium firefox
```

### Install browser + OS dependencies

```bash
npx playwright install --with-deps
```

This is particularly common in Linux/CI environments.

### Check before installing

```bash
npx playwright install --dry-run
```

---

# 9. `install-deps`

Installs operating-system dependencies needed by Playwright browsers.

```bash
npx playwright install-deps
```

Common CI/Linux usage:

```bash
npx playwright install --with-deps
```

### Important distinction

```text
playwright install
        ↓
Playwright browser binaries

playwright install-deps
        ↓
OS-level dependencies
```

---

# 10. Uninstall Browsers

## `npx playwright uninstall`

Removes Playwright browser binaries installed by the current Playwright installation.

```bash
npx playwright uninstall
```

There is also:

```bash
npx playwright uninstall --all
```

which removes all Playwright browser installations.

---

# 11. Merge Reports

## `npx playwright merge-reports`

Used mainly with **blob reports**, especially when tests are executed across shards/CI environments.

```bash
npx playwright merge-reports ./blob-reports
```

Example:

```bash
npx playwright merge-reports --reporter=html ./blob-reports
```

### Mental model

```text
Machine 1 → blob report
Machine 2 → blob report
Machine 3 → blob report
                    ↓
             merge-reports
                    ↓
             one final report
```

### Interview point

If asked:

> "How do you combine Playwright reports from sharded CI runs?"

Answer:

> Configure the `blob` reporter for the shard runs and use `npx playwright merge-reports` to combine the blob reports into a final report.

---

# 12. Clear Playwright Cache

## `npx playwright clear-cache`

Clears Playwright caches.

```bash
npx playwright clear-cache
```

This is mainly a troubleshooting/maintenance command.

---

# 13. Check Playwright Version

```bash
npx playwright --version
```

Useful when debugging:

```text
"My local Playwright version is different from CI."
```

Always check the installed version before assuming a CLI option exists.

---

# 14. Get CLI Help

## General help

```bash
npx playwright --help
```

This is extremely important.

Playwright's official CLI documentation specifically recommends using:

```bash
npx playwright --help
```

to get the current list of commands and arguments.

### Command-specific help

You can also inspect help for a command, for example:

```bash
npx playwright test --help
```

or:

```bash
npx playwright codegen --help
```

### Interview memory

**If you forget a CLI option, don't guess — use `--help`.**

---

# 15. Very Important: CLI vs `package.json` Scripts

You will often see this:

```json
{
  "scripts": {
    "test": "playwright test"
  }
}
```

Then:

```bash
npm test
```

runs:

```bash
playwright test
```

You can also pass Playwright arguments through npm scripts:

```bash
npm test -- --headed
```

Mental model:

```text
npm test
   ↓
package.json script
   ↓
playwright test
   ↓
Playwright CLI
```

---

# 16. CLI vs `playwright.config.ts`

This is an important interview concept.

You can configure many test-run behaviors in:

```text
playwright.config.ts
```

and override many of them from the CLI for a particular run.

Example configuration:

```ts
export default defineConfig({
  workers: 2,
  retries: 1
});
```

You can run:

```bash
npx playwright test --workers=1 --retries=0
```

### Mental model

```text
playwright.config.ts
        ↓
default project/test-run configuration

CLI option
        ↓
command-specific override
```

Do not assume every config option has a CLI equivalent. Check the official CLI help/docs for the specific option.

---

# 17. The Commands You MUST Know

If you are preparing for an SDET interview, prioritize these:

| Command | Why you need it |
|---|---|
| `npx playwright test` | Run tests |
| `npx playwright test file.spec.ts` | Run a specific spec |
| `npx playwright test -g "title"` | Run tests by title |
| `npx playwright test --project=chromium` | Run a specific project |
| `npx playwright test --headed` | Run with visible browser |
| `npx playwright test --debug` | Debug a test |
| `npx playwright test --ui` | Use UI Mode |
| `npx playwright test --workers=1` | Control workers |
| `npx playwright test --retries=2` | Configure retries for a run |
| `npx playwright test --repeat-each=3` | Repeat tests |
| `npx playwright codegen URL` | Generate test code |
| `npx playwright show-report` | Open HTML report |
| `npx playwright show-trace trace.zip` | Open trace |
| `npx playwright install` | Install browsers |
| `npx playwright install --with-deps` | Install browsers + OS dependencies |
| `npx playwright merge-reports` | Merge blob reports |
| `npx playwright --version` | Check version |
| `npx playwright --help` | Discover current CLI options |

---

# 18. Commands You SHOULD Know

These are useful but lower priority:

```bash
npx playwright test --grep "login"
```

```bash
npx playwright test --grep-invert "slow"
```

```bash
npx playwright test --only-changed=origin/main
```

```bash
npx playwright install --dry-run
```

```bash
npx playwright uninstall
```

```bash
npx playwright clear-cache
```

```bash
npx playwright show-report --port 8080
```

```bash
npx playwright show-trace --port 8080 trace.zip
```

---

# 19. Commands You DON'T Need to Memorize

You do **not** need to memorize every CLI option.

For example, you don't need to remember every obscure Codegen or installation flag.

Remember the command family:

```text
test
codegen
show-report
show-trace
install
install-deps
uninstall
merge-reports
clear-cache
--version
--help
```

Then use:

```bash
npx playwright <command> --help
```

when you need exact syntax.

This is a much better real-project skill than memorizing dozens of flags.

---

# 20. Common Interview Questions

## Q1. How do you run all Playwright tests?

```bash
npx playwright test
```

---

## Q2. How do you run a single test file?

```bash
npx playwright test tests/login.spec.ts
```

---

## Q3. How do you run a test by title?

```bash
npx playwright test -g "valid login"
```

---

## Q4. How do you run tests in headed mode?

```bash
npx playwright test --headed
```

---

## Q5. How do you debug a Playwright test?

```bash
npx playwright test tests/login.spec.ts --debug
```

You can also use UI Mode:

```bash
npx playwright test --ui
```

---

## Q6. How do you run tests for only Chromium?

If `chromium` is the configured project name:

```bash
npx playwright test --project=chromium
```

---

## Q7. How do you generate a Playwright test?

```bash
npx playwright codegen https://example.com
```

---

## Q8. How do you open the HTML report?

```bash
npx playwright show-report
```

---

## Q9. How do you inspect a Playwright trace?

```bash
npx playwright show-trace trace.zip
```

---

## Q10. How do you install Playwright browsers?

```bash
npx playwright install
```

For Linux/CI where OS dependencies are needed:

```bash
npx playwright install --with-deps
```

---

## Q11. How do you merge reports from sharded runs?

Use blob reports and:

```bash
npx playwright merge-reports ./blob-reports
```

---

## Q12. How do you check the installed Playwright version?

```bash
npx playwright --version
```

---

## Q13. How do you find available CLI options?

```bash
npx playwright --help
```

---

# 21. Real Project Example

Imagine you have:

```text
tests/
├── login.spec.ts
├── checkout.spec.ts
└── profile.spec.ts
```

### Developer workflow

```bash
# 1. Run everything
npx playwright test

# 2. See the browser
npx playwright test --headed

# 3. Debug login
npx playwright test tests/login.spec.ts --debug

# 4. Run only checkout tests
npx playwright test tests/checkout.spec.ts

# 5. Run tests with "login" in their title
npx playwright test -g "login"

# 6. Open HTML report
npx playwright show-report

# 7. Inspect a trace
npx playwright show-trace trace.zip
```

### CI workflow

```text
Install dependencies
       ↓
Install Playwright browsers
       ↓
Run tests
       ↓
Generate artifacts/reports
       ↓
Investigate failures using HTML report / trace
```

Typical CI installation command:

```bash
npx playwright install --with-deps
```

---

# 22. Do NOT Confuse These

## `test` vs `codegen`

```text
playwright test
    → executes tests

playwright codegen
    → helps generate test code
```

---

## `show-report` vs `show-trace`

```text
show-report
    → opens test report

show-trace
    → opens an individual Playwright trace
```

---

## `install` vs `install-deps`

```text
install
    → browser binaries

install-deps
    → operating-system dependencies
```

---

## `--headed` vs `--debug`

```text
--headed
    → simply show the browser UI

--debug
    → debug the test interactively
```

---

## `--grep` vs `--project`

```text
--grep
    → select tests by title/tag pattern

--project
    → select a configured Playwright project
```

---

# 23. Important Note About `playwright-cli`

There is a **different** CLI called:

```text
playwright-cli
```

It is associated with browser automation for coding agents.

For example:

```bash
playwright-cli open https://example.com
```

This is **not the same thing** as the Playwright Test CLI command:

```bash
npx playwright test
```

For normal Playwright Test / SDET interview preparation, your primary focus should be:

```bash
npx playwright ...
```

The two CLIs should not be mixed together.

---

# 24. Final Mental Map

Remember this:

```text
                 PLAYWRIGHT CLI
                       │
        ┌──────────────┼──────────────┐
        │              │              │
       RUN          DEBUG          REPORT
        │              │              │
      test        --debug          show-report
      -g          --ui             show-trace
      --project   codegen          merge-reports
        │
        │
     BROWSERS
        │
     install
     install-deps
     uninstall
        │
        │
   INFORMATION
        │
     --version
     --help
     clear-cache
```

---

# 25. What You Actually Need to Remember

### Tier 1 — Memorize

```bash
npx playwright test
npx playwright test tests/login.spec.ts
npx playwright test -g "login"
npx playwright test --project=chromium
npx playwright test --headed
npx playwright test --debug
npx playwright test --ui
npx playwright codegen https://example.com
npx playwright show-report
npx playwright show-trace trace.zip
npx playwright install
npx playwright install --with-deps
npx playwright merge-reports ./blob-reports
npx playwright --version
npx playwright --help
```

### Tier 2 — Understand

```text
--workers
--retries
--repeat-each
--grep
--grep-invert
--only-changed
```

### Tier 3 — Look up when needed

Everything else.

**The professional skill is not memorizing every flag.**

The professional skill is knowing:

```text
WHAT do I want to do?
        ↓
WHICH Playwright command?
        ↓
npx playwright <command> --help
```

---

# 26. One-Page Revision Cheat Sheet

```text
RUN
npx playwright test
npx playwright test tests/login.spec.ts
npx playwright test -g "login"
npx playwright test --project=chromium
npx playwright test --headed

DEBUG
npx playwright test --debug
npx playwright test --ui
npx playwright codegen https://example.com

REPORT
npx playwright show-report
npx playwright show-trace trace.zip
npx playwright merge-reports ./blob-reports

BROWSERS
npx playwright install
npx playwright install chromium
npx playwright install --with-deps
npx playwright install-deps
npx playwright uninstall

INFO / MAINTENANCE
npx playwright --version
npx playwright --help
npx playwright clear-cache

TEST SELECTION / EXECUTION
--grep
--grep-invert
--project
--workers
--retries
--repeat-each
--only-changed
```

---

## Official Reference

- Playwright Command Line documentation
- Playwright Test running/debugging documentation
- Playwright Codegen documentation
- Playwright Test sharding / merge-reports documentation
- Playwright installation documentation

> **Accuracy rule:** CLI options can change between Playwright versions. When an exact option matters, verify it against the CLI help for the Playwright version installed in the project:
>
> ```bash
> npx playwright --help
> npx playwright test --help
> ```
