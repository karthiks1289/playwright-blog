# Playwright — Pseudo-Element Handling

> **Revision-first note:** Pseudo-elements are **not a regular Playwright automation use case**. Learn enough to recognize them, decide whether they matter, and verify them when the UI requirement specifically depends on them.

---

## 1. What You Must Remember

### Pseudo-element

- Not a normal HTML/DOM element.
- A CSS-generated/special part of an element.
- Common examples: `::before`, `::after`, `::placeholder`.

Example:

```html
<label class="required">First Name</label>
```

```css
.required::before {
    content: "*";
}
```

The `*` can be displayed by CSS even though there is no separate `<span>` in the HTML.

### Pseudo-class

- Represents the **state or condition** of an element.
- Examples: `:hover`, `:focus`, `:checked`, `:disabled`.

### Memory

```text
:   → STATE / CONDITION
::  → SPECIAL CSS-GENERATED PART

:hover    → pseudo-class
::before  → pseudo-element
```

---

# 2. Mental Map

```text
                    CSS
                     │
          ┌──────────┴──────────┐
          │                     │
    Pseudo-class          Pseudo-element
          │                     │
      "STATE"             "SPECIAL PART"
          │                     │
      :hover              ::before
      :focus              ::after
      :checked            ::placeholder
```

For Playwright:

```text
See ::before / ::after
          │
          ▼
Ask: Do I need to INTERACT with it?
          │
       Usually NO
          │
          ▼
Interact with the REAL HTML element


Ask: Do I need to VERIFY the pseudo-element?
          │
         YES
          │
          ▼
locator.evaluate()
          │
          ▼
getComputedStyle(element, "::before/::after")
```

---

# 3. The Most Important Rule

## Do not treat a pseudo-element as a normal DOM element.

If you see:

```css
.delete::before {
    content: "🗑";
}
```

and the UI looks like:

```text
🗑 Delete
```

do **not** think:

```html
<button>
    <span>🗑</span>
    Delete
</button>
```

The icon may be generated entirely by CSS.

### Practical rule

> **Interact with the real HTML element. Inspect the pseudo-element only when the pseudo-element itself is part of the requirement.**

---

# 4. When Do I Actually Need This?

Pseudo-element handling is usually an **edge case**, not something you use in every test.

### Common practical scenarios

| Scenario | What to do |
|---|---|
| Required `*` generated with `::after` | Verify if required by the UI requirement |
| CSS-generated error/success icon | Verify only if the icon itself matters |
| Dropdown arrow | Usually interact with the real dropdown |
| Delete/search icon created with CSS | Usually interact with the real button |
| Tooltip arrow created with CSS | Usually test tooltip behavior, not the CSS implementation |
| CSS decoration | Usually don't automate it |
| Need exact pseudo-element CSS | Use `getComputedStyle()` |

---

# 5. The Browser JavaScript Pattern

The basic JavaScript pattern is:

```js
getComputedStyle(
    document.querySelector("label[for='input-firstname']"),
    "::before"
)
```

Meaning:

```text
document.querySelector()
        ↓
Find REAL HTML element
        ↓
getComputedStyle()
        ↓
Inspect its ::before
```

To get one property:

```js
getComputedStyle(
    document.querySelector("label[for='input-firstname']"),
    "::before"
).getPropertyValue("content")
```

This returns the computed value of the `content` property.

For example, CSS:

```css
label::before {
    content: "*";
}
```

can produce:

```text
"*"
```

---

# 6. Playwright Way

In Playwright, prefer the locator instead of manually calling `document.querySelector()`.

```ts
const label = page.locator("label[for='input-firstname']");

const content = await label.evaluate(element => {
    return getComputedStyle(element, "::before")
        .getPropertyValue("content");
});
```

### Flow

```text
page.locator()
      ↓
Find REAL element
      ↓
evaluate()
      ↓
getComputedStyle()
      ↓
::before / ::after
      ↓
getPropertyValue()
      ↓
Read required CSS property
```

---

# 7. `::before` Example

HTML:

```html
<label class="required">First Name</label>
```

CSS:

```css
.required::before {
    content: "*";
}
```

Playwright:

```ts
const label = page.locator(".required");

const content = await label.evaluate(element => {
    return getComputedStyle(element, "::before")
        .getPropertyValue("content");
});

expect(content).toBe('"*"');
```

---

# 8. `::after` Example

CSS:

```css
.required::after {
    content: "*";
}
```

Playwright:

```ts
const content = await page.locator(".required").evaluate(element => {
    return getComputedStyle(element, "::after")
        .getPropertyValue("content");
});

expect(content).toBe('"*"');
```

### Remember

```text
::before → getComputedStyle(element, "::before")
::after  → getComputedStyle(element, "::after")
```

---

# 9. `.content` vs `.getPropertyValue("content")`

Both can be used.

### Option 1

```ts
const content = await locator.evaluate(element => {
    return getComputedStyle(element, "::before").content;
});
```

### Option 2

```ts
const content = await locator.evaluate(element => {
    return getComputedStyle(element, "::before")
        .getPropertyValue("content");
});
```

Both read the computed `content` property.

### For revision

```text
getComputedStyle(...)
        ↓
CSSStyleDeclaration
        ↓
.content
OR
.getPropertyValue("content")
```

---

# 10. You Can Inspect Other CSS Properties

`getComputedStyle()` is not limited to `content`.

Example:

```css
.required::after {
    content: "*";
    color: red;
    font-size: 18px;
}
```

You can inspect:

```ts
const styles = await page.locator(".required").evaluate(element => {
    const pseudo = getComputedStyle(element, "::after");

    return {
        content: pseudo.content,
        color: pseudo.color,
        fontSize: pseudo.fontSize
    };
});
```

Possible values can look like:

```text
content  → "*"
color    → rgb(...)
fontSize → 18px
```

### Important

Only assert CSS properties that are actually part of the requirement. Avoid unnecessary styling assertions because they can make tests brittle.

---

# 11. `content: ""` Is Not the Same as `content: none`

This is a useful edge case.

### `content: "*"`

```css
::after {
    content: "*";
}
```

Generated content contains `*`.

### `content: ""`

```css
::after {
    content: "";
}
```

The pseudo-element can still be used for visual shapes/decorations.

### `content: none`

```css
::after {
    content: none;
}
```

No generated pseudo-element content is produced.

### Memory

```text
"*"   → generated content
""    → generated empty content can still be styled
none  → no generated content
```

---

# 12. Interaction vs Verification

This is the most important practical distinction.

## Scenario A — Interaction

CSS:

```css
.delete::before {
    content: "🗑";
}
```

UI:

```text
🗑 Delete
```

Requirement:

> Click Delete and delete the record.

Use:

```ts
await page.getByRole("button", { name: "Delete" }).click();
```

Do NOT make the pseudo-element the thing you interact with.

---

## Scenario B — Verification

Requirement:

> Required fields must display `*`.

CSS:

```css
.required::after {
    content: "*";
}
```

Now the generated content itself matters.

Use:

```ts
const content = await page.locator(".required").evaluate(element => {
    return getComputedStyle(element, "::after")
        .getPropertyValue("content");
});

expect(content).toBe('"*"');
```

### Decision rule

```text
Need to interact?
      ↓
Real DOM element

Need to verify pseudo-element?
      ↓
getComputedStyle()
```

---

# 13. Don't Use JavaScript Unnecessarily

Seeing `::before` or `::after` does NOT automatically mean:

> "I must use evaluate()."

First ask:

> Can normal Playwright verify the requirement?

Example:

```html
<input placeholder="Enter username">
```

Even though `::placeholder` is a pseudo-element, to verify the placeholder text you can simply use:

```ts
await expect(page.locator("input"))
    .toHaveAttribute("placeholder", "Enter username");
```

No `evaluate()` is necessary.

### Rule

> **Use the simplest reliable Playwright approach.**

---

# 14. Common Mistakes

### Mistake 1 — Treating `::after` as a normal HTML element

Wrong mental model:

```text
::after = separate <span>
```

Correct:

```text
::after = CSS-generated/special part
```

---

### Mistake 2 — Trying to click the pseudo-element

If an arrow is generated by:

```css
.dropdown::after {
    content: "▼";
}
```

usually click:

```ts
await page.locator(".dropdown").click();
```

not the CSS arrow.

---

### Mistake 3 — Using `getComputedStyle()` for normal DOM properties

If this works:

```ts
await expect(locator).toHaveAttribute(...)
```

or:

```ts
await expect(locator).toHaveText(...)
```

prefer the normal Playwright assertion.

---

### Mistake 4 — Testing implementation instead of behavior

If the application changes from:

```css
.delete::before {
    content: "🗑";
}
```

to:

```html
<span>🗑</span>
```

the Delete functionality should still work.

Don't make a business test depend unnecessarily on how the icon was implemented.

---

# 15. Pseudo-Class vs Pseudo-Element

### Pseudo-class

```css
button:hover
```

Means:

> The button is currently in the `hover` state.

Other examples:

```css
:hover
:focus
:checked
:disabled
```

### Pseudo-element

```css
button::before
```

Means:

> CSS is targeting the special/generated `::before` part of the button.

Other examples:

```css
::before
::after
::placeholder
```

### Fast memory

```text
:   → STATE

::  → SPECIAL PART / GENERATED CONTENT
```

---

# 16. Interview-Ready Answer

### Q: How do you handle pseudo-elements in Playwright?

> Pseudo-elements such as `::before` and `::after` are not normal DOM elements. I normally interact with the real HTML element. If I specifically need to verify the pseudo-element's generated content or CSS properties, I use `locator.evaluate()` with `getComputedStyle()`, for example `getComputedStyle(element, "::after").content`.

---

# 17. What You Actually Need to Remember

```text
1. Pseudo-element
   → Not a normal HTML element
   → CSS-generated/special part

2. Pseudo-class
   → Element state/condition

3. Common pseudo-elements
   → ::before
   → ::after
   → ::placeholder

4. Normal interaction
   → Interact with real DOM element

5. Need to inspect pseudo-element
   → locator.evaluate()
   → getComputedStyle()

6. Need one CSS property
   → .content
   OR
   → .getPropertyValue("content")

7. Don't use evaluate() when normal Playwright
   can already verify the requirement.
```

---

# 18. Final 30-Second Revision

```text
PSEUDO-CLASS
:
→ State / condition
→ :hover
→ :focus
→ :checked


PSEUDO-ELEMENT
::
→ CSS-generated/special part
→ ::before
→ ::after
→ ::placeholder


PLAYWRIGHT
Need interaction?
→ Real HTML element

Need pseudo-element verification?
→ evaluate()
→ getComputedStyle(element, "::before/::after")
→ read required CSS property
```

## ⭐ One sentence

> **Pseudo-class = state. Pseudo-element = CSS-generated/special part. Interact with the real element; use `evaluate() + getComputedStyle()` only when you actually need to inspect the pseudo-element.**
