# Playwright – SVG Handling

## 1. What is SVG?

SVG (Scalable Vector Graphics) is commonly used for icons, logos, charts, graphs, arrows and illustrations.

As a tester, you usually do not need to understand how SVG draws the image.

Your question is:

> **How do I identify and interact with this SVG using Playwright?**

### Mental map

```text
SVG
 ↓
Inspect HTML
 ↓
Find reliable attribute / accessible name
 ↓
Use normal Playwright locator
 ↓
Interact
```

---

# 2. Most Important Point

**SVG does not require a special Playwright API.**

You normally use Playwright's standard locators:

```ts
page.getByRole()
page.getByTestId()
page.locator()
```

Use `getByLabel()` primarily for **form controls** such as inputs that have an associated label or `aria-label`.

Think:

> **SVG is an element. Identify the actual interactive target and choose the most reliable locator.**

---

# 3. SVG Inside a Button

Example:

```html
<button aria-label="Search">
    <svg>
        <path d="..."></path>
    </svg>
</button>
```

Prefer:

```ts
await page.getByRole("button", { name: "Search" }).click();
```

Do not unnecessarily target:

```ts
await page.locator("svg").click();
```

### Mental map

```text
Button
  ↓
SVG icon
```

The **button** represents the user action. The SVG is usually only the visual icon.

> **If SVG is inside a clickable button, try to locate the button first.**

---

# 4. SVG Inside a Link

```html
<a href="/cart" aria-label="Shopping Cart">
    <svg>
        <path d="..."></path>
    </svg>
</a>
```

Prefer:

```ts
await page.getByRole("link", { name: "Shopping Cart" }).click();
```

---

# 5. SVG With Test ID

```html
<svg data-testid="search-icon">
    <path></path>
</svg>
```

Use:

```ts
await page.getByTestId("search-icon").click();
```

This is useful when the SVG itself is the required target.

---

# 6. SVG With a Stable Attribute

```html
<svg data-icon="search">
    <path></path>
</svg>
```

Use:

```ts
await page.locator('svg[data-icon="search"]').click();
```

Prefer stable attributes over generated classes when possible.

---

# 7. SVG With a Class

```html
<svg class="search-icon">
    <path></path>
</svg>
```

You can use:

```ts
await page.locator("svg.search-icon").click();
```

But don't automatically choose a class just because it exists.

### Locator thinking

```text
Accessible name?
   ↓
Test ID?
   ↓
Stable data-* attribute?
   ↓
Stable CSS/class?
```

Choose the most reliable option.

---

# 8. The `<path>` Inside SVG

Example:

```html
<svg>
    <path d="M10 10..."></path>
</svg>
```

Structure:

```text
svg
 └── path
```

The `<path>` normally describes the drawing.

Avoid using it as your first locator:

```ts
await page.locator("svg path").click();
```

Why?

- It may be an implementation detail.
- An SVG can contain multiple paths.
- Path data can change.
- It is less readable.

Prefer the actual clickable button/link or a stable attribute on the SVG.

---

# 9. Multiple SVG Icons

```html
<svg data-icon="edit"></svg>
<svg data-icon="delete"></svg>
<svg data-icon="download"></svg>
```

Don't use:

```ts
await page.locator("svg").click();
```

Use:

```ts
await page.locator('svg[data-icon="download"]').click();
```

If the SVG is inside a button, prefer the button:

```ts
await page.getByRole("button", { name: "Download" }).click();
```

---

# 10. SVG Locator Chaining

```html
<div data-testid="user-card">
    <button aria-label="Edit user">
        <svg class="edit-icon"></svg>
    </button>
</div>
```

Use:

```ts
await page
    .getByTestId("user-card")
    .getByRole("button", { name: "Edit user" })
    .click();
```

Mental map:

```text
Page
 ↓
User Card
 ↓
Edit Button
 ↓
SVG
```

You don't need to locate the SVG itself.

---

# 11. SVG Used as an Interactive Element

Sometimes the SVG itself is exposed as a button:

```html
<svg
    data-testid="refresh-icon"
    role="button"
    aria-label="Refresh">
</svg>
```

Then:

```ts
await page.getByRole("button", { name: "Refresh" }).click();
```

Prefer the accessible role and accessible name when the SVG is actually exposed with those semantics.

### Important

Do not assume that every:

```html
<svg aria-label="Search">
```

should be located with:

```ts
page.getByLabel("Search")
```

`getByLabel()` is intended primarily for form controls. For an interactive SVG, first inspect its role/accessibility semantics and use an appropriate role locator or a stable attribute.

---

# 12. SVG Charts

Charts are a common real-project scenario.

Example:

```html
<svg>
    <rect data-category="Sales"></rect>
    <rect data-category="Revenue"></rect>
    <rect data-category="Profit"></rect>
</svg>
```

To click Revenue:

```ts
await page.locator('rect[data-category="Revenue"]').click();
```

Here the SVG child **is** the required target, so locating it is appropriate.

### Mental map

```text
Chart
 ↓
SVG
 ↓
Specific shape
 ↓
Stable attribute
 ↓
Locator
```

---

# 13. Don't Depend on SVG `path` Data

You may see:

```html
<path d="M10 20 L30 40 ..."></path>
```

Avoid:

```ts
page.locator('path[d="M10 20 L30 40 ..."]')
```

The geometry can change when:

- Data changes
- Chart size changes
- The chart library changes
- Rendering changes

Prefer:

```html
<path data-category="Revenue"></path>
```

Then:

```ts
await page.locator('path[data-category="Revenue"]').click();
```

---

# 14. SVG + Shadow DOM

Example:

```text
Page
 └── <custom-search>
       └── Shadow DOM (open)
             └── SVG
```

For open Shadow DOM, Playwright's normal locators can generally pierce the shadow boundary.

Example:

```ts
await page
    .locator("custom-search")
    .getByLabel("Search")
    .click();
```

Mental map:

```text
Open Shadow DOM
       ↓
Normal Playwright locator
       ↓
SVG
```

---

# 14A. Important Shadow DOM Detail

Playwright's locators pierce **open Shadow DOM by default**.

However:

- **XPath locators do not pierce Shadow DOM.**
- **Closed-mode Shadow DOM is not supported by normal Playwright locators.**

So this is an important distinction:

```text
Open Shadow DOM
    ↓
Normal Playwright locators
    ↓
YES
```

But:

```text
Open Shadow DOM
    ↓
XPath
    ↓
NO
```

For SVG inside open Shadow DOM, prefer Playwright's normal locator APIs rather than XPath.

# 15. SVG + iframe

Example:

```text
Page
 └── iframe
       └── SVG
```

Handle the iframe first:

```ts
await page
    .frameLocator("#chartFrame")
    .locator('svg[data-icon="download"]')
    .click();
```

Mental map:

```text
Page
 ↓
iframe
 ↓
SVG
 ↓
Action
```

---

# 16. SVG + iframe + Shadow DOM

Example:

```text
Page
 └── iframe
       └── <chart-component>
             └── Shadow DOM (open)
                   └── SVG
```

Think about the boundaries:

```text
Page
 ↓
iframe → frameLocator()
 ↓
open Shadow DOM → normal locator
 ↓
SVG → normal locator
 ↓
Action
```

Example:

```ts
await page
    .frameLocator("#chartFrame")
    .locator('svg[data-icon="download"]')
    .click();
```

---

# 17. XPath With SVG

You may encounter:

```ts
await page.locator('//svg[@data-icon="search"]').click();
```

This can work.

But if CSS is clearer:

```ts
await page.locator('svg[data-icon="search"]').click();
```

Prefer the locator that is stable, readable and maintainable.

---

# 18. Common SVG Locator Patterns

| Situation | Example |
|---|---|
| SVG has test ID | `getByTestId("search-icon")` |
| Interactive SVG has an accessible button role/name | `getByRole("button", { name: "Search" })` |
| SVG is inside button | `getByRole("button", {name:"Search"})` |
| SVG has stable data attribute | `locator('svg[data-icon="search"]')` |
| SVG has stable class | `locator("svg.search-icon")` |
| Chart shape has data attribute | `locator('rect[data-category="Sales"]')` |
| SVG inside iframe | `frameLocator(...).locator(...)` |
| SVG inside open Shadow DOM | Normal Playwright locator |

---

# 19. Common Mistakes

### Mistake 1 – Always locating `svg`

Bad:

```ts
await page.locator("svg").click();
```

There may be many SVGs.

---

### Mistake 2 – Locating `<path>` unnecessarily

Bad:

```ts
await page.locator("svg path").click();
```

Prefer the button/link/SVG itself when possible.

---

### Mistake 3 – Using coordinates

Avoid:

```ts
await page.mouse.click(450, 300);
```

Coordinates are fragile.

---

### Mistake 4 – Using `path[d]`

Avoid depending on dynamic SVG geometry.

---

### Mistake 5 – Ignoring the actual clickable element

If:

```html
<button>
    <svg></svg>
</button>
```

locate the button if the button represents the action.

---

# 20. Practical Debugging Process

When an SVG cannot be located:

```text
Inspect element
     ↓
Is SVG inside button/link?
     ↓
YES → locate button/link
     ↓
NO
     ↓
Is SVG itself interactive?
     ↓
YES → role / accessible name
     ↓
NO
     ↓
Test ID?
     ↓
Stable data-* attribute?
     ↓
Stable CSS?
     ↓
Is it inside Shadow DOM?
     ↓
Is it inside iframe?
     ↓
Choose simplest stable locator
```

---

# 21. Real Project Example

Imagine a Salesforce-style application:

```html
<button aria-label="Edit Contact">
    <svg class="slds-icon">
        <path></path>
    </svg>
</button>
```

Prefer:

```ts
await page
    .getByRole("button", { name: "Edit Contact" })
    .click();
```

Not:

```ts
await page.locator("svg.slds-icon").click();
```

Why?

The business action is:

```text
Edit Contact
```

The SVG is only the visual representation.

---

# 22. Decision Tree

```text
I see an SVG
     │
     ↓
Is SVG inside button/link?
     │
   YES → Locate button/link
     │
   NO
     ↓
Is SVG itself interactive?
     │
   YES → Role / accessible name, if correctly exposed
     │
   NO
     ↓
Test ID?
     │
   YES → getByTestId()
     │
   NO
     ↓
Stable data-* attribute?
     │
   YES → locator()
     │
   NO
     ↓
Stable CSS/class?
     │
   YES → locator()
     │
   NO
     ↓
Inspect further
```

---

# 23. Interview-Ready Answer

### Q: How do you handle SVG elements in Playwright?

> SVG elements can generally be handled using Playwright's normal locator mechanisms. I first identify whether the SVG is the actual interactive element or just an icon inside a button or link. If it is inside a button, I prefer locating the button using its role and accessible name. If the SVG itself is interactive, I look for an accessible name, test ID, or stable attribute. For SVG charts, I use stable attributes on the required SVG shape rather than dynamic path data or coordinates.

### Follow-up: Do you need a special Playwright API for SVG?

> No. SVG does not require a dedicated Playwright API.

### Follow-up: Would you locate `<path>`?

> Only when the path itself is the required target and there is no better stable locator. Normally I prefer the clickable parent or a stable attribute.

---

# 24. Practical Assignments

## Assignment 1 – Basic SVG

```html
<svg data-testid="search-icon">
    <path></path>
</svg>
```

Write a Playwright locator to click it.

---

## Assignment 2 – SVG Inside Button

```html
<button aria-label="Delete User">
    <svg>
        <path></path>
    </svg>
</button>
```

Click Delete User **without locating `svg` or `path`**.

---

## Assignment 3 – Multiple SVGs

```html
<svg data-icon="edit"></svg>
<svg data-icon="delete"></svg>
<svg data-icon="download"></svg>
```

Click only Download.

Do not use:

```ts
locator("svg")
```

---

## Assignment 4 – SVG Chart

```html
<svg>
    <rect data-category="Sales"></rect>
    <rect data-category="Revenue"></rect>
    <rect data-category="Profit"></rect>
</svg>
```

Click Revenue.

Then answer:

> Why is `data-category="Revenue"` better than `nth(1)`?

---

## Assignment 5 – Real Project Locator Chaining

```html
<div data-testid="user-card">
    <button aria-label="Edit User">
        <svg class="edit-icon"></svg>
    </button>

    <button aria-label="Delete User">
        <svg class="delete-icon"></svg>
    </button>
</div>
```

Automate:

```text
User Card
   ↓
Edit User
   ↓
Click
```

Use locator chaining.

---

## Assignment 6 – Identify the Boundary

Identify the correct Playwright approach:

### A

```text
SVG
```

### B

```text
Button
 └── SVG
```

### C

```text
iframe
 └── SVG
```

### D

```text
Open Shadow DOM
 └── SVG
```

### E

```text
iframe
 └── Open Shadow DOM
       └── SVG
```

Expected thinking:

```text
SVG alone
 → normal locator

Button + SVG
 → locate button

iframe + SVG
 → frameLocator() + locator

Open Shadow DOM + SVG
 → normal locator

iframe + open Shadow DOM + SVG
 → frameLocator() + normal locator
```

---

# 25. Interview Challenge

Answer without looking at the notes:

1. Does SVG require special handling in Playwright?
2. Why is locating a button better than locating its SVG icon?
3. Why is `<path>` usually not the best locator?
4. Why are coordinates a poor SVG automation strategy?
5. When would you use a CSS selector for SVG?
6. How do you handle SVG inside an iframe?
7. How do you handle SVG inside open Shadow DOM?
8. What do you inspect if SVG is visible but Playwright cannot locate it?

---

# 26. Assignment Completion Checklist

```text
☐ I understand what SVG is.

☐ I know SVG does not require a special Playwright API.

☐ I can identify whether SVG itself is interactive.

☐ I can identify SVG inside a button.

☐ I can identify SVG inside a link.

☐ I can use getByTestId() appropriately.

☐ I can use stable data-* attributes.

☐ I understand why path is usually not the best target.

☐ I can handle SVG charts.

☐ I can handle SVG inside Shadow DOM.

☐ I can handle SVG inside iframe.

☐ I can avoid coordinate-based automation.

☐ I can explain SVG handling in an interview.
```

---

# 27. Final Mental Map

```text
                         SVG
                          │
             ┌────────────┴────────────┐
             │                         │
       Inside button/link?       SVG itself interactive?
             │                         │
            YES                       YES
             │                         │
      Locate button/link       Role / accessible name
                                      │
                                      ↓
                              Test ID / stable attribute
                                      │
                                      ↓
                                  CSS locator
```

### Boundary map

```text
Normal DOM
   ↓
Normal locator

Button + SVG
   ↓
Locate button

SVG chart shape
   ↓
Stable shape attribute

Open Shadow DOM + SVG
   ↓
Normal locator

iframe + SVG
   ↓
frameLocator() + locator

iframe + open Shadow DOM + SVG
   ↓
frameLocator() + normal locator
```

# 28. Golden Rules

1. **SVG does not need a special Playwright API.**
2. **If SVG is inside a button, locate the button.**
3. **Don't target `<path>` unless you really need to.**
4. **Prefer accessible names and stable test attributes.**
5. **Avoid coordinates.**
6. **Avoid dynamic SVG path data.**
7. **SVG + iframe → `frameLocator()`.**
8. **SVG + open Shadow DOM → normal Playwright locator.**
9. **XPath does not pierce Shadow DOM.**
10. **Use `getByLabel()` mainly for form controls, not as a generic SVG locator.**
11. **Identify the boundary before choosing the locator.**

### One-line memory trick

> **SVG itself is not the challenge — identifying the correct interactive element and choosing a stable locator is the challenge.**
