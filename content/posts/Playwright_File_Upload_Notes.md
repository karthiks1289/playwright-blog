# Playwright File Upload

## 1. What is File Upload?

In Playwright, there are **2 main ways** to handle file uploads.

```text
                    FILE UPLOAD
                         │
             ┌───────────┴───────────┐
             │                       │
     Can locate the            Cannot directly
     file input?               access file input
             │                       │
            YES                      NO
             │                       │
             ↓                       ↓
   setInputFiles()            filechooser event
                                     │
                                     ↓
                                  setFiles()
```

### ⭐ Golden Rule

> **Locator → `setInputFiles()`**  
> **FileChooser → `setFiles()`**

---

# 2. Scenario 1 — `<input type="file">`

Suppose HTML is:

```html
<input type="file" id="single-file">
```

We can directly locate the file input.

```typescript
const fileUpload = page.locator("#single-file");

await fileUpload.setInputFiles("./demo.txt");
```

That's it.

### Mental Map

```text
<input type="file">
       ↓
   Locator
       ↓
setInputFiles()
       ↓
   File uploaded
```

---

# 3. `setInputFiles()`

The method is:

```typescript
locator.setInputFiles()
```

### Upload one file

```typescript
await page.locator("#single-file")
    .setInputFiles("./demo.txt");
```

### Upload multiple files

If the input supports multiple files:

```html
<input type="file" id="multi-file" multiple>
```

Use:

```typescript
await page.locator("#multi-file").setInputFiles([
    "./demo.txt",
    "./demotwo.txt"
]);
```

### Clear uploaded files

```typescript
await page.locator("#single-file").setInputFiles([]);
```

For multiple:

```typescript
await page.locator("#multi-file").setInputFiles([]);
```

### Mental Map

```text
One file
    ↓
setInputFiles("file")


Multiple files
    ↓
setInputFiles(["file1", "file2"])


Clear files
    ↓
setInputFiles([])
```

---

# 4. Important: `multiple` Attribute

Do not confuse:

```html
type="file"
```

with:

```html
multiple
```

They have different purposes.

```text
type="file"
     ↓
This is a file input


multiple
     ↓
This input can accept multiple files
```

Example:

```html
<input type="file" id="upload" multiple>
```

Then:

```typescript
await page.locator("#upload").setInputFiles([
    "./file1.txt",
    "./file2.txt"
]);
```

---

# 5. Scenario 2 — Custom Upload Button

Sometimes the page does not give us a file input that we can directly locate.

For example:

```html
<button id="custom-upload-btn">
    Upload File
</button>
```

When we click:

```text
Click Upload
      ↓
Browser opens File Chooser
      ↓
Playwright raises "filechooser" event
```

In this situation we use:

```typescript
page.waitForEvent("filechooser")
```

---

# 6. What is `filechooser`?

Very simply:

> `filechooser` is the Playwright event that tells us that the browser has opened a file-selection dialog.

Mental map:

```text
Click Upload Button
        ↓
File Chooser opens
        ↓
"filechooser" event
        ↓
Playwright catches event
        ↓
FileChooser object
        ↓
setFiles()
```

---

# 7. FileChooser Object

When we wait for the event:

```typescript
const fileChooser = await page.waitForEvent("filechooser");
```

Playwright eventually gives us a:

```text
FileChooser object
```

Then we can use:

```typescript
await fileChooser.setFiles("./demo.txt");
```

For multiple files:

```typescript
await fileChooser.setFiles([
    "./demo.txt",
    "./demotwo.txt"
]);
```

To clear:

```typescript
await fileChooser.setFiles([]);
```

---

# 8. Why `Promise.all()` is Used

This is the important JavaScript part.

Consider:

```typescript
const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    page.locator("#custom-upload-btn").click()
]);
```

There are 3 important things here:

```text
Promise.all()
await
[fileChooser]
```

Let's understand them separately.

---

# 9. What is a Promise?

Simple definition:

> A Promise represents an asynchronous operation whose result will be available later.

Example:

```typescript
page.waitForEvent("filechooser")
```

doesn't immediately give us the FileChooser.

It gives us a Promise:

```text
waitForEvent()
      ↓
   Promise
      ↓
Waiting...
      ↓
filechooser event happens
      ↓
FileChooser object
```

---

# 10. What does `await` mean?

Think of `await` as:

> **"Wait here until this Promise gives me its result."**

Example:

```typescript
const fileChooser = await page.waitForEvent("filechooser");
```

Execution is conceptually:

```text
waitForEvent()
      ↓
Promise
      ↓
WAIT
      ↓
Event happens
      ↓
FileChooser object
      ↓
Continue to next statement
```

---

# 11. What is `Promise.all()`?

Simple definition:

> `Promise.all()` starts multiple asynchronous operations and waits until **all of them complete**.

Example:

```typescript
const results = await Promise.all([
    operation1(),
    operation2()
]);
```

Mental map:

```text
                  Promise.all()
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       operation1()        operation2()
             ↓                   ↓
          running             running
             ↓                   ↓
          finishes            finishes
             └─────────┬─────────┘
                       ↓
                Promise.all()
                  completes
                       ↓
                    results
```

---

# 12. Very Important — They Are Not Sequential

Don't think:

```text
operation1()
    ↓
finish
    ↓
operation2()
    ↓
finish
```

That is NOT what `Promise.all()` is for.

Think:

```text
operation1() ─────────────→ finishes
                            \
                             → Promise.all()
                            /
operation2() ───────→ finishes
```

Both operations are started without waiting for each other.

`Promise.all()` waits for all of them.

---

# 13. Why `Promise.all()` is Perfect for FileChooser

Our code:

```typescript
const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    page.locator("#custom-upload-btn").click()
]);
```

We have two asynchronous operations.

### Operation 1

```typescript
page.waitForEvent("filechooser")
```

Means:

> Start listening for the filechooser event.

### Operation 2

```typescript
page.locator("#custom-upload-btn").click()
```

Means:

> Click the upload button.

We need the listener to be active **before the click triggers the event**.

---

# 14. Correct Execution Flow

```text
START
  ↓
Promise.all()
  ↓
Start waiting for "filechooser"
  ↓
Start clicking Upload button
  ↓
Upload button clicked
  ↓
File chooser opens
  ↓
"filechooser" event occurs
  ↓
waitForEvent() catches it
  ↓
FileChooser object becomes available
  ↓
click() completes
  ↓
Promise.all() waits until both are complete
  ↓
Promise.all() returns results
  ↓
[fileChooser] takes first result
  ↓
fileChooser.setFiles()
  ↓
File uploaded
```

---

# 15. Why NOT This?

Don't do this:

```typescript
await page.locator("#custom-upload-btn").click();

const fileChooser =
    await page.waitForEvent("filechooser");
```

Because:

```text
Click
  ↓
File chooser opens
  ↓
filechooser event occurs
  ↓
Event is already handled
  ↓
NOW start waiting ❌
```

You started listening too late.

---

# 16. Correct Pattern

Instead:

```typescript
const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    page.locator("#custom-upload-btn").click()
]);

await fileChooser.setFiles("./demo.txt");
```

Mental map:

```text
LISTEN + CLICK
     ↓
EVENT OCCURS
     ↓
CAPTURE EVENT
     ↓
GET FILECHOOSER
     ↓
SET FILE
```

---

# 17. Understanding `[fileChooser]`

This is JavaScript array destructuring.

`Promise.all()` returns an array.

For example:

```typescript
const results = await Promise.all([
    promise1,
    promise2
]);
```

The result looks conceptually like:

```typescript
[
    resultOfPromise1,
    resultOfPromise2
]
```

So we can write:

```typescript
const results = await Promise.all([
    promise1,
    promise2
]);

const firstResult = results[0];
```

Or more conveniently:

```typescript
const [firstResult] = await Promise.all([
    promise1,
    promise2
]);
```

These are basically equivalent.

---

# 18. In Our File Upload Example

```typescript
const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    page.locator("#custom-upload-btn").click()
]);
```

Conceptually:

```typescript
[
    FileChooser object,
    click result
]
```

We only need the first result.

Therefore:

```typescript
[fileChooser]
```

means:

> Give me the first item from the returned array.

---

# 19. Complete Example — Recommended Pattern

```typescript
test("File Upload using FileChooser", async ({ page }) => {

    await page.goto(
        "https://naveenautomationlabs.com/opencart/ui/file-upload.html"
    );

    const [fileChooser] = await Promise.all([
        page.waitForEvent("filechooser"),
        page.locator("#custom-upload-btn").click()
    ]);

    await fileChooser.setFiles("./demo.txt");
});
```

---

# 20. `setInputFiles()` vs `setFiles()`

This is one of the most important things to remember.

| Situation | Object | Method |
|---|---|---|
| Direct `<input type="file">` | Locator | `setInputFiles()` |
| File chooser event | FileChooser | `setFiles()` |

### Mental Map

```text
<input type="file">
       ↓
    Locator
       ↓
setInputFiles()


Custom Upload Button
       ↓
filechooser event
       ↓
FileChooser object
       ↓
setFiles()
```

---

# 21. Important Correction to the Original Understanding

Don't use this rule:

> "If there is no `type="file"` attribute, use `filechooser`."

The better rule is:

> **Can I directly access the underlying file input?**

For example, the input could be hidden:

```html
<input type="file" id="real-upload" hidden>
<button>Upload</button>
```

Even though the user sees only the button, Playwright can potentially interact directly with the file input:

```typescript
await page.locator("#real-upload")
    .setInputFiles("./demo.txt");
```

So:

```text
Can I locate the file input?
        │
       YES
        ↓
setInputFiles()
        │
       NO
        ↓
Does clicking the upload control
trigger a filechooser event?
        │
       YES
        ↓
waitForEvent("filechooser")
        ↓
FileChooser
        ↓
setFiles()
```

---

# 22. Common Mistakes

## Mistake 1 — Using `setFiles()` on a Locator

❌

```typescript
await page.locator("#upload")
    .setFiles("./demo.txt");
```

If you're working with a Locator, use:

✅

```typescript
await page.locator("#upload")
    .setInputFiles("./demo.txt");
```

---

## Mistake 2 — Using `setInputFiles()` on FileChooser

❌

```typescript
await fileChooser.setInputFiles("./demo.txt");
```

Use:

✅

```typescript
await fileChooser.setFiles("./demo.txt");
```

---

## Mistake 3 — Starting the listener after clicking

❌

```typescript
await uploadButton.click();

const fileChooser =
    await page.waitForEvent("filechooser");
```

Potentially too late.

Use:

✅

```typescript
const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    uploadButton.click()
]);
```

---

## Mistake 4 — Forgetting `await`

❌

```typescript
fileChooser.setFiles([]);
```

Better:

✅

```typescript
await fileChooser.setFiles([]);
```

---

## Mistake 5 — Clearing the wrong locator

If you upload to:

```typescript
await page.locator("#multi-file")
    .setInputFiles(["file1", "file2"]);
```

Then clear the same input:

```typescript
await page.locator("#multi-file")
    .setInputFiles([]);
```

Not:

```typescript
await page.locator("#single-file")
    .setInputFiles([]);
```

---

# 23. `page.on()` vs `page.waitForEvent()` — Quick Connection

### `page.waitForEvent()`

Think:

> **"I need this one event right now."**

```typescript
const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    uploadButton.click()
]);
```

It waits for a specific event.

### `page.on()`

Think:

> **"Whenever this event happens, run this listener."**

Example:

```typescript
page.on("dialog", dialog => {
    console.log(dialog.message());
});
```

Mental map:

```text
waitForEvent()
     ↓
WAIT FOR ONE OCCURRENCE


page.on()
     ↓
KEEP LISTENING
```

---

# 24. Do You Always Need `Promise.all()`?

No.

You need it when an action **triggers an event that you need to capture**.

Common pattern:

```text
WAIT FOR EVENT
      +
TRIGGER ACTION
      ↓
Promise.all()
```

Examples:

```typescript
Promise.all([
    page.waitForEvent("filechooser"),
    uploadButton.click()
]);
```

```typescript
Promise.all([
    page.waitForEvent("popup"),
    page.getByRole("button", { name: "Open" }).click()
]);
```

```typescript
Promise.all([
    page.waitForEvent("download"),
    page.getByText("Download").click()
]);
```

---

# 25. Real Project Pattern

Keep the locator separate when it improves readability:

```typescript
const uploadButton =
    page.locator("#custom-upload-btn");

const [fileChooser] = await Promise.all([
    page.waitForEvent("filechooser"),
    uploadButton.click()
]);

await fileChooser.setFiles("./test-data/demo.txt");
```

---

# 26. File Paths

You can provide a relative path:

```typescript
await fileUpload.setInputFiles("./demo.txt");
```

Or a path from your test-data folder:

```typescript
await fileUpload.setInputFiles("./test-data/demo.txt");
```

For multiple:

```typescript
await fileUpload.setInputFiles([
    "./test-data/file1.txt",
    "./test-data/file2.txt"
]);
```

A good framework structure could be:

```text
project
│
├── tests
│   └── fileUpload.spec.ts
│
├── test-data
│   ├── demo.txt
│   ├── file1.txt
│   └── file2.txt
│
└── playwright.config.ts
```

---

# 27. Interview-Ready Answer

### Q: How do you handle file upload in Playwright?

> "If I can directly locate an `<input type='file'>`, I use `setInputFiles()` on the locator. For multiple files, I pass an array of file paths. If the upload is triggered through a custom button and the file input isn't directly accessible, I listen for the `filechooser` event before clicking the button, get the `FileChooser` object, and use `setFiles()` to upload the file."

### Q: Why `Promise.all()`?

> "I use `Promise.all()` because I need to start listening for the `filechooser` event before clicking the upload button. The click triggers the event, and `Promise.all()` allows me to wait for both the event and the click to complete."

---

# 28. Final Mental Map 🧠

```text
                 FILE UPLOAD
                      │
          ┌───────────┴───────────┐
          │                       │
   Direct file input        Custom upload
          │                       │
          ↓                       ↓
   <input type="file">       Click button
          │                       │
          ↓                       ↓
       Locator              filechooser event
          │                       │
          ↓                       ↓
setInputFiles()              FileChooser
          │                       │
          ↓                       ↓
      Upload                    setFiles()
```

## Promise Mental Map

```text
Promise
   ↓
"I will give you the result later"


await
   ↓
"Wait for that result"


Promise.all()
   ↓
"Start multiple async operations
 and wait until ALL finish"


[fileChooser]
   ↓
"Take the first result from the array"
```

## ⭐ 5 Things to Remember

```text
1. Locator → setInputFiles()

2. FileChooser → setFiles()

3. Multiple files → ["file1", "file2"]

4. Clear files → []

5. Event + action → Promise.all([
       waitForEvent(),
       action()
   ])
```

### 🔥 One-line memory trick

**"Input → setInputFiles | Chooser → setFiles | Event + Action → Promise.all"**
