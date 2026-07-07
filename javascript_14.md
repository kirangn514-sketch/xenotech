# Chapter 6 - Asynchronous JavaScript & Event Loop (Part 1 - Foundations)

> Level: Intermediate → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 4 Hours

---

# Table of Contents

1. Why Asynchronous JavaScript?
2. Synchronous vs Asynchronous Programming
3. Blocking vs Non-Blocking
4. JavaScript Runtime
5. Browser Architecture
6. Web APIs
7. Callback Mechanism
8. Callback Queue
9. Event Loop Introduction
10. Complete Execution Flow
11. Interview Questions
12. Cheat Sheet

---

# 1. Why Asynchronous JavaScript?

Imagine you're developing an e-commerce application.

A user clicks:

```
Place Order
```

Your backend performs:

- Validate JWT
- Save Order
- Update Inventory
- Send Email
- Send SMS
- Generate Invoice
- Notify Warehouse

If JavaScript waited for each operation to finish before doing the next one, the application would feel very slow.

Example

```
Validate User

↓

Wait...

↓

Save Order

↓

Wait...

↓

Send Email

↓

Wait...

↓

Generate PDF

↓

Wait...

↓

Response
```

Total Response Time

```
10 Seconds
```

Instead,

JavaScript starts long-running operations asynchronously.

```
Validate User

↓

Save Order

↓

Send Email (Background)

↓

Generate Invoice (Background)

↓

Return Response
```

Response Time

```
500ms
```

---

# Why JavaScript Needed Async Programming

JavaScript was originally created for browsers.

Imagine this code

```javascript
while(true){}
```

If JavaScript were purely synchronous,

The browser would freeze.

```
Browser

↓

Infinite Loop

↓

UI Frozen

↓

Can't Click Buttons

↓

Can't Scroll

↓

Crash
```

To prevent this,

Browsers introduced asynchronous APIs.

---

# 2. Synchronous Programming

Definition

Each statement executes one after another.

Next statement starts only after the previous one finishes.

Example

```javascript
console.log("A");

console.log("B");

console.log("C");
```

Execution

```
A

↓

B

↓

C
```

Output

```
A

B

C
```

---

# Call Stack

During synchronous execution

```
Call Stack

---------------

console.log(C)

---------------

console.log(B)

---------------

console.log(A)

---------------

Global

---------------
```

Each function finishes before the next starts.

---

# Characteristics

✅ Predictable

✅ Easy to understand

❌ Can block the application

---

# Example of Blocking

```javascript
console.log("Start");

for(let i=0;i<10000000000;i++){}

console.log("End");
```

Output

```
Start

(wait...)

End
```

Browser freezes.

Node.js stops accepting requests.

---

# 3. Blocking vs Non-Blocking

Blocking means

```
One task

↓

Everything waits
```

Example

```
Read File

↓

Wait

↓

Database Query

↓

Wait

↓

Response
```

---

# Non-Blocking

```
Read File

↓

Background

↓

Continue

↓

Database

↓

Background

↓

Continue
```

Application remains responsive.

---

# Real World Analogy

Imagine a restaurant.

### Blocking

```
One Chef

↓

Cook Order 1

↓

Customer 2 Waits

↓

Customer 3 Waits

↓

Customer 4 Waits
```

---

### Non-Blocking

```
Chef Starts Cooking

↓

Moves to Next Customer

↓

Receives New Orders

↓

Collects Finished Food Later
```

Much faster.

---

# Important Point

JavaScript itself is **NOT asynchronous**.

This is one of the biggest interview questions.

JavaScript is

```
Single Threaded

+

Synchronous Language
```

Then how is async possible?

Answer

```
Browser APIs

OR

Node.js Runtime
```

---

# 4. JavaScript Runtime

JavaScript Engine alone cannot perform

- HTTP Requests
- Timers
- DOM Events
- File System
- Database Queries

The Runtime provides these features.

Browser Runtime

```
JavaScript Engine

+

Web APIs

+

Event Loop

+

Queues
```

Node.js Runtime

```
V8 Engine

+

libuv

+

OS

+

Thread Pool
```

---

# Browser Runtime Architecture

```
                Browser
-------------------------------------------------

        JavaScript Engine (V8)

                │

                ▼

          Call Stack

                │

                ▼

          Web APIs

      ┌───────────────┐
      │               │
      │ setTimeout()  │
      │ fetch()       │
      │ DOM Events    │
      │ Geolocation   │
      │ WebSocket     │
      └───────────────┘

                │

                ▼

          Callback Queue

                │

                ▼

           Event Loop
```

Notice

JavaScript Engine does **NOT** implement

```
setTimeout()

fetch()

DOM
```

Browser does.

---

# 5. Web APIs

Web APIs are provided by the browser.

Examples

```
setTimeout()

setInterval()

fetch()

DOM

localStorage

sessionStorage

navigator

geolocation

WebSocket
```

---

# Example

```javascript
setTimeout(()=>{

console.log("Hello");

},2000);
```

Question

Who waits for

```
2 seconds?
```

Answer

```
Browser Timer API
```

NOT JavaScript.

---

# Execution

```
Call Stack

↓

setTimeout()

↓

Browser Timer

↓

Call Stack Empty

↓

2 Seconds Complete

↓

Callback Queue
```

---

# 6. Callback Mechanism

Example

```javascript
console.log("Start");

setTimeout(()=>{

console.log("Timer");

},1000);

console.log("End");
```

Execution

Step 1

```
Global

↓

console.log(Start)

↓

Output

Start
```

---

Step 2

```
setTimeout()

↓

Browser Timer

↓

Removed from Call Stack
```

JavaScript does NOT wait.

---

Step 3

```
console.log(End)
```

Output

```
End
```

---

After 1 Second

Browser

```
Callback

↓

Callback Queue
```

---

# Output

```
Start

End

Timer
```

---

# Callback Queue

Callbacks from Web APIs enter the Callback Queue.

```
Callback Queue

-------------------

Timer Callback

-------------------

Click Event

-------------------

XHR Callback

-------------------
```

Callbacks wait here.

They cannot directly enter the Call Stack.

---

# 7. Event Loop Introduction

Question

How does a callback move from the queue to the Call Stack?

Answer

```
Event Loop
```

The Event Loop continuously checks

```
Is Call Stack Empty?
```

If YES

↓

Move first callback to the Call Stack.

---

# Event Loop Algorithm

```
Loop Forever

↓

Call Stack Empty?

↓

No

↓

Wait

-------------------

Yes

↓

Callback Queue Empty?

↓

No

↓

Move Callback

↓

Execute

↓

Repeat
```

---

# Example

```javascript
console.log("A");

setTimeout(()=>{

console.log("B");

},0);

console.log("C");
```

Many people expect

```
A

B

C
```

Wrong.

Execution

```
A

↓

setTimeout()

↓

Browser

↓

C

↓

Stack Empty

↓

B
```

Output

```
A

C

B
```

---

# Why setTimeout(0) Doesn't Execute Immediately

Because

```
0 ms

≠

Execute Immediately
```

It means

```
Minimum Delay

↓

Queue Callback

↓

Wait Until Stack Empty
```

---

# Complete Execution Diagram

```
Source Code

        │

        ▼

JavaScript Engine

        │

        ▼

Call Stack

        │

        ▼

Encounter setTimeout()

        │

        ▼

Browser Timer API

        │

        ▼

Timer Expires

        │

        ▼

Callback Queue

        │

        ▼

Event Loop

        │

        ▼

Call Stack Empty?

      │

 ┌────┴─────┐

 │          │

No         Yes

 │          │

 │          ▼

 │     Push Callback

 │          │

 └──────────┘

        │

        ▼

Execute Callback
```

---

# Common Misconceptions

### Wrong

JavaScript performs networking.

❌ Browser/Node Runtime performs networking.

---

### Wrong

setTimeout waits on the Call Stack.

❌ Timer runs in Browser APIs.

---

### Wrong

Event Loop executes callbacks immediately.

❌ Only when the Call Stack becomes empty.

---

# Best Practices

✅ Never block the main thread with heavy loops.

✅ Prefer asynchronous APIs.

✅ Keep callbacks lightweight.

✅ Offload CPU-intensive work to Workers (browser) or Worker Threads (Node.js).

---

# Senior Interview Questions

## Beginner

- What is synchronous programming?
- What is asynchronous programming?
- What is blocking code?

---

## Intermediate

- Is JavaScript asynchronous?
- Who implements `setTimeout()`?
- Why doesn't `setTimeout(0)` execute immediately?

---

## Advanced

Explain the output:

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

console.log("3");
```

Answer

```
1

3

2
```

Reason

The timer callback waits in the Callback Queue until the Call Stack is empty.

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Synchronous | Executes line by line |
| Asynchronous | Long-running work handled by runtime |
| Blocking | Prevents other work from executing |
| Non-Blocking | Allows other work to continue |
| JavaScript Engine | Executes JavaScript code |
| Browser Runtime | Provides Web APIs |
| Web APIs | `setTimeout`, `fetch`, DOM, etc. |
| Callback Queue | Stores completed async callbacks |
| Event Loop | Moves callbacks to the Call Stack when it is empty |

---

# Key Takeaways

✅ JavaScript is **single-threaded and synchronous** by design.

✅ Asynchronous behavior comes from the **runtime environment** (Browser or Node.js), not from the JavaScript language itself.

✅ Long-running operations such as timers and network requests are delegated to **Web APIs** (browser) or the **Node.js runtime**.

✅ Completed asynchronous callbacks are placed into the **Callback Queue**.

✅ The **Event Loop** continuously checks whether the **Call Stack** is empty. Only then does it move callbacks from the queue to the Call Stack.

✅ Understanding this architecture is the foundation for learning **Promises**, **async/await**, **Microtasks**, and the **Node.js Event Loop**.
