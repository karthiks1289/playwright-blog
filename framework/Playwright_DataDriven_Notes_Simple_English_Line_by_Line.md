# Playwright TypeScript Framework: Data-Driven Testing Revision Notes

## Index

1.  [What are we building?](#1-what-are-we-building)
2.  [How the files connect](#2-how-the-files-connect)
3.  [Project structure](#3-project-structure)
4.  [Understand the common building
    blocks](#4-understand-the-common-building-blocks)
5.  [JSON helper: `JsonHelper.ts`](#5-json-helper-jsonhelperts)
6.  [CSV helper: `CsvHelper.ts`](#6-csv-helper-csvhelperts)
7.  [Excel helper: `ExcelHelper.ts`](#7-excel-helper-excelhelperts)
8.  [Data-driven test:
    `InvalidLogin.spec.ts`](#8-data-driven-test-invalidloginspects)
9.  [How to run the tests](#9-how-to-run-the-tests)
10. [What happens when a test runs?](#10-what-happens-when-a-test-runs)
11. [Common mistakes and
    troubleshooting](#11-common-mistakes-and-troubleshooting)
12. [When should we use JSON, CSV, or
    Excel?](#12-when-should-we-use-json-csv-or-excel)
13. [Interview questions and
    answers](#13-interview-questions-and-answers)
14. [Practice tasks](#14-practice-tasks)
15. [Final memory sheet](#15-final-memory-sheet)
16. [Revision schedule and forget
    log](#16-revision-schedule-and-forget-log)

------------------------------------------------------------------------

## 1. What are we building?

We want to test an application's login page using invalid details.

Examples:

-   Wrong username and wrong password.
-   Empty username.
-   Empty password.

A manual tester might type each combination by hand. In automation, we
can keep the combinations in a data file and let Playwright run the same
test flow using different rows.

### Simple definition

**Data-driven testing** means running the same test logic with different
sets of test data.

### Practical example

  ------------------------------------------------------------------------
  Scenario          Username          Password           Expected result
  ----------------- ----------------- ------------------ -----------------
  Wrong credentials `wrong-user`      `wrong-password`   Invalid username
                                                         or password

  Missing username  *(empty)*         `wrong-password`   Username is
                                                         required

  Missing password  `wrong-user`      *(empty)*          Password is
                                                         required
  ------------------------------------------------------------------------

The exact error messages depend on your application. These are sample
expectations.

**Memory hook:** One test flow + different data rows = multiple test
cases.

## 2. How the files connect

Think of the helper files as people who collect information for the
tester.

``` text
          TEST DATA FILES
       ┌────────┬────────┐
       v        v        v
     JSON      CSV     Excel
       |        |        |
       v        v        v
 JsonHelper  CsvHelper  ExcelHelper
       |        |        |
       └────────┼────────┘
                v
       InvalidLogin.spec.ts
                |
                v
       Open login page
                |
                v
       Enter username/password
                |
                v
       Click Login
                |
                v
       Check expected message
                |
                v
            PASS / FAIL
```

### What each file is responsible for

  -----------------------------------------------------------------------
  File                                Responsibility
  ----------------------------------- -----------------------------------
  `JsonHelper.ts`                     Reads JSON text and returns parsed
                                      data

  `CsvHelper.ts`                      Reads CSV text and returns rows as
                                      objects

  `ExcelHelper.ts`                    Reads a worksheet and returns rows
                                      as objects

  `InvalidLogin.spec.ts`              Uses test data, performs browser
                                      actions, and checks results

  `playwright.config.ts`              Configures settings such as the
                                      base URL, browser, and reports
  -----------------------------------------------------------------------

**Important design rule:** A helper reads data. The spec performs the
test. An assertion checks whether the result is correct.

## 3. Project structure

This is a suggested structure based on the paths you supplied. Keep your
existing structure if it already works.

``` text
Playwright_Framework/
├── src/
│   └── utils/
│       ├── CsvHelper.ts
│       ├── JsonHelper.ts
│       └── ExcelHelper.ts
├── test-data/
│   ├── invalid-login.json
│   ├── invalid-login.csv
│   └── invalid-login.xlsx
├── tests/
│   └── InvalidLogin.spec.ts
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

-   `src/utils`: reusable helper code.
-   `test-data`: the values used by tests.
-   `tests`: test scenarios and assertions.
-   `playwright.config.ts`: shared Playwright settings.

**Why keep data separate from test code?**

If the password or expected message changes, you can update the data
file without rewriting the browser steps.

------------------------------------------------------------------------

## 4. Understand the common building blocks

Before reading the helpers, understand these five pieces.

### 4.1 `import`

``` ts
import fs from 'node:fs';
```

**Simple English:** Bring code from another module into this file so we
can use it.

Here, `fs` is Node.js functionality for working with files.

### 4.2 `readFileSync`

``` ts
const fileContent = fs.readFileSync(fullPath, 'utf-8');
```

**Simple English:** Open the file and read its contents as text.

For a JSON file, the result is still text at this point.

### 4.3 `JSON.parse`

``` ts
const data = JSON.parse(fileContent);
```

**Simple English:** Convert JSON-formatted text into a JavaScript value,
such as an object or array.

Think of it this way:

``` text
File text: '{"username":"wrong-user"}'
                  |
                  | JSON.parse()
                  v
Object: { username: "wrong-user" }
```

`readFileSync()` reads the text. `JSON.parse()` interprets the text.
They do different jobs.

### 4.4 `export` and `import`

In a helper:

``` ts
export function readJson() {
  // Read and return data
}
```

In a spec:

``` ts
import { readJson } from '../src/utils/JsonHelper';
```

**Simple English:** `export` makes the function available to other
files. `import` brings it into the file that needs it.

### 4.5 A TypeScript type

``` ts
type InvalidLoginData = {
  username: string;
  password: string;
  expectedError: string;
};
```

This says that the expected data has three fields, and each should be a
string.

**Important:** A TypeScript type helps during development, but it does
not automatically validate the real contents of a JSON, CSV, or Excel
file at runtime.

**Memory hook:** Import brings code in; export lets code out; a type
describes the expected shape.

------------------------------------------------------------------------

## 5. JSON helper: `JsonHelper.ts`

### 5.1 What is JSON?

JSON is a text format that stores data using names and values. It is
useful for structured data.

Example:

``` json
{
  "username": "wrong-user",
  "password": "wrong-password",
  "expectedError": "Invalid username or password"
}
```

### 5.2 Helper code

Save this as `src/utils/JsonHelper.ts`.

``` ts
import fs from 'node:fs';
import path from 'node:path';

export function readJson<T>(relativePath: string): T {
  const fullPath = path.resolve(process.cwd(), relativePath);
  const fileContent = fs.readFileSync(fullPath, 'utf-8');

  return JSON.parse(fileContent) as T;
}
```

### 5.3 Line-by-line explanation

  ------------------------------------------------------------------------
  Code                                 Simple explanation
  ------------------------------------ -----------------------------------
  `import fs from 'node:fs'`           Gives us Node.js file-reading
                                       functions

  `import path from 'node:path'`       Gives us functions for building
                                       file paths

  `export function readJson<T>(...)`   Creates a reusable function named
                                       `readJson`; `<T>` represents the
                                       expected return type

  `relativePath: string`               The function receives a file path
                                       as text

  `process.cwd()`                      Gets the current working directory,
                                       usually the project root when you
                                       run Playwright there

  `path.resolve(...)`                  Builds an absolute path to the data
                                       file

  `fs.readFileSync(...)`               Reads the file as text

  `JSON.parse(fileContent)`            Converts JSON text into a
                                       JavaScript value

  `as T`                               Tells TypeScript to treat the
                                       result as type `T`; it does not
                                       validate the data at runtime

  `return`                             Sends the result back to the
                                       calling code
  ------------------------------------------------------------------------

### 5.4 Example data file

Create `test-data/invalid-login.json`:

``` json
{
  "username": "wrong-user",
  "password": "wrong-password",
  "expectedError": "Invalid username or password"
}
```

### 5.5 Use the helper

``` ts
type InvalidLoginData = {
  username: string;
  password: string;
  expectedError: string;
};

const data = readJson<InvalidLoginData>(
  'test-data/invalid-login.json'
);

console.log(data.username);
console.log(data.expectedError);
```

Expected output:

``` text
wrong-user
Invalid username or password
```

### 5.6 Connect the pieces

``` text
invalid-login.json
       |
       v
readJson(...)
       |
       v
JSON.parse(...)
       |
       v
data.username / data.password / data.expectedError
       |
       v
InvalidLogin.spec.ts uses the values
```

**Memory hook:** JSON helper = read text -\> parse it -\> return one
data value.

### Common mistakes

-   Invalid JSON syntax: check quotes, commas, and braces.
-   Wrong path: check the file exists and run the command from the
    project root.
-   Wrong expected type: remember that `as T` does not validate the
    file.

------------------------------------------------------------------------

## 6. CSV helper: `CsvHelper.ts`

### 6.1 What is CSV?

CSV means comma-separated values. It is a plain-text format for rows and
columns.

Example:

``` csv
username,password,expectedError
wrong-user,wrong-password,Invalid username or password
,wrong-password,Username is required
wrong-user,,Password is required
```

The first line contains the column names. Each line after it is a test
case.

### 6.2 Install the CSV package

Run from the project root:

``` bash
npm install csv-parse
```

### 6.3 Helper code

Save this as `src/utils/CsvHelper.ts`.

``` ts
import fs from 'node:fs';
import path from 'node:path';
import { parse } from 'csv-parse/sync';

export type CsvRow = Record<string, string>;

export function readCsv(relativePath: string): CsvRow[] {
  const fullPath = path.resolve(process.cwd(), relativePath);
  const fileContent = fs.readFileSync(fullPath, 'utf-8');

  return parse(fileContent, {
    columns: true,
    skip_empty_lines: true,
    trim: true
  }) as CsvRow[];
}
```

### 6.4 Line-by-line explanation

  ------------------------------------------------------------------------------
  Code                                       Simple explanation
  ------------------------------------------ -----------------------------------
  `import { parse } from 'csv-parse/sync'`   Imports a CSV parser that works
                                             synchronously

  `Record<string, string>`                   Describes an object whose keys and
                                             values are strings

  `CsvRow[]`                                 Means an array of CSV row objects

  `columns: true`                            Uses the first CSV row as object
                                             keys

  `skip_empty_lines: true`                   Skips completely empty lines

  `trim: true`                               Removes extra spaces around parsed
                                             field values

  `return parse(...)`                        Converts CSV text into row objects
                                             and returns them
  ------------------------------------------------------------------------------

### 6.5 What does `columns: true` change?

Without using headers, a parsed row may look like:

``` ts
['wrong-user', 'wrong-password', 'Invalid username or password']
```

With `columns: true`, it can look like:

``` ts
{
  username: 'wrong-user',
  password: 'wrong-password',
  expectedError: 'Invalid username or password'
}
```

Now the test can use readable names such as `row.username`.

### 6.6 Use the helper

``` ts
import { readCsv } from '../src/utils/CsvHelper';

const rows = readCsv('test-data/invalid-login.csv');

console.log(rows[0].username);
console.log(rows.length);
```

For the sample CSV, the output is:

``` text
wrong-user
3
```

The exact count assumes the sample contains three data rows and no
additional test cases.

### 6.7 Connect the pieces

``` text
CSV file
   |
   v
readFileSync() -> text
   |
   v
parse(..., { columns: true })
   |
   v
Array of row objects
   |
   v
for...of loop in InvalidLogin.spec.ts
   |
   v
One test for each row
```

**Memory hook:** CSV helper = read text -\> use headers -\> return an
array of objects.

### Common mistakes

-   Header misspelled: `expectedError` must match the name used by the
    spec.
-   Missing package: run `npm install csv-parse`.
-   Commas inside values: use valid CSV quoting, for example
    `"Invalid login, try again"`.
-   Blank cells: decide whether they are valid test inputs. For negative
    login tests, a blank username can be intentional.

------------------------------------------------------------------------

## 7. Excel helper: `ExcelHelper.ts`

### 7.1 When should we use Excel?

Use Excel when testers or business users maintain cases in a workbook.
It can be convenient for spreadsheet-based workflows.

Do not add Excel just because a framework can support it. Use the format
your team can reliably maintain.

### 7.2 Install the package

``` bash
npm install xlsx
```

Before using any third-party package at work, follow your organization's
dependency and security review process.

### 7.3 Example workbook

Create `test-data/invalid-login.xlsx`.

In the first worksheet, put these column names in row 1:

  username     password         expectedError
  ------------ ---------------- ------------------------------
  wrong-user   wrong-password   Invalid username or password
  *(empty)*    wrong-password   Username is required
  wrong-user   *(empty)*        Password is required

Keep the column names consistent.

### 7.4 Helper code

Save this as `src/utils/ExcelHelper.ts`.

``` ts
import fs from 'node:fs';
import path from 'node:path';
import * as XLSX from 'xlsx';

export type ExcelRow = Record<string, string>;

export function readExcel(relativePath: string): ExcelRow[] {
  const fullPath = path.resolve(process.cwd(), relativePath);

  if (!fs.existsSync(fullPath)) {
    throw new Error(`Excel test-data file not found: ${fullPath}`);
  }

  const workbook = XLSX.readFile(fullPath);
  const firstSheetName = workbook.SheetNames[0];

  if (!firstSheetName) {
    throw new Error(`No worksheet found in: ${fullPath}`);
  }

  const worksheet = workbook.Sheets[firstSheetName];

  const rows = XLSX.utils.sheet_to_json<Record<string, unknown>>(
    worksheet,
    { defval: '' }
  );

  return rows.map((row, index) => {
    const normalized: ExcelRow = {};

    for (const [key, value] of Object.entries(row)) {
      normalized[key.trim()] = String(value ?? '').trim();
    }

    if (
      !('username' in normalized) ||
      !('password' in normalized) ||
      !('expectedError' in normalized)
    ) {
      throw new Error(
        `Excel row ${index + 2} must contain username, password, and expectedError columns`
      );
    }

    return normalized;
  });
}
```

### 7.5 Line-by-line explanation

  -----------------------------------------------------------------------
  Code                                Simple explanation
  ----------------------------------- -----------------------------------
  `import * as XLSX from 'xlsx'`      Imports functions for reading Excel
                                      workbooks

  `fs.existsSync(fullPath)`           Checks whether the file exists

  `XLSX.readFile(fullPath)`           Opens and reads the workbook

  `workbook.SheetNames[0]`            Gets the name of the first
                                      worksheet

  `workbook.Sheets[firstSheetName]`   Gets that worksheet

  `sheet_to_json(...)`                Converts worksheet rows into
                                      JavaScript objects

  `{ defval: '' }`                    Uses an empty string for blank
                                      cells

  `rows.map(...)`                     Processes each row and returns a
                                      new array

  `Object.entries(row)`               Gets each column name and cell
                                      value from the row

  `String(value ?? '').trim()`        Converts the value to text, treats
                                      null/undefined as empty, and
                                      removes surrounding spaces

  `throw new Error(...)`              Stops with a clear error when a
                                      required file or column is missing
  -----------------------------------------------------------------------

The sample reads only the **first worksheet**. If your workbook uses a
named sheet, the helper should select that sheet explicitly.

The helper checks required column names, but it does not fully validate
the meaning of every cell. For example, it does not decide whether an
empty password is correct for a particular scenario.

### 7.6 Use the helper

``` ts
import { readExcel } from '../src/utils/ExcelHelper';

const rows = readExcel('test-data/invalid-login.xlsx');

console.log(rows[0].username);
console.log(rows.length);
```

Expected output for the sample workbook:

``` text
wrong-user
3
```

### 7.7 Connect the pieces

``` text
Excel workbook
      |
      v
Open workbook
      |
      v
Select worksheet
      |
      v
Convert rows to objects
      |
      v
Check required column names
      |
      v
Return array of rows
      |
      v
InvalidLogin.spec.ts
```

**Memory hook:** Excel helper = open workbook -\> select sheet -\>
convert rows -\> check columns -\> return data.

------------------------------------------------------------------------

## 8. Data-driven test: `InvalidLogin.spec.ts`

### 8.1 First, understand the job of a spec

The spec does not need to know how a CSV or Excel file is physically
read. It calls a helper, gets the data, and uses the values in browser
steps.

### 8.2 CSV-based example

This is a focused example using the CSV helper. Run one data source at a
time while learning; this makes failures easier to understand.

``` ts
import { test, expect } from '@playwright/test';
import { readCsv } from '../src/utils/CsvHelper';

type InvalidLoginData = {
  username: string;
  password: string;
  expectedError: string;
};

const csvData = readCsv('test-data/invalid-login.csv');

test.describe('Invalid login using CSV data', () => {
  for (const [index, row] of csvData.entries()) {
    test(`rejects invalid login - row ${index + 1}`, async ({ page }) => {
      await page.goto('/login');

      await page.getByLabel('Username').fill(row.username ?? '');
      await page.getByLabel('Password').fill(row.password ?? '');

      await page.getByRole('button', {
        name: /log in|login/i
      }).click();

      await expect(page.getByRole('alert')).toContainText(
        row.expectedError
      );
    });
  }
});
```

### 8.3 Explain the test in simple English

  ------------------------------------------------------------------------
  Code                                 Simple explanation
  ------------------------------------ -----------------------------------
  `import { test, expect } ...`        Imports Playwright's test runner
                                       and assertion functions

  `import { readCsv } ...`             Imports the function that reads the
                                       CSV file

  `readCsv(...)`                       Loads the test rows before test
                                       definitions are registered

  `test.describe(...)`                 Groups related tests under one name

  `csvData.entries()`                  Gives each row and its zero-based
                                       index

  `test(...)`                          Registers a separate test for each
                                       row

  `async ({ page })`                   Receives Playwright's `page`
                                       fixture for browser actions

  `page.goto('/login')`                Opens the login path on the
                                       configured base URL

  `getByLabel(...).fill(...)`          Finds a field by its accessible
                                       label and fills it

  `getByRole('button', ...).click()`   Finds the login button by its
                                       accessible role/name and clicks it

  `expect(...).toContainText(...)`     Checks that the error element
                                       contains the expected message
  ------------------------------------------------------------------------

### 8.4 What is `page`?

`page` is a Playwright fixture representing a browser tab. Playwright
Test provides it to the test function.

### 8.5 What is `expect`?

`expect` checks whether the application behaved as expected. If the
actual result does not match the expected result, the test fails.

### 8.6 Why do we use a loop?

If the CSV contains three rows, the loop registers three tests. Each
test gets its own row and executes the same basic steps with different
data.

``` text
CSV row 1 -> Test 1
CSV row 2 -> Test 2
CSV row 3 -> Test 3
```

### 8.7 Why use `row.username ?? ''`?

A missing value may be `undefined` or `null`. The `?? ''` expression
supplies an empty string in those cases. An empty CSV cell is also a
valid empty string, which is the desired input for a missing-username
scenario.

### 8.8 Important application-specific details

The following values are examples and must match your actual
application:

-   `/login`
-   `Username`
-   `Password`
-   Login button accessible name
-   `getByRole('alert')`
-   Expected error messages

If your application displays the error in a normal paragraph instead of
an alert, use a locator that matches the actual page.

### 8.9 JSON-based test example

The test flow stays almost the same; only the data source changes.

``` ts
import { test, expect } from '@playwright/test';
import { readJson } from '../src/utils/JsonHelper';

type InvalidLoginData = {
  username: string;
  password: string;
  expectedError: string;
};

const data = readJson<InvalidLoginData>(
  'test-data/invalid-login.json'
);

test('rejects invalid login using JSON data', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Username').fill(data.username);
  await page.getByLabel('Password').fill(data.password);

  await page.getByRole('button', {
    name: /log in|login/i
  }).click();

  await expect(page.getByRole('alert')).toContainText(data.expectedError);
});
```

### 8.10 Excel-based test example

``` ts
import { test, expect } from '@playwright/test';
import { readExcel } from '../src/utils/ExcelHelper';

const excelData = readExcel('test-data/invalid-login.xlsx');

test.describe('Invalid login using Excel data', () => {
  for (const [index, row] of excelData.entries()) {
    test(`rejects invalid login - Excel row ${index + 1}`, async ({ page }) => {
      await page.goto('/login');
      await page.getByLabel('Username').fill(row.username ?? '');
      await page.getByLabel('Password').fill(row.password ?? '');

      await page.getByRole('button', {
        name: /log in|login/i
      }).click();

      await expect(page.getByRole('alert')).toContainText(
        row.expectedError
      );
    });
  }
});
```

**Learning advice:** Do not put JSON, CSV, and Excel versions into the
same spec file at the beginning. Learn and run one version first, then
switch the import and data loading when you practise another format.

### 8.11 A note about the type

In the CSV and Excel examples, the returned row type is broad
(`Record<string, string>`). For a stronger framework, validate the data
and convert it into `InvalidLoginData[]` after checking all required
fields. A type annotation alone does not guarantee the file contains
those fields.

**Memory hook:** The helper supplies data; the spec uses it; assertions
verify the outcome.

------------------------------------------------------------------------

## 9. How to run the tests

Run commands from your project root:

``` bash
cd C:\Users\karth\Playwright_Framework
```

Run the CSV spec:

``` bash
npx playwright test tests/InvalidLogin.spec.ts
```

Run with a visible browser:

``` bash
npx playwright test tests/InvalidLogin.spec.ts --headed
```

Open the HTML report after a test run:

``` bash
npx playwright show-report
```

### `baseURL` requirement

Because the examples use `page.goto('/login')`, your
`playwright.config.ts` must define a valid `baseURL`.

Example:

``` ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'https://your-test-application.example',
    ...devices['Desktop Chrome']
  }
});
```

Replace the sample URL with your application's test URL. Do not
overwrite other existing config settings unnecessarily.

### Install packages used in these notes

``` bash
npm install csv-parse xlsx
```

The examples also use Node.js built-in modules. If TypeScript reports
Node type errors, check that Node typings are installed and your
TypeScript configuration is suitable:

``` bash
npm install --save-dev @types/node
```

Do not initialize an existing Playwright project again just to use these
helpers.

------------------------------------------------------------------------

## 10. What happens when a test runs?

Trace this flow from beginning to end:

1.  Playwright loads the test file.
2.  The spec imports the helper.
3.  The helper reads the data file.
4.  The helper returns an object or array of objects.
5.  Playwright registers the tests.
6.  Each test opens a browser page and enters the row's data.
7.  The test submits the form.
8.  The assertion checks the expected result.
9.  Playwright records Pass or Fail.

``` text
Read file
   |
   v
Parse / convert data
   |
   v
Register test cases
   |
   v
Browser actions
   |
   v
Assertion
   |
   v
Test report
```

**Important:** The test data is read before the test cases are
registered. If the file cannot be read or parsed, the spec may fail
before any browser test begins.

------------------------------------------------------------------------

## 11. Common mistakes and troubleshooting

  -------------------------------------------------------------------------
  Symptom                 Likely reason           What to check
  ----------------------- ----------------------- -------------------------
  Cannot find             Package is missing      Run
  `csv-parse/sync`                                `npm install csv-parse`

  Cannot find `xlsx`      Package is missing      Run `npm install xlsx`

  `ENOENT`                File path is incorrect  Check the exact file name
                          or working directory    and run from project root
                          differs                 

  JSON parse error        JSON syntax is invalid  Check quotes, commas, and
                                                  braces

  No worksheet found      Workbook has no sheets  Open the workbook and
                                                  check

  Required column missing Header does not match   Check exact header
                                                  spelling

  `page.goto('/login')`   `baseURL` missing or    Check Playwright config
  fails                   wrong                   

  Locator not found       Selector does not match Inspect the actual labels
                          the page                and elements

  Assertion fails         Expected text or app    Inspect actual message
                          behavior differs        and scenario

  Test registers zero     Data file has headers   Check the input file
  rows                    only or no data         

  TypeScript accepts bad  Types are not runtime   Add explicit data
  file data               validation              validation
  -------------------------------------------------------------------------

### Troubleshooting method

When a test fails, ask in this order:

1.  Did the file load?
2.  Did the helper return the expected data?
3.  Did the browser open the correct page?
4.  Did the locators find the right fields and button?
5.  Did the application display the expected message?
6.  Did the assertion check the right element and text?

This sequence avoids blaming the helper when the real problem is a
locator or application response.

------------------------------------------------------------------------

## 12. When should we use JSON, CSV, or Excel?

  ------------------------------------------------------------------------------
  Question          JSON              CSV               Excel
  ----------------- ----------------- ----------------- ------------------------
  Data format       Structured text   Rows and columns  Workbook with worksheets

  Typical helper    Parsed            Array of objects  Array of objects
  output            value/object or                     
                    array                               

  Extra package in  No extra parser   `csv-parse`       `xlsx`
  these examples    package                             

  Best fit          Structured or     Simple tabular    Spreadsheet-maintained
                    nested test data  data              test cases

  Common concern    JSON syntax and   Headers, commas,  Worksheet selection,
                    expected shape    empty cells       headers, cell values
  ------------------------------------------------------------------------------

### Mentor's recommendation

Start with JSON to understand reading and parsing. Then learn CSV
because it makes multiple test rows easy. Learn Excel when your team's
workflow needs workbook-based data.

You do not need all three formats in every framework. They are
alternatives, not mandatory layers.

------------------------------------------------------------------------

## 13. Interview questions and answers

### Q1. What is data-driven testing?

**Answer:** Data-driven testing runs the same test logic with multiple
data sets. For example, I can test wrong credentials, missing username,
and missing password without duplicating the browser steps.

### Q2. Why do you use helper files?

**Answer:** Helpers keep file-reading logic reusable and separate from
test logic. Multiple specs can use the same helper instead of repeating
the parsing code.

### Q3. What is the difference between `readFileSync()` and `JSON.parse()`?

**Answer:** `readFileSync()` reads the file as text. `JSON.parse()`
converts valid JSON text into a JavaScript value.

### Q4. Why does the CSV helper return an array?

**Answer:** A CSV file can contain many data rows. Returning an array
lets the test iterate through each row.

### Q5. What does `columns: true` do?

**Answer:** It treats the first CSV row as column names and uses those
names as keys in each returned object.

### Q6. How does Excel data-driven testing work?

**Answer:** The helper opens the workbook, selects a worksheet, converts
its rows into objects, and returns them. The spec loops through the rows
and runs the test flow for each row.

### Q7. Does a TypeScript type validate the actual JSON or Excel data?

**Answer:** No. Types help during development, but the file must be
validated at runtime if we need to guarantee its structure and values.

### Q8. Why use a separate test for each row?

**Answer:** Each scenario gets its own result in the report, which makes
failures easier to identify. Each test should be independent.

### Q9. What if the helper works but the test fails?

**Answer:** I check the returned data, page URL, locators, actual
application response, and assertion. The problem might not be in the
helper.

### Q10. How do you choose between JSON, CSV, and Excel?

**Answer:** I choose based on the data structure and who maintains it.
JSON suits structured data, CSV suits simple tabular rows, and Excel
suits spreadsheet-based workflows.

------------------------------------------------------------------------

## 14. Practice tasks

Do these in order. Try first without looking at the solutions above.

### Task 1 --- JSON

-   Change the username in `invalid-login.json`.
-   Print the username and expected error.
-   Explain the journey from file to `console.log()`.

### Task 2 --- CSV

-   Add one new invalid-login row.
-   Run the test and confirm an additional test is registered.
-   Explain what `columns: true` does.

### Task 3 --- Excel

-   Add a fourth row to the workbook.
-   Run the Excel-based spec.
-   Confirm that the new row appears as a separate test.

### Task 4 --- Troubleshooting

Intentionally misspell `expectedError` in the CSV header. Observe the
failure and explain why the helper/test cannot find the expected
property. Then correct the header.

### Task 5 --- Interview practice

Explain this sentence aloud in your own words:

> The helper reads test data, the spec uses that data to perform browser
> actions, and Playwright assertions verify the outcome.

------------------------------------------------------------------------

## 15. Final memory sheet

### Remember these seven points

1.  **JSON helper:** read file -\> parse JSON -\> return a value.
2.  **CSV helper:** read file -\> parse using headers -\> return an
    array of objects.
3.  **Excel helper:** open workbook -\> select worksheet -\> convert
    rows -\> return objects.
4.  **Spec:** load data -\> open page -\> fill fields -\> submit -\>
    assert.
5.  **Data-driven testing:** one test flow can run against multiple data
    sets.
6.  **Types:** TypeScript types describe the expected shape; they do not
    validate real file contents at runtime.
7.  **Debugging:** verify the data, page, locators, actual message, and
    assertion in that order.

### One mental map

``` text
SOURCE -> HELPER -> DATA -> SPEC -> ASSERTION -> REPORT
```

-   **Source:** JSON, CSV, or Excel.
-   **Helper:** Reads and converts the source.
-   **Data:** Values returned to the spec.
-   **Spec:** Performs browser actions.
-   **Assertion:** Checks actual versus expected behavior.
-   **Report:** Shows Pass or Fail.

**One-line memory hook:** Source -\> Helper -\> Test -\> Check -\>
Report.

------------------------------------------------------------------------

## 16. Revision schedule and forget log

### Spaced revision

  -----------------------------------------------------------------------
  Day                                 What to do
  ----------------------------------- -----------------------------------
  Day 1                               Read the mental map and trace one
                                      JSON test

  Day 3                               Re-explain CSV parsing and
                                      `columns: true` without looking

  Day 7                               Re-create one helper from memory
                                      and run a test

  Day 14                              Explain all components in a
                                      two-minute interview answer

  Day 30                              Add a new test row, run the test,
                                      and troubleshoot one intentional
                                      mistake
  -----------------------------------------------------------------------

### Forget log

Whenever you forget something, record it here:

  Date   What I forgot   Correct explanation   Review date
  ------ --------------- --------------------- -------------
                                               
                                               
                                               

Revisit the items you forgot during the next revision instead of
re-reading every page from the beginning.

------------------------------------------------------------------------

## Important project note

These are reference implementations based on the file paths you
supplied, not a review of the actual local file contents. The sample
login URL, accessible labels, button name, and error locator must match
your application. Check the dependency and TypeScript configuration in
your existing project before adopting the code. The examples have not
been executed against your application.
