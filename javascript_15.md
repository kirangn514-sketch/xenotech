# Chapter 6 - Asynchronous JavaScript & Event Loop (Part 2 - Event Loop Internals)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 5 Hours

---

# Table of Contents

1. Recap
2. Call Stack Internals
3. Task Queues
4. Macro Task Queue
5. Micro Task Queue
6. Event Loop Algorithm
7. Rendering Pipeline
8. Why Promise Executes Before setTimeout
9. queueMicrotask()
10. Microtask Starvation
11. Execution Timeline
12. Interview Questions
13. Cheat Sheet

---

# 1. Recap

JavaScript Runtime

```
                    Browser Runtime

--------------------------------------------------------

           JavaScript Engine (V8)

                   │

            Call Stack

                   │

        ┌──────────┴──────────┐

        ▼                     ▼

   Web APIs             Event Loop

        │

        ▼

   Task Queues

--------------------------------------------------------
```

In Part 1 we learned

- Call Stack
- Web APIs
- Callback Queue
- Event Loop

Now we'll understand how the Event Loop really works.

---

# 2. Call Stack Internals

The Call Stack is simply a stack data structure.

LIFO

```
Last In

↓

First Out
```

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

---

Execution

Step 1

```
Stack

------------

Global

------------
```

---

Step 2

```
one()

↓

Stack

------------

one()

------------

Global

------------
```

---

Step 3

```
two()

------------

two()

------------

one()

------------

Global
```

---

Step 4

```
three()

------------

three()

------------

two()

------------

one()

------------

Global
```

---

Step 5

```
console.log()

------------

console.log

------------

three()

------------

two()

------------

one()

------------

Global
```

---

After completion

```
console.log()

↓

Removed

↓

three()

↓

Removed

↓

two()

↓

Removed

↓

one()

↓

Removed
```

Finally

```
Global
```

---

# Important Rule

The Event Loop **cannot push another task** while the Call Stack is busy.

This explains why

```
setTimeout()

↓

waits
```

---

# 3. Task Queues

Most developers think there is only one queue.

Wrong.

Modern JavaScript has multiple queues.

```
Browser Runtime

↓

Task Queues

↓

Macro Task Queue

↓

Micro Task Queue

↓

Rendering Queue
```

---

# Priority

```
Highest

↓

Micro Tasks

↓

Rendering

↓

Macro Tasks
```

This priority is one of the most important interview topics.

---

# 4. Macro Task Queue

Also called

```
Task Queue

Callback Queue
```

Contains

```
setTimeout

setInterval

DOM Events

MessageChannel

postMessage

Network Events
```

Example

```javascript
setTimeout(()=>{

console.log("Timer");

},1000);
```

After timer completes

```
Macro Queue

↓

Timer Callback
```

---

# 5. Micro Task Queue

Highest priority queue.

Contains

```
Promise.then()

Promise.catch()

Promise.finally()

queueMicrotask()

MutationObserver
```

Example

```javascript
Promise.resolve()

.then(()=>{

console.log("Promise");

});
```

Execution

```
Micro Task Queue

↓

Promise Callback
```

---

# Why Two Queues?

Suppose

```
User clicks button

↓

Promise resolves

↓

Animation Frame

↓

Timer finishes
```

Which should execute first?

JavaScript uses priorities.

```
Micro Tasks

↓

Rendering

↓

Macro Tasks
```

---

# Event Loop Algorithm

This is the actual simplified algorithm.

```
Repeat Forever

↓

Execute Synchronous Code

↓

Call Stack Empty?

↓

YES

↓

Execute ALL Microtasks

↓

Perform Rendering

↓

Execute ONE Macro Task

↓

Repeat
```

Notice

Only ONE Macro Task executes per cycle.

All Microtasks execute before moving on.

---

# Example 1

```javascript
console.log("A");

Promise.resolve()

.then(()=>{

console.log("B");

});

console.log("C");
```

---

Execution

Step 1

```
A
```

---

Step 2

Promise callback

↓

Micro Task Queue

---

Step 3

```
C
```

---

Stack Empty

↓

Microtasks

↓

B

---

Output

```
A

C

B
```

---

# Example 2

```javascript
console.log("Start");

setTimeout(()=>{

console.log("Timer");

},0);

Promise.resolve()

.then(()=>{

console.log("Promise");

});

console.log("End");
```

---

Execution Timeline

```
Start

↓

Timer

↓

Macro Queue

↓

Promise

↓

Micro Queue

↓

End

↓

Stack Empty

↓

Micro Queue

↓

Promise

↓

Macro Queue

↓

Timer
```

---

Output

```
Start

End

Promise

Timer
```

This is one of the most common interview questions.

---

# Visual Timeline

```
Call Stack

↓

console.log(Start)

↓

setTimeout()

↓

Promise.then()

↓

console.log(End)

↓

Stack Empty

↓

Micro Queue

↓

Promise

↓

Rendering

↓

Macro Queue

↓

Timer
```

---

# 6. Rendering Pipeline

Browsers also render pages.

Rendering happens

```
AFTER

Micro Tasks

BEFORE

Macro Tasks
```

```
Call Stack

↓

Micro Tasks

↓

Paint Screen

↓

Macro Tasks
```

Reason

Keeps UI responsive.

---

# Example

```javascript
button.innerHTML="Loading";

Promise.resolve()

.then(()=>{

button.innerHTML="Done";

});
```

Browser waits until microtasks finish before painting.

---

# 7. queueMicrotask()

JavaScript provides

```javascript
queueMicrotask()
```

It directly schedules work into the Microtask Queue.

Example

```javascript
console.log("A");

queueMicrotask(()=>{

console.log("B");

});

console.log("C");
```

Output

```
A

C

B
```

Exactly like Promise.

---

# queueMicrotask vs Promise

```
queueMicrotask

↓

No Promise Creation

↓

Faster

-------------------

Promise.then

↓

Creates Promise

↓

Micro Task
```

---

# 8. Microtask Starvation

Dangerous interview topic.

Example

```javascript
function infinite(){

    queueMicrotask(infinite);

}

infinite();
```

What happens?

```
Microtask

↓

Creates Microtask

↓

Creates Microtask

↓

Creates Microtask
```

Macro Tasks never execute.

Rendering never occurs.

Browser freezes.

This is called

```
Microtask Starvation
```

---

# Example

```javascript
setTimeout(()=>{

console.log("Timer");

},0);

function again(){

    Promise.resolve()

    .then(again);

}

again();
```

Output

```
Nothing
```

Timer never runs.

Microtasks never finish.

---

# 9. Execution Timeline

Example

```javascript
console.log("1");

setTimeout(()=>{

console.log("2");

},0);

Promise.resolve()

.then(()=>{

console.log("3");

});

queueMicrotask(()=>{

console.log("4");

});

console.log("5");
```

---

Execution

```
1

↓

Timer

↓

Macro Queue

↓

Promise

↓

Micro Queue

↓

queueMicrotask

↓

Micro Queue

↓

5

↓

Stack Empty

↓

Micro Queue

↓

3

↓

4

↓

Render

↓

Macro Queue

↓

2
```

Output

```
1

5

3

4

2
```

---

# Common Mistakes

❌ Thinking `setTimeout(0)` executes immediately.

❌ Thinking there is only one callback queue.

❌ Thinking Promises execute on the Call Stack.

❌ Ignoring the Rendering step.

---

# Best Practices

✅ Use Promises for high-priority asynchronous work.

✅ Avoid recursive Microtasks.

✅ Keep Microtasks short.

✅ Don't block rendering.

---

# Senior Interview Questions

## Beginner

- What is the Call Stack?
- What is the Event Loop?

---

## Intermediate

- Difference between Macro Tasks and Micro Tasks.
- Why does Promise execute before setTimeout?

---

## Advanced

Explain the output

```javascript
console.log("1");

setTimeout(()=>console.log("2"),0);

Promise.resolve()

.then(()=>console.log("3"));

queueMicrotask(()=>console.log("4"));

console.log("5");
```

Output

```
1

5

3

4

2
```

Reason

- Synchronous code executes first.
- Promise callbacks and `queueMicrotask()` enter the Microtask Queue.
- The Event Loop drains all microtasks before rendering.
- Only after that does it execute one macro task (`setTimeout`).

---

# Cheat Sheet

| Queue | Priority | Examples |
|--------|----------|----------|
| Call Stack | Highest (currently executing) | Function calls |
| Micro Task Queue | High | `Promise.then`, `queueMicrotask`, `MutationObserver` |
| Rendering | Medium | Browser paint/layout |
| Macro Task Queue | Lower | `setTimeout`, `setInterval`, DOM events |

---

# Event Loop Priority

```
Synchronous Code

↓

Call Stack Empty?

↓

YES

↓

Execute ALL Microtasks

↓

Render UI

↓

Execute ONE Macro Task

↓

Repeat
```

---

# Key Takeaways

✅ The **Call Stack** executes synchronous JavaScript code.

✅ Modern JavaScript has multiple queues, not just a single callback queue.

✅ **Microtasks** (Promises, `queueMicrotask`) always have higher priority than **Macro Tasks** (`setTimeout`, DOM events).

✅ The Event Loop drains **all pending microtasks** before moving to rendering or macro tasks.

✅ `setTimeout(0)` does **not** mean immediate execution—it only guarantees that the callback is eligible to run after the current stack is empty and all microtasks are completed.

✅ Understanding queue priorities is essential for mastering **Promises**, **async/await**, **React rendering**, and **Node.js asynchronous execution**.
