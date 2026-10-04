# High-Level Components / High-Level Design (HLD) — Part 1

> **Goal:** Understand the major building blocks of a Playwright TypeScript automation framework and the responsibility of each component.

---

# 1. Big Picture

A typical Playwright automation framework can be thought of as:

```text
                    AUTOMATION FRAMEWORK
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
   Page Layer          Test Layer         Fixtures
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ↓
                      Test Data
                           ↓
                 Global Configuration
                           ↓
                    Environment
                           ↓
                      Utilities
                           ↓
                    Reporting
                           ↓
              Project Configuration
              ├── package.json
              ├── tsconfig.json
              └── .gitignore

Part 2 — Infrastructure
              ├── Git
              └── CI/CD
```

> **Important:** This is a conceptual HLD. Real projects can combine, rename, or add components depending on their needs. There is no single mandatory Playwright framework architecture.

---

# 2. Page Layer

## What is it?

> **The Page Layer represents application pages or UI components and contains the locators and UI operations related to them.**

For example:

```text
LoginPage
├── username locator
├── password locator
├── login button locator
│
├── enterUsername()
├── enterPassword()
└── clickLogin()
```

The test should use the page object's behavior rather than repeatedly implementing the UI details.

## BasePage

A framework may have a `BasePage` containing **genuinely common page-level functionality** used by multiple page objects.

Example:

```text
BasePage
├── common page methods
└── common page-level functionality

        ↑
        │ inherits

LoginPage
HomePage
AccountPage
```

This is an example of **inheritance**.

### Important

Do not create `BasePage` just because every Page Object needs one.

Use it when there is meaningful common behavior that benefits from sharing.

Also, not every method used by multiple pages necessarily belongs in `BasePage`. The responsibility should remain clear.

## Mental Map

```text
Page Layer
    ↓
Represents UI
    ↓
Locators + UI operations
    ↓
Test uses page behavior
```

---

# 3. Test Layer

## What is it?

> **The Test Layer contains test scenarios and test-specific verification/assertions.**

In Playwright Test, tests are commonly written in spec files:

```text
tests/
├── login.spec.ts
├── checkout.spec.ts
└── account.spec.ts
```

A test typically performs:

```text
Arrange
   ↓
Act
   ↓
Assert
```

This is the **AAA pattern**.

## AAA Pattern

### Arrange

Prepare everything required for the test.

```text
Create/use required data
Navigate/setup required state
Prepare objects
```

### Act

Perform the action being tested.

```text
Login
Click
Submit
Create record
```

### Assert

Verify the expected result.

```text
Expected result
       vs
Actual result
```

Example:

```ts
await loginPage.login("user", "password");

await expect(page).toHaveURL(/dashboard/);
```

The exact arrangement depends on the test.

## Responsibility of Assertions

For a clean separation:

```text
Page Layer
    ↓
How to interact with the UI

Test Layer
    ↓
What should be verified
```

Prefer keeping **test-specific assertions** in the Test Layer.

## Does the test create Page Objects?

Conceptually:

```text
Test
  ↓
Page Object
  ↓
UI interaction
  ↓
Application
```

In Playwright, Page Objects are commonly instantiated with the Playwright `Page` object, and tests can use those page objects directly or through custom fixtures.

Example:

```ts
const loginPage = new LoginPage(page);

await loginPage.login();
await expect(page).toHaveURL(/dashboard/);
```

## Test Independence

> **Tests should ideally be independent and should not require another test to run first.**

For example:

```text
Test A → creates customer
Test B → requires customer created by Test A
```

This creates a dependency.

Prefer:

```text
Test A → prepares what it needs → runs independently

Test B → prepares what it needs → runs independently
```

This makes tests safer to run individually, in parallel, or in a different order.

## Important Correction About Workers

Avoid the statement:

> ❌ "Workers randomly pick test cases from a test case pool."

A better understanding is:

> **Playwright Test workers are worker processes used to execute tests. Playwright's test runner schedules test work across workers according to its execution model and configuration. You should not rely on a random selection order.**

The important framework rule is:

> **Do not design tests with dependencies on execution order.**

## Parent Class for Test Files?

You generally do **not** need a parent class for spec files.

Playwright Test is designed around the test runner, test functions, fixtures, configuration, and modules rather than requiring test classes.

The important principle is:

> **Keep tests independently executable; don't create inheritance just to share test setup. Use fixtures or reusable functions where appropriate.**

---

# 4. Fixtures

## What is a Fixture?

> **A fixture provides a prepared resource, object, or setup to a test when the test needs it.**

Think:

```text
Fixture
   ↓
Prepare/provide something
   ↓
Test uses it
```

## Built-in Fixtures

Playwright Test provides built-in fixtures such as:

```text
page
context
browser
browserName
request
```

The exact fixture availability depends on the Playwright Test API being used.

Example:

```ts
test("Login test", async ({ page }) => {
    await page.goto("/login");
});
```

Here:

```text
Playwright Test Runner
        ↓
provides page fixture
        ↓
test uses page
```

## Custom Fixtures

You can create custom fixtures to provide reusable test setup or objects.

For example:

```text
Custom Fixture
      ↓
Creates LoginPage
      ↓
Provides LoginPage to test
```

Then a test can conceptually use:

```ts
test("Login", async ({ loginPage }) => {
    await loginPage.login();
});
```

### Important Correction

> ❌ "Custom Fixtures = factory of all Page Class objects."

That is too narrow.

A custom fixture can provide **page objects, test data, API clients, setup/teardown resources, authenticated state, or other reusable test dependencies**.

It is better to think:

> **Custom fixture = a controlled way to create/provide reusable test dependencies and manage their setup/teardown.**

---

# 5. Test Data

## What is it?

> **Test data is the data required to execute and validate a test.**

Common sources include:

```text
JSON
CSV
Excel
Database
API
Environment/configuration
```

The appropriate source depends on the project.

## Data-Driven Testing

> **Data-driven testing means running the same test logic with multiple sets of input data.**

Example:

```text
Test: Login

Data Set 1 → valid user
Data Set 2 → invalid password
Data Set 3 → locked user
Data Set 4 → expired user
```

The test logic can remain the same while the input data changes.

## Important Correction

Your original statement:

> "# of Rows in test data file = # of times test cases will be executed"

This is **not a universal rule**.

It can be true for a particular data-driven implementation, but it is not a framework rule.

For example:

```text
10 rows
   ↓
could result in 10 test invocations
```

But the framework may filter, combine, skip, or otherwise transform the data.

So remember:

> **Number of executions depends on how the test data is mapped to the test, not simply on the number of rows.**

## Where should test data be maintained?

There is no single mandatory location.

You may have:

```text
test-data/
├── users.json
├── login.csv
└── users.xlsx
```

Or data may be provided through:

```text
Fixtures
Environment variables
API
Database
Test code
```

Choose the approach based on the type and lifecycle of the data.

---

# 6. Global Configuration

## What is it?

In Playwright, the main global configuration file is commonly:

```text
playwright.config.ts
```

It defines project/test-run settings.

Examples include:

```text
Browser projects
Base URL
Timeouts
Retries
Workers
Parallel execution settings
Reporters
Trace
Screenshot
Video
Web server
Test directory
```

Not every setting is necessarily "global" in the same sense; some can be configured at project or test level.

## Mental Map

```text
playwright.config.ts
          ↓
Controls test-run behavior
          ↓
Tests + Projects + Reporting + Execution
```

## Important Point

The configuration file does not mean:

> "Every framework component directly uses every configuration."

Instead:

> **Playwright Test reads the configuration and uses the applicable settings while running tests.**

---

# 7. Environment

## What is it?

> **Environment configuration contains values that can change between environments or executions.**

For example:

```text
DEV
    URL = dev.example.com

QA
    URL = qa.example.com

UAT
    URL = uat.example.com
```

These values should not necessarily be hard-coded into tests.

## `.env` Files

A project may use `.env` files or another configuration/secrets mechanism.

For example:

```text
.env
.env.qa
.env.uat
```

### Important Correction

Your original statement:

> "# of environments = # .env files"

This is **not a rule**.

For example:

```text
3 environments
```

could use:

```text
1 configuration mechanism
```

or

```text
3 .env files
```

or a CI/CD secret/configuration system.

Also, `.env` is **not a Playwright requirement**. It is a common way to manage environment variables.

### Security

Do not commit sensitive credentials or secrets to Git.

Use appropriate secret management for real credentials, especially in CI/CD.

---

# 8. Utilities

## What are they?

> **Utilities are reusable helper functions or modules for common technical tasks.**

Examples:

```text
Read test data
Generate random values
Date formatting
File handling
Common API helpers
```

Example:

```text
utils/
├── testDataReader.ts
├── randomData.ts
└── dateUtils.ts
```

## Important Rule

Don't create one huge:

```text
utils.ts
```

containing unrelated functionality.

Keep utilities focused and reusable.

Mental map:

```text
Utility
   ↓
Common technical task
   ↓
Reusable helper
```

---

# 9. package.json

## What is it?

> **`package.json` is the Node.js project manifest that describes the project and its npm-related configuration, including dependencies and scripts.**

Typical areas include:

```json
{
  "scripts": {},
  "dependencies": {},
  "devDependencies": {}
}
```

### In a Playwright project

It can contain scripts such as:

```json
{
  "scripts": {
    "test": "playwright test"
  }
}
```

It also records package dependencies and development dependencies.

## Mental Map

```text
package.json
      ↓
Node project information
      ↓
Dependencies + Scripts + Project metadata
```

---

# 10. Reporting

## What is it?

> **Reporting provides information about test execution results and, depending on the configured tools, supporting execution artifacts.**

Examples include:

```text
HTML report
JUnit report
JSON report
Screenshots
Videos
Traces
Logs
```

### Important Correction

Do not treat all of these as "reports".

A better distinction is:

```text
Reports
   ↓
Test result summaries
   ├── HTML
   ├── JUnit
   └── JSON

Artifacts / Diagnostics
   ↓
Evidence for investigation
   ├── Screenshot
   ├── Video
   ├── Trace
   └── Logs
```

The exact artifacts produced depend on the Playwright configuration and test setup.

---

# 11. tsconfig.json

## What is it?

> **`tsconfig.json` is the TypeScript configuration file for the project.**

It controls how TypeScript understands and compiles/transpiles the project's TypeScript code.

Common settings you may encounter include:

```text
target
module
moduleResolution
strict
baseUrl
paths
include
exclude
```

For Playwright projects, it can also affect how TypeScript resolves imports and checks your test/framework code.

## Mental Map

```text
TypeScript Code
      ↓
tsconfig.json
      ↓
TypeScript rules
```

---

# 12. .gitignore

## What is it?

> **`.gitignore` tells Git which files and directories should be ignored instead of tracked.**

Examples commonly ignored in automation projects can include generated files such as:

```text
node_modules/
test-results/
playwright-report/
```

The exact entries depend on the project.

## Important Point

`.gitignore` does **not delete files**.

It tells Git:

> "Do not include these untracked files in normal Git tracking."

### Mental Map

```text
Project files
     ↓
.gitignore
     ↓
Tell Git what to ignore
```

---

# 13. High-Level View — Part 1

Put everything together:

```text
                         FRAMEWORK
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
   Page Layer            Test Layer           Fixtures
        │                    │                    │
        │                    │                    │
        └────────────────────┼────────────────────┘
                             ↓
                         Test Data
                             ↓
                  Playwright Configuration
                             ↓
                       Environment
                             ↓
                         Utilities
                             ↓
                  Project Configuration
                    ├── package.json
                    ├── tsconfig.json
                    └── .gitignore
                             ↓
                    Reports + Artifacts
```

---

# 14. How a Test Flows Through the Framework

A simple mental model:

```text
Test starts
    ↓
Playwright configuration
    ↓
Fixtures prepare/provide dependencies
    ↓
Test uses Page Objects
    ↓
Page Objects interact with Application
    ↓
Test performs assertions
    ↓
Playwright produces results/artifacts
    ↓
Report / diagnostics
```

Test data and environment configuration can feed into the test/fixture/page-object flow depending on the framework design.

---

# 15. What You Actually Need to Remember

## Page Layer

> **Represents UI pages/components → locators + UI operations.**

## BasePage

> **Optional shared base for genuinely common page-level behavior.**

## Test Layer

> **Contains test scenarios and test-specific verification.**

## AAA

> **Arrange → Act → Assert.**

## Test Independence

> **Tests should not depend on another test's execution or state.**

## Fixtures

> **Provide prepared/reusable dependencies or setup to tests.**

## Test Data

> **Input/expected data required by tests.**

## Data-Driven Testing

> **Same test logic + different data sets.**

## Playwright Config

> **Controls applicable test-run/project behavior.**

## Environment

> **Environment-specific values/configuration.**

## Utilities

> **Reusable helpers for common technical tasks.**

## package.json

> **Node project manifest → dependencies + scripts + metadata.**

## Reporting

> **Reports show results; traces/screenshots/videos/logs help investigate failures.**

## tsconfig.json

> **TypeScript project configuration.**

## .gitignore

> **Tells Git what to ignore.**

---

# 16. Interview-Ready Summary

> "At a high level, I separate the Playwright framework into layers and supporting components. The Page Layer encapsulates UI locators and page operations, while the Test Layer contains test scenarios and test-specific assertions. Fixtures provide reusable test dependencies and setup. Test data is separated from test logic where appropriate, and `playwright.config.ts` controls test execution and project settings. Environment configuration handles values that vary between environments. Utilities provide reusable technical helpers. `package.json` manages the Node project, `tsconfig.json` manages TypeScript configuration, `.gitignore` controls Git tracking, and reporting plus execution artifacts provide test results and failure diagnostics. Infrastructure such as Git and CI/CD can then sit around the framework as a separate concern."

---

# 17. Part 2 — Infrastructure

The next HLD section can cover:

```text
Infrastructure
│
├── Git
│   ├── Version control
│   ├── Branching
│   ├── Commit
│   ├── Push / Pull
│   └── Merge / PR
│
└── CI/CD
    ├── Build / Install
    ├── Test Execution
    ├── Reports
    ├── Artifacts
    └── Pipeline integration
```

> **Do not mix Part 2 into Part 1 yet.** First understand the framework components. Infrastructure explains how the framework is version-controlled and executed in automated pipelines.
