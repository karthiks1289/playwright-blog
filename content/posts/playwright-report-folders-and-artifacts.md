# Playwright Reports & Test Execution Artifacts

## Mental Map

**Test Run → HTML Report + Test Artifacts**

These are related, but they are **not the same thing**.

---

## 1. `playwright-report`

The `playwright-report` directory is commonly created when using the HTML reporter.

It contains the generated HTML report and its supporting assets.

The report provides an overall view of the test run, including information such as:

- passed tests
- failed tests
- skipped tests
- duration
- errors
- test details
- available attachments

The exact contents can vary depending on the Playwright version and configuration.

---

## 2. `test-results`

The `test-results` directory is commonly used for test execution artifacts.

Artifacts can include:

- screenshots
- videos
- traces
- other attachments

They are generally associated with individual test results and/or retries.

### Important

`test-results` is **not the main HTML report**.

Think:

```text
playwright-report
        ↓
Main HTML report

test-results
        ↓
Execution artifacts
```

---

## 3. Why Are Artifacts Kept?

Suppose a test fails.

The HTML report can tell us:

```text
Login Test → FAILED
```

An attached screenshot or trace can help answer:

> What actually happened when it failed?

This makes artifacts extremely useful for debugging failures, especially in CI/CD.

---

## 4. Simple Mental Model

```text
                 Test Execution
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
      playwright-report     test-results
             │                   │
        HTML report          Artifacts
                                 │
                         ┌───────┼────────┐
                         ↓       ↓        ↓
                     Screenshot Video    Trace
```

## Interview Answer

> `playwright-report` is primarily the generated HTML report directory, while `test-results` stores execution artifacts such as screenshots, videos, and traces associated with test results or retries. The HTML report is the main place for viewing the overall test execution results.

## Memory Trick

**Report = What happened?**

**Artifact = Evidence of what happened?**
