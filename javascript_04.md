# Chapter 1.5 – JavaScript Engine Internals (Part 3)

> Level: Intermediate → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐

---

# Table of Contents

1. Runtime Memory Architecture
2. Memory Heap
3. Call Stack
4. Stack vs Heap
5. Primitive vs Reference Types
6. Pass by Value vs Pass by Reference
7. Function Call Execution
8. Execution Context Creation
9. Stack Overflow
10. Memory Allocation
11. Real Execution Flow
12. Best Practices
13. Interview Questions
14. Cheat Sheet

---

# 1. Runtime Memory Architecture

Before understanding memory, let's see how JavaScript Runtime looks internally.

```
                    JavaScript Runtime

+------------------------------------------------------+
|                  JavaScript Engine                   |
+------------------------------------------------------+

        +-------------------+   +------------------+
        |    Call Stack     |   |   Memory Heap    |
        +-------------------+   +------------------+
                 |                     |
                 |                     |
      Function Calls          Objects, Arrays,
      Execution Context       Functions, Closures

                 |
                 ▼
        Web APIs / Node APIs
                 |
                 ▼
          Event Loop & Queues
```

There are **two main memory areas**:

- Call Stack
- Memory Heap

---

# 2. What is the Call Stack?

The Call Stack is a memory structure used to manage:

- Function calls
- Execution Contexts
- Local variables
- Order of execution

It follows the **LIFO (Last In, First Out)** principle.

Think of a stack of plates.

```
      Plate 4  ← Removed First
      Plate 3
      Plate 2
      Plate 1  ← Added First
```

The last plate placed on the stack is removed first.

The Call Stack works exactly the same way.

---

# 3. What is Memory Heap?

The Memory Heap is a large region of memory used to store:

- Objects
- Arrays
- Functions
- Closures
- Classes
- Dates
- Maps
- Sets

Unlike the stack, the heap has **no fixed order**.

Think of it as a huge warehouse.

```
+---------------------------------------------------+
|                     Memory Heap                   |
|                                                   |
| Object A                                          |
| Array B                                           |
| Function C                                        |
| Object D                                          |
| Closure E                                         |
|                                                   |
+---------------------------------------------------+
```

---

# Why Two Different Memory Areas?

Suppose JavaScript stored everything in the Call Stack.

Imagine:

```
Large Image Object

↓

100 MB

↓

Stored in Stack
```

Every function call would become extremely slow.

Instead,

Only **references** are stored in the stack.

Actual data lives inside the heap.

---

# 4. Stack vs Heap

| Feature | Call Stack | Memory Heap |
|----------|------------|-------------|
| Stores | Execution Context | Objects |
| Size | Small | Large |
| Access Speed | Very Fast | Slower |
| Memory Management | Automatic | Garbage Collected |
| Structure | LIFO | Random Allocation |

---

# Real Example

```javascript
let age = 25;

let user = {
    name: "John"
};
```

Memory Layout

```
CALL STACK

age → 25

user → 0x101
          |
          |
          ▼

MEMORY HEAP

0x101

{
   name:"John"
}
```

Notice:

The object is **not** stored in the stack.

Only its memory address is.

---

# 5. Primitive Types

Primitive values are stored directly inside the Call Stack.

Examples

```javascript
let a = 10;

let b = true;

let c = "Hello";
```

Memory

```
CALL STACK

a → 10

b → true

c → Hello
```

Primitive Types

- Number
- String
- Boolean
- Null
- Undefined
- Symbol
- BigInt

---

# Why Store Primitives in Stack?

They are:

- Small
- Fixed Size
- Fast to Copy
- Fast to Access

---

# 6. Reference Types

Reference types are stored in the Heap.

Examples

```javascript
let person = {
    name: "John"
};

let colors = ["red","blue"];
```

Memory

```
STACK

person → 0x200

colors → 0x300


HEAP

0x200

{
 name:"John"
}

0x300

["red","blue"]
```

---

# Why Heap?

Objects can:

- Grow
- Shrink
- Add properties
- Remove properties

Their size is unknown beforehand.

---

# 7. Pass by Value

Example

```javascript
let a = 10;

let b = a;

b = 20;

console.log(a);
console.log(b);
```

Memory

Initially

```
STACK

a → 10

b → 10
```

After

```
b = 20
```

Memory

```
STACK

a → 10

b → 20
```

Output

```
10

20
```

Reason

Primitive values are copied.

---

# 8. Pass by Reference

Example

```javascript
let user1 = {
    name: "John"
};

let user2 = user1;

user2.name = "Mike";
```

Memory

Initially

```
STACK

user1 → 0x100

user2 → 0x100


HEAP

0x100

{
 name:"John"
}
```

After

```
user2.name="Mike"
```

Heap

```
0x100

{
 name:"Mike"
}
```

Output

```javascript
console.log(user1.name);
```

```
Mike
```

Because both variables point to the same object.

---

# 9. Function Execution

Example

```javascript
function greet(){

console.log("Hello");

}

greet();
```

Execution

```
Call Stack

-------------

Global

-------------

greet()

-------------
```

After completion

```
Call Stack

-------------

Global

-------------
```

The function context is removed.

---

# Example with Multiple Functions

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

Stack

```
three()

↓

two()

↓

one()

↓

Global
```

Execution

```
Done

↓

Remove three

↓

Remove two

↓

Remove one
```

---

# 10. Execution Context Creation

Every function call creates a new Execution Context.

Example

```javascript
function add(a,b){

return a+b;

}

add(10,20);
```

Stack

```
Global Context

↓

add() Context

↓

Return

↓

Destroy add()

↓

Global
```

---

# Inside Every Execution Context

```
Execution Context

+------------------------------+
| Variable Environment         |
+------------------------------+

+------------------------------+
| Lexical Environment          |
+------------------------------+

+------------------------------+
| this                         |
+------------------------------+
```

This is exactly what we studied in Chapter 1.

---

# 11. Memory Allocation Example

```javascript
let x = 10;

let user = {

name:"John"

};

function hello(){

return "Hi";

}
```

Stack

```
x → 10

user → 0x101

hello → 0x500
```

Heap

```
0x101

{
 name:"John"
}


0x500

Function Object
```

Functions are also objects.

Therefore they live in the Heap.

---

# 12. Stack Overflow

Consider

```javascript
function test(){

test();

}

test();
```

Execution

```
Global

↓

test()

↓

test()

↓

test()

↓

test()

↓

test()

↓

...
```

The Call Stack keeps growing.

Eventually

```
RangeError

Maximum Call Stack Size Exceeded
```

This is called **Stack Overflow**.

---

# Real World Analogy

Imagine stacking books forever.

```
Book

↓

Book

↓

Book

↓

Book

↓

Book

↓

Falls
```

The Call Stack behaves the same way.

---

# 13. Complete Runtime Example

```javascript
let person = {
    name: "Alice"
};

function update(obj){
    obj.name = "Bob";
}

update(person);

console.log(person.name);
```

Step 1

Stack

```
person → 0x500
```

Heap

```
0x500

{
 name:"Alice"
}
```

Step 2

`update(person)` creates a new execution context.

Stack

```
update()

obj → 0x500
```

Both `person` and `obj` point to the same object.

Step 3

```
obj.name = "Bob"
```

Heap

```
0x500

{
 name:"Bob"
}
```

Step 4

Output

```
Bob
```

---

# Common Interview Mistakes

❌ Thinking objects are stored in the stack.

❌ Saying JavaScript is "pass by reference."

JavaScript is actually **pass-by-sharing (call-by-sharing)**:
- Primitive values are copied.
- Object references are copied, not the objects themselves.

❌ Confusing the Call Stack with the Memory Heap.

❌ Believing functions are stored in the Call Stack.

---

# Best Practices

- Keep objects small and focused.
- Avoid unnecessary object mutations.
- Prefer immutable updates (especially in React).
- Always terminate recursive functions with a base condition.
- Release references to unused large objects so the Garbage Collector can reclaim memory.

---

# Senior Interview Questions

### Beginner

1. What is the Call Stack?
2. What is the Memory Heap?
3. Difference between primitive and reference types?

### Intermediate

1. Why are objects stored in the heap?
2. Explain pass-by-value vs pass-by-sharing.
3. How are function calls managed?

### Advanced

1. Explain the memory layout of this code:

```javascript
const arr = [1,2,3];
const arr2 = arr;
arr2.push(4);
```

2. Why does infinite recursion cause a Stack Overflow?

3. How does the Call Stack interact with the Event Loop?

---

# Cheat Sheet

| Concept | Summary |
|----------|---------|
| Call Stack | Stores execution contexts and function calls |
| Memory Heap | Stores objects, arrays, functions, closures |
| Primitive Types | Stored directly in the stack |
| Reference Types | Stored in the heap; stack holds references |
| Function Call | Creates a new execution context |
| Stack Overflow | Caused by excessive nested function calls |
| Pass by Value | Copies primitive values |
| Pass by Sharing | Copies object references |
| Execution Context | Created for every function invocation |

---

# Key Takeaways

✅ The **Call Stack** controls the order of execution.

✅ The **Memory Heap** stores dynamically allocated data such as objects and arrays.

✅ Primitive values live directly in the stack, while reference types live in the heap.

✅ Every function invocation creates a new **Execution Context**, which is pushed onto the Call Stack and removed when execution finishes.

✅ Understanding the interaction between the **Stack** and **Heap** is essential for mastering **Closures**, **Garbage Collection**, **React state management**, and the **Node.js Event Loop**.
