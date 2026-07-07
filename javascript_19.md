# Chapter 6 - Asynchronous JavaScript & Event Loop
# Part 6 - Node.js Event Loop & libuv Internals

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 7-8 Hours

---

# Table of Contents

1. Browser vs Node.js Runtime
2. Why Node.js is Single Threaded
3. What is libuv?
4. Node.js Architecture
5. V8 Engine vs libuv
6. Thread Pool
7. Event Loop Phases Overview
8. Timers Phase
9. Pending Callbacks Phase
10. Idle & Prepare Phase
11. Poll Phase
12. Check Phase
13. Close Callbacks
14. Complete Event Loop Cycle
15. Interview Questions
16. Cheat Sheet

---

# 1. Browser vs Node.js Runtime

Most developers think JavaScript behaves exactly the same everywhere.

Wrong.

The JavaScript language is the same, but the runtime is different.

### Browser Runtime

```
Browser

↓

V8 Engine

+

DOM

+

Web APIs

+

Rendering Engine
```

Browser provides

- DOM
- Window
- Document
- Fetch
- Timers
- LocalStorage

---

### Node.js Runtime

```
Node.js

↓

V8 Engine

+

libuv

+

Operating System

+

File System

+

Network Stack
```

Node.js provides

- File System
- TCP
- HTTP
- DNS
- Process
- Streams
- Buffers

Notice

There is **no DOM** in Node.js.

---

# 2. Why Node.js is Single Threaded

Interview Question

Is Node.js single-threaded?

Answer

```
YES

and

NO
```

Explanation

JavaScript code executes on **one main thread**.

```
JavaScript

↓

One Thread
```

But Node.js runtime has additional threads.

```
Main Thread

↓

libuv Thread Pool

↓

Operating System
```

So,

JavaScript execution is single-threaded.

Node.js runtime is **multi-threaded internally**.

---

# Example

```javascript
console.log("Start");

fs.readFile("file.txt", ()=>{

    console.log("Done");

});

console.log("End");
```

Output

```
Start

End

Done
```

Why?

Reading the file is not performed by JavaScript.

It is delegated to libuv.

---

# 3. What is libuv?

libuv is a C library used by Node.js.

Responsibilities

- Event Loop
- Thread Pool
- File System
- DNS
- TCP
- Pipes
- Child Processes

Think of libuv as the operating system bridge.

```
JavaScript

↓

V8

↓

libuv

↓

Operating System
```

---

# Why Was libuv Created?

Without libuv,

JavaScript would block while waiting for I/O.

Example

```
Read File

↓

Wait

↓

Application Stops
```

With libuv

```
Read File

↓

Delegate

↓

Continue Processing

↓

Callback Later
```

---

# 4. Node.js Architecture

```
                     Node.js Runtime

-----------------------------------------------------

             JavaScript Application

                     │

                     ▼

                V8 Engine

                     │

                     ▼

              Node.js APIs

        fs   net   http   crypto

                     │

                     ▼

                  libuv

        ┌──────────────────────────┐
        │                          │
        │ Event Loop               │
        │ Thread Pool              │
        │ OS Event Notification    │
        └──────────────────────────┘

                     │

                     ▼

              Operating System
```

---

# 5. V8 vs libuv

Many people confuse them.

### V8

Responsible for

```
Execute JavaScript

↓

Compile

↓

Garbage Collection

↓

Memory
```

---

### libuv

Responsible for

```
Timers

↓

I/O

↓

Threads

↓

Networking

↓

Event Loop
```

---

Comparison

| V8 | libuv |
|----|--------|
| Executes JS | Handles async I/O |
| Call Stack | Event Loop |
| Memory | Thread Pool |
| Compiler | OS interaction |

---

# 6. Thread Pool

Question

If Node.js has one thread,

who reads files?

Answer

```
libuv Thread Pool
```

Default Size

```
4 Threads
```

Diagram

```
Thread Pool

-----------------------

Thread 1

Thread 2

Thread 3

Thread 4

-----------------------
```

---

# Which APIs Use Thread Pool?

```
File System

DNS Lookup

Compression

Crypto

PBKDF2

bcrypt

zlib
```

---

Which APIs do NOT use Thread Pool?

```
HTTP

TCP

UDP

WebSocket
```

These use OS asynchronous networking.

---

# Example

```javascript
fs.readFile("a.txt", callback);

fs.readFile("b.txt", callback);

fs.readFile("c.txt", callback);

fs.readFile("d.txt", callback);
```

All four can execute simultaneously.

---

What happens with five files?

```
Thread1

↓

File1

Thread2

↓

File2

Thread3

↓

File3

Thread4

↓

File4

Queue

↓

File5
```

---

# Increasing Thread Pool

```
UV_THREADPOOL_SIZE=8
```

Example

```bash
UV_THREADPOOL_SIZE=8 node app.js
```

Useful only for thread-pool based work.

Not useful for HTTP requests.

---

# 7. Event Loop Phases

Node.js Event Loop

```
        ┌────────────────────┐
        │ Timers             │
        └─────────┬──────────┘
                  │
                  ▼
        ┌────────────────────┐
        │ Pending Callbacks  │
        └─────────┬──────────┘
                  │
                  ▼
        ┌────────────────────┐
        │ Idle / Prepare     │
        └─────────┬──────────┘
                  │
                  ▼
        ┌────────────────────┐
        │ Poll               │
        └─────────┬──────────┘
                  │
                  ▼
        ┌────────────────────┐
        │ Check              │
        └─────────┬──────────┘
                  │
                  ▼
        ┌────────────────────┐
        │ Close Callbacks    │
        └────────────────────┘
```

Each phase has its own queue.

---

# 8. Timers Phase

Executes

```
setTimeout()

setInterval()
```

Example

```javascript
setTimeout(()=>{

    console.log("Timer");

},100);
```

Important

100ms means

```
Minimum Delay

NOT

Exact Execution Time
```

---

# 9. Pending Callbacks

Handles deferred system callbacks.

Examples

- TCP errors
- Network failures
- Certain I/O callbacks

Normally developers don't interact directly with this phase.

---

# 10. Idle / Prepare

Internal phase.

Used by libuv.

Interview Tip

You don't usually write code that directly targets this phase.

---

# 11. Poll Phase

This is the **heart of the Event Loop**.

Responsibilities

- Wait for I/O
- Execute I/O callbacks
- Decide whether to block or continue

Example

```javascript
fs.readFile("file.txt", ()=>{

    console.log("File Read");

});
```

Callback executes in the Poll phase.

---

# Poll Phase Decision

```
Any I/O Ready?

↓

YES

↓

Execute Callback

↓

Queue Empty?

↓

YES

↓

Any Timers Due?

↓

YES

↓

Go Timers

↓

NO

↓

Wait For New I/O
```

---

# 12. Check Phase

Executes

```
setImmediate()
```

Example

```javascript
setImmediate(()=>{

console.log("Immediate");

});
```

Runs after Poll.

---

# 13. Close Callbacks

Runs close event handlers.

Example

```javascript
socket.on("close", ()=>{

});
```

Examples

- Socket close
- Stream close
- Handle cleanup

---

# Complete Event Loop Cycle

```
Timers

↓

Pending Callbacks

↓

Idle

↓

Poll

↓

Check

↓

Close

↓

Repeat
```

---

# Real Example

```javascript
console.log("Start");

fs.readFile("test.txt", ()=>{

    console.log("File");

});

setTimeout(()=>{

    console.log("Timer");

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

Timer

Immediate

File
```

**Note:** The ordering between `setTimeout(0)` and `setImmediate()` can vary depending on whether they are scheduled from the main module or inside an I/O callback. We'll explore this in the next part.

---

# Common Misconceptions

❌ Node.js creates one thread per request.

False.

One thread handles JavaScript.

I/O is delegated.

---

❌ HTTP requests use the thread pool.

False.

Networking uses asynchronous OS facilities (epoll, kqueue, IOCP, etc.), not the libuv thread pool.

---

❌ Every async API uses threads.

False.

Only some APIs use the thread pool.

---

# Best Practices

✅ Avoid CPU-heavy work on the main thread.

✅ Use asynchronous APIs whenever possible.

✅ Increase `UV_THREADPOOL_SIZE` only when thread-pool tasks become a bottleneck.

✅ Use Worker Threads for CPU-intensive algorithms.

---

# Senior Interview Questions

## Beginner

- What is libuv?
- Why is Node.js considered single-threaded?

---

## Intermediate

- Difference between V8 and libuv?
- Which APIs use the thread pool?
- What is the Poll phase?

---

## Advanced

- Why can Node.js handle 100,000 concurrent HTTP requests?
- Why doesn't reading a file block the Event Loop?
- Explain every Event Loop phase.
- What is `UV_THREADPOOL_SIZE`?
- Why doesn't increasing the thread pool improve HTTP throughput?

---

# Cheat Sheet

| Component | Responsibility |
|-----------|----------------|
| V8 | Executes JavaScript |
| libuv | Event Loop + Async I/O |
| Thread Pool | File system, crypto, DNS, compression |
| Timers | `setTimeout`, `setInterval` |
| Poll | I/O callbacks |
| Check | `setImmediate` |
| Close | Resource cleanup |

---

# Key Takeaways

✅ JavaScript runs on a **single main thread**, but Node.js uses **libuv** internally to perform asynchronous work.

✅ **V8** executes JavaScript, while **libuv** manages the Event Loop, thread pool, and interaction with the operating system.

✅ The **libuv thread pool** (default size: 4) is used for operations like file system access, DNS lookups, cryptographic functions, and compression—not for HTTP networking.

✅ The Node.js Event Loop consists of multiple phases: **Timers → Pending Callbacks → Idle/Prepare → Poll → Check → Close Callbacks**.

✅ The **Poll phase** is the core of the Event Loop, where most I/O callbacks are processed.

✅ Understanding the separation between JavaScript execution, libuv, and the operating system is essential for writing scalable Node.js applications and answering senior-level interview questions.
