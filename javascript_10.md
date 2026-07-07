# Chapter 4 - Closures (Part 1 - Foundation)

> Level: Intermediate → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 4-5 Hours

---

# Table of Contents

1. What is a Closure?
2. Why Closures Exist
3. Closure Definition
4. Internal Working
5. Step-by-Step Execution
6. Memory Diagram
7. Closure and Lexical Environment
8. Why Variables Don't Get Destroyed
9. Multiple Closures
10. Common Misconceptions
11. Interview Questions
12. Cheat Sheet

---

# 1. What is a Closure?

## Official Definition

A closure is a function together with the lexical environment in which it was created.

Or simply,

> A closure allows a function to remember variables from its outer scope even after the outer function has finished executing.

---

# Simple Example

```javascript
function outer() {

    let message = "Hello";

    function inner() {
        console.log(message);
    }

    return inner;
}

const fn = outer();

fn();
```

Output

```
Hello
```

Question:

How is `message` still available?

`outer()` already finished execution.

Normally,

```
outer()

↓

Destroyed
```

So why does `message` still exist?

Answer:

Because of **Closure**.

---

# Why Do Closures Exist?

Imagine JavaScript removed all local variables immediately after a function returned.

Example

```javascript
function counter() {

    let count = 0;

    return function () {
        count++;
        console.log(count);
    }

}
```

Without closures

```
counter()

↓

count destroyed

↓

Returned function

↓

count not found
```

Impossible.

Closures solve this problem.

---

# Internal Definition

When a function is created,

JavaScript stores:

```
Function Object

+

Reference to Lexical Environment
```

Notice

It stores a **reference**, not a copy.

---

# Visual Representation

```
Function Object

↓

Code

↓

Outer Lexical Environment
```

This hidden reference is what creates a Closure.

---

# Example

```javascript
function outer() {

    let x = 10;

    return function () {

        console.log(x);

    }

}
```

When the inner function is created

Internally

```
Inner Function

↓

Code

↓

Reference

↓

Outer Lexical Environment
```

---

# Important Interview Point

A Closure is **not** the variable.

A Closure is **not** the function.

A Closure is

```
Function

+

Reference

to

Lexical Environment
```

---

# Step-by-Step Execution

Example

```javascript
function outer() {

    let name = "Kiran";

    function inner() {

        console.log(name);

    }

    return inner;
}

const fn = outer();

fn();
```

---

## Step 1

Global Execution Context

```
Global LE

outer

↓

Function
```

---

## Step 2

Call

```javascript
outer();
```

Call Stack

```
outer()

↓

Global
```

---

## Step 3

Create Function Execution Context

```
Outer Execution Context

↓

Lexical Environment

↓

name

↓

"Kiran"
```

---

## Step 4

Create inner function

This is the important step.

Instead of only creating

```
Function Code
```

JavaScript creates

```
Function Object

↓

Code

↓

[[Environment]]

↓

Outer Lexical Environment
```

The hidden property

```
[[Environment]]
```

stores a reference to the outer lexical environment.

---

## Step 5

Return inner function

```javascript
return inner;
```

Now

```
outer()

↓

Finished
```

Normally

Everything inside should disappear.

BUT

The Lexical Environment is still referenced by

```
Returned Function

↓

[[Environment]]
```

Therefore,

JavaScript keeps it alive.

---

# Heap Memory

After `outer()` returns

Memory

```
Call Stack

Global


Heap

-------------------------

Function Object

↓

inner()

↓

[[Environment]]

↓

Outer LE

↓

name

↓

"Kiran"

-------------------------
```

Notice

The Lexical Environment moved to the Heap because something still references it.

---

# Function Call

Now

```javascript
fn();
```

Execution Context

```
fn()

↓

Global
```

Inside

```
console.log(name)
```

Variable Lookup

```
Current LE

↓

Not Found

↓

Outer LE

↓

Found

↓

"Kiran"
```

Output

```
Kiran
```

---

# Why Isn't Memory Freed?

Garbage Collector only removes unreachable objects.

Current situation

```
fn

↓

Function Object

↓

[[Environment]]

↓

Outer LE

↓

name
```

Everything is reachable.

Therefore,

GC does NOT delete it.

---

# Closure Memory Diagram

```
Global

↓

fn

↓

Function Object

↓

[[Environment]]

↓

Outer Lexical Environment

↓

name

↓

"Kiran"
```

---

# Multiple Closures

Example

```javascript
function outer() {

    let count = 0;

    return {
        increment() {
            count++;
        },

        get() {
            return count;
        }
    };
}

const counter = outer();

counter.increment();
counter.increment();

console.log(counter.get());
```

Output

```
2
```

Both methods share

```
Same Lexical Environment

↓

count
```

---

# Visualization

```
increment()

↓

count

↓

Shared

↓

get()
```

Both functions reference the same environment.

---

# Independent Closures

Example

```javascript
function createCounter() {

    let count = 0;

    return function () {

        count++;

        return count;

    }

}

const c1 = createCounter();

const c2 = createCounter();
```

Memory

```
c1

↓

Closure

↓

count

↓

0

------------------

c2

↓

Closure

↓

count

↓

0
```

Each call creates a new Lexical Environment.

---

# Execution

```javascript
c1();
```

Output

```
1
```

```javascript
c1();
```

Output

```
2
```

```javascript
c2();
```

Output

```
1
```

Independent memory.

---

# Common Misconceptions

### Wrong

"Closure copies variables."

❌ No.

Closures store references to Lexical Environments.

---

### Wrong

"Closures are memory leaks."

❌ No.

Closures only become memory leaks if they unnecessarily keep large objects alive.

---

### Wrong

"Closures only happen when returning functions."

❌ No.

Closures occur whenever an inner function accesses variables from an outer scope, even if it isn't returned.

Example

```javascript
function outer() {

    let x = 10;

    function inner() {
        console.log(x);
    }

    inner();
}
```

`inner()` still forms a closure.

---

# Common Interview Question

```javascript
function test() {

    let x = 5;

    return function () {
        return ++x;
    }

}

const fn = test();

console.log(fn());

console.log(fn());

console.log(fn());
```

Step-by-step

Initial

```
x = 5
```

Call 1

```
6
```

Call 2

```
7
```

Call 3

```
8
```

Output

```
6
7
8
```

Reason

The closure keeps the same Lexical Environment alive between function calls.

---

# Real-World Uses of Closures

Closures are used in:

- Data encapsulation (private variables)
- Function factories
- Currying
- Memoization
- Event handlers
- Timers (`setTimeout`, `setInterval`)
- React Hooks (`useState`, `useEffect`, `useCallback`, `useMemo`)
- Express middleware
- Node.js callbacks
- Redux middleware

We'll implement all of these in the next parts.

---

# Senior Interview Questions

## Beginner

- What is a closure?
- Why are closures needed?

---

## Intermediate

- Why doesn't the outer variable get garbage collected?
- Explain closures using Lexical Environments.

---

## Advanced

- Explain how V8 stores closures internally.
- Where is the Lexical Environment stored after the outer function returns?
- Are closures stored on the Stack or Heap?
- Can closures cause memory leaks?
- Explain closures with the Garbage Collector.

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Closure | Function + Lexical Environment |
| [[Environment]] | Hidden reference to outer lexical environment |
| Lexical Environment | Stores variables and outer reference |
| Variable Lookup | Current → Parent → Global |
| Garbage Collection | Won't collect reachable closure environments |
| Independent Closures | Each function call gets its own lexical environment |
| Shared Closures | Multiple inner functions can share one environment |

---

# Key Takeaways

✅ A closure is **not just a function**; it is a **function together with its lexical environment**.

✅ JavaScript functions internally hold a hidden `[[Environment]]` reference to the lexical environment where they were created.

✅ Closures keep outer variables alive even after the outer function has finished executing.

✅ The Garbage Collector does not free a lexical environment while it is still reachable through a closure.

✅ Every invocation of an outer function creates a **new lexical environment**, allowing multiple independent closures.

✅ Understanding closures requires understanding **execution contexts**, **lexical environments**, **heap memory**, and **garbage collection**—all of which you've already studied.
