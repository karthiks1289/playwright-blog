# Playwright Workers & Parallel Execution

## 1. Workers

1. A Playwright **Worker is a separate worker process** used by the Playwright Test Runner to execute tests.
2. Do **not** remember Worker = Thread.
3. Playwright Test uses Node.js processes for its worker processes.
4. Workers allow Playwright to execute tests concurrently when multiple workers are available.
5. Workers can be configured in `playwright.config.ts`.

```ts
export default defineConfig({
  workers: 2
});
```

You can also configure workers from the command line:

```bash
npx playwright test --workers=2
```

---

## 2. `workers: 1`

```ts
workers: 1
```

Conceptually:

```text
Worker 1
   |
   +-- Test 1
   +-- Test 2
   +-- Test 3
   +-- Test 4
```

Only one worker is available, so there is no worker-level parallel execution.

Example:

```bash
npx playwright test --workers=1
```

---

## 3. `workers: 2`

```ts
workers: 2
```

Playwright can use up to 2 worker processes concurrently.

Conceptually:

```text
Worker 1  ---> Test A
Worker 2  ---> Test B

Worker 1  ---> Test C
Worker 2  ---> Test D
```

The exact assignment/order of tests is managed by Playwright. Do not assume that workers pick tests randomly.

---

## 4. Test Pool Mental Model

Think of the test suite as a pool of tests:

```text
Test Pool
-------------------------
Test A
Test B
Test C
Test D
Test E
Test F
-------------------------

        |
        | Playwright schedules tests
        v

Worker 1  ---> Tests
Worker 2  ---> Tests
```

### Important correction

Do NOT say:

> Workers randomly pick test cases from the pool.

Better:

> Playwright Test Runner schedules tests across the available workers.

The exact scheduling/order should not be relied upon.

---

# 5. What is the Default Number of Workers?

If you do not configure `workers`, Playwright determines the default worker count based on the machine's CPU availability.

The important point is:

> Do NOT memorize a formula such as `CPU cores / 2` as the universal Playwright default.

For your notes/interviews, remember:

```text
workers not configured
        ↓
Playwright uses its default worker-count behavior
        ↓
based on available CPU resources
```

If you need a specific worker count, configure it explicitly.

---

# 6. CPU Cores vs Workers

Do NOT think:

```text
1 Worker = 1 CPU Core
```

This is not a rule you should use.

For example:

```text
CPU = 8 cores
workers = 4
```

means Playwright can use up to 4 worker processes.

It does NOT mean:

```text
Worker 1 = Core 1
Worker 2 = Core 2
Worker 3 = Core 3
Worker 4 = Core 4
```

The operating system schedules processes on CPU resources.

---

# 7. What if CPU = 1 and workers = 4?

Do not say:

> It will execute sequentially because there is only one CPU core.

That is too absolute.

You can still have multiple worker processes, but the operating system has only one CPU core available for CPU execution at a time. They cannot execute CPU instructions simultaneously on that single core.

For practical Playwright execution, the important point is:

> Increasing the worker count does not guarantee true simultaneous execution or faster execution, especially when CPU/resources are limited.

---

# 8. Example: CPU = 8, workers = 4

```text
CPU resources = 8 cores
Workers = 4

Worker 1 ──┐
Worker 2 ──┤
Worker 3 ──┼──> OS schedules these processes on available CPU resources
Worker 4 ──┘
```

Do not document it as:

```text
Worker 1 → Core 1
Worker 2 → Core 2
Worker 3 → Core 3
Worker 4 → Core 4
```

because CPU scheduling is handled by the operating system.

---

# 9. CPU = 8, Workers = 8

```text
CPU resources = 8 cores
Workers = 8
```

This allows up to 8 worker processes to participate in parallel execution.

But:

> 8 workers does NOT guarantee that every worker is permanently assigned to one CPU core or that execution will be exactly 8× faster.

Real execution depends on CPU, memory, browser workload, application capacity, database/API capacity, and test design.

---

# 10. `fullyParallel` and Workers

These are different concepts.

### `workers`

Controls the number of worker processes available for the test run.

```ts
workers: 4
```

### `fullyParallel`

Controls whether tests are allowed to run fully in parallel, including tests within the same test file.

Example:

```ts
fullyParallel: true
```

Think:

```text
workers
   ↓
How many worker processes?

fullyParallel
   ↓
How freely can tests be scheduled in parallel,
including tests from the same file?
```

---

# 11. `fullyParallel: false` with `workers: 4`

Do NOT write:

> Each worker picks one spec file and executes that file.

That is too simplistic.

A better mental model is:

```text
workers: 4
fullyParallel: false
```

Tests from different files can be distributed across workers, while tests within the same file are not fully parallelized with each other.

For example:

```text
Worker 1 → File A
            Test 1
            Test 2
            Test 3

Worker 2 → File B
            Test 1
            Test 2

Worker 3 → File C
            Test 1
            Test 2

Worker 4 → File D
            Test 1
            Test 2
```

The exact distribution depends on Playwright's scheduler.

---

# 12. Important Correction

This statement is incorrect:

> 4 Workers - 4 Spec Files - Each Test case in that spec file executes parallel.

Correct understanding:

```text
4 Workers
   +
Multiple test files
   ↓
Playwright can distribute work across workers

Inside a file:
fullyParallel: false
   ↓
Tests in that file are not fully parallelized with each other.

fullyParallel: true
   ↓
Tests can be scheduled for parallel execution more freely,
including tests within the same file.
```

---

# 13. Worker ≠ Thread

Remember this for interviews:

```text
❌ Worker = Thread

✅ Playwright Worker = worker process
```

A worker is responsible for executing Playwright tests.

---

# 14. Worker ≠ Browser

Also remember:

```text
Worker
   ↓
Execution process

Browser
   ↓
Browser instance

Browser Context
   ↓
Isolated browser session

Page
   ↓
Browser tab
```

These are different concepts.

---

# 15. Parallel Execution

Parallel execution means multiple tests can execute concurrently.

Example:

```text
Sequential

Test A
  ↓
Test B
  ↓
Test C
  ↓
Test D
```

Parallel:

```text
Worker 1 → Test A
Worker 2 → Test B
Worker 3 → Test C
Worker 4 → Test D
```

The exact scheduling is controlled by Playwright.

---

# 16. Real-Project Rule: Tests Should Be Independent

Parallel execution works best when tests do not depend on each other.

Bad example:

```text
Test A → Create Customer A

Test B → Find Customer A
```

If Test B depends on Test A, parallel execution can cause problems.

Better:

```text
Test A → Create its own customer

Test B → Create its own customer → Find it
```

Each test manages the data it needs.

---

# 17. Shared Data Can Cause Parallel Failures

Example:

```text
Worker 1 → Update Customer A → Active

Worker 2 → Update Customer A → Inactive
```

Both tests use the same data.

This can create test interference.

When designing parallel tests, pay attention to:

- Test data
- Database records
- Shared files
- Shared external resources
- Environment state
- API limits

---

# 18. Why More Workers Are Not Always Better

Example:

```text
workers: 2
```

may be faster than:

```text
workers: 20
```

depending on the environment.

More workers consume more resources.

Consider:

```text
CPU
Memory
Browser workload
Application capacity
Database capacity
API limits
Test-data isolation
CI machine size
```

So:

> More workers ≠ automatically more speed.

---

# 19. Debugging

For easier debugging, you can reduce workers:

```bash
npx playwright test --workers=1
```

Mental model:

```text
Normal execution
→ multiple workers
→ faster

Debugging
→ 1 worker
→ easier to follow
```

---

# 20. Workers vs Sharding

These are different.

### Workers

Parallel execution within a Playwright test run.

```text
One CI job
   |
   +-- Worker 1
   +-- Worker 2
   +-- Worker 3
   +-- Worker 4
```

### Sharding

Splits the test suite across multiple CI jobs/machines.

```text
CI
 |
 +-- Job 1 → Shard 1
 |
 +-- Job 2 → Shard 2
 |
 +-- Job 3 → Shard 3
```

A shard can itself use multiple workers.

---

# 21. Interview Questions You Should Know

### Q1. What is a Playwright Worker?

> A Playwright Worker is a separate worker process used by the Playwright Test Runner to execute tests.

### Q2. Why do we use workers?

> To execute tests concurrently and reduce overall test execution time.

### Q3. Where do you configure workers?

> In `playwright.config.ts` using the `workers` option, or from the CLI using `--workers`.

### Q4. Does `workers: 4` mean four CPU cores?

> No. It means Playwright can use up to four worker processes. The operating system schedules those processes on the available CPU resources.

### Q5. Does more workers always mean faster execution?

> No. CPU, memory, application capacity, database/API capacity, and test-data dependencies can limit the benefit.

### Q6. Why can parallel tests fail even though they pass sequentially?

> Usually because tests share data or application state and interfere with each other when executed concurrently.

### Q7. What is `fullyParallel`?

> It controls whether tests can be fully parallelized, including tests within the same test file.

### Q8. What is the difference between workers and sharding?

> Workers provide parallelism within a test run, while sharding splits the test suite across multiple CI jobs or machines.

---

# 22. Final Mental Map

```text
                 PLAYWRIGHT TEST RUNNER
                         |
                  Test Suite / Tests
                         |
                 ┌───────┴───────┐
                 ↓               ↓
             Worker 1        Worker 2
                 ↓               ↓
              Tests           Tests
                 ↓               ↓
              Context         Context
                 ↓               ↓
               Page            Page
```

### Remember these 7 lines

```text
1. Worker = separate worker process.
2. Worker ≠ Thread.
3. `workers` controls how many worker processes can be used.
4. Multiple workers allow parallel test execution.
5. Playwright schedules tests; don't assume workers pick tests randomly.
6. CPU cores and workers are not a 1:1 mapping.
7. Parallel tests should be independent and should avoid conflicting shared data.
```

## Most important correction to your original notes

Your original notes had three major things to fix:

```text
❌ Worker = Thread
✅ Worker = worker process

❌ workers = CPU cores / 2 as a universal default
✅ Do not memorize CPU/2 as Playwright's universal default.

❌ 4 workers = 4 spec files, one file per worker
✅ Playwright schedules tests across workers; don't assume a fixed
   worker-to-file mapping.
```
