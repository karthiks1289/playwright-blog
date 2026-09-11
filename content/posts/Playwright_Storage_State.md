# Playwright Storage State

## Simple Definition

**Storage State is a Playwright feature used to save the user's login/session information into a JSON file and reuse it in tests.**

This allows us to **skip the login step** and directly navigate to the required page.

## Why do we use it?

Normally:

```text
Test → Login → Enter Username → Enter Password → Submit → Home Page
```

With Storage State:

```text
Login once → Save session → Reuse session → Directly access required page
```

👉 **Main benefit: Save execution time by avoiding repeated login.**

---

# How to Use Storage State

## Step 1: Login and Save the Session

First, automate the login functionality.

After successful login, save the browser context's storage state into a JSON file.

```typescript
await page.context().storageState({ path: 'auth.json' });
```

### What does this mean?

- `page.context()` → gets the current Browser Context
- `storageState()` → captures the session/storage information
- `auth.json` → stores that information

### Example

```typescript
import { test } from '@playwright/test';

test('Login and Save Storage State', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.getByLabel('Username').fill('admin');
    await page.getByLabel('Password').fill('password');
    await page.getByRole('button', { name: 'Login' }).click();

    await page.context().storageState({ path: 'auth.json' });
});
```

After this test runs, Playwright creates:

```text
auth.json
```

This file contains the required browser storage information, such as **cookies and local storage**, needed to restore the session.

---

# Step 2: Reuse the Saved Session

In the test/spec file, configure Playwright to use the saved JSON file.

```typescript
test.use({ storageState: 'auth.json' });
```

Then write your test normally:

```typescript
import { test } from '@playwright/test';

test.use({ storageState: 'auth.json' });

test('Access Home Page Without Login', async ({ page }) => {

    await page.goto('https://example.com/home');

});
```

## What happens?

Playwright creates the page with the **saved authentication state**.

So you don't need to do:

```text
Open Login Page
      ↓
Enter Username
      ↓
Enter Password
      ↓
Click Login
      ↓
Navigate to Home
```

Instead:

```text
auth.json
   ↓
Playwright restores session
   ↓
Open Page
   ↓
Directly access Home Page
```

---

# 🧠 Mental Map

Remember:

## Storage State = SAVE → REUSE

```text
LOGIN
  ↓
Successful Login
  ↓
Save Storage State
  ↓
auth.json
  ↓
Reuse in Tests
  ↓
Skip Login
  ↓
Directly access required page
```

---

# Interview Answer

> **Playwright Storage State allows us to save authentication-related browser state, such as cookies and local storage, into a JSON file and reuse it in other tests. This helps us avoid repeating the login process for every test and improves test execution time.**

---

# Important Syntax

### Save

```typescript
await page.context().storageState({ path: 'auth.json' });
```

### Reuse

```typescript
test.use({ storageState: 'auth.json' });
```

### Easy Rule

**`storageState({ path: 'auth.json' })` → SAVE**

**`test.use({ storageState: 'auth.json' })` → REUSE**
