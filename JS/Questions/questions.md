# Section 1: Variables, Scope, Hoisting

## 1. What is the difference between `var`, `let`, and `const`?

**Answer:**

| Feature                  | `var`                            | `let`        | `const`      |
| ------------------------ | -------------------------------- | ------------ | ------------ |
| Scope                    | Function-scoped                  | Block-scoped | Block-scoped |
| Hoisting                 | Yes (initialized as `undefined`) | Yes (TDZ)    | Yes (TDZ)    |
| Reassignment             | Yes                              | Yes          | ❌ No        |
| Redeclaration            | ✅ Yes (within same scope)       | ❌ No        | ❌ No        |
| Temporal Dead Zone (TDZ) | ❌ No                            | ✅ Yes       | ✅ Yes       |

**Example:**

```js
function testVarLetConst() {
  console.log(x); // undefined (hoisted)
  // console.log(y); // ReferenceError (TDZ)

  var x = 1;
  let y = 2;
  const z = 3;

  x = 10; // ✅ allowed
  y = 20; // ✅ allowed
  // z = 30;  // ❌ TypeError
}
```

Tips to Remember:

Prefer const by default; use let only if you need reassignment.

Avoid var in modern JS – it introduces function-level scope and hoisting pitfalls.

---

## 2. What is hoisting in JavaScript?

**Answer:**
Hoisting is JavaScript's behavior of moving declarations to the top of their scope (function or global) during the memory creation phase, before any code is executed.

This means:

- Variables declared with var, and function declarations, are processed before the code runs.

- let and const are also hoisted, but not initialized — leading to the Temporal Dead Zone (TDZ).

**_JavaScript Execution Context Has Two Main Phases:_**

1. Memory Creation Phase (a.k.a. "Hoisting Phase")

   - The JS engine scans the code before execution.
   - Memory is allocated:
     - var is hoisted and initialized as undefined.
     - let / const are hoisted but stay uninitialized (in TDZ).
     - Function declarations are hoisted with their entire body.

2. Execution Phase
   - Code runs line by line.
   - Values are assigned.
   - If you try to access let/const before their line, you get a ReferenceError due to TDZ.

**_Example: var vs let_**

```javascript
console.log(a); // undefined (hoisted)
var a = 10;

console.log(b); // ❌ ReferenceError (TDZ)
let b = 20;
```

Behind the scenes:

```javascript
// Memory Creation Phase:
var a = undefined;
let b; // in TDZ

// Execution Phase:
console.log(a); // undefined
a = 10;
console.log(b); // ❌ ReferenceError
b = 20;
```

---

## 3. [Coding] What will be the output of the following hoisting-related code snippet?

---

### Problem Statement

Predict the output of the following code and explain **why**. This problem is designed to test your deep understanding of **hoisting**, **shadowing**, **function declarations vs expressions**, and the **temporal dead zone**.

---

### Code Snippet

```js
var a = 1;

function demo() {
  console.log(a); // Line 1
  var a = 2;
  let b = 3;

  function a() {
    return 10;
  }

  console.log(a); // Line 2
  console.log(b); // Line 3
}

demo();
```

---

### Step-by-Step Breakdown

---

#### Step 1: Global Scope

```js
var a = 1;
```

Declares a global variable `a` initialized to `1`.

---

#### Step 2: Entering the `demo` function

We need to analyze the **entire function scope** before execution begins due to **hoisting**.

What gets hoisted?

1. Function declarations are hoisted **before variables**, so:

   ```js
   function a() {
     return 10;
   }
   ```

   is hoisted to the top and assigned as `a`.

2. `var a = 2` is hoisted **after** the function declaration:

   - It declares `a`, but doesn’t overwrite the function — it only **initializes it later**.
   - So `a` is **first a function**, then re-assigned to `2`.

3. `let b = 3` is hoisted but **not initialized** — it's in the **TDZ** until the line where it's declared.

So the effective rewritten version of the function before execution is:

```js
function demo() {
  var a = function a() {
    return 10;
  }; // function declaration
  var a; // redundant var hoisting — does nothing
  let b; // TDZ for b

  console.log(a); // Line 1
  a = 2;
  b = 3;

  console.log(a); // Line 2
  console.log(b); // Line 3
}
```

---

### Execution Flow

- **Line 1:** `console.log(a)`

  - `a` is currently the function `a() { return 10 }`
  - So the output will be: `function a() { return 10 }`

- **After Line 1:** `a = 2` (assignment, not declaration)

- **Then:** `b = 3` (still safe, because we're past the TDZ)

- **Line 2:** `console.log(a)`

  - `a` is now `2`, so prints: `2`

- **Line 3:** `console.log(b)`
  - `b` is `3`, so prints: `3`

---

### Final Output

```
[Function: a]
2
3
```

---

### Tips and Patterns to Remember

- **Function declarations are hoisted before `var` variables.**
- If a variable and function share the same name, the function comes first during hoisting.
- `var` hoists the declaration but **not** the value.
- `let` and `const` are hoisted but live in the **temporal dead zone** until initialization.
- Even though `var a` comes before `function a()` in the code, the function takes precedence.

---

### Follow-Up Ideas

- Change `var a = 2` to `let a = 2` — what happens?
- What if `function a()` was changed to a `const a = function() {}` instead?
- How would this behave differently in **strict mode**?

---

## 4. What is the Temporal Dead Zone (TDZ)?

The Temporal Dead Zone (TDZ) is the time between entering the block and the actual declaration where let or const exists but cannot be accessed.

```javascript
{
  // TDZ starts
  // console.log(x); // ❌ ReferenceError
  let x = 5; // TDZ ends
}
```

Even though x is hoisted, it isn’t initialized, so any access before let x = 5 is a ReferenceError.

Why is TDZ useful?
It helps catch bugs early by ensuring variables are explicitly declared before use.

---

## 5. [Coding] Create a closure-based counter with `increment()` and `getCount()` methods

---

### Problem Statement

Implement a function that returns an object with two methods:

- `increment()` increases an internal counter by 1.
- `getCount()` returns the current value of the counter.

The counter value should remain private and not be accessible directly from outside. This tests understanding of **closures** and **data encapsulation** in JavaScript.

---

### Examples

```js
const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2

// Should not allow direct access to the internal count
console.log(counter.count); // undefined
```

---

### Step-by-Step Breakdown

---

**Step 1: Understand closures**

A closure is formed when an inner function remembers and can access variables from its outer function, even after the outer function has returned.

We’ll use this to:

- Define `let count = 0` in the outer function
- Return an object of methods (`increment`, `getCount`) that access `count` through closure

---

**Step 2: Define `createCounter()`**

We create an outer function that initializes the count and returns an object with two inner functions that form a closure over that variable.

```js
function createCounter() {
  let count = 0; // private variable

  return {
    increment: function () {
      count++;
    },
    getCount: function () {
      return count;
    },
  };
}
```

---

**Step 3: Call and test**

```js
const counter = createCounter();

counter.increment();
counter.increment();

console.log(counter.getCount()); // 2
console.log(counter.count); // undefined (no direct access)
```

---

### Final Implementation

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count++;
    },
    getCount() {
      return count;
    },
  };
}
```

---

### Tips and Patterns to Remember

- A **closure** lets inner functions "remember" variables from an outer function even after it finishes.
- This is a clean way to achieve **encapsulation** without using classes or private fields.
- This pattern is commonly used in:
  - Counter modules
  - Event handling systems
  - Memoization logic
  - Custom hooks in frameworks like React

---

### Follow-Up Ideas

- Add a `reset()` method
- Allow `increment(n)` to increase the count by a given number
- Make a countdown version (start from a number and go down)
- Convert to ES6 class syntax and compare how closures differ

---

# Section 2: Data Types, Coercion, and Equality

## 6. [Theory] What is the difference between `==` and `===` in JavaScript? Why does it matter?

#### Short Answer:

- `==` is the **loose equality** operator: it compares **after performing type coercion**.
- `===` is the **strict equality** operator: it compares **without type coercion**, meaning both **type and value** must match.

#### Deep Dive Explanation:

##### `==` (Loose Equality):

- JS tries to **convert types** to match before comparing.
- This can lead to **unexpected results** if you're not careful.

```js
0 == false; // true   → because false is coerced to 0
"" == 0; // true   → '' becomes 0
null == undefined; // true → special case
"5" == 5; // true   → string '5' is coerced to number 5
```

##### `===` (Strict Equality):

No type conversion. Types must match.

```js
0 === false; // false  → different types
"5" === 5; // false  → string !== number
null === undefined; // false
5 === 5; // true   → same type and value
```

##### Example

### 🧪 Example Table

| Expression           | Type Coercion? | Result  |
| -------------------- | -------------- | ------- |
| `false == 0`         | ✅ Yes         | `true`  |
| `false === 0`        | ❌ No          | `false` |
| `'' == false`        | ✅ Yes         | `true`  |
| `'' === false`       | ❌ No          | `false` |
| `null == undefined`  | ✅ Yes         | `true`  |
| `null === undefined` | ❌ No          | `false` |
| `[] == false`        | ✅ Yes         | `true`  |
| `[] === false`       | ❌ No          | `false` |
| `[] == ![]`          | ✅ Yes         | `true`  |

When should I use ==?
Only when you intentionally want type coercion.

Example: checking for both null and undefined

```js
if (value == null) {
  // true for both null and undefined
}
```

Otherwise, prefer === for clean, predictable comparisons.

#### Tips to Remember

- Use === and !== by default.
- == is error-prone unless you're fully aware of coercion behavior.
- In interviews, always explain type coercion clearly if you use ==.

---

## 7. [Theory] What are truthy and falsy values in JavaScript?

---

### Definition

In JavaScript, **every value** has an inherent **boolean truthiness or falsiness** when used in a **boolean context** (e.g., inside an `if` condition).

- A **truthy** value is one that **evaluates to `true`** when converted to a boolean.
- A **falsy** value is one that **evaluates to `false`** when converted to a boolean.

This behavior is crucial in conditional statements, loops, and logical operations.

---

### Falsy Values (Only 7)

These **7 values** are considered falsy in JavaScript:

| Value       | Explanation                        |
| ----------- | ---------------------------------- |
| `false`     | The literal boolean `false`        |
| `0`         | The number zero                    |
| `-0`        | Negative zero                      |
| `0n`        | BigInt zero                        |
| `""`        | Empty string                       |
| `null`      | Represents absence of value        |
| `undefined` | Variable declared but not assigned |
| `NaN`       | Not a Number                       |

> Any value **other than these 7** is considered **truthy**.

---

### Truthy Examples

| Value                | Why it's truthy                                   |
| -------------------- | ------------------------------------------------- |
| `"hello"`            | Non-empty string                                  |
| `42`                 | Non-zero number                                   |
| `[]`                 | Empty array (still truthy!)                       |
| `{}`                 | Empty object                                      |
| `function() {}`      | Functions are always truthy                       |
| `Infinity`           | Not one of the falsy values                       |
| `new Boolean(false)` | Object is always truthy, even if it wraps `false` |

---

### Common Gotchas

#### 1. Empty array and object are truthy:

```js
if ([]) {
  console.log("Truthy!"); // ✅ will print
}
```

#### 2. `false` object is truthy:

```js
if (new Boolean(false)) {
  console.log("Still truthy!"); // ✅ will print
}
```

#### 3. Comparing falsy values with `==`

```js
console.log(false == 0); // true
console.log(false == ""); // true
console.log(null == undefined); // true
```

> Prefer `===` over `==` to avoid confusion caused by type coercion.

---

### Use Cases

- **Conditionals:**

  ```js
  if (userInput) {
    // executes only if userInput is truthy
  }
  ```

- **Short-circuiting:**
  ```js
  const name = userInput || "Guest"; // defaults to "Guest" if userInput is falsy
  ```

---

### Tip

To **convert any value to boolean**, use `Boolean()` or double NOT:

```js
Boolean(""); // false
!!"hello"; // true
!!{}; // true
!!undefined; // false
```

---

### Summary

- There are exactly **7 falsy values** in JS.
- Everything else is **truthy**.
- Truthy/falsy checks are key in conditionals and logical operations.
- Always use `===` to avoid type coercion surprises when comparing.

---

## 8. [Coding] Implement `deepEqual(obj1, obj2)` — Fixing Array/Object Confusion

---

### Problem Statement

We want to implement a function `deepEqual(a, b)` that returns `true` if the two values are deeply equal. That means:

- If both are primitives, compare them directly.
- If both are objects or arrays, compare all keys/elements recursively.
- If their structure is different (e.g. array vs object), return `false`.

---

### Examples

```js
deepEqual(5, 5); // true
deepEqual({ a: 1 }, { a: 1 }); // true
deepEqual({ a: 1 }, { a: 2 }); // false
deepEqual([1, { x: 2 }], [1, { x: 2 }]); // true
deepEqual([], []); // true
deepEqual({}, []); // false
deepEqual({ 0: "foo" }, ["foo"]); // false
```

---

### Step-by-Step Breakdown

---

**Step 1: Check for strict equality**

If both values are strictly equal (e.g., `5 === 5`, or they reference the same object), return `true` immediately.

```js
if (a === b) return true;
```

---

**Step 2: Eliminate non-objects and null**

If either value is not an object or is `null`, return `false`. This avoids treating primitive mismatches or null values as equal.

```js
if (
  typeof a !== "object" ||
  a === null ||
  typeof b !== "object" ||
  b === null
) {
  return false;
}
```

---

**Step 3: Ensure arrays and objects are not confused**

JavaScript treats both arrays and objects as `typeof 'object'`, so we must explicitly check if one is an array and the other isn't.

```js
const isArrayA = Array.isArray(a);
const isArrayB = Array.isArray(b);
if (isArrayA !== isArrayB) return false;
```

---

**Step 4: Recursively compare arrays (if both are arrays)**

If both are arrays, check that:

- Their lengths are the same.
- Every element at the same index is deeply equal.

```js
if (isArrayA && isArrayB) {
  if (a.length !== b.length) return false;
  for (let i = 0; i < a.length; i++) {
    if (!deepEqual(a[i], b[i])) return false;
  }
  return true;
}
```

---

**Step 5: Recursively compare objects (if both are plain objects)**

- Compare the number of keys.
- Make sure each key in `a` exists in `b`.
- Recursively compare the values at each key.

```js
const keysA = Object.keys(a);
const keysB = Object.keys(b);

if (keysA.length !== keysB.length) return false;

for (let key of keysA) {
  if (!Object.prototype.hasOwnProperty.call(b, key)) return false;
  if (!deepEqual(a[key], b[key])) return false;
}
return true;
```

---

### Final Implementation

```js
function deepEqual(a, b) {
  if (a === b) return true;

  if (
    typeof a !== "object" ||
    a === null ||
    typeof b !== "object" ||
    b === null
  ) {
    return false;
  }

  const isArrayA = Array.isArray(a);
  const isArrayB = Array.isArray(b);
  if (isArrayA !== isArrayB) return false;

  if (isArrayA && isArrayB) {
    if (a.length !== b.length) return false;
    for (let i = 0; i < a.length; i++) {
      if (!deepEqual(a[i], b[i])) return false;
    }
    return true;
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  for (let key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(b, key)) return false;
    if (!deepEqual(a[key], b[key])) return false;
  }

  return true;
}
```

---

### Tips and Patterns to Remember

- Always differentiate `null` from objects. (`typeof null === "object"` is a JavaScript quirk.)
- Use `Array.isArray()` to distinguish between arrays and objects.
- Use `Object.prototype.hasOwnProperty.call()` to safely check for keys.
- Avoid `JSON.stringify()` for deep equality. It breaks on:
  - Functions
  - `undefined`
  - Circular references
  - Key order differences
- Use recursion for nested comparisons. This is a classic recursive problem.

---

## 9. [Coding] Attempt a type coercion puzzle: determine output for a series of confusing expressions.

---

### Problem Statement

You're given a list of JavaScript expressions. Predict their output **without running them** and explain why each behaves that way.

These puzzles are designed to test your understanding of:

- Implicit type coercion
- Loose vs strict equality
- Unary and binary operators
- Truthy/falsy conversions
- Common real-world bugs due to type coercion

---

### Puzzle Set

```js
console.log(2 + "-2"); // ?
console.log("6" > 2); // ?
console.log(1 + +"abc"); // ?
console.log(false == "0"); // ?
console.log([] == false); // ?
console.log(null == undefined); // ?
console.log([] == ""); // ?
console.log("" == 0); // ?
console.log([] + {}); // ?
console.log({} + []); // ?
```

---

### Step-by-Step Solutions & Explanations

---

#### 1. `2 + "-2"` → `'2-2'`

- `2` is a number, `"-2"` is a string
- `+` triggers **string concatenation** if either operand is a string
- So: `'2' + '-2'` → ✅ `'2-2'`

---

#### 2. `"6" > 2` → `true`

- `"6"` is coerced to number `6`
- `6 > 2` → ✅ `true`

---

#### 3. `1 + +"abc"` → `NaN`

- `+"abc"` → `NaN` (invalid numeric coercion)
- `1 + NaN` → `NaN`

---

#### 4. `false == '0'` → `true`

- `'0'` → number `0`
- `false` → number `0`
- So: `0 == 0` → ✅ `true`

---

#### 5. `[] == false` → `true`

- `[]` → `''` → `0`
- `false` → `0`
- `0 == 0` → ✅ `true`

---

#### 6. `null == undefined` → `true`

- Special loose equality rule: `null == undefined` → ✅ `true`

---

#### 7. `[] == ''` → `true`

- `[]` → `''`
- So: `'' == ''` → ✅ `true`

---

#### 8. `'' == 0` → `true`

- `''` → `0`
- So: `0 == 0` → ✅ `true`

---

#### 9. `[] + {}` → `'[object Object]'`

- `[]` → `''`, `{}` → `'[object Object]'`
- `'' + '[object Object]'` → `'[object Object]'`

---

#### 10. `{ } + []` → `'0'` (depending on how it's parsed)

- If `{}` is interpreted as an empty block, then `+[]` is evaluated → `0`
- In browsers, this often logs: `'0'`

To force `{}` to be an object, wrap in parentheses:

```js
console.log({} + []); // → '[object Object]'
```

---

### Key Takeaways

- `+` is overloaded: addition or string concat depending on operand types.
- Unary `+` tries to convert its operand to number — `+"abc"` → `NaN`
- `==` does implicit coercion. Use `===` unless coercion is intended.
- Objects and arrays convert to strings when coerced (`[].toString()` → `''`).
- Be careful using falsy values (`""`, `0`, `false`, `null`, `undefined`, `NaN`) in conditionals and comparisons.

---

# Section 3: Functions & Closures

## 10. [Theory] What is a closure in JavaScript? Give a real-world use case.

---

### What is a Closure?

A **closure** in JavaScript is a function that **"remembers"** the variables from its **lexical scope**, even when that function is executed outside that scope.

In simple terms, **a closure allows a function to access variables from its outer function even after the outer function has finished executing**.

---

### Breakdown

When you define a function inside another function, the inner function gets access to:

- Its own variables
- The outer function’s variables
- The global variables

And thanks to JavaScript's closure behavior, it **retains access** to those outer variables **even after the outer function returns**.

---

### Why Closures Matter

- **State retention** without using global variables
- **Data encapsulation and privacy**
- **Functional patterns** like currying and partial application
- Used in **event handlers, timers, async callbacks**

---

### Real-World Use Case – Counter with Private State

Imagine you’re building a button that shows how many times it’s been clicked:

```js
function createClickCounter() {
  let count = 0;

  return function increment() {
    count++;
    console.log(`Button clicked ${count} times`);
  };
}

const handleClick = createClickCounter();

handleClick(); // Button clicked 1 times
handleClick(); // Button clicked 2 times
```

#### What's Happening?

- `createClickCounter()` returns an inner function.
- This inner function is a **closure** — it remembers the `count` variable from the outer scope.
- Even though `createClickCounter()` has finished running, the `count` variable is **not garbage-collected**, because `increment()` still has access to it.

---

### Closures for Data Privacy

You can simulate private variables like this:

```js
function BankAccount(initialBalance) {
  let balance = initialBalance;

  return {
    deposit(amount) {
      balance += amount;
    },
    getBalance() {
      return balance;
    },
  };
}

const account = BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500
console.log(account.balance); // undefined (no access!)
```

---

### Interview Tip

Many JS questions around **counters**, **timers**, **currying**, and **modules** are testing your understanding of **closures**.

**Closures + Lexical Scope = Private state & powerful patterns**.

---

### Summary

| Concept     | Explanation                                      |
| ----------- | ------------------------------------------------ |
| Closure     | Function + lexical environment it was created in |
| Purpose     | Retain access to outer variables after execution |
| Common Uses | Counters, encapsulation, async, callbacks        |

---

## 11. [Coding] Implement a function that returns another function (Currying)

---

### Problem Statement

Currying is a functional programming technique where a function with multiple arguments is transformed into a series of functions that each take **one argument at a time**.

Your task is to implement a curried function that:

- Takes one number at a time
- Returns a function for each new argument
- Finally returns the sum of all numbers **when no argument is passed**

---

### Example Usage

```js
add(2)(3)(4)(); // 9
add(1)(-2)(3)(4)(); // 6
add(); // 0
```

---

### Step-by-Step Thinking

#### Step 1: Understand the desired behavior

You want a function `add` that:

- Returns another function for each argument
- Keeps accumulating the total
- Stops when called **without an argument** (i.e. `()`)

This is a classic use-case for closures — we need to retain state (`sum`) across function calls.

---

#### Step 2: Use Closure to Store the Running Total

We wrap the logic in an inner function and **return that same function recursively** until an empty call is made.

---

### Final Code

```js
function add(num) {
  let sum = 0;

  function inner(next) {
    if (next === undefined) return sum;
    sum += next;
    return inner;
  }

  return num === undefined ? 0 : inner(num);
}
```

---

### How it Works

1. The outer `add()` starts the process.
2. Each call with a number accumulates to `sum`.
3. When `()` is called with `undefined`, we return the accumulated `sum`.

---

### Tips & Patterns

| Pattern        | Insight                                    |
| -------------- | ------------------------------------------ |
| Closure        | Retain `sum` across function calls         |
| Recursion      | Return the same function until base case   |
| Flexible Arity | Supports chaining of arguments dynamically |

---

### Common Interview Variants

- Create a curried function that accepts `N` parameters over multiple calls.
- Write a curry utility for `f(a, b, c)` so it can be called like `curry(f)(a)(b)(c)`.
- Extend to support infinite chaining: `add(1)(2)(3)...().valueOf()`

---

### Practice Challenge

Extend the logic so the user can call `.value()` instead of `()`:

```js
add(1)(2)(3).value(); // 6
```

You’ll need to return an object with a `value()` method or override `.toString()` or `.valueOf()` for clean coercion.

---

## 12. [Theory] How does `this` behave in regular vs. arrow functions?

---

### Short Answer

- **Regular functions:** `this` is **dynamic**, determined by **how the function is called**.
- **Arrow functions:** `this` is **lexically bound**, i.e., it inherits `this` from the **surrounding scope** where it was defined.

---

### Explanation with Example

#### 1. Regular Function Example

```js
const user = {
  name: "Ayush",
  sayHi: function () {
    console.log("Hi,", this.name);
  },
};

user.sayHi(); // Hi, Ayush
```

Here, `this` refers to the `user` object because the function is called using `user.sayHi()`.

---

#### 2. Arrow Function Example

```js
const user = {
  name: "Ayush",
  sayHi: () => {
    console.log("Hi,", this.name);
  },
};

user.sayHi(); // Hi, undefined
```

Here, `this` does **not** refer to `user` — it refers to the `this` of the outer scope (usually `window` or `undefined` in strict mode). That’s why we get `undefined`.

---

### Comparison Table

| Feature                 | Regular Function                         | Arrow Function                           |
| ----------------------- | ---------------------------------------- | ---------------------------------------- |
| `this` binding          | Dynamic (based on call-site)             | Lexical (based on surrounding scope)     |
| Can be used as method?  | Yes                                      | Usually no (loses correct `this`)        |
| `new` constructible?    | Yes                                      | No (throws error)                        |
| Suitable for callbacks? | Depends — use `.bind()` or arrow instead | Yes — especially in `setTimeout`, `.map` |

---

### Real-World Bug Example

```js
function Timer() {
  this.seconds = 0;

  setInterval(function () {
    this.seconds++;
    console.log(this.seconds);
  }, 1000);
}

new Timer(); // NaN or TypeError
```

- Inside the regular function, `this` refers to `window`, not the `Timer` instance.

✅ Fix it with arrow function:

```js
function Timer() {
  this.seconds = 0;

  setInterval(() => {
    this.seconds++;
    console.log(this.seconds);
  }, 1000);
}
```

---

### Pro Interview Insight

Use arrow functions:

- When you want to **inherit** `this` from parent scope (e.g., event handlers inside class methods or closures).
- When used as **callback functions** inside methods where traditional `function()` would lose `this`.

Avoid arrow functions:

- When you need a **method** that relies on object context.
- When you want to use the function as a **constructor** (they can’t be used with `new`).

---

### Summary

```js
const obj = {
  name: "JS",
  regular: function () {
    console.log(this.name);
  },
  arrow: () => {
    console.log(this.name);
  },
};

obj.regular(); // "JS"
obj.arrow(); // undefined
```

- Regular: `this` → `obj`
- Arrow: `this` → outer scope (not `obj`)

---

## 13. [Coding] Implement a polyfill for `Function.prototype.bind`

---

### Problem Statement

Create a polyfill for the `bind` method, called `myBind`, that:

- Returns a new function
- When called, calls the original function with a specified `this` context
- Optionally pre-fills arguments (partial application)
- Preserves proper `this` binding when used with `new` (advanced case — optional)

---

### Examples

```js
function greet(greeting, punctuation) {
  return `${greeting}, my name is ${this.name}${punctuation}`;
}

const person = { name: "Ayush" };
const greetAyush = greet.bind(person, "Hello");

greetAyush("!"); // "Hello, my name is Ayush!"
```

---

### Step-by-Step Breakdown

---

**Step 1: Understand how `bind` differs from `call`**

- `call`/`apply` invoke the function immediately
- `bind` returns a **new function** with `this` and optionally some arguments **pre-bound**

```js
const bound = fn.bind(obj, preArg1);
bound(extraArg); // calls fn with obj and both args
```

---

**Step 2: Extend `Function.prototype`**

```js
Function.prototype.myBind = function (thisArg, ...presetArgs) {
  // returns a new function
};
```

Here, `this` is the original function, `thisArg` is the context, and `presetArgs` are the first set of arguments.

---

**Step 3: Return a new function** ( Read slowly and carefully. Read again if not clear at once)

This new function:

- Accepts more arguments
- Calls the original function with both preset and new arguments
- Uses `call` or `apply` to enforce context binding

---

**Final Implementation**

```js
Function.prototype.myBind = function (thisArg, ...presetArgs) {
  const originalFn = this;

  return function (...laterArgs) {
    return originalFn.call(thisArg, ...presetArgs, ...laterArgs);
  };
};
```

---

## 14. [Coding] Implement a polyfill for `Function.prototype.call`

---

### Problem Statement

Implement your own version of the `call` method, i.e., `Function.prototype.myCall`, which mimics the behavior of the native `Function.prototype.call`. This method allows you to invoke any function with a specified `this` context and arguments.

---

### Examples

```js
function greet(age) {
  return `Hello, my name is ${this.name} and I am ${age} years old.`;
}

const person = { name: "Ayush" };

greet.call(person, 35); // "Hello, my name is Ayush and I am 35 years old."
greet.myCall(person, 35); // should return the same
```

---

### Step-by-Step Breakdown

---

**Step 1: Understand how `call` works**

The native `call` method does three things:

1. Temporarily assigns the function to the passed object (`thisArg`)
2. Invokes it with any passed arguments
3. Cleans up the temporary property

Example behavior:

```js
greet.call(obj, arg1, arg2, ...)
// is equivalent to:
obj.tempFn = greet;
obj.tempFn(arg1, arg2, ...);
delete obj.tempFn;
```

---

**Step 2: Extend `Function.prototype`**

To define our own version, we attach it to `Function.prototype`:

```js
Function.prototype.myCall = function (thisArg, ...args) {
  // implementation
};
```

- `this` will refer to the function being called (`greet` in our case)
- `thisArg` is the object to bind as `this`
- `args` are arguments to pass to the function

---

**Step 3: Handle null/undefined context**

If `thisArg` is `null` or `undefined`, we default to the global object (`globalThis`):

```js
thisArg = thisArg ?? globalThis;
```

---

**Step 4: Temporarily assign and call the function**

We attach the function to the object with a temporary key, call it, then delete it.

To avoid name collisions, we use a unique symbol as the key.

---

**Final Implementation**

```js
Function.prototype.myCall = function (thisArg, ...args) {
  // Step 1: Default to globalThis if null/undefined
  thisArg = thisArg ?? globalThis;

  // Step 2: Create a unique key to avoid overwriting existing properties
  const tempFn = Symbol("tempFn"); // can do this without Symbol as well in the interviews. this will get you brownie points

  // Step 3: Assign the function (this) to the object
  thisArg[tempFn] = this;

  // Step 4: Call the function and store the result
  const result = thisArg[tempFn](...args);

  // Step 5: Clean up
  delete thisArg[tempFn];

  return result;
};
```

---

### Tips and Patterns to Remember

- Use `Symbol()` for safe temporary property names.
- `call`, `apply`, and `bind` are part of many interview rounds — especially polyfills.
- `call` executes immediately, passing arguments one-by-one.
- Default to `globalThis` when context is `null` or `undefined` (like native behavior).

---

### Follow-Up Ideas

- Implement `apply` as `myApply(context, argsArray)`
- Implement `bind` as `myBind(context)`

---

### Tips and Patterns to Remember

- `bind` is used for **partial application** and **context preservation**
- `bind` **does not execute immediately** — it returns a new callable function

---

# Section 4: Objects, Prototypes, and Inheritance

## 15. [Theory] How does prototypal inheritance work in JavaScript?

---

### Introduction

JavaScript uses **prototypal inheritance** — an object can inherit properties and methods from another object via a prototype chain.

Instead of classical inheritance (like in Java, C++), where classes inherit from other classes, JavaScript allows **objects to inherit directly from other objects**.

---

### Core Concepts

#### 1. Every object has an internal `[[Prototype]]`

This is accessible via:

- `Object.getPrototypeOf(obj)` (modern way)
- `obj.__proto__` (legacy, not recommended)

#### 2. Prototype Chain

When you try to access a property on an object:

- JavaScript first looks for the property directly on the object.
- If not found, it looks up the prototype chain until it finds the property or reaches `null`.

```js
const parent = { greeting: "Hello" };
const child = Object.create(parent);

console.log(child.greeting); // "Hello"
```

Here, `child` doesn’t have `greeting`, so JS looks in `parent`.

---

### Constructor Functions and `.prototype`

Functions in JavaScript have a special property called `.prototype`. When used with `new`, it sets the prototype of the created object.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function () {
  return `Hi, I am ${this.name}`;
};

const p1 = new Person("Ayush");
console.log(p1.sayHi()); // "Hi, I am Ayush"
```

Here:

- `p1.__proto__ === Person.prototype` → true
- `p1.sayHi()` is found on `Person.prototype`

---

### Prototype Chain Example

```js
const grandParent = { gpProp: "GP" };
const parent = Object.create(grandParent);
parent.pProp = "P";

const child = Object.create(parent);
child.cProp = "C";

console.log(child.cProp); // "C" (own property)
console.log(child.pProp); // "P" (inherited from parent)
console.log(child.gpProp); // "GP" (inherited from grandparent)
```

---

### Object.create vs Class

Both achieve inheritance, but differ in syntax:

```js
// Using Object.create
const parent = {
  greet() {
    return "Hi";
  },
};
const child = Object.create(parent);

// Using ES6 class
class A {
  greet() {
    return "Hi";
  }
}
class B extends A {}

const b = new B();
```

Under the hood, classes still use prototypes. The `class` syntax is syntactic sugar over the prototype-based system.

---

### Summary

- JS uses a **prototype chain** to share behavior across objects.
- Inheritance is **delegation-based**, not copy-based.
- All objects ultimately inherit from `Object.prototype` (unless created with `Object.create(null)`).
- Prototypal inheritance allows flexible and powerful object composition.

---

## 16. [Theory] What’s the difference between `__proto__`, `prototype`, and `constructor`?

---

JavaScript has multiple ways to connect and extend objects. These three — `__proto__`, `prototype`, and `constructor` — are often confused because they all relate to inheritance and object creation.

Let's break them down.

---

### 1. `__proto__` (aka `[[Prototype]]`)

- This is an **internal reference** to the prototype of an object.
- Every object (except the root object) has it.
- It forms the **prototype chain** — how JavaScript performs inheritance.

```js
const person = { name: "Ayush" };
const student = Object.create(person);

console.log(student.__proto__ === person); // true
```

- **Modern & recommended way**: use `Object.getPrototypeOf(obj)` and `Object.setPrototypeOf(obj, proto)`
- `__proto__` is **legacy**, though still widely used.

---

### 2. `.prototype`

- This is a property **only on constructor functions** and **classes**.
- It is **not** present on regular objects.
- When a function is used as a constructor with `new`, the created object's `__proto__` is set to the constructor’s `.prototype`.

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.sayHi = function () {
  return `Hi, I am ${this.name}`;
};

const a1 = new Animal("Leo");

console.log(a1.__proto__ === Animal.prototype); // true
```

So, `.prototype` defines the prototype for all instances created with `new`.

---

### 3. `constructor`

- The `constructor` is a **property on the prototype** object.
- It points back to the function that created the instance.

```js
console.log(a1.constructor === Animal); // true
```

- You can use it to recreate objects or check types (with caution).

---

### Putting it All Together

```js
function Foo() {}
const obj = new Foo();

// Relationships:
obj.__proto__ === Foo.prototype; // true
Foo.prototype.constructor === Foo; // true
Object.getPrototypeOf(obj) === Foo.prototype; // true
```

---

### Visual Summary

```
Function: Foo
|
|-- Foo.prototype  --> { constructor: Foo }
      |
      |-- used as prototype for
            ↓
        obj.__proto__
```

---

### Summary Table

| Term          | Belongs To      | Purpose                                                      |
| ------------- | --------------- | ------------------------------------------------------------ |
| `__proto__`   | Every object    | Points to prototype of object (forms inheritance chain)      |
| `prototype`   | Constructor Fn  | Defines shared props/methods for instances created via `new` |
| `constructor` | On `.prototype` | Points back to function/class that created the object        |

---

### Important Notes

- **Modifying `.prototype`** affects all future instances.
- **Changing `__proto__`** can be done at runtime, but is discouraged due to performance costs.
- Use **`Object.create()`** or **class syntax** for clean inheritance.

---

## 17. [Coding] Create your own version of `Object.create()` using constructor function

---

### Problem Statement

You are given a task to **simulate the behavior of `Object.create(proto)`**.

This means:  
You need to create a new object such that the **prototype of the new object is the object passed in** (`proto`).

---

### Real-Life Analogy

Think of `proto` as a **blueprint** or **parent**.  
You want to create a **child object** that **inherits from this parent**, without copying the properties — just inheriting them.

---

### Step-by-Step Breakdown

#### Step 1: What does `Object.create(proto)` do?

When you call:

```js
const child = Object.create(parent);
```

You're not copying `parent`, you're just saying:

> “Make a new empty object, but whenever this new object doesn’t have a property, go ask `parent`.”

So internally, this new object’s **`__proto__` is set to `parent`**.

---

#### Step 2: Can we do this without `Object.create()`?

Yes! JavaScript gives us a way to set the `__proto__` of an object using **constructor functions**.

---

### Our Plan:

1. Create a temporary constructor function.
2. Set the prototype of this constructor to `proto`.
3. Return a new object using `new`, which links `__proto__` to `proto`.

---

### Code Implementation

```js
function customCreate(proto) {
  function Temp() {} // Step 1: Temporary constructor
  Temp.prototype = proto; // Step 2: Link prototype
  return new Temp(); // Step 3: Create new object
}
```

---

### 🧪 Example Usage

```js
const parent = {
  sayHi() {
    return "Hi from parent!";
  },
};

const child = customCreate(parent);

console.log(child.sayHi()); // Hi from parent!
console.log(child.__proto__ === parent); // true
```

---

### Summary

| Feature                    | `Object.create()` | `customCreate()` |
| -------------------------- | ----------------- | ---------------- |
| Links to prototype         | Yes               | Yes              |
| Copies properties          | No                | No               |
| Uses constructor hack      | Native            | Yes              |
| Works in ES3/ES5 and above | Yes               | Yes              |

---

> So the idea is simple: use a function to create an empty object, and link it to the object you want to use as the prototype.

---

## 18. [Theory] What is the difference between shallow and deep copy?

---

### What is a copy in JavaScript?

In JavaScript, when we copy an object or array, we either create a **new reference** that still points to the same nested values (shallow), or we recursively copy all nested values and create a fully independent structure (deep).

---

### Shallow Copy

A **shallow copy** means:

- The first level of the object is copied.
- Nested objects or arrays are **still referenced** — not cloned.

#### Example:

```js
const original = {
  name: "Ayush",
  address: {
    city: "Pune",
  },
};

const shallow = { ...original };

shallow.name = "Raj"; // ✅ Only updates shallow.name
shallow.address.city = "Delhi"; // ❗ Affects original.address.city too

console.log(original.address.city); // "Delhi"
```

#### Why?

Because `shallow.address` points to the **same object in memory** as `original.address`.

---

### Deep Copy

A **deep copy** means:

- Every level of nested structure is copied.
- There are no shared references between the original and the copy.

#### Example:

```js
const original = {
  name: "Ayush",
  address: {
    city: "Pune",
  },
};

const deep = JSON.parse(JSON.stringify(original)); // simple deep copy (not perfect)

deep.address.city = "Delhi";

console.log(original.address.city); // "Pune" — not affected
```

This deep copy is **independent**, so changing `deep.address.city` does **not** affect the original.

> Note: `JSON.parse(JSON.stringify(...))` has limitations — it doesn’t handle `undefined`, `functions`, `symbols`, or circular references.

---

### Summary Table

| Feature                | Shallow Copy                             | Deep Copy                            |
| ---------------------- | ---------------------------------------- | ------------------------------------ |
| Nested objects copied? | ❌ No (shared references)                | ✅ Yes (recursively copied)          |
| Performance            | Fast                                     | Slower (recursive copying)           |
| Tools / Methods        | `Object.assign()`, spread `...`          | `structuredClone()`, recursion, libs |
| Risk                   | Changes to nested values affect original | Safe, fully independent              |

---

### Interview Tip

If you're unsure whether a method makes a shallow or deep copy:

- Ask yourself: _"If I modify a nested object, will it affect the original?"_
- If **yes**, it's a shallow copy.
- If **no**, it's a deep copy.

---

## 19. [Coding] Implement a deep clone utility without using `structuredClone`

> (Also, do **not** use `JSON.parse(JSON.stringify(...))`)

---

### Problem Statement

Write a function `deepClone(obj)` that deeply copies any object or array, including nested structures, without using:

- `structuredClone()`
- `JSON.parse(JSON.stringify(...))`  
  These are commonly disallowed because:
  - They **fail for special values**: `undefined`, `function`, `symbol`
  - They **remove circular references**
  - They **convert Date to strings**

---

### Step-by-Step Thinking Process

When cloning, ask yourself:

- Is the value a **primitive**? → Return it as is.
- Is it an **array**? → Create a new array, recursively clone each item.
- Is it a **plain object**? → Create a new object, clone each key recursively.

We’ll handle these cases step-by-step.

---

### Step 1: Base Case — Return primitive as-is

```js
function deepClone(value) {
  // If value is not an object or is null, return it directly (e.g., string, number, null)
  if (typeof value !== "object" || value === null) {
    return value;
  }

  // We'll handle arrays and objects below...
}
```

---

### Step 2: Handle arrays

```js
if (Array.isArray(value)) {
  // Create a new array and recursively clone each element
  return value.map((item) => deepClone(item));
}
```

---

### Step 3: Handle plain objects

```js
const result = {};

for (const key in value) {
  // hasOwnProperty check avoids copying properties inherited from the prototype chain
  if (Object.prototype.hasOwnProperty.call(value, key)) {
    result[key] = deepClone(value[key]);
  }
}

return result;
```

> **Why use `hasOwnProperty`?**
>
> - Prevents copying properties from the prototype chain.
> - Safer than using `value.hasOwnProperty(key)` directly, because some objects might not inherit from `Object.prototype` (e.g., created via `Object.create(null)`).

> **Alternative:** Use `Object.keys(value).forEach(...)`, which iterates only over own, enumerable properties.

---

### Final Version — Clean Deep Clone Function

```js
function deepClone(value) {
  if (typeof value !== "object" || value === null) {
    return value; // base case: primitive or null
  }

  if (Array.isArray(value)) {
    return value.map((item) => deepClone(item));
  }

  const result = {};
  for (const key in value) {
    if (Object.prototype.hasOwnProperty.call(value, key)) {
      result[key] = deepClone(value[key]);
    }
  }

  return result;
}
```

---

### Test Cases

```js
const original = {
  name: "John",
  age: 30,
  address: {
    city: "CityName",
    coordinates: {
      lat: 12.34,
      lng: 56.78,
    },
  },
  hobbies: ["reading", "sports"],
};

const copy = deepClone(original);

// Mutate copy
copy.name = "Alice";
copy.address.city = "OtherCity";
copy.hobbies.push("coding");

console.log(original.name); // John
console.log(original.address.city); // CityName
console.log(original.hobbies.length); // 2 — unaffected by mutation in copy
```

---

### Caveats

This version does **not handle**:

- `Date`, `RegExp`, `Map`, `Set`
- `Function`, `Symbol`
- Circular references

You can mention these as limitations in interviews and suggest libraries like `lodash.cloneDeep` for production use.

---

### Patterns to Remember

- Always check for `null` explicitly.
- For objects, prefer `Object.prototype.hasOwnProperty.call(...)` or `Object.keys(...)`.
- Recursively call your function for each property.
- Be aware of special types not handled (e.g., Dates, Maps).

---
