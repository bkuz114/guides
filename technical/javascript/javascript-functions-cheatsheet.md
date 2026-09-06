# JavaScript Functions Cheatsheet

## How to Read This Guide

A function is a reusable block of code. You **define** it once, then **call** it multiple times with different values.

```javascript
// DEFINE: Parameters (a, b) are placeholders
const add = (a, b) => a + b;

// CALL: Arguments (5, 10) are actual values
add(5, 10);   // returns 15
add(20, 30);  // returns 50
add(0, 5);    // returns 5
```

The parameters (`a`, `b` in the definition) don't exist until you call the function. When you call `add(5, 10)`, JavaScript assigns `a = 5` and `b = 10`, runs the function body, and returns the result.

Every example in this guide shows the definition and the usage together.

---

## Function Syntaxes (5 ways, all different)

### 1. Function Declaration

```javascript
// Define
function add(a, b) {
  return a + b;
}

// Use
const result = add(5, 10);
console.log(result);  // 15
```

**Key trait:** Hoisted — can be called before its definition in the code.

```javascript
// This works because function declarations are hoisted
console.log(add(5, 10));  // 15

function add(a, b) {
  return a + b;
}
```

**When to use:** Named, standalone functions. Good for larger, reusable logic.

---

### 2. Function Expression

```javascript
// Define
const add = function(a, b) {
  return a + b;
};

// Use
const result = add(5, 10);
console.log(result);  // 15
```

**Key trait:** NOT hoisted — cannot be called before definition.

```javascript
// This throws an error
console.log(add(5, 10));

const add = function(a, b) {
  return a + b;
};
// ReferenceError: Cannot access 'add' before initialization
```

**When to use:** Rarely used now — arrow functions are cleaner and more predictable with `this`.

---

### 3. Arrow Function (implicit return)

```javascript
// Define
const add = (a, b) => a + b;

// Use
const result = add(5, 10);
console.log(result);  // 15
```

**Key trait:** Single expression. No `return` keyword needed. The expression after `=>` is automatically returned.

**When to use:** One-liners. Most common in modern JavaScript.

---

### 4. Arrow Function (block body)

```javascript
// Define
const add = (a, b) => {
  return a + b;
};

// Use
const result = add(5, 10);
console.log(result);  // 15
```

**Key trait:** Multiple statements allowed. Must use explicit `return`.

**When to use:** When you need multiple lines of logic before returning.

---

### 5. Arrow Function (single param, no parens)

```javascript
// Define
const double = x => x * 2;

// Use
const result = double(21);
console.log(result);  // 42
```

**Key trait:** Exactly one parameter — parentheses optional.

**When to use:** Cleanest syntax for single-parameter functions.

---

## Quick Reference: Which Syntax Should I Use?

| Situation | Use |
|-----------|-----|
| One-liner with multiple params | Arrow implicit: `(a, b) => a + b` |
| One-liner with one param | Arrow implicit: `x => x * 2` |
| Multiple statements | Arrow block: `(x) => { ... }` |
| Need `this` bound to caller | Regular `function` |
| Need `arguments` object | Regular `function` |
| Need hoisting | Function declaration |
| Constructor with `new` | Function declaration |

---

## Arrow Function Nuances

### No parameters: empty parens required

```javascript
// Define
const getTimestamp = () => Date.now();

// Use
const now = getTimestamp();
console.log(now);  // 1751999999999
```

**Rule:** If there are zero parameters, you must use `()`.

---

### One parameter: parens optional

```javascript
// Both work
const double = x => x * 2;
const doubleAlso = (x) => x * 2;

// Use
double(21);  // 42
```

**Rule:** One parameter → parens optional. Zero or multiple → parens required.

---

### Multiple parameters: parens required

```javascript
// Define
const add = (a, b) => a + b;

// Use
add(5, 10);  // 15
```

**Rule:** Two or more parameters → parens required.

---

### Returning an object literal: wrap in parens

```javascript
// ❌ This doesn't work — JavaScript thinks {} is the function body
const makePerson = (name, age) => { name: name, age: age };
// Returns undefined!

// ✅ Wrap in parens to return an object
const makePerson = (name, age) => ({ name: name, age: age });

// Use
const alice = makePerson('Alice', 30);
console.log(alice);  // { name: 'Alice', age: 30 }
```

**Rule:** Returning an object literal from an implicit-return arrow function requires wrapping the object in `()`.

---

### Block body: explicit return required

```javascript
// Define
const processValue = (value) => {
  const doubled = value * 2;
  const withBonus = doubled + 1;
  return withBonus;
};

// Use
processValue(21);  // 43
```

**Rule:** Once you use `{}`, you're in a block body. If you want to return something, you must write `return` explicitly.

---

### Common mistake: forgetting the return in a block body

```javascript
// ❌ Bug: This returns undefined
const add = (a, b) => {
  a + b;
};

add(5, 10);  // undefined!

// ✅ Fix: Add explicit return
const addFixed = (a, b) => {
  return a + b;
};

addFixed(5, 10);  // 15
```

This is the #1 arrow function bug. You add `{}` for "clarity" and forget the function no longer auto-returns.

---

## When Arrow Functions Behave Differently

### `this` binding

```javascript
// Regular function: `this` depends on how it's called
const person = {
  name: 'Alice',
  greet: function() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

person.greet();  // "Hello, I'm Alice"

const detachedGreet = person.greet;
detachedGreet();  // "Hello, I'm undefined" — `this` is lost

// Arrow function: `this` is captured from surrounding scope
const person2 = {
  name: 'Bob',
  greet: () => {
    console.log(`Hello, I'm ${this.name}`);
  }
};

person2.greet();  // "Hello, I'm undefined" — `this` is NOT person2
```

**Rule of thumb:**

- Arrow function: `this` = whatever `this` was where the function was **defined** (lexical scope)
- Regular function: `this` = whatever called the function (dynamic scope)

---

### `arguments` object

```javascript
// Regular function: has access to `arguments`
function logAll() {
  console.log(arguments);  // [1, 2, 3, 4]
}

logAll(1, 2, 3, 4);

// Arrow function: NO `arguments`
const logAllArrow = () => {
  console.log(arguments);  // ReferenceError: arguments is not defined
};

// Use rest parameters instead
const logAllFixed = (...args) => {
  console.log(args);  // [1, 2, 3, 4]
};

logAllFixed(1, 2, 3, 4);
```

**Rule:** Arrow functions don't have `arguments`. Use rest parameters (`...args`) instead.

---

### Constructor with `new`

```javascript
// Regular function: can be a constructor
function Person(name) {
  this.name = name;
}

const alice = new Person('Alice');
console.log(alice.name);  // "Alice"

// Arrow function: CANNOT be used with `new`
const PersonArrow = (name) => {
  this.name = name;
};

const bob = new PersonArrow('Bob');  // TypeError: PersonArrow is not a constructor
```

**Rule:** Arrow functions cannot be used as constructors.

---

## Rest Parameters (`...`)

Collects remaining arguments into an array.

```javascript
// Define
const logAll = (first, ...rest) => {
  console.log(first);  // 1
  console.log(rest);   // [2, 3, 4, 5]
};

// Use
logAll(1, 2, 3, 4, 5);
```

**Common use case: logging with context**

```javascript
const logWithTimestamp = (level, ...messages) => {
  const timestamp = new Date().toISOString();
  console.log(`${timestamp} [${level}]`, ...messages);
};

logWithTimestamp('INFO', 'User logged in');
// "2025-07-08T12:00:00.000Z [INFO] User logged in"

logWithTimestamp('ERROR', 'DB connection failed', { retryCount: 3 });
// "2025-07-08T12:00:01.000Z [ERROR] DB connection failed { retryCount: 3 }"
```

**Rule:** Rest parameters collect remaining arguments into an array. Always last in the parameter list.

---

## Common Patterns

### Optional method call

```javascript
const obj = {
  onSave: (result) => console.log('Saved:', result)
};

// Before
if (obj.onSave) {
  obj.onSave({ status: 'ok' });
}

// After
obj.onSave?.({ status: 'ok' });  // "Saved: { status: 'ok' }"
```

### Async arrow function

```javascript
// Define
const fetchUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

// Use
const user = await fetchUser(123);
console.log(user);

// Implicit return of promise (single expression)
const fetchUserShort = async (id) => (await fetch(`/api/users/${id}`)).json();
```

### Callback function

```javascript
// Define a function that takes a callback
const processItems = (items, callback) => {
  items.forEach(callback);
};

// Use
const users = ['Alice', 'Bob', 'Charlie'];
processItems(users, (name) => {
  console.log(`Processing ${name}`);
});
// "Processing Alice"
// "Processing Bob"
// "Processing Charlie"
```

### Returning a function (closure)

```javascript
// Define a function that returns a function
const createMultiplier = (factor) => {
  return (value) => value * factor;
};

// Use
const double = createMultiplier(2);
const triple = createMultiplier(3);

double(10);  // 20
triple(10);  // 30
```

---

## Quick Reference Card

| I want to... | Use |
|--------------|-----|
| One-liner, multiple params | `const f = (a, b) => a + b;` |
| One-liner, single param | `const f = x => x * 2;` |
| Multiple statements | `const f = (x) => { ...; return y; };` |
| Return an object literal | `const f = (x) => ({ key: x });` |
| No parameters | `const f = () => value;` |
| Collect remaining args | `(...args) => { }` |
| Call method only if exists | `obj.method?.()` |
| Constructor with `new` | Use function declaration |
| Need `arguments` object | Use function declaration |
| Need dynamic `this` | Use function declaration |
| Need lexical `this` | Use arrow function |
| Async one-liner | `async (x) => (await fetch(x)).json()` |
