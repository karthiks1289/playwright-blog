# Playwright Trace Viewer

## Trace Viewer

1. It provides a detailed **"execution record of a test case"**, including the actions performed, screenshots/snapshots, DOM information, network activity, console messages, and other execution details.

2. It helps us analyze exactly what happened during test execution, especially when a test fails, so we can identify the specific action/step where the failure occurred.

3. We can enable tracing in `playwright.config.ts` using the `trace` property.

```ts
use: {
    trace: 'on'
}
```

4. We can also enable tracing using the CLI:

```bash
npx playwright test --trace=on
```

5. After execution, we can open the generated trace in **Trace Viewer** to step through the test execution and inspect what happened at each point.

6. In the **`test-results`** folder, we can find the results of individual tests along with their generated artifacts, such as trace files, screenshots, videos, etc.

## 🧠 Simple Mental Map

**Trace Viewer = See exactly what happened during test execution**
