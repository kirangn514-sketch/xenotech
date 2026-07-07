# Chapter 3 - Scope & Lexical Environment (Part 2)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 2-3 Hours

---

# Table of Contents

1. What is a Lexical Environment?
2. Components of a Lexical Environment
3. Environment Record
4. Outer Lexical Environment Reference
5. Global Lexical Environment
6. Function Lexical Environment
7. Block Lexical Environment
8. Variable Resolution Algorithm
9. Lexical Environment Creation Process
10. Execution Context vs Scope vs Lexical Environment
11. Complete Internal Flow
12. Interview Questions
13. Cheat Sheet

---

# 1. What is a Lexical Environment?

## Definition

A **Lexical Environment** is an internal data structure created by the JavaScript engine that stores:

- Variables
- Function declarations
- Parameters
- A reference to its parent (outer) environment

> A Lexical Environment is **not** something you write in JavaScript.
>
> It is created internally by the JavaScript engine.

---

## Simple Definition

Think of it as a dictionary.

```
Variable Name

↓

Value
```

Example

```javascript
let name = "Kiran";

const age = 25;

function greet(){}
```

Internally

```
Lexical Environment

------------------------

name  → "Kiran"

age   → 25

greet → Function

------------------------
```

---

# Real World Analogy

Imagine every office has:

- Employees
- Documents
- Manager's Phone Number

```
Office

↓

Employees

↓

Manager Contact
```

Similarly

```
Lexical Environment

↓

Variables

↓

Outer Environment Reference
```

---

# 2. Components of Lexical Environment

Every Lexical Environment has two parts.

```
Lexical Environment

+-------------------------------+

Environment Record

-------------------------------

Outer Lexical Environment

+-------------------------------+
```

---

# Environment Record

Stores everything declared in the current scope.

Example

```javascript
let city = "Mumbai";

const pin = 421201;

function print(){}
```

Environment Record

```
city

↓

"Mumbai"

----------------

pin

↓

421201

----------------

print

↓

Function Object
```

Think of it as a hash map.

```
Key

↓

Value
```

---

# Outer Lexical Environment Reference

This is the most important concept.

Every environment stores a reference to its parent environment.

```
Current Scope

↓

Parent Scope

↓

Parent Scope

↓

Global Scope
```

This is what enables the **Scope Chain**.

---

# Example

```javascript
let country = "India";

function outer(){

    let state = "Maharashtra";

    function inner(){

        console.log(country);

    }

}
```

Environment Structure

```
Global LE

country

↓

"India"

↓

outer()


↓

Outer = null

----------------------------

Outer LE

state

↓

"Maharashtra"

↓

inner()

↓

Outer = Global LE

----------------------------

Inner LE

(no variables)

↓

Outer = Outer LE
```

Notice

The inner environment knows where its parent is.

---

# 3. Global Lexical Environment

The very first Lexical Environment.

Created when the JavaScript program starts.

Example

```javascript
let app = "Food Delivery";

function login(){}
```

Global LE

```
Environment Record

--------------------

app

↓

"Food Delivery"

--------------------

login

↓

Function

--------------------

Outer

↓

null
```

Global has no parent.

---

# 4. Function Lexical Environment

Every function call creates a new Lexical Environment.

Example

```javascript
function add(a,b){

let total=a+b;

return total;

}
```

Calling

```javascript
add(10,20);
```

Creates

```
Function LE

Environment Record

-----------------

a

↓

10

-----------------

b

↓

20

-----------------

total

↓

30

-----------------

Outer

↓

Global LE
```

---

# Multiple Function Calls

```javascript
function test(x){

return x;

}

test(10);

test(20);
```

Each call creates a brand new environment.

```
Call 1

x

↓

10

---------------

Call 2

x

↓

20
```

The environments are independent.

---

# 5. Block Lexical Environment

Blocks using `let` and `const` also create Lexical Environments.

Example

```javascript
{
    let score = 100;
}
```

Environment

```
Block LE

score

↓

100

Outer

↓

Global
```

After block ends

Block LE is destroyed (unless captured by a closure).

---

# 6. Variable Resolution Algorithm

Suppose

```javascript
let company = "OpenAI";

function outer(){

    let department = "AI";

    function inner(){

        console.log(company);

    }

}
```

How does JavaScript find `company`?

Step 1

Search current environment.

```
Inner LE

↓

company ?

↓

No
```

Step 2

Move to parent.

```
Outer LE

↓

company ?

↓

No
```

Step 3

Move to global.

```
Global LE

↓

company ?

↓

Found
```

Output

```
OpenAI
```

---

# Variable Lookup Algorithm

```
Current Environment

↓

Found?

↓

Yes

↓

Return Value

↓

No

↓

Outer Environment

↓

Found?

↓

Yes

↓

Return

↓

No

↓

Global

↓

Found?

↓

No

↓

ReferenceError
```

---

# Example

```javascript
let a = 1;

function one(){

    let b = 2;

    function two(){

        let c = 3;

        console.log(a,b,c);

    }

    two();

}

one();
```

Search for

```
a

↓

two()

↓

No

↓

one()

↓

No

↓

Global

↓

Found
```

Search for

```
b

↓

two()

↓

No

↓

one()

↓

Found
```

Search for

```
c

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

# 7. Lexical Environment Creation Process

Whenever JavaScript enters a new scope

```
↓

Create Execution Context

↓

Create Lexical Environment

↓

Create Environment Record

↓

Store Variables

↓

Store Functions

↓

Link Parent Environment
```

---

Example

```javascript
function greet(){

    let msg="Hello";

}
```

Creation Phase

```
Execution Context

↓

Lexical Environment

↓

Environment Record

msg

↓

<uninitialized>

Outer

↓

Global
```

Execution

```
msg="Hello"
```

---

# 8. Execution Context vs Scope vs Lexical Environment

This is one of the biggest interview questions.

## Execution Context

Represents the runtime environment for executing code.

Contains

```
Variable Environment

Lexical Environment

this

```

Created whenever code starts executing.

---

## Scope

Represents where variables are accessible.

Example

```
Global

↓

Function

↓

Block
```

Scope is a **language concept**, not a runtime object.

---

## Lexical Environment

Internal runtime structure.

Stores

```
Variables

Functions

Outer Reference
```

---

# Relationship

```
Execution Context

│

├──Variable Environment

│

├──Lexical Environment

│      │

│      ├──Environment Record

│      └──Outer Reference

│

└──this
```

---

# Example

```javascript
let x=10;

function demo(){

let y=20;

}
```

Execution

```
Global Execution Context

↓

Global Lexical Environment

↓

Environment Record

↓

x

↓

10

↓

demo()

↓

Function Execution Context

↓

Function LE

↓

y

↓

20
```

---

# Complete Internal Diagram

```
Global Execution Context

│

├──Lexical Environment

│      │

│      ├──Environment Record

│      │      │

│      │      ├──a

│      │      ├──b

│      │      └──test()

│      │

│      └──Outer

│             │

│             ▼

│            null

│

└──this
```

Function Call

```
Function Execution Context

│

├──Lexical Environment

│      │

│      ├──Environment Record

│      │

│      ├──x

│      ├──y

│      └──return

│

└──Outer

↓

Global LE
```

---

# Common Mistakes

❌ Thinking Scope and Lexical Environment are the same.

❌ Thinking Lexical Environment is stored in the Heap.

❌ Thinking variables are searched using the Call Stack.

❌ Confusing Execution Context with Lexical Environment.

---

# Best Practices

- Keep scope as small as possible.
- Avoid unnecessary global variables.
- Prefer `const`.
- Avoid deep nesting when possible.
- Understand lexical scope before learning closures.

---

# Interview Questions

## Beginner

- What is a Lexical Environment?
- What does the Environment Record store?
- What is the Outer Environment Reference?

---

## Intermediate

- Explain variable lookup.
- Explain the Scope Chain.
- What is the difference between Scope and Lexical Environment?

---

## Advanced

Explain the internal process for:

```javascript
let a = 10;

function outer(){

    let b = 20;

    function inner(){

        console.log(a+b);

    }

    inner();

}

outer();
```

Expected explanation:

1. Global Execution Context is created.
2. Global Lexical Environment stores `a` and `outer`.
3. Calling `outer()` creates a new Execution Context.
4. A new Lexical Environment stores `b` and `inner`.
5. Calling `inner()` creates another Execution Context.
6. Variable lookup starts in `inner`, then moves to `outer`, then to `global`.
7. `a + b` evaluates to `30`.

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Lexical Environment | Internal runtime structure for variables and parent references |
| Environment Record | Stores variables, functions, parameters |
| Outer Reference | Points to parent lexical environment |
| Global LE | Created once when program starts |
| Function LE | Created on every function call |
| Block LE | Created for `let`/`const` blocks |
| Variable Lookup | Current → Parent → Global |
| Scope Chain | Chain of linked lexical environments |

---

# Key Takeaways

✅ Every scope has its own **Lexical Environment**.

✅ A Lexical Environment consists of an **Environment Record** and an **Outer Lexical Environment Reference**.

✅ Variable lookup follows the **Outer Reference** chain until the variable is found or a `ReferenceError` is thrown.

✅ **Scope** is a language rule, **Execution Context** is the runtime execution container, and the **Lexical Environment** is the internal data structure that makes lexical scoping possible.

✅ Closures work because JavaScript functions retain a reference to the **Lexical Environment** in which they were created—not where they are called.
