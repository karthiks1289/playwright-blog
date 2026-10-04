# Playwright Browser Context — Complete Notes

## 1. What is Browser Context?

A **Browser Context is an isolated browser session created inside a single Browser instance.**

It allows us to create multiple independent sessions without launching a separate browser instance for every user.

```text
                    Browser
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Context 1    Context 2    Context 3
        Admin        Manager      Employee
```

Each Context behaves like an independent browser session.

---

## 2. How does it work?

A single Browser can contain multiple Browser Contexts.

```text
Browser
   │
   ├── Context 1 → Admin
   ├── Context 2 → Manager
   └── Context 3 → Employee
```

Each Context has its own independent session state.

---

## 3. Is Browser Context a separate process?

**No.**

A Browser Context is **NOT a separate browser process**.

```text
Browser
   │
   ├── Context 1
   ├── Context 2
   └── Context 3
```

Remember:

> **Browser = actual browser instance**  
> **Context = isolated session inside the browser**

---

## 4. What is isolated?

Each Browser Context maintains its own browser session information, such as:

- Cookies
- Local Storage
- Session Storage
- Authentication state
- Other session-related browser state

Therefore, one Context does not normally affect another Context's session.

### Example

```text
Context 1 → Admin → Logged in as Admin

Context 2 → Manager → Logged in as Manager

Context 3 → Employee → Logged in as Employee
```

Admin's login does not automatically make Context 2 or Context 3 logged in as Admin.

---

# 5. Why do we need Browser Context?

This is the important practical question.

We **can** use separate browsers:

```text
Browser 1 → Admin
Browser 2 → Manager
Browser 3 → Employee
```

This works perfectly.

So Browser Context is **not providing something that multiple browsers cannot do**.

The advantage is that we can achieve **multiple isolated sessions without launching a separate browser instance for every user**.

```text
Multiple Browsers

Browser 1 → Admin
Browser 2 → Manager
Browser 3 → Employee
```

versus:

```text
One Browser

Browser
   ├── Context 1 → Admin
   ├── Context 2 → Manager
   └── Context 3 → Employee
```

The second approach is generally more efficient and scalable when we need many independent sessions.

---

# 6. Practical Example — Role-Based Application

Suppose our application has three roles:

```text
Admin
Manager
Employee
```

We need to test:

> Admin creates an employee → Manager approves → Employee verifies.

We can create:

```text
Browser
   │
   ├── Context 1 → Admin
   ├── Context 2 → Manager
   └── Context 3 → Employee
```

Now each role has its own independent login/session.

We can perform actions independently without one user's login affecting another user's session.

---

# 7. Why not simply launch 3 browsers?

We can.

For example:

```text
Browser 1 → Admin
Browser 2 → Manager
Browser 3 → Employee
```

There is nothing wrong with this approach.

But each Browser is a **full browser instance**, so creating many browsers can consume more system resources.

Browser Context allows us to do:

```text
1 Browser
   ↓
Many isolated Contexts
```

This becomes especially useful when testing many users or running scenarios in parallel.

---

# 8. Important distinction

Do NOT remember Browser Context as:

> "A separate browser."

Instead remember it as:

> **"A separate browser session inside a browser."**

### Comparison

```text
Separate Browser:

Browser
   ↓
Separate browser instance
   ↓
Separate session


Browser Context:

Browser
   ↓
Context
   ↓
Separate session
```

---

# 9. Simple real-world analogy

Think about a **hotel**.

```text
Hotel = Browser

Room 101 = Context 1
Room 102 = Context 2
Room 103 = Context 3
```

Each room is independent.

But you don't build a completely new hotel for every room.

Similarly:

```text
1 Browser
   ↓
Multiple isolated Contexts
```

---

# 10. Practical Playwright Example

The following example creates **two isolated Browser Contexts** from the same Browser instance.

- Context One logs in as `ibp@invalid.com`
- Context Two logs in as `iscips@invalid.com`
- Each Context has its own Page and login/session
- We verify that each account shows its own email address

```typescript
import { test, Page, BrowserContext, expect } from '@playwright/test';

test('Browser Context (Multiple Isolated Sessions of Same Browser)', async ({ browser }) => {

    // -------------------------------
    // Context One → User 1
    // -------------------------------

    let contextOne: BrowserContext = await browser.newContext();
    let pageOne: Page = await contextOne.newPage();

    await pageOne.goto(
        'https://naveenautomationlabs.com/opencart/index.php?route=account/login'
    );

    await pageOne
        .getByRole('textbox', { name: 'E-Mail Address' })
        .fill('ibp@invalid.com');

    await pageOne.getByLabel('Password').fill('test');

    await pageOne
        .getByRole('button', { name: 'Login' })
        .click();

    await pageOne
        .getByRole('link', { name: 'Edit your account information' })
        .click();

    await expect(
        pageOne.getByRole('textbox', { name: '* E-Mail' })
    ).toHaveAttribute('value', 'ibp@invalid.com');


    // -------------------------------
    // Context Two → User 2
    // -------------------------------

    let contextTwo: BrowserContext = await browser.newContext();
    let pageTwo: Page = await contextTwo.newPage();

    await pageTwo.goto(
        'https://naveenautomationlabs.com/opencart/index.php?route=account/login'
    );

    await pageTwo
        .getByRole('textbox', { name: 'E-Mail Address' })
        .fill('iscips@invalid.com');

    await pageTwo.getByLabel('Password').fill('test');

    await pageTwo
        .getByRole('button', { name: 'Login' })
        .click();

    await pageTwo
        .getByRole('link', { name: 'Edit your account information' })
        .click();

    await expect(
        pageTwo.getByRole('textbox', { name: '* E-Mail' })
    ).toHaveAttribute('value', 'iscips@invalid.com');

});
```

---

# 11. Understand the Code Through the Architecture

The most important part of the code is:

```typescript
let contextOne: BrowserContext = await browser.newContext();
```

This creates:

```text
Browser
   │
   └── Context One
```

Then:

```typescript
let pageOne: Page = await contextOne.newPage();
```

creates a Page inside Context One:

```text
Browser
   │
   └── Context One
          │
          └── Page One
```

Then we create another Context:

```typescript
let contextTwo: BrowserContext = await browser.newContext();
```

Now:

```text
Browser
   ├── Context One
   │      └── Page One
   │
   └── Context Two
          └── Page Two
```

This is the key concept.

---

# 12. What is actually happening in this example?

### Context One

```text
Browser
   │
   └── Context One
          │
          └── Page One
                 │
                 └── Login as ibp@invalid.com
```

The login/session belongs to **Context One**.

### Context Two

```text
Browser
   │
   └── Context Two
          │
          └── Page Two
                 │
                 └── Login as iscips@invalid.com
```

The login/session belongs to **Context Two**.

Therefore:

```text
Context One ≠ Context Two

User 1 session ≠ User 2 session
```

---

# 13. What does the test prove?

The test proves that two different users can use the **same Browser instance** while maintaining separate sessions.

```text
                    SAME BROWSER
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
        Context One             Context Two
             │                       │
        User 1 login             User 2 login
             │                       │
      ibp@invalid.com         iscips@invalid.com
```

The final assertions verify that each Context displays the correct user's email.

---

# 14. Why is this better than sharing one Context?

If we used the **same Context** for both users:

```text
Browser
   │
   └── Context
         │
         ├── User 1 login
         │
         └── User 2 login
```

User 2 could overwrite User 1's authentication/session.

That is **not isolated multi-user testing**.

Instead:

```text
Browser
   │
   ├── Context 1 → User 1
   │
   └── Context 2 → User 2
```

Now both sessions are independent.

---

# 15. Browser Context vs Browser

| | Browser | Browser Context |
|---|---|---|
| Represents | Browser instance | Isolated browser session |
| Separate browser process? | Yes, browser instance is launched | No |
| Multiple can exist? | Yes | Yes, inside a Browser |
| Own cookies? | — | Yes |
| Own local storage? | — | Yes |
| Own authentication/session state? | — | Yes |
| Useful for multi-user testing? | Yes | Yes |
| Main benefit | Actual browser | Efficient session isolation |

---

# 16. Browser Context vs Page

This is another important distinction.

```text
Browser
   │
   ├── Context 1
   │      ├── Page 1
   │      └── Page 2
   │
   └── Context 2
          └── Page 3
```

### Browser

The actual browser instance.

### Context

An isolated browser session.

### Page

A tab/page inside a Context.

So remember:

> **Browser → Context → Page**

---

# 17. Mental Map

```text
                    BROWSER
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      CONTEXT 1    CONTEXT 2    CONTEXT 3
        Admin        Manager      Employee
          │            │            │
       Session      Session      Session
          │            │            │
       Cookies      Cookies      Cookies
       Storage      Storage      Storage
       Login        Login        Login
          │            │            │
        Page          Page         Page
```

### The easiest memory trick:

> **1 Browser → Multiple Contexts → Multiple Isolated Sessions**

And:

> **Context ≠ Browser Process**

---

# 18. Your Original Summary — Corrected

### Browser Context:

1. **Getting multiple isolated browser sessions (Browser Contexts) from a single Browser instance.**

2. **Each Browser Context is isolated separately.** They maintain their own cookies, storage, authentication/session state, etc., and do not share session state with other contexts.

3. **Advantage:**

   If my application is role-based, I can create different Browser Contexts for different users/roles and perform actions independently.

4. **Important:** We can achieve the same user separation by launching separate browsers, but Browser Context allows us to create those isolated sessions **inside one Browser instance**, making the approach more efficient and scalable.

---

# 19. Interview-Ready Answer

> **"Browser Context is an isolated browser session created inside a Browser instance. Multiple contexts can exist within one browser, and each context maintains its own cookies, storage, and authentication state. This is useful for testing multiple users or roles independently. Although we can achieve the same isolation by launching separate browsers, contexts are more efficient because we don't need a separate browser instance for every user."**

---

# 20. Senior SDET Perspective

In real automation frameworks, Browser Context is particularly useful for:

- Multi-user/role-based testing
- Parallel scenarios
- Maintaining independent authentication states
- Avoiding unnecessary browser launches
- Creating isolated test environments

The key architectural idea is:

> **Browser represents the browser instance; Context represents an isolated user/session environment within that browser.**
