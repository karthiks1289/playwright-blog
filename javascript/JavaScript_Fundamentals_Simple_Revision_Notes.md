# JavaScript Fundamentals — Simple Learning & Revision Notes

> **One-line overview:** Learn how to run JavaScript, recognize common data types, and use `let`, `const`, and `var` correctly.

## Index — What We Cover

1. [Old vs. Modern JavaScript](#1-old-vs-modern-javascript) — Understand ES6 and module systems.
2. [Run JavaScript](#2-run-javascript) — Run a `.js` file with Node.js.
3. [VS Code Basics](#3-vs-code-basics) — Know the useful shortcut and optional extensions.
4. [Packages and Modules](#4-packages-and-modules) — Understand npm, packages, and `package.json`.
5. [Data Types](#5-data-types) — Recognize primitive values and objects.
6. [`typeof`](#6-typeof) — Check a value's type.
7. [`var`, `let`, and `const`](#7-var-let-and-const) — Choose the right declaration.
8. [Reassignment vs. Redeclaration](#8-reassignment-vs-redeclaration) — Understand two commonly confused terms.
9. [Common Interview Questions](#9-common-interview-questions) — Revise the most important interview questions.
10. [30-Second Revision](#10-30-second-revision) — Remember the key points quickly.

---

## 1. Old vs. Modern JavaScript

**Simple meaning:** ES6 (2015) introduced many useful features, including `let`, `const`, and a standard module system called ES Modules. JavaScript has continued to evolve since then.

**Important:** CommonJS and ES Modules are two ways to share code. They do **not** mean “old JavaScript” and “new JavaScript.”

| CommonJS | ES Modules (ESM) |
|---|---|
| Uses `require()` to import code. | Uses `import` to import code. |
| Uses `module.exports` to export code. | Uses `export` to export code. |
| Common in existing Node.js projects. | Standard JavaScript module system. |

CommonJS example:

```javascript
const message = require("./message.cjs");
```

ES Modules example:

```javascript
import message from "./message.js";
```

In Node.js, adding this to `package.json` makes `.js` files use ES Modules:

```json
{
  "type": "module"
}
```

**Remember:** `"type": "module"` is Node.js project configuration, not a JavaScript language version.

## 2. Run JavaScript

1. Create `jstutorial/test.js`.
2. Add this code:

```javascript
console.log("Hello JavaScript");
```

3. Open the VS Code terminal and run:

```bash
node jstutorial/test.js
```

Expected output:

```text
Hello JavaScript
```

**Mental map:** Write code → run it with Node.js → see output in the terminal.

**Note:** Node.js must be installed. Code Runner is optional.

## 3. VS Code Basics

- **Code Runner:** Optional extension that can run code.
- **JavaScript (ES6) Code Snippets:** Optional extension for inserting code patterns.
- **`Ctrl + Shift + P`:** Opens the VS Code Command Palette.

**Correction:** `Ctrl + Shift + P` does not restart VS Code. To restart, close and reopen VS Code, or use an available restart command.

## 4. Packages and Modules

JavaScript **does have packages**.

| Term | Simple meaning |
|---|---|
| Module | A unit of code that can be imported or exported. |
| Package | Reusable code distributed for use in projects. |
| npm | Tool for installing and managing packages. |
| `package.json` | Project information and configuration, including dependencies. |

Example:

```bash
npm install dotenv
```

This installs `dotenv`, a package often used to load environment variables from a `.env` file.

**Remember:** JavaScript is the language; Node.js runs JavaScript outside the browser; npm manages packages.

## 5. Data Types

A **data type** tells you what kind of value you are using.

### Primitive types

JavaScript has seven primitive types:

- `number`
- `string`
- `boolean`
- `undefined`
- `null`
- `bigint`
- `symbol`

### Object values

Examples include objects, arrays, functions, Maps, and Sets.

```javascript
let employee = { name: "Karthik" };
let scores = [80, 90, 95];

function greet() {
  console.log("Hello");
}
```

- **Object:** Holds related information as properties.
- **Array:** Holds an ordered collection of values.
- **Function:** Holds reusable behavior that can be called.

**Avoid this misconception:** Primitive types are not defined as “fixed memory size” and objects as “variable memory size.” JavaScript manages memory internally.

### Common examples

**Number** — Used for ordinary integers and decimals.

```javascript
let age = 37;
let price = 12.33;

console.log(typeof age);   // "number"
console.log(typeof price); // "number"
```

**String** — Used for text. Single quotes, double quotes, and backticks can be used.

```javascript
let name = "Karthik";
console.log(typeof name); // "string"
```

**Do not memorize:** “Every character always takes 2 bytes.” JavaScript strings use UTF-16 code units, but actual memory use depends on the characters and engine.

**Undefined** — A declared variable without an assigned value is `undefined`.

```javascript
let result;
console.log(result); // undefined
```

Usually, `let result;` is enough; you do not need to write `let result = undefined;`.

**Boolean** — Has only two values: `true` or `false`.

```javascript
let isPublished = true;
console.log(typeof isPublished); // "boolean"
```

**Null** — Used to represent an intentionally empty or absent value.

```javascript
let selectedUser = null;
console.log(selectedUser);        // null
console.log(typeof selectedUser); // "object"
```

The `typeof null` result is a historical JavaScript quirk. `null` is still a primitive value.

**BigInt** — Used for integers beyond the safe integer precision of regular `number`.

```javascript
let largeNumber = 12345678901234567890n;
console.log(typeof largeNumber); // "bigint"
```

**Symbol** — Creates a unique primitive value.

```javascript
let id = Symbol("id");
console.log(typeof id); // "symbol"
```

## 6. `typeof`

Use `typeof` to check a value's type. It returns a string.

| Value | `typeof` result |
|---|---|
| `10` | `"number"` |
| `"Hello"` | `"string"` |
| `true` | `"boolean"` |
| `undefined` | `"undefined"` |
| `10n` | `"bigint"` |
| `Symbol("id")` | `"symbol"` |
| `{}` | `"object"` |
| `[]` | `"object"` |
| `null` | `"object"` — historical quirk |
| A function | `"function"` |

**Memory trick:** Remember the special case: `typeof null` is `"object"`.

## 7. `var`, `let`, and `const`

A variable is a name used to access a value.

```javascript
let age = 37;
console.log(age);
```

- `let` — declaration keyword
- `age` — variable name (identifier)
- `=` — assignment operator
- `37` — value

### `let`: value can change

```javascript
let age = 30;
age = 31;

console.log(age); // 31
```

You can reassign a `let` variable, but you cannot redeclare it in the same scope.

### `const`: no reassignment

```javascript
const collegeName = "VIST";
console.log(collegeName);
```

This is invalid because `const` needs an initial value:

```javascript
const title; // SyntaxError
```

**Important:** `const` prevents reassignment of the variable, but an object declared with `const` can still have its properties changed.

```javascript
const employee = { name: "Karthik" };
employee.name = "Rajesh"; // Allowed
// employee = {};         // Not allowed
```

### `var`: older declaration style

```javascript
var city = "Hyderabad";
var city = "Chennai";

console.log(city); // Chennai
```

`var` allows redeclaration in the same scope and allows reassignment.

### Quick comparison

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Reassignment | Yes | Yes | No |
| Redeclaration in the same scope | Yes | No | No |
| Initial value required | No | No | Yes |
| Scope | Function | Block | Block |

**Recommendation:** Prefer `const` by default. Use `let` when the variable needs reassignment. Understand `var` for older code, but usually avoid it in new code.

## 8. Reassignment vs. Redeclaration

- **Reassignment:** Change the value of an existing variable.
- **Redeclaration:** Declare the same variable name again in the same scope.

Reassignment:

```javascript
let name = "Karthik";
name = "Rajesh";

console.log(name); // Rajesh
```

Redeclaration with `let` in the same scope:

```javascript
let name = "Karthik";
let name = "Rajesh"; // SyntaxError
```

JavaScript is dynamically typed, so a `let` variable can hold different types at different times:

```javascript
let x = 10;
x = "Karthik";

console.log(x);        // Karthik
console.log(typeof x); // "string"
```

### Mental map

```text
Need to declare a variable
          |
          v
Will you reassign it?
      /         \
    No           Yes
    |             |
  const          let

Use var mainly when working with older code.
```

## 9. Common Interview Questions

These are high-value questions for JavaScript fundamentals. Answers are intentionally short for revision.

### Level 1 — Must know

**Q1. What is JavaScript?**

JavaScript is a programming language used to add behavior to web pages. It can also run outside browsers using environments such as Node.js.

**Q2. What is the difference between `var`, `let`, and `const`?**

`var` allows redeclaration in the same scope. `let` allows reassignment but not redeclaration in the same scope. `const` does not allow reassignment and must be initialized when declared.

**Q3. What is the difference between reassignment and redeclaration?**

Reassignment changes an existing variable's value. Redeclaration declares the same variable name again in the same scope.

**Q4. What are JavaScript's primitive data types?**

`string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, and `null`.

**Q5. What is the difference between `undefined` and `null`?**

`undefined` commonly means a value has not been assigned. `null` is used to represent an intentionally empty or absent value.

### Level 2 — Critical concepts

**Q6. Why does `typeof null` return `"object"`?**

It is a historical JavaScript quirk. `null` is a primitive value even though `typeof null` returns `"object"`.

**Q7. What is the difference between CommonJS and ES Modules?**

CommonJS commonly uses `require()` and `module.exports`. ES Modules use `import` and `export`.

**Q8. Does `const` make an object immutable?**

No. It prevents reassignment of the variable binding, but the object's properties can still be changed.

**Q9. What is the difference between block scope and function scope?**

`let` and `const` are limited to the block where they are declared. `var` is function-scoped, or global-scoped when declared at the top level of a classic script.

**Q10. Does JavaScript have packages?**

Yes. Packages provide reusable code and can be installed and managed with tools such as npm.

### Level 3 — Output-based questions

**Q11. What is the output?**

```javascript
let x = 10;
x = "Hello";
console.log(typeof x);
```

**Answer:** `"string"` — the variable now holds a string.

**Q12. What is the output?**

```javascript
let x = 10;

{
  let x = 20;
}

console.log(x);
```

**Answer:** `10` — the inner `x` belongs to the block and is separate from the outer `x`.

**Q13. What is the output?**

```javascript
const user = { name: "Karthik" };
user.name = "Rajesh";
console.log(user.name);
```

**Answer:** `Rajesh` — changing an object property is allowed.

**Q14. What happens here?**

```javascript
let name = "Karthik";
let name = "Rajesh";
```

**Answer:** A `SyntaxError` occurs because `let` cannot be redeclared in the same scope.

### Level 4 — Practical automation question

**Q15. Why do we often declare Playwright locators with `const`?**

Because we usually do not need to reassign the locator variable. For example:

```typescript
const loginButton = page.getByRole("button", {
  name: "Login"
});
```

Using `const` communicates that the variable binding will not be reassigned. It does not mean the page or the locator's matching element can never change.

## 10. 30-Second Revision

- `const` — use when you do not need to reassign the variable.
- `let` — use when reassignment is needed.
- `var` — older style; allows redeclaration in the same scope.
- Primitive types: `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`.
- `typeof null` returns `"object"` because of a historical quirk.
- CommonJS uses `require()`; ES Modules use `import`.
- JavaScript projects can use packages managed by npm.
- Reassignment changes a value; redeclaration declares the name again.

**Final memory line:** Use `const` by default, `let` when reassignment is needed, and remember that CommonJS vs. ES Modules is a module-system difference—not old vs. new JavaScript.
