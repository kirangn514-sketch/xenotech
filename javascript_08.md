# Chapter 3 - Scope & Lexical Environment

> Level: Beginner → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 3-4 Hours

---

# Table of Contents

1. What is Scope?
2. Why Scope Exists
3. Types of Scope
4. Global Scope
5. Function Scope
6. Block Scope
7. Nested Scope
8. Lexical Scope
9. Variable Lookup
10. Scope Chain
11. Lexical Environment
12. Environment Record
13. Outer Environment Reference
14. Real Execution Flow
15. Interview Questions
16. Cheat Sheet

---

# 1. What is Scope?

## Definition

Scope defines **where a variable can be accessed** in your program.

Think of scope as the **visibility area** of a variable.

Example

```javascript
let name = "Kiran";

function greet() {
    console.log(name);
}

greet();
```

Output

```
Kiran
```

The function can access `name` because it is inside its scope chain.

---

# Real World Analogy

Imagine a company.

```
Company

│

├──CEO

│

├──HR

│

├──Engineering

│      │

│      ├──Frontend Team

│      │

│      └──Backend Team

│

└──Finance
```

Employees inside Frontend can access:

- Frontend resources
- Engineering resources
- Company resources

But HR employees cannot directly access Frontend private files.

JavaScript scope works exactly like this.

---

# Why Scope Exists?

Imagine every variable in every file were globally accessible.

Example

```javascript
let total = 100;
```

Another developer writes

```javascript
let total = 500;
```

Which value should JavaScript use?

Scope solves this problem by isolating variables.

Benefits:

- Prevents naming conflicts
- Improves security
- Makes code maintainable
- Reduces bugs

---

# JavaScript is Lexically Scoped

This is one of the most asked interview questions.

JavaScript determines scope **where the function is written**, **not where it is called**.

This is called **Lexical Scope** (or Static Scope).

We'll understand this in detail later.

---

# Types of Scope

JavaScript has four main scopes.

```
Global Scope

↓

Function Scope

↓

Block Scope

↓

Module Scope (ES Modules)
```

---

# 2. Global Scope

Variables declared outside all functions belong to the Global Scope.

Example

```javascript
let company = "OpenAI";

function showCompany() {
    console.log(company);
}

showCompany();
```

Output

```
OpenAI
```

Diagram

```
Global Scope

company

↓

"OpenAI"

↓

showCompany()
```

Everything inside the program can access global variables (unless shadowed).

---

# Browser vs Node.js Global Scope

### Browser

Global object:

```javascript
window
```

Example

```javascript
var city = "Mumbai";

console.log(window.city);
```

Output

```
Mumbai
```

### Node.js

Global object:

```javascript
global
```

Node.js modules have their own scope.

---

# Problem with Global Variables

Example

```javascript
let counter = 0;
```

Any part of the application can modify it.

```javascript
counter++;
counter = 100;
counter = -5;
```

Large applications become difficult to maintain.

Best Practice:

Minimize global variables.

---

# 3. Function Scope

Variables declared inside a function are only accessible inside that function.

Example

```javascript
function login() {
    let token = "abc123";

    console.log(token);
}

login();

console.log(token);
```

Output

```
abc123

ReferenceError
```

Diagram

```
Global Scope

↓

login()

↓

token
```

After the function finishes,

```
token

↓

Destroyed
```

---

# Function Scope Example

```javascript
function calculate() {
    let tax = 100;

    return tax;
}

calculate();

console.log(tax);
```

Output

```
ReferenceError
```

Reason:

`tax` only exists inside `calculate()`.

---

# 4. Block Scope

Introduced in ES6.

Variables declared using

- let
- const

are block scoped.

Example

```javascript
{
    let x = 10;
}

console.log(x);
```

Output

```
ReferenceError
```

---

# Block Scope Diagram

```
Global Scope

│

├──Block

│

│   x = 10

│

└──Outside Block
```

Outside the block,

`x` no longer exists.

---

# var is NOT Block Scoped

Example

```javascript
{
    var age = 25;
}

console.log(age);
```

Output

```
25
```

Diagram

```
Global Scope

age

↓

25
```

The block does not create a new scope for `var`.

---

# Comparison

```javascript
{
    var a = 1;
    let b = 2;
    const c = 3;
}

console.log(a);
console.log(b);
console.log(c);
```

Output

```
1

ReferenceError

ReferenceError
```

---

# 5. Nested Scope

Functions can be nested.

Example

```javascript
function outer() {

    let country = "India";

    function inner() {
        console.log(country);
    }

    inner();
}

outer();
```

Output

```
India
```

Diagram

```
Global

↓

outer()

↓

inner()
```

`inner()` can access variables from `outer()`.

---

# 6. Lexical Scope

This is one of the most important concepts.

Example

```javascript
let language = "JavaScript";

function outer() {

    function inner() {
        console.log(language);
    }

    inner();
}

outer();
```

Output

```
JavaScript
```

Question:

How did `inner()` find `language`?

Answer:

JavaScript searches the **Lexical Scope Chain**.

---

# Variable Lookup Process

Suppose JavaScript executes:

```javascript
console.log(language);
```

Search order:

```
Current Scope

↓

Parent Scope

↓

Global Scope

↓

Not Found

↓

ReferenceError
```

This search process is called the **Scope Chain**.

---

# Example

```javascript
let a = 1;

function one() {

    let b = 2;

    function two() {

        let c = 3;

        console.log(a, b, c);

    }

    two();
}

one();
```

Lookup Process

```
Find a

↓

two()

↓

Not Found

↓

one()

↓

Not Found

↓

Global

↓

Found

----------------

Find b

↓

two()

↓

Not Found

↓

one()

↓

Found

----------------

Find c

↓

two()

↓

Found
```

Output

```
1 2 3
```

---

# Scope Chain Diagram

```
Global Scope

│

├──a

│

└──one()

      │

      ├──b

      │

      └──two()

             │

             └──c
```

Search order:

```
two()

↓

one()

↓

Global

↓

null
```

---

# Common Mistakes

❌ Thinking scope depends on where a function is called.

❌ Using too many global variables.

❌ Assuming `var` is block scoped.

❌ Confusing scope with execution context.

---

# Best Practices

✅ Prefer `const` by default.

✅ Use `let` only when reassignment is required.

✅ Avoid `var` in modern code.

✅ Keep variable scope as small as possible.

✅ Avoid polluting the global scope.

---

# Interview Questions

### Beginner

1. What is scope?
2. Difference between global and local scope?
3. What is block scope?

### Intermediate

1. Difference between function scope and block scope?
2. Why is `var` not block scoped?
3. Explain lexical scope.

### Advanced

1. Explain the scope chain.
2. How does JavaScript perform variable lookup?
3. What is the relationship between lexical scope and closures?

---

# Cheat Sheet

| Concept | Summary |
|---------|---------|
| Scope | Visibility of variables |
| Global Scope | Accessible everywhere |
| Function Scope | Accessible only inside the function |
| Block Scope | Accessible only inside `{}` with `let`/`const` |
| Lexical Scope | Determined where the function is defined |
| Scope Chain | Variable lookup through parent scopes |
| Variable Lookup | Current → Parent → Global → ReferenceError |

---

# Key Takeaways

✅ Scope controls where variables are accessible.

✅ JavaScript uses **Lexical (Static) Scoping**, meaning scope is determined by where code is written, not where functions are called.

✅ There are four main scopes: Global, Function, Block, and Module.

✅ Variable lookup follows the **Scope Chain**: Current Scope → Parent Scope → Global Scope.

✅ `var` is function-scoped, while `let` and `const` are block-scoped.
