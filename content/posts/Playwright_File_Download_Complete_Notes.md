# Playwright File Download

## 1. What is File Download Automation?

In a real application, the user may click:

- Download PDF
- Download Excel
- Download CSV
- Export Report
- Download Invoice
- Download Attachment

Our automation should verify that the file is actually downloaded successfully.

In Playwright, we can:

1. Wait for the download event
2. Perform the click that starts the download
3. Capture the downloaded file
4. Get the filename
5. Save the file to a location
6. Verify the download
7. Verify the file

---

# 2. Mental Map

Remember File Download as:

```text
USER CLICKS DOWNLOAD
        ↓
BROWSER STARTS DOWNLOAD
        ↓
PLAYWRIGHT CATCHES DOWNLOAD EVENT
        ↓
DOWNLOAD OBJECT
        ↓
GET FILE NAME
        ↓
SAVE FILE
        ↓
VALIDATE
```

### One-line memory trick

**WAIT → CLICK → CAPTURE → SAVE → VERIFY**

---

# 3. How Does Playwright Know a Download Happened?

Playwright provides the `download` event.

We can listen for it using:

```ts
page.waitForEvent("download")
```

Simple meaning:

> "Playwright, wait until a file download happens."

Example:

```ts
const download = await page.waitForEvent("download");
```

But there is an important problem.

If the download is triggered by a click, we should register the download listener **before** performing the click.

That is why we normally use `Promise.all()`.

---

# 4. Why Do We Use `Promise.all()`?

Suppose this button starts a download:

```ts
await page.getByRole("button", { name: "Download" }).click();
```

We need to do two things:

```text
1. Wait for download
2. Click Download
```

The correct pattern is:

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    page.getByRole("button", { name: "Download" }).click()
]);
```

### Mental Map

```text
             Promise.all()
                  ↓
       ┌──────────┴──────────┐
       ↓                     ↓
WAIT FOR DOWNLOAD          CLICK
       ↓                     ↓
"I am listening"       "Start download"
       └──────────┬──────────┘
                  ↓
          Download happens
                  ↓
          Download object
```

### Why this order?

We want the listener to be ready **before** the click triggers the download.

Think:

```text
LISTEN FIRST
     ↓
CLICK SECOND
     ↓
DOWNLOAD
```

Not:

```text
CLICK
  ↓
DOWNLOAD
  ↓
START LISTENING  ❌
```

---

# 5. Understanding `Promise.all()` in This Scenario

This is one of the most important JavaScript concepts in file downloads.

Consider:

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    downloadButton.click()
]);
```

There are two operations:

### Operation 1

```ts
page.waitForEvent("download")
```

Means:

> Wait for a download.

### Operation 2

```ts
downloadButton.click()
```

Means:

> Trigger the download.

`Promise.all()` waits for both operations.

The first operation gives us the `Download` object.

The click does not give us anything useful here.

Therefore:

```ts
const [download]
```

captures the first result.

---

# 6. Why `[download]`?

`Promise.all()` returns an array.

For example:

```ts
const result = await Promise.all([
    promise1,
    promise2
]);
```

Conceptually:

```text
[result1, result2]
```

For our download:

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    downloadButton.click()
]);
```

Conceptually:

```text
[
    Download object,
    click result
]
```

We only need the first value:

```ts
download
```

So we use array destructuring:

```ts
const [download]
```

### Mental Map

```text
Promise.all()
      ↓
returns array
      ↓
[Download object, click result]
      ↓
we need only first value
      ↓
[download]
```

---

# 7. What is the `Download` Object?

After:

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    downloadButton.click()
]);
```

`download` is a Playwright `Download` object.

It represents the download that Playwright detected.

Now we can perform operations on it.

For example:

```ts
download.suggestedFilename()
download.saveAs()
download.failure()
```

### Mental Map

```text
             Download Object
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
 suggestedFilename saveAs()   failure()
        ↓           ↓           ↓
    file name     save file   check failure
```

---

# 8. Getting the Downloaded Filename

Use:

```ts
download.suggestedFilename()
```

Example:

```ts
const fileName = download.suggestedFilename();

console.log(fileName);
```

Output might be:

```text
sample.pdf
```

### Simple meaning

> Get the filename suggested for this download.

### Mental Map

```text
Download Object
      ↓
suggestedFilename()
      ↓
"sample.pdf"
```

---

# 9. Saving the Downloaded File

Use:

```ts
download.saveAs(filePath);
```

Example:

```ts
await download.saveAs("./download/sample.pdf");
```

Simple meaning:

> Save the downloaded file to this location.

### Mental Map

```text
Download Object
      ↓
   saveAs()
      ↓
Your selected location
      ↓
sample.pdf
```

---

# 10. Very Important: `waitForEvent()` vs `saveAs()`

Do not confuse these two.

### `waitForEvent("download")`

```ts
page.waitForEvent("download")
```

Means:

> Wait for and capture the download event.

### `saveAs()`

```ts
download.saveAs(filePath)
```

Means:

> Save the downloaded file to the location we specify.

### Remember

```text
waitForEvent()
      ↓
CAPTURE

saveAs()
      ↓
SAVE
```

---

# 11. Creating the Download Folder

Suppose we want:

```text
download/
    sample.pdf
```

The folder should exist before saving.

Using Node.js `fs`:

```ts
import * as fs from "fs";

fs.mkdirSync("./download", { recursive: true });
```

### What does this mean?

> Create the `download` folder if it doesn't already exist.

`recursive: true` also allows the required parent folders to be created.

---

# 12. Creating the File Path

You can create a path like:

```ts
const filePath = "./download/" + fileName;
```

This works.

A cleaner Node.js approach is:

```ts
import * as path from "path";

const filePath = path.join("download", fileName);
```

Example:

```text
download + sample.pdf
        ↓
download/sample.pdf
```

### Remember

```text
path.join()
    ↓
Build a proper file path
```

---

# 13. Checking Whether the Download Failed

Playwright provides:

```ts
download.failure()
```

Example:

```ts
expect(await download.failure()).toBeNull();
```

If the download succeeds:

```text
null
```

If the download fails:

```text
failure reason
```

Therefore:

```ts
expect(await download.failure()).toBeNull();
```

means:

> The download should not have failed.

### Mental Map

```text
download.failure()
       ↓
   ┌───┴────┐
   ↓        ↓
 null    failure reason
   ↓        ↓
Success   Failed
```

---

# 14. Checking Whether the File Exists

Playwright itself handles the download, but Node.js can help us verify the actual file.

Import:

```ts
import * as fs from "fs";
```

Then:

```ts
fs.existsSync(filePath)
```

This answers:

> Does a file exist at this path?

Example:

```ts
expect(fs.existsSync(filePath)).toBeTruthy();
```

### Mental Map

```text
filePath
   ↓
existsSync()
   ↓
true / false
```

---

# 15. Checking File Size

We can also verify that the file isn't empty.

```ts
const fileSize = fs.statSync(filePath).size;

expect(fileSize).toBeGreaterThan(0);
```

### What happens?

```ts
fs.statSync(filePath)
```

gets information about the file.

Then:

```ts
.size
```

gets the file size in bytes.

Example:

```text
sample.pdf
   ↓
25,678 bytes
```

Then:

```ts
toBeGreaterThan(0)
```

checks that the file contains data.

### Mental Map

```text
filePath
   ↓
statSync()
   ↓
size
   ↓
> 0 ?
   ↓
YES
```

---

# 16. Complete Practical Example

```ts
import { expect, test } from "@playwright/test";
import * as fs from "fs";
import * as path from "path";

test("File Download", async ({ page }) => {

    await page.goto(
        "https://www.shapemyinterview.com/study/playwright-locator-playground.html?v=2026-07-26"
    );

    // Create download folder if it doesn't exist
    fs.mkdirSync("./download", { recursive: true });

    // Wait for download + trigger download
    const [download] = await Promise.all([
        page.waitForEvent("download"),
        page.getByRole("link", {
            name: "Download sample PDF"
        }).click()
    ]);

    // Get downloaded filename
    const fileName = download.suggestedFilename();

    console.log("Downloaded file:", fileName);

    // Create file path
    const filePath = path.join("download", fileName);

    // Save downloaded file
    await download.saveAs(filePath);

    // Verify download did not fail
    expect(await download.failure()).toBeNull();

    // Verify file exists
    expect(fs.existsSync(filePath)).toBeTruthy();

    // Verify file size
    const fileSize = fs.statSync(filePath).size;

    expect(fileSize).toBeGreaterThan(0);

    console.log("File size:", fileSize, "bytes");
});
```

---

# 17. Execution Flow of the Complete Code

Let's understand the execution instead of memorizing the code.

## Step 1 — Open application

```ts
await page.goto(url);
```

```text
Open page
   ↓
Download link available
```

---

## Step 2 — Create folder

```ts
fs.mkdirSync("./download", { recursive: true });
```

```text
download/
```

is created if necessary.

---

## Step 3 — Start listening for download

```ts
page.waitForEvent("download")
```

Playwright says:

> "I am waiting for a download."

---

## Step 4 — Click Download

```ts
downloadButton.click()
```

Browser starts the download.

---

## Step 5 — Playwright captures it

```text
Download Event
      ↓
Download Object
```

Now:

```ts
download
```

contains the Playwright `Download` object.

---

## Step 6 — Get filename

```ts
download.suggestedFilename()
```

Example:

```text
sample.pdf
```

---

## Step 7 — Create path

```ts
path.join("download", fileName)
```

Result:

```text
download/sample.pdf
```

---

## Step 8 — Save

```ts
await download.saveAs(filePath);
```

Now the file is stored at:

```text
download/sample.pdf
```

---

## Step 9 — Validate

```text
Download failed?
       ↓
      NO

File exists?
       ↓
      YES

File size > 0?
       ↓
      YES

       ↓

     PASS
```

---

# 18. What Exactly Are We Validating?

There are multiple levels of validation.

### Level 1 — Download event occurred

```ts
page.waitForEvent("download")
```

We know Playwright detected a download.

### Level 2 — Download didn't fail

```ts
expect(await download.failure()).toBeNull();
```

### Level 3 — File exists

```ts
expect(fs.existsSync(filePath)).toBeTruthy();
```

### Level 4 — File isn't empty

```ts
expect(fs.statSync(filePath).size).toBeGreaterThan(0);
```

### Mental Map

```text
Download triggered
       ↓
Download captured
       ↓
Download successful
       ↓
File exists
       ↓
File has data
```

---

# 19. Does File Size > 0 Mean the File Is Valid?

No.

This is an important point.

If:

```text
size > 0
```

it only means:

> The file is not empty.

It does NOT guarantee that:

- the PDF is valid
- the Excel file is valid
- the expected data is present
- the file contains the correct business information

For stronger validation, we can inspect the actual file content.

---

# 20. Example: PDF Download Validation

Suppose the requirement is:

> Download an invoice PDF and verify invoice number `INV-1001`.

Basic validation:

```text
Download
   ↓
File exists
   ↓
Size > 0
```

Stronger validation:

```text
Download
   ↓
File exists
   ↓
Size > 0
   ↓
Read PDF
   ↓
Verify invoice number
```

So there are two different goals:

```text
Download Testing
        vs
File Content Testing
```

Don't confuse them.

---

# 21. Download vs Upload

These are completely different operations.

## Download

```text
Application
     ↓
Browser
     ↓
File comes TO us
```

Playwright:

```ts
page.waitForEvent("download")
```

---

## Upload

```text
Our machine
     ↓
Browser
     ↓
Application
```

Playwright commonly uses:

```ts
setInputFiles()
```

or:

```ts
filechooser
```

### Mental Map

```text
DOWNLOAD
App → Browser → File

UPLOAD
File → Browser → App
```

---

# 22. Common Download Scenarios

## Scenario 1 — Normal Download Link

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    page.getByRole("link", {
        name: "Download"
    }).click()
]);
```

---

## Scenario 2 — Download Button

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    page.getByRole("button", {
        name: "Download"
    }).click()
]);
```

---

## Scenario 3 — Download Using Locator

```ts
const downloadButton = page.locator("#download");

const [download] = await Promise.all([
    page.waitForEvent("download"),
    downloadButton.click()
]);
```

The important thing is not whether it is a link or button.

The important thing is:

```text
ACTION triggers download
        ↓
waitForEvent("download")
```

---

# 23. Common Mistakes

## Mistake 1 — Click first, wait later

```ts
await button.click();
const download = await page.waitForEvent("download");
```

Avoid this pattern.

Use:

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    button.click()
]);
```

### Memory

**LISTEN FIRST → ACTION SECOND**

---

## Mistake 2 — Thinking `waitForEvent()` saves the file

It doesn't.

```ts
page.waitForEvent("download")
```

captures the download.

```ts
download.saveAs(path)
```

saves it.

---

## Mistake 3 — Forgetting the download folder

If your code uses:

```text
./download/sample.pdf
```

make sure the folder exists.

```ts
fs.mkdirSync("./download", { recursive: true });
```

---

## Mistake 4 — Checking only file existence

This:

```ts
fs.existsSync(filePath)
```

only tells us:

> File exists.

It doesn't tell us whether the file is empty.

Add:

```ts
fs.statSync(filePath).size
```

---

## Mistake 5 — Thinking size > 0 means correct content

It doesn't.

```text
size > 0
```

means only:

> File contains some data.

For business validation, inspect the actual content.

---

# 24. Interview Questions

## Q1. How do you handle file downloads in Playwright?

### Interview-ready answer

> I use Playwright's `download` event. I register `page.waitForEvent("download")` and trigger the download action together using `Promise.all()`. Once I receive the `Download` object, I use `suggestedFilename()` to get the filename and `saveAs()` to save it to the required location. Then I validate the download using `failure()` and verify that the file exists and has a size greater than zero.

---

## Q2. Why do you use `Promise.all()` for downloads?

### Simple answer

> Because I need to start listening for the download before triggering the action that causes the download. `Promise.all()` lets me wait for the download event while performing the click.

---

## Q3. What does `page.waitForEvent("download")` return?

It gives us a Playwright:

```text
Download object
```

which we can use to:

- get filename
- save the file
- check download failure

---

## Q4. What is `suggestedFilename()`?

It returns the filename suggested for the downloaded file.

Example:

```ts
download.suggestedFilename()
```

might return:

```text
invoice.pdf
```

---

## Q5. What does `saveAs()` do?

```ts
download.saveAs(filePath)
```

saves the downloaded file to the specified path.

---

## Q6. How do you verify that a downloaded file exists?

Using Node.js:

```ts
fs.existsSync(filePath)
```

---

## Q7. How do you verify that the downloaded file is not empty?

```ts
fs.statSync(filePath).size
```

and:

```ts
expect(fileSize).toBeGreaterThan(0);
```

---

# 25. Method Memory Table

| Method | Simple meaning |
|---|---|
| `page.waitForEvent("download")` | Wait for download |
| `download.suggestedFilename()` | Get filename |
| `download.saveAs(path)` | Save downloaded file |
| `download.failure()` | Check download failure |
| `fs.mkdirSync()` | Create folder |
| `fs.existsSync(path)` | Check file exists |
| `fs.statSync(path).size` | Get file size |
| `path.join()` | Build file path |

---

# 26. Final Mental Map

```text
             FILE DOWNLOAD
                   |
                   ↓
        WAIT FOR DOWNLOAD EVENT
                   |
                   ↓
              CLICK ACTION
                   |
                   ↓
          DOWNLOAD STARTS
                   |
                   ↓
          DOWNLOAD OBJECT
                   |
          ┌────────┼────────┐
          ↓        ↓        ↓
     Filename    Save     Failure
          ↓        ↓        ↓
    suggested   saveAs   validate
    Filename
                   |
                   ↓
          FILE ON DISK
                   |
          ┌────────┴────────┐
          ↓                 ↓
      Exists?            Size > 0?
          ↓                 ↓
         YES               YES
          └────────┬────────┘
                   ↓
                 PASS
```

---

# 27. Final Memory Sheet

### Core Pattern

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    downloadButton.click()
]);

const fileName = download.suggestedFilename();

const filePath = path.join(
    "download",
    fileName
);

await download.saveAs(filePath);

expect(await download.failure()).toBeNull();

expect(fs.existsSync(filePath)).toBeTruthy();

expect(fs.statSync(filePath).size).toBeGreaterThan(0);
```

### Remember These 5 Things

```text
1. waitForEvent("download")
   → Catch download

2. Promise.all()
   → Wait + trigger together

3. suggestedFilename()
   → Get filename

4. saveAs()
   → Save file

5. fs
   → Verify file
```

### Final Memory Trick

**WAIT → CLICK → CAPTURE → NAME → SAVE → VERIFY**

---

# 28. Practice Assignment

Build a Playwright test that:

1. Opens the locator playground.
2. Clicks **Download sample PDF**.
3. Captures the download.
4. Prints the filename.
5. Saves it inside `download/`.
6. Verifies `download.failure()` is `null`.
7. Verifies the file exists.
8. Verifies file size is greater than zero.

### Bonus

Extend the test to verify:

```text
filename ends with .pdf
```

and then think about how you would verify actual PDF content.

---

# 29. The Most Important Understanding

Don't memorize the entire code.

Understand this flow:

```text
Something triggers download
          ↓
I must LISTEN for download
          ↓
I get Download object
          ↓
I can ask the object for filename
          ↓
I can save it
          ↓
I can verify it
```

That understanding will allow you to handle different download buttons, links, PDFs, Excel files, CSV files, invoices, reports, and attachments without memorizing separate code for each one.
