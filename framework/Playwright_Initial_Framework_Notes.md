# Playwright TypeScript Framework — Initial Setup & Basic Framework

> **Purpose:** These notes document the initial setup of a Playwright TypeScript automation framework and the basic Page Object Model (POM) structure.

---

# 1. Project Initial Setup

## Step 1 — Create a Workspace

In VS Code:

1. Open **VS Code**
2. Select **File → Open Folder**
3. Create/select your project folder.

Example:

```text
Playwright Automation Practice
```

This folder becomes the **root folder of the Playwright project**.

---

# 2. Initialize Node.js

Open the VS Code terminal and run:

```bash
npm init -y
```

This creates:

```text
package.json
```

### What is `package.json`?

`package.json` is the **Node.js project configuration file**.

It keeps information such as:

- Project name
- Project version
- Dependencies
- Scripts
- Other Node.js project settings

### About `"type": "module"`

If your project uses ES module syntax such as:

```typescript
import { test, expect } from "@playwright/test";
```

the project may use:

```json
"type": "module"
```

However, **do not treat `"type": "module"` as a mandatory Playwright setting in every project**. Its requirement depends on the project's module configuration.

---

# 3. Install / Initialize Playwright

Run:

```bash
npm init playwright@latest
```

The Playwright setup process creates/configures the Playwright project and installs the required Playwright packages.

Depending on the options selected during setup, files such as these may be created:

```text
playwright.config.ts
tests/
package.json
```

and other Playwright-related files.

> **Important:** If Playwright was initialized using `npm init playwright@latest`, you do not need to manually install Playwright again unless there is a specific reason.

---

# 4. Add `tsconfig.json`

Create:

```text
tsconfig.json
```

This file contains **TypeScript compiler configuration**.

It controls how TypeScript code in the project is interpreted/compiled.

A typical project may contain settings related to:

- Target JavaScript version
- Module system
- Module resolution
- Included files
- Compiler strictness
- Path configuration

> The exact `tsconfig.json` settings should be based on the project's requirements. Do not blindly copy configuration from another project.

---

# 5. Basic Framework Folder Structure

A simple framework can be organized like this:

```text
Playwright Automation Practice/
│
├── src/
│   ├── pages/
│   │   ├── BasePage.ts
│   │   └── LoginPage.ts
│   │
│   ├── fixtures/
│   │
│   └── util/
│
├── data/
│
├── config/
│
├── tests/
│   └── LoginPage.spec.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

The exact folder names are a **framework design choice**. Playwright does not require this exact structure.

---

# 6. `src` Directory

The `src` directory contains reusable framework/application automation code.

## 6.1 `pages`

Example:

```text
src/
└── pages/
    ├── BasePage.ts
    └── LoginPage.ts
```

Used for:

- Page classes
- Base page
- Page-specific locators
- Page-specific actions/behaviors

### Mental Model

```text
Page class = UI knowledge + UI actions
```

For example:

```text
LoginPage
    ↓
Knows login page locators
    ↓
Knows how to perform login
```

---

## 6.2 `fixtures`

Used for **custom Playwright fixtures** when the framework needs its own reusable test dependencies/setup.

Example future use:

```text
fixtures/
    └── test.ts
```

A custom fixture could provide something such as:

```text
test
 └── loginPage
```

so that tests do not need to manually create the page object every time.

> You do not need custom fixtures immediately. Start with Playwright's built-in fixtures and introduce custom fixtures when the framework needs them.

---

## 6.3 `util`

Contains reusable utility/helper functions used across the framework.

Examples:

```text
util/
├── DateUtil.ts
├── FileUtil.ts
└── StringUtil.ts
```

Mental model:

```text
util = common reusable helpers
```

---

# 7. `data` Directory

The `data` directory can contain test data files.

Examples:

```text
data/
├── users.json
├── testdata.xlsx
└── users.csv
```

Possible test data:

- Usernames
- Passwords
- Customer information
- Test scenarios
- Expected values

Mental model:

```text
tests     → test logic
data      → test data
```

---

# 8. `config` Directory

A project may have a `config` directory for environment/configuration-related files.

For example:

```text
config/
├── dev.env
├── qa.env
└── prod.env
```

However, **the `config` directory is a framework convention, not a Playwright requirement**.

Also be careful with secrets.

Do not commit sensitive credentials/passwords to Git.

---

# 9. Reports

You do **not** need to manually create a `report` directory just to make Playwright reporting work.

Playwright Test provides built-in reporting capabilities.

Reports can be configured through:

```text
playwright.config.ts
```

For example, Playwright supports reporters such as:

```text
list
html
json
junit
```

The exact reporter configuration depends on the project requirements.

---

# 10. `tests` Directory

The `tests` directory contains test specification files.

Example:

```text
tests/
└── LoginPage.spec.ts
```

Typical naming:

```text
*.spec.ts
```

Example:

```text
LoginPage.spec.ts
CheckoutPage.spec.ts
SearchPage.spec.ts
```

The spec file contains:

- Test cases
- Assertions
- Hooks
- Test-level orchestration

---

# 11. Basic Framework Architecture

The basic flow is:

```text
Test Spec
   │
   │ creates/uses
   ↓
Page Object
   │
   │ uses
   ↓
Locators
   │
   │ interacts with
   ↓
Application UI
```

Example:

```text
LoginPage.spec.ts
       ↓
LoginPage.ts
       ↓
BasePage.ts
       ↓
Playwright Page
       ↓
Browser
       ↓
Application
```

---

# 12. BasePage.ts

The BasePage contains common functionality that can be shared by multiple page classes.

Your current `BasePage.ts` follows this basic design:

```typescript
import { Page } from "@playwright/test";

export class BasePage {

    protected readonly page: Page;

    constructor(page: Page) {
        this.page = page;
    }
}
```

This matches your uploaded implementation. fileciteturn0file0L1-L11

---

## 12.1 Why do we have `page`?

Playwright's `Page` represents a browser tab/page.

The page object is used for operations such as:

```typescript
page.goto()
page.locator()
page.getByRole()
page.title()
```

---

## 12.2 Why `protected readonly page: Page`?

```typescript
protected readonly page: Page;
```

### `Page`

The variable is of Playwright's `Page` type.

### `readonly`

The reference should not be reassigned after initialization.

### `protected`

The `page` property can be accessed by:

- `BasePage`
- Classes that extend `BasePage`

This is useful because page classes inherit the common page reference.

---

## 12.3 Constructor

```typescript
constructor(page: Page) {
    this.page = page;
}
```

The constructor receives the Playwright `Page` object and stores it in the class.

Mental model:

```text
Test
 ↓
Playwright gives Page
 ↓
LoginPage(page)
 ↓
BasePage(page)
 ↓
this.page = page
```

---

# 13. Page Classes

A page class represents a page or meaningful UI area of the application.

Example:

```text
LoginPage.ts
```

Your `LoginPage` extends `BasePage`:

```typescript
export class LoginPage extends BasePage {
```

Your uploaded implementation follows this pattern. fileciteturn0file1L1-L3

Mental model:

```text
BasePage
   ↑
LoginPage
```

---

# 14. Page Class — Step 1: Declare Locators

Example:

```typescript
private readonly emailID: Locator;
private readonly password: Locator;
private readonly loginBtn: Locator;
```

Your current `LoginPage.ts` declares private readonly locators for the login page. fileciteturn0file1L15-L20

### Why `private`?

The test should not directly manipulate page locators.

Instead of:

```typescript
loginPage.emailID.fill(...)
```

the test should use a meaningful page action:

```typescript
loginPage.doLogin(...)
```

This is part of **encapsulation**.

Mental model:

```text
Locator = private implementation detail
Method  = public behavior
```

---

# 15. Page Class — Step 2: Constructor

Example:

```typescript
constructor(page: Page) {
    super(page);

    this.emailID =
        page.getByRole('textbox', { name: 'E-Mail Address' });

    this.password =
        page.getByLabel('Password');

    this.loginBtn =
        page.getByRole('button', { name: 'Login' });
}
```

Your uploaded `LoginPage.ts` uses this pattern: it receives `Page`, passes it to `BasePage` using `super(page)`, and initializes the page locators. fileciteturn0file1L22-L30

---

# 16. What does `super(page)` mean?

`LoginPage` extends `BasePage`.

Therefore, the `BasePage` constructor also needs to be initialized.

```typescript
super(page);
```

means:

```text
Send the page object
        ↓
to BasePage constructor
        ↓
BasePage stores it in this.page
```

So:

```typescript
this.page
```

is available in `LoginPage` because `LoginPage` inherits from `BasePage`.

---

# 17. Page Class — Step 3: Public Page Actions

The page class should expose meaningful actions/behaviors.

Example:

```typescript
async doLogin(
    username: string,
    password: string
): Promise<void> {

    await this.emailID.fill(username);
    await this.password.fill(password);
    await this.loginBtn.click();
}
```

Your current `LoginPage.ts` follows this page-action approach. fileciteturn0file1L33-L52

The test does not need to know:

```text
Which locator is used?
How is the username entered?
Which button is clicked?
```

The test simply says:

```typescript
await loginPage.doLogin("test", "test");
```

Mental model:

```text
Test says WHAT
      ↓
Page class knows HOW
```

This is one of the most important ideas in Page Object Model.

---

# 18. Other Page Methods

Your current page class also exposes methods such as:

```typescript
async getLoginPageTitle(): Promise<string>
```

and:

```typescript
async isForgottenPasswordExist(): Promise<boolean>
```

and:

```typescript
async isInvalidLoginErrorDisplayed(): Promise<boolean>
```

These methods hide the locator/UI implementation from the test. fileciteturn0file1L43-L60

---

# 19. Spec File

Example:

```text
tests/LoginPage.spec.ts
```

The spec file contains the actual test scenarios.

Your current spec file imports:

```typescript
import { Page, test, expect } from '@playwright/test';
import { LoginPage } from '../src/pages/LoginPage';
```

and maintains a `LoginPage` reference. fileciteturn0file2L7-L10

---

# 20. Do We Create Classes in Spec Files?

Normally, **no**.

The spec file is primarily responsible for:

- Defining tests
- Defining hooks
- Calling page-object methods
- Performing assertions

The page classes contain the page-specific UI implementation.

Mental model:

```text
Spec = WHAT to test
Page  = HOW to interact with the page
```

---

# 21. Creating the Page Object in the Test

Your current framework uses:

```typescript
let loginPage: LoginPage;
```

Then initializes it inside `beforeEach`:

```typescript
test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goToLoginPage();
});
```

This is present in your uploaded spec file. fileciteturn0file2L10-L15

### Why `beforeEach`?

`beforeEach` runs before **each test in that scope**.

So if you have:

```text
Test 1
Test 2
Test 3
```

the setup runs before:

```text
Test 1
Test 2
Test 3
```

This gives each test the required setup.

---

# 22. `page` Fixture

This is a very important Playwright concept.

When you write:

```typescript
test('login test', async ({ page }) => {
```

`page` is a **built-in Playwright Test fixture**.

It is not something you manually create using:

```typescript
new Page()
```

Playwright Test provides the fixture to the test.

Your current spec uses:

```typescript
test.beforeEach(async ({ page }) => {
```

and passes that `page` into the page object. fileciteturn0file2L12-L15

---

# 23. What Does `{ page }` Mean?

This part:

```typescript
{ page }
```

uses **JavaScript object destructuring**.

Conceptually, Playwright provides an object containing fixtures.

You can think of it as:

```text
Fixture object
{
    page: <Playwright Page>
}
```

Then:

```typescript
{ page }
```

extracts the `page` property.

So:

```typescript
async ({ page }) => {
```

means:

```text
Give me the Playwright `page` fixture.
```

---

# 24. Important Correction — "Server Fixture"

Avoid saying:

> `page` is a server fixture.

That is not the right terminology.

Say:

> **`page` is a built-in Playwright Test fixture.**

It provides a Playwright `Page` object to the test.

---

# 25. What Does the `page` Fixture Provide?

Mental model:

```text
Playwright Test
      │
      │ provides
      ↓
    page fixture
      │
      ↓
 Playwright Page object
      │
      ↓
Browser tab/page
```

The `Page` object gives you APIs such as:

```typescript
page.goto()
page.locator()
page.getByRole()
page.title()
page.screenshot()
```

---

# 26. Test Execution Flow

For your current framework, the flow is approximately:

```text
Playwright starts test
        ↓
Creates/provides `page` fixture
        ↓
beforeEach executes
        ↓
new LoginPage(page)
        ↓
LoginPage constructor executes
        ↓
super(page)
        ↓
BasePage stores page
        ↓
LoginPage initializes locators
        ↓
goToLoginPage()
        ↓
Test executes
        ↓
Page-object method is called
        ↓
Assertion validates result
```

This is the **mental map you should remember**.

---

# 27. Complete Example Flow

## Test

```typescript
test('verify login page title', async () => {

    expect(
        await loginPage.getLoginPageTitle()
    ).toBe('Account Login');

});
```

Your uploaded spec follows this pattern. fileciteturn0file2L18-L20

### What happens?

```text
Test
 ↓
loginPage.getLoginPageTitle()
 ↓
LoginPage
 ↓
this.page.title()
 ↓
Browser
 ↓
Returns title
 ↓
expect()
 ↓
Assertion
```

---

# 28. Framework Responsibilities

A clean separation is:

| Component | Responsibility |
|---|---|
| `*.spec.ts` | Test scenario + assertions |
| Page class | Page locators + page actions |
| `BasePage.ts` | Common page functionality |
| `fixtures/` | Custom reusable test dependencies |
| `util/` | Common reusable utilities |
| `data/` | Test data |
| `playwright.config.ts` | Playwright test configuration |
| `tsconfig.json` | TypeScript configuration |

---

# 29. What Should NOT Go Into the Spec?

Avoid putting large amounts of UI implementation directly into the spec.

### Avoid

```typescript
test('login', async ({ page }) => {

    await page.getByRole('textbox', {
        name: 'E-Mail Address'
    }).fill('test');

    await page.getByLabel('Password').fill('test');

    await page.getByRole('button', {
        name: 'Login'
    }).click();

});
```

### Prefer

```typescript
test('login', async () => {

    await loginPage.doLogin('test', 'test');

});
```

Why?

Because the page class owns the UI implementation.

---

# 30. Encapsulation in This Framework

The login locators are:

```typescript
private readonly
```

The test does not directly access them.

Instead:

```text
Private Locator
      ↓
Public Page Method
      ↓
Test
```

Example:

```text
emailID
password
loginBtn
     ↓
doLogin()
     ↓
Login test
```

This is **encapsulation** in practical terms.

---

# 31. Important Naming Understanding

### Locator

Represents a way to identify an element.

Example:

```typescript
private readonly loginBtn: Locator;
```

### Page

Represents the browser page/tab.

```typescript
page: Page
```

### Page Object

A class representing a page/UI area.

```typescript
LoginPage
```

### Page Object Model (POM)

A design approach where page-specific UI locators and actions are organized into page classes.

---

# 32. Current Framework — Simple Mental Map

Remember this:

```text
                 TEST
                  │
                  │ calls
                  ↓
            PAGE OBJECT
             LoginPage
                  │
          ┌───────┴────────┐
          ↓                ↓
       LOCATORS          ACTIONS
          │                │
          └───────┬────────┘
                  ↓
             BASE PAGE
                  │
                  ↓
          PLAYWRIGHT PAGE
                  │
                  ↓
              BROWSER
                  │
                  ↓
             APPLICATION
```

---

# 33. What You Actually Need to Remember

## MUST KNOW

### 1. `page`

```typescript
async ({ page }) => {}
```

`page` is a built-in Playwright Test fixture that provides a Playwright `Page` object.

### 2. BasePage

```typescript
protected readonly page: Page;
```

Stores the Playwright page reference for reuse by page classes.

### 3. Page class

Example:

```typescript
class LoginPage extends BasePage
```

Contains:

- Locators
- Page actions
- Page-specific behavior

### 4. `super(page)`

Passes the page object to the parent `BasePage` constructor.

### 5. Private locators

```typescript
private readonly loginBtn: Locator;
```

Keep locator implementation inside the page class.

### 6. Public actions

```typescript
doLogin()
```

Expose meaningful page behavior to tests.

### 7. Spec file

Contains:

```text
Tests
Hooks
Assertions
Page-object calls
```

### 8. `beforeEach`

Runs setup before each test in its scope.

---

# 34. One-Line Memory Trick

> **Spec says WHAT, Page says HOW, BasePage provides COMMON support, Fixture provides the Page.**

```text
SPEC
 ↓
WHAT

PAGE OBJECT
 ↓
HOW

BASE PAGE
 ↓
COMMON

FIXTURE
 ↓
PROVIDES PAGE
```

---

# 35. Current Framework — Final Picture

```text
Playwright Test
      │
      │ built-in fixture
      ↓
    { page }
      │
      ↓
LoginPage(page)
      │
      ├── super(page)
      │       ↓
      │   BasePage
      │       ↓
      │   this.page
      │
      ├── Locators
      │
      └── Page Actions
              │
              ↓
        Application UI

LoginPage.spec.ts
      │
      ├── beforeEach()
      ├── test()
      └── expect()
              │
              ↓
        LoginPage methods
```

This is the basic foundation of your framework. The next framework layers can be added gradually rather than introducing everything at once.
