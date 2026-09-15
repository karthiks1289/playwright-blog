# Playwright — Capturing Values, Text, Attributes & Multiple Elements

## 1. Capture Value Entered in a Text Field — `inputValue()`

### Simple Understanding

**`inputValue()` tells us what value is currently entered inside an input field.**

### Syntax

```ts
const value = await page.locator('selector').inputValue();
```

### Example

```ts
const username = await page.locator('#username').inputValue();

console.log(username);
```

### Remember

> **`inputValue()` → What value is inside this input?**

---

## 2. Capture Text of an Element — `textContent()`

### Simple Understanding

**`textContent()` gets the text content of an element, including the text inside its child elements.**

### Example

```html
<div>
    Hello
    <span>World</span>
</div>
```

```ts
const text = await page.locator('div').textContent();

console.log(text);
```

Result:

```text
Hello World
```

### Important Point

`textContent()` can return text that is **not visible on the page**, because it reads the element's text content from the DOM.

### Remember

> **`textContent()` → Give me the text content from the DOM.**

---

## 3. Capture Visible Text — `innerText()`

### Simple Understanding

**`innerText()` gets the visible, rendered text of an element, including visible text from its child elements.**

### Example

```html
<div>
    Hello
    <span>World</span>
</div>
```

```ts
const text = await page.locator('div').innerText();

console.log(text);
```

Result:

```text
Hello World
```

### Important Correction

`innerText()` does **not** mean that it ignores child elements.

If the child contains visible text, that text can also be returned.

### `textContent()` vs `innerText()`

| Method | Simple Meaning |
|---|---|
| `textContent()` | Text content in the DOM |
| `innerText()` | Visible rendered text |

### Remember

> **`textContent()` → DOM text**  
> **`innerText()` → Visible text**

---

# 4. Capture Multiple Matching Elements — `all()`

### Simple Understanding

**`all()` gives us all matching elements as an array of individual Playwright Locators.**

### Syntax

```ts
const elements = await page.locator('selector').all();
```

### Return Type

Before `await`:

```text
Promise<Locator[]>
```

After `await`:

```text
Locator[]
```

### Example

Suppose a page contains:

```html
<div class="product">Laptop</div>
<div class="product">Mouse</div>
<div class="product">Keyboard</div>
```

We can get all products:

```ts
const products = await page.locator('.product').all();

console.log(products.length);
```

Result:

```text
3
```

We can also work with each element:

```ts
const products = await page.locator('.product').all();

for (const product of products) {
    console.log(await product.innerText());
}
```

Output:

```text
Laptop
Mouse
Keyboard
```

### Remember

> **`all()` → Give me all matching elements.**

---

# 5. Count Matching Elements — `count()`

### Simple Understanding

**`count()` directly tells us how many elements match the locator.**

### Syntax

```ts
const count = await page.locator('selector').count();
```

### Example

```ts
const productCount = await page.locator('.product').count();

console.log(productCount);
```

Result:

```text
3
```

### `all()` vs `count()`

```ts
const products = await page.locator('.product').all();

console.log(products.length);
```

Here:

> `all()` → gives us the actual elements.

Whereas:

```ts
const count = await page.locator('.product').count();

console.log(count);
```

Here:

> `count()` → gives us only the number.

### Remember

> **`all()` → Elements**  
> **`count()` → Number of elements**

---

# 6. Capture an Attribute Value — `getAttribute()`

### Simple Understanding

**`getAttribute()` tells us the value stored in a specific HTML attribute.**

### Syntax

```ts
const value = await page.locator('selector').getAttribute('attributeName');
```

### Example

HTML:

```html
<input id="username" placeholder="Enter username">
```

Playwright:

```ts
const placeholder = await page
    .locator('#username')
    .getAttribute('placeholder');

console.log(placeholder);
```

Result:

```text
Enter username
```

### Other Examples

```ts
await page.locator('#username').getAttribute('id');

await page.locator('#username').getAttribute('placeholder');

await page.locator('a').getAttribute('href');

await page.locator('button').getAttribute('class');
```

### Remember

> **`getAttribute()` → Give me the value of this attribute.**

---

# 7. Capture Text from All Matching Elements

When we have multiple matching elements and want their text, we have two useful methods.

---

## 7.1 `allTextContents()`

### Simple Understanding

**`allTextContents()` gets the text content of all matching elements at once.**

### Syntax

```ts
const texts = await page.locator('selector').allTextContents();
```

### Return Type

```text
string[]
```

### Example

```ts
const products = await page.locator('.product').allTextContents();

console.log(products);
```

Result:

```text
["Laptop", "Mouse", "Keyboard"]
```

### Remember

> **`allTextContents()` → Text content of all matching elements.**

---

## 7.2 `allInnerTexts()`

### Simple Understanding

**`allInnerTexts()` gets the visible rendered text of all matching elements at once.**

### Syntax

```ts
const texts = await page.locator('selector').allInnerTexts();
```

### Return Type

```text
string[]
```

### Example

```ts
const products = await page.locator('.product').allInnerTexts();

console.log(products);
```

Result:

```text
["Laptop", "Mouse", "Keyboard"]
```

### Remember

> **`allInnerTexts()` → Visible text of all matching elements.**

---

# 8. Complete Comparison

| Method | What does it do? | Works with | Return |
|---|---|---|---|
| `inputValue()` | Gets value inside an input | One element | `string` |
| `textContent()` | Gets DOM text content | One element | `string \| null` |
| `innerText()` | Gets visible rendered text | One element | `string` |
| `getAttribute()` | Gets a specific attribute value | One element | `string \| null` |
| `all()` | Gets all matching elements | Multiple elements | `Locator[]` |
| `count()` | Gets number of matching elements | Multiple elements | `number` |
| `allTextContents()` | Gets text content of all matches | Multiple elements | `string[]` |
| `allInnerTexts()` | Gets visible text of all matches | Multiple elements | `string[]` |

---

# 9. 🧠 Mental Map

Think about what you want to **capture**:

```text
                     LOCATOR
                        │
        ┌───────────────┼────────────────┐
        │               │                │
       VALUE            TEXT           ATTRIBUTE
        │               │                │
 inputValue()     ┌──────┴──────┐   getAttribute()
                  │             │
            textContent()   innerText()
             DOM text       Visible text


                 MULTIPLE ELEMENTS
                        │
                 ┌──────┴──────┐
                 │             │
                all()        count()
                 │             │
            Locator[]        Number
                 │
          ┌──────┴──────┐
          │             │
 allTextContents()  allInnerTexts()
    DOM text         Visible text
```

---

# 10. ⭐ Easy Memory Trick

Remember these questions:

| If you ask... | Use |
|---|---|
| **What value is inside this input?** | `inputValue()` |
| **What text is in the DOM?** | `textContent()` |
| **What text can the user see?** | `innerText()` |
| **What is the value of this attribute?** | `getAttribute()` |
| **Give me all matching elements.** | `all()` |
| **How many matching elements are there?** | `count()` |
| **Give me text content from all matches.** | `allTextContents()` |
| **Give me visible text from all matches.** | `allInnerTexts()` |

## Final One-Liner

> **`inputValue` = input value | `textContent` = DOM text | `innerText` = visible text | `getAttribute` = attribute value | `all` = all elements | `count` = number of elements | `allTextContents` = all DOM texts | `allInnerTexts` = all visible texts**
