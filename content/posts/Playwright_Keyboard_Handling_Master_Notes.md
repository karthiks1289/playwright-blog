# Playwright Keyboard Handling with TypeScript — Master Note

> **Purpose:** One practical note for LEARN → IMPLEMENT → INTERVIEW → BLOG → REVISION  
> **Style:** Simple English, real-project focused, interview focused  
> **Accuracy:** Based on the current Playwright documentation checked on 2026-09-26.

---

# 1. Big Picture

Keyboard handling means making Playwright perform keyboard actions such as:

- Enter
- Tab / Shift+Tab
- Escape
- Arrow keys
- Home / End
- PageUp / PageDown
- Backspace / Delete
- Keyboard shortcuts
- Modifier keys
- Holding and releasing keys
- Text insertion / sequential typing

The important skill is **not memorizing key names**.

The real SDET skill is:

```text
UI requirement
      ↓
What keyboard behavior is required?
      ↓
Is there a specific element?
      ↓
Choose the correct Playwright API
      ↓
Perform the action
      ↓
Verify the result
```

---

# 2. The Core Mental Map

```text
                 KEYBOARD REQUIREMENT
                         |
              +----------+----------+
              |                     |
       Specific element?       Page/focused context?
              |                     |
              ↓                     ↓
      locator.press()       page.keyboard.press()
              |
              |
       Need to HOLD a key?
              |
              ↓
     page.keyboard.down()
              |
        perform action
              |
              ↓
      page.keyboard.up()
```

For text:

```text
Normal form input
      ↓
locator.fill()                  ← usually preferred

Special keyboard handling
      ↓
locator.pressSequentially()     ← modern locator API

Need direct text insertion
      ↓
page.keyboard.insertText()
```

---

# 3. `locator.press()` — 🔴 MUST KNOW

## Simple meaning

Use `locator.press()` when you want to perform a keyboard action on a **specific element**.

```ts
const searchBox = page.getByRole('textbox', { name: 'Search' });

await searchBox.press('Enter');
```

Playwright focuses the matching element and then performs the key action.

### Common examples

```ts
await searchBox.press('Enter');
await input.press('Escape');
await dropdown.press('ArrowDown');
await input.press('ControlOrMeta+A');
await input.press('Shift+ArrowLeft');
```

### Key mental model

> **"I know the element that should receive this keyboard action." → `locator.press()`**

### Important current behavior

`locator.press()` focuses the matching element, then uses keyboard down/up behavior.

---

# 4. `page.keyboard.press()` — 🔴 MUST KNOW

`page.keyboard` operates through the page's keyboard context.

```ts
await page.keyboard.press('Escape');
```

It is useful when the interaction is naturally page/focus based, for example:

- Close the currently open modal with Escape
- Send a page-level shortcut
- Continue a keyboard-only workflow after focus has been established
- Send keyboard input to the currently focused UI

### Important

Do not think:

> "`page.keyboard` automatically finds the correct element."

Focus still matters.

If an element needs to receive the keyboard input, establish or verify the correct focus when appropriate.

Example:

```ts
await searchBox.focus();
await page.keyboard.press('Enter');
```

### Practical decision

```text
Specific target element
        ↓
locator.press()

Current page/focused context
        ↓
page.keyboard.press()
```

---

# 5. `locator.press()` vs `page.keyboard.press()`

| Situation | Preferred approach |
|---|---|
| Press Enter on Search box | `searchBox.press('Enter')` |
| Press Escape on a currently active page/modal workflow | `page.keyboard.press('Escape')` |
| Press Ctrl+A in a known input | `input.press('ControlOrMeta+A')` |
| Send a shortcut to the current focused UI | `page.keyboard.press(...)` |
| Need to explicitly target a locator | `locator.press()` |

### Interview answer

> "`locator.press()` targets a specific element and focuses it before performing the keyboard action. `page.keyboard.press()` sends the key through the page's keyboard context, so the currently focused element and application keyboard handling matter."

---

# 6. Common Key Names — 🔴 MUST KNOW

Playwright supports logical key names such as:

```text
Enter
Tab
Escape
Backspace
Delete
ArrowUp
ArrowDown
ArrowLeft
ArrowRight
Home
End
PageUp
PageDown
Insert
F1 ... F12
```

Examples:

```ts
await input.press('Enter');
await input.press('Escape');
await input.press('Tab');
await input.press('Backspace');
await input.press('Delete');
await input.press('ArrowDown');
await input.press('Home');
await input.press('End');
```

### Important

Use Playwright's key names.

For example:

```ts
await input.press('Control+A');   // correct
await input.press('Ctrl+A');      // wrong key name
```

---

# 7. Modifier Keys — 🔴 MUST KNOW

Important modifiers:

```text
Control
Shift
Alt
Meta
ControlOrMeta
```

Examples:

```ts
await input.press('Control+A');
await input.press('Shift+Tab');
await input.press('Alt+Enter');
await input.press('Meta+A');
```

For a shortcut that should work across Windows/Linux and macOS:

```ts
await input.press('ControlOrMeta+A');
```

---

# 8. `ControlOrMeta` — 🔴 MUST KNOW

Useful for cross-platform tests.

```ts
await input.press('ControlOrMeta+A');
```

Meaning:

```text
Windows / Linux → Control+A
macOS           → Meta(Command)+A
```

### Use it for

- Select all
- Copy
- Paste
- Cut
- Undo
- Other platform-dependent shortcuts

Examples:

```ts
await input.press('ControlOrMeta+A');
await input.press('ControlOrMeta+C');
await input.press('ControlOrMeta+V');
await input.press('ControlOrMeta+X');
await input.press('ControlOrMeta+Z');
```

### Interview answer

> "`ControlOrMeta` resolves to Control on Windows/Linux and Meta on macOS, making keyboard shortcuts portable across those platforms."

---

# 9. `keyboard.down()` and `keyboard.up()` — 🔴 MUST KNOW

These are methods of `page.keyboard`.

```ts
await page.keyboard.down('Shift');

// perform another action while Shift is held

await page.keyboard.up('Shift');
```

There is no locator equivalent such as:

```ts
locator.down() // ❌
locator.up()   // ❌
```

### `down()`

Dispatches a `keydown` and keeps the key active.

### `up()`

Dispatches a `keyup` and releases the key.

### Real example: Shift + click

```ts
await page.keyboard.down('Shift');

await row2.click();

await page.keyboard.up('Shift');
```

This is useful when the modifier must remain held **across another action**.

### Mental model

```text
press()
  ↓
complete key action

down()
  ↓
key is held

other action
  ↓
up()
  ↓
key released
```

### Interview answer

> "`press()` performs a complete key press. `down()` and `up()` give explicit control over holding and releasing a key, which is useful for modifier-based multi-step interactions."

---

# 10. Keyboard Shortcuts — 🔴 MUST KNOW

Examples:

```ts
await input.press('ControlOrMeta+A');
await input.press('ControlOrMeta+C');
await input.press('ControlOrMeta+V');
await input.press('ControlOrMeta+X');
await input.press('ControlOrMeta+Z');
await input.press('Shift+Tab');
```

### Common shortcut map

| Shortcut | Typical purpose |
|---|---|
| `ControlOrMeta+A` | Select all |
| `ControlOrMeta+C` | Copy |
| `ControlOrMeta+V` | Paste |
| `ControlOrMeta+X` | Cut |
| `ControlOrMeta+Z` | Undo |
| `Shift+Tab` | Previous focusable element |

---

# 11. Focus — 🔴 MUST KNOW

**Focus = the element currently receiving keyboard input.**

You can explicitly focus:

```ts
await username.focus();
```

Then use page keyboard:

```ts
await page.keyboard.press('Enter');
```

Verify focus:

```ts
await expect(username).toBeFocused();
```

### Keyboard navigation example

```ts
await username.press('Tab');
await expect(password).toBeFocused();
```

### Important

Do not merely send `Tab` and assume the correct element received focus.

For accessibility or keyboard-navigation tests:

```text
Keyboard action
      ↓
Expected focus
      ↓
Assert focus
```

---

# 12. `Tab` and `Shift+Tab` — 🔴 MUST KNOW

```text
Tab
 ↓
Forward focus movement

Shift+Tab
 ↑
Backward focus movement
```

Example:

```ts
await username.press('Tab');
await expect(password).toBeFocused();
```

Backward:

```ts
await password.press('Shift+Tab');
await expect(username).toBeFocused();
```

### Real use

Keyboard-only form testing:

```ts
await username.fill('Karthik');

await username.press('Tab');
await expect(password).toBeFocused();

await password.fill('Password123');

await password.press('Tab');
await expect(loginButton).toBeFocused();

await loginButton.press('Enter');
```

---

# 13. Text Selection — 🔴 MUST KNOW

## Select all

```ts
await input.press('ControlOrMeta+A');
```

## Move cursor

```ts
await input.press('Home');
await input.press('End');
```

## Select while moving

```ts
await input.press('Shift+ArrowLeft');
await input.press('Shift+ArrowRight');
```

Mental model:

```text
ArrowLeft
    ↓
Move cursor

Shift+ArrowLeft
    ↓
Move cursor + extend selection
```

Example:

```text
Playwright|
```

Three times:

```ts
await input.press('Shift+ArrowLeft');
await input.press('Shift+ArrowLeft');
await input.press('Shift+ArrowLeft');
```

selects the last three characters.

---

# 14. Copy / Paste — 🔴 MUST KNOW

Perform copy:

```ts
await source.press('ControlOrMeta+A');
await source.press('ControlOrMeta+C');
```

Focus target:

```ts
await target.focus();
```

Paste:

```ts
await page.keyboard.press('ControlOrMeta+V');
```

Verify result:

```ts
const expected = await source.inputValue();

await expect(target).toHaveValue(expected);
```

### Important

Performing:

```ts
ControlOrMeta+C
```

does **not by itself verify** the clipboard content.

Action and verification are separate.

---

# 15. Text Entry: `fill()` vs `pressSequentially()` vs `insertText()` — 🔴 MUST KNOW

This is an important current-Playwright correction.

## `fill()` — preferred for normal form input

```ts
await username.fill('Karthik');
```

Playwright recommends `locator.fill()` for normal text input in most cases.

Use it when your requirement is simply:

> Put this value into the field.

---

## `locator.pressSequentially()` — current locator API for character-by-character typing

Current Playwright documentation recommends `locator.pressSequentially()` when you specifically need to press characters one by one because the page has special keyboard handling.

```ts
await input.pressSequentially('Hello');
```

With delay:

```ts
await input.pressSequentially('Hello', { delay: 100 });
```

It focuses the locator and sends keyboard events for each character.

### Real use

Use it when the application reacts to individual typing events.

Examples:

- Search suggestions triggered during typing
- Custom editor behavior
- Validation/formatting tied to key events
- UI that deliberately handles individual keyboard input

---

## `page.keyboard.insertText()`

```ts
await input.focus();
await page.keyboard.insertText('Hello');
```

Important current behavior:

- It dispatches the `input` event.
- It does **not** emit `keydown`, `keyup`, or `keypress`.
- Modifier keys do not affect `insertText()`.

So it is **text insertion**, not full key-by-key simulation.

---

## What about `keyboard.type()`?

`page.keyboard.type()` still exists in Playwright, but the current API documentation marks it **deprecated**. For new TypeScript Playwright code, do not choose it as the default typing API.

Current guidance is:

- Normal text input → `locator.fill()`
- Fine-grained character-by-character keyboard behavior → `locator.pressSequentially()`
- Existing legacy code may still contain `keyboard.type()`; understand it, but prefer the modern locator APIs when writing new tests.

The same deprecation applies to the older `page.type()` API.

`keyboard.type()` is therefore mainly **interview/legacy-code knowledge**, not a preferred modern API.

### Modern decision map

```text
Need normal form value?
        ↓
locator.fill()

Need character-by-character keyboard events?
        ↓
locator.pressSequentially()

Need direct text insertion into focused editable UI?
        ↓
page.keyboard.insertText()

Need a special key / shortcut?
        ↓
locator.press() / page.keyboard.press()
```

---

# 16. `press()` vs `pressSequentially()` vs `insertText()`

| API | Main purpose | Keyboard events | Priority |
|---|---|---|---|
| `locator.press()` | One key / shortcut | Key action | 🔴 |
| `locator.pressSequentially()` | Character-by-character typing | Keyboard events per character | 🔴 |
| `page.keyboard.press()` | Page/focus keyboard action | Key action | 🔴 |
| `page.keyboard.insertText()` | Insert text | `input` only | 🟡 |
| `page.keyboard.type()` | Deprecated legacy character typing API | Per-character keyboard events | ⚪ |
| `locator.fill()` | Set normal form value | Input event | 🔴 |

---

# 17. Native Select vs Custom Dropdown — 🔴 MUST KNOW

Don't assume every dropdown should be controlled with ArrowDown.

## Native `<select>`

Use:

```ts
await country.selectOption('US');
```

If the requirement is simply:

> Select USA.

This is generally cleaner.

## Custom dropdown / combobox

Keyboard interaction may be part of the component:

```text
ArrowDown → next option
ArrowUp   → previous option
Enter     → select
Escape    → close
```

Example:

```ts
await dropdown.press('ArrowDown');
await dropdown.press('Enter');
```

### Interview thought process

> First identify whether it is a native select or a custom component. Then choose the API based on the actual requirement.

If the requirement specifically tests keyboard selection, keyboard APIs may be appropriate even for a native control.

---

# 18. Keyboard + Forms — 🔴 MUST KNOW

Example:

```ts
await username.fill('Karthik');

await username.press('Tab');
await expect(password).toBeFocused();

await password.fill('Password123');

await password.press('Tab');
await expect(loginButton).toBeFocused();

await loginButton.press('Enter');
```

Test:

```text
Input
 ↓
Tab
 ↓
Focus
 ↓
Tab
 ↓
Focus
 ↓
Enter
 ↓
Expected result
```

Always verify the result.

---

# 19. Keyboard + Modals — 🔴 MUST KNOW

Requirement:

> Press Escape to close modal.

```ts
await page.keyboard.press('Escape');

await expect(loginModal).toBeHidden();
```

The pattern is:

```text
Keyboard action
      ↓
Application behavior
      ↓
Assertion
```

---

# 20. Keyboard + Tables / Editable Grids — 🟡 SHOULD KNOW

Custom enterprise grids may support:

```text
ArrowRight → next cell
ArrowLeft  → previous cell
ArrowDown  → next row
ArrowUp    → previous row
Enter      → edit
Escape     → cancel
```

Example:

```ts
await statusCell.press('ArrowRight');
await expect(amountCell).toBeFocused();
```

Important:

These behaviors are **application/component-specific**. Playwright does not decide that ArrowRight means "next grid cell."

The application must implement that behavior.

---

# 21. Keyboard + Mouse — 🟡 SHOULD KNOW

When a modifier must remain held during a mouse action:

```ts
await page.keyboard.down('Shift');

await row2.click();

await page.keyboard.up('Shift');
```

The application/browser decides what Shift+click means.

Playwright only provides the keyboard/mouse actions.

### Alternative: mouse action modifiers

For a simple modifier-click, Playwright also supports modifiers directly on locator mouse actions:

```ts
await row2.click({ modifiers: ['Shift'] });
```

This can be cleaner when you only need the modifier for that one click.

Use `keyboard.down()` / `keyboard.up()` when the key must remain held across multiple separate actions.

---

# 22. Menus, Date Pickers, Comboboxes — 🟡 SHOULD KNOW

Common patterns include:

```text
ArrowUp / ArrowDown → navigate
Enter               → select
Escape              → close
Tab                 → move focus
```

But always inspect the actual component behavior.

Don't assume every date picker or menu uses the same keys.

---

# 23. Custom Components — 🟡 SHOULD KNOW

For custom UI:

```text
div-based dropdown
ARIA combobox
editable grid
rich-text editor
custom menu
```

Keyboard behavior may be implemented by application JavaScript.

Your approach:

```text
Understand component
      ↓
Identify supported keyboard behavior
      ↓
Use Playwright keyboard API
      ↓
Verify UI state
```

---

# 24. Accessibility / Keyboard-only Testing — 🔴 MUST KNOW

A realistic keyboard-only workflow:

```text
Focus first control
      ↓
Tab
      ↓
Verify focus
      ↓
Tab
      ↓
Verify focus
      ↓
Enter / Space
      ↓
Verify result
```

Example:

```ts
await username.focus();

await expect(username).toBeFocused();

await username.press('Tab');
await expect(password).toBeFocused();

await password.press('Tab');
await expect(loginButton).toBeFocused();

await loginButton.press('Enter');
```

This is useful for testing:

- Focus order
- Keyboard operability
- Keyboard submission
- Modal behavior
- Menus/dropdowns
- Custom controls

---

# 25. Common Mistakes

## Mistake 1 — Using `Ctrl`

```ts
await input.press('Ctrl+A'); // ❌
```

Use:

```ts
await input.press('Control+A');
```

---

## Mistake 2 — Using keyboard typing for ordinary input

Avoid unnecessary:

```ts
await page.keyboard.type('Karthik');
```

Prefer:

```ts
await input.fill('Karthik');
```

---

## Mistake 3 — Using keyboard when direct API exists

Native select:

```ts
await country.selectOption('US');
```

instead of simulating several ArrowDown actions when keyboard behavior isn't what you're testing.

---

## Mistake 4 — Forgetting focus

With page keyboard:

```ts
await page.keyboard.press('Enter');
```

Ask:

> What currently has focus?

---

## Mistake 5 — Performing an action without verifying result

Bad:

```ts
await modal.press('Escape');
```

Better:

```ts
await modal.press('Escape');
await expect(modal).toBeHidden();
```

---

## Mistake 6 — Assuming component behavior

Don't assume:

```text
ArrowDown = next option
```

for every custom component.

Verify how the application actually works.

---

## Mistake 7 — Confusing `down()` with `press()`

```text
press()
→ complete key action

down()
→ key is held

up()
→ key is released
```

---

# 26. API Decision Table

| Requirement | API | Priority |
|---|---|---|
| Press Enter on a textbox | `locator.press('Enter')` | 🔴 |
| Press Escape on page/focused UI | `page.keyboard.press('Escape')` | 🔴 |
| Press Ctrl/Cmd+A | `locator.press('ControlOrMeta+A')` | 🔴 |
| Hold Shift across a click | `keyboard.down()` + `keyboard.up()` | 🔴 |
| Normal text input | `locator.fill()` | 🔴 |
| Character-by-character keyboard behavior | `locator.pressSequentially()` | 🔴 |
| Direct text insertion | `keyboard.insertText()` | 🟡 |
| Existing code uses `keyboard.type()` | Understand it; consider modern alternatives | 🟡 |
| Verify focus | `expect(locator).toBeFocused()` | 🔴 |
| Native select | `selectOption()` | 🔴 |
| Custom dropdown keyboard behavior | Keyboard APIs as required | 🟡 |
| Keyboard-only workflow | `press()` + focus assertions | 🔴 |

---

# 27. Interview Questions

## Q1. When would you use `locator.press()` vs `page.keyboard.press()`?

### Interview-ready answer

> "`locator.press()` is used when I want to perform a keyboard action on a specific element and it focuses that element first. `page.keyboard.press()` sends keyboard input through the page's keyboard context, so the current focus and application keyboard handling matter."

---

## Q2. How do you press Ctrl+A?

```ts
await input.press('Control+A');
```

For cross-platform:

```ts
await input.press('ControlOrMeta+A');
```

---

## Q3. How do you hold a modifier?

```ts
await page.keyboard.down('Shift');

// action

await page.keyboard.up('Shift');
```

---

## Q4. What is the difference between `press`, `down`, and `up`?

> "`press()` performs a complete key press. `down()` dispatches the key-down and keeps the key active. `up()` releases the key."

---

## Q5. What is the difference between `insertText()` and character-by-character typing?

> "`insertText()` performs text insertion and emits the input event without individual keydown/keyup/keypress events. For character-by-character keyboard behavior in modern locator-based Playwright tests, I would use `locator.pressSequentially()`."

---

## Q6. Should you always use `keyboard.type()` for typing?

> "No. For normal form fields, Playwright recommends `locator.fill()`. If the application has special keyboard handling and I need character-by-character events, I use `locator.pressSequentially()`."

---

## Q7. How do you test keyboard navigation?

> "I use Tab or Shift+Tab, then verify the expected control receives focus with `toBeFocused()`. I continue with Enter, Escape, or other keys based on the component behavior and verify the resulting UI state."

---

## Q8. How do you handle Ctrl/Cmd differences?

```ts
await input.press('ControlOrMeta+A');
```

> "`ControlOrMeta` maps to Control on Windows/Linux and Meta on macOS."

---

## Q9. How would you test a keyboard-only login?

```ts
await username.fill('Karthik');
await username.press('Tab');
await expect(password).toBeFocused();

await password.fill('Password123');
await password.press('Tab');
await expect(loginButton).toBeFocused();

await loginButton.press('Enter');
```

---

## Q10. How do you handle a custom dropdown?

> "First I identify how the custom component is implemented and what keyboard behavior it supports. If keyboard selection is part of the requirement, I use keys such as ArrowDown, ArrowUp, Enter, or Escape and verify the selected value."

---

# 28. Practical Assignments

## Assignment 1 — Login Keyboard Flow 🔴

Automate:

```text
Username
 ↓ Tab
Password
 ↓ Tab
Login
 ↓ Enter
```

Verify focus after each Tab.

---

## Assignment 2 — Text Selection 🔴

Input contains:

```text
Playwright Automation
```

Move to the end and select the last 10 characters using keyboard actions.

Verify the resulting behavior.

---

## Assignment 3 — Copy/Paste 🔴

Copy the value from one field and paste it into another.

Verify the target value.

Use:

```text
ControlOrMeta+A
ControlOrMeta+C
ControlOrMeta+V
```

---

## Assignment 4 — Modal 🔴

Open a modal.

Press Escape.

Verify the modal is hidden.

---

## Assignment 5 — Custom Dropdown 🟡

Use keyboard navigation:

```text
ArrowDown
ArrowDown
Enter
```

Verify the expected option is selected.

---

## Assignment 6 — Modifier + Mouse 🟡

Hold Shift while clicking another row.

Verify the application's range-selection behavior.

---

# 29. Fast Revision Sheet

```text
Specific element
→ locator.press()

Page/focused keyboard
→ page.keyboard.press()

Hold key
→ keyboard.down()

Release key
→ keyboard.up()

Normal text
→ locator.fill()

Character-by-character keyboard behavior
→ locator.pressSequentially()

Direct text insertion
→ keyboard.insertText()

Cross-platform shortcut
→ ControlOrMeta

Forward focus
→ Tab

Backward focus
→ Shift+Tab

Close/cancel
→ Escape

Submit/select
→ Enter

Navigate
→ ArrowUp/Down/Left/Right

Cursor beginning
→ Home

Cursor end
→ End

Delete before cursor
→ Backspace

Delete after cursor
→ Delete

Verify focus
→ expect(locator).toBeFocused()
```

---

# 30. The Most Important Decision Tree

```text
                KEYBOARD REQUIREMENT
                         |
              +----------+----------+
              |                     |
        Specific element?      Page/focused UI?
              |                     |
              ↓                     ↓
       locator.press()       page.keyboard.press()
              |
              |
       Need a key held?
              |
              ↓
       keyboard.down()
              |
        perform action
              |
              ↓
       keyboard.up()
```

For text:

```text
Need normal form value?
        ↓
     fill()

Need special keyboard events
for every character?
        ↓
pressSequentially()

Need text insertion only?
        ↓
insertText()
```

---

# 31. What You Actually Need to Remember

## 🔴 MUST KNOW

1. `locator.press()` → specific element keyboard action.
2. `page.keyboard.press()` → page/focused keyboard action.
3. `down()` + `up()` → hold/release a key.
4. `ControlOrMeta` → cross-platform Control/Command shortcut.
5. `Tab` / `Shift+Tab` → forward/backward focus navigation.
6. `toBeFocused()` → verify keyboard focus.
7. `ControlOrMeta+A` → select all.
8. `Shift+Arrow` → extend text selection.
9. `ControlOrMeta+C/V` → copy/paste shortcuts.
10. `fill()` → normal text input.
11. `pressSequentially()` → character-by-character keyboard behavior.
12. `insertText()` → text insertion without individual keydown/keyup/keypress events.
13. Native `<select>` → `selectOption()` when simply selecting an option.
14. Always verify the result of keyboard actions.
15. Keyboard behavior of custom components is application-specific.

## 🟡 SHOULD KNOW

- Keyboard behavior in grids
- Custom dropdowns/comboboxes
- Modal focus behavior
- Keyboard + mouse modifiers
- Keyboard-only accessibility workflows
- Clipboard validation
- Date-picker/menu keyboard behavior

## ⚪ KNOW WHEN NEEDED

- Low-level event details
- Rare browser-specific keyboard behavior
- Complex custom editors
- OS-specific application shortcuts
- Rare key combinations

---

# 32. Final Mental Map

```text
                  PLAYWRIGHT KEYBOARD
                          |
          +---------------+---------------+
          |               |               |
       TARGET          KEY STATE        TEXT
          |               |               |
          ↓               ↓               ↓
 locator.press()      down() / up()     fill()
          |                               |
          |                          pressSequentially()
          ↓                               |
 page.keyboard.press()                insertText()
          |
          ↓
     Shortcuts
          |
   +------+------+
   |      |      |
 Control Shift  Meta
   |
ControlOrMeta
          |
          ↓
     Navigation
          |
   +------+------+------+
   |      |      |      |
  Tab   Shift+  Arrows Enter/Escape
        Tab
          |
          ↓
    Focus verification
          |
          ↓
      toBeFocused()
          |
          ↓
   Real UI components
          |
  +-------+-------+-------+
  |       |       |       |
 Forms Dropdown Grid  Modal
          |
          ↓
      Assertion
```

---

# 33. One-Sentence Interview Summary

> **"In Playwright, I use locator-based keyboard actions when a specific element is the target, page keyboard APIs when the interaction is page/focus based, `down/up` when I need to hold a key, `fill` for normal text entry, `pressSequentially` when individual typing events matter, and I always verify the resulting UI state or focus."**

---

# 34. Official References

- Playwright Locator API: https://playwright.dev/docs/api/class-locator
- Playwright Keyboard API: https://playwright.dev/docs/api/class-keyboard
- Playwright Input/Actions guide: https://playwright.dev/docs/input
- Playwright Locator guide: https://playwright.dev/docs/locators

---

## Final checklist

Before calling yourself comfortable with Playwright keyboard handling, make sure you can answer **without looking at notes**:

- Why `locator.press()`?
- When would I use `page.keyboard.press()`?
- How do I press Enter/Escape/Tab?
- How do I navigate with Arrow keys?
- How do I select all?
- How do I select part of text?
- How do I copy/paste?
- How do I hold Shift?
- What is `ControlOrMeta`?
- What is focus?
- How do I verify focus?
- `fill()` vs `pressSequentially()`?
- `pressSequentially()` vs `insertText()`?
- Native select vs custom dropdown?
- How do I test keyboard-only navigation?
- How do I verify the result of a keyboard action?

If you can explain and implement those confidently, you have the **core Playwright keyboard knowledge needed for real projects and common SDET interviews**.
