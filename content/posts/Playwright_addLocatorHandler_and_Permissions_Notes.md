# Playwright — `addLocatorHandler()` & Browser Permissions

## 1. `addLocatorHandler()`

### What is it?

`addLocatorHandler()` is used to automatically handle an **unexpected UI element** that appears during a test.

Examples:

- Cookie banner
- HTML modal
- Login/session popup
- "Got it" notification
- Unexpected overlay

### Mental Map

```text
Unexpected HTML/UI element
        ↓
addLocatorHandler()
        ↓
Detect the locator
        ↓
Run handler code
        ↓
Continue test
```

### Syntax

```ts
await page.addLocatorHandler(
    page.getByRole('dialog'),
    async () => {
        await page.getByRole('button', { name: 'Close' }).click();
    }
);
```

Think:

```text
Locator  → WHAT should I detect?
Handler  → WHAT should I do?
```

---

## 2. What `addLocatorHandler()` DOES NOT Handle

It does **not** handle JavaScript dialogs.

JavaScript dialogs include:

- `alert()`
- `confirm()`
- `prompt()`

For these, use the `dialog` event:

```ts
page.on('dialog', async dialog => {
    await dialog.accept();
});
```

### Easy comparison

| Situation | Playwright solution |
|---|---|
| HTML/UI modal | `addLocatorHandler()` |
| Cookie banner | `addLocatorHandler()` |
| Unexpected DOM popup | `addLocatorHandler()` |
| JavaScript `alert()` | `page.on('dialog')` |
| JavaScript `confirm()` | `page.on('dialog')` |
| JavaScript `prompt()` | `page.on('dialog')` |

### Memory Trick

> **UI popup → Locator Handler**  
> **JS popup → Dialog event**

---

# 3. Browser Permissions in `use`

Playwright can grant browser permissions through the `permissions` option inside the `use` configuration.

### Example

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
    use: {
        permissions: ['notifications']
    }
});
```

This tells Playwright to grant the specified browser permission to the browser context.

---

## Common Permissions

Examples include:

```ts
permissions: [
    'notifications',
    'geolocation',
    'camera',
    'microphone'
]
```

You can configure multiple permissions:

```ts
use: {
    permissions: ['notifications', 'geolocation']
}
```

### Mental Map

```text
Playwright config
      ↓
     use
      ↓
 permissions
      ↓
Grant browser permissions
```

---

# 4. `addLocatorHandler()` vs `permissions`

| Feature | Purpose |
|---|---|
| `addLocatorHandler()` | Handles an unexpected **HTML/UI element** |
| `use.permissions` | Grants **browser permissions** |
| `page.on('dialog')` | Handles **JavaScript dialogs** |

### Remember This

```text
HTML/UI popup
    → addLocatorHandler()

JavaScript alert/confirm/prompt
    → page.on('dialog')

Browser permission
    → use.permissions
```

---

# 5. Real-Project Example

Suppose your application has:

1. A cookie popup that sometimes appears.
2. A notification permission requirement.

You can handle them separately.

### Cookie popup

```ts
await page.addLocatorHandler(
    page.getByRole('dialog'),
    async () => {
        await page.getByRole('button', { name: 'Accept' }).click();
    }
);
```

### Notification permission

```ts
use: {
    permissions: ['notifications']
}
```

These solve **different problems**.

---

# 6. Important Interview Point

### Question:
**Does `addLocatorHandler()` handle JavaScript alerts?**

### Answer:

> No. `addLocatorHandler()` is for locator-based UI elements such as HTML popups or overlays. JavaScript dialogs such as alert, confirm, and prompt are handled using Playwright's `dialog` event.

---

# What You Actually Need to Remember

```text
1. addLocatorHandler()
   → unexpected HTML/UI element

2. page.on('dialog')
   → JavaScript alert / confirm / prompt

3. use.permissions
   → browser permissions

4. Locator handler:
   Locator = what to detect
   Handler = what action to perform
```

## One-Line Memory Formula

> **UI Popup = Locator Handler | JS Popup = Dialog | Browser Permission = `use.permissions`**
