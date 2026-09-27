# Playwright Window & Tab Handling

> **Purpose:** Practical learning notes, revision, and blog-ready
> reference for Playwright Window/Tab Handling.

------------------------------------------------------------------------

## 1. The One Mental Model to Remember

``` text
Browser
   ↓
BrowserContext = Session
   ↓
Pages = Tabs / Windows
```

The most important idea:

> **In Playwright, a browser tab or popup window is represented by a
> `Page` object.**

So Playwright does not normally use Selenium-style:

``` text
switchTo().window()
```

Instead:

``` text
New Tab / Window
      ↓
New Page object
      ↓
Capture the Page
      ↓
Use that Page directly
```

------------------------------------------------------------------------

# 2. Browser → BrowserContext → Page

A single Browser can have multiple BrowserContexts.

``` text
Browser
   │
   ├── Context 1 = Session 1
   │      ├── Page 1
   │      └── Page 2
   │
   └── Context 2 = Session 2
          └── Page 1
```

### Simple memory

-   **Browser = actual browser**
-   **BrowserContext = separate session / isolated browser environment**
-   **Page = tab / window**

A BrowserContext can contain multiple Pages.

------------------------------------------------------------------------

# 3. Scenario: A Page Opens a New Tab

Suppose:

``` text
Current Page
     ↓
Click "Open Customer"
     ↓
New Tab opens
```

Use:

``` ts
const newPagePromise = page.waitForEvent('popup');

await page.getByRole('link', {
    name: 'Open Customer'
}).click();

const newPage = await newPagePromise;
```

Now:

``` text
page
 ↓
Old tab

newPage
 ↓
New tab
```

You can directly interact with the new Page:

``` ts
await newPage.getByRole('heading', {
    name: 'Customer Details'
}).isVisible();
```

## Golden Rule

> **WAIT FOR THE EVENT → PERFORM THE ACTION → GET THE NEW PAGE → USE THE
> PAGE**

Do not start waiting after the action.

### Wrong

``` ts
await page.getByRole('link', {
    name: 'Open Customer'
}).click();

const newPage = await page.waitForEvent('popup');
```

### Correct

``` ts
const newPagePromise = page.waitForEvent('popup');

await page.getByRole('link', {
    name: 'Open Customer'
}).click();

const newPage = await newPagePromise;
```

The event listener must be registered before the action that triggers
the event.

------------------------------------------------------------------------

# 4. `page.waitForEvent('popup')`

Use this when the **current Page opens another Page**.

``` text
Page A
   │
   └──── opens ────→ Page B
                         ↑
                      popup
```

Example:

``` ts
const popupPromise = page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Open Report'
}).click();

const popup = await popupPromise;

console.log(popup.url());
```

### Memory trick

> **PAGE → POPUP**

------------------------------------------------------------------------

# 5. `context.waitForEvent('page')`

Use this when you want to detect a **new Page created anywhere inside
the BrowserContext**.

``` text
BrowserContext
      │
      ├── Page 1
      ├── Page 2
      └── NEW Page
             ↓
       context 'page' event
```

Example:

``` ts
const context = page.context();

const newPagePromise = context.waitForEvent('page');

await page.getByRole('link', {
    name: 'Open Customer'
}).click();

const newPage = await newPagePromise;

console.log(newPage.url());
```

### Memory trick

> **CONTEXT → PAGE**

------------------------------------------------------------------------

# 6. `popup` vs `page`

This is one of the most important distinctions.

  -----------------------------------------------------------------------
  API                                 Meaning
  ----------------------------------- -----------------------------------
  `page.waitForEvent('popup')`        This particular Page opens another
                                      Page

  `context.waitForEvent('page')`      A new Page is created anywhere in
                                      this BrowserContext
  -----------------------------------------------------------------------

### Visual memory

``` text
page.waitForEvent('popup')

Page 1
  │
  └────→ Page 2
```

``` text
context.waitForEvent('page')

BrowserContext
  │
  ├── Page 1
  ├── Page 2
  └── NEW Page ← detected
```

------------------------------------------------------------------------

# 7. `waitForEvent()` vs `on()`

This is another important concept.

## `waitForEvent()`

Use when you are expecting a specific event, usually as part of a
specific action.

``` ts
const popupPromise = page.waitForEvent('popup');

await page.getByText('Open').click();

const popup = await popupPromise;
```

Think:

> **I am expecting an event now.**

------------------------------------------------------------------------

## `on()`

Use when you want to keep listening for events.

``` ts
page.on('popup', popup => {
    console.log('Popup opened');
});
```

Think:

> **Keep listening.**

### Memory

``` text
waitForEvent()
     ↓
Wait for an expected event

on()
     ↓
Keep listening for events
```

------------------------------------------------------------------------

# 8. `Promise.all()` Pattern

A common real-project pattern is:

``` ts
const [newPage] = await Promise.all([
    page.waitForEvent('popup'),
    page.getByRole('link', {
        name: 'Open Customer'
    }).click()
]);
```

This works because the event wait and the action are started together.

Conceptually:

``` text
Start waiting for popup
        +
Perform click
        ↓
Wait for both
        ↓
Get new Page
```

### Important

Do not memorize `Promise.all()` as the window-handling concept.

The actual concept is:

> **Register the event wait before triggering the action.**

------------------------------------------------------------------------

# 9. Getting All Currently Open Pages

If you want all currently existing Pages in a BrowserContext:

``` ts
const pages = context.pages();
```

Example:

``` ts
const openedPages = context.pages();

console.log(openedPages.length);

for (const pg of openedPages) {
    console.log(pg.url());
}
```

Mental model:

``` text
BrowserContext
   │
   ├── Page 1
   ├── Page 2
   └── Page 3

context.pages()
       ↓
[Page 1, Page 2, Page 3]
```

### Important distinction

``` text
context.waitForEvent('page')
        ↓
Wait/capture a new Page

context.pages()
        ↓
Get Pages that currently exist
```

------------------------------------------------------------------------

# 10. Don't Blindly Use `pages()[1]`

You may see:

``` ts
const pages = context.pages();
const newPage = pages[1];
```

This can work in a simple scenario, but it is not a reliable way to
identify the Page created by a particular action.

For example:

``` text
Page 1 = original page
Page 2 = some other page
Page 3 = the page you expected
```

So:

``` ts
pages[1]
```

would not necessarily be the Page you wanted.

If an action is expected to create a new Page, explicitly wait for that
event:

``` ts
const newPagePromise = context.waitForEvent('page');

await page.getByText('Open').click();

const newPage = await newPagePromise;
```

### Memory

> **`context.pages()` = see all Pages.**\
> **`waitForEvent()` = capture the Page created by an expected event.**

------------------------------------------------------------------------

# 11. Do You Need `bringToFront()`?

Usually, **no**.

You can interact directly with the Page object:

``` ts
await newPage.getByRole('button', {
    name: 'Download'
}).click();
```

You do not normally need:

``` ts
await newPage.bringToFront();
```

`bringToFront()` is only useful when you specifically want that Page
brought to the front of the browser UI.

### Memory

> **Capture the Page → use the Page.**

------------------------------------------------------------------------

# 12. Do You Need `waitForLoadState()` Every Time?

No.

This is valid:

``` ts
const newPage = await newPagePromise;

await newPage.waitForLoadState();
```

But don't make it a mandatory step after every new Page.

Playwright's locator actions and assertions already perform automatic
waiting for many UI conditions.

For example:

``` ts
await newPage.getByRole('button', {
    name: 'Submit'
}).click();
```

You do not automatically need:

``` ts
await newPage.waitForLoadState();
```

Use load-state waits when you have a specific reason to wait for a
navigation/load milestone.

------------------------------------------------------------------------

# 13. No Selenium-Style Window Switching

### Selenium mental model

``` text
Window Handle
      ↓
switchTo().window()
      ↓
Work with window
```

### Playwright mental model

``` text
New Tab / Window
      ↓
New Page
      ↓
Capture Page
      ↓
Work with Page
```

There is normally no:

``` ts
driver.switchTo().window(...)
```

equivalent that you need to use.

------------------------------------------------------------------------

# 14. Real Project Example

Imagine a Salesforce application:

``` text
Account Page
    ↓
Click "View Account"
    ↓
New Account Details Tab
```

Test:

``` ts
const accountPagePromise = page.waitForEvent('popup');

await page.getByRole('link', {
    name: 'View Account'
}).click();

const accountPage = await accountPagePromise;

await expect(
    accountPage.getByRole('heading', {
        name: 'Account Details'
    })
).toBeVisible();
```

Here:

``` text
page
   ↓
Account/list page

accountPage
   ↓
Account details tab
```

No window switching is required.

------------------------------------------------------------------------

# 15. Real Practice Example

A practice page can open multiple tabs:

``` ts
await page.goto('https://www.playwrightautomation.com/practice.html');

const context = page.context();

await page.getByRole('button', {
    name: 'Open Playwright Docs'
}).click();

await page.getByRole('button', {
    name: 'Open One Random Tab'
}).click();

const openedPages = context.pages();

console.log(openedPages.length);

for (const pg of openedPages) {
    console.log(pg.url());
}
```

This demonstrates:

``` text
BrowserContext
      ↓
Multiple Pages
      ↓
context.pages()
      ↓
Get all currently open Pages
```

------------------------------------------------------------------------

# 16. Common Mistakes

## Mistake 1 --- Thinking in Selenium

``` text
❌ Window Handle → Switch
```

Instead:

``` text
✅ New Tab → New Page → Use Page
```

------------------------------------------------------------------------

## Mistake 2 --- Waiting after clicking

❌

``` ts
await page.getByText('Open').click();

const popup = await page.waitForEvent('popup');
```

✅

``` ts
const popupPromise = page.waitForEvent('popup');

await page.getByText('Open').click();

const popup = await popupPromise;
```

------------------------------------------------------------------------

## Mistake 3 --- Using the old `page`

If a new Page opens:

``` ts
const newPage = await popupPromise;
```

Then interact with:

``` ts
newPage
```

not automatically:

``` ts
page
```

------------------------------------------------------------------------

## Mistake 4 --- Assuming `pages()[1]` is always the new Page

Avoid relying on a hard-coded index when you know an action created a
Page.

------------------------------------------------------------------------

## Mistake 5 --- Thinking `bringToFront()` is mandatory

It isn't required just to interact with a Page.

------------------------------------------------------------------------

## Mistake 6 --- Adding `waitForLoadState()` everywhere

Don't add waits without a reason.

------------------------------------------------------------------------

# 17. API Cheat Sheet

  --------------------------------------------------------------------------------
  API                              What it means           Priority
  -------------------------------- ----------------------- -----------------------
  `page.waitForEvent('popup')`     Current Page opens      🔴 MUST KNOW
                                   another Page            

  `context.waitForEvent('page')`   New Page created in the 🔴 MUST KNOW
                                   Context                 

  `context.pages()`                Get currently open      🔴 MUST KNOW
                                   Pages                   

  `page.on('popup', ...)`          Keep listening for      🔴 MUST KNOW
                                   popups                  

  `page.bringToFront()`            Bring Page to browser   🟡 SHOULD KNOW
                                   front                   

  `page.waitForLoadState()`        Wait for a load state   🟡 SHOULD KNOW
                                   when needed             
  --------------------------------------------------------------------------------

------------------------------------------------------------------------

# 18. Interview Questions

### Q1. How do you handle a new tab in Playwright?

**Answer:**

> In Playwright, a tab or popup window is represented by a `Page`. When
> an action opens a new Page, I wait for the appropriate event, capture
> the Page, and interact with that Page directly.

------------------------------------------------------------------------

### Q2. Does Playwright have Selenium-style window handles?

**Answer:**

> Playwright does not use Selenium-style window handles for normal tab
> handling. Each tab or popup is represented by a `Page` object.

------------------------------------------------------------------------

### Q3. What is the difference between `page.waitForEvent('popup')` and `context.waitForEvent('page')`?

**Answer:**

> `page.waitForEvent('popup')` is associated with a specific Page
> opening another Page, while `context.waitForEvent('page')` detects a
> new Page created anywhere in the BrowserContext.

------------------------------------------------------------------------

### Q4. How do you get all currently open tabs?

``` ts
const pages = context.pages();
```

------------------------------------------------------------------------

### Q5. What is the difference between `waitForEvent()` and `on()`?

**Answer:**

> `waitForEvent()` is useful when I expect a particular event and need
> to wait for it. `on()` registers a listener that keeps listening for
> that event.

------------------------------------------------------------------------

### Q6. Do you need `bringToFront()` before interacting with another tab?

**Answer:**

> No. Once I have the Page object, I can interact with it directly.
> `bringToFront()` is only needed when I specifically want that Page
> brought to the front of the browser UI.

------------------------------------------------------------------------

# 19. Final Mental Map

``` text
                         BROWSER
                            │
                     BROWSER CONTEXT
                       = SESSION
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           PAGE 1        PAGE 2        PAGE 3
           Tab 1         Tab 2         Tab 3
```

### New Page handling

``` text
Page opens another Page
          ↓
page.waitForEvent('popup')
```

### Any new Page in Context

``` text
New Page in BrowserContext
          ↓
context.waitForEvent('page')
```

### All existing Pages

``` text
context.pages()
```

### Continuous listening

``` text
page.on('popup', ...)
```

### No switching

``` text
❌ switchTo().window()

✅ Capture Page → Use Page
```

------------------------------------------------------------------------

# 20. What You Actually Need to Remember

If you forget everything else, remember these:

``` text
Browser
   ↓
Context = Session
   ↓
Page = Tab / Window
```

``` text
Page opens new Page
→ page.waitForEvent('popup')
```

``` text
Context detects new Page
→ context.waitForEvent('page')
```

``` text
Get all existing Pages
→ context.pages()
```

``` text
Keep listening
→ page.on('popup', ...)
```

And the golden rule:

> **WAIT → ACTION → CAPTURE PAGE → USE PAGE**

### One-line interview memory

> **Playwright doesn't switch windows; it works with Page objects.**
