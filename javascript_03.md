# Chapter 1.5 – JavaScript Engine Internals (Part 2)

> Level: Intermediate → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐

---

# Table of Contents

1. How V8 Processes JavaScript
2. Source Code
3. Lexical Analysis (Tokenizer)
4. Parser
5. Abstract Syntax Tree (AST)
6. Ignition Interpreter
7. Bytecode
8. TurboFan Compiler
9. Just-In-Time (JIT) Compilation
10. Deoptimization
11. Hidden Classes
12. Inline Caching
13. Complete Flow
14. Performance Tips
15. Interview Questions
16. Cheat Sheet

---

# 1. Complete Overview

Whenever you write JavaScript like:

```javascript
function add(a, b) {
    return a + b;
}

console.log(add(10,20));
```

The CPU **cannot execute** this code directly.

Instead, V8 processes it through multiple stages.

```
JavaScript Source Code
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
    Execute Bytecode
          │
          ▼
 Frequently Executed?
          │
     Yes ▼
     TurboFan
          │
          ▼
 Optimized Machine Code
          │
          ▼
         CPU
```

Every stage has a specific responsibility.

---

# Step 1 - Source Code

This is the code written by developers.

Example

```javascript
let x = 10;

let y = 20;

console.log(x+y);
```

Currently, this is just **plain text**.

The engine still doesn't understand it.

---

# Step 2 - Lexical Analysis (Tokenizer)

The first stage is called **Lexical Analysis**.

It is performed by the **Tokenizer (Lexer)**.

The tokenizer breaks the source code into **Tokens**.

Think of tokens as the smallest meaningful pieces of the language.

Example

```javascript
let total = price + tax;
```

Tokenizer Output

```
KEYWORD     let

IDENTIFIER  total

OPERATOR    =

IDENTIFIER  price

OPERATOR    +

IDENTIFIER  tax

SEMICOLON   ;
```

Notice:

The engine does **not** understand the sentence.

It only recognizes individual pieces.

Real-world analogy:

```
Sentence

↓

Words

↓

Dictionary Meaning
```

Similarly,

```
JavaScript

↓

Tokens

↓

Parser
```

---

# Types of Tokens

Common token types include:

```
Keyword

let

const

function

return
```

Identifiers

```
price

total

userName
```

Operators

```
+

-

*

/

=

==
===
```

Literals

```
100

"Hello"

true
```

Punctuation

```
(

)

{

}

;

,
```

---

# Why Tokenization?

Imagine parsing this directly:

```
let total = price + tax;
```

Instead, V8 first converts it into structured pieces.

Without tokenization, parsing would be much more complex.

---

# Step 3 - Parser

Now the Parser receives the tokens.

Its responsibilities:

✅ Check syntax

✅ Detect errors

✅ Build program structure

Example

```javascript
let x = ;
```

Tokenizer succeeds.

Parser fails.

Why?

Because the syntax is invalid.

Parser Error

```
Unexpected token ;
```

The parser ensures your code follows JavaScript grammar.

---

# Parser Analogy

Imagine an English teacher checking grammar.

Sentence:

```
I am going school.
```

Teacher says

❌ Wrong Grammar.

Similarly,

Parser checks JavaScript grammar.

---

# Step 4 - Abstract Syntax Tree (AST)

After parsing, V8 creates an AST.

AST = Abstract Syntax Tree.

This is **one of the most important interview topics.**

AST represents your program as a tree.

Example

```javascript
let x = 10 + 20;
```

AST

```
Program

│

VariableDeclaration

│

Identifier(x)

│

BinaryExpression

├──10

├──+

└──20
```

Notice

The engine no longer thinks in text.

It thinks in tree structures.

---

# Bigger Example

```javascript
function add(a,b){

return a+b;

}
```

AST

```
Program

│

FunctionDeclaration

├──Identifier(add)

├──Parameter(a)

├──Parameter(b)

└──ReturnStatement

        │

BinaryExpression

├──a

├──+

└──b
```

Everything becomes a tree.

---

# Why AST?

AST makes optimization possible.

Examples

Code Minifiers

```
Terser

SWC

Babel
```

ESLint

Prettier

Webpack

TypeScript

All of them work using AST.

---

# Step 5 - Ignition Interpreter

V8 doesn't immediately generate machine code.

Instead,

AST

↓

Ignition Interpreter

↓

Bytecode

Ignition is a lightweight interpreter.

Its job:

Convert AST into Bytecode.

---

# Why Bytecode?

Machine code generation is expensive.

Suppose you execute

```javascript
2+2
```

only once.

Generating optimized machine code is wasteful.

Instead,

V8 generates Bytecode.

Bytecode executes quickly enough.

---

# Bytecode

Bytecode is an intermediate language.

```
JavaScript

↓

Bytecode

↓

Machine Code
```

Think of Bytecode as:

English

↓

Simple English

↓

Machine Language

---

# Example Flow

JavaScript

```javascript
let x = 10;

x++;
```

Bytecode (simplified)

```
LoadConstant 10

Store x

Load x

Increment

Store x
```

Actual bytecode is more complex.

---

# Why Bytecode?

Advantages

✔ Faster startup

✔ Less memory

✔ Easier optimization

✔ Platform independent

---

# Step 6 - Execute Bytecode

Ignition executes Bytecode.

Example

```
Load Constant

↓

Store Variable

↓

Read Variable

↓

Addition

↓

Store Result
```

The CPU now performs the actual calculations.

---

# Step 7 - TurboFan Compiler

Suppose this function executes 10 million times.

```javascript
function square(x){

return x*x;

}
```

Should V8 continue interpreting?

No.

It becomes slow.

Instead,

TurboFan optimizes it.

```
Bytecode

↓

Profiler

↓

Hot Function?

↓

Yes

↓

TurboFan

↓

Machine Code
```

---

# Hot Function

Hot = Frequently Executed

Example

```javascript
for(let i=0;i<1000000;i++){

square(i);

}
```

Profiler detects

```
square()

called

1,000,000 times
```

TurboFan optimizes it.

---

# Machine Code

Instead of interpreting

```
square()

↓

Bytecode

↓

Interpreter
```

Now

```
square()

↓

Machine Code

↓

CPU
```

Much faster.

---

# Step 8 - Just-In-Time Compilation (JIT)

Traditional Languages

```
Source

↓

Compiler

↓

Machine Code

↓

Run
```

JavaScript

```
Source

↓

Interpreter

↓

Profiler

↓

Compiler

↓

Machine Code
```

This is called

**Just-In-Time Compilation**

Because compilation happens while the program is running.

---

# Why JIT?

Without JIT

Startup Slow

Compilation Slow

Memory High

With JIT

Fast Startup

Fast Execution

Optimized Performance

---

# Deoptimization

Sometimes TurboFan makes assumptions.

Example

```javascript
function add(a,b){

return a+b;

}
```

Profiler observes

```
10+20

30+40

100+50
```

Assumption

Both parameters are Numbers.

TurboFan optimizes.

Later

```javascript
add("Hello","World");
```

Oops.

Assumption failed.

TurboFan discards optimized code.

Returns to Bytecode.

This is called

**Deoptimization**

---

# Hidden Classes

JavaScript objects are dynamic.

Example

```javascript
let user = {

name:"John",

age:25

}
```

Internally

V8 creates

```
Hidden Class A

name

age
```

Another object

```javascript
let emp={

name:"Bob",

age:30

}
```

Same Hidden Class.

Fast property access.

---

# Bad Example

```javascript
let user={};

user.name="John";

user.age=25;

delete user.age;
```

Hidden Classes keep changing.

Performance decreases.

---

# Inline Caching

Repeated property access

```javascript
user.name

user.name

user.name

user.name
```

Instead of looking up every time,

V8 caches the location.

Next access

↓

Direct memory lookup.

Much faster.

---

# Complete V8 Flow

```
JavaScript

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

Profiler

↓

Hot Code?

↓

No → Continue Bytecode

↓

Yes

↓

TurboFan

↓

Machine Code

↓

CPU
```

---

# Performance Tips

✅ Keep object shapes consistent.

✅ Avoid deleting object properties frequently.

✅ Reuse functions.

✅ Avoid unnecessary type changes.

✅ Prefer stable data structures.

---

# Senior Interview Questions

### Beginner

- What is AST?
- What is Bytecode?
- Why do we need a Parser?

### Intermediate

- Difference between Parser and Tokenizer?
- Why does V8 use Bytecode?
- Explain Ignition.

### Senior

- Explain TurboFan optimization.
- What is Hidden Class?
- What is Inline Cache?
- What is Deoptimization?
- Explain JIT Compilation.

---

# Cheat Sheet

| Component | Responsibility |
|------------|---------------|
| Tokenizer | Converts source code into tokens |
| Parser | Validates syntax and builds AST |
| AST | Tree representation of code |
| Ignition | Converts AST to bytecode |
| Bytecode | Intermediate executable format |
| Profiler | Detects hot code |
| TurboFan | Compiles hot code to optimized machine code |
| JIT | Compiles during execution |
| Hidden Classes | Optimize object property access |
| Inline Cache | Cache repeated property lookups |
| Deoptimization | Falls back when optimization assumptions fail |
