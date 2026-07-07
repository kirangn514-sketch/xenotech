# Chapter 1.5 – JavaScript Engine Internals (Part 5)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐

---

# Table of Contents

1. Complete JavaScript Execution Journey
2. Step-by-Step Internal Flow
3. Example Program
4. Parser Execution
5. AST Generation
6. Bytecode Generation
7. Global Execution Context
8. Function Execution Context
9. Memory Allocation
10. Call Stack Execution
11. TurboFan Optimization
12. Garbage Collection
13. Complete Architecture Diagram
14. Interview Questions
15. Cheat Sheet

---

# 1. Complete JavaScript Execution Journey

Let's understand what happens internally when you execute a JavaScript file.

Example

```javascript
let x = 10;

function square(num) {
    return num * num;
}

let result = square(x);

console.log(result);
```

Although this appears simple, internally V8 performs many steps.

---

# High-Level Flow

```
JavaScript File
        │
        ▼
Browser / Node.js Loads File
        │
        ▼
Tokenizer
        │
        ▼
Parser
        │
        ▼
Abstract Syntax Tree (AST)
        │
        ▼
Ignition Interpreter
        │
        ▼
Bytecode
        │
        ▼
Create Global Execution Context
        │
        ▼
Memory Allocation
        │
        ▼
Execute Statements
        │
        ▼
Function Call?
        │
        ▼
Create Function Execution Context
        │
        ▼
Execute Function
        │
        ▼
Return Result
        │
        ▼
Destroy Function Context
        │
        ▼
Continue Global Execution
        │
        ▼
Garbage Collection
```

---

# Step 1 – Source Code

Developer writes

```javascript
let x = 10;
```

Currently this is only plain text.

The CPU cannot execute it.

---

# Step 2 – Tokenizer

Input

```javascript
let x = 10;
```

Output

```
KEYWORD      let

IDENTIFIER   x

OPERATOR     =

NUMBER       10

SEMICOLON    ;
```

The tokenizer only breaks the code into meaningful symbols.

---

# Step 3 – Parser

The parser checks whether the tokens follow JavaScript grammar.

Correct

```javascript
let x = 10;
```

Incorrect

```javascript
let = 10;
```

If the syntax is invalid, execution stops.

---

# Step 4 – AST Generation

The parser creates an Abstract Syntax Tree.

```
Program

│

VariableDeclaration

│

Identifier(x)

│

NumericLiteral(10)
```

The engine no longer thinks in text.

It now works with a tree.

---

# Step 5 – Ignition Creates Bytecode

AST

↓

Bytecode

Example (simplified)

```
LoadConstant 10

Store x
```

This bytecode is executed by the interpreter.

---

# Step 6 – Global Execution Context

Now JavaScript creates the first Execution Context.

```
Global Execution Context

+----------------------------+

Variable Environment

Lexical Environment

this

+----------------------------+
```

This context lives until the program finishes.

---

# Memory Creation Phase

JavaScript scans the code.

Memory becomes

```
Variables

x

↓

<uninitialized>

result

↓

<uninitialized>

square

↓

Entire Function
```

No code has executed yet.

Only memory has been reserved.

---

# Execution Phase

Now JavaScript executes line by line.

```
let x = 10;
```

Memory

```
x

↓

10
```

---

# Function Encountered

```
function square(){}
```

Already stored during the memory phase.

Nothing new happens.

---

# Function Call

```
square(x)
```

JavaScript now creates another execution context.

Call Stack

```
+-------------------+

square()

+-------------------+

Global

+-------------------+
```

---

# Function Memory

Inside

```javascript
function square(num){

return num*num;

}
```

Memory

```
num

↓

10
```

Execution

```
10 * 10

↓

100
```

Return

```
100
```

---

# Destroy Function Context

After returning,

```
square()

removed
```

Call Stack

```
Global
```

---

# Store Result

```
result = 100
```

Memory

```
x

↓

10

result

↓

100
```

---

# Console.log

```
console.log(result)
```

Output

```
100
```

---

# Program Ends

Global Execution Context is removed.

Objects that are no longer reachable become candidates for Garbage Collection.

---

# Complete Call Stack Visualization

```
Program Starts

↓

Call Stack

+---------------------+

Global

+---------------------+

↓

square()

+---------------------+

square()

+---------------------+

Global

+---------------------+

↓

square() Returns

↓

Global

↓

Program Ends

↓

Empty Stack
```

---

# Memory During Execution

```
Call Stack

Global

↓

square()




Memory Heap

Function Object

↓

square()

Object

↓

console

String Objects

Number Objects
```

---

# TurboFan Optimization

Suppose

```javascript
for(let i=0;i<1000000;i++){

square(i);

}
```

Profiler notices

```
square()

executed

1,000,000 times
```

Flow

```
Bytecode

↓

Profiler

↓

Hot Function

↓

TurboFan

↓

Machine Code
```

Future calls become much faster.

---

# What Happens if Types Change?

Example

```javascript
square(10)

square(20)

square("Hello")
```

TurboFan assumed

```
Number

Number

Number
```

Suddenly

```
String
```

Optimization becomes invalid.

```
Machine Code

↓

Discarded

↓

Back to Bytecode
```

This is called

**Deoptimization**.

---

# Garbage Collection

Suppose

```javascript
function test(){

let user={

name:"John"

};

}

test();
```

Execution

```
Object Created

↓

Function Ends

↓

No References

↓

GC Marks Unreachable

↓

Memory Released
```

---

# Complete Internal Flow Diagram

```
Developer Writes Code
          │
          ▼
Source Code
          │
          ▼
Tokenizer
          │
          ▼
Parser
          │
          ▼
AST
          │
          ▼
Ignition Interpreter
          │
          ▼
Bytecode
          │
          ▼
Create Global Execution Context
          │
          ▼
Memory Allocation
          │
          ▼
Execute Statements
          │
          ▼
Function Call?
          │
      ┌───┴────┐
      │        │
     No       Yes
      │        │
      │        ▼
      │  Create Function
      │  Execution Context
      │        │
      │        ▼
      │  Execute Function
      │        │
      │        ▼
      │  Return Value
      │        │
      └────────┘
          │
          ▼
Profiler Watches Execution
          │
          ▼
Hot Code?
      │
 ┌────┴─────┐
 │          │
No         Yes
 │          │
 │          ▼
 │     TurboFan
 │          │
 └──────────┘
          │
          ▼
Optimized Machine Code
          │
          ▼
CPU Executes
          │
          ▼
Garbage Collector
          │
          ▼
Memory Released
```

---

# Real Interview Scenario

Question:

Explain what happens internally when JavaScript executes:

```javascript
const x = 5;

function add(a){

return a+1;

}

console.log(add(x));
```

Expected Answer

1. Browser loads script.

2. Tokenizer creates tokens.

3. Parser validates syntax.

4. AST is created.

5. Ignition converts AST into bytecode.

6. Global Execution Context is created.

7. Memory creation phase allocates:

```
x

↓

<uninitialized>

add

↓

Function

```

8. Execution phase assigns

```
x=5
```

9. Function call creates Function Execution Context.

10. Parameter receives value.

11. Function returns 6.

12. Context is removed from the Call Stack.

13. Global context continues.

14. Program ends.

15. Garbage Collector cleans unreachable objects.

---

# Senior Interview Questions

## Beginner

- What is the JavaScript execution flow?
- What is bytecode?
- Why is an AST required?

## Intermediate

- Explain the complete lifecycle of JavaScript code.
- What is the difference between Ignition and TurboFan?
- What happens during the memory creation phase?

## Advanced

- Why does V8 use both an interpreter and a compiler?
- Explain deoptimization with an example.
- How does TurboFan decide to optimize a function?
- Explain the relationship between the parser, AST, bytecode, execution context, and garbage collection.

---

# Complete Revision

```
JavaScript Source

↓

Tokenizer

↓

Parser

↓

AST

↓

Ignition

↓

Bytecode

↓

Execution Context

↓

Call Stack

↓

Heap Memory

↓

Function Calls

↓

Profiler

↓

TurboFan

↓

Machine Code

↓

CPU

↓

Garbage Collection
```

---

# Cheat Sheet

| Component | Responsibility |
|------------|----------------|
| Tokenizer | Breaks source code into tokens |
| Parser | Validates syntax and creates AST |
| AST | Tree representation of the program |
| Ignition | Generates and executes bytecode |
| Bytecode | Intermediate executable instructions |
| Execution Context | Environment where code executes |
| Call Stack | Manages function execution order |
| Memory Heap | Stores objects, arrays, functions |
| TurboFan | Optimizes hot code into machine code |
| Profiler | Detects frequently executed functions |
| Garbage Collector | Frees unreachable memory |
| CPU | Executes optimized machine instructions |

---

# Final Key Takeaways

✅ JavaScript source code is **not** executed directly by the CPU.

✅ V8 first tokenizes and parses the code, then builds an **AST**.

✅ The **Ignition interpreter** converts the AST into **bytecode**.

✅ Before executing code, JavaScript creates a **Global Execution Context** and allocates memory.

✅ Each function invocation creates its own **Function Execution Context**, managed by the **Call Stack**.

✅ Frequently executed ("hot") code is optimized by **TurboFan** into native machine code.

✅ When objects become unreachable, the **Garbage Collector** reclaims memory.

✅ Together, the **Parser → AST → Ignition → Bytecode → Execution Context → Call Stack → TurboFan → Garbage Collection** pipeline explains how JavaScript runs from start to finish.
