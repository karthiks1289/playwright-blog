# Playwright Waits

> **Purpose:** Notes, revision, interview preparation, and practical
> understanding\
> **Level:** Beginner → Real-project QA/SDET\
> **Rule:** Do not add waits blindly. First identify **what condition
> you are waiting for**.

------------------------------------------------------------------------

# 1. What Are Playwright Waits?

A web application does not always become ready immediately.

Example:

``` text
Click Login
   ↓
Request sent
   ↓
Server processes request
   ↓
Page/UI changes
   ↓
Dashboard appears
```

If automation tries to interact with something before it is ready, the
test can fail.

Playwright handles a large part of this synchronization automatically.

The most important idea is:

> **Playwright waits for the right condition instead of making you add
> fixed delays everywhere.**

------------------------------------------------------------------------

# 2. The Mental Model

Do not start with:

> "Where should I add a wait?"

Start with:

> **"What exactly am I waiting for?"**

Then choose the appropriate mechanism.

``` text
                 WHAT AM I WAITING FOR?
                         |
        +----------------+----------------+
        |                |                |
     UI action       UI result          Specific event
        |                |                |
   Auto-waiting       expect()       URL / Network /
                                    Locator state
```

Examples:

``` text
Click button
   → Locator action + auto-waiting

Check success message
   → expect()

Wait for URL change
   → waitForURL()

Wait for an element state
   → locator.waitFor()

Wait for an API response
   → waitForResponse()

Wait for a specific request
   → waitForRequest()

Wait for a particular page load state
   → waitForLoadState()
```

------------------------------------------------------------------------

# 3. Auto-Waiting --- The Most Important Topic

When you use a Locator action such as:

``` ts
await page.getByRole("button", { name: "Submit" }).click();
```

Playwright does not simply find the button and immediately attempt the
click.

It performs the relevant **actionability checks** and waits until the
element is ready for the action, subject to the applicable timeout.

Think:

``` text
click()
   ↓
Is the element ready?
   ↓
No → wait/retry
   ↓
Yes
   ↓
click
```

## What Does "Actionability" Mean?

For an action such as `click()`, Playwright checks relevant conditions
before performing the action.

Common conditions include:

-   The element is visible.
-   The element is stable.
-   The element can receive pointer events.
-   The element is enabled.

The exact checks depend on the action.

### Important

Do not memorize a fixed list and assume every Playwright action checks
exactly the same things.

Instead remember:

> **Locator action → Playwright checks whether the element is actionable
> → waits if necessary → performs the action.**

------------------------------------------------------------------------

# 4. Example of Auto-Waiting

Suppose a button appears after some application processing:

``` ts
await page.getByRole("button", { name: "Submit" }).click();
```

You normally do **not** need:

``` ts
await page.waitForTimeout(3000);
await page.getByRole("button", { name: "Submit" }).click();
```

The Locator action already has Playwright's built-in waiting behavior.

This is one of the biggest differences from the old habit of adding
explicit sleeps everywhere.

------------------------------------------------------------------------

# 5. Web-First Assertions

Assertions are another major part of Playwright synchronization.

Example:

``` ts
await expect(page.getByText("Order placed")).toBeVisible();
```

This does not mean:

> "Check once."

Playwright's web-first assertions automatically retry until the expected
condition is satisfied or the assertion timeout is reached.

Think:

``` text
expect(toBeVisible)
        ↓
Is it visible?
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

This is why Playwright assertions are called **web-first assertions**.

------------------------------------------------------------------------

# 6. Auto-Waiting vs Assertions

Do not mix these concepts.

## Locator action

``` ts
await button.click();
```

The question is:

> **"Can Playwright perform this action now?"**

## Assertion

``` ts
await expect(message).toBeVisible();
```

The question is:

> **"Has the expected result happened yet?"**

Mental model:

``` text
ACTION
click()
fill()
selectOption()
     ↓
"Can I perform this action?"

ASSERTION
toBeVisible()
toHaveText()
toHaveValue()
toHaveCount()
     ↓
"Has the expected result happened?"
```

------------------------------------------------------------------------

# 7. Real Login Example

``` ts
await page.getByLabel("Username").fill("admin");

await page.getByLabel("Password").fill("secret");

await page.getByRole("button", { name: "Login" }).click();

await expect(page.getByText("Welcome Admin")).toBeVisible();
```

Notice that there is no arbitrary:

``` ts
await page.waitForTimeout(3000);
```

between every step.

The flow is:

``` text
fill username
   ↓
Locator action handles relevant waiting

fill password
   ↓
Locator action handles relevant waiting

click Login
   ↓
Locator action handles relevant waiting

verify welcome message
   ↓
expect() retries until condition is satisfied
```

------------------------------------------------------------------------

# 8. `page.waitForTimeout()` --- Fixed Wait

Example:

``` ts
await page.waitForTimeout(3000);
```

This means:

> Pause execution for 3 seconds.

It is a **fixed delay**.

## Why is this usually a poor synchronization strategy?

Suppose the application becomes ready after 500 ms:

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

The test wasted time.

Now suppose the application needs 5 seconds:

``` text
Application still processing
     ↓
3000 ms reached
     ↓
Test continues too early
```

So a fixed wait can be:

-   too long
-   too short
-   slower than necessary
-   unrelated to the real application condition

## Preferred approach

Use:

-   Playwright's auto-waiting
-   web-first assertions
-   condition-specific waits

when they match the situation.

## Interview answer

> "`page.waitForTimeout()` is a fixed delay. I avoid using it as normal
> synchronization in production tests because it does not wait for an
> actual application condition. I prefer Playwright's auto-waiting,
> web-first assertions, or a condition-specific wait."

------------------------------------------------------------------------

# 9. `locator.waitFor()`

Sometimes you specifically need to wait for a Locator to reach a
particular state.

Example:

``` ts
await page.locator("#spinner").waitFor({
  state: "hidden"
});
```

The mental model is:

``` text
Locator
   ↓
waitFor()
   ↓
"Wait until THIS locator reaches THIS state"
```

Common states are:

``` text
attached
detached
visible
hidden
```

## Example

Suppose:

``` text
Loading spinner
     ↓
API processing
     ↓
Spinner disappears
     ↓
Data appears
```

You could synchronize with:

``` ts
await page.locator("#spinner").waitFor({
  state: "hidden"
});
```

But always ask whether you actually need this.

If the real requirement is:

> "The customer table should appear."

Then this may communicate the test intention better:

``` ts
await expect(page.getByRole("table")).toBeVisible();
```

------------------------------------------------------------------------

# 10. `page.waitForURL()`

Use this when the URL change itself is the condition you care about.

Example:

``` ts
await page.getByRole("button", { name: "Login" }).click();

await page.waitForURL("**/dashboard");
```

Mental model:

``` text
Click Login
    ↓
Navigation happens
    ↓
URL changes
    ↓
waitForURL()
    ↓
Expected URL reached
```

This is useful when navigation to a particular URL is an important part
of the test.

------------------------------------------------------------------------

# 11. `page.waitForLoadState()`

Playwright provides page load states such as:

``` ts
await page.waitForLoadState("load");
```

and:

``` ts
await page.waitForLoadState("domcontentloaded");
```

The important point is:

> A page load state is not automatically the same thing as "the
> application is ready."

For example:

``` text
HTML loads
   ↓
load event
   ↓
JavaScript executes
   ↓
API request
   ↓
API response
   ↓
Table renders
```

If your test really cares about the table, waiting for the table's
expected state can be more meaningful:

``` ts
await expect(page.getByRole("table")).toBeVisible();
```

Do not use `waitForLoadState()` as a universal replacement for all other
synchronization.

------------------------------------------------------------------------

# 12. `networkidle` --- Important Concept

You may see:

``` ts
await page.waitForLoadState("networkidle");
```

The idea is to wait until network activity becomes idle.

However, modern applications may continuously make background requests.

For example:

``` text
Page
 ↓
API calls
 ↓
Analytics
 ↓
Polling
 ↓
Background requests
```

Therefore:

> **"Network is idle" does not automatically mean "the UI is ready for
> my test."**

Use a synchronization condition that matches what your test actually
needs.

For example:

``` ts
await expect(page.getByRole("table")).toBeVisible();
```

if the table is the actual requirement.

------------------------------------------------------------------------

# 13. `page.waitForResponse()`

This is useful when your test needs to synchronize with a particular API
response.

Example:

``` ts
const responsePromise = page.waitForResponse(
  response =>
    response.url().includes("/api/customers") &&
    response.status() === 200
);

await page.getByRole("button", { name: "Search" }).click();

const response = await responsePromise;
```

## Why create the wait before the action?

Because the action may trigger the request immediately.

Mental model:

``` text
Create response wait
        ↓
Click Search
        ↓
Application sends request
        ↓
Matching response arrives
        ↓
Promise resolves
```

This pattern is important in real automation when network activity is
part of the test condition.

------------------------------------------------------------------------

# 14. `page.waitForRequest()`

Similar idea, but this waits for a matching **request**.

Example:

``` ts
const requestPromise = page.waitForRequest(
  request => request.url().includes("/api/customers")
);

await page.getByRole("button", { name: "Search" }).click();

const request = await requestPromise;
```

Remember:

``` text
waitForRequest()
    ↓
"Wait until this request happens."

waitForResponse()
    ↓
"Wait until this response arrives."
```

------------------------------------------------------------------------

# 15. Request vs Response

This distinction is easy to remember.

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
Wait for outgoing request

waitForResponse()
       ↓
Wait for incoming response
```

------------------------------------------------------------------------

# 16. `page.waitForSelector()`

You may encounter:

``` ts
await page.waitForSelector("#username");
```

Know what it does, but do not automatically make it your first choice.

For example, instead of:

``` ts
await page.waitForSelector("#username");

await page.locator("#username").fill("admin");
```

you can normally rely on the Locator action:

``` ts
await page.locator("#username").fill("admin");
```

The Locator action provides the relevant auto-waiting.

### Simple rule

> **Prefer Locator-based actions and web-first assertions instead of
> adding separate waits unnecessarily.**

------------------------------------------------------------------------

# 17. The Most Important Wait Decision Tree

Use this in real projects.

``` text
I need synchronization
        |
        v
What am I waiting for?
        |
        +----------------------------+
        |                            |
   UI interaction              UI verification
        |                            |
   click/fill/etc.               expect()
        |                            |
   Auto-waiting                Web-first assertion
        |
        |
   Need a specific condition?
        |
   +----+---------+----------+
   |              |          |
  URL          Network     Locator
   |              |          |
waitForURL()  Request/     waitFor()
              Response
```

------------------------------------------------------------------------

# 18. Practical Example --- Insurance Application

Imagine a claims application.

Flow:

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

await expect(page.getByText("Claim created")).toBeVisible();
```

The 5 seconds are arbitrary.

## Better UI-based synchronization

``` ts
await page.getByRole("button", { name: "Submit" }).click();

await expect(page.getByText("Claim created")).toBeVisible();
```

Now the test waits for the actual UI result.

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

await expect(page.getByText("Claim created")).toBeVisible();
```

The correct synchronization depends on what the test needs to prove.

------------------------------------------------------------------------

# 19. What You MUST Learn

## 🔴 MUST KNOW

1.  Playwright auto-waiting
2.  Actionability checks
3.  Web-first assertions
4.  Assertion retry behavior
5.  Why fixed waits are usually avoided
6.  `locator.waitFor()`
7.  `page.waitForURL()`
8.  `page.waitForResponse()`
9.  `page.waitForLoadState()`
10. Why `networkidle` is not a universal "application ready" signal

------------------------------------------------------------------------

# 20. What You SHOULD KNOW

These are useful, but you do not need to spend excessive time on them
initially:

-   `page.waitForRequest()`
-   `page.waitForSelector()`
-   Different page load states
-   Timeout configuration

------------------------------------------------------------------------

# 21. What You DON'T Need to Study Deeply

For your current SDET preparation, do not spend significant learning
time on:

-   Internal implementation of Playwright's waiting engine
-   Internal polling implementation
-   Browser protocol internals related to waiting

Know the behavior and how to use it.

------------------------------------------------------------------------

# 22. Common Mistakes

## Mistake 1 --- Adding `waitForTimeout()` everywhere

``` ts
await page.waitForTimeout(3000);
await button.click();

await page.waitForTimeout(2000);
await expect(message).toBeVisible();
```

### Better thinking

Ask:

``` text
What am I actually waiting for?
```

Then synchronize with that condition.

------------------------------------------------------------------------

## Mistake 2 --- Waiting for page load when you need a UI condition

``` ts
await page.waitForLoadState("load");
```

does not necessarily mean:

``` text
"My table is ready."
```

If the table is what matters:

``` ts
await expect(page.getByRole("table")).toBeVisible();
```

------------------------------------------------------------------------

## Mistake 3 --- Thinking every Locator needs `waitFor()`

This is unnecessary:

``` ts
await page.getByRole("button", { name: "Submit" }).waitFor();

await page.getByRole("button", { name: "Submit" }).click();
```

The click action already has its own relevant waiting behavior.

------------------------------------------------------------------------

## Mistake 4 --- Using network idle as a universal solution

Do not think:

``` text
networkidle = application ready
```

That is not a safe general assumption for modern applications.

------------------------------------------------------------------------

# 23. Interview Questions

## Q1. Does Playwright support automatic waiting?

**Answer:**

Yes. Playwright automatically waits for relevant actionability
conditions before performing supported Locator actions. This reduces the
need for manual waits.

------------------------------------------------------------------------

## Q2. Why is `waitForTimeout()` generally discouraged?

**Answer:**

It is a fixed delay. It does not synchronize with the actual application
condition, so it can make tests slower or still fail if the application
takes longer than the chosen delay.

------------------------------------------------------------------------

## Q3. What is the difference between auto-waiting and assertions?

**Answer:**

Auto-waiting helps Playwright determine when an action can safely be
performed. Web-first assertions retry until the expected application
state is reached or the assertion timeout is exceeded.

------------------------------------------------------------------------

## Q4. When would you use `waitForURL()`?

**Answer:**

When a specific URL change is an important condition of the test, such
as verifying that login navigates to the dashboard.

------------------------------------------------------------------------

## Q5. When would you use `waitForResponse()`?

**Answer:**

When the test needs to synchronize with a particular API response
triggered by an action, such as waiting for a successful POST request
after submitting a form.

------------------------------------------------------------------------

## Q6. Is `networkidle` always the best way to wait for a page?

**Answer:**

No. Modern applications can continue making background requests. Network
idle does not necessarily mean that the specific UI condition required
by the test is ready.

------------------------------------------------------------------------

## Q7. Why should `waitForResponse()` usually be set up before the action that triggers the request?

**Answer:**

Because the request can happen immediately after the action. Setting up
the wait first ensures that Playwright is already waiting when the
request occurs.

------------------------------------------------------------------------

# 24. One-Line Memory Tricks

  Concept                Remember it as
  ---------------------- ------------------------------------------------------
  Auto-waiting           **"Can I perform this action?"**
  `expect()`             **"Has the expected result happened?"**
  `waitFor()`            **"Wait for this Locator state."**
  `waitForURL()`         **"Wait for this URL."**
  `waitForRequest()`     **"Wait for this request."**
  `waitForResponse()`    **"Wait for this response."**
  `waitForLoadState()`   **"Wait for this page load state."**
  `waitForTimeout()`     **"Wait exactly this long."**
  `networkidle`          **"Network is quiet --- not necessarily UI ready."**

------------------------------------------------------------------------

# 25. The Golden Rule

> **Do not add a wait because the test is failing. First identify what
> the application is waiting for.**

Then choose the synchronization mechanism that represents that
condition.

``` text
Element ready?
    → Locator action / auto-waiting

Expected UI state?
    → expect()

Specific element state?
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
```

------------------------------------------------------------------------

# 26. Final Mental Map

``` text
                    PLAYWRIGHT WAITS
                           |
             +-------------+-------------+
             |                           |
        AUTOMATIC                    EXPLICIT
             |                           |
       +-----+------+            +-------+--------+
       |            |            |       |        |
   Locator       expect()      URL    Network   State
    action      assertion       |       |         |
       |            |       waitForURL  |     locator
       |            |                   |     .waitFor()
       |            |              Request/
       |            |              Response
       |
   Playwright
   waits for
   actionability
```

------------------------------------------------------------------------

# 27. The One Sentence to Remember Forever

> **Don't ask "Where should I put a wait?" --- ask "What condition am I
> waiting for?"**

That mindset is more important than memorizing every wait API.

------------------------------------------------------------------------

# 28. Your Required Learning Depth

For a QA/SDET role, you do **not** need to spend several days studying
Playwright waits.

Learn these well:

``` text
1. Auto-waiting
2. Actionability
3. Web-first assertions
4. waitForTimeout() — especially why to avoid it
5. locator.waitFor()
6. waitForURL()
7. waitForResponse()
8. waitForRequest()
9. waitForLoadState()
10. networkidle limitations
```

Then practice choosing the correct synchronization method in real
scenarios.

**Master the decision, not the API list.**
