# Playwright Locator Mastery — Master Lessons

> **How these notes are written:** Every concept starts with **simple understanding first**, followed by the technical explanation, practical examples, important rules, and Senior SDET/interview thinking.
>
> **Goal:** Do not memorize locator syntax. Learn to look at a UI element, understand what makes it identifiable and stable, choose the best locator, write the Playwright code, and explain why.

---

# 1. Locator Mindset

## Simple Understanding

A **locator is the way we tell Playwright which element we want to work with**.

For example, if we see a button called **Save Changes**, we need a reliable way to tell Playwright:

> "I want that Save Changes button."

Example:

```ts
page.getByRole('button', { name: 'Save Changes' })
```

## Technical Understanding

A Playwright locator represents a way to find an element or set of elements in the page.

Locators can be based on:

- accessible role
- accessible name
- label
- placeholder
- visible text
- alt text
- title
- test ID
- DOM/CSS selectors
- component context

## The Real Skill

The important skill is **not memorizing many locator syntaxes**.

The important question is:

> **What makes this element uniquely identifiable and stable?**

Think:

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
Choose the best locator
```

## Most Important Mental Model

```text
ROLE = What is it?
NAME = Which one?
```

Example:

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

means:

```text
Role → button
Name → Save Changes
```

## What Makes a Good Locator?

A good locator should ideally be:

- meaningful
- stable
- unique
- readable
- maintainable
- independent of unnecessary DOM structure

A locator that works today is **not automatically a good locator**.

---

# 2. Day 1 — `getByRole()`

# What is `getByRole()`?

## Simple Understanding

`getByRole()` lets us find an element by **what kind of UI element it is** and, usually, **which one we want**.

Think:

> **What is it? → Role**  
> **Which one? → Name**

For example:

```ts
page.getByRole('button', { name: 'Save Changes' })
```

means:

> Find the button whose accessible name is "Save Changes."

## Technical Understanding

`getByRole()` uses the element's **accessible role** and optionally its **accessible name**.

Common roles include:

- `button`
- `link`
- `heading`
- `textbox`
- `checkbox`
- `radio`
- `combobox`
- `tab`
- `dialog`
- `list`
- `listitem`
- `row`
- `cell`

---

## Why Do We Use It?

### Simple Understanding

We use it because it describes the element in a way that is close to how a user understands the UI.

Instead of saying:

> "Find the third `div` having this CSS class..."

we can say:

> "Find the Save Changes button."

That makes the test easier to understand and usually less dependent on implementation details.

---

## Examples

### Button

```ts
await page.getByRole('button', {
    name: 'Save Changes'
}).click();
```

### Link

```ts
await page.getByRole('link', {
    name: 'Playwright Official Docs'
}).click();
```

### Heading

```ts
await expect(
    page.getByRole('heading', {
        name: 'Practice Sandbox'
    })
).toBeVisible();
```

### Checkbox

```ts
await page.getByRole('checkbox', {
    name: 'Remember me on this device'
}).check();
```

### Radio Button

```ts
await page.getByRole('radio', {
    name: 'TypeScript'
}).check();
```

### Textbox

```ts
await page.getByRole('textbox', {
    name: 'Search'
}).fill('Playwright');
```

---

# Accessible Name

## Simple Understanding

The **accessible name is the name that Playwright/accessibility technology uses to identify an element**.

It is not always the same as visible text.

For example:

```html
<button aria-label="Delete Account">
    🗑️
</button>
```

The user sees an icon, but the button has an accessible name:

```text
Delete Account
```

So we can use:

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

## Technical Understanding

An accessible name can be provided or computed from sources such as:

- visible text
- associated labels
- `aria-label`
- `aria-labelledby`

This is why understanding accessibility information is important when designing Playwright locators.

---

# `exact: true`

## Simple Understanding

Sometimes several elements have similar names.

`exact: true` tells Playwright:

> "Match this name exactly."

Example:

```ts
page.getByRole('button', {
    name: 'Save Changes',
    exact: true
})
```

## Why Use It?

Suppose the page contains:

```text
Save Changes
Save Changes and Continue
```

Exact matching helps identify only:

```text
Save Changes
```

---

# Regex with `getByRole()`

## Simple Understanding

Regex is useful when **part of the name changes but part remains predictable**.

Example:

```ts
page.getByRole('button', {
    name: /Save.*/
})
```

## Technical Understanding

Regex here is standard JavaScript regular expression syntax.

Regex is not a special Playwright language.

---

# Important Rule

## Simple Understanding

Do not force `getByRole()` on every element.

Ask:

> **Which locator best represents this element?**

Examples:

```text
Semantic control → often getByRole()
Form purpose     → getByLabel()
Placeholder      → getByPlaceholder()
Visible text     → getByText()
Image alt        → getByAltText()
Title attribute  → getByTitle()
```

---

# 3. Day 2 — `getByLabel()` and `getByPlaceholder()`

# `getByLabel()`

## Simple Understanding

`getByLabel()` finds a form control using its **label**.

The easiest way to remember it:

> **Label tells us the purpose of the field.**

Example:

```text
First Name
[________________]
```

The label tells us:

> "This field is for First Name."

So:

```ts
await page.getByLabel('First Name').fill('Karthik');
```

## Technical Understanding

`getByLabel()` finds form controls associated with a label.

It can be used with controls such as:

- text inputs
- password inputs
- checkboxes
- radio buttons
- textareas
- select controls

---

# Examples

### Text Input

```ts
await page.getByLabel('Email Address')
    .fill('test@example.com');
```

### Checkbox

```ts
await page.getByLabel(
    'Remember me on this device'
).check();
```

### Radio

```ts
await page.getByLabel('TypeScript').check();
```

---

# `getByPlaceholder()`

## Simple Understanding

`getByPlaceholder()` finds an input using its **placeholder text**.

The easiest way to remember it:

> **Placeholder gives the user a hint or example.**

Example:

```text
Enter first name
[________________]
```

The placeholder is telling the user what they can enter.

Playwright:

```ts
await page.getByPlaceholder(
    'Enter first name'
).fill('Karthik');
```

---

# Label vs Placeholder

This is an important distinction.

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

## Simple Decision

If the field has a meaningful label:

```ts
page.getByLabel('First Name')
```

is often a very clear choice.

If a useful placeholder is the appropriate stable identity:

```ts
page.getByPlaceholder('Enter first name')
```

is also a good choice.

Do not treat this as a rigid ranking.

The final choice should consider:

- meaning
- uniqueness
- stability
- actual UI
- maintainability

---

# Example: Both Label and Placeholder Exist

Suppose:

```html
<input
    aria-label="First Name"
    placeholder="Enter first name">
```

Possible locators:

```ts
page.getByLabel('First Name')
```

```ts
page.getByPlaceholder('Enter first name')
```

If the purpose of the field is the important identity, `getByLabel()` is usually clearer.

---

# Form Control Mental Model

```text
FORM CONTROL
      ↓
What tells me the purpose?
      ↓
Meaningful label → getByLabel()
      ↓
If placeholder is the useful identifier
      ↓
getByPlaceholder()
```

---

# 4. Day 3 — `getByText()`, `getByAltText()`, `getByTitle()` and Regex

# `getByText()`

## Simple Understanding

`getByText()` finds an element using **text that appears on the page**.

Example:

```ts
await expect(
    page.getByText('Practice Sandbox', {
        exact: true
    })
).toBeVisible();
```

Think:

> "I can see this text on the page, so I can use that text to find it."

---

# Exact Text

## Simple Understanding

If several pieces of text are similar, `exact: true` tells Playwright:

> "I want exactly this text."

Example:

```ts
page.getByText('Locator', {
    exact: true
})
```

A page could contain:

```text
Locator
Locator Playground
Locator Strategy
```

The exact locator targets:

```text
Locator
```

---

# Partial Text

## Simple Understanding

Without `exact: true`, text matching can be more flexible.

This can be useful when only part of the text is stable.

But be careful:

> Flexible matching can also produce multiple matches.

So always consider uniqueness.

---

# Regex with `getByText()`

## Simple Understanding

Use regex when the text contains a **dynamic part**.

Example:

```text
Live counter: 247
```

Later:

```text
Live counter: 248
```

We should not hard-code `247`.

Instead:

```ts
await expect(
    page.getByText(/Live counter: \d+/)
).toBeVisible();
```

## Technical Understanding

In:

```text
/Live counter: \d+/
```

- `/.../` → JavaScript regex delimiters
- `\d` → a digit
- `+` → one or more

So:

```text
\d+
```

means:

> One or more digits.

This can match:

```text
Live counter: 1
Live counter: 25
Live counter: 247
Live counter: 9999
```

---

# `getByAltText()`

## Simple Understanding

`getByAltText()` finds an image using the image's **alt text**.

Example:

```html
<img alt="Playwright logo placeholder">
```

Use:

```ts
page.getByAltText(
    'Playwright logo placeholder'
)
```

## Why Is Alt Text Important?

Alt text describes an image, especially for accessibility.

So if the image's alt text is the appropriate identity, `getByAltText()` is useful.

---

# `getByTitle()`

## Simple Understanding

`getByTitle()` finds an element using its **title attribute**.

Example:

```html
<button title="Copy">
    ...
</button>
```

Use:

```ts
page.getByTitle('Copy', {
    exact: true
})
```

---

# `getByText()` vs `getByRole()`

## Simple Understanding

If you see text on a button, you might think:

```ts
getByText()
```

But ask:

> "What is the element?"

If it is a button with a clear accessible name, this is usually more meaningful:

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

rather than:

```ts
page.getByText('Save Changes')
```

## Senior SDET Thinking

The goal is not simply to find the text.

The goal is to identify the **actual UI control** the test intends to use.

---

# 5. Day 4 — `getByTestId()` and `locator()`

# `getByTestId()`

## Simple Understanding

`getByTestId()` finds an element using a **special attribute provided for testing**.

Example:

```html
<input data-testid="input-month">
```

Playwright:

```ts
await page.getByTestId('input-month').click();
```

Think:

> "The application has given this element a stable test identity."

---

# Why Do We Use Test IDs?

## Simple Understanding

Sometimes a UI element does not have a good user-facing identity.

In that situation, a stable test ID gives automation a reliable way to identify it.

Good situations include:

- no suitable role/name
- difficult custom UI
- stable testing contract provided by developers

---

# `id` vs `data-testid`

These are different:

```html
id="username"
```

and:

```html
data-testid="username-input"
```

`getByTestId()` is for the configured test ID attribute.

It does not simply mean:

> "Find any HTML id."

---

# Should We Use Test IDs Everywhere?

## Simple Understanding

No.

If a strong user-facing locator already identifies the element, there may be no need for a test ID.

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

If the semantic locator is clear, it is often preferable.

---

# `locator()`

## Simple Understanding

`locator()` is the **general-purpose locator mechanism**.

When the more user-facing locator methods do not give us a good solution, `locator()` lets us target the DOM directly.

Example:

```ts
page.locator("[type='email']")
```

---

# CSS Selector Basics

### ID

```ts
page.locator('#username')
```

### Class

```ts
page.locator('.btn-primary')
```

### Attribute

```ts
page.locator("[type='email']")
```

### Tag + Attribute

```ts
page.locator("input[type='email']")
```

---

# CSS Is Not Bad

## Simple Understanding

Do not think:

> "CSS is always bad."

That is not correct.

The correct thinking is:

> **"Is this CSS selector stable, unique, and appropriate?"**

CSS becomes a problem when we use fragile implementation details unnecessarily.

Example of potentially fragile DOM dependency:

```ts
page.locator(
    'div:nth-child(3) > div:nth-child(2) > button'
)
```

If the DOM changes, this can break.

---

# Locator Strategy

A useful general guideline is:

```text
User-facing locator
        ↓
Stable test ID
        ↓
DOM/CSS locator
```

But this is **not an absolute ranking**.

The real priorities are:

```text
Meaning
+
Stability
+
Uniqueness
+
Maintainability
```

---

# 6. Day 5 — Locator Chaining and Filtering

# Why Do We Need Chaining?

## Simple Understanding

Sometimes there are many similar components on the page.

We first need to find the **correct component**, then find the element **inside that component**.

Example:

```text
Rahul Verma
    Edit
    Delete

Sneha Iyer
    Edit
    Delete
```

If we need Sneha's Edit button, we should not simply find the first Edit button.

We should:

```text
Find Sneha's row
      ↓
Find Edit inside that row
```

---

# `.filter({ hasText })`

## Simple Understanding

`hasText` helps us answer:

> **"Which component contains this text?"**

Example:

```ts
const studentRow = page
    .getByRole('row')
    .filter({
        hasText: 'Sneha Iyer'
    });
```

Now:

```ts
await studentRow
    .getByRole('button', {
        name: 'Edit'
    })
    .click();
```

---

# What Is Happening?

```text
All rows
   ↓
Find row containing Sneha Iyer
   ↓
Correct row
   ↓
Find Edit inside it
```

This is a powerful real-project pattern.

---

# `.filter({ has })`

## Simple Understanding

`has` lets us say:

> "Find the parent/component that contains this other element."

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

This is one of the most important Day 5 concepts.

## FILTER

> **Which one?**

## CHAIN

> **What inside it?**

Mental shortcut:

```text
FILTER → WHICH COMPONENT?
CHAIN  → WHAT ELEMENT INSIDE IT?
```

---

# `first()`

## Simple Understanding

Use `first()` when the requirement actually says:

> "Use the first matching element."

Example:

```ts
page.getByRole('button', {
    name: 'Learn more'
}).first()
```

---

# `last()`

## Simple Understanding

Use `last()` when the requirement actually says:

> "Use the last matching element."

Example:

```ts
page.getByRole('button', {
    name: 'Delete'
}).last()
```

---

# `nth()`

## Simple Understanding

Use `nth()` when **position itself matters**.

Important:

> `nth()` is zero-based.

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

> Click the third question.

Then:

```ts
page.getByRole('listitem').nth(2)
```

is appropriate.

---

# When `nth()` Is Wrong

Requirement:

> Delete Rahul Verma.

Do not use:

```ts
page.getByRole('button', {
    name: 'Delete'
}).nth(2)
```

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

# Day 5 Golden Rule

> **Use meaning to identify elements. Use position only when position is genuinely part of the requirement.**

---

# 7. Day 6 — Difficult and Real-World Locators

# Duplicate Elements

## Simple Understanding

If Playwright finds multiple matching elements, don't immediately use `nth()`.

First ask:

> **"Which component does the element I need belong to?"**

Example:

```text
Edit
Edit
Edit
```

Instead of:

```ts
page.getByRole('button', {
    name: 'Edit'
}).nth(1)
```

identify the correct row/card/component first.

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

---

# Dynamic IDs

## Simple Understanding

If an ID changes every time the page is loaded, don't depend on it.

Example:

```html
id="input-83921"
```

If the number can change:

```ts
page.locator('#input-83921')
```

is fragile.

Look for a stable identity such as:

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

## Simple Understanding

Some applications generate classes dynamically.

Example:

```html
class="dynamic-83921"
```

If that value changes, using it as a locator is fragile.

Prefer a stable semantic locator or stable test attribute.

---

# Dynamic Text

## Simple Understanding

If only part of the text changes, keep the stable part and use regex for the changing part.

Example:

```text
Live counter: 247
```

then:

```text
Live counter: 248
```

Use:

```ts
page.getByText(/Live counter: \d+/)
```

instead of hard-coding:

```ts
page.getByText('Live counter: 247')
```

---

# Fragile DOM-Dependent Locators

Example:

```ts
page.locator(
    'div:nth-child(3) > div:nth-child(2) > button'
)
```

## Why Is This Fragile?

Because it depends on:

- exact DOM structure
- exact position
- exact nesting

If the UI changes, the locator can break or target another element.

If a meaningful semantic locator exists:

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

is usually much better.

---

# Strict Mode

## Simple Understanding

Strict mode is Playwright telling us:

> **"Your locator matched more than one element, and I don't know which one you mean."**

This is useful feedback.

It helps us discover ambiguous locators.

---

# Example

Suppose there are three Edit buttons:

```ts
page.getByRole('button', {
    name: 'Edit'
})
```

If a single-element action is attempted and multiple elements match, Playwright can report a strict mode violation.

The solution is usually **not**:

```ts
.nth(0)
```

The better question is:

> Which Edit button do I actually need?

---

# Strict Mode vs `nth()`

## Simple Understanding

If the requirement is:

> Edit Rahul Verma.

Use identification:

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
    .getByRole('button', {
        name: 'Edit'
    })
```

If the requirement is:

> Click the third item.

Use position:

```ts
page.getByRole('listitem').nth(2)
```

---

# Senior SDET Principle

> **Make the locator identify the correct element, rather than making the test accidentally click the correct element.**

---

# 8. Day 7 — Advanced Locator Scenarios

# Iframes

## Simple Understanding

An iframe is like a **separate page/document living inside the main page**.

Because it has its own document, we first need to enter its context.

Think:

```text
Main Page
   ↓
iframe
   ↓
Element inside iframe
```

Use:

```ts
page.frameLocator(...)
```

Then use normal Playwright locators inside it.

---

# Technical Understanding

An iframe represents a separate browsing context/document.

`frameLocator()` lets us locate elements inside that frame.

Example:

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
Page
  ↓
frameLocator()
  ↓
Element inside iframe
```

### Important Rule

> **Iframe → establish frame context first → then use normal Playwright locators.**

---

# Open Shadow DOM

## Simple Understanding

Shadow DOM is a way web components can keep their internal DOM separated from the regular page DOM.

For an **open Shadow DOM**, Playwright locators can generally **pierce through the shadow boundary automatically**.

So we normally do not need a special Shadow DOM locator syntax.

Mental model:

```text
Open Shadow DOM
       ↓
Normal Playwright locator
```

---

# Iframe vs Shadow DOM

This distinction is important.

```text
iframe
→ separate document
→ use frameLocator()

open Shadow DOM
→ component DOM boundary
→ normal Playwright locators can generally pierce it
```

---

# SVG and Icon Buttons

## Simple Understanding

Sometimes the user sees only an icon:

```text
🗑️
```

But the thing the user actually interacts with is the **button**.

Example:

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

## Why?

Because:

> **Locate the user interaction, not an unnecessary implementation detail.**

---

# ARIA Attributes

## Simple Understanding

ARIA attributes help describe UI elements to **assistive technologies such as screen readers**.

They can provide information such as:

- what an element is called
- what another element labels
- whether something is expanded
- whether something is selected
- whether something is checked

Common ARIA attributes include:

```text
aria-label
aria-labelledby
aria-describedby
aria-expanded
aria-checked
aria-selected
```

---

# `aria-label`

## Simple Understanding

`aria-label` directly gives an element an **accessible name**.

Example:

```html
<button aria-label="Delete Account">
    🗑️
</button>
```

The user sees the icon, but the accessible name is:

```text
Delete Account
```

So:

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

can identify it.

## Technical Understanding

`aria-label` provides an accessible name directly when appropriate.

---

# `aria-labelledby`

## Simple Understanding

`aria-labelledby` tells screen reader technology:

> **"Use the text from this other element as the accessible name for me."**

In other words:

> Instead of writing the accessible name directly on this element, point to another element that contains the name.

Example:

```html
<h2 id="payment-title">
    Payment Details
</h2>

<div
    role="dialog"
    aria-labelledby="payment-title">
</div>
```

The dialog's accessible name comes from the text:

```text
Payment Details
```

because:

```text
aria-labelledby="payment-title"
```

points to:

```text
id="payment-title"
```

## Technical Understanding

`aria-labelledby` references one or more element IDs whose text is used to calculate the accessible name.

This is especially useful when the visible text already exists elsewhere in the UI and should serve as the accessible name.

## Why Is It Useful for Playwright?

Because Playwright's role-based locators can use the resulting accessible name.

For example:

```ts
page.getByRole('dialog', {
    name: 'Payment Details'
})
```

The important connection is:

```text
Visible label element
        ↓
aria-labelledby
        ↓
Accessible name
        ↓
getByRole(..., { name: ... })
```

---

# `aria-describedby`

## Simple Understanding

`aria-describedby` tells assistive technology:

> **"Use this other element's text as additional descriptive information for me."**

Think:

```text
aria-labelledby
→ What is this called?

aria-describedby
→ What additional information describes it?
```

Example:

```html
<label id="password-label">
    Password
</label>

<p id="password-help">
    Minimum 8 characters
</p>

<input
    aria-labelledby="password-label"
    aria-describedby="password-help">
```

Conceptually:

```text
Accessible name
→ Password

Additional description
→ Minimum 8 characters
```

---

# Other Important ARIA Attributes

## `aria-expanded`

### Simple Understanding

Tells assistive technology whether something such as a menu or accordion is currently expanded or collapsed.

Example:

```html
<button
    aria-expanded="false">
    Details
</button>
```

---

## `aria-checked`

### Simple Understanding

Helps communicate the checked state of certain custom controls.

---

## `aria-selected`

### Simple Understanding

Helps communicate which item is currently selected, such as in tabs or other selectable widgets.

---

# Custom Controls

## Simple Understanding

Do not assume the HTML tag always tells us how the user experiences the element.

Example:

```html
<div
    role="button"
    aria-label="Open Settings">
    ⚙️
</div>
```

Even though the HTML tag is:

```text
div
```

the declared role is:

```text
button
```

So use:

```ts
page.getByRole('button', {
    name: 'Open Settings'
})
```

---

# Locator Debugging

## Simple Understanding

When a locator fails, don't immediately try random selectors.

First ask:

> **"Why couldn't Playwright identify this element?"**

---

# Debugging Checklist

Check in this order:

```text
1. Does the element exist?
2. What is its actual role?
3. What is its accessible name?
4. Is it inside an iframe?
5. Is it inside a Shadow DOM?
6. Are there multiple matching elements?
7. Is the locator stable?
8. Am I targeting the correct component?
```

---

# Useful Debugging Tools

Debug mode:

```bash
npx playwright test --debug
```

Code generation:

```bash
npx playwright codegen <URL>
```

## Important

Codegen can suggest locators.

But:

> **Codegen gives us a starting point; the engineer still decides whether the locator is good.**

Review generated locators for:

- stability
- uniqueness
- readability
- semantic quality
- maintainability

---

# 9. Day 8 — Locator Mastery and Decision Making

## Simple Understanding

Day 8 is not about learning another locator syntax.

It is about combining everything we learned and making the right decision independently.

The goal is:

> **See UI → understand UI → identify the element → choose the best locator → explain why.**

---

# The Senior SDET Question

Instead of:

> "Which locator syntax can I use?"

ask:

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

Possible locators:

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

The semantic locator is often the strongest choice:

```ts
page.getByRole('button', {
    name: 'Approve Request'
})
```

because it identifies the user-facing control.

The test ID is also a good stable option.

---

# Why Prefer Semantic Locators?

## Simple Understanding

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

The second tells another engineer:

> "I am interacting with the Approve Request button."

The first tells them:

> "Find something having this CSS class."

The semantic locator expresses more test intent.

---

# Real-World Repeated Components

Suppose:

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

Think:

```text
Find Rahul's row
       ↓
Find Delete inside Rahul's row
```

Code:

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

# Why `nth()` Is Dangerous Here

This:

```ts
await page.getByRole('button', {
    name: 'Delete'
}).nth(2).click();
```

assumes the target is always the third Delete button.

But if someone:

- adds a row
- removes a row
- sorts the table
- filters the table
- changes the data

the position can change.

The test could then act on the wrong user.

---

# When `nth()` Is Correct

Requirement:

> Click the third item.

Then:

```ts
await page.getByRole('listitem').nth(2).click();
```

is correct.

Because:

> **Third is part of the requirement.**

---

# Identification vs Position

This distinction should become automatic.

## Identification Requirement

```text
Delete Rahul Verma
```

Use identity:

```text
Rahul Verma → correct row
Delete       → inside row
```

## Position Requirement

```text
Click the third item
```

Use:

```ts
nth(2)
```

---

# The Golden Rule

> **Use meaning to identify elements. Use position only when position is genuinely part of the requirement.**

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

If a meaningful semantic locator clearly identifies the element, prefer it over unnecessary implementation details.

---

## Rule 3 — Stability Matters

Avoid selectors based on values that frequently change.

Examples:

```text
dynamic IDs
dynamic classes
DOM position
```

---

## Rule 4 — Uniqueness Matters

A locator should identify the intended element.

If multiple elements match, make the locator more specific.

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

just because multiple elements exist.

Use it when position itself is the requirement.

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

A strict mode violation can tell you:

> **Your locator is not uniquely identifying the intended element.**

---

## Rule 10 — Debug the Cause

When a locator fails:

> **Understand why before changing it.**

---

## Rule 11 — Codegen Is a Helper

Codegen can suggest a locator.

It does not replace engineering judgment.

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
Is it maintainable?
```

---

# 12. Practical Locator Patterns

## Pattern 1 — Semantic Button

```ts
page.getByRole('button', {
    name: 'Save Changes'
})
```

### Simple Meaning

> Find the Save Changes button.

---

## Pattern 2 — Form Field by Purpose

```ts
page.getByLabel('First Name')
```

### Simple Meaning

> Find the field whose purpose is First Name.

---

## Pattern 3 — Form Field by Hint

```ts
page.getByPlaceholder('Enter first name')
```

### Simple Meaning

> Find the field using its placeholder hint.

---

## Pattern 4 — Visible Text

```ts
page.getByText('Locator', {
    exact: true
})
```

### Simple Meaning

> Find the element displaying exactly this text.

---

## Pattern 5 — Dynamic Text

```ts
page.getByText(/Live counter: \d+/)
```

### Simple Meaning

> Find text where the stable part is "Live counter:" and the number can change.

---

## Pattern 6 — Test ID

```ts
page.getByTestId('input-month')
```

### Simple Meaning

> Find the element using its dedicated testing identity.

---

## Pattern 7 — CSS Fallback

```ts
page.locator("[type='email']")
```

### Simple Meaning

> Find the DOM element using a stable attribute when a better user-facing locator is not available.

---

## Pattern 8 — Find Component by Text

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
```

### Simple Meaning

> Find the row that belongs to Rahul Verma.

---

## Pattern 9 — Find Target Inside Component

```ts
page.getByRole('row')
    .filter({
        hasText: 'Rahul Verma'
    })
    .getByRole('button', {
        name: 'Delete'
    })
```

### Simple Meaning

> Find Rahul's row, then find Delete inside that row.

---

## Pattern 10 — Positional Requirement

```ts
page.getByRole('listitem').nth(2)
```

### Simple Meaning

> Find the third list item.

---

## Pattern 11 — Iframe

```ts
page.frameLocator('#demoFrame')
    .getByRole('button', {
        name: 'Submit'
    })
```

### Simple Meaning

> Enter the iframe, then find the Submit button inside it.

---

## Pattern 12 — Icon Button

```ts
page.getByRole('button', {
    name: 'Delete Account'
})
```

### Simple Meaning

> Even though I see an icon, find the actual button using its accessible name.

---

# 13. Final Mental Model

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

# 14. Final Locator Philosophy

## Simple Understanding

When you see an element, don't immediately start writing CSS.

First understand the element.

Ask:

```text
What is it?
What is it called?
What is its purpose?
What identifies it?
Is that identity stable?
Is it unique?
Does it belong to a repeated component?
Is it inside an iframe?
Is it inside open Shadow DOM?
```

Then choose the locator.

---

# Final One-Line Rule

> **Find the element by what it means, narrow to the correct component when necessary, and use position only when position is actually the requirement.**

# Final Senior SDET Statement

> **A Senior SDET does not choose a locator simply because it works. They choose the locator that most reliably expresses why that exact element is the one the test intends to interact with.**
