# Chapter 5 - `this` Keyword (Part 1 - Core Concepts)

> Level: Intermediate → Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 4 Hours

---

# Table of Contents

1. What is `this`?
2. Why Do We Need `this`?
3. How JavaScript Determines `this`
4. Global Context
5. Function Invocation
6. Method Invocation
7. Constructor Invocation (`new`)
8. Class Methods
9. Arrow Functions
10. `this` vs Lexical Scope
11. Internal Execution Flow
12. Common Mistakes
13. Interview Questions
14. Cheat Sheet

---

# 1. What is `this`?

## Definition

`this` is a special keyword that refers to **the object associated with the current function execution**.

> Important:
>
> `this` is determined **when a function is called**, **not when it is created**.

This is one of the most important interview points.

---

# Biggest Myth

Many developers think:

```
this = object where function is written
```

❌ Wrong

JavaScript decides `this` based on **how the function is invoked**.

---

# Example

```javascript
const user = {
    name: "Kiran",

    greet() {
        console.log(this.name);
    }
};

user.greet();
```

Output

```
Kiran
```

Why?

Because the function is called as:

```
user.greet()
```

So,

```
this

↓

user
```

---

# 2. Why Do We Need `this`?

Imagine we had no `this`.

```javascript
const person = {
    name: "Kiran",

    greet() {
        console.log(person.name);
    }
};
```

Works...

But now

```javascript
const another = person;

person = null;
```

Now

```
person.name
```

Fails.

Using `this`

```javascript
greet() {
    console.log(this.name);
}
```

makes the function reusable.

---

# Example

```javascript
const person1 = {
    name: "Kiran",

    greet() {
        console.log(this.name);
    }
};

const person2 = {
    name: "Rahul",

    greet: person1.greet
};

person1.greet();

person2.greet();
```

Output

```
Kiran

Rahul
```

Same function.

Different `this`.

---

# 3. How JavaScript Determines `this`

When JavaScript encounters

```javascript
obj.sayHello();
```

It does NOT simply execute the function.

Internally

```
Find Function

↓

Determine Call Type

↓

Assign this

↓

Create Execution Context

↓

Execute Function
```

Notice

`this` is stored inside the Execution Context.

---

# Execution Context

Every Function Execution Context contains

```
Execution Context

------------------------

Lexical Environment

Variable Environment

this

------------------------
```

The value of `this` depends on the invocation type.

---

# 4. Global Context

## Browser

```javascript
console.log(this);
```

Output

```
window
```

Global Execution Context

```
Global EC

↓

this

↓

window
```

---

## Node.js

CommonJS Module

```javascript
console.log(this);
```

Output

```
{}
```

Why?

Node wraps every file inside

```javascript
(function(exports, require, module){

// your code

});
```

So

```
this

↓

module.exports
```

---

# 5. Function Invocation

Example

```javascript
function greet() {
    console.log(this);
}

greet();
```

### Browser (Non-Strict Mode)

Output

```
window
```

---

### Strict Mode

```javascript
"use strict";

function greet() {
    console.log(this);
}

greet();
```

Output

```
undefined
```

Reason

JavaScript does NOT automatically bind the global object in strict mode.

---

# Execution Flow

```
Function Call

↓

Normal Function?

↓

Yes

↓

Strict Mode?

↓

No

↓

this = window

----------------

Strict?

↓

Yes

↓

this = undefined
```

---

# 6. Method Invocation

Example

```javascript
const car = {

    brand: "BMW",

    show() {

        console.log(this.brand);

    }

};

car.show();
```

Output

```
BMW
```

Execution

```
car.show()

↓

Method Call

↓

this = car
```

---

# Important Rule

The object **before the dot (`.`)** becomes `this`.

Example

```javascript
user.login();
```

```
this

↓

user
```

---

# Nested Objects

```javascript
const company = {

    name: "ABC",

    department: {

        title: "IT",

        print() {

            console.log(this.title);

        }

    }

};

company.department.print();
```

Output

```
IT
```

Because

```
this

↓

department
```

---

# 7. Constructor Invocation

Example

```javascript
function User(name){

    this.name = name;

}

const u = new User("Kiran");
```

Question

What is `this`?

Answer

A newly created object.

---

# Internal Working

```
new User()

↓

Create Empty Object

↓

Assign this

↓

Execute Constructor

↓

Return Object
```

Equivalent

```javascript
const obj = {};

User.call(obj, "Kiran");

return obj;
```

---

# Memory Diagram

```
Heap

↓

Object

↓

name

↓

"Kiran"
```

`this`

↓

Points to that object.

---

# 8. Classes

Classes internally use constructors.

```javascript
class Employee{

    constructor(name){

        this.name=name;

    }

}
```

Calling

```javascript
new Employee("Kiran")
```

is similar to

```javascript
new User("Kiran")
```

`this`

↓

New Instance

---

# Class Method

```javascript
class User{

    constructor(name){

        this.name=name;

    }

    greet(){

        console.log(this.name);

    }

}

const u=new User("Kiran");

u.greet();
```

Output

```
Kiran
```

---

# 9. Arrow Functions

Arrow functions are completely different.

They **do not have their own `this`**.

Instead,

they inherit `this` from the surrounding lexical scope.

Example

```javascript
const obj = {

    name: "Kiran",

    greet() {

        const inner = () => {

            console.log(this.name);

        };

        inner();

    }

};

obj.greet();
```

Output

```
Kiran
```

---

# Why?

Arrow Function

```
No own this

↓

Look outside

↓

Method

↓

this = obj
```

---

# Normal Function

```javascript
const obj = {

    name: "Kiran",

    greet() {

        function inner(){

            console.log(this);

        }

        inner();

    }

};

obj.greet();
```

Browser Output

```
window
```

Strict Mode

```
undefined
```

Because

```
inner()

↓

Normal Function Call

↓

Own this
```

---

# Arrow vs Normal Function

| Feature | Normal Function | Arrow Function |
|----------|----------------|----------------|
| Own `this` | ✅ Yes | ❌ No |
| `this` decided | Call Time | Lexical |
| Constructor | ✅ Yes | ❌ No |
| Can use `new` | ✅ | ❌ |
| `arguments` object | ✅ | ❌ |

---

# 10. `this` vs Lexical Scope

Many developers confuse these.

Example

```javascript
let x = 100;

const obj = {

    x: 10,

    show(){

        console.log(this.x);

    }

};
```

Output

```
10
```

Why?

Variable lookup uses

```
Lexical Scope
```

But

```
this

↓

Call Site
```

These are completely different mechanisms.

---

# Complete Internal Flow

```
Function Call

↓

Create Execution Context

↓

Determine Invocation Type

↓

Assign this

↓

Initialize Variables

↓

Execute Function

↓

Destroy Execution Context
```

---

# Common Mistakes

❌ Thinking `this` is decided where the function is written.

❌ Using arrow functions as object methods without understanding lexical `this`.

❌ Forgetting strict mode changes `this`.

❌ Assuming `this` and lexical scope are the same.

---

# Best Practices

✅ Use regular methods in objects and classes.

✅ Use arrow functions for callbacks that need the parent's `this`.

✅ Prefer strict mode or ES Modules.

✅ Understand invocation patterns instead of memorizing rules.

---

# Senior Interview Questions

## Beginner

- What is `this`?
- How is `this` determined?

---

## Intermediate

- Difference between normal and arrow functions?
- Why does strict mode change `this`?

---

## Advanced

Explain the output:

```javascript
const obj = {

    value: 10,

    method(){

        function inner(){

            console.log(this);

        }

        inner();

    }

};

obj.method();
```

Answer

`inner()` is a normal function call.

Therefore

Browser

```
window
```

Strict Mode

```
undefined
```

---

# Cheat Sheet

| Invocation | Value of `this` |
|------------|-----------------|
| Global (Browser) | `window` |
| Global (Node CommonJS) | `module.exports` |
| Normal Function (Non-Strict) | `window` |
| Normal Function (Strict) | `undefined` |
| Object Method | Object before `.` |
| Constructor (`new`) | Newly created object |
| Class Method | Instance |
| Arrow Function | Inherited from surrounding scope |

---

# Key Takeaways

✅ `this` is determined **at function invocation**, not function definition.

✅ Every Function Execution Context contains a `this` binding.

✅ Normal functions create their own `this`.

✅ Arrow functions do **not** create a new `this`; they inherit it lexically.

✅ Constructor functions and classes bind `this` to the newly created instance.

✅ Understanding invocation patterns is the key to mastering `this`.
