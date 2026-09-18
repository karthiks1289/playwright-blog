# Playwright Native Select Dropdown Mastery

## Learning Style

This document follows this pattern:

> **Simple understanding → Technical meaning → Syntax → Practical
> example → Important point → Interview takeaway**

The goal is not to memorize syntax. The goal is to understand **when to
use each method, what it returns, and how to validate the result**.

------------------------------------------------------------------------

# 1. First Understand: What Type of Dropdown?

## Simple Understanding

Before writing code, first identify whether the dropdown is a **real
HTML `<select>`** or a **custom dropdown**.

### Native Dropdown

``` html
<select id="country">
    <option value="in">India</option>
    <option value="us">United States</option>
    <option value="au">Australia</option>
</select>
```

For this type, Playwright provides:

``` ts
selectOption()
```

### Custom Dropdown

A custom dropdown may look like:

``` html
<div class="dropdown">India</div>

<div class="options">
    <div>India</div>
    <div>Australia</div>
</div>
```

This is **not** a `<select>`.

Usually the flow is:

``` text
Click dropdown
     ↓
Find option
     ↓
Click option
```

## Remember

> **Native `<select>` → `selectOption()`**

> **Custom dropdown → click + locate option + click**

This distinction is very important in interviews.

------------------------------------------------------------------------

# 2. Locate the Dropdown

## Simple Understanding

First capture the dropdown as a Playwright `Locator`.

``` ts
const country = page.locator("#country");
```

Or use an accessible role/name:

``` ts
const country = page.getByRole("combobox", {
    name: "Country"
});
```

## Practical Rule

If a good accessible role and name are available, prefer `getByRole()`.

Otherwise, use a reliable locator such as an ID.

------------------------------------------------------------------------

# 3. `selectOption()` --- Main Method

## Simple Understanding

> `selectOption()` is used to select an option from a **native HTML
> `<select>` dropdown**.

This is the main Playwright method for native dropdowns.

------------------------------------------------------------------------

## 3.1 Select by Value

HTML:

``` html
<option value="in">India</option>
```

Code:

``` ts
await country.selectOption({ value: "in" });
```

### Memory

``` text
HTML value → select using value
```

------------------------------------------------------------------------

## 3.2 Select by Visible Text / Label

HTML:

``` html
<option value="in">India</option>
```

Code:

``` ts
await country.selectOption({ label: "India" });
```

## Simple Understanding

`label` means the **visible text shown to the user**.

``` text
Visible text = label
HTML value  = value
```

------------------------------------------------------------------------

## 3.3 Select by Index

``` ts
await country.selectOption({ index: 2 });
```

## Simple Understanding

Select based on the option's position.

### Important

> **Index starts from 0.**

Example:

``` text
0 → India
1 → United States
2 → Australia
```

### Interview Point

Index is generally less stable than value or label because the order of
options can change.

------------------------------------------------------------------------

# 4. `selectOption()` Return Value

## Simple Understanding

> `selectOption()` returns the selected option's **value** as an array.

Example:

``` ts
const selectedValue = await country.selectOption({
    label: "Australia"
});

console.log(selectedValue);
```

Result:

``` ts
["au"]
```

Because:

``` html
<option value="au">Australia</option>
```

## Important

Do not confuse:

``` text
Australia → visible text
au        → value
```

------------------------------------------------------------------------

# 5. `inputValue()`

## Simple Understanding

> `inputValue()` tells us the **current value of the dropdown**.

Example:

``` ts
await country.selectOption({ label: "India" });

const value = await country.inputValue();

console.log(value);
```

Result:

``` text
in
```

If:

``` html
<option value="in">India</option>
```

then:

``` ts
await country.inputValue()
```

returns:

``` text
"in"
```

not:

``` text
"India"
```

## Memory

> `inputValue()` → HTML/current value

------------------------------------------------------------------------

# 6. `toHaveValue()`

## Simple Understanding

> `toHaveValue()` checks whether a **Locator currently has the expected
> value**.

Example:

``` ts
await country.selectOption({ label: "India" });

await expect(country).toHaveValue("in");
```

## Very Important

`toHaveValue()` works with the **Locator**.

Correct:

``` ts
await expect(country).toHaveValue("in");
```

Incorrect:

``` ts
const value = await country.inputValue();

expect(value).toHaveValue("in"); // Wrong
```

Why?

Because `value` is a JavaScript string, not a Locator.

## Memory

``` text
Locator → toHaveValue()
String  → toBe()
```

------------------------------------------------------------------------

# 7. `option:checked`

## Simple Understanding

> `option:checked` finds the `<option>` that is currently selected.

Example:

``` ts
const selectedOption =
    country.locator("option:checked");
```

This gives us a **Locator** for the selected option.

We can then read its visible text:

``` ts
const selectedText =
    await country.locator("option:checked").innerText();
```

If India is selected:

``` text
India
```

------------------------------------------------------------------------

# 8. Get Selected Option Text

## Simple Understanding

> If I want to know **which option is currently selected**, use
> `option:checked` and then `innerText()`.

``` ts
const selectedText =
    await country.locator("option:checked").innerText();

expect(selectedText).toBe("India");
```

## Flow

``` text
country
   ↓
option:checked
   ↓
innerText()
   ↓
"India"
```

------------------------------------------------------------------------

# 9. Get Selected Option's Value

## Simple Understanding

> If I want the HTML `value` of the selected option, use
> `getAttribute("value")`.

``` ts
const selectedValue =
    await country
        .locator("option:checked")
        .getAttribute("value");
```

For:

``` html
<option value="in">India</option>
```

the result is:

``` text
"in"
```

## Important Difference

``` text
innerText()           → India
getAttribute("value") → in
```

------------------------------------------------------------------------

# 10. `allInnerTexts()`

## Simple Understanding

> `allInnerTexts()` gets the visible text of **all matching elements**
> and returns them as an array.

Example:

``` ts
const options =
    await country.getByRole("option").allInnerTexts();
```

Result:

``` ts
[
    "-- Select --",
    "India",
    "United States",
    "Australia"
]
```

## Useful For

-   Checking available options
-   Checking a particular option exists
-   Checking multiple options
-   Comparing the complete option list

------------------------------------------------------------------------

# 11. `count()`

## Simple Understanding

> `count()` tells us **how many matching elements exist**.

Example:

``` ts
const totalOptions =
    await country.getByRole("option").count();

expect(totalOptions).toBe(4);
```

## Memory

``` text
count()         → How many?
allInnerTexts() → What text?
all()           → Give me the Locators
```

------------------------------------------------------------------------

# 12. `all()`

## Simple Understanding

> `all()` gives us all matching elements as an array of **Locator
> objects**.

Example:

``` ts
const options: Locator[] =
    await country.getByRole("option").all();
```

Now we can work with each option:

``` ts
for (const option of options) {
    console.log(await option.innerText());
}
```

## When Useful?

Use `all()` when you need to **work with each option individually**,
such as looping through options and dynamically reading attributes.

------------------------------------------------------------------------

# 13. `toContain()`

## Simple Understanding

> `toContain()` checks whether an array contains a particular item.

Example:

``` ts
const options =
    await country.getByRole("option").allInnerTexts();

expect(options).toContain("India");
```

## Negative Check

``` ts
expect(options).not.toContain("Germany");
```

Meaning:

> Germany should NOT be available.

------------------------------------------------------------------------

# 14. `toEqual()` --- Exact List

## Simple Understanding

> `toEqual()` checks whether two arrays contain exactly the expected
> contents **in the same order**.

``` ts
const actualOptions =
    await country.getByRole("option").allInnerTexts();

const expectedOptions = [
    "-- Select --",
    "India",
    "United States",
    "Australia"
];

expect(actualOptions).toEqual(expectedOptions);
```

## Memory

``` text
toEqual()
→ Everything must match
→ Content + order
```

------------------------------------------------------------------------

# 15. Exact List When Order Does Not Matter

Sometimes options can appear in a different order.

Example:

``` ts
const actualOptions = [
    "India",
    "Australia",
    "United States"
];

const expectedOptions = [
    "United States",
    "India",
    "Australia"
];
```

We can sort both:

``` ts
expect(actualOptions.sort())
    .toEqual(expectedOptions.sort());
```

## Simple Understanding

> Sort both lists first, then compare them.

This verifies:

``` text
Same options
Different order is allowed
No extra/missing options
```

------------------------------------------------------------------------

# 16. `expect.arrayContaining()`

## Simple Understanding

> `arrayContaining()` means: **"I only care that these required items
> are present."**

Example:

``` ts
expect(actualOptions).toEqual(
    expect.arrayContaining([
        "India",
        "United States",
        "Australia"
    ])
);
```

If actual is:

``` ts
[
    "India",
    "United States",
    "Australia",
    "Canada",
    "Singapore"
]
```

the assertion passes.

## Important

`arrayContaining()` **allows extra items**.

Therefore, it is NOT suitable when you want:

> "The dropdown must contain exactly these options."

For exact validation use:

``` ts
expect(actualOptions).toEqual(expectedOptions);
```

Or when order does not matter:

``` ts
expect(actualOptions.sort())
    .toEqual(expectedOptions.sort());
```

------------------------------------------------------------------------

# 17. `getAttribute()`

## Simple Understanding

> `getAttribute()` allows us to read an HTML attribute from an element.

Example:

``` ts
const value =
    await option.getAttribute("value");
```

For:

``` html
<option value="in">India</option>
```

we get:

``` text
"in"
```

## Important TypeScript Point

`getAttribute()` returns:

``` ts
string | null
```

So you may see:

``` ts
value!
```

Example:

``` ts
await country.selectOption({
    value: value!
});
```

The `!` tells TypeScript:

> "I know this value is not null here."

It does not change the runtime value.

### Safer Approach

``` ts
if (value !== null) {
    await country.selectOption({ value });
}
```

------------------------------------------------------------------------

# 18. Placeholder Validation

Example:

``` html
<option value="">-- Select --</option>
```

We can verify the selected placeholder text:

``` ts
const selectedText =
    await country.locator("option:checked").innerText();

expect(selectedText).toBe("-- Select --");
```

And its value:

``` ts
const selectedValue =
    await country
        .locator("option:checked")
        .getAttribute("value");

expect(selectedValue).toBe("");
```

## Simple Understanding

Check both:

``` text
Visible text → -- Select --
Value         → empty
```

------------------------------------------------------------------------

# 19. Verify Previous Selection Is Replaced

For a **single-select** dropdown, selecting another option replaces the
previous selection.

``` ts
await country.selectOption({ label: "India" });
await expect(country).toHaveValue("in");

await country.selectOption({ label: "Australia" });
await expect(country).toHaveValue("au");
```

You can also verify the selected text:

``` ts
expect(
    await country.locator("option:checked").innerText()
).toBe("Australia");
```

## Important

``` text
India selected
     ↓
Australia selected
     ↓
India is no longer selected
```

------------------------------------------------------------------------

# 20. Dynamic Dropdown Values

Suppose the application generates values dynamically:

``` html
<option value="country_83921">India</option>
<option value="country_92741">United States</option>
```

## Simple Understanding

Do not hardcode a dynamic value.

Instead, select using the stable visible label:

``` ts
await country.selectOption({ label: "India" });
```

Or dynamically read the value:

``` ts
const value = await option.getAttribute("value");

if (value !== null) {
    await country.selectOption({ value });
}
```

## Interview Takeaway

> When the option value is dynamic but the visible text is stable,
> selecting by `label` is usually more maintainable.

------------------------------------------------------------------------

# 21. Duplicate Visible Text

Suppose:

``` html
<option value="in">India</option>
<option value="in_old">India</option>
```

Selecting by:

``` ts
await country.selectOption({ label: "India" });
```

can be ambiguous because the visible text is duplicated.

If the value uniquely identifies the required option:

``` ts
await country.selectOption({ value: "in_old" });
```

## Interview Thinking

> When labels are duplicated, look for a unique `value` or another
> reliable attribute.

------------------------------------------------------------------------

# 22. Complete Native Dropdown Validation Pattern

A practical test can combine the important techniques:

``` ts
const country = page.getByRole("combobox", {
    name: "Country"
});

const options =
    await country.getByRole("option").allInnerTexts();

expect(options).toContain("India");

await country.selectOption({ label: "India" });

await expect(country).toHaveValue("in");

const selectedText =
    await country.locator("option:checked").innerText();

expect(selectedText).toBe("India");
```

## Mental Picture

``` text
              DROPDOWN
                  |
        +---------+---------+
        |                   |
     Validate             Select
        |                   |
   options/text       selectOption()
        |                   |
   count/toContain          |
   toEqual                  |
        |                   |
        +---------+---------+
                  |
              Verify
                  |
        toHaveValue()
        option:checked
        innerText()
```

------------------------------------------------------------------------

# 23. Most Important Methods --- Quick Memory Table

  Method / Matcher      Simple Meaning
  --------------------- ---------------------------------------------
  `selectOption()`      Select an option
  `inputValue()`        Get current dropdown value
  `option:checked`      Find currently selected option
  `innerText()`         Get visible text
  `getAttribute()`      Get HTML attribute
  `allInnerTexts()`     Get visible text of all matching elements
  `count()`             Count matching elements
  `all()`               Get all matching Locators
  `toHaveValue()`       Verify Locator's value
  `toBe()`              Verify an actual value/string
  `toContain()`         Verify item exists in array
  `toEqual()`           Verify exact equality
  `arrayContaining()`   Verify required items exist, extras allowed
  `.not`                Negative assertion

------------------------------------------------------------------------

# 24. Most Important Differences

## `inputValue()` vs `innerText()`

``` text
inputValue()
→ HTML/current value
→ "in"

innerText()
→ visible text
→ "India"
```

------------------------------------------------------------------------

## `getAttribute("value")` vs `inputValue()`

For a native select:

``` text
getAttribute("value")
→ value attribute of the selected <option>

inputValue()
→ current value of the <select>
```

Example:

``` html
<select>
    <option value="in">India</option>
</select>
```

Both can give:

``` text
"in"
```

But they operate on different elements/concepts.

------------------------------------------------------------------------

## `toHaveValue()` vs `toBe()`

``` ts
await expect(country).toHaveValue("in");
```

Use when `country` is a **Locator**.

``` ts
expect(selectedText).toBe("India");
```

Use when `selectedText` is a **JavaScript value/string**.

------------------------------------------------------------------------

## `toContain()` vs `arrayContaining()`

``` ts
expect(options).toContain("India");
```

Means:

> India exists.

``` ts
expect(options).toEqual(
    expect.arrayContaining(["India", "Australia"])
);
```

Means:

> India and Australia exist somewhere in the array.

------------------------------------------------------------------------

## `toEqual()` vs `arrayContaining()`

``` text
toEqual()
→ exact comparison

arrayContaining()
→ required items must exist
→ extra items allowed
```

------------------------------------------------------------------------

# 25. Reusable Dropdown Utility

A simple framework-style method:

``` ts
async function selectDropDown(
    dropDown: Locator,
    optionLabel: string
): Promise<void> {

    await dropDown.selectOption({
        label: optionLabel
    });

    const selectedText =
        await dropDown.locator("option:checked").innerText();

    expect(selectedText).toEqual(optionLabel);
}
```

Usage:

``` ts
await selectDropDown(country, "India");
await selectDropDown(country, "Australia");
```

## Why This Matters

Instead of repeating:

``` text
select
verify
select
verify
select
verify
```

in every test, create one reusable method.

This is the beginning of **framework thinking**.

------------------------------------------------------------------------

# 26. Interview Questions You Should Know

## Basic

1.  How do you handle a native dropdown in Playwright?
2.  What is `selectOption()`?
3.  How do you select by value?
4.  How do you select by visible text?
5.  How do you select by index?
6.  Is dropdown index zero-based?
7.  What does `selectOption()` return?

## Validation

8.  How do you get the selected value?
9.  How do you get selected option text?
10. How do you verify the selected value?
11. What is `option:checked`?
12. Is `option:checked` a CSS selector?
13. How do you get all dropdown options?
14. How do you count dropdown options?
15. How do you verify an option does not exist?

## Practical

16. What if the option value is dynamic?
17. What if visible labels are duplicated?
18. How do you verify the complete option list?
19. How do you verify the list when order does not matter?
20. How do you loop through all options?
21. How do you skip the placeholder?
22. How would you create a reusable dropdown method?
23. How would you debug a failing dropdown assertion?
24. Difference between `inputValue()` and `innerText()`?
25. Difference between `toHaveValue()` and `toBe()`?

------------------------------------------------------------------------

# 27. Practice Assignments

These assignments progress from basic selection to framework-style
scenarios.

## Level 1 --- Basic Selection

### Assignment 1 --- Select by Value

Select `India` using its HTML value and verify the selection.

### Assignment 2 --- Select by Label

Select `India` using its visible text and verify the selected text.

### Assignment 3 --- Select by Index

Select an option using its index and verify both its visible text and
value.

### Assignment 4 --- Capture Return Value

Use `selectOption()` and capture its returned value. Verify the returned
array.

### Assignment 5 --- Capture Current Value

Select an option and capture its current value using `inputValue()`.

### Assignment 6 --- Verify Using `toHaveValue()`

Select an option and verify its value using `toHaveValue()`.

### Assignment 7 --- Capture Selected Option

Use `option:checked` to locate the currently selected option.

### Assignment 8 --- Capture Selected Text

Use `option:checked` and `innerText()` to capture the selected option's
visible text.

### Assignment 9 --- Verify Default Selection

Verify the default selected option when the page loads.

### Assignment 10 --- Verify Selection Replacement

Select India, then Australia. Verify that Australia is selected and
India is no longer selected.

------------------------------------------------------------------------

## Level 2 --- Option Validation

### Assignment 11 --- Count Options

Count all `<option>` elements and verify the expected count.

### Assignment 12 --- Get All Option Texts

Capture all option texts using `allInnerTexts()`.

### Assignment 13 --- Exact List --- Order Matters

Compare actual and expected option arrays using `toEqual()`.

### Assignment 14 --- Exact List --- Order Does Not Matter

Compare the actual and expected arrays after sorting both.

### Assignment 15 --- Verify Option Exists

Verify that `India` exists using `toContain()`.

### Assignment 16 --- Verify Option Does Not Exist

Verify that `Germany` does not exist.

### Assignment 17 --- Verify Multiple Required Options

Use `expect.arrayContaining()` to verify multiple required options.

### Assignment 18 --- Verify Only Expected Options Exist

Verify that the dropdown contains exactly the expected options and no
extra options.

### Assignment 19 --- Verify Placeholder

Verify both the placeholder's visible text and its empty value.

### Assignment 20 --- Verify Selection Changes Value

Select two different options and verify that the dropdown value changes
correctly.

------------------------------------------------------------------------

## Level 3 --- Real Project Scenarios

### Assignment 21 --- Dynamic Value, Stable Label

The option values are dynamically generated. Select India without
hardcoding its value.

### Assignment 22 --- Changing Label, Stable Value

The visible label changes but the value remains stable. Select the
option using its stable value.

### Assignment 23 --- Duplicate Visible Text

Two options have the same visible text but different values. Select the
correct one using its unique value.

### Assignment 24 --- Dynamically Find and Select an Option

Get all options, find Australia by visible text, dynamically read its
value, select using that value, and verify Australia.

### Assignment 25 --- Loop Through Every Option

Loop through every option, skip the placeholder, dynamically get each
option's value, select it, and verify the selected value.

### Assignment 26 --- Verify Every Option Can Be Selected

Loop through every real option, dynamically get its visible text, select
it using `label`, and verify the selected text.

### Assignment 27 --- Dynamically Skip Placeholder

Identify the placeholder using its empty `value`, skip it, and select
every remaining option.

### Assignment 28 --- Verify Selection After Another Action

Select India, verify it, select Australia, verify it, and verify that
India is no longer selected. Use `option:checked`.

### Assignment 29 --- Create a Reusable Dropdown Method

Create:

``` ts
selectDropDown(dropDown, optionLabel)
```

The method should select the option and verify that it is selected.

### Assignment 30 --- Debug a Failing Dropdown Test

Given a failing test where `toHaveValue()` is incorrectly used on a
JavaScript string, identify the problem and correct the assertion.

------------------------------------------------------------------------

# 28. Final Mental Map

``` text
NATIVE <select>
      |
      +---- SELECT
      |       |
      |       +-- selectOption({value})
      |       +-- selectOption({label})
      |       +-- selectOption({index})
      |
      +---- READ
      |       |
      |       +-- inputValue()       → current value
      |       +-- option:checked     → selected option
      |       +-- innerText()        → visible text
      |       +-- getAttribute()     → HTML attribute
      |
      +---- GET OPTIONS
      |       |
      |       +-- count()            → how many
      |       +-- all()              → Locators
      |       +-- allInnerTexts()    → texts
      |
      +---- ASSERT
              |
              +-- toHaveValue()     → Locator value
              +-- toBe()            → JS value
              +-- toContain()       → item exists
              +-- toEqual()         → exact match
              +-- arrayContaining() → required items exist
```

------------------------------------------------------------------------

# 29. Six Things to Remember

``` text
1. Native <select>
   → selectOption()

2. Select by visible text
   → { label: "India" }

3. Select by HTML value
   → { value: "in" }

4. Current dropdown value
   → inputValue()

5. Currently selected option
   → option:checked

6. Verify Locator's value
   → toHaveValue()
```

## Final Interview Memory

> **Identify → Locate → Select → Read → Assert**

``` text
Identify
  ↓
Is it native <select>?
  ↓
Locate
  ↓
Get the dropdown Locator
  ↓
Select
  ↓
selectOption()
  ↓
Read
  ↓
inputValue() / option:checked / innerText()
  ↓
Assert
  ↓
toHaveValue() / toBe() / toContain() / toEqual()
```

------------------------------------------------------------------------

# 30. Important Rule About Await

Most Playwright operations used above are asynchronous.

Use:

``` ts
await country.selectOption({ label: "India" });

const value = await country.inputValue();

const text = await country.locator("option:checked").innerText();
```

But `expect()` itself is not normally awaited when comparing an already
captured JavaScript value:

``` ts
expect(text).toBe("India");
```

For async Playwright assertions such as:

``` ts
await expect(country).toHaveValue("in");
```

use `await`.

## Memory

``` text
Playwright async operation → await

Captured JS value + expect.toBe()
→ no await

Playwright async matcher
→ await
```

------------------------------------------------------------------------

# Native Select Dropdown Mastery --- Completion Checklist

Before moving to multi-select dropdowns, you should be comfortable with:

-   [ ] Identify native vs custom dropdown
-   [ ] Locate a native dropdown
-   [ ] Select by value
-   [ ] Select by label
-   [ ] Select by index
-   [ ] Understand zero-based index
-   [ ] Understand `selectOption()` return value
-   [ ] Use `inputValue()`
-   [ ] Use `option:checked`
-   [ ] Use `innerText()`
-   [ ] Use `getAttribute("value")`
-   [ ] Use `allInnerTexts()`
-   [ ] Use `count()`
-   [ ] Use `all()`
-   [ ] Use `toHaveValue()`
-   [ ] Use `toBe()`
-   [ ] Use `toContain()`
-   [ ] Use `toEqual()`
-   [ ] Use `arrayContaining()`
-   [ ] Perform negative assertions
-   [ ] Validate placeholders
-   [ ] Handle dynamic values
-   [ ] Handle duplicate labels
-   [ ] Loop through options
-   [ ] Dynamically skip placeholders
-   [ ] Create reusable dropdown methods
-   [ ] Debug dropdown assertion failures

**Next topic:** Multi-Select Native Dropdown Mastery.
