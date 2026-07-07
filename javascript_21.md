# Chapter 6 - Asynchronous JavaScript & Event Loop
# Part 8 - How Node.js Handles 100,000 Concurrent Requests

> Level: Expert
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 8 Hours

---

# Table of Contents

1. The Most Asked Interview Question
2. What Happens When a Request Arrives?
3. Complete Request Lifecycle
4. JavaScript Thread vs OS Threads
5. Why Node.js Can Handle Massive Concurrency
6. I/O Bound vs CPU Bound
7. Database Requests
8. File System Requests
9. HTTP Requests
10. Event Loop During Concurrent Requests
11. What Happens with 100,000 Requests?
12. Worker Threads
13. Cluster
14. PM2
15. Production Architecture
16. Best Practices
17. Interview Questions

---

# 1. The Most Asked Interview Question

Interviewer asks

```
Node.js is Single Threaded.

How can it serve

100,000 users simultaneously?
```

Many developers answer

```
Thread Pool
```

Wrong.

The correct answer is

```
Operating System

+

Non-Blocking I/O

+

Event Loop

+

libuv
```

The thread pool is only used for certain operations.

---

# 2. What Happens When a Request Arrives?

Suppose a client calls

```
GET /products
```

Architecture

```
Browser

↓

Internet

↓

Nginx

↓

Node.js

↓

Express

↓

Route Handler

↓

Database

↓

Response
```

---

Example

```javascript
app.get("/products", async(req,res)=>{

    const products = await db.query(
        "SELECT * FROM Products"
    );

    res.json(products);

});
```

Looks simple.

Internally, a lot happens.

---

# 3. Complete Request Lifecycle

```
Client

↓

TCP Connection

↓

Operating System

↓

Node.js HTTP Server

↓

Event Loop

↓

Express Middleware

↓

Route Handler

↓

Database Driver

↓

Database

↓

Database Responds

↓

Event Loop

↓

Response Sent
```

---

# Step-by-Step Execution

Request arrives

↓

OS receives packet

↓

Socket created

↓

Node.js notified

↓

JavaScript callback scheduled

↓

Express route executes

↓

Database query starts

↓

JavaScript thread becomes FREE

↓

Database works

↓

Database returns data

↓

Event Loop schedules callback

↓

JavaScript sends response

---

# Key Observation

During database execution

JavaScript is NOT waiting.

```
JavaScript

↓

Free

↓

Can Handle More Requests
```

This is the secret.

---

# 4. JavaScript Thread vs OS Threads

Many developers imagine

```
Request

↓

Thread

↓

Response
```

This is how Java works.

Node.js is different.

```
100000 Requests

↓

One JavaScript Thread

↓

OS Networking

↓

Callbacks

↓

Responses
```

---

# Visual Diagram

```
                Node.js

        JavaScript Thread

                │

      ┌─────────┴─────────┐

      ▼                   ▼

 Event Loop         libuv

      │

      ▼

Operating System

      │

      ▼

Thousands of Open Sockets
```

The operating system keeps track of sockets.

Node.js does not create one thread per request.

---

# 5. Why Node.js Can Handle Massive Concurrency

Suppose

```
1000 Users

↓

GET /products
```

Traditional Thread-per-Request Model

```
1000 Requests

↓

1000 Threads

↓

Huge Memory

↓

Context Switching

↓

Slow
```

---

Node.js

```
1000 Requests

↓

One JavaScript Thread

↓

1000 Open Sockets

↓

OS Handles Waiting

↓

Callbacks Execute Only When Ready
```

---

Memory Usage

Traditional

```
1000 Threads

×

1 MB

=

1000 MB
```

Node.js

```
1 Thread

+

Sockets

≈ Much Less Memory
```

---

# 6. I/O Bound vs CPU Bound

This is one of the most important concepts.

---

## I/O Bound

Examples

- Database Query
- HTTP Request
- Redis
- MongoDB
- PostgreSQL
- Reading File
- Upload File

Timeline

```
Start

↓

Waiting

↓

Response
```

CPU is mostly idle.

Perfect for Node.js.

---

## CPU Bound

Examples

- Image Processing
- Video Encoding
- PDF Generation
- Machine Learning
- Encryption
- Large Loops

Timeline

```
CPU

↓

Busy

↓

Busy

↓

Busy
```

The Event Loop cannot execute other callbacks while CPU work is running.

---

Example

```javascript
app.get("/hash",()=>{

    while(true){}

});
```

Every request blocks.

```
User A

↓

Infinite Loop

↓

User B Waits

↓

User C Waits

↓

Server Frozen
```

---

# 7. Database Requests

Example

```javascript
const users = await db.query(
    "SELECT * FROM Users"
);
```

Execution

```
JavaScript

↓

Database Driver

↓

Socket

↓

Operating System

↓

Database

↓

Waiting...

↓

Database Responds

↓

Event Loop

↓

Callback
```

JavaScript waits **logically**, but not **physically**.

The thread is free.

---

# Multiple Database Queries

```javascript
await Promise.all([

    getUsers(),

    getOrders(),

    getPayments()

]);
```

Execution

```
DB1

DB2

DB3

↓

Parallel Waiting

↓

Responses

↓

Event Loop
```

---

# 8. File System Requests

Example

```javascript
fs.readFile("users.json", callback);
```

Execution

```
JavaScript

↓

libuv Thread Pool

↓

Read File

↓

Complete

↓

Poll Phase

↓

Callback
```

Notice

File System uses

```
Thread Pool
```

not OS async networking.

---

# 9. HTTP Requests

Example

```javascript
fetch("https://api.example.com");
```

Execution

```
JavaScript

↓

Socket

↓

Operating System

↓

Waiting

↓

Network Response

↓

Event Loop

↓

Callback
```

No thread pool involved.

---

# 10. Event Loop During Concurrent Requests

Imagine

```
Request A

↓

Database

Waiting

----------------

Request B

↓

Redis

Waiting

----------------

Request C

↓

API Call

Waiting

----------------

Request D

↓

MongoDB

Waiting
```

JavaScript Thread

```
Free

↓

Free

↓

Free

↓

Callbacks

↓

Responses
```

This is why Node.js scales.

---

# 11. What Happens with 100,000 Requests?

Timeline

```
100000 Requests Arrive

↓

OS Accepts Connections

↓

Socket Created

↓

Node.js Receives Events

↓

JavaScript Executes Small Callback

↓

Starts Database Query

↓

Returns To Event Loop

↓

Next Request

↓

Repeat
```

Eventually

```
Database Responses Arrive

↓

Callbacks

↓

Responses Sent
```

JavaScript is busy only for tiny periods.

Most time is spent waiting.

---

# Visualization

```
JavaScript Thread

|Run|Wait|Run|Wait|Run|Wait|

Database

|-----Query------|

Network

|------Waiting------|

Operating System

|Socket Events|
```

Notice

The JavaScript thread is **not idle because of blocking**.

It is **available to process other events**.

---

# 12. Worker Threads

CPU-heavy work should not run on the Event Loop.

Example

```
Image Resize

↓

Worker Thread

↓

Main Thread Free
```

```javascript
const { Worker } = require("worker_threads");
```

Use for

- OCR
- AI inference
- Video processing
- PDF generation
- Data compression

---

# 13. Cluster

One Node.js process uses one CPU core.

Modern servers have

```
8

16

32

64

cores
```

Cluster creates

```
CPU Core 1

↓

Node Process

CPU Core 2

↓

Node Process

CPU Core 3

↓

Node Process
```

Each process has its own Event Loop.

---

# 14. PM2

PM2 manages multiple Node.js processes.

```
Internet

↓

Nginx

↓

PM2

↓

App 1

↓

App 2

↓

App 3

↓

App 4
```

Benefits

- Auto restart
- Load balancing
- Monitoring
- Zero downtime deployment

---

# 15. Production Architecture

```
                Internet

                    │

                    ▼

               Load Balancer

                    │

                    ▼

                 Nginx

         ┌──────────┴──────────┐

         ▼                     ▼

     PM2 Process 1        PM2 Process 2

         ▼                     ▼

     Express App         Express App

         │                     │

         └──────────┬──────────┘

                    ▼

               Redis Cache

                    │

                    ▼

              PostgreSQL Cluster

                    │

                    ▼

            Background Workers
```

---

# Best Practices

✅ Keep route handlers lightweight.

✅ Avoid blocking the Event Loop.

✅ Use asynchronous APIs.

✅ Use `Promise.all()` for independent I/O.

✅ Move CPU-intensive work to Worker Threads.

✅ Use Redis caching for frequently requested data.

✅ Use database connection pools.

✅ Scale across CPU cores with Cluster or PM2.

---

# Common Misconceptions

❌ Node.js creates one thread per request.

No.

The operating system manages thousands of sockets.

---

❌ Thread Pool handles HTTP requests.

No.

HTTP networking uses asynchronous OS networking APIs.

The thread pool is mainly for:

- File System
- DNS
- Crypto
- Compression

---

❌ await blocks Node.js.

No.

It only suspends the current async function.

The Event Loop continues processing other requests.

---

# Senior Interview Questions

## Beginner

- Why is Node.js good for APIs?
- What is I/O-bound work?
- What is CPU-bound work?

---

## Intermediate

- Explain how Node.js serves multiple requests with one thread.
- Why doesn't a database query block the Event Loop?
- Which operations use the thread pool?

---

## Advanced

- Explain the lifecycle of an HTTP request in Node.js.
- Why can Node.js handle 100,000 concurrent users?
- How would you optimize an API serving 50,000 RPS?
- When would you use Worker Threads?
- Cluster vs Worker Threads?
- How does PostgreSQL communicate with Node.js?
- How does Express process concurrent requests?

---

# Complete Flow Diagram

```
Client Request

↓

Operating System accepts socket

↓

Node.js HTTP Server

↓

Express Route

↓

Database Query Started

↓

JavaScript Thread Free

↓

Next Request

↓

Database Response

↓

Event Loop Callback

↓

Send HTTP Response
```

---

# Cheat Sheet

| Component | Responsibility |
|-----------|----------------|
| JavaScript Thread | Executes route handlers |
| Event Loop | Schedules callbacks |
| libuv | Async I/O + Event Loop |
| Operating System | Socket management |
| Thread Pool | File system, crypto, DNS |
| Worker Threads | CPU-intensive work |
| Cluster | Multi-core scaling |
| PM2 | Process management |

---

# Key Takeaways

✅ Node.js handles massive concurrency because it **does not dedicate one thread per request**.

✅ The operating system manages network sockets asynchronously, while Node.js executes **small JavaScript callbacks** only when work is ready.

✅ Most web applications are **I/O-bound**, spending most of their time waiting for databases, caches, or external APIs rather than using the CPU.

✅ During these waits, the JavaScript thread is **free to process other incoming requests**, allowing a single Node.js process to manage thousands of concurrent connections.

✅ CPU-intensive work should be moved to **Worker Threads**, while applications should scale across multiple CPU cores using **Cluster** or process managers like **PM2**.

✅ The key to scalable Node.js applications is keeping the Event Loop free from blocking operations and leveraging asynchronous I/O effectively.
