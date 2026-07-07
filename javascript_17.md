# Chapter 6 - Asynchronous JavaScript & Event Loop (Part 4 - async/await Internals)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 5-6 Hours

---

# Table of Contents

1. Why async/await was introduced
2. What is an async function?
3. What does async return?
4. What does await actually do?
5. Execution Flow of await
6. await vs Blocking
7. await and the Call Stack
8. async/await vs Promise.then()
9. Multiple await vs Promise.all()
10. Error Handling
11. Memory Diagram
12. Interview Questions
13. Cheat Sheet

---

# 1. Why async/await was Introduced

Promises solved Callback Hell.

But long Promise chains were still difficult to read.

Example

```javascript
login()

.then(getUser)

.then(getOrders)

.then(makePayment)

.then(sendEmail)

.then(generateInvoice)

.catch(console.error);
```

Works perfectly.

But deeply nested business logic becomes difficult.

ES2017 introduced

```
async

+

await
```

to make asynchronous code look synchronous.

---

# Promise Version

```javascript
fetch("/users")

.then(res => res.json())

.then(data => {

    console.log(data);

});
```

---

# async/await Version

```javascript
async function loadUsers(){

    const response = await fetch("/users");

    const data = await response.json();

    console.log(data);

}
```

Much easier to read.

---

# Important Interview Point

`async/await`

does NOT replace Promises.

It is built **on top of Promises**.

Internally

```
async

↓

Promise

↓

.then()

↓

Microtask Queue
```

---

# 2. What is an async Function?

Example

```javascript
async function greet(){

    return "Hello";

}
```

Question

What does it return?

Many developers answer

```
Hello
```

Wrong.

Output

```javascript
console.log(greet());
```

```
Promise { "Hello" }
```

An async function ALWAYS returns a Promise.

---

# Internal Transformation

JavaScript roughly transforms

```javascript
async function greet(){

    return "Hello";

}
```

into

```javascript
function greet(){

    return Promise.resolve("Hello");

}
```

These are almost equivalent.

---

# Returning Existing Promise

```javascript
async function load(){

    return fetch("/api");

}
```

Still returns

```
Promise
```

No extra wrapping needed.

---

# 3. await Keyword

Example

```javascript
const data = await fetch("/users");
```

Question

Does await stop JavaScript?

No.

It only pauses

```
Current async function
```

Everything else continues.

---

# Visual Flow

```
Async Function

↓

await

↓

Suspend Function

↓

Return Control

↓

JavaScript Continues

↓

Promise Resolves

↓

Resume Function
```

---

# Example

```javascript
async function demo(){

    console.log("A");

    await Promise.resolve();

    console.log("B");

}

console.log("Start");

demo();

console.log("End");
```

---

# Step-by-Step Execution

Output

```
Start

A

End

B
```

Why?

Step 1

```
Start
```

---

Step 2

```
demo()

↓

A
```

---

Step 3

```
await

↓

Suspend demo()

↓

Schedule continuation as Microtask
```

---

Step 4

```
End
```

---

Step 5

Call Stack Empty

↓

Microtask Queue

↓

Resume demo()

↓

B
```

---

# Execution Timeline

```
Call Stack

↓

demo()

↓

await

↓

Suspend

↓

Stack Empty

↓

Microtask Queue

↓

Resume

↓

Complete
```

---

# Important Rule

`await`

does NOT block

```
Entire JavaScript Thread
```

It only pauses

```
Current Async Function
```

---

# 4. await and the Call Stack

Example

```javascript
async function fetchUser(){

    console.log("1");

    await Promise.resolve();

    console.log("2");

}
```

During await

Call Stack

```
fetchUser()

↓

Global
```

After suspension

```
Global
```

The async function frame is removed.

Later

```
Microtask

↓

fetchUser()

↓

Resume
```

This is why JavaScript remains responsive.

---

# Memory Diagram

During suspension

```
Heap

----------------------------

Promise

↓

Continuation

↓

Remaining Code

↓

console.log("2")

----------------------------
```

The continuation is stored until the Promise settles.

---

# 5. await is NOT Blocking

Example

```javascript
async function first(){

    await Promise.resolve();

    console.log("First");

}

async function second(){

    console.log("Second");

}

first();

second();
```

Output

```
Second

First
```

Why?

```
first()

↓

Suspend

↓

second()

↓

Runs Immediately

↓

Resume first()
```

---

# Restaurant Analogy

Imagine

```
Chef Starts Soup

↓

Waits for Soup

↓

Instead of Standing Idle

↓

Starts Salad

↓

Soup Ready

↓

Finishes Soup
```

Exactly how await behaves.

---

# 6. async/await vs Promise.then()

These are functionally equivalent.

Using await

```javascript
const user = await getUser();
```

Equivalent to

```javascript
getUser()

.then(user => {

});
```

Internally

```
await

↓

.then()

↓

Continuation Function
```

---

# 7. Multiple await

Example

```javascript
const a = await fetchA();

const b = await fetchB();

const c = await fetchC();
```

Execution

```
A

↓

Wait

↓

B

↓

Wait

↓

C

↓

Wait
```

Total Time

```
1s

+

1s

+

1s

=

3 Seconds
```

---

# Better Approach

```javascript
const [a,b,c] = await Promise.all([

    fetchA(),

    fetchB(),

    fetchC()

]);
```

Execution

```
A

B

C

↓

Parallel

↓

1 Second
```

---

# Performance Comparison

Sequential

```
A

↓

B

↓

C

↓

3 sec
```

Parallel

```
A

B

C

↓

1 sec
```

This is one of the most asked Node.js interview questions.

---

# 8. Error Handling

Promise Style

```javascript
fetch()

.then()

.catch();
```

async/await Style

```javascript
try{

    const data = await fetch();

}

catch(err){

    console.log(err);

}
```

Internally

```
Rejected Promise

↓

Throw Exception

↓

catch()
```

---

# Multiple try/catch

```javascript
try{

    const user = await getUser();

    const orders = await getOrders();

}

catch(err){

    console.log(err);

}
```

One catch handles both Promise rejections.

---

# 9. Common Mistakes

### Mistake 1

Using await outside async

```javascript
const data = await fetch();
```

❌ SyntaxError

---

### Mistake 2

Sequential awaits

```javascript
await a();

await b();

await c();
```

When independent,

use Promise.all().

---

### Mistake 3

Ignoring rejected Promises

```javascript
await fetch();
```

without

```
try/catch
```

can terminate Node.js processes depending on configuration.

---

# 10. Internal Execution Diagram

```
async Function

↓

Returns Promise

↓

Execute Code

↓

Encounter await

↓

Pause Function

↓

Register Continuation

↓

Return Control

↓

Promise Resolves

↓

Microtask Queue

↓

Resume Function

↓

Return Final Value

↓

Resolve Outer Promise
```

---

# Advanced Interview Question

Explain output

```javascript
async function test(){

    console.log("1");

    await Promise.resolve();

    console.log("2");

}

console.log("3");

test();

console.log("4");
```

Execution

```
3

↓

1

↓

await

↓

Suspend

↓

4

↓

Microtask

↓

2
```

Output

```
3

1

4

2
```

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| async | Always returns a Promise |
| await | Pauses only the current async function |
| await Queue | Microtask Queue |
| Blocking | ❌ No |
| Promise.all | Parallel execution |
| try/catch | Handles Promise rejections |
| Continuation | Remaining async function after await |

---

# Key Takeaways

✅ `async` functions always return a Promise, even when returning a plain value.

✅ `await` pauses only the current async function—it never blocks the JavaScript thread.

✅ When `await` is encountered, the async function is suspended, its continuation is stored, and execution returns to the Event Loop.

✅ Once the awaited Promise settles, the continuation is scheduled as a **microtask**, ensuring it runs before the next macro task.

✅ `async/await` is syntactic sugar over Promises and `.then()`.

✅ Use `Promise.all()` for independent asynchronous operations to maximize concurrency and reduce total execution time.
