# Playwright Frame Handling — Mastery Notes

> **Goal:** Understand frames so well that when you see an element on a web page, you can immediately decide whether it is in the main DOM, an iframe, or a nested iframe — and choose the correct Playwright approach.

---

## 📚 Table of Contents

1. [What is a Frame / iFrame?](#1-what-is-a-frame--iframe)
2. [The Most Important Mental Map](#2-the-most-important-mental-map)
3. [Why Can't We Directly Use `page.locator()`?](#3-why-cant-we-directly-use-pagelocator)
4. [`frameLocator()`](#4-framelocator)
5. [Using Normal Locators Inside a Frame](#5-using-normal-locators-inside-a-frame)
6. [Direct Chaining](#6-direct-chaining)
7. [`frameLocator()` vs `locator()`](#7-framelocator-vs-locator)
8. [`frameLocator()` vs `frame()`](#8-framelocator-vs-frame)
9. [Using `page.frame()`](#9-using-pageframe)
10. [Why `?.` Is Used](#10-why--is-used)
11. [Finding a Frame by Name](#11-finding-a-frame-by-name)
12. [`page.frames()`](#12-pageframes)
13. [Finding a Frame by URL](#13-finding-a-frame-by-url)
14. [`page.mainFrame()`](#14-pagemainframe)
15. [Nested Frames](#15-nested-frames)
16. [Frame + Locator Chaining](#16-frame--locator-chaining)
17. [Dynamic Iframes](#17-dynamic-iframes)
18. [Frame Events](#18-frame-events)
19. [`page.on()` vs `waitForEvent()`](#19-pageon-vs-waitforevent)
20. [Frame URL](#20-frame-url)
21. [Cross-Origin Iframes](#21-cross-origin-iframes)
22. [Real-World Examples](#22-real-world-examples)
23. [When to Use Which API](#23-when-to-use-which-api)
24. [Common Mistakes](#24-common-mistakes)
25. [Debugging an iframe Problem](#25-debugging-an-iframe-problem)
26. [Senior SDET Thinking](#26-senior-sdet-thinking)
27. [Interview Questions](#27-interview-questions)
28. [Must Know vs Nice to Know](#28-must-know-vs-nice-to-know)
29. [Final Mental Map](#29-final-mental-map)
30. [⚡ 30-Second Revision](#30-30-second-revision)
31. [Final Memory Rules](#31-final-memory-rules)

---

# 1. What is a Frame / iFrame?

An **iframe** is an HTML document embedded inside another HTML page.

```text
Main Page
────────────────────────────────

Username: __________

┌───────────────────────────────┐
│           iframe              │
│                               │
│ Card Number: __________       │
│ Expiry:      __________       │
│ [ Pay ]                      │
│                               │
└───────────────────────────────┘
```

The important point:

> **An iframe has its own DOM.**

So Playwright treats the iframe content separately from the main page DOM.

### Simple way to think about it

```text
Main Page
   │
   └── iframe
          │
          └── Separate HTML document
```

---

# 2. The Most Important Mental Map

```text
PAGE
│
├── Main Page DOM
│     │
│     └── page.locator()
│
└── iframe
      │
      └── Separate DOM
            │
            └── frameLocator()
                    │
                    └── locator()
                         │
                         └── Element
```

### Golden Rule

> **Main page element → `page.locator()`**

> **Element inside iframe → `page.frameLocator()` → locator**

---

# 3. Why Can't We Directly Use `page.locator()`?

Suppose:

```html
<iframe id="paymentFrame">
    <input id="cardNumber">
</iframe>
```

This will not work:

```typescript
await page.locator("#cardNumber").fill("1234");
```

Why?

Because `#cardNumber` is **not part of the main page DOM**.

It belongs to the iframe's DOM.

Correct:

```typescript
await page
    .frameLocator("#paymentFrame")
    .locator("#cardNumber")
    .fill("1234");
```

### Think like this

```text
Main Page DOM
│
├── Header
├── Menu
├── iframe
│     │
│     └── cardNumber  ← element lives here
└── Footer
```

Therefore:

```text
page.locator("#cardNumber")
        ↓
Looks in main DOM
        ↓
Doesn't find it
```

Whereas:

```text
page.frameLocator("#paymentFrame")
        ↓
Enters iframe context
        ↓
locator("#cardNumber")
        ↓
Finds element
```

---

# 4. `frameLocator()`

## What does it do?

`frameLocator()` creates a locator context for an iframe.

### Syntax

```typescript
page.frameLocator("iframe selector")
```

### Example

```typescript
const paymentFrame = page.frameLocator("#paymentFrame");
```

Then:

```typescript
await paymentFrame
    .locator("#cardNumber")
    .fill("1234");
```

### Read this as English

```text
page.frameLocator("#paymentFrame")
        ↓
"Work inside this iframe"

locator("#cardNumber")
        ↓
"Find this element inside the iframe"
```

---

## Complete Example

HTML:

```html
<iframe id="loginFrame">
    <input id="username">
    <input id="password">
    <button>Login</button>
</iframe>
```

Playwright:

```typescript
const loginFrame = page.frameLocator("#loginFrame");

await loginFrame
    .locator("#username")
    .fill("Karthik");

await loginFrame
    .locator("#password")
    .fill("Password123");

await loginFrame
    .getByRole("button", { name: "Login" })
    .click();
```

---

# 5. Using Normal Locators Inside a Frame

Once you have:

```typescript
const frame = page.frameLocator("#loginFrame");
```

you can use your normal Playwright locator knowledge.

Examples:

```typescript
frame.locator()
frame.getByRole()
frame.getByText()
frame.getByLabel()
frame.getByPlaceholder()
frame.getByTestId()
```

### `getByRole()`

```typescript
await frame
    .getByRole("textbox", { name: "Username" })
    .fill("Karthik");
```

### `getByLabel()`

```typescript
await frame
    .getByLabel("Username")
    .fill("Karthik");
```

### `getByPlaceholder()`

```typescript
await frame
    .getByPlaceholder("Enter username")
    .fill("Karthik");
```

### CSS / other locator

```typescript
await frame
    .locator("#username")
    .fill("Karthik");
```

### Important

> **`frameLocator()` changes the context. Your normal Playwright locator skills still apply inside that context.**

---

# 6. Direct Chaining

You don't always need a variable.

Instead of:

```typescript
const frame = page.frameLocator("#loginFrame");

await frame
    .getByRole("button", { name: "Login" })
    .click();
```

You can write:

```typescript
await page
    .frameLocator("#loginFrame")
    .getByRole("button", { name: "Login" })
    .click();
```

Both are valid.

### When a variable is useful

If you interact with the same frame multiple times:

```typescript
const paymentFrame =
    page.frameLocator("#paymentFrame");

await paymentFrame
    .getByLabel("Card Number")
    .fill("1234");

await paymentFrame
    .getByLabel("Expiry")
    .fill("12/30");

await paymentFrame
    .getByLabel("CVV")
    .fill("123");
```

This is usually easier to read.

---

# 7. `frameLocator()` vs `locator()`

This distinction is extremely important.

## `locator()`

```typescript
page.locator("#username")
```

Means:

> Find an element in the current page/frame DOM context.

## `frameLocator()`

```typescript
page.frameLocator("#loginFrame")
```

Means:

> Locate an iframe and create a context for interacting with elements inside it.

### Mental Map

```text
locator()
   ↓
ELEMENT

frameLocator()
   ↓
IFRAME CONTEXT
   ↓
ELEMENT
```

### Easy memory trick

> **`locator()` → find element**

> **`frameLocator()` → enter iframe context**

---

# 8. `frameLocator()` vs `frame()`

There are two important approaches.

## Approach 1 — `frameLocator()`

```typescript
const frame =
    page.frameLocator("#payment");

await frame
    .locator("#cardNumber")
    .fill("1234");
```

This gives you a:

```text
FrameLocator
```

It is primarily used for normal UI interaction inside an iframe.

---

## Approach 2 — `frame()`

```typescript
const frame = page.frame({
    name: "payment"
});
```

This gives you the actual:

```text
Frame
```

object.

### Simple mental map

```text
Need to interact with elements?
        ↓
frameLocator()
```

```text
Need the actual Frame object?
        ↓
frame()
```

### Interview memory

> **`frameLocator()` → Locator mindset**

> **`frame()` → Frame object mindset**

---

# 9. Using `page.frame()`

Suppose:

```html
<iframe name="paymentFrame">
```

You can do:

```typescript
const frame = page.frame({
    name: "paymentFrame"
});
```

Then:

```typescript
await frame?.locator("#cardNumber").fill("1234");
```

`page.frame()` is useful when you need to work with the actual `Frame` object.

---

# 10. Why `?.` Is Used

This is TypeScript **optional chaining**.

`page.frame()` can return:

```text
Frame
OR
null
```

Therefore:

```typescript
const frame = page.frame({
    name: "paymentFrame"
});
```

may result in:

```text
frame = Frame
```

or:

```text
frame = null
```

So:

```typescript
frame?.locator()
```

means:

> If the frame exists, continue. If it is `null`, don't try to call the method.

### Important TypeScript point

```text
page.frame()
      ↓
Frame | null
```

Whereas:

```text
page.frameLocator()
      ↓
FrameLocator
```

---

# 11. Finding a Frame by Name

If the iframe is:

```html
<iframe name="paymentFrame">
```

Use:

```typescript
const frame = page.frame({
    name: "paymentFrame"
});
```

Then:

```typescript
await frame?.locator("#cardNumber").fill("1234");
```

### Important

Use a stable identifier when possible.

Avoid relying on frame position such as:

```typescript
page.frames()[1]
```

unless there is a specific reason.

---

# 12. `page.frames()`

```typescript
const frames = page.frames();
```

This gives you the frames associated with the page.

Think:

```text
PAGE
│
├── Main Frame
├── iframe 1
├── iframe 2
└── iframe 3
```

### Mental map

```text
page.frames()
      ↓
all available frames
```

---

# 13. Finding a Frame by URL

Sometimes the iframe doesn't have a useful ID or name.

You can inspect frames:

```typescript
const frames = page.frames();
```

Or directly find one:

```typescript
const paymentFrame = page
    .frames()
    .find(frame =>
        frame.url().includes("payment")
    );
```

Then:

```typescript
await paymentFrame
    ?.locator("#cardNumber")
    .fill("1234");
```

### Mental Map

```text
page
 ↓
frames()
 ↓
all frames
 ↓
find()
 ↓
required frame
```

---

# 14. `page.mainFrame()`

The page itself has a main frame.

You can access it using:

```typescript
page.mainFrame()
```

Mental map:

```text
PAGE
│
├── Main Frame
│
├── Child Frame
│
└── Child Frame
```

The main frame represents the main document.

### Useful concept

> An iframe is a child frame of the page's main frame.

---

# 15. Nested Frames

This is one of the most important real-world and interview scenarios.

Suppose:

```text
Main Page
    │
    └── Frame A
          │
          └── Frame B
                │
                └── Button
```

You must follow the hierarchy.

```typescript
await page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
    .getByRole("button", {
        name: "Submit"
    })
    .click();
```

### Read it like English

> Go to Frame A → go to Frame B → find Submit → click.

---

## Nested Frame Mental Map

```text
page
 ↓
frameLocator("#frameA")
 ↓
iframe A
 ↓
frameLocator("#frameB")
 ↓
iframe B
 ↓
getByRole("button")
 ↓
click()
```

### Golden Rule

> **If an iframe is inside another iframe, follow the frame hierarchy.**

---

# 16. Frame + Locator Chaining

Suppose:

```text
iframe
 ↓
form
 ↓
button
```

You can do:

```typescript
await page
    .frameLocator("#paymentFrame")
    .locator("form")
    .getByRole("button", {
        name: "Pay"
    })
    .click();
```

This is the same locator-chaining concept you already know.

### Think

```text
page
 ↓
frame
 ↓
form
 ↓
button
 ↓
click
```

---

# 17. Dynamic Iframes

Sometimes an iframe is created after an action.

Example:

```text
Click Pay
   ↓
Payment iframe appears
   ↓
Enter card details
```

You can still use:

```typescript
const frame =
    page.frameLocator("#paymentFrame");

await frame
    .locator("#cardNumber")
    .fill("4111111111111111");
```

Playwright's locator model provides normal waiting behavior for the locator/action.

### Don't automatically add:

```typescript
await page.waitForTimeout(5000);
```

Just because an iframe appears dynamically.

Prefer meaningful locator-based synchronization.

---

# 18. Frame Events

Playwright provides frame lifecycle events.

Important events:

```typescript
page.on("frameattached", ...)
page.on("framenavigated", ...)
page.on("framedetached", ...)
```

### Mental Map

```text
iframe created
      ↓
frameattached

iframe navigates
      ↓
framenavigated

iframe removed
      ↓
framedetached
```

---

## `frameattached`

Used when a frame is attached.

```typescript
page.on("frameattached", frame => {
    console.log("Frame attached:", frame.url());
});
```

Useful for debugging dynamically created frames.

---

## `framenavigated`

Used when a frame navigates.

```typescript
page.on("framenavigated", frame => {
    console.log("Frame navigated:", frame.url());
});
```

---

## `framedetached`

Used when a frame is removed.

```typescript
page.on("framedetached", frame => {
    console.log("Frame detached:", frame.url());
});
```

---

# 19. `page.on()` vs `waitForEvent()`

This connects directly to the event concept you already learned.

## `page.on()`

Means:

> **Keep listening for the event.**

Example:

```typescript
page.on("frameattached", frame => {
    console.log(frame.url());
});
```

It can react whenever the event occurs.

---

## `page.waitForEvent()`

Means:

> **Wait for one occurrence of an event.**

Conceptually:

```typescript
const framePromise =
    page.waitForEvent("frameattached");
```

Then you perform the action that causes the event.

### Mental Map

```text
page.on()
   ↓
"Keep listening"

waitForEvent()
   ↓
"Wait for one event occurrence"
```

Don't use either one just because an iframe exists. Use them when event-driven synchronization or debugging is actually required.

---

# 20. Frame URL

A `Frame` object has a URL.

```typescript
console.log(frame.url());
```

Useful for identifying and debugging frames.

Example:

```typescript
const paymentFrame = page
    .frames()
    .find(frame =>
        frame.url().includes("payment")
    );
```

---

## Inspect All Frame URLs

Very useful debugging technique:

```typescript
for (const frame of page.frames()) {
    console.log(frame.url());
}
```

This gives you a simple picture of the frames currently associated with the page.

---

# 21. Cross-Origin Iframes

Example:

```text
Application
     │
     └── iframe
            │
            └── payment-provider.com
```

The iframe may belong to another domain.

Do not use this shortcut in your thinking:

> Different domain = Playwright cannot automate it.

Playwright provides frame APIs for interacting with frame content where browser automation permits it.

The important practical model is:

```text
Main application
       ↓
iframe
       ↓
frameLocator()
       ↓
element
```

---

# 22. Real-World Examples

## Example 1 — Payment iframe

Imagine:

```text
Checkout Page

Name
Email

┌───────────────────────────┐
│ Payment Provider iframe   │
│                           │
│ Card Number               │
│ Expiry                    │
│ CVV                       │
└───────────────────────────┘
```

Automation:

```typescript
const paymentFrame =
    page.frameLocator("#paymentFrame");

await paymentFrame
    .getByLabel("Card number")
    .fill("4111111111111111");

await paymentFrame
    .getByLabel("Expiry")
    .fill("12/30");

await paymentFrame
    .getByLabel("CVV")
    .fill("123");
```

The important concept is:

```text
Page
 ↓
Payment iframe
 ↓
Elements inside iframe
```

---

## Example 2 — Rich Text Editor

Some applications use iframe-based editors.

Example:

```text
Application
    │
    └── iframe
          │
          └── Editor body
```

Possible automation:

```typescript
await page
    .frameLocator("#editorFrame")
    .locator("body")
    .fill("This is my content");
```

The exact locator depends on the editor implementation.

---

## Example 3 — Embedded Report

Applications sometimes embed reporting applications through iframes.

Example:

```text
Application
    │
    └── iframe
          │
          └── Report
                │
                └── Filter
```

Possible automation:

```typescript
await page
    .frameLocator("#reportFrame")
    .getByRole("button", {
        name: "Filters"
    })
    .click();
```

The exact locator depends on the application.

---

# 23. When to Use Which API

| Situation | Preferred approach |
|---|---|
| Normal element on main page | `page.locator()` |
| Element inside iframe | `page.frameLocator()` |
| Nested iframe | Chain `frameLocator()` |
| Need actual `Frame` object | `page.frame()` |
| Need to inspect all frames | `page.frames()` |
| Find frame programmatically by URL | `page.frames()` + `find()` |
| Access main document frame | `page.mainFrame()` |
| Debug frame creation | `frameattached` |
| Debug frame navigation | `framenavigated` |
| Debug frame removal | `framedetached` |

### Quick Decision Tree

```text
Is the element inside an iframe?
             │
         ┌───┴───┐
        NO      YES
        │         │
        ↓         ↓
page.locator()  Need normal UI interaction?
                  │
                  YES
                  ↓
            frameLocator()
```

If you need the actual frame:

```text
Need Frame object?
       ↓
page.frame()
```

If you need to inspect/find frames:

```text
Need to inspect frames?
       ↓
page.frames()
```

---

# 24. Common Mistakes

## Mistake 1 — Using `page.locator()` for iframe content

### Wrong

```typescript
await page
    .locator("#cardNumber")
    .fill("1234");
```

### Why?

`#cardNumber` is inside an iframe.

### Correct

```typescript
await page
    .frameLocator("#paymentFrame")
    .locator("#cardNumber")
    .fill("1234");
```

---

## Mistake 2 — Treating `locator()` as entering an iframe

### Wrong

```typescript
page
    .locator("#paymentFrame")
    .locator("#cardNumber");
```

This means:

> Find an element called `#paymentFrame`, then find a descendant `#cardNumber`.

It does **not** mean:

> Enter the iframe.

Use:

```typescript
page
    .frameLocator("#paymentFrame")
    .locator("#cardNumber");
```

---

## Mistake 3 — Accessing nested iframe directly

Structure:

```text
Frame A
  ↓
Frame B
  ↓
Button
```

### Wrong

```typescript
page.frameLocator("#frameB")
```

### Correct

```typescript
page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
```

---

## Mistake 4 — Relying on frame index

Avoid:

```typescript
const frame = page.frames()[1];
```

unless there is a specific reason.

Frame order can change.

Prefer a stable identification method.

For normal UI interaction:

```typescript
page.frameLocator("#loginFrame")
```

---

## Mistake 5 — Using arbitrary sleeps

Avoid:

```typescript
await page.waitForTimeout(5000);
```

Prefer locator-based synchronization.

---

## Mistake 6 — Confusing FrameLocator and Frame

```text
frameLocator()
       ↓
FrameLocator

frame()
       ↓
Frame
```

They are different types and serve different purposes.

---

## Mistake 7 — Assuming every iframe has an ID

The iframe may be identified using:

- ID
- name
- CSS selector
- other attributes
- URL
- relationship to another frame

Example:

```typescript
page.frameLocator('iframe[name="payment"]')
```

---

# 25. Debugging an iframe Problem

When Playwright cannot find an element, don't immediately assume the locator is wrong.

Use this debugging flow:

```text
1. Confirm iframe exists
        ↓
2. Inspect iframe selector
        ↓
3. Confirm element is actually inside iframe
        ↓
4. Check whether iframe is nested
        ↓
5. Inspect page.frames()
        ↓
6. Check frame URLs
        ↓
7. Verify locator inside the frame
```

### Useful debugging code

```typescript
console.log(
    page.frames().map(frame => frame.url())
);
```

Or:

```typescript
for (const frame of page.frames()) {
    console.log("FRAME:", frame.url());
}
```

---

# 26. Senior SDET Thinking

When you encounter an element that Playwright cannot find:

Don't immediately think:

> "My locator is wrong."

Think:

```text
Can I see the element?
       ↓
YES
       ↓
Is it inside an iframe?
       ↓
YES
       ↓
Which frame?
       ↓
Is it nested?
       ↓
Build the frame hierarchy.
```

### Senior-level debugging mindset

```text
Element not found
       ↓
Check DOM
       ↓
Check iframe
       ↓
Check nested iframe
       ↓
Check locator
       ↓
Check synchronization
```

This prevents random changes to locators.

---

# 27. Interview Questions

## Q1. What is an iframe?

### Interview-ready answer

> An iframe is a separate HTML document embedded inside a web page. Because it has its own DOM context, elements inside the iframe cannot normally be located directly from the parent page locator. In Playwright, I generally use `frameLocator()` to interact with elements inside an iframe.

---

## Q2. How do you handle iframes in Playwright?

### Answer

> For normal UI interaction, I use `page.frameLocator()` and then use standard Playwright locators such as `getByRole()`, `getByLabel()` or `locator()` to interact with elements inside the frame. If I need the actual `Frame` object, I use `page.frame()` or inspect frames through `page.frames()`.

---

## Q3. Difference between `frameLocator()` and `frame()`?

### Answer

> `frameLocator()` returns a `FrameLocator` and is convenient for locating and interacting with elements inside an iframe. `page.frame()` returns the actual `Frame` object, which is useful when I need frame-level APIs or programmatic frame handling.

---

## Q4. How do you handle nested frames?

### Answer

> I follow the frame hierarchy using `frameLocator()`. If Frame B is inside Frame A, I first enter Frame A and then locate Frame B.

Example:

```typescript
await page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
    .getByRole("button", { name: "Submit" })
    .click();
```

---

## Q5. How do you find a frame when it has no ID?

Possible approaches include:

```typescript
page.frameLocator("iframe[name='payment']")
```

or another stable iframe selector.

If programmatic identification is needed:

```typescript
const frame = page
    .frames()
    .find(f => f.url().includes("payment"));
```

---

## Q6. How do you debug an iframe problem?

### Answer

> First I verify whether the element is inside an iframe. Then I check whether the iframe is nested, inspect `page.frames()`, check frame URLs, and finally verify the locator within the correct frame context.

---

## Q7. What is `page.frames()` used for?

### Answer

> `page.frames()` returns the frames associated with the page. I use it when I need to inspect available frames or find a specific frame programmatically, such as by its URL.

---

## Q8. What is `page.mainFrame()`?

### Answer

> `page.mainFrame()` returns the main frame representing the page's main document.

---

## Q9. What are `frameattached`, `framenavigated`, and `framedetached`?

### Answer

> They are frame lifecycle events. `frameattached` occurs when a frame is attached, `framenavigated` when a frame navigates, and `framedetached` when a frame is removed.

---

# 28. Must Know vs Nice to Know

This is useful when preparing for interviews.

## 🔴 MUST KNOW

You should be able to write these without looking them up:

```typescript
page.frameLocator()
```

```typescript
page
    .frameLocator("#frame")
    .getByRole("button")
    .click();
```

```typescript
page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
```

Know:

- Main DOM vs iframe DOM
- `frameLocator()`
- `frame()`
- `page.frames()`
- Nested frames
- `frameLocator()` vs `frame()`
- iframe locator chaining
- Debugging frame issues

---

## 🟡 SHOULD KNOW

Know the concept and be able to explain it:

- `page.mainFrame()`
- Find frame by URL
- `frame.url()`
- Dynamic iframes
- Cross-origin iframe concept
- `frameattached`
- `framenavigated`
- `framedetached`
- `page.on()` vs `waitForEvent()`

---

## 🟢 NICE TO KNOW

Useful for senior-level debugging and framework work:

- Frame lifecycle
- Programmatic frame discovery
- Complex nested frame structures
- Frame-related debugging strategies
- Choosing between locator-based and frame-object approaches

---

# 29. Final Mental Map

```text
                    FRAME HANDLING
                          │
          ┌───────────────┴────────────────┐
          │                                │
     Normal UI                         Programmatic
     interaction                         handling
          │                                │
          ↓                                ↓
   frameLocator()                       frame()
          │                                │
          ↓                                ↓
   FrameLocator                         Frame
          │                                │
          ↓                                ├── url()
   getByRole()                           ├── locator()
   getByLabel()                          ├── evaluate()
   locator()                             └── other APIs
          │
          ↓
      Element
```

For finding/inspecting frames:

```text
page.frames()
      ↓
all frames
      ↓
find required frame
      ↓
Frame
```

---

# 30. ⚡ 30-Second Revision

Read only this section before an interview.

- `page.locator()` → element in current DOM context
- `page.frameLocator()` → iframe UI context
- `page.frame()` → actual `Frame` object
- `page.frames()` → all frames
- `page.mainFrame()` → main document frame
- iframe has its own DOM
- Element inside iframe → use `frameLocator()`
- Nested iframe → chain `frameLocator()`
- `frame.url()` → identify/debug frame
- Avoid relying on `page.frames()[1]`
- Avoid unnecessary `waitForTimeout()`
- `frameattached` → frame created/attached
- `framenavigated` → frame navigated
- `framedetached` → frame removed

### The Golden Question

> **Where does the element live?**

```text
Main DOM
   ↓
page.locator()

Iframe
   ↓
page.frameLocator()

Nested iframe
   ↓
frameLocator().frameLocator()

Need actual Frame object
   ↓
page.frame()

Need to inspect frames
   ↓
page.frames()
```

---

# 31. Final Memory Rules

### Rule 1

> **`locator()` finds an element.**

### Rule 2

> **`frameLocator()` gives you an iframe context for UI interaction.**

### Rule 3

> **`frame()` gives you the actual Frame object.**

### Rule 4

> **`frames()` helps you inspect/find frames.**

### Rule 5

> **Nested iframe = follow the hierarchy.**

### Rule 6

> **First ask where the element lives, then choose the locator strategy.**

---

# 🧠 Final Mental Model

```text
              PAGE
                │
        ┌───────┴────────┐
        │                │
    Main DOM          iframe
        │                │
   locator()       frameLocator()
                         │
                         ↓
                    elements
```

For nested frames:

```text
PAGE
 ↓
Frame A
 ↓
Frame B
 ↓
Frame C
 ↓
Element
```

Code:

```typescript
page
    .frameLocator("#A")
    .frameLocator("#B")
    .frameLocator("#C")
    .locator("#element");
```

> **Don't think "How do I find the element?" first.**
>
> Think:
>
> **"Where does the element live?"**

That single question will guide most of your Frame Handling decisions.
