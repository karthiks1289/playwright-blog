# Playwright Fixtures with Page Objects — Notes & Revision

## 1. What Are We Trying to Achieve?

In a Page Object Model framework, tests need Page Object instances such as:

- `LoginPage`
- `RegisterPage`

Without fixtures, every test may need to create those objects manually:

```ts
const loginPage = new LoginPage(page);
const registerPage = new RegisterPage(page);
```

A Playwright fixture lets us centralize that object creation.

### Core idea

> **When a test asks for `loginPage`, the fixture creates a `LoginPage` object and provides it to the test.**

The same idea applies to `registerPage`.

---

# 2. The Mental Model

Remember the algorithm, not the syntax:

```text
Test asks for loginPage
        ↓
Fixture gets Playwright's page
        ↓
Fixture creates LoginPage object
        ↓
use(loginPage)
        ↓
Test receives LoginPage object
```

For `registerPage`:

```text
Test asks for registerPage
        ↓
Fixture gets Playwright's page
        ↓
Fixture creates RegisterPage object
        ↓
use(registerPage)
        ↓
Test receives RegisterPage object
```

### One-line memory trick

> **Fixture = prepare something and provide it to the test.**

---

# 3. The Complete Fixture File

Example file:

```text
src/
└── fixtures/
    └── pagefixtures.ts
```

```ts
import { test as baseTest } from "@playwright/test";
import { LoginPage } from "../pages/LoginPage";
import { RegisterPage } from "../pages/RegisterPage";

type Fixtures = {
    loginPage: LoginPage;
    registerPage: RegisterPage;
};

export const test = baseTest.extend<Fixtures>({
    loginPage: async ({ page }, use) => {
        const loginPage = new LoginPage(page);
        await use(loginPage);
    },

    registerPage: async ({ page }, use) => {
        const registerPage = new RegisterPage(page);
        await use(registerPage);
    }
});

export { expect } from "@playwright/test";
```

> Note: `loginPage` is used here consistently. If your current code uses `logingPage`, it is better to correct the spelling to `loginPage`.

---

# 4. Understand the Fixture File Step by Step

## Step 1 — Import Playwright's test

```ts
import { test as baseTest } from "@playwright/test";
```

Playwright provides `test`.

We rename it locally to `baseTest`.

```text
Playwright test
      ↓
   baseTest
```

Why?

Because we are going to extend the existing Playwright test with our own fixtures.

---

## Step 2 — Import the Page Object classes

```ts
import { LoginPage } from "../pages/LoginPage";
import { RegisterPage } from "../pages/RegisterPage";
```

We need these classes because the fixture will create objects from them.

For example:

```ts
new LoginPage(page)
```

creates a `LoginPage` object.

---

# 5. Define the Fixture Types

```ts
type Fixtures = {
    loginPage: LoginPage;
    registerPage: RegisterPage;
};
```

This tells TypeScript:

> Our customized test will have two additional fixtures.

```text
Fixtures
│
├── loginPage    → LoginPage
└── registerPage → RegisterPage
```

### Very important

This:

```ts
loginPage: LoginPage;
```

does **not** create an object.

It only describes the type:

> "`loginPage` will be a `LoginPage`."

The object is created later:

```ts
const loginPage = new LoginPage(page);
```

---

# 6. Extend the Base Test

```ts
export const test = baseTest.extend<Fixtures>({
```

Read this as:

> **Take Playwright's existing test and add my `Fixtures` to it.**

Conceptually:

```text
Playwright baseTest
        +
our fixtures
        ↓
custom test
```

Our custom `test` now knows about:

```text
loginPage
registerPage
```

---

# 7. How the loginPage Fixture Works

```ts
loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
}
```

Don't memorize this line.

Understand the three steps.

### Step 1 — Get Playwright's `page`

```ts
{ page }
```

`page` is Playwright's built-in `page` fixture.

It represents the browser page used by the test.

---

### Step 2 — Create our Page Object

```ts
const loginPage = new LoginPage(page);
```

This creates a `LoginPage` object using Playwright's `page`.

Conceptually:

```text
LoginPage class
       ↓
new LoginPage(page)
       ↓
LoginPage object
```

---

### Step 3 — Provide the object to the test

```ts
await use(loginPage);
```

Think of `use()` as:

> **"Here is the object. Give it to the test."**

So the complete flow is:

```text
page
 ↓
new LoginPage(page)
 ↓
loginPage object
 ↓
use(loginPage)
 ↓
test receives loginPage
```

---

# 8. The registerPage Fixture

It follows exactly the same algorithm:

```ts
registerPage: async ({ page }, use) => {
    const registerPage = new RegisterPage(page);
    await use(registerPage);
}
```

Algorithm:

```text
Get page
   ↓
Create RegisterPage object
   ↓
Provide object using use()
   ↓
Test receives registerPage
```

---

# 9. Why `use()` Is Important

This:

```ts
const loginPage = new LoginPage(page);
```

only creates the object.

The test does not automatically receive it just because you created it.

This:

```ts
await use(loginPage);
```

is the point where the fixture provides the value to the test.

Think:

```text
Create
  ↓
Provide
  ↓
Test uses it
```

---

# 10. Export `expect`

```ts
export { expect } from "@playwright/test";
```

This allows test files to import both `test` and `expect` from the same fixture file:

```ts
import { expect, test } from "../src/fixtures/pagefixtures";
```

---

# 11. How the Fixture Is Used in a Test

Example:

```ts
import { expect, test } from "../src/fixtures/pagefixtures";

test.beforeEach("Launch Application", async ({ loginPage }) => {
    await loginPage.goToLoginPage();
});

test("verify Register Page", async ({ loginPage, registerPage }) => {
    await loginPage.navToRegister();

    expect(await registerPage.checkRegisterPage()).toBe(0);
});
```

---

# 12. Understand the Test Code

## `beforeEach`

```ts
test.beforeEach("Launch Application", async ({ loginPage }) => {
    await loginPage.goToLoginPage();
});
```

The test needs `loginPage`.

So we request it:

```ts
{ loginPage }
```

The fixture creates and provides it.

Then:

```ts
await loginPage.goToLoginPage();
```

calls the method defined in the `LoginPage` class.

Conceptually:

```text
Fixture
   ↓
LoginPage object
   ↓
goToLoginPage()
   ↓
Application login page
```

---

# 13. The Actual Test

```ts
test("verify Register Page", async ({ loginPage, registerPage }) => {
```

The test requests two fixtures:

```text
loginPage
registerPage
```

The fixture system provides both.

Then:

```ts
await loginPage.navToRegister();
```

uses the `LoginPage` Page Object to navigate to the Register page.

Then:

```ts
expect(await registerPage.checkRegisterPage()).toBe(0);
```

uses the `RegisterPage` Page Object to verify the Register page.

---

# 14. Complete Execution Flow

This is the most important diagram to remember.

```text
                     TEST STARTS
                          │
                          ↓
             Test requests loginPage
                          │
                          ↓
              Fixture creates object
                          │
                          ↓
                new LoginPage(page)
                          │
                          ↓
                  use(loginPage)
                          │
                          ↓
               Test receives object
                          │
                          ↓
              goToLoginPage()
                          │
                          ↓
                Login Page displayed
                          │
                          ↓
               Test requests registerPage
                          │
                          ↓
              Fixture creates object
                          │
                          ↓
             new RegisterPage(page)
                          │
                          ↓
                use(registerPage)
                          │
                          ↓
               Test receives object
                          │
                          ↓
                navToRegister()
                          │
                          ↓
               Register Page displayed
                          │
                          ↓
              checkRegisterPage()
                          │
                          ↓
                  expect(...).toBe(0)
                          │
                     PASS / FAIL
```

---

# 15. What Exactly Is `test`?

In Playwright, `test` is a **function object**.

For beginner-level understanding:

> **`test` is a Playwright function used to define tests.**

It also has methods such as:

```ts
test.beforeEach(...)
test.afterEach(...)
test.describe(...)
```

Mental model:

```text
test
│
├── test(...)          → define a test
├── beforeEach(...)    → method
├── afterEach(...)     → method
└── describe(...)      → method
```

So:

```ts
test("Login test", async () => {});
```

calls the `test` function.

And:

```ts
test.beforeEach(...)
```

calls the `beforeEach` method associated with the Playwright test function object.

---

# 16. What Is `baseTest`?

This:

```ts
import { test as baseTest } from "@playwright/test";
```

means:

```text
Playwright's test
       ↓
renamed locally as
       ↓
baseTest
```

Then:

```ts
baseTest.extend<Fixtures>(...)
```

creates a customized test with our additional fixtures.

So:

```text
baseTest
   ↓
extend with Fixtures
   ↓
custom test
```

---

# 17. What Is `LoginPage` vs `loginPage`?

This distinction is very important.

### `LoginPage`

```ts
LoginPage
```

is the class/type.

Think:

> Blueprint.

### `loginPage`

```ts
const loginPage = new LoginPage(page);
```

is an object created from that class.

Think:

> Actual object created from the blueprint.

### Simple mental map

```text
LoginPage
   ↓
class / type
   ↓
new LoginPage(page)
   ↓
loginPage
   ↓
object
```

---

# 18. What Does the Type Definition Do?

```ts
type Fixtures = {
    loginPage: LoginPage;
};
```

Think:

> "Describe what my fixture will provide."

It does not create:

```ts
new LoginPage(page)
```

It only tells TypeScript:

```text
loginPage → LoginPage
```

The actual creation happens here:

```ts
const loginPage = new LoginPage(page);
```

---

# 19. Why Can the Test Simply Write `loginPage`?

You might wonder:

> "Where did this `loginPage` variable come from?"

When you write:

```ts
test("Login", async ({ loginPage }) => {
```

the `loginPage` value is being supplied by the fixture system.

You do not need to write:

```ts
myFixture.loginPage
```

inside the test.

The fixture is designed to inject the requested fixture directly into the test callback.

So:

```ts
async ({ loginPage }) => {
```

means:

> "Give this test the fixture named `loginPage`."

---

# 20. Why Use Fixtures Instead of Creating Page Objects in Every Test?

Without fixture:

```ts
test("Login test", async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.login();
});
```

Another test:

```ts
test("Register test", async ({ page }) => {
    const loginPage = new LoginPage(page);
    const registerPage = new RegisterPage(page);

    // test logic
});
```

With fixtures:

```ts
test("Login test", async ({ loginPage }) => {
    await loginPage.login();
});
```

And:

```ts
test("Register test", async ({ loginPage, registerPage }) => {
    await loginPage.navToRegister();
    // test logic
});
```

The repeated Page Object creation is centralized in the fixture file.

---

# 21. Page Object Responsibility vs Fixture Responsibility

Keep these responsibilities separate.

### Page Object

Responsible for:

```text
Locators
Actions
Page-specific methods
Page-specific behavior
```

Example:

```ts
loginPage.goToLoginPage();
loginPage.navToRegister();
```

### Fixture

Responsible for:

```text
Creating Page Object instances
Providing them to tests
```

Example:

```ts
const loginPage = new LoginPage(page);
await use(loginPage);
```

### Test

Responsible for:

```text
Test scenario
Test steps
Assertions
```

Example:

```ts
await loginPage.navToRegister();
expect(await registerPage.checkRegisterPage()).toBe(0);
```

### Mental map

```text
PAGE OBJECT
"How do I interact with this page?"

        ↓

FIXTURE
"How do I provide this Page Object?"

        ↓

TEST
"What scenario am I testing?"
```

---

# 22. The Algorithm You Should Remember

Don't memorize this:

```ts
loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
}
```

Instead remember:

```text
I need a LoginPage fixture
        ↓
Get Playwright's page
        ↓
Create LoginPage using page
        ↓
Give LoginPage to the test
```

Then you can write the code yourself.

For any Page Object:

```text
Need Page Object?
      ↓
Get page
      ↓
new PageObject(page)
      ↓
use(object)
```

---

# 23. Adding Another Page Object

Suppose later you create:

```ts
HomePage
```

The thought process is:

### 1. Import it

```ts
import { HomePage } from "../pages/HomePage";
```

### 2. Add it to the fixture type

```ts
type Fixtures = {
    loginPage: LoginPage;
    registerPage: RegisterPage;
    homePage: HomePage;
};
```

### 3. Create and provide it

```ts
homePage: async ({ page }, use) => {
    const homePage = new HomePage(page);
    await use(homePage);
}
```

### 4. Request it in a test

```ts
test("Home page test", async ({ homePage }) => {
    await homePage.someMethod();
});
```

Notice the algorithm did not change.

```text
Import
  ↓
Declare fixture type
  ↓
Create object
  ↓
use(object)
  ↓
Request fixture in test
```

---

# 24. Important Naming Rule

Be consistent.

Prefer:

```ts
loginPage
```

not:

```ts
logingPage
```

A spelling mistake in a fixture name can cause confusion because the fixture name used in the type, fixture implementation, and test must correspond.

Recommended:

```ts
type Fixtures = {
    loginPage: LoginPage;
    registerPage: RegisterPage;
};
```

And:

```ts
loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
}
```

And:

```ts
test("Login", async ({ loginPage }) => {
});
```

---

# 25. Important `await use(...)` Note

A fixture can provide its value with:

```ts
await use(loginPage);
```

This is the preferred form for the async fixture pattern.

The important concept is still:

```text
create object
      ↓
await use(object)
      ↓
test uses object
```

For your current learning, focus on the purpose of `use()` rather than treating it as a method you need to memorize independently.

---

# 26. Quick Revision Table

| Concept | Simple Meaning |
|---|---|
| `test` | Playwright test function/function object |
| `baseTest` | Playwright's original `test`, renamed locally |
| `baseTest.extend()` | Create a customized test with additional fixtures |
| `Fixtures` | Type describing the fixtures we provide |
| `LoginPage` | Page Object class/type |
| `loginPage` | Instance/object of `LoginPage` |
| `page` | Playwright's built-in Page fixture |
| `new LoginPage(page)` | Creates a LoginPage object |
| `use(loginPage)` | Provides the object to the test |
| `{ loginPage }` in test | Requests the `loginPage` fixture |
| `expect()` | Playwright assertion function |
| `test.beforeEach()` | Runs before each test in its scope |

---

# 27. Final Mental Map

```text
                    PLAYWRIGHT
                         │
                         ↓
                    baseTest
                         │
                         │ extend()
                         ↓
                CUSTOMIZED test
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
         loginPage             registerPage
              │                     │
              ↓                     ↓
      new LoginPage(page)   new RegisterPage(page)
              │                     │
              ↓                     ↓
           use(...)              use(...)
              │                     │
              └──────────┬──────────┘
                         ↓
                       TEST
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
      loginPage.method()     registerPage.method()
             │                       │
             └───────────┬───────────┘
                         ↓
                     ASSERTION
                         ↓
                    PASS / FAIL
```

# 28. What You Actually Need to Remember

### Remember these 6 things:

1. **`test` is a Playwright function/function object used to define tests.**
2. **`baseTest.extend()` creates a customized test with additional fixtures.**
3. **The `Fixtures` type describes what fixtures are available.**
4. **The fixture creates the Page Object using Playwright's `page`.**
5. **`use(pageObject)` provides that object to the test.**
6. **The test requests the fixture by putting its name inside `{ }`.**

### The core algorithm

```text
DEFINE WHAT I NEED
       ↓
CREATE IT
       ↓
PROVIDE IT
       ↓
REQUEST IT IN TEST
       ↓
USE IT
```

For Page Objects:

```text
LoginPage fixture
       ↓
new LoginPage(page)
       ↓
use(loginPage)
       ↓
async ({ loginPage })
       ↓
loginPage.login()
```

> **If you remember this flow, you don't need to memorize the fixture code. You can reconstruct it.**
