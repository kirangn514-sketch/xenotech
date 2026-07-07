# Chapter 1.5 – JavaScript Engine Internals (Part 4)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐

---

# Table of Contents

1. Introduction to Memory Management
2. Why Garbage Collection is Needed
3. Memory Allocation Process
4. Reachable vs Unreachable Objects
5. Garbage Collection Algorithms
6. Reference Counting
7. Mark and Sweep Algorithm
8. Generational Garbage Collection
9. New Space
10. Old Space
11. Minor GC
12. Major GC
13. Incremental Marking
14. Memory Leaks
15. Common Memory Leak Scenarios
16. WeakMap & WeakSet
17. Memory Optimization
18. Chrome DevTools Memory Profiling
19. Node.js Memory Debugging
20. Interview Questions
21. Cheat Sheet

---

# 1. Why Memory Management Exists

Every program creates data.

Example

```javascript
let user = {
    name: "John",
    age: 25
};
```

Memory is allocated.

```
Memory Heap

+----------------------+
| Object User          |
| name                 |
| age                  |
+----------------------+
```

Question

After the object is no longer needed,
who removes it?

Answer

The JavaScript Garbage Collector.

Without it,

Memory would continuously increase.

Eventually,

```
Out Of Memory
```

---

# Real World Analogy

Imagine renting hotel rooms.

Guest arrives

↓

Room allocated

↓

Guest leaves

↓

Room becomes available

If rooms are never cleaned,

Eventually,

```
HOTEL FULL
```

Garbage Collection works exactly the same way.

---

# 2. Memory Lifecycle

Every object follows four stages.

```
Create Object

↓

Allocate Memory

↓

Use Object

↓

Become Unreachable

↓

Garbage Collector Removes It
```

---

Example

```javascript
function createUser(){

    let user = {
        name:"Alice"
    };

}

createUser();
```

Flow

```
Function Starts

↓

Object Created

↓

Object Used

↓

Function Ends

↓

No Reference Exists

↓

GC Deletes Object
```

---

# 3. Memory Allocation

Primitive Values

Stored in Stack

```
let age = 25;
```

Objects

Stored in Heap

```
let person = {
   name:"John"
};
```

Heap

```
0x100

{
 name:"John"
}
```

---

# 4. Reachable Objects

This is the most important concept.

Garbage Collection depends on

**Reachability**

An object is kept alive only if it can still be reached.

Example

```javascript
let user = {
    name:"John"
};
```

```
Global Object

↓

user

↓

Heap Object
```

The object is reachable.

GC does NOT remove it.

---

# Unreachable Object

```javascript
let user = {
   name:"John"
};

user = null;
```

Memory

Before

```
user

↓

Object
```

After

```
user

↓

null


Object

(No Reference)
```

Now

No variable points to it.

It becomes unreachable.

GC removes it.

---

# Reachability Graph

```
Global

│

├──window

│

├──document

│

├──user

│      │

│      ▼

│   Object A

│

└──settings

       │

       ▼

   Object B
```

Every object reachable from the root is preserved.

---

# GC Root

Garbage Collection starts from

GC Roots.

Examples

Browser

```
window

document

DOM

Global Variables
```

Node.js

```
global

process

Module Scope
```

Everything reachable from these roots survives.

---

# 5. Reference Counting (Old Algorithm)

Old JavaScript engines used

Reference Counting.

Example

```
Object A

↑

user

↑

admin
```

Reference Count

```
2
```

When one variable disappears

```
Reference Count =1
```

When zero

```
Delete Object
```

---

Problem

Circular References.

Example

```javascript
let a = {};

let b = {};

a.ref = b;

b.ref = a;
```

```
A

↓

B

↓

A
```

Reference Count never reaches zero.

Memory leak.

Therefore,

Modern JavaScript engines no longer rely on this algorithm.

---

# 6. Mark and Sweep Algorithm

Modern V8 uses

Mark-and-Sweep.

This is one of the most important interview topics.

Two phases

```
Mark

↓

Sweep
```

---

# Mark Phase

Start from GC Root.

Visit every reachable object.

Mark it.

Diagram

```
window

│

├──user

│    │

│    ▼

│ Object A

│

├──cart

│    │

│    ▼

│ Object B

│

└──temp

     X

(Object C Not Reachable)
```

Marked

```
Object A

Object B
```

Not Marked

```
Object C
```

---

# Sweep Phase

Delete everything NOT marked.

Before

```
Heap

A

B

C

D
```

Marked

```
A

B
```

Sweep

```
Delete C

Delete D
```

Remaining

```
A

B
```

---

# Why Mark and Sweep is Better

Advantages

✔ Handles circular references

✔ Efficient

✔ Automatic

✔ Safe

---

# 7. Generational Garbage Collection

Observation

Most objects die quickly.

Example

```javascript
function login(){

let token={};

}
```

Object exists

Few milliseconds.

Deleted.

Only a few objects live long.

Example

```javascript
const appConfig={};
```

Lives entire application.

So

V8 divides memory.

```
Heap

↓

New Space

↓

Old Space
```

---

# 8. New Space

Stores

Short-lived objects.

Examples

Temporary arrays

Function variables

Temporary objects

```
Login Request

↓

Object Created

↓

Response Sent

↓

Deleted
```

---

# 9. Old Space

Stores

Long-lived objects.

Examples

Application Config

Cache

Singletons

Large Collections

---

# Promotion

Object survives several GCs

↓

Move

↓

Old Space

Diagram

```
New Space

↓

GC

↓

Still Alive

↓

GC Again

↓

Move

↓

Old Space
```

---

# 10. Minor GC

Works only on

New Space.

Very fast.

Runs frequently.

---

# 11. Major GC

Works on

Old Space.

Slower.

Runs less often.

Can pause application briefly.

---

# 12. Incremental Marking

Earlier

GC paused the whole application.

```
Stop Everything

↓

GC

↓

Resume
```

Bad User Experience.

Modern V8

Splits GC work.

```
Run Program

↓

Mark Few Objects

↓

Resume

↓

Mark Again

↓

Resume
```

Application remains responsive.

---

# 13. Memory Leak

Definition

Memory that is no longer useful

BUT

Cannot be collected.

---

Example 1

Global Variable

```javascript
let cache = [];
```

```
cache.push(largeObject);
```

Never removed.

Memory grows forever.

---

Example 2

Forgotten Timer

```javascript
setInterval(()=>{

console.log("running");

},1000);
```

Never cleared.

Timer

↓

Callback

↓

References

↓

Memory Leak.

---

Example 3

DOM Reference

```javascript
const button =
document.getElementById("btn");
```

Element removed

But

Reference still exists.

GC cannot remove it.

---

Example 4

Closure

```javascript
function outer(){

let hugeArray=new Array(1000000);

return function(){

console.log(hugeArray.length);

}

}
```

Closure still references

hugeArray.

Cannot be collected.

---

# 14. WeakMap

Normal Map

```
Map

↓

Reference

↓

Object
```

GC cannot remove object.

WeakMap

```
WeakMap

↓

Weak Reference

↓

Object
```

If object becomes unreachable,

GC removes it.

---

Example

```javascript
let user={};

const map=new WeakMap();

map.set(user,"Admin");

user=null;
```

Object removed automatically.

---

# 15. WeakSet

Works similarly.

Stores weak object references.

Useful for

Caching

DOM metadata

Visited objects

---

# 16. Chrome DevTools

Useful Tabs

```
Memory

↓

Heap Snapshot

↓

Allocation Timeline

↓

Garbage Collection
```

Can detect

Memory leaks

Detached DOM

Growing Objects

---

# 17. Node.js Memory Debugging

Useful APIs

```javascript
process.memoryUsage();
```

Output

```
rss

heapTotal

heapUsed

external
```

Useful for monitoring production servers.

---

# Example

```javascript
console.log(process.memoryUsage());
```

Output

```
{
 rss: 45MB,
 heapTotal:20MB,
 heapUsed:12MB
}
```

---

# Best Practices

✅ Remove unused event listeners.

✅ Clear intervals.

```javascript
clearInterval(id);
```

✅ Use WeakMap when appropriate.

✅ Avoid unnecessary global variables.

✅ Don't keep huge objects alive.

✅ Remove cache entries when unused.

---

# Senior Interview Questions

### Beginner

1. What is Garbage Collection?

2. Why is GC needed?

3. What is a Memory Leak?

---

### Intermediate

1. Explain Mark and Sweep.

2. What is Reachability?

3. Why was Reference Counting replaced?

---

### Senior

1. Explain Generational Garbage Collection.

2. Difference between Minor GC and Major GC?

3. Explain WeakMap.

4. Why do Closures sometimes cause memory leaks?

5. How would you debug a memory leak in Node.js?

6. How do you monitor heap usage in production?

---

# Cheat Sheet

| Topic | Summary |
|--------|---------|
| Garbage Collection | Automatically frees unused memory |
| Reachability | Objects reachable from GC roots survive |
| GC Roots | window, global, process, document |
| Mark Phase | Marks reachable objects |
| Sweep Phase | Removes unmarked objects |
| New Space | Short-lived objects |
| Old Space | Long-lived objects |
| Minor GC | Cleans New Space |
| Major GC | Cleans Old Space |
| WeakMap | Weak object references |
| WeakSet | Weak object collections |
| Memory Leak | Memory retained unnecessarily |

---

# Key Takeaways

✅ JavaScript automatically manages memory using **Garbage Collection**.

✅ V8 primarily uses the **Mark-and-Sweep** algorithm.

✅ Objects are retained only if they are **reachable** from GC roots.

✅ The heap is divided into **New Space** and **Old Space** to optimize collection.

✅ **Minor GC** is frequent and fast, while **Major GC** is slower and targets long-lived objects.

✅ Memory leaks often result from lingering references, global variables, timers, event listeners, caches, or closures.

✅ Tools like **Chrome DevTools** and **process.memoryUsage()** are essential for diagnosing memory issues in browsers and Node.js.
