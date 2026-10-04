# Playwright Frame Handling — Mastery Notes

## 1. What is a Frame / iFrame?

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

> An iframe has its **own DOM**.

So Playwright treats the iframe content separately from the main page DOM.

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

### Remember:

> **Element in main page → `page.locator()`**

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

Therefore:

```typescript
await page
    .frameLocator("#paymentFrame")
    .locator("#cardNumber")
    .fill("1234");
```

---

# 4. `frameLocator()` — The Most Important API

Syntax:

```typescript
page.frameLocator("iframe selector")
```

Example:

```typescript
const paymentFrame = page.frameLocator("#paymentFrame");
```

Now:

```typescript
await paymentFrame.locator("#cardNumber").fill("1234");
```

### Read this as English:

```text
page.frameLocator("#paymentFrame")
        ↓
"Work inside this iframe"

locator("#cardNumber")
        ↓
"Find this element inside that iframe"
```

---

# 5. Complete Example

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

await loginFrame.locator("#username").fill("Karthik");

await loginFrame.locator("#password").fill("Password123");

await loginFrame
    .getByRole("button", { name: "Login" })
    .click();
```

---

# 6. You Can Use All Normal Playwright Locators Inside a Frame

Once you have:

```typescript
const frame = page.frameLocator("#loginFrame");
```

you can use:

```typescript
frame.locator()
frame.getByRole()
frame.getByText()
frame.getByLabel()
frame.getByPlaceholder()
frame.getByTestId()
```

Example:

```typescript
await frame.getByRole("textbox", {
    name: "Username"
}).fill("Karthik");
```

Another example:

```typescript
await frame.getByPlaceholder("Enter username")
    .fill("Karthik");
```

### Important:

> `frameLocator()` changes the **context**.  
> After that, your normal Playwright locator knowledge still applies.

---

# 7. Direct Chaining

You don't always need a variable.

Instead of:

```typescript
const frame = page.frameLocator("#loginFrame");

await frame
    .getByRole("button", { name: "Login" })
    .click();
```

you can write:

```typescript
await page
    .frameLocator("#loginFrame")
    .getByRole("button", { name: "Login" })
    .click();
```

Both are valid.

---

# 8. `frameLocator()` vs `locator()`

This is extremely important.

### `locator()`

```typescript
page.locator("#username")
```

Means:

> Find an element in the current page DOM.

### `frameLocator()`

```typescript
page.frameLocator("#loginFrame")
```

Means:

> Locate an iframe and create a context for interacting with elements inside it.

Mental map:

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

---

# 9. `frameLocator()` vs `frame()`

There are two important approaches.

## Approach 1 — `frameLocator()`

```typescript
const frame = page.frameLocator("#payment");

await frame.locator("#cardNumber").fill("1234");
```

This gives you a:

```text
FrameLocator
```

It is mainly used for UI interaction inside the iframe.

---

## Approach 2 — `frame()`

```typescript
const frame = page.frame({
    name: "payment"
});
```

This gives you the actual:

```text
Frame object
```

You can then work with the frame.

---

# 10. Simple Difference

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

### Easy memory trick:

> **`frameLocator()` → Locator mindset**

> **`frame()` → Frame object mindset**

---

# 11. Using `page.frame()`

Suppose:

```html
<iframe name="paymentFrame">
```

We can do:

```typescript
const frame = page.frame({
    name: "paymentFrame"
});
```

Then:

```typescript
await frame?.locator("#cardNumber").fill("1234");
```

---

# 12. Why Do We Use `?.`?

This is TypeScript optional chaining.

`page.frame()` may return:

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

could result in:

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

> If frame exists, continue. Otherwise don't try to call a method on `null`.

---

# 13. Important Difference in Return Types

Remember this:

```text
page.frameLocator()
        ↓
FrameLocator
```

Whereas:

```text
page.frame()
        ↓
Frame | null
```

This difference is useful in TypeScript and interviews.

---

# 14. Finding a Frame by Name

If iframe HTML is:

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

---

# 15. Finding a Frame by URL

Sometimes the iframe doesn't have a useful ID or name.

You can inspect all frames:

```typescript
const frames = page.frames();
```

Then find one based on URL:

```typescript
const paymentFrame = page
    .frames()
    .find(frame => frame.url().includes("payment"));
```

Then:

```typescript
await paymentFrame?.locator("#cardNumber").fill("1234");
```

Mental map:

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

# 16. `page.frames()`

```typescript
const frames = page.frames();
```

This returns the frames associated with the page.

Think:

```text
PAGE
│
├── Main Frame
├── iframe 1
├── iframe 2
└── iframe 3
```

`page.frames()` lets you inspect those frames.

---

# 17. Debugging With `page.frames()`

This is extremely useful when you don't know why Playwright can't find an iframe element.

```typescript
console.log(
    page.frames().map(frame => frame.url())
);
```

You might get:

```text
https://myapplication.com
https://payment.com/card
https://analytics.com/report
```

Now you know which frames exist.

---

# 18. Main Frame

The main page itself has a main frame.

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

---

# 19. Nested Frames

This is a very important real-world and interview scenario.

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

You need to enter Frame A first and then Frame B.

```typescript
await page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
    .getByRole("button", {
        name: "Submit"
    })
    .click();
```

### Read it like English:

> Go to Frame A → go to Frame B → find Submit → click.

---

# 20. Nested Frame Mental Map

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

### Golden rule:

> If an iframe is inside another iframe, follow the hierarchy.

---

# 21. Why This Doesn't Work for Nested Frames

Suppose:

```text
Frame A
  ↓
Frame B
  ↓
Button
```

This is wrong:

```typescript
page.frameLocator("#frameB")
```

Why?

Because `#frameB` is not directly inside the main page.

It is inside Frame A.

Correct:

```typescript
page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
```

---

# 22. Frame Inside Frame Inside Frame

Yes, it can be chained further.

```typescript
await page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
    .frameLocator("#frameC")
    .getByRole("button", {
        name: "Submit"
    })
    .click();
```

Mental map:

```text
Page
 ↓
A
 ↓
B
 ↓
C
 ↓
Button
```

---

# 23. Frame Locator With Locator Chaining

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

This is exactly the same locator-chaining concept.

---

# 24. Frame + `getByRole()`

Example:

```typescript
await page
    .frameLocator("#loginFrame")
    .getByRole("textbox", {
        name: "Username"
    })
    .fill("Karthik");
```

This is a preferred style when the accessibility information is good.

---

# 25. Frame + `getByLabel()`

```typescript
await page
    .frameLocator("#loginFrame")
    .getByLabel("Username")
    .fill("Karthik");
```

---

# 26. Frame + `getByPlaceholder()`

```typescript
await page
    .frameLocator("#loginFrame")
    .getByPlaceholder("Enter username")
    .fill("Karthik");
```

---

# 27. Frame + CSS Locator

```typescript
await page
    .frameLocator("#loginFrame")
    .locator("#username")
    .fill("Karthik");
```

---

# 28. Dynamic iframe

Sometimes the iframe is not present immediately.

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
const frame = page.frameLocator("#paymentFrame");

await frame
    .locator("#cardNumber")
    .fill("4111111111111111");
```

Playwright's locator model handles waiting for the relevant iframe/element to become available during the interaction.

You generally don't need arbitrary:

```typescript
await page.waitForTimeout(5000);
```

just because an iframe appears dynamically.

---

# 29. Frame Events

Playwright provides frame lifecycle events.

Important events:

```typescript
page.on("frameattached", ...)
page.on("framenavigated", ...)
page.on("framedetached", ...)
```

Mental map:

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

# 30. `frameattached`

Used when a frame is attached to the page.

```typescript
page.on("frameattached", frame => {
    console.log("Frame attached:", frame.url());
});
```

Useful for debugging dynamically created frames.

---

# 31. `framenavigated`

Used when a frame navigates.

```typescript
page.on("framenavigated", frame => {
    console.log("Frame navigated:", frame.url());
});
```

---

# 32. `framedetached`

Used when a frame is removed.

```typescript
page.on("framedetached", frame => {
    console.log("Frame detached:", frame.url());
});
```

---

# 33. Important Difference: `page.on()` vs `waitForEvent()`

### `page.on()`

Means:

> Keep listening for this event.

Example:

```typescript
page.on("frameattached", frame => {
    console.log(frame.url());
});
```

### `page.waitForEvent()`

Means:

> Wait for one occurrence of this event.

Use events when you actually need event-driven synchronization or debugging. Don't add them just because an iframe exists.

---

# 34. Frame URL

A `Frame` object has a URL.

```typescript
console.log(frame.url());
```

Useful for identifying a frame.

Example:

```typescript
const paymentFrame = page
    .frames()
    .find(frame =>
        frame.url().includes("payment")
    );
```

---

# 35. Inspect All Frame URLs

Very useful debugging technique:

```typescript
for (const frame of page.frames()) {
    console.log(frame.url());
}
```

This gives you a simple picture of the frame structure.

---

# 36. Cross-Origin iframe

Example:

```text
Application
     │
     └── iframe
            │
            └── payment-provider.com
```

The iframe may belong to another domain.

Do not assume:

> Different domain = Playwright cannot automate it.

Playwright provides frame APIs for interacting with frame content where browser automation permits it.

The important practical point is:

```text
Main application
       ↓
iframe
       ↓
Use frameLocator()
```

---

# 37. Real-World Payment Example

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

# 38. Real-World Rich Text Editor

Some applications use iframe-based editors.

Example:

```text
Application
    │
    └── iframe
          │
          └── Editor body
```

Then:

```typescript
await page
    .frameLocator("#editorFrame")
    .locator("body")
    .fill("This is my content");
```

The exact locator depends on the editor implementation.

---

# 39. Real-World Embedded Reports

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

You may need:

```typescript
await page
    .frameLocator("#reportFrame")
    .getByRole("button", {
        name: "Filters"
    })
    .click();
```

Exact locators depend on the application.

---

# 40. Frame Handling With Variables

For complex tests, a variable often improves readability.

```typescript
const paymentFrame =
    page.frameLocator("#paymentFrame");

await paymentFrame
    .getByRole("textbox", { name: "Card Number" })
    .fill("1234");

await paymentFrame
    .getByRole("textbox", { name: "Expiry" })
    .fill("12/30");
```

This is often easier to maintain.

---

# 41. When to Use `frameLocator()` vs `page.frames()`

## Normal UI automation

Prefer:

```typescript
page.frameLocator()
```

Example:

```typescript
await page
    .frameLocator("#payment")
    .getByRole("textbox")
    .fill("1234");
```

## Need to inspect/find a frame programmatically

Use:

```typescript
page.frames()
```

Example:

```typescript
const frame = page
    .frames()
    .find(f => f.url().includes("payment"));
```

## Need actual Frame object

Use:

```typescript
page.frame()
```

---

# 42. Practical Decision Tree

```text
Is the element inside an iframe?
             │
          YES
             ↓
Do I just need to interact with it?
             │
          YES
             ↓
       frameLocator()
```

If you need to inspect/find frames:

```text
page.frames()
```

If you know the frame by name:

```text
page.frame()
```

If you need to find it based on URL:

```text
page.frames()
   ↓
find()
```

---

# 43. Common Mistake #1

### Wrong

```typescript
await page.locator("#cardNumber").fill("1234");
```

### Why?

`#cardNumber` is inside iframe.

### Correct

```typescript
await page
    .frameLocator("#paymentFrame")
    .locator("#cardNumber")
    .fill("1234");
```

---

# 44. Common Mistake #2

### Wrong

```typescript
page.locator("#paymentFrame")
    .locator("#cardNumber");
```

This does not mean:

> "Go inside the iframe."

`locator()` finds DOM elements.

Use:

```typescript
page.frameLocator("#paymentFrame")
```

---

# 45. Common Mistake #3

Trying to access a nested iframe directly.

### Structure

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

# 46. Common Mistake #4

Using `page.frames()` for every simple interaction.

You might write:

```typescript
const frames = page.frames();

const frame = frames[1];

await frame?.locator("#username").fill("Karthik");
```

This may work, but relying on:

```typescript
frames[1]
```

is fragile because frame order can change.

Prefer a stable identification method.

For normal UI interaction:

```typescript
page.frameLocator("#loginFrame")
```

---

# 47. Common Mistake #5

Using arbitrary sleeps.

Avoid:

```typescript
await page.waitForTimeout(5000);
```

Prefer Playwright's locator-based synchronization.

---

# 48. Common Mistake #6

Confusing FrameLocator and Frame.

```text
frameLocator()
       ↓
FrameLocator

frame()
       ↓
Frame
```

They are not the same object.

---

# 49. Common Mistake #7

Assuming iframe always has an ID.

The iframe may be identified using:

- ID
- name
- CSS
- other attributes
- URL
- relationship to another frame

Example:

```typescript
page.frameLocator('iframe[name="payment"]')
```

---

# 50. Interview Question: What is an iframe?

### Interview-ready answer

> An iframe is a separate HTML document embedded inside a web page. Because it has its own DOM context, elements inside the iframe cannot normally be located directly from the parent page locator. In Playwright, I generally use `frameLocator()` to interact with elements inside an iframe.

---

# 51. Interview Question: How do you handle iframes in Playwright?

### Answer

> For normal UI interaction, I use `page.frameLocator()` and then use standard Playwright locators such as `getByRole()`, `getByLabel()` or `locator()` to interact with elements inside the frame. If I need the actual `Frame` object, I use `page.frame()` or inspect frames through `page.frames()`.

---

# 52. Interview Question: Difference Between `frameLocator()` and `frame()`?

### Answer

> `frameLocator()` returns a `FrameLocator` and is convenient for locating and interacting with elements inside an iframe. `page.frame()` returns the actual `Frame` object, which is useful when I need frame-level APIs or programmatic frame handling.

---

# 53. Interview Question: How Do You Handle Nested Frames?

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

# 54. Interview Question: How Do You Find a Frame When It Has No ID?

Possible approaches include:

```typescript
page.frameLocator("iframe[name='payment']")
```

or another suitable iframe selector.

If programmatic identification is needed:

```typescript
const frame = page
    .frames()
    .find(f => f.url().includes("payment"));
```

---

# 55. Interview Question: How Do You Debug an iframe Problem?

Practical debugging flow:

```text
1. Confirm iframe exists
        ↓
2. Inspect iframe selector
        ↓
3. Check whether element is actually inside iframe
        ↓
4. Check nested frame hierarchy
        ↓
5. Inspect page.frames()
        ↓
6. Check frame URLs
        ↓
7. Verify locator inside frame
```

Useful debugging code:

```typescript
console.log(
    page.frames().map(frame => frame.url())
);
```

---

# 56. Senior SDET Thinking

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
Then build the frame hierarchy.
```

This is a better debugging mindset.

---

# 57. Complete Frame Handling Mental Map

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

# 58. One-Page Memory Sheet

## Main page

```typescript
page.locator("#username")
```

## Inside iframe

```typescript
page
    .frameLocator("#loginFrame")
    .locator("#username")
```

## Using role inside iframe

```typescript
page
    .frameLocator("#loginFrame")
    .getByRole("button", { name: "Login" })
    .click();
```

## Nested iframe

```typescript
page
    .frameLocator("#frameA")
    .frameLocator("#frameB")
    .getByRole("button")
    .click();
```

## Get Frame object by name

```typescript
const frame = page.frame({
    name: "paymentFrame"
});
```

## Get all frames

```typescript
const frames = page.frames();
```

## Find frame by URL

```typescript
const frame = page
    .frames()
    .find(f => f.url().includes("payment"));
```

## Main frame

```typescript
page.mainFrame()
```

## Frame events

```typescript
page.on("frameattached", ...)
page.on("framenavigated", ...)
page.on("framedetached", ...)
```

---

# 59. The 4 Things You MUST Remember

### 1. `page.locator()`

```typescript
page.locator()
```

➡️ Find element in the current page/frame context.

### 2. `page.frameLocator()`

```typescript
page.frameLocator()
```

➡️ Enter an iframe context for UI interaction.

### 3. `page.frame()`

```typescript
page.frame()
```

➡️ Get the actual `Frame` object.

### 4. `page.frames()`

```typescript
page.frames()
```

➡️ Get/inspect all frames.

---

# 60. Final Mental Model

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

If frames are nested:

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

---

# 61. Final Rule

> **Don't think "How do I find the element?" first.**

Think:

> **"Where does the element live?"**

Then:

```text
Main page?
    ↓
page.locator()

Iframe?
    ↓
frameLocator()

Nested iframe?
    ↓
frameLocator().frameLocator()

Need actual Frame object?
    ↓
frame()

Need to inspect all frames?
    ↓
frames()
```

This is the core mental model for **Playwright Frame Handling**.
