# JavaScript Array Operations Cheatsheet

## How to Read This Guide

Every example in this guide shows the array **before**, the operation, and the result **after**. The key distinction to always keep in mind:

- **Mutating methods** modify the original array in place.
- **Non-mutating methods** return a new array and leave the original untouched.

```javascript
// Non-mutating: original is unchanged
const numbers = [1, 2, 3];
const doubled = numbers.map(x => x * 2);

console.log(numbers);  // [1, 2, 3] — original unchanged
console.log(doubled);  // [2, 4, 6] — new array

// Mutating: original is modified
const numbers2 = [1, 2, 3];
numbers2.push(4);

console.log(numbers2);  // [1, 2, 3, 4] — original modified
```

When in doubt, check the method's entry in this guide — every method is labeled **mutating** or **non-mutating**.

---

## Core Methods (the ones you'll use constantly)

### `map` — Transform each element (non-mutating)

Returns a new array where each element is the result of calling the callback on the original element.

```javascript
const users = [
  { name: 'Alice', age: 30 },
  { name: 'Bob', age: 25 },
  { name: 'Charlie', age: 35 }
];

const names = users.map(user => user.name);
console.log(names);  // ['Alice', 'Bob', 'Charlie']

const ages = users.map(user => user.age);
console.log(ages);  // [30, 25, 35]
```

**Use when:** You want to transform each element into something else. Same length as original.

---

### `filter` — Keep only elements that match (non-mutating)

Returns a new array containing only elements where the callback returns `true`.

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

const evens = numbers.filter(x => x % 2 === 0);
console.log(evens);  // [2, 4, 6]

const users = [
  { name: 'Alice', age: 30 },
  { name: 'Bob', age: 17 },
  { name: 'Charlie', age: 25 }
];

const adults = users.filter(user => user.age >= 18);
console.log(adults);  // [{ name: 'Alice', age: 30 }, { name: 'Charlie', age: 25 }]
```

**Use when:** You want a subset of the original array. Length may be shorter.

---

### `reduce` — Combine all elements into a single value (non-mutating)

Returns a single accumulated value.

```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum);  // 15

// Product
const product = numbers.reduce((acc, curr) => acc * curr, 1);
console.log(product);  // 120

// Find max
const max = numbers.reduce((acc, curr) => Math.max(acc, curr));
console.log(max);  // 5
```

**The callback takes two required arguments:**
- `acc` (accumulator) — the running total
- `curr` (current element) — the current item being processed

**The second argument to `reduce` is the initial value of the accumulator.**

```javascript
// Without initial value: first element becomes the accumulator
const sumNoInit = numbers.reduce((acc, curr) => acc + curr);
console.log(sumNoInit);  // 15 — same result here, but risky with empty arrays

// With initial value: always safe
const sumWithInit = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sumWithInit);  // 15
```

**Use when:** You want a single value from an array (sum, average, max, object, etc.).

---

### `find` — Get first matching element (non-mutating)

Returns the first element where the callback returns `true`, or `undefined` if none match.

```javascript
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
];

const user = users.find(u => u.id === 2);
console.log(user);  // { id: 2, name: 'Bob' }

const missing = users.find(u => u.id === 99);
console.log(missing);  // undefined
```

**Use when:** You want one specific element.

---

### `findIndex` — Get index of first matching element (non-mutating)

Returns the index of the first matching element, or `-1` if none match.

```javascript
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
];

const index = users.findIndex(u => u.id === 2);
console.log(index);  // 1

const missingIndex = users.findIndex(u => u.id === 99);
console.log(missingIndex);  // -1
```

**Use when:** You need the position of an element, not the element itself.

---

### `some` — Check if ANY element matches (non-mutating)

Returns `true` if at least one element matches, `false` otherwise.

```javascript
const numbers = [1, 2, 3, 4, 5];

const hasNegative = numbers.some(x => x < 0);
console.log(hasNegative);  // false

const hasEven = numbers.some(x => x % 2 === 0);
console.log(hasEven);  // true
```

**Use when:** You need a yes/no answer about whether anything matches.

---

### `every` — Check if ALL elements match (non-mutating)

Returns `true` if every element matches, `false` otherwise.

```javascript
const numbers = [2, 4, 6, 8];

const allEven = numbers.every(x => x % 2 === 0);
console.log(allEven);  // true

const allPositive = numbers.every(x => x > 0);
console.log(allPositive);  // true

const mixed = [2, 4, 5, 8];
const allEvenMixed = mixed.every(x => x % 2 === 0);
console.log(allEvenMixed);  // false
```

**Use when:** You need to verify every element satisfies a condition.

---

### `includes` — Check if a value exists (non-mutating)

Returns `true` if the array contains the exact value, `false` otherwise. Uses strict equality (`===`).

```javascript
const fruits = ['apple', 'banana', 'cherry'];

console.log(fruits.includes('banana'));  // true
console.log(fruits.includes('grape'));   // false

// Works with numbers
const numbers = [1, 2, 3];
console.log(numbers.includes(2));  // true
console.log(numbers.includes('2'));  // false — strict equality
```

**Use when:** You want to check if a primitive value exists. Not for objects (use `find` or `some` instead).

---

### `forEach` — Do something for each element (non-mutating, but no return value)

Executes the callback for each element. Does NOT return a new array.

```javascript
const users = ['Alice', 'Bob', 'Charlie'];

users.forEach(name => {
  console.log(`Hello, ${name}`);
});
// "Hello, Alice"
// "Hello, Bob"
// "Hello, Charlie"
```

**Use when:** You want side effects (logging, DOM updates, etc.), not a return value.

**Common mistake:** Trying to `return` from `forEach`:

```javascript
// ❌ This doesn't work — forEach always returns undefined
const names = ['Alice', 'Bob', 'Charlie'];
const upperNames = names.forEach(name => name.toUpperCase());
console.log(upperNames);  // undefined

// ✅ Use map instead
const upperNamesFixed = names.map(name => name.toUpperCase());
console.log(upperNamesFixed);  // ['ALICE', 'BOB', 'CHARLIE']
```

---

## Adding / Removing Elements

### `push` — Add to end (mutating)

```javascript
const fruits = ['apple', 'banana'];
fruits.push('cherry');
console.log(fruits);  // ['apple', 'banana', 'cherry']

// Can push multiple
fruits.push('date', 'elderberry');
console.log(fruits);  // ['apple', 'banana', 'cherry', 'date', 'elderberry']
```

**Returns:** The new length of the array (not the array itself).

---

### `pop` — Remove from end (mutating)

```javascript
const fruits = ['apple', 'banana', 'cherry'];
const removed = fruits.pop();
console.log(removed);  // 'cherry'
console.log(fruits);   // ['apple', 'banana']
```

**Returns:** The removed element.

---

### `unshift` — Add to beginning (mutating)

```javascript
const fruits = ['banana', 'cherry'];
fruits.unshift('apple');
console.log(fruits);  // ['apple', 'banana', 'cherry']
```

**Returns:** The new length of the array.

---

### `shift` — Remove from beginning (mutating)

```javascript
const fruits = ['apple', 'banana', 'cherry'];
const removed = fruits.shift();
console.log(removed);  // 'apple'
console.log(fruits);   // ['banana', 'cherry']
```

**Returns:** The removed element.

---

### `splice` — Add/remove at any position (mutating)

The Swiss Army knife of array modification. Three arguments: start index, number of elements to remove, elements to add.

```javascript
const fruits = ['apple', 'banana', 'cherry', 'date'];

// Remove 1 element at index 2
fruits.splice(2, 1);
console.log(fruits);  // ['apple', 'banana', 'date']

// Remove 2 elements at index 1, insert 'kiwi' and 'mango'
const fruits2 = ['apple', 'banana', 'cherry', 'date'];
fruits2.splice(1, 2, 'kiwi', 'mango');
console.log(fruits2);  // ['apple', 'kiwi', 'mango', 'date']

// Insert without removing (0 elements removed)
const fruits3 = ['apple', 'banana'];
fruits3.splice(1, 0, 'kiwi');
console.log(fruits3);  // ['apple', 'kiwi', 'banana']
```

**Returns:** An array of removed elements (empty array if none removed).

---

### `slice` — Extract a portion (non-mutating)

Takes a start index (inclusive) and end index (exclusive). Returns a new array.

```javascript
const numbers = [1, 2, 3, 4, 5];

const firstThree = numbers.slice(0, 3);
console.log(firstThree);  // [1, 2, 3]

const fromIndex = numbers.slice(2);
console.log(fromIndex);  // [3, 4, 5]

const lastTwo = numbers.slice(-2);
console.log(lastTwo);  // [4, 5] — negative index counts from end

console.log(numbers);  // [1, 2, 3, 4, 5] — original unchanged
```

---

### `concat` — Combine arrays (non-mutating)

Returns a new array combining two or more arrays.

```javascript
const fruits = ['apple', 'banana'];
const vegetables = ['carrot', 'spinach'];

const combined = fruits.concat(vegetables);
console.log(combined);  // ['apple', 'banana', 'carrot', 'spinach']

console.log(fruits);      // ['apple', 'banana'] — unchanged
console.log(vegetables);  // ['carrot', 'spinach'] — unchanged
```

**Note:** In modern JavaScript, the spread operator (`...`) is often used instead:

```javascript
const combined = [...fruits, ...vegetables];
console.log(combined);  // ['apple', 'banana', 'carrot', 'spinach']
```

Both are non-mutating. Use whichever reads more clearly to you.

---

## Searching and Sorting

### `indexOf` — Find index of a value (non-mutating)

Returns the first index of the exact value, or `-1` if not found. Uses strict equality.

```javascript
const fruits = ['apple', 'banana', 'cherry', 'banana'];

console.log(fruits.indexOf('banana'));  // 1 — first occurrence
console.log(fruits.indexOf('grape'));   // -1
console.log(fruits.indexOf('Banana'));  // -1 — case-sensitive
```

---

### `lastIndexOf` — Find last index of a value (non-mutating)

Same as `indexOf`, but searches from the end.

```javascript
const fruits = ['apple', 'banana', 'cherry', 'banana'];

console.log(fruits.lastIndexOf('banana'));  // 3 — last occurrence
```

---

### `sort` — Sort elements (mutating)

Sorts the array **in place** and returns it. By default, sorts as **strings**.

```javascript
// Default sort: lexicographic (alphabetical for strings, but problematic for numbers)
const fruits = ['cherry', 'apple', 'banana'];
fruits.sort();
console.log(fruits);  // ['apple', 'banana', 'cherry'] — works for strings

// ❌ Gotcha: numbers sorted as strings
const numbers = [10, 2, 1, 20];
numbers.sort();
console.log(numbers);  // [1, 10, 2, 20] — WRONG for numeric sorting

// ✅ Numeric sort: provide a comparator
const numbersFixed = [10, 2, 1, 20];
numbersFixed.sort((a, b) => a - b);
console.log(numbersFixed);  // [1, 2, 10, 20] — ascending

// Descending
const numbersDesc = [10, 2, 1, 20];
numbersDesc.sort((a, b) => b - a);
console.log(numbersDesc);  // [20, 10, 2, 1] — descending
```

**The comparator function:**

- Return negative → `a` comes before `b`
- Return positive → `a` comes after `b`
- Return 0 → unchanged order

**Gotcha:** `sort` mutates the original array. If you need the original, copy it first:

```javascript
const numbers = [3, 1, 2];
const sorted = [...numbers].sort((a, b) => a - b);
console.log(sorted);  // [1, 2, 3]
console.log(numbers);  // [3, 1, 2] — unchanged
```

---

### `reverse` — Reverse order (mutating)

Reverses the array in place.

```javascript
const numbers = [1, 2, 3, 4, 5];
numbers.reverse();
console.log(numbers);  // [5, 4, 3, 2, 1]
```

---

## Flattening and Joining

### `flat` — Flatten nested arrays (non-mutating)

Returns a new array with sub-arrays flattened to the specified depth.

```javascript
const nested = [1, [2, 3], [4, [5, 6]]];

const flatOne = nested.flat();
console.log(flatOne);  // [1, 2, 3, 4, [5, 6]] — one level deep

const flatTwo = nested.flat(2);
console.log(flatTwo);  // [1, 2, 3, 4, 5, 6] — two levels deep
```

---

### `flatMap` — Map then flatten (non-mutating)

Combines `map` and `flat(1)`.

```javascript
const users = [
  { name: 'Alice', hobbies: ['reading', 'hiking'] },
  { name: 'Bob', hobbies: ['cooking'] }
];

const allHobbies = users.flatMap(user => user.hobbies);
console.log(allHobbies);  // ['reading', 'hiking', 'cooking']

// Equivalent to:
const sameResult = users.map(user => user.hobbies).flat();
console.log(sameResult);  // ['reading', 'hiking', 'cooking']
```

---

### `join` — Convert to string (non-mutating)

Joins all elements into a string with a separator.

```javascript
const fruits = ['apple', 'banana', 'cherry'];

const commaSep = fruits.join(', ');
console.log(commaSep);  // "apple, banana, cherry"

const dashSep = fruits.join('-');
console.log(dashSep);  // "apple-banana-cherry"

const noSep = fruits.join('');
console.log(noSep);  // "applebananacherry"
```

---

## Obfuscated Patterns (and how to read them)

JavaScript developers love chaining array methods into dense one-liners. They're common in codebases, tutorials, and Stack Overflow answers. Here's how to unpack them.

### The pattern

When you see multiple array methods chained together:

```javascript
const result = items
  .filter(item => item.active)
  .map(item => item.value)
  .reduce((sum, val) => sum + val, 0);
```

Read it **top to bottom, left to right**:

1. Start with `items`
2. Keep only active items
3. Extract their values
4. Sum them

Each method passes its result to the next method in the chain.

---

### Step-by-step breakdown

```javascript
const items = [
  { active: true, value: 10 },
  { active: false, value: 20 },
  { active: true, value: 30 }
];

// The chain
const result = items
  .filter(item => item.active)   // [{ active: true, value: 10 }, { active: true, value: 30 }]
  .map(item => item.value)       // [10, 30]
  .reduce((sum, val) => sum + val, 0);  // 40

console.log(result);  // 40
```

---

### Common patterns you'll see

**Pattern 1: Filter + Map**

```javascript
// Get names of all active users
const activeNames = users
  .filter(user => user.active)
  .map(user => user.name);
```

**Pattern 2: Map + Filter**

```javascript
// Get all non-null values after transformation
const validEmails = users
  .map(user => user.email)
  .filter(email => email !== null);
```

**Pattern 3: Filter + Map + Reduce**

```javascript
// Sum of prices for items in cart that are in stock
const total = cart
  .filter(item => item.inStock)
  .map(item => item.price)
  .reduce((sum, price) => sum + price, 0);
```

**Pattern 4: Sort + Map**

```javascript
// Get names sorted alphabetically
const sortedNames = users
  .sort((a, b) => a.name.localeCompare(b.name))
  .map(user => user.name);
```

**Pattern 5: Map + flatMap + filter**

```javascript
// Get all unique tags from posts that are published
const tags = posts
  .filter(post => post.published)
  .flatMap(post => post.tags)
  .filter((tag, index, arr) => arr.indexOf(tag) === index);  // dedupe
```

---

### How to read any chain

1. **Find the starting array** (what comes before the first dot).
2. **Read each method in order**, asking "what does this return?"
3. **Trace the data transformation** — what shape is the data after each step?

Example:

```javascript
const result = orders
  .filter(o => o.status === 'pending')    // array of pending orders
  .flatMap(o => o.items)                  // array of all items from those orders
  .filter(item => item.price > 100)       // array of expensive items
  .map(item => item.name)                 // array of item names
  .sort();                                // sorted alphabetically
```

Step by step:
- `orders` → array of order objects
- After `filter` → array of pending orders
- After `flatMap` → array of items (flattened from all orders)
- After `filter` → array of items with price > 100
- After `map` → array of item names (strings)
- After `sort` → array of sorted names

---

### The problem with dense chains

While compact, heavily chained methods can be hard to debug:

- **No intermediate variables** to inspect
- **Error messages** point to the chain, not which step failed
- **Readability suffers** when a single line does too much

### When to break them apart

If a chain is hard to read, break it into steps:

```javascript
// Dense (harder to read)
const result = orders.filter(o => o.status === 'pending').flatMap(o => o.items).filter(item => item.price > 100).map(item => item.name).sort();

// Better: break into logical steps
const pendingOrders = orders.filter(o => o.status === 'pending');
const pendingItems = pendingOrders.flatMap(o => o.items);
const expensiveItems = pendingItems.filter(item => item.price > 100);
const itemNames = expensiveItems.map(item => item.name);
const sortedNames = itemNames.sort();
```

The broken-out version is longer but easier to debug and understand. There's no performance difference.

---

### Chaining vs. loops

A chain is just a more declarative way to write what you'd otherwise do with a loop:

```javascript
// Chained
const total = items
  .filter(item => item.active)
  .map(item => item.value)
  .reduce((sum, val) => sum + val, 0);

// Equivalent loop
let total = 0;
for (const item of items) {
  if (item.active) {
    total += item.value;
  }
}
```

Both do the same thing. The chain expresses intent more clearly; the loop gives you more control. Use whichever is more readable for the situation.

---

## Quick Reference Card

| Method | What it does | Mutates? | Returns |
|--------|-------------|----------|---------|
| `map` | Transform each element | No | New array (same length) |
| `filter` | Keep matching elements | No | New array (possibly shorter) |
| `reduce` | Combine into single value | No | Single value |
| `find` | Get first matching element | No | Element or `undefined` |
| `findIndex` | Get index of first match | No | Index or `-1` |
| `some` | Check if any match | No | `true`/`false` |
| `every` | Check if all match | No | `true`/`false` |
| `includes` | Check if value exists | No | `true`/`false` |
| `forEach` | Execute for each element | No | `undefined` |
| `push` | Add to end | Yes | New length |
| `pop` | Remove from end | Yes | Removed element |
| `unshift` | Add to beginning | Yes | New length |
| `shift` | Remove from beginning | Yes | Removed element |
| `splice` | Add/remove at position | Yes | Removed elements |
| `slice` | Extract portion | No | New array |
| `concat` | Combine arrays | No | New array |
| `indexOf` | Find index of value | No | Index or `-1` |
| `lastIndexOf` | Find last index of value | No | Index or `-1` |
| `sort` | Sort elements | Yes | Original array (sorted) |
| `reverse` | Reverse order | Yes | Original array (reversed) |
| `flat` | Flatten nested arrays | No | New array |
| `flatMap` | Map then flatten | No | New array |
| `join` | Convert to string | No | String |

---

