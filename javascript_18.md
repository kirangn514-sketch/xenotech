# Chapter 6 - Asynchronous JavaScript & Event Loop (Part 5 - Advanced Promise APIs & Concurrency)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 6 Hours

---

# Table of Contents

1. Promise.all()
2. Promise.allSettled()
3. Promise.race()
4. Promise.any()
5. AggregateError
6. Sequential vs Parallel Execution
7. Concurrency vs Parallelism
8. Limiting Concurrency
9. Retry Pattern
10. AbortController
11. Real-world Examples
12. Interview Questions
13. Cheat Sheet

---

# 1. Promise.all()

Promise.all() executes multiple promises concurrently and waits for all of them.

Example

```javascript
const p1 = Promise.resolve("A");
const p2 = Promise.resolve("B");
const p3 = Promise.resolve("C");

Promise.all([p1, p2, p3])
    .then(console.log);
```

Output

```
["A","B","C"]
```

---

# Internal Workflow

```
Promise.all()

↓

Start Every Promise

↓

Wait

↓

Did Every Promise Fulfill?

↓

YES

↓

Resolve Array

↓

NO

↓

Reject Immediately
```

---

# Important Rule

Execution is concurrent.

It does NOT wait like this:

```
P1

↓

P2

↓

P3
```

Instead

```
P1

P2

P3

↓

Running Together
```

---

# Example

```javascript
const delay = ms =>
    new Promise(resolve => setTimeout(resolve, ms));

Promise.all([
    delay(1000),
    delay(1000),
    delay(1000)
]);
```

Execution

```
Start

↓

1 Second

↓

Done
```

NOT

```
3 Seconds
```

---

# Fail Fast Behavior

```javascript
Promise.all([
    Promise.resolve(1),
    Promise.reject("Failed"),
    Promise.resolve(3)
])
.catch(console.log);
```

Output

```
Failed
```

Remaining promises continue running in the background, but the returned Promise is already rejected.

---

# Memory Diagram

```
Promise.all

↓

Pending Counter = 3

↓

P1 ✓

↓

Counter = 2

↓

P2 ✓

↓

Counter = 1

↓

P3 ✓

↓

Counter = 0

↓

Resolve
```

---

# 2. Promise.allSettled()

Unlike Promise.all(),

this never rejects.

Example

```javascript
Promise.allSettled([
    Promise.resolve("A"),
    Promise.reject("Error"),
    Promise.resolve("C")
])
.then(console.log);
```

Output

```javascript
[
 {status:"fulfilled", value:"A"},
 {status:"rejected", reason:"Error"},
 {status:"fulfilled", value:"C"}
]
```

---

# Internal Flow

```
Run All Promises

↓

Wait For Everyone

↓

Collect Results

↓

Resolve
```

Perfect for dashboards where partial failures are acceptable.

---

# 3. Promise.race()

Returns the first settled Promise.

Example

```javascript
Promise.race([
    fetch("/server1"),
    fetch("/server2"),
    fetch("/server3")
]);
```

Whoever finishes first wins.

```
Server1

↓

800ms

Winner
```

The others continue running unless cancelled.

---

# Timeout Example

```javascript
function timeout(ms){

    return new Promise((_, reject)=>{

        setTimeout(()=>reject("Timeout"), ms);

    });

}

Promise.race([
    fetch("/api"),
    timeout(3000)
]);
```

If fetch takes longer than 3 seconds,

```
Timeout
```

is returned.

---

# 4. Promise.any()

Returns the first fulfilled Promise.

Rejected promises are ignored.

Example

```javascript
Promise.any([
    Promise.reject(),
    Promise.reject(),
    Promise.resolve("Success")
])
.then(console.log);
```

Output

```
Success
```

---

# Difference

Promise.race()

```
First Settled
```

Promise.any()

```
First Fulfilled
```

Very important interview question.

---

# 5. AggregateError

If every Promise fails

```javascript
Promise.any([
    Promise.reject("A"),
    Promise.reject("B")
]);
```

Output

```
AggregateError
```

Contains

```
All Errors
```

---

# 6. Sequential vs Parallel

Sequential

```javascript
await fetchUser();
await fetchOrders();
await fetchProducts();
```

Timeline

```
1 sec

↓

1 sec

↓

1 sec

↓

3 sec
```

---

Parallel

```javascript
await Promise.all([
    fetchUser(),
    fetchOrders(),
    fetchProducts()
]);
```

Timeline

```
Start Together

↓

1 sec

↓

Done
```

---

# When NOT to Use Promise.all()

Example

```
Login

↓

Get JWT

↓

Use JWT

↓

Fetch Orders
```

Here,

Step 2 depends on Step 1.

Must remain sequential.

---

# 7. Concurrency vs Parallelism

This question is extremely common.

## Concurrency

Multiple tasks make progress during the same period.

One CPU can still be concurrent.

```
Task A

↓

Pause

↓

Task B

↓

Pause

↓

Task A
```

---

## Parallelism

Tasks truly execute simultaneously.

Requires multiple CPU cores or worker threads.

```
Core1

↓

Task A

----------------

Core2

↓

Task B
```

---

# JavaScript

JavaScript provides

```
Concurrency
```

using the Event Loop.

Parallelism requires

- Web Workers
- Worker Threads
- Cluster
- Child Processes

---

# 8. Limiting Concurrency

Suppose

```
10000

API Calls
```

Using Promise.all()

```
10000 Requests

↓

Database Crash
```

Better

```
10

At a Time
```

---

# Simple Promise Pool

```javascript
async function process(tasks, limit){

    let index = 0;

    async function worker(){

        while(index < tasks.length){

            const current = index++;

            await tasks[current]();

        }

    }

    await Promise.all(

        Array(limit)

        .fill()

        .map(worker)

    );

}
```

Now

```
10000 Jobs

↓

10 Workers

↓

Controlled Concurrency
```

---

# 9. Retry Pattern

Production systems retry transient failures.

Example

```javascript
async function retry(fn, attempts){

    while(attempts--){

        try{

            return await fn();

        }

        catch(err){

            if(attempts===0){

                throw err;

            }

        }

    }

}
```

---

# Exponential Backoff

Instead of

```
1 sec

1 sec

1 sec
```

Use

```
1 sec

↓

2 sec

↓

4 sec

↓

8 sec
```

Reduces server load.

---

# 10. AbortController

Cancel unnecessary requests.

```javascript
const controller = new AbortController();

fetch("/api",{

    signal: controller.signal

});

controller.abort();
```

Useful for

- React search
- Auto complete
- Route changes
- File uploads

---

# React Example

Without cancellation

```
Search A

↓

Search B

↓

A finishes last

↓

Old Data Appears
```

With AbortController

```
Search A

↓

Cancelled

↓

Search B

↓

Latest Data
```

---

# Node.js Microservice Example

```javascript
const [

    user,

    orders,

    payments

] = await Promise.all([

    getUser(),

    getOrders(),

    getPayments()

]);
```

Three independent services.

One network round-trip instead of three sequential waits.

---

# Next.js Example

```javascript
const [

    products,

    categories,

    offers

] = await Promise.all([

    fetchProducts(),

    fetchCategories(),

    fetchOffers()

]);
```

Much faster page rendering.

---

# Common Mistakes

❌ Using Promise.all() for dependent operations.

❌ Ignoring rejected promises.

❌ Launching thousands of requests simultaneously.

❌ Forgetting cancellation.

---

# Best Practices

✅ Use Promise.all() for independent tasks.

✅ Use Promise.allSettled() for dashboards.

✅ Use Promise.race() for timeouts.

✅ Use Promise.any() for fallback servers.

✅ Limit concurrency for heavy workloads.

✅ Retry transient failures.

✅ Cancel unused requests.

---

# Senior Interview Questions

## Beginner

- Difference between Promise.all() and Promise.allSettled()?
- Difference between Promise.race() and Promise.any()?

---

## Intermediate

- Why is Promise.all() faster?

- Explain fail-fast behavior.

- When should Promise.all() not be used?

---

## Advanced

- Design a Promise pool with limited concurrency.

- Implement retry with exponential backoff.

- How do you cancel an HTTP request?

- Explain concurrency vs parallelism.

- How would you call five microservices efficiently?

---

# Cheat Sheet

| API | Resolves When | Rejects When | Use Case |
|------|---------------|--------------|----------|
| Promise.all | All fulfilled | First rejection | Independent parallel tasks |
| Promise.allSettled | All settled | Never | Dashboard / batch reporting |
| Promise.race | First settled | First rejection if it settles first | Timeouts |
| Promise.any | First fulfilled | All rejected (`AggregateError`) | Fallback services |

---

# Concurrency Decision Tree

```
Need every result?

↓

YES

↓

Can one failure stop everything?

↓

YES

↓

Promise.all()

↓

NO

↓

Promise.allSettled()

----------------------------

Need fastest response?

↓

YES

↓

Any success works?

↓

YES

↓

Promise.any()

↓

NO

↓

Promise.race()
```

---

# Key Takeaways

✅ `Promise.all()` runs independent asynchronous operations concurrently and resolves only when all succeed.

✅ `Promise.allSettled()` waits for every Promise regardless of success or failure.

✅ `Promise.race()` settles as soon as the first Promise settles, while `Promise.any()` waits for the first successful result.

✅ JavaScript provides **concurrency**, not true parallel execution. Parallelism requires Worker Threads, Web Workers, or multiple processes.

✅ Launching too many asynchronous operations simultaneously can overwhelm downstream services. Use concurrency limits or Promise pools for production systems.

✅ Patterns such as retries, exponential backoff, and request cancellation are essential for building reliable, high-performance Node.js and React applications.
