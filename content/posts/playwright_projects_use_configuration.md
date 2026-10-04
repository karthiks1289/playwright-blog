# Playwright `projects` and `use` Configuration

## Interview Question

**What happens when `projects` is blank, removed, or when `use` has no configuration?**

---

## 1. `projects: []` is blank

```ts
export default defineConfig({
  projects: []
});
```

### What happens?

- Zero projects are defined.
- Playwright has no project to execute.
- Therefore, no tests are executed.
- Depending on the command/configuration, you may see a message such as **"No tests found."**

### Remember

> `projects: []` → **0 projects → Nothing executes**

---

## 2. `projects` is completely removed

```ts
export default defineConfig({
  use: {
    browserName: 'chromium'
  }
});
```

### What happens?

Playwright uses the **top-level configuration** defined directly inside `defineConfig()`.

For example:

```ts
use: {
  browserName: 'chromium'
}
```

means the tests run using **Chromium**.

### Remember

> No `projects` → **Use the top-level configuration**

---

## 3. `use` also has no browser configuration

```ts
export default defineConfig({
  testDir: './tests'
});
```

There is no:

```ts
use: {
  browserName: 'chromium'
}
```

### What happens?

Playwright uses its **built-in default values** for the relevant options.

For `browserName`, the default is:

> **Chromium**

### Remember

> Setting not configured → **Playwright uses its built-in default**

---

# Final Interview Answer

> **a. If `projects: []` is defined, zero projects are available, so no tests are executed.**
>
> **b. If `projects` is completely removed, Playwright uses the top-level configuration from `playwright.config.ts`.**
>
> **c. If the `use` object also does not specify a browser, Playwright uses its built-in default, which is Chromium.**

---

# 🧠 Mental Map

```text
playwright.config.ts
        |
        v
Is projects: [] ?
   |
   +--- YES ---> 0 projects ---> Nothing executes
   |
   +--- NO / projects removed
                |
                v
        Use top-level config
                |
                v
       Is browserName set?
          /          \
        YES           NO
         |             |
         v             v
   Use that browser   Playwright
                      built-in default
                           |
                           v
                       Chromium
```

## ⚠️ Interview Terminology

Say **"Chromium"** in an interview rather than **"CFT"**.

`CFT` (Chrome for Testing) is an implementation/distribution detail. The Playwright browser/configuration value is **Chromium**.
