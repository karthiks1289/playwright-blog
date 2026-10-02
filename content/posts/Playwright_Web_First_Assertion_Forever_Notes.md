# Playwright Web-First Assertion — Forever Notes

## 1. Core Meaning

**Web-first:** First locate/identify the element on the web page.

**Assertion:** Then check the expected condition of that element.

## 2. Mental Model

```text
🌐 Web Page
   ↓
🔎 Locate the element
   ↓
❓ Check expected condition
   ↓
⏳ Wait + verify
   ↓
✅ PASS / ❌ FAIL
```

## 3. Example

```ts
await expect(page.getByText("Welcome")).toBeVisible();
```

Think:

> **“Find Welcome on the web page → check whether it is visible.”**

## 4. One-Line Memory

> **Web-first = Where is the element? | Assertion = Is it in the expected state?**

## 5. Important Distinction

```ts
page.getByText("Welcome")
```

= **Locator/query** — describes how Playwright can locate the element.

```ts
await expect(page.getByText("Welcome")).toBeVisible();
```

= **Web-first assertion** — checks the element's expected condition and waits for it to become true.
