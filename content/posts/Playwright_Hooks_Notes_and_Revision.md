# Playwright Hooks — Complete Notes

> **Purpose:** Real-project understanding + interview preparation + quick revision  
> **Level:** Beginner-friendly, practical  
> **Core idea:** Hooks control what happens automatically before/after tests.

---

# 1. What is a Playwright Hook?

A **hook** is code that Playwright automatically runs at a particular point in the test lifecycle.

Instead of repeating common setup in every test:

```ts
test('Create Account', async ({ page }) => {
    await login(page);
    // test
});

test('Edit Account', async ({ page }) => {
    await login(page);
    // test
});
```

we can use:

```ts
test.beforeEach(async ({ page }) => {
    await login(page);
});
```

Now Playwright automatically runs `login()` before every test.

### Mental Model

```text
Hook = "Do this automatically at this point"
```

---

# 2. The Four Main Hooks

The four hooks you MUST know are:

```text
beforeAll
beforeEach
afterEach
afterAll
```

The easiest memory trick:

```text
ALL  → once
EACH → every test
```

## Lifecycle Mental Model

```text
                 TEST SUITE
                     │
                 beforeAll
                     │
          ┌──────────┴──────────┐
          │                     │
      beforeEach            beforeEach
          │                     │
        Test 1               Test 2
          │                     │
      afterEach             afterEach
          │                     │
          └──────────┬──────────┘
                     │
                  afterAll
```

---

# 3. `beforeEach` — MUST KNOW

## Meaning

`beforeEach` runs **before every test**.

```ts
test.beforeEach(async ({ page }) => {
    await page.goto('/login');
});
```

If there are three tests:

```text
beforeEach → Test 1
beforeEach → Test 2
beforeEach → Test 3
```

## Real Project Example

Suppose every Account test requires login:

```ts
test.beforeEach(async ({ page }) => {
    await login(page);
});
```

Then:

```ts
test('Create Account', async ({ page }) => {
    // create account
});

test('Edit Account', async ({ page }) => {
    // edit account
});
```

Both tests automatically receive the common setup.

## Common Uses

- Login
- Navigation to a common page
- Preparing common test state
- Other setup required by every test

## Interview Answer

> `beforeEach` runs before every test and is commonly used for test-level setup such as login or navigation.

---

# 4. `afterEach` — MUST KNOW

## Meaning

`afterEach` runs **after every test**.

```ts
test.afterEach(async ({ page }) => {
    console.log('Test completed');
});
```

Mental model:

```text
beforeEach
    ↓
TEST
    ↓
afterEach
```

## Common Uses

- Cleanup
- Reset test data
- Logging
- Custom reporting/actions after a test

Example:

```ts
test.afterEach(async ({ page }, testInfo) => {
    console.log(`Finished: ${testInfo.title}`);
});
```

## Important

Do not put actions into `afterEach` just because the hook exists.

Use it when the action is genuinely needed.

---

# 5. `beforeAll` — MUST KNOW

## Meaning

`beforeAll` runs **once before the tests in its scope**.

```ts
test.beforeAll(async () => {
    console.log('Runs once');
});
```

Execution:

```text
beforeAll
   ↓
Test 1
   ↓
Test 2
   ↓
Test 3
```

Only one `beforeAll` execution occurs for that scope.

## Common Uses

Use it for setup that does not need to happen before every test.

Examples:

- Preparing shared external data
- One-time setup
- Expensive setup that genuinely needs to happen once

## Important Interview Point

`beforeAll` does **NOT** mean:

> "All tests use the same page."

Do not confuse hook lifecycle with Playwright's browser/page fixtures and test isolation.

## Interview Answer

> `beforeAll` runs once before the tests in its scope and is useful for one-time setup.

---

# 6. `afterAll` — MUST KNOW

## Meaning

`afterAll` runs **once after the tests in its scope finish**.

```ts
test.afterAll(async () => {
    console.log('Cleanup once');
});
```

Mental model:

```text
beforeAll
    ↓
Test 1
    ↓
Test 2
    ↓
Test 3
    ↓
afterAll
```

## Common Uses

- One-time cleanup
- Cleanup of resources created by `beforeAll`

---

# 7. The Most Important Comparison

| Hook | When? | Think |
|---|---|---|
| `beforeAll` | Once before tests | Prepare once |
| `beforeEach` | Before every test | Prepare every test |
| `afterEach` | After every test | Clean up every test |
| `afterAll` | Once after tests | Clean up once |

### Memory Trick

```text
ALL
 ↓
Once

EACH
 ↓
Every test
```

---

# 8. Hooks Inside `test.describe()` — MUST KNOW

Hooks can be scoped to a `test.describe()` block.

```ts
test.describe('Account Tests', () => {

    test.beforeEach(async ({ page }) => {
        await page.goto('/accounts');
    });

    test('Create Account', async ({ page }) => {
    });

    test('Edit Account', async ({ page }) => {
    });

});
```

The `beforeEach` applies to the tests inside that `describe` scope.

Mental model:

```text
Account Tests
│
├── beforeEach
│
├── Create Account
│
└── Edit Account
```

### Important Idea

```text
describe = container/group

hook inside describe
        ↓
affects tests inside that group
```

---

# 9. Scope and Nested Hooks — SHOULD KNOW

You can have hooks at different scopes.

Example:

```ts
test.beforeEach(async () => {
    console.log('Common setup');
});

test.describe('Orders', () => {

    test.beforeEach(async () => {
        console.log('Order setup');
    });

    test('Create Order', async () => {});
});
```

The Order test can receive setup from both applicable scopes.

You do not need to memorize complicated execution rules immediately.

Remember:

> **Hooks are scope-based.**

---

# 10. Hook vs Fixture — MUST KNOW

This is an important Playwright interview topic.

## Hook

Controls **WHEN something happens**.

```ts
test.beforeEach(async ({ page }) => {
    await login(page);
});
```

Think:

```text
HOOK
 ↓
WHEN?
```

## Fixture

Provides something the test can use.

```ts
test('Create Account', async ({ page }) => {
});
```

Here `page` is a Playwright fixture.

Think:

```text
FIXTURE
 ↓
WHAT do I get?
```

### Easy Memory

```text
Hook    → WHEN
Fixture → WHAT
```

---

# 11. Don't Put Everything Inside Hooks — MUST KNOW

Avoid a huge `beforeEach`:

```ts
test.beforeEach(async ({ page }) => {

    // login

    // create customer

    // create account

    // configure application

    // 50 more lines...
});
```

Now every test has a large hidden setup.

Better:

```ts
test.beforeEach(async ({ page }) => {
    await login(page);
});
```

Keep reusable logic separately:

```ts
async function login(page) {
    await page.goto('/login');

    await page.getByLabel('Username').fill('admin');
    await page.getByLabel('Password').fill('password');

    await page.getByRole('button', { name: 'Login' }).click();
}
```

### Rule

> **A hook should manage setup; it should not become a dumping ground for automation code.**

---

# 12. Test Independence — MUST KNOW

Avoid test dependencies like this:

```text
Test A
  ↓
creates customer

Test B
  ↓
needs customer created by Test A

Test C
  ↓
needs Test B
```

If Test B fails, Test C may also fail.

Prefer:

```text
Test A → independent
Test B → independent
Test C → independent
```

## Why?

Independent tests:

- can run separately
- are easier to debug
- can run in parallel
- do not depend on another test's success

## Interview Answer

> Tests should be independent so that one test does not affect another and tests can run reliably and in parallel.

---

# 13. What If `beforeEach` Fails? — SHOULD KNOW

Example:

```ts
test.beforeEach(async ({ page }) => {
    await login(page);
});
```

If the setup fails, the test cannot proceed with the expected setup.

Mental model:

```text
beforeEach ❌
    ↓
Test cannot execute normally
```

Therefore:

> Hooks should contain reliable and necessary setup only.

---

# 14. `test.step()` — SHOULD KNOW

`test.step()` is useful for organizing actions inside one test.

It is **NOT a hook**.

Example:

```ts
test('Create Account', async ({ page }) => {

    await test.step('Login', async () => {
        await login(page);
    });

    await test.step('Create Account', async () => {
        await page.getByLabel('Account Name').fill('ABC Corp');
        await page.getByRole('button', { name: 'Save' }).click();
    });

    await test.step('Verify Account', async () => {
        await expect(page.getByText('Account created')).toBeVisible();
    });

});
```

Mental model:

```text
TEST
 │
 ├── STEP: Login
 │
 ├── STEP: Create Account
 │
 └── STEP: Verify Account
```

## Why use `test.step()`?

Mainly:

- Better readability
- Better reporting
- Easier debugging
- Makes logical/business actions visible

### Important

`test.step()` does **not** create another test.

You still have:

```text
1 test
```

with:

```text
Step 1
Step 2
Step 3
```

---

# 15. Don't Make `test.step()` Too Granular

Avoid:

```ts
await test.step('Fill username', async () => {
    await page.getByLabel('Username').fill('admin');
});

await test.step('Fill password', async () => {
    await page.getByLabel('Password').fill('password');
});

await test.step('Click login', async () => {
    await page.getByRole('button', { name: 'Login' }).click();
});
```

Better:

```ts
await test.step('Login', async () => {
    await page.getByLabel('Username').fill('admin');
    await page.getByLabel('Password').fill('password');
    await page.getByRole('button', { name: 'Login' }).click();
});
```

### Rule

> A step should represent a meaningful logical/business action, not every individual Playwright command.

---

# 16. `test.step()` Can Return a Value

A step can return a value.

```ts
const accountId = await test.step('Create Account', async () => {

    await page.getByRole('button', { name: 'Save' }).click();

    return await getAccountId();
});
```

Now:

```ts
accountId
```

contains the returned value.

This is useful, but it is not something you need to master on Day 1.

---

# 17. `test.describe()` — MUST KNOW

`test.describe()` groups related tests.

Example:

```ts
test.describe('Account Tests', () => {

    test('Create Account', async ({ page }) => {
    });

    test('Edit Account', async ({ page }) => {
    });

    test('Delete Account', async ({ page }) => {
    });

});
```

Mental model:

```text
Account Tests
 ├── Create Account
 ├── Edit Account
 └── Delete Account
```

### Why use it?

- Organization
- Grouping related tests
- Creating a scope for hooks
- Making reports easier to understand

---

# 18. `describe` vs `step`

This is one of the most important comparisons.

| | `test.describe()` | `test.step()` |
|---|---|---|
| Purpose | Group tests | Group actions |
| Level | Test suite | Inside one test |
| Creates test? | No, it groups tests | No |
| Main benefit | Organization + scope | Reporting + readability |
| Contains | Tests and hooks | Actions |
| Example | Account Tests | Login |

### Mental Model

```text
test.describe()
       ↓
GROUP OF TESTS

test.step()
       ↓
GROUP OF ACTIONS
```

---

# 19. `describe` + Hook + Test + Step

This is the complete picture.

```ts
test.describe('Account Tests', () => {

    test.beforeEach(async ({ page }) => {
        await login(page);
    });

    test('Create Account', async ({ page }) => {

        await test.step('Navigate to Accounts', async () => {
            await page.goto('/accounts');
        });

        await test.step('Create Account', async () => {
            await page.getByLabel('Account Name').fill('ABC Corp');
            await page.getByRole('button', { name: 'Save' }).click();
        });

        await test.step('Verify Account', async () => {
            await expect(page.getByText('ABC Corp')).toBeVisible();
        });

    });

});
```

Mental map:

```text
Account Tests                  ← describe
│
├── beforeEach                 ← hook
│
└── Create Account             ← test
      │
      ├── Navigate              ← step
      ├── Create Account        ← step
      └── Verify Account        ← step
```

---

# 20. Hooks vs `describe` vs `step`

Remember the three levels:

```text
test.describe()
       ↓
GROUP TESTS

Hook
       ↓
SETUP / CLEANUP

test.step()
       ↓
GROUP ACTIONS
```

Or even simpler:

```text
describe → WHERE tests belong
hook     → WHEN setup/cleanup happens
step     → WHAT logical action is happening
test     → WHAT behavior is being verified
```

---

# 21. Hooks vs Global Setup — SHOULD KNOW

Do not confuse:

```ts
test.beforeEach(...)
```

with global setup.

### Hook

Part of the test lifecycle.

```text
Test lifecycle
     ↓
beforeEach
     ↓
Test
     ↓
afterEach
```

### Global setup

Used for broader setup before the test run.

Mental model:

```text
Global Setup
     ↓
Test execution
     ↓
Hooks
     ↓
Tests
     ↓
Global Teardown
```

For now:

> Understand the difference. Learn advanced global setup when building a framework.

---

# 22. Real Project Example

Imagine Salesforce Account tests:

```ts
test.describe('Account Tests', () => {

    test.beforeEach(async ({ page }) => {
        await login(page);
        await page.goto('/accounts');
    });

    test('Create Account', async ({ page }) => {

        await test.step('Create Account', async () => {
            // create account
        });

        await test.step('Verify Account', async () => {
            // verify account
        });

    });

    test('Edit Account', async ({ page }) => {
        // edit account
    });

    test('Delete Account', async ({ page }) => {
        // delete account
    });

    test.afterEach(async () => {
        // cleanup if required
    });

});
```

Think:

```text
Account Test Suite
       │
       ▼
   beforeEach
       │
       ├── Login
       └── Navigate
       │
       ▼
     Test
       │
       ├── Step: Create
       └── Step: Verify
       │
       ▼
   afterEach
       │
       ▼
   Next Test
```

---

# 23. What You MUST Learn vs Learn Later

## 🔴 MUST KNOW

```text
✓ What is a hook?
✓ beforeAll
✓ beforeEach
✓ afterEach
✓ afterAll

✓ ALL = once
✓ EACH = every test

✓ Hook scope
✓ Hooks inside describe
✓ Hook vs fixture
✓ Test independence
✓ What should/shouldn't go inside hooks

✓ test.describe()
✓ test.step()
✓ describe vs step
✓ Practical framework usage
```

## 🟡 SHOULD KNOW

```text
✓ Nested/scoped hooks
✓ Hooks + fixtures
✓ Hook failure
✓ test.step() return value
✓ Hook vs global setup
```

## ⚪ LEARN LATER

```text
○ Complex custom fixture architecture
○ Advanced worker-level setup
○ Complex global setup/teardown
○ Advanced fixture dependency design
```

Do not spend time memorizing these now.

---

# 24. Interview Questions

## Basic

### Q1. What are Playwright hooks?

> Hooks are lifecycle functions that automatically execute before or after tests.

### Q2. Difference between `beforeAll` and `beforeEach`?

> `beforeAll` runs once for the scope; `beforeEach` runs before every test.

### Q3. Difference between `afterAll` and `afterEach`?

> `afterAll` runs once after the scope finishes; `afterEach` runs after every test.

---

## Intermediate

### Q4. Can hooks be placed inside `test.describe()`?

> Yes. The hook can be scoped to the tests inside that `describe` block.

### Q5. Why use `beforeEach`?

> To avoid repeating common test setup such as login or navigation.

### Q6. Should tests depend on each other?

> No. Tests should generally be independent so they can run reliably and in parallel.

---

## Senior SDET

### Q7. Hook vs fixture?

> A hook controls when setup or cleanup happens, while a fixture provides resources or prepared state to tests.

### Q8. What should you avoid putting inside `beforeEach`?

> Large amounts of business logic or unnecessary setup. Keep it focused on common, required test setup.

### Q9. When would you use `beforeAll` instead of `beforeEach`?

> When setup genuinely needs to happen only once for the relevant scope rather than before every test.

### Q10. What is `test.step()`?

> `test.step()` groups logical actions inside a test and improves readability, reporting, and debugging.

### Q11. Does `test.step()` create a separate test?

> No. It creates a step inside the existing test.

### Q12. Difference between `test.describe()` and `test.step()`?

> `test.describe()` groups related tests; `test.step()` groups related actions within one test.

---

# 25. Quick Revision Card

```text
PLAYWRIGHT HOOKS
================

beforeAll
→ once before tests

beforeEach
→ before EVERY test

afterEach
→ after EVERY test

afterAll
→ once after tests


MEMORY:
ALL  = ONCE
EACH = EVERY TEST


DESCRIBE
→ GROUP TESTS
→ creates scope
→ can contain hooks


STEP
→ GROUP ACTIONS
→ inside ONE test
→ improves reporting/readability


HOOK vs FIXTURE
→ Hook = WHEN
→ Fixture = WHAT


TEST
→ Should be independent


GOOD HOOK
→ small
→ common
→ necessary
→ predictable


BAD HOOK
→ huge
→ hidden business logic
→ unnecessary setup
```

---

# 26. Final Mental Map

```text
                    TEST FILE
                       │
                       ▼
                test.describe()
                  GROUP TESTS
                       │
              ┌────────┴────────┐
              │                 │
            TEST              TEST
              │                 │
              ▼                 ▼
        test.step()       test.step()
        GROUP ACTIONS     GROUP ACTIONS
```

Hooks sit around the tests:

```text
describe
   │
   ├── beforeEach
   │
   ├── Test
   │    ├── step
   │    ├── step
   │    └── step
   │
   └── afterEach
```

### The 4 words to remember

```text
describe → GROUP TESTS

hook → SETUP / CLEANUP

step → GROUP ACTIONS

test → VERIFY BEHAVIOR
```

---

# 27. Final Interview Mental Model

If an interviewer gives you a Playwright framework and asks you to explain its structure:

```text
describe
   ↓
organizes related tests

beforeEach
   ↓
prepares every test

test
   ↓
verifies functionality

test.step
   ↓
organizes meaningful actions inside the test

afterEach
   ↓
cleans up every test
```

And for one-time operations:

```text
beforeAll → prepare once
afterAll  → clean up once
```

> **If you understand this mental map, you know the Hooks topic at the depth required for real Playwright work and interviews.**
