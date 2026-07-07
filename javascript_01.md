# Chapter 1 - JavaScript Execution Context

> Level: Beginner → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 2–3 Hours

---

# Table of Contents

1. What is JavaScript Execution Context?
2. Why Execution Context is Needed
3. JavaScript Execution Model
4. Types of Execution Context
5. Phases of Execution Context
6. Memory Creation Phase
7. Code Execution Phase
8. Execution Context Stack (Call Stack)
9. Detailed Working with Example
10. Function Execution Context
11. Global Object
12. this Keyword
13. Lexical Environment
14. Variable Environment
15. Interview Questions
16. Best Practices
17. Cheat Sheet

---

# 1. What is Execution Context?

Execution Context is the environment in which JavaScript code is executed.

Think of it as a "box" that stores everything JavaScript needs to execute your code.

It contains:

- Variables
- Functions
- Scope information
- Value of `this`
- Outer (parent) environment reference

Without an execution context, JavaScript cannot execute any code.

---

# Real World Analogy

Imagine a chef in a restaurant.

Before cooking, the chef prepares:

- Ingredients
- Cooking tools
- Recipe
- Gas Stove

Only after everything is ready does cooking begin.

Execution Context works the same way.

First it prepares memory.

Then it executes code.

---

# Execution Context Structure

```
Execution Context

+--------------------------------------+
| Variable Environment                 |
|                                      |
| let                                  |
| const                                |
| var                                  |
| function declarations                |
+--------------------------------------+

+--------------------------------------+
| Lexical Environment                  |
| Parent Scope Reference               |
+--------------------------------------+

+--------------------------------------+
| this                                 |
+--------------------------------------+
```

---

# Why Execution Context Exists

Suppose JavaScript directly started executing code.

```javascript
console.log(a);

var a = 10;
```

Question:

Where should JavaScript find `a`?

It doesn't know yet.

Therefore JavaScript first scans the code and creates memory.

Then execution starts.

---

# JavaScript Execution Flow

```
JavaScript File

↓

Create Global Execution Context

↓

Memory Creation Phase

↓

Execution Phase

↓

Function Call?

↓

Create Function Execution Context

↓

Execute Function

↓

Destroy Function Context

↓

Back to Global Context

↓

Program Ends
```

---

# Types of Execution Context

JavaScript has three execution contexts.

## 1 Global Execution Context (GEC)

Created once.

Example

```javascript
var name = "John";

function hello(){}

console.log(name);
```

Everything outside functions belongs to Global Execution Context.

---

## 2 Function Execution Context (FEC)

Created every time a function is called.

Example

```javascript
function add(a,b){
   return a+b;
}

add(10,20);
```

Calling `add()` creates a new execution context.

---

## 3 Eval Execution Context

Created by

```javascript
eval()
```

Rarely used.

Not recommended.

---

# Memory Creation Phase

Also called

Creation Phase

Compilation Phase

Hoisting Phase

During this phase JavaScript DOES NOT execute code.

It only allocates memory.

---

Example

```javascript
var x = 10;

let y = 20;

const z = 30;

function greet(){}

console.log(x);
```

Memory becomes

```
Memory

x → undefined

y → <uninitialized>

z → <uninitialized>

greet → Entire Function
```

Notice

Functions are stored completely.

Variables declared using var become undefined.

let and const remain inside the Temporal Dead Zone.

---

# Code Execution Phase

Now JavaScript executes line by line.

Example

```javascript
var x = 10;

console.log(x);
```

Step 1

Memory

```
x → undefined
```

Step 2

Execute

```
x = 10
```

Memory

```
x → 10
```

Step 3

```
console.log(x)
```

Output

```
10
```

---

# Example 1

```javascript
var a = 5;

var b = 10;

function sum(){

console.log(a+b);

}

sum();
```

Memory Creation

```
a → undefined

b → undefined

sum → function
```

Execution

```
a = 5

b = 10

call sum()
```

Function Context

```
console.log(15)
```

Output

```
15
```

---

# Example 2

```javascript
console.log(a);

var a = 50;
```

Memory Phase

```
a → undefined
```

Execution

```
console.log(undefined)

a = 50
```

Output

```
undefined
```

---

# Function Execution Context

Whenever JavaScript encounters

```javascript
hello();
```

It creates a new execution context.

Example

```javascript
function hello(){

var name = "John";

console.log(name);

}

hello();
```

Memory

```
name → undefined
```

Execution

```
name = John

console.log(name)
```

Output

```
John
```

After completion

Execution Context is removed.

---

# Execution Context Stack

Example

```javascript
function one(){

two();

}

function two(){

three();

}

function three(){

console.log("Done");

}

one();
```

Stack

```
Call Stack

------------

Global

------------

one()

------------

two()

------------

three()

------------
```

After completion

```
three removed

↓

two removed

↓

one removed

↓

Global remains
```

---

# Detailed Execution Flow

Example

```javascript
var x = 5;

function square(n){

return n*n;

}

var result = square(x);

console.log(result);
```

Step 1

Create Global Context

↓

Step 2

Memory Phase

```
x → undefined

square → function

result → undefined
```

↓

Execution

```
x = 5

call square()
```

↓

Create Function Context

Memory

```
n → 5
```

↓

Execute

```
return 25
```

↓

Destroy Function Context

↓

Global

```
result = 25
```

↓

Output

```
25
```

---

# Global Object

In Browser

```
window
```

In Node.js

```
global
```

Example

```javascript
var x = 10;

console.log(window.x);
```

Output

```
10
```

---

# this Keyword

Inside Global Context

Browser

```
this === window
```

Node.js (CommonJS)

```
this === module.exports
```

Inside Function

Depends on how function is called.

Later chapters cover this in detail.

---

# Lexical Environment

Every execution context stores

Current Scope

+

Reference to Parent Scope

Example

```javascript
var a = 10;

function one(){

var b = 20;

function two(){

console.log(a,b);

}

two();

}

one();
```

Lexical Chain

```
two()

↓

one()

↓

Global

↓

null
```

JavaScript searches

```
two

↓

one

↓

Global
```

until variable is found.

---

# Variable Environment

Stores

- Variables
- Function declarations

Example

```javascript
var x = 5;

var y = 10;
```

Variable Environment

```
x → 5

y →10
```

---

# Common Mistakes

❌ Assuming JavaScript executes immediately.

❌ Thinking variables are created when encountered.

❌ Forgetting Memory Creation Phase.

❌ Confusing Execution Context with Call Stack.

❌ Ignoring Lexical Environment.

---

# Interview Questions

### Beginner

What is Execution Context?

Difference between Global and Function Execution Context?

Why is Memory Creation Phase needed?

What happens before JavaScript starts executing?

---

### Intermediate

Explain both phases of Execution Context.

How does JavaScript allocate memory?

Why are functions hoisted completely?

What is Lexical Environment?

---

### Advanced

Explain the execution of this code.

```javascript
var a = 10;

function test(){

console.log(a);

var a = 20;

}

test();
```

Answer

Memory

```
a → undefined
```

Execution

```
console.log(undefined)

a = 20
```

Output

```
undefined
```

---

# Senior Developer Notes

Execution Context is the foundation of

- Hoisting
- Closures
- Scope
- Event Loop
- Async/Await
- React Rendering
- Node.js Event Loop

Mastering this topic makes all advanced JavaScript concepts easier.

---

# Cheat Sheet

| Topic | Summary |
|-------|---------|
| Execution Context | Environment where JS executes |
| Types | Global, Function, Eval |
| Creation Phase | Memory allocation |
| Execution Phase | Executes code |
| var | undefined during creation |
| let | TDZ |
| const | TDZ |
| Function | Stored completely |
| Call Stack | Manages execution contexts |
| Lexical Environment | Scope chain |
| Variable Environment | Variables + Functions |

---

# Key Takeaways

✅ Every JavaScript program starts by creating a Global Execution Context.

✅ JavaScript first allocates memory, then executes code.

✅ Every function call creates a new Function Execution Context.

✅ Execution contexts are managed using the Call Stack (LIFO).

✅ Understanding Execution Context is essential before learning Hoisting, Closures, Event Loop, and Async/Await.
