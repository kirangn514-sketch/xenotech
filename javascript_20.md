# Chapter 6 - Asynchronous JavaScript & Event Loop
# Part 7 - process.nextTick(), setImmediate(), setTimeout(), Microtasks & Execution Order

> Level: Expert
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 7 Hours

---

# Table of Contents

1. Introduction
2. Execution Priority in Node.js
3. process.nextTick()
4. Promise Microtasks
5. queueMicrotask()
6. setTimeout()
7. setImmediate()
8. nextTick vs Promise
9. setTimeout vs setImmediate
10. Event Loop Starvation
11. Execution Order Examples
12. Real Production Examples
13. Best Practices
14. Interview Questions
15. Cheat Sheet

---

# 1. Introduction

In browsers we usually think about

```
Call Stack

↓

Microtasks

↓

Macrotasks
```

Node.js adds one extra queue.

```
process.nextTick()

↓

Promise Queue

↓

Event Loop
```

This makes Node.js behavior different from browsers.

---

# Complete Priority

Highest Priority

```
Current JavaScript

↓

process.nextTick()

↓

Promise Microtasks

↓

Timers

↓

Poll

↓

Check

↓

Close
```

This order is extremely important.

---

# Node.js Runtime

```
                    Node.js Runtime

-------------------------------------------------

            JavaScript Execution

                     │

                     ▼

              Call Stack

                     │

                     ▼

           process.nextTick Queue

                     │

                     ▼

             Promise Queue

                     │

                     ▼

              Event Loop

Timers

↓

Pending

↓

Poll

↓

Check

↓

Close
```

---

# 2. process.nextTick()

Interview Question

What is process.nextTick()?

Answer

It schedules a callback that executes

```
AFTER

Current JavaScript

BEFORE

Event Loop Continues
```

---

Example

```javascript
console.log("Start");

process.nextTick(()=>{

    console.log("Tick");

});

console.log("End");
```

Output

```
Start

End

Tick
```

---

Execution

```
Start

↓

nextTick()

↓

Queue Callback

↓

End

↓

Current Execution Ends

↓

Run nextTick Queue
```

Notice

Event Loop never started.

---

# Internal Flow

```
JavaScript

↓

Finished?

↓

YES

↓

nextTick Queue Empty?

↓

NO

↓

Execute ALL nextTick

↓

Continue Event Loop
```

---

# Why nextTick Exists

Node.js internally uses it for

- Error handling
- Streams
- Module initialization
- Internal scheduling

---

# 3. Promise Microtasks

Example

```javascript
console.log("A");

Promise.resolve()

.then(()=>{

    console.log("Promise");

});

console.log("B");
```

Output

```
A

B

Promise
```

---

Question

Who wins?

```
process.nextTick()

OR

Promise.then()
```

Answer

```
process.nextTick()
```

---

Example

```javascript
Promise.resolve()

.then(()=>{

    console.log("Promise");

});

process.nextTick(()=>{

    console.log("Tick");

});
```

Output

```
Tick

Promise
```

Priority

```
nextTick Queue

↓

Promise Queue
```

---

# 4. queueMicrotask()

Node.js also supports

```javascript
queueMicrotask()
```

Example

```javascript
queueMicrotask(()=>{

    console.log("Microtask");

});
```

It behaves similarly to Promise callbacks.

Priority

```
nextTick

↓

Promise

↓

queueMicrotask
```

(Practically, Promise callbacks and `queueMicrotask()` share the same microtask queue.)

---

# 5. setTimeout()

Schedules execution in the

```
Timers Phase
```

Example

```javascript
setTimeout(()=>{

    console.log("Timer");

},0);
```

Question

Does it execute immediately?

No.

It executes only after

- Current JS
- nextTick Queue
- Promise Queue
- Event Loop reaches Timers phase

---

# 6. setImmediate()

Schedules execution in

```
Check Phase
```

Example

```javascript
setImmediate(()=>{

    console.log("Immediate");

});
```

Execution

```
Poll

↓

Check

↓

setImmediate
```

---

# Important

Many developers think

```
setImmediate

=

setTimeout(0)
```

Wrong.

They belong to different Event Loop phases.

---

# 7. Execution Priority

```
Current JS

↓

process.nextTick()

↓

Promise.then()

↓

queueMicrotask()

↓

Timers

↓

Poll

↓

Check

↓

Close
```

---

# Example 1

```javascript
console.log("1");

process.nextTick(()=>{

    console.log("2");

});

Promise.resolve()

.then(()=>{

    console.log("3");

});

console.log("4");
```

Execution

```
1

↓

Queue nextTick

↓

Queue Promise

↓

4

↓

nextTick

↓

Promise
```

Output

```
1

4

2

3
```

---

# Example 2

```javascript
console.log("Start");

setTimeout(()=>{

    console.log("Timeout");

},0);

setImmediate(()=>{

    console.log("Immediate");

});

console.log("End");
```

Possible Output

```
Start

End

Timeout

Immediate
```

OR

```
Start

End

Immediate

Timeout
```

Why?

When executed from the main module,

ordering is NOT guaranteed.

---

# Inside I/O Callback

```javascript
const fs = require("fs");

fs.readFile(__filename, ()=>{

    setTimeout(()=>{

        console.log("Timeout");

    },0);

    setImmediate(()=>{

        console.log("Immediate");

    });

});
```

Output

```
Immediate

Timeout
```

Reason

```
Poll Phase

↓

Check Phase

↓

Immediate

↓

Next Loop

↓

Timers
```

This is a classic Node.js interview question.

---

# 8. Event Loop Starvation

Example

```javascript
function again(){

    process.nextTick(again);

}

again();
```

What happens?

```
nextTick

↓

Creates nextTick

↓

Creates nextTick

↓

Infinite
```

The Event Loop never reaches

```
Timers

Poll

Check
```

Application freezes.

This is called

```
Event Loop Starvation
```

---

Same with Promise

```javascript
function loop(){

    Promise.resolve()

    .then(loop);

}

loop();
```

Microtasks keep executing.

Timers never run.

---

# Visual Timeline

```
Current JS

↓

nextTick

↓

nextTick

↓

nextTick

↓

Promise

↓

Promise

↓

Timers

↓

Poll

↓

Check
```

If nextTick never ends,

everything below waits forever.

---

# Real Production Example

### Express Request

```javascript
app.get("/", async(req,res)=>{

    process.nextTick(()=>{

        console.log("Audit");

    });

    const user = await getUser();

    res.json(user);

});
```

Execution

```
Current Handler

↓

nextTick

↓

Promise

↓

Response
```

---

# Heavy CPU Work

Wrong

```javascript
app.get("/",()=>{

    while(true){}

});
```

Blocks the Event Loop.

All users wait.

---

Correct

```
Worker Thread

↓

CPU Work

↓

Main Thread Free
```

---

# Best Practices

✅ Use `process.nextTick()` only for internal scheduling.

✅ Prefer Promises for application logic.

✅ Use `setImmediate()` after I/O callbacks.

✅ Avoid recursive `nextTick()`.

✅ Avoid recursive microtasks.

---

# Senior Interview Questions

## Beginner

- What is `process.nextTick()`?
- Difference between `setImmediate()` and `setTimeout()`?

---

## Intermediate

- Why does `nextTick()` execute before Promise callbacks?
- Which Event Loop phase executes `setImmediate()`?

---

## Advanced

Explain the output

```javascript
console.log("A");

setTimeout(()=>{

    console.log("B");

},0);

setImmediate(()=>{

    console.log("C");

});

process.nextTick(()=>{

    console.log("D");

});

Promise.resolve()

.then(()=>{

    console.log("E");

});

console.log("F");
```

Execution

```
A

↓

Queue Timer

↓

Queue Immediate

↓

Queue nextTick

↓

Queue Promise

↓

F

↓

nextTick

↓

Promise

↓

Timers/Check
```

Possible Output

```
A

F

D

E

B

C
```

or

```
A

F

D

E

C

B
```

depending on whether the timer or immediate phase is reached first from the main module.

---

# Cheat Sheet

| API | Queue / Phase | Priority |
|------|---------------|----------|
| process.nextTick() | nextTick Queue | Highest |
| Promise.then() | Microtask Queue | High |
| queueMicrotask() | Microtask Queue | High |
| setTimeout() | Timers Phase | Medium |
| setImmediate() | Check Phase | Medium |
| I/O Callback | Poll Phase | Event Loop |

---

# Complete Execution Order

```
Current JavaScript

↓

process.nextTick()

↓

Promise.then()

↓

queueMicrotask()

↓

Timers Phase

↓

Pending Callbacks

↓

Poll Phase

↓

Check Phase

↓

Close Callbacks
```

---

# Browser vs Node.js

| Feature | Browser | Node.js |
|---------|----------|----------|
| Promise Queue | ✅ | ✅ |
| queueMicrotask() | ✅ | ✅ |
| process.nextTick() | ❌ | ✅ |
| setImmediate() | ❌ | ✅ |
| Event Loop | Browser | libuv |

---

# Key Takeaways

✅ `process.nextTick()` is unique to Node.js and has a higher priority than the Promise microtask queue.

✅ Promise callbacks (`.then()`, `.catch()`, `.finally()`) and `queueMicrotask()` are scheduled as microtasks and execute after the `nextTick` queue is drained.

✅ `setTimeout()` executes during the **Timers** phase, while `setImmediate()` executes during the **Check** phase.

✅ From the main module, the order between `setTimeout(0)` and `setImmediate()` is not guaranteed. Inside an I/O callback, `setImmediate()` typically executes first because the Event Loop moves directly from the **Poll** phase to the **Check** phase.

✅ Recursive use of `process.nextTick()` or microtasks can starve the Event Loop, preventing timers, I/O callbacks, and other phases from executing.

✅ Choosing the correct scheduling mechanism (`nextTick`, Promises, `setImmediate`, or `setTimeout`) is essential for writing efficient, predictable Node.js applications.
