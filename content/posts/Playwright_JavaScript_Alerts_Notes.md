# Playwright — JavaScript Alerts / Dialogs

## 1. What are JavaScript Alerts?

JavaScript provides 3 common types of browser dialogs.

### 1. `alert`

Used to show some important information to the user.

Example:

```text
Your session is about to expire.
```

User can only click:

```text
OK
```

### 2. `confirm`

Used when the application wants the user to confirm an action.

Example:

```text
Do you want to delete this record?
```

User can choose:

```text
OK      -> Confirm
Cancel  -> Dismiss
```

### 3. `prompt`

Used when the application wants the user to enter some text.

Example:

```text
Enter your name:
```

User can:

```text
Enter text + OK
OR
Cancel
```

---

# 2. Mental Map

```text
JavaScript Dialogs
       |
       +---- alert
       |       |
       |       +---- Information
       |       +---- OK
       |
       +---- confirm
       |       |
       |       +---- Confirmation
       |       +---- OK / Cancel
       |
       +---- prompt
               |
               +---- Input required
               +---- OK / Cancel
```

### One-line memory trick

```text
ALERT   -> Tell me something       -> accept()
CONFIRM -> Are you sure?           -> accept() / dismiss()
PROMPT  -> Give me some text       -> accept("text") / dismiss()
```

---

# 3. Does Playwright Handle JavaScript Alerts Automatically?

**Yes.**

Playwright automatically handles JavaScript dialogs.

If you don't explicitly register a dialog listener, Playwright will **auto-dismiss the dialog**.

So, we don't normally need special code just to prevent the test from getting stuck.

If we need to **inspect or control** the dialog, we handle it explicitly.

---

# 4. How to Handle a Dialog Explicitly?

There are two important approaches:

```text
1. page.on("dialog")
2. page.waitForEvent("dialog")
```

For normal event-based handling, `page.on("dialog")` is very common.

---

# 5. `page.on("dialog")`

We register a listener for the `dialog` event.

```javascript
page.on("dialog", async (dialog) => {

});
```

Think:

```text
page.on("dialog")
       |
       v
"Playwright, watch for a JavaScript dialog."
       |
       v
dialog object
```

The `dialog` object gives us methods/properties to inspect and control the dialog.

---

# 6. Important Dialog Methods

| Method | Purpose |
|---|---|
| `dialog.accept()` | Clicks OK |
| `dialog.accept("Hello")` | Enters text and clicks OK — mainly for prompt |
| `dialog.dismiss()` | Clicks Cancel |
| `dialog.message()` | Gets the message displayed |
| `dialog.type()` | Gets the dialog type |
| `dialog.defaultValue()` | Gets the default value of a prompt |

## Important correction

There is **no `dialog.type()` for entering text**.

`dialog.type()` tells us the **type of dialog**.

To enter text into a prompt:

```javascript
await dialog.accept("Hello World");
```

---

# 7. `dialog.type()`

Returns the type of JavaScript dialog.

Common values:

```text
"alert"
"confirm"
"prompt"
"beforeunload"
```

Example:

```javascript
page.on("dialog", async (dialog) => {

    console.log(dialog.type());

    await dialog.accept();
});
```

Mental map:

```text
dialog.type()
      |
      +--> alert
      +--> confirm
      +--> prompt
      +--> beforeunload
```

---

# 8. `dialog.message()`

Gets the message displayed inside the dialog.

Example:

```javascript
page.on("dialog", async (dialog) => {

    console.log(dialog.message());

    await dialog.accept();
});
```

If the browser shows:

```text
Do you want to delete this record?
```

Then:

```javascript
dialog.message()
```

returns:

```text
Do you want to delete this record?
```

---

# 9. `dialog.defaultValue()`

Gets the default value of a prompt.

Example:

```javascript
page.on("dialog", async (dialog) => {

    console.log(dialog.message());
    console.log(dialog.defaultValue());

    await dialog.accept("Hello World");
});
```

Mental map:

```text
message()       -> What message is displayed?
defaultValue()  -> What value is already inside the prompt?
```

---

# 10. Handle JavaScript Alert

Example:

```javascript
test("Alert handling", async ({ page }) => {

    await page.goto(
        "https://the-internet.herokuapp.com/javascript_alerts"
    );

    page.on("dialog", async (dialog) => {

        console.log(dialog.message());

        await dialog.accept();
    });

    await page.getByRole("button", {
        name: "Click for JS Alert",
        exact: true
    }).click();

});
```

### Flow

```text
Register listener
       |
       v
Click button
       |
       v
Alert appears
       |
       v
dialog object captured
       |
       +--> message()
       |
       +--> accept()
```

---

# 11. Handle JavaScript Confirm

A confirm dialog has:

```text
OK / Cancel
```

## Click OK

```javascript
page.on("dialog", async (dialog) => {

    console.log(dialog.message());

    await dialog.accept();
});

await page.getByRole("button", {
    name: "Click for JS Confirm",
    exact: true
}).click();
```

## Click Cancel

```javascript
page.on("dialog", async (dialog) => {

    console.log(dialog.message());

    await dialog.dismiss();
});

await page.getByRole("button", {
    name: "Click for JS Confirm",
    exact: true
}).click();
```

Mental map:

```text
CONFIRM
   |
   +---- OK     -> accept()
   |
   +---- Cancel -> dismiss()
```

---

# 12. Handle JavaScript Prompt

A prompt allows the user to enter text.

Example:

```text
What is your name?

[____________]

OK    Cancel
```

Playwright:

```javascript
page.on("dialog", async (dialog) => {

    console.log(dialog.message());

    await dialog.accept("Hello World");
});

await page.getByRole("button", {
    name: "Click for JS Prompt",
    exact: true
}).click();
```

### What does this do?

```javascript
dialog.accept("Hello World");
```

means:

```text
Enter "Hello World"
       |
       v
Click OK
```

If we want to cancel:

```javascript
await dialog.dismiss();
```

---

# 13. Complete Example

```javascript
test("Alert handling", async ({ page }) => {

    await page.goto(
        "https://the-internet.herokuapp.com/javascript_alerts"
    );

    page.on("dialog", async (dialog) => {

        console.log("Dialog Type:", dialog.type());
        console.log("Dialog Message:", dialog.message());

        await dialog.accept("Hello World");
    });

    await page.getByRole("button", {
        name: "Click for JS Prompt",
        exact: true
    }).click();

});
```

---

# 14. `page.on()` vs `page.once()`

If we want to listen for the event every time it occurs:

```javascript
page.on("dialog", async (dialog) => {
    await dialog.accept();
});
```

If we expect the dialog only once:

```javascript
page.once("dialog", async (dialog) => {
    await dialog.accept();
});
```

Mental map:

```text
page.on()
   |
   +--> Keep listening

page.once()
   |
   +--> Listen only once
```

For normal alert handling, `page.on()` is commonly used.

---

# 15. Alternative: `page.waitForEvent("dialog")`

We can also explicitly wait for the dialog event.

Important: prepare the listener **before** performing the action that triggers the dialog.

```javascript
const dialogPromise = page.waitForEvent("dialog");

await button.click();

const dialog = await dialogPromise;

console.log(dialog.message());

await dialog.accept();
```

### Mental Map

```text
PREPARE listener
       |
       v
TRIGGER dialog
       |
       v
CAPTURE dialog
       |
       v
HANDLE dialog
```

---

# 16. Important: Listener Before Trigger

This is one of the most important points.

Correct:

```javascript
page.on("dialog", async (dialog) => {
    await dialog.accept();
});

await button.click();
```

Think:

```text
First  -> Tell Playwright:
          "Watch for a dialog"

Then   -> Click the button
          that opens the dialog
```

With `waitForEvent()`:

```javascript
const dialogPromise = page.waitForEvent("dialog");

await button.click();

const dialog = await dialogPromise;
```

Do not wait for the dialog only **after** triggering the action, because the dialog may appear during the action.

---

# 17. Complete Dialog Method Map

```text
dialog
   |
   +-- type()
   |      |
   |      +--> alert
   |      +--> confirm
   |      +--> prompt
   |      +--> beforeunload
   |
   +-- message()
   |      |
   |      +--> Gets displayed message
   |
   +-- defaultValue()
   |      |
   |      +--> Gets prompt's default value
   |
   +-- accept()
   |      |
   |      +--> Click OK
   |
   +-- accept("text")
   |      |
   |      +--> Enter text + Click OK
   |
   +-- dismiss()
          |
          +--> Click Cancel
```

---

# 18. Quick Comparison

| Dialog | Purpose | User Action | Playwright |
|---|---|---|---|
| `alert` | Show information | OK | `accept()` |
| `confirm` | Confirm action | OK / Cancel | `accept()` / `dismiss()` |
| `prompt` | Get text input | Text + OK / Cancel | `accept("text")` / `dismiss()` |

---

# 19. Interview-Ready Answer

> Playwright handles JavaScript dialogs automatically by dismissing them if no dialog listener is registered. If I need explicit control, I register a `page.on("dialog")` listener before triggering the dialog. From the dialog object, I can use `message()` to read the message, `type()` to identify the dialog type, `defaultValue()` to read a prompt's default value, `accept()` to click OK, `dismiss()` to click Cancel, and `accept("text")` to enter text into a prompt and accept it.

---

# 20. Common Mistakes

### Mistake 1 — Using `type()` to enter text

❌ Wrong:

```javascript
dialog.type("Hello");
```

✅ Correct:

```javascript
dialog.accept("Hello");
```

---

### Mistake 2 — Registering listener after clicking

❌ Avoid:

```javascript
await button.click();

const dialog = await page.waitForEvent("dialog");
```

✅ Correct:

```javascript
const dialogPromise = page.waitForEvent("dialog");

await button.click();

const dialog = await dialogPromise;
```

Or simply:

```javascript
page.on("dialog", async (dialog) => {
    await dialog.accept();
});

await button.click();
```

---

### Mistake 3 — Using `dismiss()` when you want OK

```text
accept()  -> OK
dismiss() -> Cancel
```

---

# 21. Final Memory Sheet

```text
              JAVASCRIPT DIALOG
                     |
       +-------------+-------------+
       |             |             |
     ALERT         CONFIRM       PROMPT
       |             |             |
    Info          Confirm       Input
       |             |             |
     OK          OK / Cancel    Text / OK
       |             |             |
  accept()    accept()/dismiss()  accept("text")
```

## Remember only this

```text
ALERT
-> Information
-> accept()

CONFIRM
-> OK / Cancel
-> accept() / dismiss()

PROMPT
-> Input
-> accept("text") / dismiss()

DIALOG OBJECT
-> type()
-> message()
-> defaultValue()
-> accept()
-> accept("text")
-> dismiss()

IMPORTANT
-> Listener BEFORE trigger
```
