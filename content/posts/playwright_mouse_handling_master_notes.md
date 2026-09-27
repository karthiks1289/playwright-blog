# Playwright Mouse Handling --- Master Notes

## 1. What is Mouse Handling in Playwright?

Mouse handling means controlling mouse actions such as:

-   Click
-   Double-click
-   Right-click
-   Hover
-   Drag and drop
-   Mouse movement
-   Press and release
-   Custom mouse gestures
-   Coordinate-based interactions

The most important thing is **not memorizing methods**. First decide
whether the interaction is a normal UI action or a low-level/custom
mouse interaction.

------------------------------------------------------------------------

# 2. Mental Map

``` text
                         MOUSE ACTION
                              |
                    Can I identify the element?
                       /                \
                     YES                CUSTOM / COORDINATE
                      |                       |
                      v                       v
                   LOCATOR                page.mouse
                      |                       |
          +-----------+-----------+       +---+---+---+
          |           |           |       |       |   |
        click       hover      dblclick  move    down up
          |
          +-- right click
          +-- modifiers
          +-- position
          +-- dragTo()
```

### Golden rule

> **For a normal identifiable UI element, prefer Locator APIs. Use
> `page.mouse` when you need low-level, viewport-coordinate, or custom
> mouse control.**

------------------------------------------------------------------------

# 3. Locator Mouse Actions

## 3.1 Normal click

``` ts
await page.getByRole('button', { name: 'Save' }).click();
```

Use this when you can identify the element.

Playwright performs actionability checks, scrolls the element into view
when needed, and then performs the click.

------------------------------------------------------------------------

## 3.2 Hover

``` ts
await page.getByText('Products').hover();
```

Real example:

``` ts
await page.getByText('Products').hover();

await expect(
    page.getByText('Mobiles')
).toBeVisible();
```

Typical use:

-   Menus
-   Tooltips
-   Hover-based controls

------------------------------------------------------------------------

## 3.3 Double-click

``` ts
await page.getByText('EMP101').dblclick();
```

Use `dblclick()` for a normal double-click because the intent is clear.

> Method name is `dblclick()`, not `dblClick()`.

------------------------------------------------------------------------

## 3.4 Right-click

There is no need for a `rightClick()` locator method.

Use:

``` ts
await page.getByText('EMP101').click({
    button: 'right'
});
```

Useful for:

-   Context menus
-   Row actions
-   Custom right-click behavior

------------------------------------------------------------------------

# 4. Important `click()` Options

``` ts
await element.click({
    button: 'right',
    clickCount: 2,
    modifiers: ['Control'],
    position: { x: 20, y: 10 },
    force: true
});
```

  Option         Meaning
  -------------- ----------------------------------------------------
  `button`       Mouse button: `left`, `right`, `middle`
  `clickCount`   Number of clicks
  `modifiers`    `Alt`, `Control`, `ControlOrMeta`, `Meta`, `Shift`
  `position`     Point relative to the element
  `force`        Bypasses actionability checks

### `clickCount`

``` ts
await element.click({ clickCount: 2 });
```

performs two clicks.

For a normal double-click, prefer:

``` ts
await element.dblclick();
```

------------------------------------------------------------------------

# 5. Modifiers

Example:

``` ts
await page.getByText('EMP101').click();

await page.getByText('EMP102').click({
    modifiers: ['Control']
});
```

This performs the second click with the Control modifier active.

Useful for interactions such as application-specific multi-selection.

Important: the application determines what `Control`, `Shift`, etc.
mean. Do not assume every application uses them in the same way.

Also available:

``` ts
['Alt']
['Control']
['ControlOrMeta']
['Meta']
['Shift']
```

`ControlOrMeta` resolves to Control on Windows/Linux and Meta on macOS.

------------------------------------------------------------------------

# 6. `position` --- Very Important

Suppose:

``` ts
const canvas = page.locator('#canvas');
```

You need to click 100px from the left and 50px from the top **inside the
canvas**:

``` ts
await canvas.click({
    position: { x: 100, y: 50 }
});
```

The coordinates are relative to the **top-left of the element's padding
box**.

### Mental model

``` text
Canvas
+--------------------------------+
|                                |
|       (100,50)                 |
|          *                     |
|                                |
+--------------------------------+
```

Remember:

``` text
locator.click({ position })
        |
        +--> ELEMENT-relative coordinates
```

------------------------------------------------------------------------

# 7. `force: true`

Example:

``` ts
await button.click({
    force: true
});
```

`force` tells Playwright to bypass its normal actionability checks.

### Don't use it as the first solution

If a loading overlay is covering a button:

``` text
Click fails
   |
   v
Understand why
   |
   v
Wait for application state / overlay
   |
   v
Normal click()
```

Example:

``` ts
await expect(page.locator('.loading-overlay')).toBeHidden();

await page.getByRole('button', { name: 'Submit' }).click();
```

Use `force` only when you have a valid reason to intentionally bypass
the checks.

------------------------------------------------------------------------

# 8. Drag and Drop

## 8.1 Normal element-to-element drag

If source and target are identifiable:

``` ts
const source = page.getByText('Product A');
const target = page.getByText('Drop Product Here');

await source.dragTo(target);
```

### Mental model

``` text
Source + Target identifiable
          |
          v
       dragTo()
```

------------------------------------------------------------------------

## 8.2 Drag from a specific point

`dragTo()` can also specify source and target positions:

``` ts
await source.dragTo(target, {
    sourcePosition: { x: 30, y: 20 },
    targetPosition: { x: 10, y: 15 }
});
```

These positions are relative to the respective element padding boxes.

This is an important correction to a common oversimplification:

> You do **not** automatically need `page.mouse` just because a drag
> starts at a specific point.

Use `dragTo()` when its options provide the control you need. Use
`page.mouse` for genuinely custom mouse behavior or a gesture that
`dragTo()` does not adequately represent.

------------------------------------------------------------------------

# 9. `page.mouse`

Every `page` has a Mouse object:

``` ts
page.mouse
```

It operates in **main-frame CSS pixels relative to the top-left of the
viewport**.

Example:

``` ts
await page.mouse.move(100, 100);
```

means:

> Move the mouse to viewport coordinate `(100,100)`.

It does **not** mean:

-   100px inside the currently hovered element
-   physical monitor coordinates
-   a locator lookup

------------------------------------------------------------------------

# 10. `page.mouse` Methods You Need

  Method             Meaning
  ------------------ --------------------------------------
  `move(x, y)`       Move mouse to viewport coordinates
  `click(x, y)`      Click at viewport coordinates
  `down()`           Dispatch mouse-down / press button
  `up()`             Dispatch mouse-up / release button
  `dblclick(x, y)`   Double-click at viewport coordinates
  `wheel(x, y)`      Dispatch wheel event

For this topic, the most important are:

``` ts
page.mouse.move()
page.mouse.click()
page.mouse.down()
page.mouse.up()
```

------------------------------------------------------------------------

# 11. `page.mouse.click()`

``` ts
await page.mouse.click(400, 300);
```

`mouse.click()` is a shortcut for:

``` text
move → down → up
```

at the specified viewport coordinate.

It does **not** locate a DOM element first.

Use it for situations such as:

-   Canvas interaction
-   Coordinate-based UI
-   Low-level mouse interaction

------------------------------------------------------------------------

# 12. `mouse.down()` and `mouse.up()`

Think of a physical mouse:

``` text
move()
  |
  v
down()      <-- press and hold
  |
  v
move()
move()
move()
  |
  v
up()        <-- release
```

Example:

``` ts
await page.mouse.move(100, 100);
await page.mouse.down();

await page.mouse.move(200, 150);
await page.mouse.move(300, 200);
await page.mouse.move(400, 250);

await page.mouse.up();
```

This pattern is useful for:

-   Canvas drawing
-   Custom drag gestures
-   Graphical controls
-   Other continuous mouse interactions

------------------------------------------------------------------------

# 13. Locator Coordinates vs Mouse Coordinates

This is one of the most important interview distinctions.

## Element-relative

``` ts
await canvas.click({
    position: { x: 100, y: 50 }
});
```

Means:

> `(100,50)` relative to the canvas.

## Viewport-relative

``` ts
await page.mouse.click(100, 50);
```

Means:

> `(100,50)` relative to the main-frame viewport.

### Memory trick

``` text
ELEMENT  → locator position
VIEWPORT  → page.mouse
```

------------------------------------------------------------------------

# 14. When Should I Use Locator vs `page.mouse`?

## Use Locator when:

-   You can identify the UI element.
-   You are clicking a button.
-   You are hovering a menu.
-   You are double-clicking a row.
-   You are right-clicking an element.
-   You are dragging one identifiable element to another.
-   You need a specific point inside an identified element.

Examples:

``` ts
await button.click();

await menu.hover();

await row.dblclick();

await row.click({ button: 'right' });

await source.dragTo(target);

await canvas.click({
    position: { x: 100, y: 50 }
});
```

## Use `page.mouse` when:

-   You need viewport-coordinate interaction.
-   You need custom mouse movement.
-   You need a custom gesture.
-   You are drawing on a canvas.
-   You need low-level mouse down/up control.

Example:

``` ts
await page.mouse.move(100, 100);
await page.mouse.down();
await page.mouse.move(200, 150);
await page.mouse.move(300, 200);
await page.mouse.up();
```

------------------------------------------------------------------------

# 15. Real-Project Examples

## Tooltip

``` ts
await page.getByRole('button', { name: 'Help' }).hover();

await expect(
    page.getByText('Employee ID must be unique')
).toBeVisible();
```

## Context menu

``` ts
await page.getByText('EMP101').click({
    button: 'right'
});
```

## Double-click

``` ts
await page.getByText('EMP102').dblclick();
```

## Ctrl + click

``` ts
await page.getByText('EMP101').click();

await page.getByText('EMP102').click({
    modifiers: ['Control']
});
```

## Element-to-element drag

``` ts
const employeeCard = page.getByText('Employee Card');
const qaDropArea = page.getByText('QA Team Drop Area');

await employeeCard.dragTo(qaDropArea);
```

## Point inside canvas

``` ts
const canvas = page.locator('#canvas');

await canvas.click({
    position: { x: 150, y: 100 }
});
```

## Canvas drawing

``` ts
await page.mouse.move(100, 100);
await page.mouse.down();

await page.mouse.move(200, 150);
await page.mouse.move(300, 200);
await page.mouse.move(400, 250);

await page.mouse.up();
```

------------------------------------------------------------------------

# 16. Tricky Scenarios

## Scenario 1: Button covered by overlay

Don't immediately do:

``` ts
await button.click({ force: true });
```

First understand and synchronize with the application state.

------------------------------------------------------------------------

## Scenario 2: Canvas is identifiable

Don't automatically use:

``` ts
await page.mouse.click(150, 100);
```

If the requirement is specifically:

> Click `(150,100)` inside the canvas

prefer:

``` ts
await canvas.click({
    position: { x: 150, y: 100 }
});
```

------------------------------------------------------------------------

## Scenario 3: Normal drag

Prefer:

``` ts
await source.dragTo(target);
```

Don't manually implement mouse down/move/up unless you actually need
custom control.

------------------------------------------------------------------------

## Scenario 4: Custom drawing

Use:

``` ts
move → down → move(s) → up
```

with `page.mouse`.

------------------------------------------------------------------------

# 17. Common Mistakes

### ❌ Mistake 1

``` ts
await page.mouse();
```

Wrong.

`page.mouse` is an object.

Correct:

``` ts
await page.mouse.move(100, 100);
```

------------------------------------------------------------------------

### ❌ Mistake 2

``` ts
await element.dblClick();
```

Correct:

``` ts
await element.dblclick();
```

------------------------------------------------------------------------

### ❌ Mistake 3

Using viewport coordinates when you really mean coordinates inside an
element.

``` ts
await page.mouse.click(100, 50);
```

vs.

``` ts
await element.click({
    position: { x: 100, y: 50 }
});
```

Know which coordinate system you need.

------------------------------------------------------------------------

### ❌ Mistake 4

Using `page.mouse` for every click.

If you can identify:

``` ts
await page.getByRole('button', { name: 'Save' }).click();
```

prefer the locator.

------------------------------------------------------------------------

### ❌ Mistake 5

Using `force: true` to hide a synchronization problem.

First understand why the element is not actionable.

------------------------------------------------------------------------

# 18. Interview-Ready Answers

### Q1. What is the difference between `locator.click()` and `page.mouse.click()`?

> `locator.click()` clicks an identified DOM element and performs
> Playwright's normal actionability handling. `page.mouse.click(x, y)`
> performs a low-level mouse click at viewport coordinates without first
> identifying a DOM element.

### Q2. When would you use `page.mouse`?

> I use `page.mouse` for low-level or coordinate-based interactions,
> such as canvas drawing, custom mouse gestures, or interactions where I
> need direct control over mouse movement and button state.

### Q3. What is the difference between `position` and `page.mouse` coordinates?

> `locator.click({ position })` uses coordinates relative to the
> element's padding box. `page.mouse` uses coordinates relative to the
> main-frame viewport.

### Q4. When would you use `dragTo()`?

> When I have identifiable source and target elements and need a normal
> drag-and-drop interaction.

### Q5. When would you use manual mouse actions for drag?

> When I need custom low-level control over the mouse path, button
> state, or gesture that normal `dragTo()` does not represent
> adequately.

### Q6. Why not immediately use `force: true`?

> Because force bypasses Playwright's normal actionability checks. I
> first investigate synchronization or UI-state problems and use force
> only when intentionally bypassing those checks is appropriate.

------------------------------------------------------------------------

# 19. Practical Assignment

## Assignment 1 --- Tooltip

``` ts
await page.getByText('Help').hover();

await expect(
    page.getByText('Employee ID must be unique')
).toBeVisible();
```

## Assignment 2 --- Context Menu

Right-click `EMP101`.

## Assignment 3 --- Double Click

Double-click `EMP102`.

## Assignment 4 --- Multi-select

Select `EMP101`, then Control-click `EMP102`.

## Assignment 5 --- Drag and Drop

Drag `Employee Card` to `QA Team Drop Area`.

## Assignment 6 --- Canvas Point

Click `(150,100)` inside the canvas.

## Assignment 7 --- Custom Drawing

Draw:

``` text
(100,100)
    ↓
(200,150)
    ↓
(300,200)
    ↓
(400,250)
```

using `page.mouse`.

## Assignment 8 --- Decision Practice

Choose Locator or `page.mouse`:

  Requirement                            Expected approach
  -------------------------------------- ----------------------
  Click Submit                           Locator
  Hover Products                         Locator
  Draw on canvas                         `page.mouse`
  Drag Product → Cart                    Locator / `dragTo()`
  Click point inside identified canvas   Locator + `position`
  Custom mouse gesture                   `page.mouse`
  Right-click employee                   Locator

------------------------------------------------------------------------

# 20. What You Actually Need to Remember

### 🔴 MUST KNOW

``` text
Normal UI element
      ↓
    Locator
```

``` text
Custom / low-level mouse interaction
      ↓
   page.mouse
```

### Remember these methods

``` ts
click()
dblclick()
hover()
click({ button: 'right' })
click({ modifiers: ['Control'] })
click({ position: { x, y } })
dragTo(target)

page.mouse.move(x, y)
page.mouse.click(x, y)
page.mouse.down()
page.mouse.up()
```

### The biggest interview distinction

``` text
locator.click({ position })
→ element-relative

page.mouse.click(x, y)
→ viewport-relative
```

### Drag rule

``` text
Normal source → target
        ↓
    dragTo()

Custom gesture
        ↓
    page.mouse
```

### Force rule

``` text
Don't use force just because click failed.
First understand the reason.
```

### Final memory line

> **Locator = "interact with this element."\
> `page.mouse` = "control the mouse here."**

------------------------------------------------------------------------

## Official references

-   Playwright Locator API: citeturn0search0
-   Playwright Mouse API: citeturn0search1
