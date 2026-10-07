# Dotenv in Playwright — Simple Practical Notes

## 1. What Problem Does `dotenv` Solve?

Suppose your test needs:

```text
Username
Password
Environment
Base URL
API key
```

We generally don't want to hard-code environment-specific values directly in test code.

Instead, we can keep configuration values in `.env` files.

Example:

```text
.env.qa
.env.dev
.env.prod
```

Then `dotenv` loads the selected `.env` file into Node.js's environment variables.

### Core idea

```text
.env file
   ↓
dotenv loads it
   ↓
process.env
   ↓
your Playwright code
```

### One-line memory trick

> **dotenv loads values from a `.env` file into `process.env`.**

---

# 2. What Is `npm install dotenv`?

Run:

```bash
npm install dotenv
```

This installs the `dotenv` package into your project.

It also adds `dotenv` to the `dependencies` section of `package.json`.

Conceptually:

```text
npm install dotenv
        ↓
dotenv package installed
        ↓
Node/Playwright code can import and use it
```

---

# 3. What Is `.env`?

A `.env` file is a plain text configuration file containing key-value pairs.

Example:

```text
userID=test
passWord=test
```

Another example:

```text
BASE_URL=https://example.com
userID=testuser
passWord=testpassword
```

The basic format is:

```text
KEY=VALUE
```

### Important

Don't write:

```text
userID = test
```

Prefer:

```text
userID=test
```

The simple and conventional `.env` format is:

```text
KEY=VALUE
```

---

# 4. Why Have Different `.env` Files?

Different environments can have different values.

For example:

```text
config/
│
├── .env.qa
├── .env.dev
└── .env.prod
```

### `.env.qa`

```text
userID=test
passWord=test
```

### `.env.dev`

```text
userID=devuser
passWord=devpassword
```

The test code does not need to change.

Only the environment configuration changes.

### Mental model

```text
              Same test
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     QA        DEV       PROD
       │         │         │
   .env.qa    .env.dev  .env.prod
```

---

# 5. Install Dotenv

From the project root:

```bash
npm install dotenv
```

Verify it appears in `package.json`:

```json
{
  "dependencies": {
    "dotenv": "..."
  }
}
```

The exact version depends on the version installed in your project.

---

# 6. Import `config` From `dotenv`

In the file where you want to load the environment file:

```ts
import { config } from "dotenv";
```

`config()` is the function provided by `dotenv` that loads variables from a `.env` file.

---

# 7. What Is `process.env`?

This is one of the most important concepts.

Node.js provides:

```ts
process.env
```

It contains environment variables available to the Node.js process.

For example, after loading:

```text
userID=test
passWord=test
```

you can access them using:

```ts
process.env.userID
process.env.passWord
```

Think:

```text
.env.qa
│
├── userID=test
└── passWord=test

        ↓ dotenv

process.env
│
├── userID → "test"
└── passWord → "test"
```

### One-line memory trick

> **`.env` stores the values; `process.env` is how your Node.js code reads them after they are loaded.**

---

# 8. Your Environment Selection Logic

Your example:

```ts
const currentEnv = process.env.ENV || "qa";
```

This means:

> Use the value of `ENV` if it exists; otherwise use `"qa"`.

For example:

### If you run:

```bash
ENV=dev
```

then:

```ts
currentEnv
```

will be:

```text
dev
```

If `ENV` is not provided:

```text
currentEnv
```

becomes:

```text
qa
```

### Mental model

```text
Is ENV available?
      │
   ┌──┴──┐
  YES    NO
   │      │
   ↓      ↓
  dev     qa
```

This is the JavaScript/TypeScript `||` fallback pattern.

---

# 9. Loading the Selected `.env` File

Your example:

```ts
config({ path: `./config/.env.${currentEnv}` });
```

Suppose:

```ts
currentEnv = "qa";
```

Then the path becomes:

```text
./config/.env.qa
```

If:

```ts
currentEnv = "dev";
```

the path becomes:

```text
./config/.env.dev
```

So the same code can load different environment files.

### Mental model

```text
currentEnv = qa
      ↓
./config/.env.qa
      ↓
dotenv loads file
      ↓
process.env
```

---

# 10. Your `playwright.config.ts`

A simple version of your approach:

```ts
import { defineConfig } from "@playwright/test";
import { config } from "dotenv";

const currentEnv = process.env.ENV || "qa";

console.log("Current Environment is " + currentEnv);

config({
    path: `./config/.env.${currentEnv}`
});

export default defineConfig({
    // other Playwright configuration
});
```

The important part for environment handling is:

```ts
const currentEnv = process.env.ENV || "qa";

config({
    path: `./config/.env.${currentEnv}`
});
```

---

# 11. Project Structure

A simple structure for this approach:

```text
Playwright Project
│
├── config/
│   ├── .env.qa
│   ├── .env.dev
│   └── .env.prod
│
├── src/
│   └── pages/
│       └── LoginPage.ts
│
├── tests/
│   └── login.spec.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

---

# 12. Example `.env.qa`

File:

```text
config/.env.qa
```

Content:

```text
userID=test
passWord=test
```

This means the QA environment has:

```text
userID → test
passWord → test
```

---

# 13. How the Login Test Uses It

Your basic example:

```ts
test("user is able to login or not", async ({ page }) => {
    await loginPage.doLogin(
        process.env.userID,
        process.env.passWord
    );
});
```

The important flow is:

```text
.env.qa
   ↓
userID=test
passWord=test
   ↓
dotenv loads the file
   ↓
process.env.userID
process.env.passWord
   ↓
doLogin()
```

So the test does not contain:

```ts
doLogin("test", "test");
```

Instead it reads the values from the environment configuration.

---

# 14. Important TypeScript Issue With `process.env`

In Node.js TypeScript, environment variables are typed as potentially undefined.

So:

```ts
process.env.userID
```

is not guaranteed by TypeScript to be a string.

Conceptually:

```text
process.env.userID
        ↓
string | undefined
```

But your Page Object method may expect:

```ts
doLogin(userID: string, passWord: string)
```

Therefore TypeScript may complain when you pass:

```ts
process.env.userID
```

directly.

---

# 15. Simple Safe Handling

For a basic framework, one approach is to check the values before using them:

```ts
const userID = process.env.userID;
const passWord = process.env.passWord;

if (!userID || !passWord) {
    throw new Error("userID or passWord is missing");
}

await loginPage.doLogin(userID, passWord);
```

Now TypeScript knows that after the check, both values exist.

### Algorithm

```text
Read userID
Read password
      ↓
Are both available?
      │
   ┌──┴──┐
  YES    NO
   │      │
   ↓      ↓
Login    Fail fast
```

This is better than allowing the test to fail later with a confusing error.

---

# 16. Complete Example

## `config/.env.qa`

```text
userID=test
passWord=test
```

## `playwright.config.ts`

```ts
import { defineConfig } from "@playwright/test";
import { config } from "dotenv";

const currentEnv = process.env.ENV || "qa";

console.log("Current Environment is " + currentEnv);

config({
    path: `./config/.env.${currentEnv}`
});

export default defineConfig({
    // Playwright configuration
});
```

## `login.spec.ts`

```ts
import { test } from "@playwright/test";

test("user is able to login or not", async ({ page }) => {

    const userID = process.env.userID;
    const passWord = process.env.passWord;

    if (!userID || !passWord) {
        throw new Error("userID or passWord is missing");
    }

    await loginPage.doLogin(userID, passWord);
});
```

The exact import and creation of `loginPage` depends on your Page Object/fixture setup.

---

# 17. The Complete End-to-End Algorithm

This is the most important section for revision.

```text
1. Install dotenv
        ↓
npm install dotenv

2. Create environment file
        ↓
config/.env.qa

3. Put configuration values inside
        ↓
userID=test
passWord=test

4. Decide which environment to use
        ↓
process.env.ENV || "qa"

5. Build the .env file path
        ↓
./config/.env.qa

6. Ask dotenv to load that file
        ↓
config({ path: ... })

7. dotenv loads the values
        ↓
process.env

8. Test reads the values
        ↓
process.env.userID
process.env.passWord

9. Pass them to the Page Object
        ↓
loginPage.doLogin(userID, passWord)
```

---

# 18. How Environment Switching Works

Suppose you have:

```text
config/
├── .env.qa
├── .env.dev
└── .env.prod
```

And:

```ts
const currentEnv = process.env.ENV || "qa";

config({
    path: `./config/.env.${currentEnv}`
});
```

### QA

If no `ENV` value is supplied:

```text
currentEnv = qa
```

Loads:

```text
config/.env.qa
```

### DEV

If:

```text
ENV=dev
```

Loads:

```text
config/.env.dev
```

### PROD

If:

```text
ENV=prod
```

Loads:

```text
config/.env.prod
```

The test code remains the same.

---

# 19. Why This Is Useful in Automation

Without environment configuration:

```text
Test code
   ↓
contains QA values
   ↓
Change code for DEV
   ↓
Change code again for PROD
```

With environment configuration:

```text
                 Same test
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       QA          DEV         PROD
        │           │           │
    .env.qa     .env.dev     .env.prod
```

The test logic remains unchanged.

Only configuration changes.

---

# 20. What `dotenv` Does NOT Do

`dotenv` does not:

- create `.env` files for you
- decide which environment you should use
- automatically know that your file is `.env.qa`
- replace `process.env`
- execute your tests

Your code decides which file to load:

```ts
config({
    path: `./config/.env.${currentEnv}`
});
```

`dotenv` loads that file.

---

# 21. Common Mental Confusion

### Question: Where is `userID` stored?

Initially:

```text
config/.env.qa
```

contains:

```text
userID=test
```

After `dotenv` loads the file, your Node.js process can access it through:

```ts
process.env.userID
```

---

### Question: Is `dotenv` the environment variable?

No.

```text
dotenv
   ↓
package/library that loads .env values
```

Whereas:

```text
process.env
   ↓
Node.js object used to access environment variables
```

---

### Question: Is `.env.qa` a JavaScript/TypeScript file?

No.

It is a plain text environment/configuration file.

Example:

```text
userID=test
passWord=test
```

---

# 22. Security Note

Environment files are commonly used for configuration and secrets, but putting a value in `.env` does **not** automatically make it secure.

For example:

```text
passWord=test
```

is still plain text in the file.

Do not commit real passwords, API keys, tokens, or other secrets to Git.

A common practice is to add environment files containing secrets to `.gitignore`.

Example:

```gitignore
.env
.env.*
```

If your project needs a committed template, a common approach is to provide a non-secret example file such as:

```text
.env.example
```

with placeholder values.

---

# 23. Recommended Naming

Environment variable names are commonly written in uppercase:

```text
USER_ID=test
PASSWORD=test
```

Then:

```ts
process.env.USER_ID
process.env.PASSWORD
```

Your current names:

```text
userID
passWord
```

can work as environment variable names, but uppercase names are generally easier to recognize as environment/configuration values.

For consistency in a real project, you could use:

```text
USER_ID=test
PASSWORD=test
```

and:

```ts
process.env.USER_ID
process.env.PASSWORD
```

---

# 24. What You Actually Need to Remember

Do not memorize the complete code.

Remember these concepts:

### 1. Install

```bash
npm install dotenv
```

### 2. Create `.env` file

```text
userID=test
passWord=test
```

### 3. Import dotenv

```ts
import { config } from "dotenv";
```

### 4. Select environment

```ts
const currentEnv = process.env.ENV || "qa";
```

Meaning:

> Use `ENV` if supplied; otherwise use QA.

### 5. Load the selected file

```ts
config({
    path: `./config/.env.${currentEnv}`
});
```

### 6. Read values

```ts
process.env.userID
process.env.passWord
```

### 7. Use the values

```ts
await loginPage.doLogin(userID, passWord);
```

---

# 25. Final Mental Map

```text
              ENVIRONMENT CONFIGURATION
                         │
                         ↓
                 .env.qa / .env.dev
                         │
                         ↓
                      dotenv
                         │
                         ↓
                    process.env
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
      process.env.userID      process.env.passWord
             │                       │
             └───────────┬───────────┘
                         ↓
                Page Object / Test
                         ↓
              loginPage.doLogin(...)
```

## The one sentence to remember

> **`.env` stores configuration values, `dotenv` loads them, and `process.env` lets your Node.js/Playwright code access them.**

## The environment-switching sentence

> **`process.env.ENV || "qa"` chooses the environment, and `config({ path: ... })` loads the corresponding `.env` file.**

---

# 26. Quick Revision Card

```text
INSTALL
npm install dotenv

STORE
config/.env.qa

VALUES
USER_ID=test
PASSWORD=test

IMPORT
import { config } from "dotenv";

SELECT ENV
const currentEnv = process.env.ENV || "qa";

LOAD
config({
    path: `./config/.env.${currentEnv}`
});

READ
process.env.USER_ID
process.env.PASSWORD

USE
loginPage.doLogin(userID, password);
```

### Final algorithm

```text
INSTALL
  ↓
CREATE .env
  ↓
SELECT ENVIRONMENT
  ↓
LOAD .env WITH DOTENV
  ↓
READ THROUGH process.env
  ↓
USE VALUES IN TEST / PAGE OBJECT
```
