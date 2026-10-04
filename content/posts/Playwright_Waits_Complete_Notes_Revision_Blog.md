# Playwright Waits & Synchronization

> **Purpose:** Learning + Revision + Interview Preparation + Blog\
> **Core idea:** **Do not ask "Where should I add a wait?" Ask "What
> condition am I waiting for?"**

------------------------------------------------------------------------

## 1. What Problem Do Waits Solve?

Web applications are not always ready immediately.

``` text
Click Login
   ↓
Request sent
   ↓
Server processes request
   ↓
Application updates UI
   ↓
Dashboard appears
```

If automation interacts before the required condition is ready, the test
can fail.

Playwright handles a large part of this synchronization automatically.

> **Playwright waits for relevant conditions instead of requiring fixed
> delays everywhere.**

------------------------------------------------------------------------

## 2. The Big Picture

``` text
                 WHAT AM I WAITING FOR?
                         |
          +--------------+--------------+
          |              |              |
      UI ACTION       UI RESULT      SPECIFIC EVENT
          |              |              |
     Auto-waiting      expect()     URL / Network /
                                   Locator state
```

Typical choices:

  -----------------------------------------------------------------------
  Need                                Prefer
  ----------------------------------- -----------------------------------
  Perform a UI action                 Locator action + auto-waiting

  Verify UI state                     `expect()`

  Wait for Locator state              `locator.waitFor()`

  Wait for URL                        `page.waitForURL()`

  Wait for request                    `page.waitForRequest()`

  Wait for response                   `page.waitForResponse()`

  Wait for page load state            `page.waitForLoadState()`

  Fixed delay                         `page.waitForTimeout()` --- usually
                                      avoid as normal synchronization
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 3. Auto-Waiting --- The Most Important Concept

For supported Locator actions, Playwright performs relevant
**actionability checks**, waits for them to pass, and then performs the
action.

Example:

``` ts
await page.getByRole("button", { name: "Submit" }).click();
```

Mental model:

``` text
click()
   ↓
Can Playwright safely perform this action?
   ↓
Not yet → wait/retry
   ↓
Ready
   ↓
click
```

If the required checks do not pass within the applicable timeout, the
action fails with a timeout error.

------------------------------------------------------------------------

# 4. Actionability --- What Does It Mean?

**Actionability** means:

> **Is the element in a state where Playwright can safely perform the
> requested action?**

For `click()`, Playwright checks that:

1.  The Locator resolves to exactly one element.
2.  The element is visible.
3.  The element is stable.
4.  The element receives pointer events.
5.  The element is enabled.

This is much better than simply remembering:

> "Playwright waits."

You should remember:

> **Playwright waits for the conditions relevant to the requested
> action.**

------------------------------------------------------------------------

# 5. Different Actions Have Different Checks

This is a critical accuracy point.

Do **not** memorize:

> "Every action checks visible + stable + enabled + receives events."

That is not correct.

The checks depend on the action.

Examples:

``` text
click()
   → Visible
   → Stable
   → Receives Events
   → Enabled

fill()
   → Visible
   → Enabled
   → Editable

hover()
   → Visible
   → Stable
   → Receives Events

selectOption()
   → Visible
   → Enabled
```

### Permanent mental model

> **The action determines the actionability checks.**

You do not need to memorize the complete actionability table for
everyday work.

------------------------------------------------------------------------

# 6. Locator Uniqueness

For actions such as `click()`, the Locator needs to resolve to exactly
one element.

``` ts
const submitButton =
  page.getByRole("button", { name: "Submit" });

await submitButton.click();
```

Think:

``` text
Locator
   ↓
Exactly one element?
   |
   +-- No → Locator problem
   |
   +-- Yes
          ↓
      Actionability checks
          ↓
      Action
```

### Important distinction

If a Locator matches multiple elements, simply waiting longer does not
solve the underlying problem.

> **Wait problem ≠ Locator problem**

------------------------------------------------------------------------

# 7. Actionability Checks in Simple English

  -----------------------------------------------------------------------
  Check                               Simple meaning
  ----------------------------------- -----------------------------------
  **Visible**                         The element has a visible/non-empty
                                      rendered area and is not
                                      `visibility:hidden`.

  **Stable**                          The element is not still
                                      moving/animating.

  **Receives Events**                 The pointer action can actually
                                      reach the element; another element
                                      is not blocking it.

  **Enabled**                         The control is not disabled.

  **Editable**                        The element is enabled and not
                                      readonly.
  -----------------------------------------------------------------------

You do not need to memorize the browser internals behind these
definitions.

------------------------------------------------------------------------

# 8. Visible

Playwright considers an element visible when it has a non-empty bounding
box and does not have `visibility:hidden`.

Important examples:

``` text
display:none
    → Not visible

Zero-size element
    → Not visible

opacity:0
    → Playwright still considers it visible
```

### Remember

> **Playwright has a defined visibility check. "I cannot visually see
> it" is not always enough to predict the result.**

------------------------------------------------------------------------

# 9. Stable

An element is considered stable when its bounding box stays the same for
at least two consecutive animation frames.

Simple meaning:

> **The element has stopped moving enough for Playwright to safely act
> on it.**

``` text
Button is moving
      ↓
Playwright waits
      ↓
Button becomes stable
      ↓
Action proceeds
```

You do not need to calculate animation frames yourself.

------------------------------------------------------------------------

# 10. Receives Events

Imagine:

``` text
Your button
     ↓
Overlay on top
     ↓
Mouse click reaches overlay
```

The button may be visible, but it is not the actual hit target.

Playwright checks whether the target can receive the pointer event.

Simple meaning:

> **"Can the click actually reach this element?"**

This is useful when troubleshooting:

-   overlays
-   loading screens
-   modal layers
-   another element covering the target

------------------------------------------------------------------------

# 11. Enabled

An element is enabled when it is not disabled.

Example:

``` html
<button disabled>Submit</button>
```

Simple meaning:

> **"Is the control allowed to be used?"**

------------------------------------------------------------------------

# 12. Editable

Editable is particularly relevant to actions such as `fill()`.

An element is considered editable when it is enabled and not readonly.

Example:

``` html
<input readonly />
```

Mental model:

``` text
fill()
  ↓
Visible?
  ↓
Enabled?
  ↓
Editable?
  ↓
Fill
```

------------------------------------------------------------------------

# 13. `click()` Example

``` ts
await page.getByRole("button", { name: "Submit" }).click();
```

Conceptually:

``` text
Find exactly one element
        ↓
Visible?
        ↓
Stable?
        ↓
Receives events?
        ↓
Enabled?
        ↓
CLICK
```

If a required condition is not satisfied, Playwright waits/retries until
the applicable timeout.

------------------------------------------------------------------------

# 14. `fill()` Example

``` ts
await page.getByLabel("Username").fill("admin");
```

The relevant checks are different from `click()`.

Conceptually:

``` text
Find element
    ↓
Visible?
    ↓
Enabled?
    ↓
Editable?
    ↓
Fill
```

Therefore:

> **Never assume all Playwright actions use the same actionability
> checks.**

------------------------------------------------------------------------

# 15. Auto-Waiting Does NOT Mean "Wait Until Everything Is Ready"

Playwright does not mean:

> "Wait until the entire application is completely finished."

It means:

> **Wait for conditions relevant to the requested action.**

Example:

``` ts
await page.getByRole("button", { name: "Submit" }).click();
```

Playwright is concerned with whether that button can be clicked.

It is not automatically saying:

> "Wait until every API call on the page has finished."

------------------------------------------------------------------------

# 16. Web-First Assertions

Example:

``` ts
await expect(
  page.getByText("Order placed")
).toBeVisible();
```

Playwright provides **auto-retrying assertions**.

Mental model:

``` text
expect(toBeVisible)
        ↓
Visible?
        ↓
No
 ↓
wait/retry
 ↓
check again
 ↓
Yes
 ↓
PASS
```

If the expected condition is not satisfied within the assertion timeout,
the assertion fails.

------------------------------------------------------------------------

# 17. Auto-Waiting vs Assertion

## Action

``` ts
await button.click();
```

Question:

> **"Can Playwright perform this action?"**

## Assertion

``` ts
await expect(message).toBeVisible();
```

Question:

> **"Has the expected result happened?"**

Mental model:

``` text
ACTION
click()
fill()
check()
selectOption()
      ↓
“Can I perform this action?”

ASSERTION
toBeVisible()
toHaveText()
toHaveValue()
toHaveCount()
      ↓
“Has the expected result happened?”
```

------------------------------------------------------------------------

# 18. Real Login Example

``` ts
await page.getByLabel("Username").fill("admin");

await page.getByLabel("Password").fill("secret");

await page.getByRole("button", { name: "Login" }).click();

await expect(
  page.getByText("Welcome Admin")
).toBeVisible();
```

Notice there is no arbitrary:

``` ts
await page.waitForTimeout(3000);
```

between every step.

The flow is:

``` text
fill
 ↓
Locator action + relevant auto-waiting

click
 ↓
Locator action + relevant auto-waiting

expect
 ↓
Retry until expected UI state
```

------------------------------------------------------------------------

# 19. `page.waitForTimeout()` --- Fixed Wait

``` ts
await page.waitForTimeout(3000);
```

Meaning:

> **Pause execution for exactly 3 seconds.**

Why is it usually a poor synchronization strategy?

### Application ready in 500 ms

``` text
Application ready
     ↓
500 ms
     ↓
Still waiting...
     ↓
3000 ms
     ↓
Continue
```

### Application needs 5 seconds

``` text
Application still processing
     ↓
3000 ms
     ↓
Test continues
     ↓
Potential failure
```

Therefore fixed waits can be:

-   too long
-   too short
-   slower than necessary
-   unrelated to the actual application condition

### Better approach

Prefer:

-   auto-waiting
-   web-first assertions
-   condition-specific synchronization

when they represent the condition you actually need.

------------------------------------------------------------------------

# 20. When Is `waitForTimeout()` Useful?

You may occasionally see it during:

-   debugging
-   temporary local investigation
-   experiments

But it should not become your normal production synchronization
strategy.

> **If you know what you are waiting for, wait for that condition
> instead of sleeping for an arbitrary number of milliseconds.**

------------------------------------------------------------------------

# 21. `locator.waitFor()`

Use this when you specifically need a Locator to reach a state.

Example:

``` ts
await page.locator("#spinner").waitFor({
  state: "hidden"
});
```

Mental model:

``` text
Locator
   ↓
waitFor()
   ↓
“Wait until THIS Locator reaches THIS state”
```

Common states:

``` text
attached
detached
visible
hidden
```

------------------------------------------------------------------------

# 22. `locator.waitFor()` vs `expect()`

### `locator.waitFor()`

``` ts
await spinner.waitFor({
  state: "hidden"
});
```

Meaning:

> **Wait for this Locator to reach this state.**

### `expect()`

``` ts
await expect(table).toBeVisible();
```

Meaning:

> **Verify that the application reached the expected state.**

For a test assertion, `expect()` often communicates the intent more
clearly.

------------------------------------------------------------------------

# 23. `page.waitForURL()`

Use this when the URL change itself is the condition you care about.

``` ts
await page.getByRole("button", { name: "Login" }).click();

await page.waitForURL("**/dashboard");
```

Mental model:

``` text
Click Login
     ↓
Navigation
     ↓
URL changes
     ↓
waitForURL()
     ↓
Expected URL reached
```

------------------------------------------------------------------------

# 24. `page.waitForLoadState()`

Example:

``` ts
await page.waitForLoadState("load");
```

or:

``` ts
await page.waitForLoadState("domcontentloaded");
```

Important:

> **A page load state does not automatically mean the application's
> business UI is ready.**

Example:

``` text
HTML loads
   ↓
load event
   ↓
JavaScript runs
   ↓
API request
   ↓
Response
   ↓
Table renders
```

If your test cares about the table:

``` ts
await expect(page.getByRole("table")).toBeVisible();
```

may represent the actual requirement better.

------------------------------------------------------------------------

# 25. `networkidle` --- Important Limitation

You may see:

``` ts
await page.waitForLoadState("networkidle");
```

It waits for network activity to become idle.

But modern applications can continuously make background requests:

``` text
Analytics
Polling
Background APIs
Other network activity
```

Therefore:

> **Network idle does not automatically mean the UI condition you need
> is ready.**

Do not treat:

``` text
networkidle = application ready
```

as a universal rule.

Instead ask:

> **What does my test actually need to wait for?**

------------------------------------------------------------------------

# 26. `page.waitForRequest()`

Use it when you need to synchronize with a specific outgoing request.

``` ts
const requestPromise = page.waitForRequest(
  request => request.url().includes("/api/customers")
);

await page.getByRole("button", { name: "Search" }).click();

const request = await requestPromise;
```

Mental model:

> **`waitForRequest()` = "Wait until this request happens."**

------------------------------------------------------------------------

# 27. `page.waitForResponse()`

Use it when you need to synchronize with a specific response.

``` ts
const responsePromise = page.waitForResponse(
  response =>
    response.url().includes("/api/customers") &&
    response.status() === 200
);

await page.getByRole("button", { name: "Search" }).click();

const response = await responsePromise;
```

Mental model:

> **`waitForResponse()` = "Wait until this response arrives."**

------------------------------------------------------------------------

# 28. Why Set Up Network Waits Before the Action?

Recommended pattern:

``` ts
const responsePromise = page.waitForResponse(...);

await page.getByRole("button", { name: "Search" }).click();

const response = await responsePromise;
```

Why?

Because the action can trigger the request immediately.

``` text
Create wait
    ↓
Click
    ↓
Request happens
    ↓
Response arrives
    ↓
Wait resolves
```

This makes the synchronization safer and easier to reason about.

------------------------------------------------------------------------

# 29. Request vs Response

``` text
Browser
   |
   | ---- REQUEST ---->
   |
   | <---- RESPONSE ----
   |
```

Therefore:

``` text
waitForRequest()
    ↓
Outgoing request

waitForResponse()
    ↓
Incoming response
```

------------------------------------------------------------------------

# 30. `page.waitForSelector()`

You may encounter:

``` ts
await page.waitForSelector("#username");
```

Know what it is, but do not automatically make it your first choice.

Instead of:

``` ts
await page.waitForSelector("#username");

await page.locator("#username").fill("admin");
```

you can normally use:

``` ts
await page.locator("#username").fill("admin");
```

The Locator action provides relevant auto-waiting.

### Rule

> **Do not add a separate wait when the action already provides the
> synchronization you need.**

------------------------------------------------------------------------

# 31. Locators Are Central to Auto-Waiting

A Locator represents a way to find element(s) on the page.

Example:

``` ts
const submitButton =
  page.getByRole("button", { name: "Submit" });
```

Mental model:

``` text
Locator
   ↓
Way to identify the element
   ↓
Action uses Locator
   ↓
Current matching element is resolved
   ↓
Relevant actionability checks
   ↓
Action
```

A Locator is not simply a permanently stored DOM element.

When used for an action, Playwright can resolve the current matching
element. This is useful when the DOM changes or the page re-renders.

------------------------------------------------------------------------

# 32. Real-Project Example: Loading Spinner

Application flow:

``` text
Click Search
     ↓
Spinner appears
     ↓
API call
     ↓
Spinner disappears
     ↓
Customer table appears
```

### If you care about spinner disappearing

``` ts
await page.locator(".spinner").waitFor({
  state: "hidden"
});
```

### If you care about the table being ready

``` ts
await expect(
  page.getByRole("table")
).toBeVisible();
```

### If the API response itself matters

``` ts
const responsePromise = page.waitForResponse(
  response =>
    response.url().includes("/api/customers") &&
    response.status() === 200
);

await page.getByRole("button", { name: "Search" }).click();

await responsePromise;
```

The correct choice depends on **what the test needs to prove**.

------------------------------------------------------------------------

# 33. Real-Project Example: Insurance Claim

Imagine:

``` text
Create Claim
     ↓
Click Submit
     ↓
POST /claims
     ↓
Server processes claim
     ↓
Claim ID appears
     ↓
Success message appears
```

## Poor approach

``` ts
await page.getByRole("button", { name: "Submit" }).click();

await page.waitForTimeout(5000);

await expect(
  page.getByText("Claim created")
).toBeVisible();
```

The 5 seconds are arbitrary.

## Better UI-based synchronization

``` ts
await page.getByRole("button", { name: "Submit" }).click();

await expect(
  page.getByText("Claim created")
).toBeVisible();
```

## If the API response itself matters

``` ts
const responsePromise = page.waitForResponse(
  response =>
    response.url().includes("/claims") &&
    response.request().method() === "POST" &&
    response.status() === 201
);

await page.getByRole("button", { name: "Submit" }).click();

const response = await responsePromise;

await expect(
  page.getByText("Claim created")
).toBeVisible();
```

------------------------------------------------------------------------

# 34. `force: true` --- Know It, Don't Abuse It

Some actions, such as `click()`, support:

``` ts
await button.click({ force: true });
```

`force: true` disables **non-essential actionability checks**.

For `click()`, this can skip the check that the target actually receives
click events.

Mental model:

``` text
Normal click
    ↓
Actionability checks
    ↓
Click

force: true
    ↓
Some non-essential checks bypassed
    ↓
Click
```

### Important

Do not use `force: true` as the first fix when a click fails.

First investigate:

``` text
Overlay?
Wrong Locator?
Disabled control?
Animation?
Application defect?
```

`force` can hide the real problem.

------------------------------------------------------------------------

# 35. Common Mistakes

## Mistake 1 --- Add `waitForTimeout()` everywhere

``` ts
await page.waitForTimeout(3000);
await button.click();

await page.waitForTimeout(2000);
await expect(message).toBeVisible();
```

Better question:

``` text
What condition am I actually waiting for?
```

------------------------------------------------------------------------

## Mistake 2 --- Wait for page load when you need a UI condition

``` ts
await page.waitForLoadState("load");
```

does not necessarily mean:

``` text
“My table is ready.”
```

Use the actual UI condition when that is what matters:

``` ts
await expect(page.getByRole("table")).toBeVisible();
```

------------------------------------------------------------------------

## Mistake 3 --- Add `waitFor()` before every action

Unnecessary pattern:

``` ts
await button.waitFor();

await button.click();
```

If `click()` already has the relevant auto-waiting, the extra wait adds
no useful synchronization.

------------------------------------------------------------------------

## Mistake 4 --- Think every action has the same checks

Incorrect:

``` text
Every action
    ↓
Visible + Stable + Enabled + Receives Events
```

Correct:

``` text
Action
   ↓
Relevant actionability checks for THAT action
```

------------------------------------------------------------------------

## Mistake 5 --- Assume network idle means application ready

``` text
networkidle
    ≠
every business UI condition is ready
```

------------------------------------------------------------------------

## Mistake 6 --- Use `force: true` to hide a problem

Before forcing an action, understand why normal actionability is
failing.

------------------------------------------------------------------------

# 36. Synchronization Decision Tree

``` text
I need synchronization
        |
        v
What am I waiting for?
        |
        +-------------------------------+
        |                               |
   UI ACTION                       UI VERIFICATION
        |                               |
   click/fill/etc.                  expect()
        |                               |
   Auto-waiting                  Web-first assertion
        |
        |
   Need a specific condition?
        |
   +----+-----------+------------+
   |                |            |
  URL            NETWORK       LOCATOR
   |                |            |
waitForURL()   Request/       waitFor()
               Response
```

------------------------------------------------------------------------

# 37. MUST / SHOULD / KNOW WHEN NEEDED

## 🔴 MUST KNOW

1.  Auto-waiting
2.  Actionability
3.  Locator uniqueness for actions such as `click()`
4.  Different actions have different actionability checks
5.  Web-first assertions
6.  Assertion retry behavior
7.  Why `waitForTimeout()` is generally avoided
8.  `locator.waitFor()`
9.  `waitForURL()`
10. `waitForResponse()`
11. `waitForRequest()`
12. `waitForLoadState()`
13. `networkidle` limitation
14. `force: true` concept

## 🟠 SHOULD KNOW

-   Different page load states
-   Timeout concepts
-   `waitForSelector()` because you will encounter it in existing code
-   How Locator actions and assertions reduce explicit waits

## ⚪ DON'T STUDY DEEPLY

Do not spend significant time on:

-   internal implementation of the waiting engine
-   internal polling algorithms
-   browser protocol internals behind waiting

You need:

> **Behavior + correct usage + troubleshooting + interview
> explanation.**

------------------------------------------------------------------------

# 38. Interview Questions

### Q1. How does Playwright handle synchronization?

**Answer:**

> Playwright provides automatic waiting for relevant actionability
> conditions before supported Locator actions. It also provides
> auto-retrying web-first assertions. When a test depends on a specific
> condition such as a URL, Locator state, request, or response, I use
> the corresponding condition-specific synchronization instead of fixed
> delays.

### Q2. What is actionability?

> Actionability means checking whether an element is in a suitable state
> for the requested action. For example, a click checks conditions such
> as uniqueness, visibility, stability, receiving pointer events, and
> enabled state.

### Q3. Does every Playwright action have the same checks?

> No. The checks depend on the action. For example, `click()` and
> `fill()` do not require exactly the same checks.

### Q4. Why is `waitForTimeout()` discouraged?

> It is a fixed delay and does not synchronize with an actual
> application condition. It can make tests unnecessarily slow or still
> fail if the application takes longer than the chosen delay.

### Q5. What is the difference between auto-waiting and web-first assertions?

> Auto-waiting helps Playwright determine when an action can be
> performed safely. A web-first assertion retries until the expected
> application state is reached or the assertion times out.

### Q6. When would you use `waitForURL()`?

> When a specific URL change is an important condition of the test, such
> as verifying that login navigates to the dashboard.

### Q7. When would you use `waitForResponse()`?

> When the test needs to synchronize with a specific API response
> triggered by an action, such as waiting for a successful claim
> creation response.

### Q8. Why should `waitForResponse()` generally be registered before the triggering action?

> Because the request can happen immediately after the action.
> Registering the wait first ensures Playwright is already waiting when
> the response occurs.

### Q9. Is `networkidle` always the best synchronization method?

> No. Modern applications can continue making background network
> requests. Network idle does not necessarily mean that the specific UI
> condition required by the test is ready.

### Q10. What does `force: true` do?

> For actions that support it, `force: true` disables non-essential
> actionability checks. I would use it only when I understand why the
> normal action cannot be performed.

------------------------------------------------------------------------

# 39. One-Line Memory Tricks

  Concept                Remember it as
  ---------------------- -------------------------------------------------------
  Auto-waiting           **"Can I perform this action safely?"**
  Actionability          **"Is this element ready for THIS action?"**
  `expect()`             **"Has the expected result happened?"**
  `locator.waitFor()`    **"Wait for this Locator state."**
  `waitForURL()`         **"Wait for this URL."**
  `waitForRequest()`     **"Wait for this outgoing request."**
  `waitForResponse()`    **"Wait for this incoming response."**
  `waitForLoadState()`   **"Wait for this page load state."**
  `waitForTimeout()`     **"Wait exactly this long."**
  `networkidle`          **"Network is quiet --- not necessarily UI ready."**
  `force:true`           **"Bypass some non-essential actionability checks."**

------------------------------------------------------------------------

# 40. Golden Rule

> **Do not add a wait because the test is failing. First identify what
> the application is waiting for.**

Then choose the synchronization mechanism that represents that
condition.

``` text
Element/action ready?
    → Locator action + auto-waiting

Expected UI state?
    → expect()

Specific Locator state?
    → locator.waitFor()

Specific URL?
    → waitForURL()

Specific request?
    → waitForRequest()

Specific response?
    → waitForResponse()

Specific page load state?
    → waitForLoadState()

Fixed delay?
    → waitForTimeout()
       Usually avoid as normal synchronization.

Need to bypass some non-essential actionability checks?
    → force: true
       Use only when you understand why.
```

------------------------------------------------------------------------

# 41. Final Mental Map

``` text
                         PLAYWRIGHT
                      SYNCHRONIZATION
                             |
          +------------------+------------------+
          |                                     |
     AUTOMATIC                              EXPLICIT
          |                                     |
   +------+-------+                    +--------+---------+
   |              |                    |        |         |
Locator actions  expect()             URL    Network    State
   |              |                    |        |         |
Actionability   Auto-retry         waitForURL  |     locator
checks          assertion                    |     .waitFor()
                                            |
                                      Request / Response
```

------------------------------------------------------------------------

# 42. The One Sentence to Remember Forever

> **Don't ask "Where should I put a wait?" --- ask "What condition am I
> waiting for?"**

That mindset is more important than memorizing every wait API.

------------------------------------------------------------------------

# 43. 60-Second Revision

Before an interview, remember:

``` text
1. Playwright has automatic waiting.
2. Locator actions wait for relevant actionability conditions.
3. Different actions have different checks.
4. click() requires the Locator to resolve to exactly one element.
5. click() checks visibility, stability, receiving events and enabled state.
6. fill() has its own relevant checks, including editable state.
7. expect() uses auto-retrying web-first assertions.
8. waitForTimeout() is a fixed delay — usually avoid it for synchronization.
9. locator.waitFor() waits for a Locator state.
10. waitForURL() waits for a URL condition.
11. waitForRequest() waits for a request.
12. waitForResponse() waits for a response.
13. waitForLoadState() waits for a page load state, not necessarily business UI readiness.
14. networkidle is not a universal “application ready” signal.
15. force:true bypasses non-essential actionability checks for supported actions.
16. Always identify the actual condition before choosing a wait.
```

------------------------------------------------------------------------

# 44. Official References

Primary Playwright documentation:

-   https://playwright.dev/docs/actionability
-   https://playwright.dev/docs/locators

The Actionability documentation is the authoritative reference for:

-   auto-waiting
-   actionability checks
-   action-specific checks
-   forcing actions
-   auto-retrying assertions
-   definitions of visibility, stability, enabled, editable, and
    receiving events

Use the official documentation when you need the exact behavior of a
particular action.

------------------------------------------------------------------------

## Final Takeaway

**Master the decision, not the API list.**

``` text
WHAT AM I WAITING FOR?
        ↓
Action?        → Auto-waiting
UI result?     → expect()
Locator state? → locator.waitFor()
URL?           → waitForURL()
Request?       → waitForRequest()
Response?      → waitForResponse()
Load state?    → waitForLoadState()
Fixed time?    → waitForTimeout()  ← usually avoid
```
