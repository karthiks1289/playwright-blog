# Playwright `expect` --- Master Notes & Revision Guide

> **Purpose:** Practical Playwright `expect` notes for real projects +
> SDET interviews.
>
> **Learning principle:** Do not memorize the entire matcher library.
> Learn to identify **what you are proving** and choose the appropriate
> assertion.

------------------------------------------------------------------------

## 1. The One Mental Model to Remember

``` text
ACTION  = DO something
QUERY   = ASK for information
EXPECT  = PROVE something
```

Examples:

``` ts
await button.click();                 // ACTION
const count = await rows.count();     // QUERY
await expect(rows).toHaveCount(5);    // EXPECT
```

### The master rule

> **First identify what you are validating. Then choose the assertion.**

------------------------------------------------------------------------

# 2. What Goes Inside `expect()`?

Playwright's `expect()` can assert different kinds of things:

``` text
expect(locator)  → web element assertion
expect(page)     → page assertion
expect(response) → API response assertion
expect(value)    → JavaScript/value assertion
```

Examples:

``` ts
await expect(button).toBeVisible();

await expect(page).toHaveURL(/dashboard/);

await expect(response).toBeOK();

expect(count).toBe(5);
```

------------------------------------------------------------------------

# 3. Web-First Assertions --- MOST IMPORTANT

Web-first assertions are designed to wait/retry until the expected web
condition is satisfied or the assertion timeout is reached.

Example:

``` ts
await expect(message).toBeVisible();
```

If the message appears after a short delay, Playwright keeps checking
instead of checking only once.

### Mental model

``` text
expect(locator)
      ↓
check condition
      ↓
not ready?
      ↓
retry
      ↓
pass OR timeout
```

### Important

Web-first assertions are asynchronous.

``` ts
await expect(locator).toBeVisible();
```

Not:

``` ts
expect(locator).toBeVisible();
```

------------------------------------------------------------------------

# 4. `expect()` vs Query Methods

## Query

``` ts
const visible = await button.isVisible();
```

This asks:

> "What is the state right now?"

It returns a value.

## Assertion

``` ts
await expect(button).toBeVisible();
```

This asks:

> "Should this condition become true?"

It waits/retries.

### Remember forever

``` text
isVisible()    → ASK
toBeVisible()  → PROVE
```

Do not replace a web-first assertion with a query + immediate assertion
when the condition can change asynchronously.

------------------------------------------------------------------------

# 5. Element State Assertions

## 🔴 MUST KNOW

### Visible

``` ts
await expect(button).toBeVisible();
```

### Enabled

``` ts
await expect(button).toBeEnabled();
```

### Disabled

``` ts
await expect(button).toBeDisabled();
```

### Checked

``` ts
await expect(checkbox).toBeChecked();
```

### Not checked

``` ts
await expect(checkbox).not.toBeChecked();
```

### Editable

``` ts
await expect(input).toBeEditable();
```

### Focused

``` ts
await expect(input).toBeFocused();
```

### Hidden

``` ts
await expect(message).toBeHidden();
```

Use the assertion whose meaning matches the requirement.

------------------------------------------------------------------------

# 6. `.not` --- Reverse the Expectation

``` ts
await expect(error).not.toBeVisible();
```

Mental model:

``` text
normal assertion → I expect THIS
.not             → I expect the OPPOSITE
```

Examples:

``` ts
await expect(button).not.toBeDisabled();

await expect(checkbox).not.toBeChecked();

await expect(message).not.toContainText("Error");

await expect(input).not.toHaveValue("Karthik");
```

------------------------------------------------------------------------

# 7. Text Assertions

## `toHaveText()`

Use when validating the expected text/content of an element.

``` ts
await expect(header).toHaveText("Form Controls");
```

Can also use a regular expression for dynamic text:

``` ts
await expect(message)
    .toHaveText(/Order ORD-\d+ created successfully/);
```

## `toContainText()`

Use when the expected text should be contained in the element's text.

``` ts
await expect(message).toContainText("successfully");
```

### Remember

``` text
toHaveText()     → expected text/content
toContainText()  → expected text is contained
```

Do not choose between them based only on "strict vs partial"
memorization. Think about the actual requirement and the element's
content.

------------------------------------------------------------------------

# 8. Dynamic Text

Bad approach when the dynamic part changes:

``` ts
await expect(message)
    .toHaveText("Order ORD-12345 created successfully");
```

Better:

``` ts
await expect(message)
    .toHaveText(/Order ORD-\d+ created successfully/);
```

### Technique

``` text
Static text     → exact expected text
Dynamic text    → RegExp/pattern
```

------------------------------------------------------------------------

# 9. Form/Input Assertions

## `toHaveValue()`

Checks the value of a form control.

``` ts
await input.fill("Karthik");

await expect(input).toHaveValue("Karthik");
```

Do not confuse:

``` text
toHaveText()  → element text/content
toHaveValue() → form control value
```

------------------------------------------------------------------------

# 10. Attribute, Class and CSS Assertions

## `toHaveAttribute()`

``` ts
await expect(input).toHaveAttribute("id", "name");
```

Use when the actual HTML attribute is part of what you need to verify.

## `toHaveClass()`

``` ts
await expect(button).toHaveClass(/active/);
```

Use when the class represents meaningful UI state.

## `toHaveCSS()`

``` ts
await expect(input).toHaveCSS("font-size", "0.875rem");
```

Use when the CSS property itself is a real requirement.

### Senior SDET rule

> Do not assert implementation details just because you can.

If the requirement is "button is usable", prefer a meaningful behavioral
assertion such as:

``` ts
await expect(button).toBeEnabled();
```

rather than testing unrelated CSS/class details.

------------------------------------------------------------------------

# 11. Collections and `toHaveCount()`

``` ts
const rows = page.locator("table tr");

await expect(rows).toHaveCount(5);
```

### Critical mental model

``` text
page.locator("table tr")
        ↓
ONE Locator OBJECT
        ↓
that Locator may match MANY elements
        ↓
row1, row2, row3, ...
```

It does NOT mean the Locator itself is one DOM element.

### `.all()`

``` ts
const rows = await page.locator("table tr").all();
```

Now you get an array of individual Locator objects:

``` text
[
  Locator(row1),
  Locator(row2),
  Locator(row3)
]
```

### Remember forever

``` text
locator() → ONE Locator object that can represent one or many matches
all()     → array of individual Locator objects for current matches
```

------------------------------------------------------------------------

# 12. `count()` vs `toHaveCount()`

## Query

``` ts
const count = await rows.count();
```

Gets the current number.

## Assertion

``` ts
await expect(rows).toHaveCount(5);
```

Asserts the expected count and uses assertion waiting/retry behavior.

### Decision rule

``` text
Need the number for logic?
→ count()

Need to prove the number?
→ toHaveCount()
```

------------------------------------------------------------------------

# 13. Page Assertions

## URL

``` ts
await expect(page).toHaveURL(/dashboard/);
```

## Title

``` ts
await expect(page).toHaveTitle(/Playwright/);
```

### Mental model

``` text
Element → expect(locator)
Page    → expect(page)
```

Compare:

``` ts
page.url()
```

with:

``` ts
await expect(page).toHaveURL(...);
```

`page.url()` retrieves the current URL; `toHaveURL()` is a page
assertion that can wait/retry.

------------------------------------------------------------------------

# 14. JavaScript / Value Assertions

When you already have a JavaScript value, use generic assertions.

``` ts
const count = 5;

expect(count).toBe(5);
```

These are normally immediate checks, so they do not need `await`.

## 🔴 MUST KNOW

### `toBe()`

``` ts
expect(count).toBe(5);
```

Use for exact primitive values and identity/reference comparisons.

### `toEqual()`

``` ts
expect(actual).toEqual(expected);
```

Use for deep equality of objects/arrays.

### `toContain()`

``` ts
expect(ids).toContain(102);
```

Use when a string contains a substring or an array/set contains an
element.

### `toMatch()`

``` ts
expect(id).toMatch(/^EMP-\d+$/);
```

Use for a string pattern/RegExp.

### Quick memory table

``` text
toBe()       → exact value/reference
toEqual()    → contents / deep equality
toContain()  → contains
toMatch()    → pattern
```

------------------------------------------------------------------------

# 15. `toBe()` vs `toEqual()` --- Important Interview Topic

``` ts
expect(actual).toBe(expected);
```

`toBe()` compares using `Object.is()`.

For objects:

``` ts
expect(actual).toEqual(expected);
```

checks their contents using deep equality.

Example:

``` ts
const actual = {
  id: 101,
  role: "QA"
};

expect(actual).toEqual({
  id: 101,
  role: "QA"
});
```

### Practical rule

``` text
Primitive/exact value → toBe()
Object/array contents → toEqual()
```

For complete strict structural matching, Playwright also provides
`toStrictEqual()`.

------------------------------------------------------------------------

# 16. API Response Assertions

Suppose:

``` ts
const response = await request.get("/employees/101");
```

## Successful response

``` ts
await expect(response).toBeOK();
```

`toBeOK()` checks that the HTTP response status is in the `200..299`
range.

## Exact status

``` ts
expect(response.status()).toBe(200);
```

Use this when the requirement specifically says HTTP 200.

### Remember

``` text
toBeOK()             → successful 2xx response
response.status()    → get exact status number
```

------------------------------------------------------------------------

# 17. API Response Body

``` ts
const body = await response.json();

expect(body.id).toBe(101);
expect(body.name).toBe("Karthik");
expect(body.role).toBe("QA");
```

After `response.json()`:

``` text
API response
     ↓
JavaScript object/value
     ↓
generic expect assertions
```

This connects API testing with the value assertions from Lesson 11.

------------------------------------------------------------------------

# 18. Full Object vs Specific Fields

If the requirement is to validate specific fields:

``` ts
expect(body.id).toBe(101);
expect(body.role).toBe("QA");
```

If the requirement is to validate the whole expected object:

``` ts
expect(body).toEqual({
  id: 101,
  name: "Karthik",
  role: "QA"
});
```

### Practical rule

> Validate the fields that represent the business requirement. Do not
> compare the entire response blindly when it contains dynamic fields.

------------------------------------------------------------------------

# 19. `expect.soft()`

Normal:

``` ts
await expect(revenue).toBeVisible();
await expect(orders).toBeVisible();
```

A failed normal assertion terminates normal test execution at that
point.

Soft:

``` ts
await expect.soft(revenue).toBeVisible();
await expect.soft(orders).toBeVisible();
await expect.soft(users).toBeVisible();
```

A failed soft assertion does not immediately terminate test execution;
it records the failure and the test is still marked failed.

### Mental model

``` text
expect()
   ↓
FAIL → stop

expect.soft()
   ↓
FAIL → record → continue
```

### Use soft assertions for

Independent validations:

``` text
Dashboard widgets
Multiple independent fields
Multiple independent UI checks
```

### Do NOT use soft assertions for critical prerequisites

Example:

``` text
Login fails
   ↓
Dashboard validations are meaningless
```

Use a normal assertion for the critical prerequisite.

------------------------------------------------------------------------

# 20. `expect.poll()`

Use `expect.poll()` when a value can change over time and you need to
repeatedly retrieve it.

Example:

``` ts
await expect.poll(async () => {
  const response = await request.get("/jobs/101");
  const body = await response.json();

  return body.status;
}).toBe("Completed");
```

Mental model:

``` text
function
  ↓
get value
  ↓
assert
  ↓
not ready?
  ↓
get value again
  ↓
repeat
```

### Use cases

-   API status changes
-   backend jobs
-   eventually consistent systems
-   external processing status

### Key rule

> **Changing JavaScript/API value → consider `expect.poll()`.**

------------------------------------------------------------------------

# 21. `expect.poll()` vs Web-First Assertion

If the thing changing is a web element:

``` ts
await expect(message).toBeVisible();
```

Do NOT unnecessarily do:

``` ts
await expect.poll(() => message.isVisible()).toBe(true);
```

The web-first assertion already handles the web condition.

### Remember

``` text
Web element condition
→ web-first assertion

Changing value/API condition
→ expect.poll()
```

------------------------------------------------------------------------

# 22. `expect().toPass()`

Use `toPass()` when you need to retry a whole callback containing
multiple operations/assertions.

``` ts
await expect(async () => {
  const response = await request.get("/job/101");

  expect(response.status()).toBe(200);

  const body = await response.json();

  expect(body.status).toBe("Completed");
}).toPass();
```

Mental model:

``` text
poll()
→ repeatedly check ONE returned value

toPass()
→ repeatedly execute the WHOLE callback
```

### Key rule

``` text
One changing value
→ poll()

Multiple related checks/operations that must retry together
→ toPass()
```

------------------------------------------------------------------------

# 23. The Four Special Tools --- Never Confuse Them

``` text
expect()
      ↓
normal assertion

expect.soft()
      ↓
assert + record failure + continue

expect.poll()
      ↓
repeatedly get/check a changing value

expect(...).toPass()
      ↓
retry a whole callback
```

### Memory sentence

> **SOFT continues. POLL rechecks a value. PASS retries a block.**

------------------------------------------------------------------------

# 24. Timeout Mental Model

Think in four levels:

``` text
TEST
 ↓
whole test

ACTION
 ↓
click/fill/check/etc.

NAVIGATION
 ↓
goto/navigation

EXPECT
 ↓
one assertion
```

Example assertion timeout:

``` ts
await expect(message).toBeVisible({
  timeout: 10000
});
```

This gives that assertion up to 10 seconds.

### Important

Do not solve every timeout problem by increasing the timeout.

First ask:

``` text
What is actually timing out?
```

Then fix that category.

------------------------------------------------------------------------

# 25. Don't Use `waitForTimeout()` as Your Main Synchronization Strategy

Avoid:

``` ts
await page.waitForTimeout(5000);
await expect(message).toBeVisible();
```

when the real requirement is simply that the message eventually appears.

Prefer:

``` ts
await expect(message).toBeVisible();
```

Why?

``` text
waitForTimeout()
→ guesses a duration

web-first assertion
→ waits for the actual condition
```

------------------------------------------------------------------------

# 26. Real-Project Assertion Patterns

## Login

``` ts
await loginButton.click();

await expect(page).toHaveURL(/dashboard/);
await expect(
  page.getByRole("heading", { name: "Dashboard" })
).toBeVisible();
```

## Form submission

``` ts
await submitButton.click();

await expect(successMessage).toBeVisible();
```

## Dynamic order message

``` ts
await expect(message)
  .toHaveText(/Order ORD-\d+ created successfully/);
```

## Web table

``` ts
const row = page.locator("table tr").filter({
  hasText: "Karthik"
});

await expect(row).toHaveCount(1);
await expect(row).toContainText("Karthik");
```

## Negative validation

``` ts
await expect(errorMessage).not.toBeVisible();
```

## Multiple independent dashboard checks

``` ts
await expect.soft(revenue).toBeVisible();
await expect.soft(orders).toBeVisible();
await expect.soft(users).toBeVisible();
```

## API + UI

``` text
Create through API
      ↓
Validate API response
      ↓
Open UI
      ↓
Search created record
      ↓
Validate UI
```

This is a strong real-project pattern because it validates the business
flow at more than one layer.

------------------------------------------------------------------------

# 27. The Assertion Decision Tree

When you get stuck in an interview, use this:

``` text
WHAT AM I PROVING?
        |
        +-- Web element state?
        |      → toBeVisible()
        |      → toBeEnabled()
        |      → toBeChecked()
        |      → toBeEditable()
        |      → toBeFocused()
        |
        +-- Element text?
        |      → toHaveText()
        |      → toContainText()
        |
        +-- Input value?
        |      → toHaveValue()
        |
        +-- Attribute/class/CSS?
        |      → toHaveAttribute()
        |      → toHaveClass()
        |      → toHaveCSS()
        |
        +-- Number of matching elements?
        |      → toHaveCount()
        |
        +-- Page?
        |      → toHaveURL()
        |      → toHaveTitle()
        |
        +-- API response?
        |      → toBeOK()
        |      → response.status()
        |
        +-- JavaScript value?
        |      → toBe()
        |      → toEqual()
        |      → toContain()
        |      → toMatch()
        |
        +-- Opposite condition?
        |      → .not
        |
        +-- Independent validations?
        |      → expect.soft()
        |
        +-- Changing value?
        |      → expect.poll()
        |
        +-- Multiple checks retry together?
               → toPass()
```

------------------------------------------------------------------------

# 28. Most Important Interview Comparisons

## `isVisible()` vs `toBeVisible()`

``` text
isVisible()
→ query
→ current boolean

toBeVisible()
→ assertion
→ waits/retries
```

## `count()` vs `toHaveCount()`

``` text
count()
→ current number

toHaveCount()
→ assertion + retry
```

## `page.url()` vs `toHaveURL()`

``` text
page.url()
→ retrieve current URL

toHaveURL()
→ assert URL with waiting
```

## `toHaveText()` vs `toContainText()`

``` text
toHaveText()
→ validate expected text/content

toContainText()
→ validate contained text
```

## `toHaveText()` vs `toHaveValue()`

``` text
toHaveText()
→ element text/content

toHaveValue()
→ form control value
```

## `expect()` vs `expect.soft()`

``` text
expect()
→ fail and stop normal execution

expect.soft()
→ record failure and continue
```

## `poll()` vs `toPass()`

``` text
poll()
→ retry a value-producing function

toPass()
→ retry the entire callback
```

------------------------------------------------------------------------

# 29. Common Mistakes

## Mistake 1 --- Using query methods as assertions

``` ts
expect(await button.isVisible()).toBe(true);
```

This is a current-value check.

Prefer:

``` ts
await expect(button).toBeVisible();
```

when you need Playwright to wait for the web condition.

------------------------------------------------------------------------

## Mistake 2 --- Extracting text unnecessarily

Instead of:

``` ts
const text = await message.textContent();
expect(text).toContain("Success");
```

when a web-first assertion is appropriate:

``` ts
await expect(message).toContainText("Success");
```

------------------------------------------------------------------------

## Mistake 3 --- Using `waitForTimeout()` to synchronize

``` ts
await page.waitForTimeout(3000);
```

Don't guess how long the application needs when you can wait for the
actual condition.

------------------------------------------------------------------------

## Mistake 4 --- Using soft assertions for prerequisites

Don't hide a critical failure just so the test can continue.

------------------------------------------------------------------------

## Mistake 5 --- Using `poll()` for normal UI assertions

If this works:

``` ts
await expect(message).toBeVisible();
```

don't replace it with custom polling.

------------------------------------------------------------------------

## Mistake 6 --- Over-asserting implementation details

Don't verify CSS, classes, and attributes unless they are meaningful
requirements.

------------------------------------------------------------------------

## Mistake 7 --- Confusing Locator with DOM element

``` ts
const rows = page.locator("table tr");
```

means:

``` text
ONE Locator object
that can match MANY elements
```

It does not mean "one row."

------------------------------------------------------------------------

# 30. What You Actually Need to Remember

If you forget everything else, remember this:

``` text
1. Locator/Page/Response → Playwright assertion

2. JavaScript value → generic assertion

3. Web condition changes over time → web-first assertion

4. Changing API/JS value → expect.poll()

5. Multiple related checks need retry → toPass()

6. Multiple independent checks → expect.soft()

7. Need opposite → .not

8. Need exact count → toHaveCount()

9. Don't use fixed waits when you can wait for the condition

10. Assert business behavior, not implementation details
```

------------------------------------------------------------------------

# 31. 30-Second Revision Sheet

``` text
VISIBLE      → toBeVisible()
ENABLED      → toBeEnabled()
CHECKED      → toBeChecked()
EDITABLE     → toBeEditable()
FOCUSED      → toBeFocused()

TEXT         → toHaveText()
CONTAINS TEXT→ toContainText()
VALUE        → toHaveValue()

ATTRIBUTE    → toHaveAttribute()
CLASS        → toHaveClass()
CSS          → toHaveCSS()
COUNT        → toHaveCount()

URL          → toHaveURL()
TITLE        → toHaveTitle()

API SUCCESS  → toBeOK()
EXACT STATUS → response.status()

VALUE        → toBe()
OBJECT/ARRAY → toEqual()
CONTAINS     → toContain()
PATTERN      → toMatch()

OPPOSITE     → .not
CONTINUE     → expect.soft()
POLL VALUE   → expect.poll()
RETRY BLOCK  → toPass()
```

------------------------------------------------------------------------

# 32. Interview-Ready Answer

### "What is Playwright expect?"

> `expect` is Playwright Test's assertion API. It provides web-first
> assertions for locators, pages, and API responses that can
> automatically wait and retry, as well as generic assertions for
> already-available JavaScript values.

### "Why prefer web-first assertions?"

> They wait for the expected condition instead of checking the web state
> only once, which helps avoid timing-related flaky tests.

### "How do you decide which assertion to use?"

> I first identify what I need to prove---element state, text, value,
> count, page state, API response, or a JavaScript value---and then
> choose the assertion that directly represents that requirement.

### "When do you use `expect.poll()`?"

> When a JavaScript, API, or external value changes over time and I need
> to repeatedly retrieve it until it satisfies the expectation.

### "When do you use `toPass()`?"

> When multiple operations or assertions need to be retried together as
> one block.

------------------------------------------------------------------------

# 33. Final Master Mental Model

``` text
                    EXPECT
                      |
        +-------------+-------------+
        |             |             |
      WEB           PAGE           VALUE
        |             |             |
     Locator         URL          JS value
        |            Title            |
        |             |          toBe / toEqual
        |             |          toContain / toMatch
        |
   web-first
   assertions
        |
   state / text /
   value / count /
   attribute / etc.
        |
        +----------------------+
                               |
                         VALUE CHANGES?
                               |
                         +-----+-----+
                         |           |
                        YES          NO
                         |           |
                    expect.poll()   normal
                                    expect

MULTIPLE CHECKS RETRY TOGETHER
            ↓
        toPass()

MULTIPLE INDEPENDENT CHECKS
            ↓
       expect.soft()

OPPOSITE CONDITION
            ↓
           .not
```

------------------------------------------------------------------------

# 34. Final Rule

> **Do not memorize `expect` as a list of methods.**
>
> **Train yourself to think: "What exactly am I trying to prove?"**
>
> Once you identify the thing being proven, the assertion usually
> becomes obvious.

------------------------------------------------------------------------

## Accuracy Note

These notes were reviewed against the current official Playwright
Assertions documentation before being prepared. The core concepts
covered here are intentionally limited to the practical API needed for
real-world Playwright work and interviews.

### Official documentation used for verification

-   Playwright Assertions
-   Playwright Generic Assertions
-   Playwright Page Assertions
-   Playwright API Response Assertions
-   Playwright Best Practices
