# JavaScript Fundamentals --- Reading & Revision Notes

> **One-line overview:** Learn how to run JavaScript, understand its
> data types, and choose correctly between `var`, `let`, and `const`.

## Index --- What We Are Covering

1.  [Old JavaScript vs. Modern
    JavaScript](#1-old-javascript-vs-modern-javascript) --- Understand
    ES6 and the difference between JavaScript versions and module
    systems.
2.  [How to Run JavaScript](#2-how-to-run-javascript) --- Run a `.js`
    file with Node.js from the VS Code terminal.
3.  [VS Code Shortcuts and
    Extensions](#3-vs-code-shortcuts-and-extensions) --- Know the
    purpose of common tools and shortcuts.
4.  [Packages and Modules](#4-packages-and-modules) --- Understand
    packages, npm, modules, and `package.json`.
5.  [JavaScript Data Types](#5-javascript-data-types) --- Identify
    primitive values and objects.
6.  [The `typeof` Operator](#6-the-typeof-operator) --- Check the type
    of a value and remember its special cases.
7.  [Variable Declaration Keywords](#7-variable-declaration-keywords)
    --- Understand `var`, `let`, and `const`.
8.  [Reassignment vs. Redeclaration](#8-reassignment-vs-redeclaration)
    --- Learn the difference with practical examples.
9.  [Quick Revision Table](#9-quick-revision-table) --- Compare the
    declaration keywords at a glance.
10. [Mental Map and Memory Tricks](#10-mental-map-and-memory-tricks) ---
    Revise the main ideas quickly.
11. [Interview-Ready Summary](#11-interview-ready-summary) --- Practice
    a short explanation.

------------------------------------------------------------------------

## 1. Old JavaScript vs. Modern JavaScript

### Simple understanding

-   **JavaScript** is a programming language.
-   **ES6 (ECMAScript 2015)** introduced many widely used features,
    including `let`, `const`, classes, and ES Modules.
-   JavaScript has continued to evolve after 2015. ES6 is not the final
    or only version of modern JavaScript.
-   **CommonJS and ES Modules are module systems**, not names for old
    and new JavaScript.

  -----------------------------------------------------------------------
  CommonJS                            ES Modules (ESM)
  ----------------------------------- -----------------------------------
  Commonly uses `require()` to import Uses `import` to import code.
  code.                               

  Commonly uses `module.exports` to   Uses `export` to export code.
  export code.                        

  Common in many existing Node.js     Standard JavaScript module system,
  projects.                           supported by browsers and Node.js.
  -----------------------------------------------------------------------

**CommonJS example:**

``` javascript
const message = require("./message.cjs");
```

**ES Modules example:**

``` javascript
import message from "./message.js";
```

In Node.js, a project can use `"type": "module"` in `package.json` to
treat `.js` files as ES Modules.

``` json
{
  "type": "module"
}
```

**Remember:** `"type": "module"` is a Node.js project configuration. It
does not mean the JavaScript language itself is ES6.

## 2. How to Run JavaScript

### Step 1 --- Create a file

Example file path:

``` text
jstutorial/test.js
```

### Step 2 --- Add JavaScript code

``` javascript
console.log("Hello JavaScript");
```

### Step 3 --- Run it in the VS Code terminal

``` bash
node jstutorial/test.js
```

Expected output:

``` text
Hello JavaScript
```

**Mental map:** Write code in a `.js` file → run it with Node.js → see
the output in the terminal.

**Note:** Node.js must be installed to run the command `node`. Code
Runner is optional; the terminal is enough.

## 3. VS Code Shortcuts and Extensions

-   **Code Runner:** An optional extension that can run supported code
    files.
-   **JavaScript (ES6) Code Snippets:** An optional extension that
    provides shortcuts for inserting code patterns.
-   **Command Palette:** Press `Ctrl + Shift + P` to search for and run
    VS Code commands.

**Correction:** `Ctrl + Shift + P` opens the Command Palette; it does
not restart VS Code. To restart, close and reopen VS Code, or use an
appropriate restart command if one is available.

## 4. Packages and Modules

### Simple understanding

JavaScript projects **do have packages**.

  -----------------------------------------------------------------------
  Term                    Simple meaning          Example
  ----------------------- ----------------------- -----------------------
  Module                  A unit of code that can `login.js`
                          export or import        
                          functionality; often a  
                          file.                   

  Package                 Code distributed for    `dotenv`
                          reuse, often with       
                          metadata and            
                          dependencies.           

  npm                     A tool for installing   `npm install dotenv`
                          and managing JavaScript 
                          packages.               

  `package.json`          Project metadata and    `"type": "module"`
                          configuration,          
                          including dependencies  
                          and sometimes the       
                          module type.            
  -----------------------------------------------------------------------

Example command:

``` bash
npm install dotenv
```

This installs the `dotenv` package, which can load environment variables
from a `.env` file.

**Remember:** JavaScript is the language; Node.js runs JavaScript
outside the browser; npm manages packages.

## 5. JavaScript Data Types

A **data type** tells us what kind of value we are working with.

JavaScript values are commonly grouped into two categories:

### 5.1 Primitive types

JavaScript has seven primitive types:

1.  `number`
2.  `string`
3.  `boolean`
4.  `undefined`
5.  `null`
6.  `bigint`
7.  `symbol`

### 5.2 Objects (non-primitive values)

Examples include:

-   Objects
-   Arrays
-   Functions
-   Maps
-   Sets

Arrays, functions, Maps, and Sets are object-related values in
JavaScript. A function is callable, and `typeof` reports `"function"`
for functions.

**Important correction:** Do not define primitive vs. object values by
claiming one always has fixed memory size and the other has variable
memory size. JavaScript manages memory internally, and such a rule is
not a correct definition.

### 5.3 Number

The `number` type represents ordinary integers and decimal numbers.

``` javascript
let age = 37;
let price = 12.33;

console.log(typeof age);   // "number"
console.log(typeof price); // "number"
```

Both values have the same type: `number`.

JavaScript's `number` type uses IEEE 754 double-precision floating-point
representation. Avoid memorizing "all numbers take exactly 8 bytes" as a
general memory rule for JavaScript variables.

### 5.4 String

A string represents text. Use single quotes, double quotes, or
backticks.

``` javascript
let firstName = "Karthik";
let city = 'Hyderabad';
let message = `Hello ${firstName}`;

console.log(typeof firstName); // "string"
console.log(message);          // Hello Karthik
```

**Correction:** Do not memorize "every character always takes 2 bytes."
JavaScript strings use UTF-16 code units, and actual memory usage
depends on the characters and the JavaScript engine's implementation.

### 5.5 Undefined

`undefined` commonly means a value has not been assigned.

``` javascript
let result;

console.log(result);        // undefined
console.log(typeof result); // "undefined"
```

A declared `let` or `var` variable without an assigned value evaluates
to `undefined`.

You usually do not need to write `let result = undefined;`. Simply write
`let result;`.

### 5.6 Boolean

A Boolean value is either `true` or `false`.

``` javascript
let isResultPublished = true;

console.log(typeof isResultPublished); // "boolean"
```

Practical example: A test result can be marked `true` when published and
`false` when not published.

JavaScript is case-sensitive. `isResultPublished` and
`isresultPublished` are different names.

### 5.7 Null

`null` represents an intentionally empty or absent value.

``` javascript
let selectedUser = null;

console.log(selectedUser);        // null
console.log(typeof selectedUser); // "object"
```

**Important:** `typeof null` returns `"object"` because of a historical
JavaScript quirk. This does not mean that `null` is actually an object;
it is a primitive value.

### 5.8 BigInt

`bigint` represents integers that are too large for safe integer
precision with the regular `number` type.

``` javascript
let largeNumber = 12345678901234567890n;

console.log(typeof largeNumber); // "bigint"
```

The `n` suffix identifies a BigInt literal.

### 5.9 Symbol

A `symbol` is a primitive type used to create unique values. One use is
creating unique object property keys.

``` javascript
let id = Symbol("id");

console.log(typeof id); // "symbol"
```

For beginner-level Playwright learning, understanding the basic purpose
is enough for now.

### 5.10 Object, Array, and Function

``` javascript
let employee = { name: "Karthik", age: 37 };
let scores = [80, 90, 95];

function greet() {
    console.log("Hello");
}

console.log(typeof employee); // "object"
console.log(typeof scores);   // "object"
console.log(typeof greet);    // "function"
```

-   **Object:** Stores related information as properties.
-   **Array:** Stores an ordered collection of values.
-   **Function:** Holds reusable behavior that can be called.
-   **Map:** Stores key-value pairs.
-   **Set:** Stores unique values.

JavaScript has no built-in primitive type named `list`. Arrays are
commonly used as lists.

## 6. The `typeof` Operator

Use `typeof` to check the type of a value.

``` javascript
let x = 10;
console.log(typeof x); // "number"
```

Common results:

  Value              Result of `typeof`
  ------------------ ---------------------------------
  `10`               `"number"`
  `"Hello"`          `"string"`
  `true`             `"boolean"`
  `undefined`        `"undefined"`
  `10n`              `"bigint"`
  `Symbol("id")`     `"symbol"`
  `{}`               `"object"`
  `[]`               `"object"`
  `null`             `"object"` --- historical quirk
  `function () {}`   `"function"`

**Remember:** `typeof` returns a string containing the type label.

## 7. Variable Declaration Keywords

A variable is a named place through which a value can be accessed.

``` javascript
let age = 37;
console.log(age);
```

  Code                 Meaning
  -------------------- ----------------------------
  `let`                Declaration keyword
  `age`                Variable name (identifier)
  `=`                  Assignment operator
  `37`                 Assigned value
  `console.log(age)`   Prints the value

In modern JavaScript, prefer `const` by default and use `let` when the
variable needs reassignment. Learn `var` to understand older code, but
usually avoid it in new code.

### 7.1 `var`

``` javascript
var x = 10;
var x = 20;

console.log(x); // 20
```

-   Redeclaration in the same scope is allowed.
-   Reassignment is allowed.
-   `var` is function-scoped, not block-scoped.

### 7.2 `let`

``` javascript
let x = 10;
x = 20;

console.log(x); // 20
```

-   Redeclaration in the same scope is not allowed.
-   Reassignment is allowed.
-   `let` is block-scoped.

### 7.3 `const`

``` javascript
const collegeName = "VIST";

console.log(collegeName);
```

-   Redeclaration in the same scope is not allowed.
-   Reassignment is not allowed.
-   An initial value is required at declaration.
-   `const` is block-scoped.

This is invalid:

``` javascript
const title;
```

It causes a `SyntaxError` because a `const` declaration requires an
initializer.

**Important:** `const` prevents reassignment of the variable binding; it
does not automatically make an object or array immutable.

``` javascript
const employee = { name: "Karthik" };

employee.name = "Rajesh"; // Allowed: object property changes
// employee = {};         // Not allowed: variable reassignment
```

## 8. Reassignment vs. Redeclaration

These terms mean different things.

-   **Reassignment:** Giving an existing variable a new value.
-   **Redeclaration:** Declaring a variable with the same name again in
    the same scope.

### Reassignment with `let`

``` javascript
let name = "Karthik";
name = "Rajesh";

console.log(name); // Rajesh
```

This is valid because the existing variable receives a new value.

### Redeclaration with `let`

``` javascript
let name = "Karthik";
let name = "Rajesh";
```

This produces a `SyntaxError` when both declarations are in the same
scope.

### Changing the type of a value

``` javascript
let x = 10;
x = "Karthik";

console.log(x);        // Karthik
console.log(typeof x); // "string"
```

This is valid because JavaScript is dynamically typed. A variable can
hold a number and later hold a string.

## 9. Quick Revision Table

  Feature                           `var`      `let`   `const`
  --------------------------------- ---------- ------- ---------
  Redeclaration in the same scope   Yes        No      No
  Reassignment                      Yes        Yes     No
  Initial value required            No         No      Yes
  Scope                             Function   Block   Block

These rules describe declarations in the same scope. Different scopes
can affect whether a declaration is allowed.

## 10. Mental Map and Memory Tricks

### Mental map: choosing a variable keyword

``` text
Need to declare a variable
          |
          v
Will the variable be reassigned?
          |
      +---+---+
      |       |
     No      Yes
      |       |
    const    let
```

Use `var` mainly when reading or maintaining older JavaScript code.

### Memory tricks

-   **`const` = constant binding:** You cannot assign a new value to
    that variable.
-   **`let` = can change:** You can assign a new value later.
-   **`var` = older style:** Allows redeclaration in the same scope.
-   **`undefined` = not assigned yet:** A variable may have been
    declared without a value.
-   **`null` = intentionally empty:** A value is deliberately set to
    represent absence.
-   **Reassignment = change the value.**
-   **Redeclaration = declare the name again.**

## 11. Interview-Ready Summary

JavaScript is a dynamically typed programming language with seven
primitive types and object values. `var`, `let`, and `const` are used to
declare variables. `var` allows redeclaration in the same scope and is
function-scoped; `let` allows reassignment but not redeclaration in the
same scope and is block-scoped; `const` does not allow reassignment and
must be initialized when declared. CommonJS and ES Modules are different
module systems, and npm is used to manage packages.

------------------------------------------------------------------------

## Final One-Line Revision

**Use `const` by default, `let` when reassignment is needed, understand
`var` for older code, and remember that CommonJS vs. ES Modules is a
module-system difference---not old vs. new JavaScript.**
