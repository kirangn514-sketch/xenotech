# Chapter 2 - Hoisting in JavaScript

> Level: Beginner → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 3 Hours

---

# Table of Contents

1. What is Hoisting?
2. Is JavaScript Really Moving Code?
3. Why Hoisting Exists
4. Memory Creation Phase
5. Variable Hoisting
6. Function Hoisting
7. var vs let vs const
8. Temporal Dead Zone (TDZ)
9. Function Declaration vs Function Expression
10. Arrow Functions and Hoisting
11. Block Scope and Hoisting
12. Hoisting in Modules
13. Common Mistakes
14. Interview Questions
15. Cheat Sheet

---

# 1. What is Hoisting?

## Definition

Hoisting is JavaScript's behavior during the **Memory Creation Phase** where it allocates memory for variables and functions **before** executing the code.

> **Important**
>
> JavaScript does **NOT** physically move your code to the top.
>
> Hoisting is the result of how the JavaScript Engine creates the Execution Context.

---

# Biggest Interview Myth

Many tutorials say:

```
JavaScript moves variables to the top.
```

❌ Wrong

Actual process

```
Source Code

↓

Execution Context Created

↓

Memory Allocated

↓

Code Executes
```

Nothing is physically moved.

---

# Example

```javascript
console.log(a);

var a = 10;
```

People think JavaScript converts it into:

```javascript
var a;

console.log(a);

a = 10;
```

It **looks** like this behavior, but internally the engine does something different.

---

# Internal Working

Step 1

JavaScript reads entire file.

```
console.log(a);

var a = 10;
```

Step 2

Global Execution Context is created.

---

# Memory Creation Phase

```
Memory

a

↓

undefined
```

Function declarations would also be stored here.

---

# Execution Phase

Execute line by line.

```
console.log(a);
```

Output

```
undefined
```

Next line

```
a = 10
```

Memory becomes

```
a

↓

10
```

---

# Execution Diagram

```
Source Code

↓

Memory Phase

a → undefined

↓

Execution Phase

console.log(a)

↓

undefined

↓

a = 10

↓

Program Ends
```

---

# Why Does Hoisting Exist?

Imagine JavaScript didn't allocate memory first.

```javascript
function greet() {
    console.log("Hello");
}

greet();
```

Question:

How would JavaScript know what `greet` is when execution reaches `greet()`?

Answer:

Because memory for the function was created before execution started.

---

# Variable Hoisting

Example

```javascript
var city = "Mumbai";
```

Memory Phase

```
city

↓

undefined
```

Execution

```
city = "Mumbai"
```

---

# Example

```javascript
console.log(city);

var city = "Pune";
```

Memory

```
city

↓

undefined
```

Execution

```
console.log(undefined)

↓

city = "Pune"
```

Output

```
undefined
```

---

# Function Hoisting

Function declarations are hoisted differently.

Example

```javascript
greet();

function greet() {
    console.log("Hello");
}
```

Memory Phase

```
greet

↓

Entire Function
```

Execution

```
greet()

↓

Hello
```

Output

```
Hello
```

Notice

The whole function exists before execution begins.

---

# Why Functions Behave Differently

Variables receive:

```
undefined
```

Functions receive:

```
Entire Function Object
```

Diagram

```
Memory Phase

var a

↓

undefined


function greet()

↓

Entire Function
```

---

# var vs let vs const

This is one of the most asked interview questions.

| Feature | var | let | const |
|----------|-----|-----|--------|
| Hoisted | ✅ Yes | ✅ Yes | ✅ Yes |
| Initialized | `undefined` | ❌ No | ❌ No |
| Scope | Function | Block | Block |
| Redeclare | ✅ | ❌ | ❌ |
| Reassign | ✅ | ✅ | ❌ |

Most people incorrectly say:

```
let and const are not hoisted.
```

❌ Incorrect

They **are hoisted**, but they remain **uninitialized** until execution reaches their declaration.

---

# Example

```javascript
console.log(a);

let a = 10;
```

Memory Phase

```
a

↓

<uninitialized>
```

Execution

```
console.log(a)
```

Result

```
ReferenceError:
Cannot access 'a' before initialization
```

---

# Temporal Dead Zone (TDZ)

## Definition

The TDZ is the period between:

1. Memory allocation
2. Variable initialization

Diagram

```
Memory Created
      │
      ▼
a exists

↓

TDZ

↓

let a = 10

↓

Initialized

↓

Usable
```

---

# Example

```javascript
{
    console.log(score);

    let score = 100;
}
```

Memory Phase

```
score

↓

<uninitialized>
```

Execution

```
console.log(score)

↓

ReferenceError
```

After

```
score = 100
```

TDZ ends.

---

# Why Does TDZ Exist?

Without TDZ

```javascript
let total;

console.log(total);
```

You might accidentally use a variable before assigning a meaningful value.

TDZ helps catch programming mistakes early.

---

# const Hoisting

Example

```javascript
console.log(PI);

const PI = 3.14;
```

Memory Phase

```
PI

↓

<uninitialized>
```

Execution

```
ReferenceError
```

---

# Function Expression

Example

```javascript
greet();

var greet = function () {
    console.log("Hello");
};
```

Memory

```
greet

↓

undefined
```

Execution

```
greet()

↓

TypeError:
greet is not a function
```

Later

```
greet

↓

Function
```

---

# Arrow Function

Example

```javascript
sayHi();

const sayHi = () => {
    console.log("Hi");
};
```

Memory

```
sayHi

↓

<uninitialized>
```

Execution

```
ReferenceError
```

---

# Comparison

## Function Declaration

```javascript
hello();

function hello() {}
```

✅ Works

---

## Function Expression

```javascript
hello();

var hello = function() {};
```

❌ TypeError

---

## Arrow Function

```javascript
hello();

const hello = () => {};
```

❌ ReferenceError

---

# Block Scope

Example

```javascript
{
    var a = 1;
    let b = 2;
}

console.log(a);
console.log(b);
```

Output

```
1

ReferenceError
```

Reason

`var` is function-scoped.

`let` is block-scoped.

---

# Block Memory

```
Global

↓

a

↓

1


Block

↓

b

↓

2
```

When the block ends,

`b` becomes inaccessible.

---

# Hoisting in Modules

ES Modules are also hoisted, but they have module scope.

Example

```javascript
import { add } from "./math.js";

console.log(add(1, 2));
```

Imports are resolved before module execution.

---

# Real Interview Example

```javascript
var a = 10;

function test() {
    console.log(a);

    var a = 20;
}

test();
```

Memory Phase (inside `test`)

```
a

↓

undefined
```

Execution

```
console.log(undefined)

↓

a = 20
```

Output

```
undefined
```

---

# Another Interview Question

```javascript
let x = 10;

function demo() {
    console.log(x);

    let x = 20;
}

demo();
```

Memory Phase (inside `demo`)

```
x

↓

<uninitialized>
```

Execution

```
console.log(x)
```

Output

```
ReferenceError
```

---

# Common Mistakes

❌ Thinking JavaScript physically moves declarations.

❌ Saying `let` and `const` are not hoisted.

❌ Confusing `ReferenceError` with `TypeError`.

❌ Assuming function expressions behave like function declarations.

---

# Best Practices

✅ Prefer `const` by default.

✅ Use `let` only when reassignment is required.

✅ Avoid `var` in modern code.

✅ Declare variables close to where they are used.

✅ Do not rely on hoisting for readability.

---

# Senior Interview Questions

### Beginner

- What is hoisting?
- Is JavaScript moving code?
- Why does `var` print `undefined`?

### Intermediate

- Explain the Memory Creation Phase.
- What is the Temporal Dead Zone?
- Why are function declarations callable before their declaration?

### Advanced

Explain the output:

```javascript
console.log(foo);

var foo = function () {
    return "Hello";
};
```

Answer

Memory

```
foo

↓

undefined
```

Execution

```
console.log(undefined)

↓

foo = Function
```

Output

```
undefined
```

---

# Cheat Sheet

| Declaration | Hoisted | Initial Value | Access Before Declaration |
|-------------|----------|---------------|---------------------------|
| var | ✅ | `undefined` | ✅ (`undefined`) |
| let | ✅ | `<uninitialized>` | ❌ (`ReferenceError`) |
| const | ✅ | `<uninitialized>` | ❌ (`ReferenceError`) |
| Function Declaration | ✅ | Entire Function | ✅ |
| Function Expression (`var`) | ✅ | `undefined` | ❌ (`TypeError`) |
| Arrow Function (`const`) | ✅ | `<uninitialized>` | ❌ (`ReferenceError`) |

---

# Key Takeaways

✅ Hoisting is a result of the **Memory Creation Phase**, not code movement.

✅ `var`, `let`, `const`, and function declarations are all hoisted.

✅ `var` is initialized to `undefined`.

✅ `let` and `const` are hoisted but remain in the **Temporal Dead Zone (TDZ)** until initialization.

✅ Function declarations are fully initialized during the memory phase, while function expressions and arrow functions follow the initialization rules of the variable they are assigned to.

✅ Understanding hoisting requires understanding **Execution Context**, which is why it was covered first.
