# Playwright API Automation Framework — Final Revision Notes

> **Purpose:** Quick revision of the actual API automation code and its connection to environment variables and `playwright.config.ts`.
>
> **Security:** No real API token or secret is included in these notes. The token is represented only by a placeholder. Never commit a real `.env` file to Git.

## Index

1. [Framework at a glance](#1-framework-at-a-glance) — how the files connect.
2. [Environment variables and dotenv](#2-environment-variables-and-dotenv) — how the QA environment is loaded.
3. [Playwright configuration](#3-playwright-configuration) — browser, reporter, retries, and projects.
4. [APIUtility.ts](#4-apiutilityts) — reusable GET, POST, PUT, and DELETE methods.
5. [apifixtures.ts](#5-apifixturests) — creates the reusable `apiHelper` fixture.
6. [user.apifixture.spec.ts](#6-userapifixturespec-ts) — CRUD API test flow.
7. [Execution commands](#7-execution-commands) — run the tests on Windows.
8. [Execution flow and mental map](#8-execution-flow-and-mental-map) — remember the connections.
9. [What You Actually Need to Remember](#9-what-you-actually-need-to-remember) — fast revision.
10. [Common mistakes and improvement points](#10-common-mistakes-and-improvement-points) — things to check.
11. [Interview questions and scenario questions](#11-interview-questions-and-scenario-questions) — short answers.
12. [Active recall and revision schedule](#12-active-recall-and-revision-schedule) — test your memory.

---

## 1. Framework at a glance

Your shared code uses these files:

| File | Responsibility |
|---|---|
| `playwright.config.ts` | Loads the selected environment file and configures Playwright Test. |
| `config/.env.qa` | Holds QA environment values such as API base URL and credentials/tokens. Keep secrets private. |
| `src/api/APIUtility.ts` | Reusable methods for sending HTTP requests. |
| `src/fixtures/apifixtures.ts` | Creates an `apiHelper` fixture and injects it into tests. |
| `tests/api/user.apifixture.spec.ts` | Tests the user API's GET, POST, PUT, and DELETE operations. |

### Simple meaning

Instead of writing request-handling code inside every test, the framework puts reusable API methods in `APIUtility`. A fixture creates the utility object and supplies it to each test.

**Memory hook:** Config loads settings → fixture builds helper → test calls helper → API returns response → assertions verify the result.

---

## 2. Environment variables and dotenv

### What is an environment variable?

An environment variable stores a value outside the test code. For example, the API base URL can be different in QA and staging environments.

### Environment-loading code from your configuration

```ts
import { config } from "dotenv";

// Read the environment passed through the command line.
// If ENV is not supplied, use "qa".
const currentEnv = process.env.ENV || "qa";

console.log("Current Environment is " + currentEnv);

// Load the matching environment file.
config({ path: `./config/.env.${currentEnv}` });
```

### Line-by-line explanation

- `import { config } from "dotenv"` imports dotenv's configuration function.
- `process.env.ENV` reads the `ENV` operating-system environment variable.
- `|| "qa"` uses `"qa"` when `ENV` is missing or an empty string.
- The template literal builds a path such as `./config/.env.qa`.
- `config(...)` loads values from that file into `process.env`.

### Example `.env.qa` structure — placeholders only

```dotenv
userID=your-test-user@example.com
passWord=replace-with-a-local-secret
API_BASE_URL=https://gorest.co.in
API_TOKEN=REDACTED
```

The email, password, and token above are examples/placeholders, not a working credential set. Keep the actual token in your local environment file only.

### How values are read later

```ts
const baseURL = process.env.API_BASE_URL!;
const TOKEN = process.env.API_TOKEN!;
```

`process.env.API_BASE_URL` and `process.env.API_TOKEN` read values loaded into the Node.js process.

The `!` is TypeScript's **non-null assertion**. It tells TypeScript to assume the value exists. It does **not** check at runtime that the value is present.

### Environment switching on Windows

**PowerShell:**

```powershell
$env:ENV="qa"; npx playwright test
```

**Command Prompt (CMD):**

```bat
set ENV=qa && npx playwright test
```

For a future environment, such as `staging`, create `config/.env.staging` and run with `ENV` set to `staging`. Do this only when that environment file and its settings exist.

### Security rules

- Never paste a real API token into learning notes, screenshots, chat messages, or source code.
- Add `.env.*` files containing secrets to `.gitignore`, while keeping a safe example template such as `.env.example` if needed.
- If a real token has been exposed, revoke or rotate it with the API provider.
- Do not print tokens in logs.
- Consider failing early with a clear error if a required environment variable is missing.

**Memory hook:** `dotenv` loads the file; `process.env` reads the values.

---

## 3. Playwright configuration

### Your configuration, with comments retained and formatting cleaned up

```ts
import { defineConfig, devices } from '@playwright/test';
import { config } from 'dotenv';

// Capture environment details passed from the command line.
// Default to "qa" when ENV is not supplied.
const currentEnv = process.env.ENV || 'qa';

console.log('Current Environment is ' + currentEnv);

// Load the matching environment file.
config({ path: `./config/.env.${currentEnv}` });

export default defineConfig({
  testDir: './tests',

  // Run tests in files in parallel.
  fullyParallel: true,

  // Fail on CI if test.only was accidentally left in the code.
  forbidOnly: !!process.env.CI,

  // Retry failed tests twice on CI; do not retry locally.
  retries: process.env.CI ? 2 : 0,

  // Use one worker on CI; otherwise use Playwright's default.
  ...(process.env.CI ? { workers: 1 } : {}),

  reporter: 'html',

  use: {
    headless: false,
    trace: 'on-first-retry',
    baseURL: 'https://naveenautomationlabs.com/',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },

    // Firefox, WebKit, mobile, and branded-browser projects
    // are currently commented out.
  ],
});
```

### Important settings

| Setting | Meaning in your configuration |
|---|---|
| `testDir: './tests'` | Playwright discovers tests under the `tests` directory. |
| `fullyParallel: true` | Allows tests to run in parallel where the test structure permits it. |
| `forbidOnly: !!process.env.CI` | On CI, fail if a focused `test.only` remains. `!!` converts a value to a boolean. |
| `retries: process.env.CI ? 2 : 0` | Two retries on CI; zero retries when `CI` is not set. |
| `workers` spread | Sets one worker on CI; otherwise leaves the worker count to Playwright defaults. |
| `reporter: 'html'` | Uses the HTML report. |
| `headless: false` | Opens the browser UI for browser tests. API requests do not require a visible browser window. |
| `trace: 'on-first-retry'` | Collects a trace on the first retry. |
| `baseURL` | Default base URL for Playwright operations that use this option, especially browser navigation. |
| `projects` | Defines the browser/device configurations to run. Only Chromium is currently enabled. |

### Important distinction: browser base URL vs API base URL

Your Playwright `use.baseURL` is:

```ts
baseURL: 'https://naveenautomationlabs.com/'
```

Your API fixture uses:

```ts
process.env.API_BASE_URL!
```

These are **two separate settings**:

- `use.baseURL` is a Playwright test option, commonly used with browser navigation such as `page.goto('/')`.
- `API_BASE_URL` is an environment variable passed explicitly into your `APIUtility` constructor.

Changing one does not automatically change the other.

### Why is `config(...)` above `defineConfig(...)`?

The environment file is loaded before the configuration object is exported. This allows code that reads `process.env` later—such as the API fixture and API test—to access the loaded values.

The relative path `./config/.env.${currentEnv}` is resolved from the process's current working directory. Run Playwright from the project root unless you deliberately manage paths differently.

---

## 4. `APIUtility.ts`

### Actual shared implementation

```ts
import { APIRequestContext, APIResponse, request } from "@playwright/test";

export class APIUtility {
  private readonly request: APIRequestContext;
  private readonly baseURL: string;

  constructor(request: APIRequestContext, baseURL: string) {
    this.request = request;
    this.baseURL = baseURL;
  }

  async get(endPoint: string, headers?: Record<string, string>) {
    let response = await this.request.get(`${this.baseURL}${endPoint}`, {
      headers: headers,
    });
    console.log(await response.json(), response.status());
    return { status: response.status(), body: await response.json() };
  }

  async post(
    endPoint: string,
    data: object,
    headers?: Record<string, string>
  ) {
    let response = await this.request.post(`${this.baseURL}${endPoint}`, {
      headers: headers,
      data: data,
    });
    console.log(await response.json(), response.status());
    return { status: response.status(), body: await response.json() };
  }

  async put(
    endPoint: string,
    data: object,
    headers?: Record<string, string>
  ) {
    let response = await this.request.put(`${this.baseURL}${endPoint}`, {
      headers: headers,
      data: data,
    });
    console.log(await response.json(), response.status());
    return { status: response.status(), body: await response.json() };
  }

  async delete(endPoint: string, headers?: Record<string, string>) {
    let response = await this.request.delete(`${this.baseURL}${endPoint}`, {
      headers: headers,
    });
    console.log(response.status());
    return { status: response.status() };
  }
}
```

### Simple explanation

- `APIRequestContext` represents the API request context provided by Playwright.
- `request` is imported but is not used in this class's shown implementation.
- `APIResponse` is also imported but not used in the shown implementation.
- `APIUtility` is a reusable class that groups API methods.
- `private readonly request` stores the request context passed to the constructor.
- `private readonly baseURL` stores the API server's base URL.
- `constructor(...)` receives those dependencies when the helper object is created.
- `endPoint` is the API path, such as `/public/v2/users`.
- `headers` is an optional object for HTTP headers.
- `data` is the request body for POST and PUT.
- `await` waits for the HTTP request to complete.
- The helper returns a simplified object containing `status` and, where applicable, `body`.

### What each method does

| Method | Purpose | Data sent? | Returned data in your code |
|---|---|---|---|
| `get()` | Read resource(s) | No request body passed | `{ status, body }` |
| `post()` | Create a resource | Yes, `data` | `{ status, body }` |
| `put()` | Update a resource | Yes, `data` | `{ status, body }` |
| `delete()` | Delete a resource | No request body passed | `{ status }` |

### How the URL is built

```ts
`${this.baseURL}${endPoint}`
```

If `baseURL` is `https://gorest.co.in` and `endPoint` is `/public/v2/users`, the result is:

```text
https://gorest.co.in/public/v2/users
```

The base URL and endpoint must join correctly. Your example works because the base URL has no trailing slash and the endpoint begins with `/`.

### Why accept headers?

The API test supplies an authorization header:

```ts
let AUTH_HEADER = { Authorization: `Bearer ${TOKEN}` };
```

The helper forwards that object as the request's `headers` option. The token itself is not included in these notes.

### Note about response JSON

Your GET, POST, and PUT methods call `response.json()` for logging and again when building the return object. This may work because Playwright caches the response body, but it is unnecessary to parse it twice. A cleaner version would parse once:

```ts
const response = await this.request.get(`${this.baseURL}${endPoint}`, {
  headers,
});
const body = await response.json();

return { status: response.status(), body };
```

This is an **improvement suggestion**, not a claim that your existing code already uses this version.

---

## 5. `apifixtures.ts`

### Actual shared implementation

```ts
import { APIRequest, test as baseTest, request } from "@playwright/test";
import { APIUtility } from "../api/APIUtility";

type apiInfo = {
  apiHelper: APIUtility;
};

export const test = baseTest.extend<apiInfo>({
  apiHelper: async ({ request }, use) => {
    let apiHelper = new APIUtility(
      request,
      process.env.API_BASE_URL!
    );
    await use(apiHelper);
  },
});

export { expect } from "@playwright/test";
```

### What is a fixture?

A fixture prepares and provides something a test needs. Here, the custom fixture provides an `APIUtility` object named `apiHelper`.

### How it works

1. `baseTest` is Playwright's built-in `test`, renamed during import.
2. `baseTest.extend<apiInfo>(...)` extends the built-in test with a custom fixture.
3. `apiHelper` is the fixture name available to tests.
4. Playwright supplies its built-in `{ request }` fixture to the fixture callback.
5. `new APIUtility(request, process.env.API_BASE_URL!)` creates the helper using the request context and API base URL.
6. `await use(apiHelper)` makes the helper available to the test and marks the fixture's usable lifetime.

The import of `APIRequest` and `request` is not used in the shown fixture code. They can likely be removed after confirming there are no other uses in the actual file.

### How a test receives it

```ts
test('example', async ({ apiHelper }) => {
  const response = await apiHelper.get('/public/v2/users');
});
```

Playwright injects `apiHelper` into the test callback. The test does not need to instantiate `APIUtility` itself.

**Memory hook:** A fixture prepares; `use(...)` supplies; the test consumes.

---

## 6. `user.apifixture.spec.ts`

### Actual shared implementation, with the secret replaced by a placeholder

```ts
import { APIResponse } from "@playwright/test";
import { test, expect } from "../../src/fixtures/apifixtures";
import process from "node:process";

const TOKEN = process.env.API_TOKEN!;
let AUTH_HEADER = { Authorization: `Bearer ${TOKEN}` };
let userId: number;

test.describe.serial('running e2e go rest crud apis tests', () => {
  test('@smoke @regression GET API - get all users', async ({ apiHelper }) => {
    let response = await apiHelper.get('/public/v2/users', AUTH_HEADER);
    expect(response.status).toBe(200);
    expect(response.body.length).toBeGreaterThan(0);
  });

  test('@regression POST API -- create a user', async ({ apiHelper }) => {
    let userData = {
      name: 'manish',
      email: `pwautomation_${Date.now()}@open.com`,
      gender: 'male',
      status: 'active'
    };

    let response = await apiHelper.post(
      '/public/v2/users',
      userData,
      AUTH_HEADER
    );
    expect(response.status).toBe(201);
    userId = response.body.id;
    console.log('created user id : ', userId);
  });

  test('@regression PUT API -- update a user', async ({ apiHelper }) => {
    let userData = {
      name: 'manish automation labs',
      status: 'inactive'
    };

    let response = await apiHelper.put(
      `/public/v2/users/${userId}`,
      userData,
      AUTH_HEADER
    );
    expect(response.status).toBe(200);
    expect(response.body.name).toBe(userData.name);
    expect(response.body.status).toBe(userData.status);
  });

  test('@regression DELETE API -- delete a user', async ({ apiHelper }) => {
    let response = await apiHelper.delete(
      `/public/v2/users/${userId}`,
      AUTH_HEADER
    );
    expect(response.status).toBe(204);
  });
});
```

`APIResponse` is imported but is not used in the shown spec. The `process` import is used to access the environment variable in this code.

### GET — retrieve users

```ts
let response = await apiHelper.get('/public/v2/users', AUTH_HEADER);
expect(response.status).toBe(200);
expect(response.body.length).toBeGreaterThan(0);
```

Checks:
- Status is `200`, meaning the request succeeded according to the expected API contract.
- The response body contains at least one user.

A non-empty list is an appropriate assertion only if the test environment is expected to contain users.

### POST — create a user

```ts
let userData = {
  name: 'manish',
  email: `pwautomation_${Date.now()}@open.com`,
  gender: 'male',
  status: 'active'
};
```

- `userData` is the JSON request body.
- `Date.now()` generates a timestamp in milliseconds, helping create a different email for each run.
- The test expects HTTP `201`, commonly used for successful resource creation.
- `userId = response.body.id` saves the new user's ID for later tests.

### PUT — update the created user

```ts
let userData = {
  name: 'manish automation labs',
  status: 'inactive'
};
```

The endpoint includes the saved ID:

```ts
`/public/v2/users/${userId}`
```

The test expects status `200` and checks that the returned name and status match the submitted values.

### DELETE — delete the created user

The endpoint again uses the saved `userId`. The test expects status `204`, commonly used for a successful response with no content.

### Why `test.describe.serial(...)`?

It declares that tests in this group run serially and in declaration order. This matters because:
- POST creates the user and stores `userId`.
- PUT needs that ID.
- DELETE needs the same ID.

If POST fails, later tests in a serial group are skipped. Serial tests are dependent on shared state, so use them intentionally.

### What do `@smoke` and `@regression` mean?

They are tags embedded in the test titles. They can be used to select test groups with Playwright's `--grep` option.

- `@smoke`: typically a small set of critical checks.
- `@regression`: typically broader checks to detect previously working behavior breaking.

The exact meanings of these tags are team-defined conventions.

---

## 7. Execution commands

Run these commands from the project root.

### Run all tests using the default QA environment

**PowerShell:**

```powershell
npx playwright test
```

Because your config uses `process.env.ENV || "qa"`, it defaults to `qa` if `ENV` is not set.

### Explicitly select QA

**PowerShell:**

```powershell
$env:ENV="qa"; npx playwright test
```

**CMD:**

```bat
set ENV=qa && npx playwright test
```

### Run the API test file only

```powershell
npx playwright test tests/api/user.apifixture.spec.ts
```

### Run only tests tagged smoke

```powershell
npx playwright test --grep "@smoke"
```

### Run tests tagged regression

```powershell
npx playwright test --grep "@regression"
```

### Open the HTML report

```powershell
npx playwright show-report
```

The HTML report is configured as the reporter. A report is available after a test run that generates it.

---

## 8. Execution flow and mental map

```text
Command line (ENV=qa)
        |
        v
playwright.config.ts
        |
        | dotenv loads ./config/.env.qa
        v
process.env.API_BASE_URL + process.env.API_TOKEN
        |                          |
        v                          v
apifixtures.ts                API test spec
        |                          |
        | creates APIUtility       | reads token for auth header
        v                          v
apiHelper --------------------> GET / POST / PUT / DELETE
                                   |
                                   v
                            status + response body
                                   |
                                   v
                            Playwright assertions
```

### The CRUD lifecycle

```text
GET     -> read existing users
POST    -> create a user -> save its ID
PUT     -> update the same user using that ID
DELETE  -> delete the same user using that ID
```

**Memory hook:** Create → save ID → update by ID → delete by ID.

---

## 9. What You Actually Need to Remember

1. **`dotenv` loads** a chosen `.env` file into `process.env`.
2. **`process.env` reads** environment values; TypeScript's `!` does not validate them at runtime.
3. **`playwright.config.ts` configures** test discovery, browser defaults, projects, retries, workers, and reporting.
4. **`APIUtility` centralizes** HTTP request methods so tests do not repeat request code.
5. **The fixture injects** `apiHelper` into the test callback.
6. **The test asserts** the response status and important body fields.
7. **`userId` connects** the POST, PUT, and DELETE tests in the serial CRUD flow.
8. **Browser `use.baseURL` and `API_BASE_URL` are separate values.**
9. **Never hard-code or publish API secrets.**
10. **A passing status assertion alone is not enough:** validate important response fields and relevant business rules too.

---

## 10. Common mistakes and improvement points

### Mistake 1: Environment file not found

**Possible reason:** The relative path is resolved from the current working directory.

**Check:** Run commands from the project root and confirm that `config/.env.qa` exists.

### Mistake 2: `process.env.API_BASE_URL` is undefined

**Possible reasons:**
- dotenv did not load the expected file.
- The variable name differs between the file and code.
- The test command was started from an unexpected directory.

**Check:** Verify the selected environment name and file path. Do not log secret values.

### Mistake 3: Treating `!` as a runtime check

This:

```ts
process.env.API_TOKEN!
```

does not ensure a token exists. A safer pattern is to validate required variables and throw a clear error when missing.

Example improvement:

```ts
const apiToken = process.env.API_TOKEN;

if (!apiToken) {
  throw new Error('API_TOKEN is missing. Check the selected environment file.');
}
```

### Mistake 4: Assuming browser `baseURL` configures APIUtility

It does not in your current code. The API helper receives `process.env.API_BASE_URL` explicitly.

### Mistake 5: Running dependent CRUD tests independently

PUT and DELETE depend on the `userId` created by POST. If you run just the PUT test without creating a user first, `userId` will not have the intended value.

For more independent and robust tests, each test can create its own test data and clean it up, or use an appropriate setup/teardown pattern.

### Mistake 6: Printing secrets

Never log the authorization header or token. Log safe diagnostics such as endpoint, status, and a sanitized error message.

### Mistake 7: Parsing the same response body repeatedly

In GET, POST, and PUT, the current helper calls `response.json()` for logging and again for the return value. Parse it once and reuse the result for clearer code.

### Other code-cleanup observations from the shared snippets

- `APIResponse` and `request` are imported but unused in `APIUtility.ts`.
- `APIRequest` and `request` are imported but unused in `apifixtures.ts`.
- `APIResponse` is imported but unused in the spec.
- Prefer `const` when a variable is not reassigned. For example, `AUTH_HEADER` and `response` can generally be `const`.
- Consider validating environment variables before constructing the helper.
- Consider avoiding logging full API response bodies if they can contain personal or sensitive data.

These are improvement points based on the code shared, not claims that they are already fixed in your project.

---

## 11. Interview questions and scenario questions

### Q1. How do you perform API automation using Playwright?

**Answer:** I use Playwright's API request context to send HTTP requests. I created a reusable API utility for GET, POST, PUT, and DELETE, exposed it through a custom fixture, and use Playwright assertions to validate status codes and response data.

### Q2. Why did you create `APIUtility`?

**Answer:** To centralize reusable request logic. This reduces duplication and makes API tests easier to read and maintain.

### Q3. Why use a custom fixture?

**Answer:** The fixture creates and supplies the API helper to tests. Tests can use `apiHelper` directly instead of creating the utility object themselves.

### Q4. What does dotenv do?

**Answer:** It loads values from an environment file into `process.env`, allowing the same test code to use environment-specific settings.

### Q5. What is the difference between `use.baseURL` and `API_BASE_URL` here?

**Answer:** `use.baseURL` is a Playwright test option. The API helper receives `API_BASE_URL` explicitly from the environment. They are independent in this implementation.

### Q6. Why do you use `test.describe.serial()`?

**Answer:** The POST test creates a user and stores its ID. PUT and DELETE depend on that ID, so the tests are grouped to run in order.

### Q7. Why use `Date.now()` in the email?

**Answer:** It helps generate a unique email value on each run and reduces duplicate-email conflicts.

### Q8. What is the purpose of `expect(response.status).toBe(201)`?

**Answer:** It verifies that the API returned the expected HTTP status for the create operation.

### Q9. What does `process.env.API_TOKEN!` mean?

**Answer:** It tells TypeScript to treat the value as non-null. It does not validate that the token exists at runtime.

### Q10. What would you improve in this framework?

**Answer:** Validate required environment variables, parse each response body once, remove unused imports, avoid logging sensitive data, and make tests independent where practical.

### Scenario: POST succeeds but PUT fails

Check:
1. Did POST return the expected status?
2. Was the response body valid and did it contain `id`?
3. Was `userId` assigned correctly?
4. Is the PUT endpoint using the correct ID?
5. Does the authorization header have the required permissions?
6. What status and safe error details did the API return?

### Scenario: Works locally but fails on CI

Check:
1. Is `ENV` set to the intended environment?
2. Is the matching environment file available in CI, or are secrets configured as CI environment variables?
3. Are required variables present without printing their values?
4. Can CI reach the API?
5. Is the token valid and permitted for that environment?
6. Are rate limits or data dependencies affecting the test?

---

## 12. Active recall and revision schedule

### Try answering without looking above

1. What is the responsibility of `APIUtility.ts`?
2. How does `apiHelper` reach the test?
3. What is the difference between `dotenv` and `process.env`?
4. Why does the POST test save `response.body.id`?
5. Why are the CRUD tests serial?
6. What is the difference between the two base URL settings?
7. Does `!` guarantee that an environment variable exists at runtime?
8. Which status codes does the shared test expect for GET, POST, PUT, and DELETE?
9. Name two ways to protect the API token.
10. Name two improvements you would make to the helper.

### Answer key

1. It centralizes reusable HTTP request methods.
2. The custom Playwright fixture creates the helper and provides it through `use(apiHelper)`.
3. `dotenv` loads a file; `process.env` provides access to environment variables.
4. PUT and DELETE need the created user's ID.
5. Later operations depend on the user created by POST.
6. `use.baseURL` is a Playwright option; `API_BASE_URL` is passed to the API helper from the environment.
7. No. It is a TypeScript assertion, not a runtime validation.
8. GET `200`, POST `201`, PUT `200`, DELETE `204`.
9. Keep secrets in protected environment variables/files, and never commit or log them. Rotate any exposed token.
10. Parse the response body once, validate required environment variables, remove unused imports, or avoid logging sensitive response data.

### Suggested revision schedule

- **Today:** Read sections 1–9 and explain the execution flow aloud.
- **Tomorrow:** Recreate the mental map from memory and answer the active-recall questions.
- **After 3 days:** Explain the fixture and CRUD flow without looking at the code.
- **After 7 days:** Answer the interview questions and describe two framework improvements.

---

## Final 30-second revision

`playwright.config.ts` selects and loads the environment file. `process.env` exposes the API base URL and token to the relevant code. The custom fixture creates `APIUtility` and injects it as `apiHelper`. The spec uses that helper for GET, POST, PUT, and DELETE and asserts the expected statuses and response values. POST saves the created user's ID, which PUT and DELETE reuse. Keep the API token private, validate required environment values, and avoid unnecessary or sensitive logging.
