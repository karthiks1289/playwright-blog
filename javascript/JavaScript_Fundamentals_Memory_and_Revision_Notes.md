# JavaScript Fundamentals — Memory-Friendly Notes

> **Goal:** Understand the basics today and still remember them after a few days.
>
> **How to use these notes:** Read one small section. Hide it. Say the idea in your own words. Try the code from memory. Check only after trying.

## Index

1. [Run JavaScript](#1-run-javascript) — Run a JavaScript file with Node.js.
2. [Old vs. Modern JavaScript](#2-old-vs-modern-javascript) — Understand CommonJS and ES Modules.
3. [Packages](#3-packages) — Know what npm does.
4. [Data Types](#4-data-types) — Recognize the main kinds of values.
5. [`typeof`](#5-typeof) — Check a value's type.
6. [`let`, `const`, `var`](#6-let-const-var) — Choose how to declare a variable.
7. [Reassignment vs. Redeclaration](#7-reassignment-vs-redeclaration) — Tell changing a value from declaring a name again.
8. [Most Important Interview Questions](#8-most-important-interview-questions) — Practise the critical questions.
9. [Close-the-Notes Self-Test](#9-close-the-notes-self-test) — Test what you remember without looking.
10. [Revision Schedule](#10-revision-schedule) — Review at the right times.
11. [My Forget Log](#11-my-forget-log) — Record what you forgot so it becomes easier next time.

---

## 1. Run JavaScript

**Remember:** Node.js can run JavaScript files outside a browser.

File: `test.js`

```javascript
console.log("Hello");
```

Run in the VS Code terminal:

```bash
node test.js
```

Output:

```text
Hello
```

**Say it from memory:** What command runs a JavaScript file?

---

## 2. Old vs. Modern JavaScript

**Remember:** CommonJS and ES Modules are two ways to share code. They are not the same as old JavaScript and new JavaScript.

| CommonJS | ES Modules |
|---|---|
| `require()` | `import` |
| `module.exports` | `export` |

Modern Node.js projects can use this in `package.json`:

```json
{
  "type": "module"
}
```

**Memory hook:** CommonJS → `require`; ES Modules → `import`.

---

## 3. Packages

**Remember:** JavaScript projects can use packages. npm helps install and manage them.

```bash
npm install dotenv
```

- **Package:** Reusable code.
- **npm:** Tool to install/manage packages.
- **`package.json`:** Project details and configuration.

**Memory hook:** JavaScript = language; Node.js = runs JS outside the browser; npm = manages packages.

---

## 4. Data Types

Do not try to memorize a long definition. Learn each type with one example.

| Type | Remember it as | Example |
|---|---|---|
| `string` | Text | `"Hello"` |
| `number` | Number or decimal | `25`, `12.5` |
| `boolean` | Yes/no value | `true`, `false` |
| `undefined` | No value assigned yet | `let result;` |
| `null` | Intentionally empty | `let user = null;` |
| `bigint` | Very large integer | `123n` |
| `symbol` | Unique value | `Symbol("id")` |
| Object | Related information | `{ name: "Sam" }` |
| Array | Ordered list | `[10, 20, 30]` |
| Function | Reusable action | `function greet() {}` |

Example:

```javascript
let name = "Sam";
let age = 25;
let isReady = true;
let result;

console.log(typeof name);    // "string"
console.log(typeof age);     // "number"
console.log(typeof isReady); // "boolean"
console.log(typeof result);  // "undefined"
```

### Two important traps

```javascript
console.log(typeof null); // "object" — historical JavaScript quirk
console.log(typeof []);   // "object"
```

**Memory hook:** `undefined` = not assigned; `null` = deliberately empty.

**Accuracy note:** Do not memorize “every string character uses exactly 2 bytes” or “primitive types have fixed sizes.” These are not reliable general rules for JavaScript memory use.

---

## 5. `typeof`

**Remember:** `typeof` tells you the type label of a value.

```javascript
console.log(typeof 10);       // "number"
console.log(typeof "Hi");     // "string"
console.log(typeof true);     // "boolean"
console.log(typeof undefined);// "undefined"
console.log(typeof null);     // "object" — special case
```

**Memory hook:** `typeof` returns a string, such as `"number"`.

---

## 6. `let`, `const`, `var`

Learn these three rules first:

- **`const`:** Do not reassign the variable.
- **`let`:** Reassignment is allowed.
- **`var`:** Older style; redeclaration in the same scope is allowed.

```javascript
const country = "India";
// country = "Japan"; // Error: reassignment is not allowed

let age = 25;
age = 26; // Allowed

var city = "Delhi";
var city = "Pune"; // Allowed
```

### Quick comparison

| Rule | `var` | `let` | `const` |
|---|---|---|---|
| Reassign value? | Yes | Yes | No |
| Redeclare in same scope? | Yes | No | No |
| Must give initial value? | No | No | Yes |
| Scope | Function | Block | Block |

**Memory hook:** `const` stays assigned; `let` can change; `var` is the older option.

### Important `const` trap

```javascript
const user = { name: "Sam" };
user.name = "Lee"; // Allowed
// user = {};      // Not allowed
```

`const` stops reassignment of the variable. It does not freeze an object.

**Practical habit:** Prefer `const`. Use `let` when you need to reassign. Usually avoid `var` in new code.

---

## 7. Reassignment vs. Redeclaration

These words sound similar but mean different things.

- **Reassignment:** Change the value.
- **Redeclaration:** Declare the same name again in the same scope.

```javascript
let name = "Sam";
name = "Lee"; // Reassignment: allowed
```

```javascript
let score = 10;
let score = 20; // Redeclaration: SyntaxError in the same scope
```

JavaScript is dynamically typed, so a variable can hold different types at different times:

```javascript
let value = 10;
value = "Hello";

console.log(typeof value); // "string"
```

**Memory hook:** Reassignment = new value. Redeclaration = declare the name again.

---

## 8. Most Important Interview Questions

Try to answer aloud before reading each answer. Keep the answer short; expand only if the interviewer asks.

### Q1. What is the difference between `let`, `const`, and `var`?

**Answer:** `let` allows reassignment, `const` does not allow reassignment, and `var` allows redeclaration in the same scope. They also differ in scope.

### Q2. What is the difference between reassignment and redeclaration?

**Answer:** Reassignment changes an existing variable's value. Redeclaration declares the same name again in the same scope.

### Q3. What are JavaScript's primitive types?

**Answer:** `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, and `null`.

### Q4. What is the difference between `undefined` and `null`?

**Answer:** `undefined` commonly means no value has been assigned. `null` represents an intentionally empty or absent value.

### Q5. Why does `typeof null` return `"object"`?

**Answer:** It is a historical JavaScript quirk. `null` is still a primitive value.

### Q6. Does `const` make an object unchangeable?

**Answer:** No. You can change its properties, but you cannot reassign the variable to a different object.

### Q7. What is the difference between CommonJS and ES Modules?

**Answer:** CommonJS commonly uses `require()` and `module.exports`. ES Modules use `import` and `export`.

### Q8. Does JavaScript have packages?

**Answer:** Yes. npm is commonly used to install and manage JavaScript packages.

### Q9. What does `typeof` do?

**Answer:** It returns a string describing the type of a value, such as `"number"` or `"string"`.

### Q10. Why do Playwright examples often use `const` for locators?

**Answer:** The locator variable usually does not need to be reassigned, so `const` is suitable.

---

## 9. Close-the-Notes Self-Test

**First, hide the notes above. Answer these from memory. Do not peek immediately.**

1. Which command runs `test.js` using Node.js?
2. Complete the memory hook: CommonJS uses `_____`; ES Modules use `_____`.
3. What is the difference between `undefined` and `null`?
4. What does `typeof null` return?
5. Which keyword allows reassignment but not redeclaration in the same scope?
6. Which keyword requires an initial value?
7. What is the difference between reassignment and redeclaration?
8. Can you change an object's property when the object variable is declared with `const`?
9. Name all seven primitive types.
10. Why is `const` a good default for a Playwright locator variable?

### Answers — check only after trying

1. `node test.js`
2. `require()`; `import`
3. `undefined` commonly means not assigned; `null` means intentionally empty.
4. `"object"` (historical quirk).
5. `let`
6. `const`
7. Reassignment changes a value; redeclaration declares the same name again in the same scope.
8. Yes, the property's value can change; the variable cannot be reassigned.
9. `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`.
10. The locator variable usually does not need reassignment.

**If you missed an answer:** Do not reread the whole file. Revisit only that small section, then answer the question again without looking.

---

## 10. Revision Schedule

Forgetting after a day or two is common. Understanding something while listening is not the same as being able to recall it later. Practice retrieving the answer from memory.

| When | What to do | Time |
|---|---|---:|
| Today | Learn one section, hide it, explain it aloud, and run the code. | 10 min |
| Day 1 | Do the self-test without looking at the notes. | 5 min |
| Day 3 | Repeat missed questions and write the key rules from memory. | 5 min |
| Day 7 | Answer the interview questions aloud and predict code output. | 5–10 min |
| Day 14 | Take the self-test again without help. | 5 min |
| Day 30 | Do a final recall test; review only weak areas. | 5 min |

**The key technique is active recall:** try to remember first, then check. Repeatedly rereading can feel familiar without proving you can recall the concept.

### A simple learning cycle

**Understand → Close notes → Recall → Check → Practise → Repeat later**

---

## 11. My Forget Log

Use this small table when you forget something. The goal is to learn from the gap, not to criticize yourself.

| Date | What I forgot | One-line memory rule | Next review |
|---|---|---|---|
| | | | |

Example:

| Date | What I forgot | One-line memory rule | Next review |
|---|---|---|---|
| Today | `let` vs. `const` | `let` can be reassigned; `const` cannot. | Day 1 |

---

## Final 20-Second Revision

- `const` = no reassignment; `let` = reassignment allowed; `var` = older style.
- `undefined` = commonly not assigned; `null` = intentionally empty.
- `typeof null` = `"object"` (historical quirk).
- CommonJS = `require()`; ES Modules = `import`.
- Reassignment changes a value; redeclaration declares the name again.
- To remember: **close the notes, recall the idea, check the gap, and repeat after a delay.**
