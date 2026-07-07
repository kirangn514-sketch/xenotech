# Chapter 6 - Asynchronous JavaScript & Event Loop (Part 3 - Promises Internals)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 5-6 Hours

---

# Table of Contents

1. Why Promises Were Introduced
2. What is a Promise?
3. Promise States
4. Promise Lifecycle
5. Internal Structure of a Promise
6. Promise Executor
7. Resolve & Reject
8. Promise Jobs (Microtasks)
9. Promise Chaining
10. Error Propagation
11. Promise Combinators
12. Memory Diagram
13. Interview Questions
14. Cheat Sheet

---

# 1. Why Promises Were Introduced

Before Promises, JavaScript relied heavily on callbacks.

Example

```javascript
login(function(user){

    getOrders(user,function(orders){

        makePayment(orders,function(payment){

            sendEmail(payment,function(){

                console.log("Completed");

            });

        });

    });

});
```

This is known as

```
Callback Hell

↓

Nested Functions

↓

Poor Readability

↓

Difficult Error Handling
```

Problems

❌ Pyramid of Doom

❌ Error propagation is difficult

❌ Difficult to compose

❌ Hard to maintain

Promises solve these issues.

---

# 2. What is a Promise?

## Definition

A Promise is an object representing the eventual completion or failure of an asynchronous operation.

Think of it as a **placeholder for a future value**.

Example

```javascript
const promise = fetch("/api/products");
```

Immediately after creation

```
Promise

↓

Pending
```

Later

```
Server Response

↓

Promise

↓

Fulfilled
```

or

```
Network Error

↓

Promise

↓

Rejected
```

---

# Real Life Analogy

Imagine ordering food online.

```
Place Order

↓

Restaurant Accepts

↓

Food Preparing

↓

Promise = Pending

↓

Delivered

↓

Fulfilled

OR

Cancelled

↓

Rejected
```

---

# 3. Promise States

A Promise has only three states.

```
Pending

↓

Fulfilled

OR

Rejected
```

Important Rule

Once a Promise leaves the Pending state,

it can **never change again**.

```
Pending

↓

Fulfilled

×

Rejected
```

Impossible.

Likewise,

```
Pending

↓

Rejected

×

Fulfilled
```

Also impossible.

---

# Example

```javascript
const p = new Promise((resolve,reject)=>{

    resolve("Success");

});
```

State

```
Pending

↓

Fulfilled
```

---

# Rejected Example

```javascript
const p = new Promise((resolve,reject)=>{

    reject("Error");

});
```

State

```
Pending

↓

Rejected
```

---

# 4. Promise Lifecycle

```
Create Promise

↓

Pending

↓

Async Operation Starts

↓

Completed?

↓

Yes

↓

Success?

↓

Yes

↓

Fulfilled

↓

Run .then()

----------------

No

↓

Rejected

↓

Run .catch()
```

---

# 5. Internal Structure

Internally, a Promise stores information similar to:

```
Promise

-------------------------

[[PromiseState]]

Pending

-------------------------

[[PromiseResult]]

undefined

-------------------------

[[PromiseFulfillReactions]]

[]

-------------------------

[[PromiseRejectReactions]]

[]

-------------------------
```

These internal slots are not accessible directly.

---

# Example

```javascript
const p = new Promise(()=>{});
```

Internal View

```
State

↓

Pending

Result

↓

undefined

Fulfill Handlers

↓

[]

Reject Handlers

↓

[]
```

---

# 6. Promise Executor

Example

```javascript
const p = new Promise((resolve,reject)=>{

    console.log("Executor Running");

});
```

Output

```
Executor Running
```

Interview Question

Does the executor run asynchronously?

Answer

❌ No.

The executor runs **immediately and synchronously** when the Promise is created.

Execution

```
new Promise()

↓

Create Promise Object

↓

Run Executor Immediately

↓

Register Handlers

↓

Return Promise
```

---

# Example

```javascript
console.log("A");

new Promise(()=>{

    console.log("B");

});

console.log("C");
```

Output

```
A

B

C
```

Reason

Executor runs synchronously.

---

# 7. Resolve & Reject

Example

```javascript
const p = new Promise((resolve)=>{

    resolve("Done");

});
```

Question

Does `resolve()` immediately execute `.then()`?

Answer

❌ No.

`resolve()`

↓

Changes Promise State

↓

Queues Microtask

↓

`.then()` executes later

---

Execution

```
resolve()

↓

Promise State = Fulfilled

↓

Queue Microtask

↓

Stack Empty

↓

Run .then()
```

---

# Example

```javascript
console.log("1");

const p = Promise.resolve();

p.then(()=>{

    console.log("2");

});

console.log("3");
```

Output

```
1

3

2
```

---

# 8. Promise Jobs (Microtasks)

Every `.then()`

```
.then()

↓

Creates Promise Job

↓

Microtask Queue
```

Example

```javascript
Promise.resolve()

.then(()=>{

    console.log("A");

});
```

Memory

```
Promise

↓

Fulfilled

↓

Promise Job

↓

Microtask Queue
```

---

# 9. Promise Chaining

Example

```javascript
Promise.resolve(10)

.then((value)=>{

    return value+5;

})

.then((value)=>{

    return value*2;

})

.then(console.log);
```

Output

```
30
```

---

Execution

```
10

↓

15

↓

30
```

Each `.then()`

creates a **new Promise**.

Diagram

```
Promise A

↓

.then()

↓

Promise B

↓

.then()

↓

Promise C
```

---

# Value Propagation

```javascript
Promise.resolve(5)

.then(v=>v+5)

.then(v=>v*10)

.then(console.log);
```

Output

```
100
```

Pipeline

```
5

↓

10

↓

100
```

---

# 10. Error Propagation

Example

```javascript
Promise.resolve()

.then(()=>{

    throw new Error("Failure");

})

.catch(err=>{

    console.log(err.message);

});
```

Output

```
Failure
```

Flow

```
Promise

↓

then()

↓

Throw Error

↓

Reject Promise

↓

catch()
```

---

# Error Recovery

```javascript
Promise.reject("Oops")

.catch(()=>{

    return 100;

})

.then(console.log);
```

Output

```
100
```

Why?

`.catch()` returns a fulfilled Promise unless it throws again.

---

# 11. Promise Combinators

## Promise.all()

Waits for all Promises.

```javascript
Promise.all([p1,p2,p3]);
```

```
All Success

↓

Resolve

One Failure

↓

Reject Immediately
```

---

## Promise.allSettled()

Waits for all, regardless of success or failure.

```
Promise1

↓

Success

Promise2

↓

Failure

Promise3

↓

Success

↓

Return All Results
```

---

## Promise.race()

Returns the first settled Promise.

```
P1

↓

2 sec

P2

↓

500 ms

↓

Winner
```

---

## Promise.any()

Returns the first fulfilled Promise.

Ignores rejections until all fail.

---

# Comparison

| Method | Resolves When | Rejects When |
|----------|--------------|--------------|
| Promise.all | All succeed | First rejection |
| Promise.allSettled | All finish | Never rejects |
| Promise.race | First settles | First rejection if it settles first |
| Promise.any | First fulfillment | All reject |

---

# Memory Diagram

```
Promise

↓

State

↓

Fulfilled

↓

Result

↓

100

↓

.then()

↓

Microtask Queue
```

---

# Common Mistakes

❌ Thinking Promise executor is asynchronous.

❌ Thinking `resolve()` immediately executes `.then()`.

❌ Confusing Promise creation with Promise resolution.

❌ Forgetting every `.then()` returns a new Promise.

---

# Best Practices

✅ Always return values from `.then()` when chaining.

✅ Use `.catch()` once at the end of a chain.

✅ Prefer `Promise.all()` for independent parallel tasks.

✅ Use `Promise.allSettled()` when all results are needed.

---

# Senior Interview Questions

### Beginner

- What is a Promise?
- What are Promise states?

---

### Intermediate

- Does Promise executor run synchronously?
- Why does `.then()` run asynchronously?
- What queue is used by Promises?

---

### Advanced

Explain the output

```javascript
console.log("1");

new Promise(resolve=>{

    console.log("2");

    resolve();

})

.then(()=>{

    console.log("3");

});

console.log("4");
```

Output

```
1

2

4

3
```

Reason

- Promise executor runs synchronously.
- `resolve()` changes state immediately.
- `.then()` callback is scheduled as a microtask.
- Synchronous code completes before microtasks execute.

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Promise | Placeholder for a future value |
| States | Pending → Fulfilled / Rejected |
| Executor | Runs immediately (synchronously) |
| resolve() | Fulfills the Promise and schedules reactions |
| reject() | Rejects the Promise and schedules reactions |
| .then() | Registers fulfillment handler (microtask) |
| .catch() | Registers rejection handler |
| .finally() | Runs regardless of outcome |
| Promise.all | Parallel execution, fails fast |
| Promise.allSettled | Waits for all outcomes |
| Promise.race | First settled wins |
| Promise.any | First fulfilled wins |

---

# Key Takeaways

✅ A Promise is an object representing the future result of an asynchronous operation.

✅ A Promise has only three states: **Pending**, **Fulfilled**, and **Rejected**, and its state is immutable once settled.

✅ The Promise **executor runs synchronously** when the Promise is created.

✅ Calling `resolve()` or `reject()` settles the Promise immediately, but the corresponding `.then()`/`.catch()` handlers are scheduled as **microtasks**.

✅ Every call to `.then()`, `.catch()`, or `.finally()` returns a **new Promise**, enabling Promise chaining.

✅ Promise combinators (`all`, `allSettled`, `race`, `any`) provide different strategies for coordinating multiple asynchronous operations.
