# Playwright CodeGen

## Mental Map

**Record User Actions → Generate Playwright Code → Review/Edit Code → Save → Run with Test Runner**

## 1. What Is CodeGen?

Playwright CodeGen is a code-generation tool that records browser interactions and generates Playwright code based on those interactions.

It is useful when learning Playwright and when quickly creating an initial version of an automation script.

---

## 2. What Can CodeGen Help With?

CodeGen can:

- record user interactions
- generate Playwright code
- inspect elements during recording
- suggest suitable locators
- help create an initial automation flow

Example:

```bash
npx playwright codegen https://example.com
```

---

## 3. Does CodeGen Execute Tests?

CodeGen is primarily for **recording and generating code**.

It is not the same thing as the Playwright Test Runner.

Think:

```text
CodeGen
   ↓
Generates code

Test Runner
   ↓
Executes tests
```

After generating the code, you normally review and adapt it into a proper test and execute it using Playwright Test.

---

## 4. Practical Workflow

```text
Open CodeGen
     ↓
Perform login manually
     ↓
CodeGen records actions
     ↓
Generated Playwright code
     ↓
Review locators and assertions
     ↓
Save as a test
     ↓
Run using Test Runner
```

## Interview Answer

> Playwright CodeGen is a recording and code-generation tool. It observes browser interactions and generates Playwright code, including locators. The generated code should be reviewed and can then be incorporated into tests and executed using the Playwright Test Runner.

## Memory Trick

**CodeGen = Create**

**Test Runner = Execute**
