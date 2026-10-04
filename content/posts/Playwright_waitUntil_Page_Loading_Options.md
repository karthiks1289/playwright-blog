# Playwright `waitUntil` — Page Loading Options

## Property: `waitUntil`

It tells Playwright **how far it should wait while loading a page**.

### 1. `commit`

Wait until we get the **server response** and the document starts loading.

### 2. `domcontentloaded`

Wait until the **DOM is loaded and parsed**.

It doesn't wait for images and other resources to load.

### 3. `networkidle`

Wait until there are **no network connections for 500ms**.

### 4. `load`

Wait until the page's **`load` event is fired**, meaning the page's dependent resources like images, CSS and scripts have loaded.

---

## 🧠 Easy Memory

**Commit → DOM → Network → Load**

- `commit` → **Response received**
- `domcontentloaded` → **DOM ready**
- `networkidle` → **Network quiet 500ms**
- `load` → **Page resources loaded**

---

## Practical Example

```ts
import {test} from '@playwright/test'

test('Page Load Options', async({page})=>{

    await page.goto(
        "https://naveenautomationlabs.com/opencart/index.php?route=account/login",
        {waitUntil:'load'}
    );

})
```

### What is happening here?

```text
page.goto()
    ↓
waitUntil: 'load'
    ↓
Wait for the page's load event
    ↓
Page resources are loaded
    ↓
goto() completes
```

> **Important:** `waitUntil` is about page navigation/loading. It does **not** guarantee that every element is visible, stable, or that all application/API activity is complete.
