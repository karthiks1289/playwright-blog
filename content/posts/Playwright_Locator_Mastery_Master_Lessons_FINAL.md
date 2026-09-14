# Playwright Locator Mastery — Master Lessons

> **Training goal:** Do not memorize locator syntax. Learn to look at a UI element, understand what makes it identifiable and stable, choose the best locator, write the Playwright code, and explain the decision like a Senior SDET.

---

# Table of Contents

1. Locator Mindset
2. Day 1 — `getByRole()`
3. Day 2 — `getByLabel()` and `getByPlaceholder()`
4. Day 3 — `getByText()`, `getByAltText()`, `getByTitle()` and Regex
5. Day 4 — `getByTestId()` and `locator()`
6. Day 5 — Chaining and Filtering
7. Day 6 — Difficult and Real-World Locators
8. Day 7 — Advanced Locator Scenarios
9. Day 8 — Locator Mastery and Decision Making
10. Master Locator Decision Tree
11. Senior SDET Rules
12. Final Mental Model

---

# 1. Locator Mindset

## What is a Locator?

A locator tells Playwright **which element on the page we want to interact with or verify**.

Examples:

```ts
page.getByRole('button', { name: 'Save Changes' })
```

```ts
page.getByLabel('First Name')
```

```ts
page.getByPlaceholder('Enter first name')
```

The important skill is not knowing many selector syntaxes.

The important skill is answering:

> **What makes this element uniquely identifiable and stable?**

---

# The Core Locator Thinking Process

Whenever you see an element:

```text
UI Element
    ↓
What is it?
    ↓
What identifies it?
    ↓
Is that identity stable?
    ↓
Is it unique?
    ↓
Choose the best Playwright locator
```

Do not start with:

> "Which CSS selector can I write?"

Start with:

> **"How would I describe this element to another person?"**

---

# The Most Important Mental Model

```text
ROLE = What is it?
NAME = Which one?
```

For example:

```ts
page.getByRole('button', { name: 'Save Changes' })
```

means:

```text
Role → button
Name → Save Changes
```

---

# Locator Quality

A good locator should ideally be:

- meaningful
- stable
- unique
- readable
- maintainable
- independent of unnecessary DOM structure

A locator that happens to work is not automatically a good locator.

---

# 2. Day 1 — `getByRole()`

## Simple Definition

`getByRole()` locates an element using its **accessible role** and, when needed, its **accessible name**.

Common roles include:

- button
- link
- heading
- textbox
- checkbox
- radio
- combobox
- tab
- dialog
- list
- listitem
- row
- cell

---

## Why Use `getByRole()`?

It is one of Playwright's strongest user-facing locator strategies.

It describes the element according to how it is understood by the user/accessibility tree instead of depending on implementation details such as:

- CSS classes
- generated IDs
- DOM position

Example:

```ts
await page.getByRole('button', { name: 'Save Changes' }).click();
```

This communicates exactly what the test is doing.

---

# Basic Examples

## Button

```ts
await page.getByRole('button', {
    name: 'Save Changes'
}).click();
```

## Link

```ts
await page.getByRole('link', {
    name: 'Playwright Official Docs'
}).click();
```

## Heading

```ts
await expect(
    page.getByRole('heading', {
        name: 'Practice Sandbox'
    })
).toBeVisible();
```

## Checkbox

```ts
await page.getByRole('checkbox', {
    name: 'Remember me on this device'
}).check();
```

## Radio

```ts
await page.getByRole('radio', {
    name: 'TypeScript'
}).check();
```

## Textbox

```ts
await page.getByRole('textbox', {
    name: 'Search'
}).fill('Playwright');
```

---

# Accessible Name

The accessible name is the name used to identify an element from the accessibility perspective.

It can come from sources such as:

- visible text
- associated label
- `aria-label`
- `aria-labelledby`

Example:

```html
<button aria-label="Delete Account">
    🗑️
</button>
```

There is no visible "Delete Account" text.

But the accessible identity is:

```text
Role → button
Name → Delete Account
```

So:

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

is an excellent locator.

---

# `exact: true`

Use `exact: true` when you want the locator to match the name exactly.

```ts
page.getByRole('button', {
    name: 'Save Changes',
    exact: true
})
```

This is especially useful when similar names could otherwise match.

---

# Regex with `getByRole()`

Regex is useful when part of an accessible name is dynamic.

Example:

```ts
page.getByRole('button', {
    name: /Save.*/
})
```

Regex is JavaScript regex syntax. It is not special Playwright syntax.

---

# Important `getByRole()` Rule

Do not force `getByRole()` onto every element.

Use the locator that best represents the element's actual identity and is reliable.

For example:

- form purpose → `getByLabel()`
- useful placeholder → `getByPlaceholder()`
- visible text → `getByText()`
- image alt → `getByAltText()`
- title attribute → `getByTitle()`
- semantic control → often `getByRole()`

---

# 3. Day 2 — `getByLabel()` and `getByPlaceholder()`

# `getByLabel()`

## Simple Definition

`getByLabel()` locates a form control using its associated label.

### Easy way to remember

> **Label tells us the purpose of the field.**

Example:

```html
<label>First Name</label>
<input>
```

Playwright:

```ts
await page.getByLabel('First Name').fill('Karthik');
```

---

# What Can `getByLabel()` Locate?

It can be used for form controls such as:

- text inputs
- password inputs
- checkboxes
- radio buttons
- textareas
- select controls

Examples:

```ts
await page.getByLabel('Email Address')
    .fill('test@example.com');
```

```ts
await page.getByLabel('Remember me on this device')
    .check();
```

```ts
await page.getByLabel('TypeScript')
    .check();
```

---

# `getByPlaceholder()`

## Simple Definition

`getByPlaceholder()` locates an input using its placeholder text.

### Easy way to remember

> **Placeholder gives the user a hint or example.**

Example:

```html
<input placeholder="Enter first name">
```

Playwright:

```ts
await page.getByPlaceholder('Enter first name')
    .fill('Karthik');
```

---

# Label vs Placeholder

This distinction is important.

```text
LABEL
→ Purpose / Meaning

PLACEHOLDER
→ Hint / Example
```

Example:

```text
Label:
First Name

Placeholder:
Enter first name
```

Both can identify the same field, but they communicate different information.

---

# Which One Should I Prefer?

When a meaningful label exists, `getByLabel()` is often a strong choice because it describes the field's purpose.

If the placeholder is the useful stable identifier, `getByPlaceholder()` is also a good user-facing locator.

Do not treat this as an absolute ranking.

Always ask:

> Which locator is the most meaningful, unique, and stable for this actual element?

---

# Important Example

Suppose an input has:

```html
<input
    aria-label="First Name"
    placeholder="Enter first name"
>
```

Possible locators:

```ts
page.getByLabel('First Name')
```

or:

```ts
page.getByPlaceholder('Enter first name')
```

If the requirement is based on the field's purpose, `getByLabel()` is usually the clearer choice.

---

# Form Control Mental Model

```text
FORM CONTROL
      ↓
Meaningful label?
      ↓
getByLabel()
      ↓
Otherwise useful placeholder?
      ↓
getByPlaceholder()
```

Again, this is a decision guide, not a rigid ranking.

---

# 4. Day 3 — `getByText()`, `getByAltText()`, `getByTitle()` and Regex

# `getByText()`

## Simple Definition

`getByText()` locates an element using visible text.

Example:

```ts
await expect(
    page.getByText('Practice Sandbox', {
        exact: true
    })
).toBeVisible();
```

---

# Exact Text

```ts
page.getByText('Locator', {
    exact: true
})
```

Exact matching is useful when similar text appears in multiple places.

For example, a page may contain:

```text
Locator
Locator Playground
Locator Strategy
```

If you need the exact `Locator` text:

```ts
page.getByText('Locator', {
    exact: true
})
```

---

# Partial Text

Without `exact: true`, matching can be more flexible.

Use partial text carefully when multiple elements may contain similar text.

---

# Regex

Regex is useful when part of the text changes.

Example:

```ts
await expect(
    page.getByText(/Live counter: \d+/)
).toBeVisible();
```

This can match:

```text
Live counter: 247
Live counter: 248
Live counter: 249
```

---

# Regex Basics

```text
/Live counter: \d+/
```

Meaning:

```text
/ ... /  → JavaScript regex delimiters
\d       → one digit
+        → one or more
```

So:

```text
\d+
```

means:

> One or more digits.

Regex is especially useful for predictable dynamic text.

---

# `getByAltText()`

## Simple Definition

`getByAltText()` locates an image using its `alt` text.

Example:

```html
<img alt="Playwright logo placeholder">
```

Locator:

```ts
page.getByAltText('Playwright logo placeholder')
```

Use it when the image's alt text is the appropriate stable identity.

Do not force `getByAltText()` if another locator better represents the intended interaction.

---

# `getByTitle()`

## Simple Definition

`getByTitle()` locates an element using its `title` attribute.

Example:

```html
<button title="Copy">
    ...
</button>
```

Locator:

```ts
page.getByTitle('Copy', {
    exact: true
})
```

---

# `getByText()` vs `getByRole()`

If an element is a semantic control with a clear role and accessible name, `getByRole()` is often preferable.

Example:

```html
<button>Save Changes</button>
```

Possible:

```ts
page.getByText('Save Changes')
```

Better semantic choice:

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

Why?

Because the second locator communicates:

> This is a button called Save Changes.

---

# Locator Choice

```text
Meaningful semantic control
→ often getByRole()

Visible text
→ getByText()

Image alt text
→ getByAltText()

title attribute
→ getByTitle()
```

---

# 5. Day 4 — `getByTestId()` and `locator()`

# `getByTestId()`

## Simple Definition

`getByTestId()` locates an element using a dedicated test ID.

Example:

```html
<input data-testid="input-month">
```

Playwright:

```ts
await page.getByTestId('input-month').click();
```

Another example:

```html
<input data-testid="input-week">
```

```ts
await expect(
    page.getByTestId('input-week')
).toBeVisible();
```

---

# Why Use Test IDs?

A test ID is useful when:

- no good user-facing locator exists
- the UI is difficult to identify semantically
- the application provides a stable testing attribute
- the team intentionally creates a testing contract

---

# `id` vs `data-testid`

Do not confuse these:

```html
id="username"
```

with:

```html
data-testid="username-input"
```

`getByTestId()` is intended for the configured test ID attribute, not simply any normal HTML `id`.

---

# Test ID Configuration

Playwright can be configured to use a different attribute as its test ID attribute.

The important idea is:

> `getByTestId()` uses the application's agreed testing attribute.

---

# Should We Use Test IDs Everywhere?

No.

If a clear and stable user-facing locator already identifies the element, a test ID may be unnecessary.

Example:

```html
<button
    data-testid="save-button"
    aria-label="Save Changes">
    Save Changes
</button>
```

Both can work:

```ts
page.getByTestId('save-button')
```

and:

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

When the semantic locator is clear and unique, the semantic locator is often the better choice.

---

# `locator()`

## Simple Definition

`locator()` is the general Playwright locator mechanism.

It is useful when user-facing locators or test IDs are not sufficient.

Examples:

```ts
page.locator("[type='email']")
```

```ts
page.locator("input[data-testid='username-input']")
```

---

# Basic CSS Selectors

## ID

```ts
page.locator('#username')
```

## Class

```ts
page.locator('.btn-primary')
```

## Attribute

```ts
page.locator("[type='email']")
```

## Tag + Attribute

```ts
page.locator("input[type='email']")
```

---

# CSS Is Not Automatically Bad

A common mistake is to think:

> CSS = bad.

That is not correct.

The real question is:

> **Is this selector reliable, stable, and appropriate for this element?**

CSS is useful when it is the best available strategy.

The problem is using fragile implementation details unnecessarily.

---

# Locator Strategy

A useful general guideline:

```text
User-facing locator
        ↓
Stable test ID
        ↓
DOM/CSS locator
```

This is a guideline, not an absolute ranking.

Stability and uniqueness always matter.

---

# 6. Day 5 — Locator Chaining and Filtering

# Why Chaining?

Chaining lets us locate an element **inside another element or component**.

Example:

```ts
row.getByRole('button', {
    name: 'Edit'
})
```

Meaning:

> Find the Edit button inside this row.

---

# `.filter({ hasText })`

Use `hasText` to identify the correct component based on text.

Example:

```ts
const studentRow = page
    .getByRole('row')
    .filter({
        hasText: 'Sneha Iyer'
    });

await studentRow
    .getByRole('button', {
        name: 'Edit'
    })
    .click();
```

---

# How to Think About It

```text
All rows
   ↓
Find the row containing Sneha Iyer
   ↓
Inside that row
   ↓
Find Edit
```

This is far better than selecting an Edit button only by its global position.

---

# `.filter({ has })`

`has` allows us to identify a component that contains another locator.

Example:

```ts
const studentRow = page
    .getByRole('row')
    .filter({
        has: page.getByRole('cell', {
            name: 'Sneha Iyer',
            exact: true
        })
    });
```

Then:

```ts
await studentRow
    .getByRole('button', {
        name: 'Edit'
    })
    .click();
```

---

# Filtering vs Chaining

This distinction is extremely important.

### FILTER

> **Which one?**

### CHAIN

> **What inside it?**

```text
FILTER → WHICH COMPONENT?
CHAIN  → WHAT ELEMENT INSIDE IT?
```

---

# Repeated Component Pattern

For:

- tables
- cards
- lists
- menus
- grids
- repeated forms

use:

```text
Find the correct component
        ↓
Find the target element inside it
```

This is one of the most important real-project locator patterns.

---

# `first()`

Use `first()` when the requirement genuinely means the first matching element.

```ts
page.getByRole('button', {
    name: 'Learn more'
}).first()
```

---

# `last()`

Use `last()` when the requirement genuinely means the last matching element.

```ts
page.getByRole('button', {
    name: 'Delete'
}).last()
```

---

# `nth()`

`nth()` is zero-based.

```text
1st → nth(0)
2nd → nth(1)
3rd → nth(2)
```

Example:

```ts
page.getByRole('listitem').nth(2)
```

means:

> Select the third list item.

---

# When `nth()` Is Correct

Requirement:

> Click the third item.

Then:

```ts
page.getByRole('listitem').nth(2)
```

is appropriate.

---

# When `nth()` Is Wrong

Requirement:

> Delete Rahul Verma.

Do not do:

```ts
page.getByRole('button', {
    name: 'Delete'
}).nth(2)
```

because the position of Rahul can change.

Instead:

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
    .getByRole('button', {
        name: 'Delete'
    })
```

---

# Core Rule

> **Use meaning to identify elements. Use position only when position is genuinely part of the requirement.**

---

# 7. Day 6 — Difficult and Real-World Locators

# Duplicate Elements

Suppose the page contains:

```text
Edit
Edit
Edit
```

A global locator:

```ts
page.getByRole('button', {
    name: 'Edit'
})
```

may match multiple elements.

Do not automatically solve this with `nth()`.

First ask:

> Which component does this Edit button belong to?

---

# Component-First Approach

Example:

```ts
const row = page
    .getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    });

await row
    .getByRole('button', {
        name: 'Edit'
    })
    .click();
```

This identifies the correct row first.

---

# Dynamic IDs

Avoid selectors based on IDs that are generated dynamically.

Example:

```html
id="input-83921"
```

If the number changes, this is fragile:

```ts
page.locator('#input-83921')
```

Prefer a stable identity such as:

```ts
page.getByLabel('First Name')
```

or:

```ts
page.getByPlaceholder('Enter first name')
```

depending on the actual UI.

---

# Dynamic CSS Classes

Avoid generated classes such as:

```html
class="dynamic-83921"
```

if the value can change.

Prefer a stable semantic locator or stable testing attribute.

---

# Dynamic Text

Example:

```text
Live counter: 247
```

then later:

```text
Live counter: 248
```

Do not hard-code the current number.

Use:

```ts
page.getByText(/Live counter: \d+/)
```

---

# Fragile DOM-Dependent Locator

Example:

```ts
page.locator(
    'div:nth-child(3) > div:nth-child(2) > button'
)
```

This depends heavily on DOM structure and position.

If the UI changes, the locator may break or target another element.

If a semantic locator exists, prefer it:

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

---

# Strict Mode

Strict mode is useful feedback from Playwright.

If a locator intended for a single-element action matches multiple elements, Playwright can report a strict mode violation.

This means:

> **The locator is not specific enough for the intended action.**

Strict mode is not the problem.

The ambiguous locator is the problem.

---

# How to Respond to Strict Mode

Do not immediately:

```ts
.nth(0)
```

Instead:

```text
Multiple matches
      ↓
Why are there multiple matches?
      ↓
What component identifies the correct one?
      ↓
Filter to that component
      ↓
Locate the target inside it
```

---

# Strict Mode Example

Bad:

```ts
page.getByRole('button', {
    name: 'Delete'
}).nth(2)
```

when the requirement is:

> Delete Rahul Verma.

Better:

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
    .getByRole('button', {
        name: 'Delete'
    })
```

---

# Senior SDET Principle

> **Make the locator identify the correct element, rather than making the test accidentally click the correct element.**

---

# 8. Day 7 — Advanced Locator Scenarios

# Iframes

## Simple Definition

An iframe contains a separate document.

Elements inside the iframe need to be accessed through the iframe context.

Use:

```ts
page.frameLocator('#demoFrame')
```

Then use normal Playwright locators inside that frame.

---

# Example

```ts
const frameLocate: FrameLocator =
    page.frameLocator('#demoFrame');

await frameLocate
    .getByRole('textbox', {
        name: 'Username inside iframe',
        exact: true
    })
    .fill('I am inside a frame');

await frameLocate
    .getByRole('button', {
        name: 'Submit inside iframe',
        exact: true
    })
    .click();
```

---

# Iframe Mental Model

```text
Main Page
    ↓
iframe
    ↓
frameLocator()
    ↓
Normal Playwright locators
```

The important idea is:

> **First establish the frame context, then locate the element inside the frame.**

---

# Open Shadow DOM

For an **open Shadow DOM**, Playwright's locators can generally pierce through the shadow boundary automatically.

You normally do not need special Shadow DOM locator syntax.

Mental model:

```text
Open Shadow DOM
       ↓
Normal Playwright locator
```

This is different from an iframe.

An iframe represents a separate document, while an open Shadow DOM is part of the page's DOM structure.

---

# SVG and Icon Buttons

Suppose:

```html
<button aria-label="Delete Account">
    <svg>...</svg>
</button>
```

Prefer:

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

rather than:

```ts
page.locator('svg')
```

Why?

The user interacts with the **button**, not the implementation detail of the SVG.

---

# ARIA Attributes

Important ARIA attributes include:

- `aria-label`
- `aria-labelledby`
- `aria-describedby`
- `aria-expanded`
- `aria-checked`
- `aria-selected`

ARIA can influence how an element is represented in the accessibility tree and therefore its accessible identity.

---

# Example: `aria-label`

```html
<button aria-label="Delete Account">
    🗑️
</button>
```

Use:

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

The visible icon does not prevent the button from having a meaningful accessible name.

---

# Custom Controls

The HTML tag does not always tell the complete user-facing story.

Example:

```html
<div
    role="button"
    aria-label="Open Settings">
    ⚙️
</div>
```

Use:

```ts
page.getByRole('button', {
    name: 'Open Settings'
})
```

Why?

Because the element explicitly declares:

```text
Role → button
Accessible Name → Open Settings
```

---

# Locator Debugging

When a locator fails, do not immediately:

- increase the timeout
- switch to XPath
- add `nth()`
- create a random CSS selector

Instead investigate.

---

# Debugging Checklist

```text
1. Does the element exist?
2. What is its actual role?
3. What is its accessible name?
4. Is it inside an iframe?
5. Is it inside a Shadow DOM?
6. Are there multiple matching elements?
7. Is the locator stable?
8. Is the locator targeting the intended component?
```

---

# Useful Playwright Debugging Tools

Debug mode:

```bash
npx playwright test --debug
```

Code generation:

```bash
npx playwright codegen <URL>
```

Codegen is useful for discovering possible locators.

But:

> **Do not blindly accept generated locators.**

Review them for:

- readability
- stability
- uniqueness
- semantic quality
- maintainability

---

# 9. Day 8 — Locator Mastery and Decision Making

Day 8 is about combining all previous concepts.

There is no new locator type to memorize.

The goal is:

> **Look at a UI element → understand it → choose the best locator → explain why.**

---

# The Senior SDET Question

Instead of asking:

> "Which locator syntax can I use?"

Ask:

> **"What makes this element uniquely identifiable and stable?"**

---

# Example: Many Possible Locators

Suppose:

```html
<button
    class="action-btn dynamic-82731"
    id="button-938472"
    data-testid="approve-request"
    aria-label="Approve Request">
    ✓
</button>
```

Possible locators include:

```ts
page.locator('.action-btn')
```

```ts
page.locator('#button-938472')
```

```ts
page.getByTestId('approve-request')
```

```ts
page.getByRole('button', {
    name: 'Approve Request'
})
```

The semantic locator is often the best choice here:

```ts
page.getByRole('button', {
    name: 'Approve Request'
})
```

because it identifies the actual user interaction.

The test ID is also a good stable fallback.

---

# Why Semantic Locators Are Powerful

Compare:

```ts
page.locator('.action-btn')
```

with:

```ts
page.getByRole('button', {
    name: 'Approve Request'
})
```

The second communicates:

> Find the button that the user understands as "Approve Request."

The first communicates:

> Find something having this CSS class.

The semantic locator usually carries more useful intent.

---

# Real-World Repeated Components

Suppose a table contains:

```text
Rahul Verma
    Edit
    Delete

Sneha Iyer
    Edit
    Delete

Priya Sharma
    Edit
    Delete
```

Requirement:

> Delete Rahul Verma.

Best thinking:

```text
Find Rahul's row
       ↓
Find Delete inside Rahul's row
```

Example:

```ts
await page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
    .getByRole('button', {
        name: 'Delete'
    })
    .click();
```

---

# Why Position Is Dangerous

This is risky:

```ts
await page.getByRole('button', {
    name: 'Delete'
}).nth(2).click();
```

It assumes the target is always the third Delete button.

If somebody:

- adds a row
- removes a row
- sorts the table
- filters the table
- changes the data

the position can change.

The test might then interact with the wrong user.

---

# When Position Is Correct

Requirement:

> Click the third question.

Use:

```ts
await page.getByRole('listitem').nth(2).click();
```

Because **third** is explicitly part of the requirement.

---

# Identification vs Position

This distinction should become automatic.

### Identification

```text
Delete Rahul Verma
```

Use:

```text
Meaning / identity
```

### Position

```text
Click the third item
```

Use:

```text
nth(2)
```

### Golden Rule

> **Use meaning to identify elements. Use position only when position is genuinely the requirement.**

---

# 10. Master Locator Decision Tree

```text
                         UI ELEMENT
                              ↓
                 What makes it identifiable?
                              ↓
               Is there a meaningful user-facing
                         identity?
                              ↓
        ┌─────────────────────┴─────────────────────┐
        ↓                                           ↓
   Semantic control                            Other element
        ↓                                           ↓
   getByRole()                               getByText()
   getByLabel()                              getByAltText()
   getByPlaceholder()                        getByTitle()
        ↓
   No suitable user-facing locator?
        ↓
   Stable test ID?
        ↓
   getByTestId()
        ↓
   Still no?
        ↓
   locator() / CSS / DOM strategy
        ↓
   Multiple matches?
        ↓
   Identify correct component
        ↓
   Locate target inside component
        ↓
   Is position actually the requirement?
        ↓
     YES                 NO
      ↓                   ↓
first/last/nth       Use identity/component
```

---

# 11. Senior SDET Rules

## Rule 1 — Think About Identity

> **Don't ask which locator syntax you know. Ask what uniquely identifies the element.**

---

## Rule 2 — Prefer Meaning

If a meaningful semantic locator exists, prefer it over unnecessary implementation details.

---

## Rule 3 — Stability Matters

Avoid selectors based on values that change frequently.

Examples:

```text
dynamic IDs
dynamic classes
DOM position
```

---

## Rule 4 — Uniqueness Matters

A locator should identify the intended element.

If multiple elements match, improve the locator instead of blindly selecting one by position.

---

## Rule 5 — Component First

For repeated UI:

```text
Correct component
      ↓
Target inside component
```

---

## Rule 6 — Position Is Not a Shortcut

Do not use:

```ts
nth()
```

just because there are multiple matches.

Use it when the requirement itself is positional.

---

## Rule 7 — Iframe Is a Context Boundary

For iframe content:

```ts
page.frameLocator(...)
```

then locate the element inside the frame.

---

## Rule 8 — Open Shadow DOM Is Different

For open Shadow DOM, normal Playwright locators can generally pierce the shadow boundary.

---

## Rule 9 — Strict Mode Is Helpful

A strict mode violation can be useful feedback.

It tells you:

> **Your locator is not uniquely identifying the intended element.**

---

## Rule 10 — Debug the Cause

When a locator fails:

> **Understand why before changing it.**

---

## Rule 11 — Codegen Is a Helper, Not the Decision Maker

Codegen can suggest locators.

The engineer must still decide whether the generated locator is:

- stable
- readable
- unique
- maintainable
- appropriate

---

## Rule 12 — A Locator That Works Is Not Automatically a Good Locator

Ask:

```text
Does it work?
        +
Is it stable?
        +
Is it meaningful?
        +
Is it unique?
        +
Will another engineer understand it?
```

---

# 12. Final Mental Model

```text
                  PLAYWRIGHT LOCATOR MASTERY

                           UI ELEMENT
                                ↓
                    Understand the element
                                ↓
                 What makes it identifiable?
                                ↓
                 ┌──────────────┴──────────────┐
                 ↓                             ↓
        User-facing identity             Other identity
                 ↓                             ↓
        getByRole()                       getByTestId()
        getByLabel()                      locator()
        getByText()                       CSS
        getByPlaceholder()
        getByAltText()
        getByTitle()
                 ↓
             Is it unique?
              ↓        ↓
             YES       NO
              ↓         ↓
          Interact   Identify the
                     correct component
                           ↓
                     Filter / narrow
                           ↓
                    Chain inside it
                           ↓
                 Is position required?
                     ↓           ↓
                    YES          NO
                     ↓            ↓
                first/last/nth   Identity
```

---

# The Most Important Patterns to Remember

## Pattern 1 — Semantic Element

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

---

## Pattern 2 — Form Field by Purpose

```ts
page.getByLabel('First Name')
```

---

## Pattern 3 — Form Field by Hint

```ts
page.getByPlaceholder('Enter first name')
```

---

## Pattern 4 — Visible Text

```ts
page.getByText('Locator', {
    exact: true
})
```

---

## Pattern 5 — Dynamic Text

```ts
page.getByText(/Live counter: \d+/)
```

---

## Pattern 6 — Test ID

```ts
page.getByTestId('input-month')
```

---

## Pattern 7 — CSS Fallback

```ts
page.locator("[type='email']")
```

---

## Pattern 8 — Component Filtering

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
```

---

## Pattern 9 — Target Inside Component

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
    .getByRole('button', {
        name: 'Delete'
    })
```

---

## Pattern 10 — Positional Requirement

```ts
page.getByRole('listitem').nth(2)
```

---

## Pattern 11 — Iframe

```ts
page.frameLocator('#demoFrame')
    .getByRole('button', {
        name: 'Submit'
    })
```

---

## Pattern 12 — Icon Button

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

---

# Final One-Line Rule

> **Find the element by what it means, narrow to the correct component when necessary, and use position only when position is actually the requirement.**

---

# Locator Mastery in One Sentence

> **A Senior SDET does not choose a locator because it works; they choose the locator that most reliably expresses why that exact element is the one the test intends to interact with.**
