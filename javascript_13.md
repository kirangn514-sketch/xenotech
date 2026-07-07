# Chapter 5 - `this` Keyword (Part 2 - Advanced `this`)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 4-5 Hours

---

# Table of Contents

1. Four Binding Rules
2. Default Binding
3. Implicit Binding
4. Explicit Binding (`call`, `apply`, `bind`)
5. New Binding
6. Binding Priority
7. `call()`
8. `apply()`
9. `bind()`
10. Hard Binding
11. `this` in Callbacks
12. `this` in setTimeout
13. `this` in Event Listeners
14. `this` in React
15. `this` in Node.js
16. Interview Questions
17. Cheat Sheet

---

# 1. Four Binding Rules

JavaScript follows four rules to determine `this`.

```
            Function Call
                  │
                  ▼
      How was function called?
                  │
        ┌─────────┼─────────┐
        │         │         │
        ▼         ▼         ▼
 Default   Implicit   Explicit
 Binding    Binding    Binding
                  │
                  ▼
             New Binding
```

Priority (Highest → Lowest)

```
new

↓

call/apply/bind

↓

object.method()

↓

normal function
```

This priority is very important in interviews.

---

# 2. Default Binding

When a function is called normally,

```javascript
show();
```

JavaScript uses Default Binding.

Example

```javascript
function show() {
    console.log(this);
}

show();
```

Browser (Non-Strict)

```
Window
```

Strict Mode

```
undefined
```

Diagram

```
show()

↓

Normal Function

↓

Default Binding

↓

window / undefined
```

---

# 3. Implicit Binding

Whenever a function is called through an object,

```javascript
obj.show();
```

The object becomes `this`.

Example

```javascript
const user = {

    name: "Kiran",

    show() {

        console.log(this.name);

    }

};

user.show();
```

Output

```
Kiran
```

Memory

```
user

↓

show()

↓

this

↓

user
```

---

# Losing Implicit Binding

Example

```javascript
const user = {

    name: "Kiran",

    show() {

        console.log(this.name);

    }

};

const fn = user.show;

fn();
```

Output (Browser)

```
undefined
```

Why?

Because

```
fn()
```

is now a normal function call.

Implicit binding is lost.

Diagram

```
user.show()

↓

this = user

-------------------

fn()

↓

Default Binding

↓

window
```

---

# 4. Explicit Binding

JavaScript provides three methods

```
call()

apply()

bind()
```

to manually decide `this`.

---

# call()

Syntax

```javascript
fn.call(thisArg, arg1, arg2);
```

Example

```javascript
function greet(city) {

    console.log(this.name, city);

}

const user = {

    name: "Kiran"

};

greet.call(user, "Mumbai");
```

Output

```
Kiran Mumbai
```

Execution

```
call()

↓

Explicitly sets this

↓

user
```

---

# apply()

Same as call()

Difference

Arguments are passed as an array.

```javascript
function greet(city, state) {

    console.log(this.name, city, state);

}

const user = {

    name: "Kiran"

};

greet.apply(user, ["Mumbai", "Maharashtra"]);
```

Output

```
Kiran Mumbai Maharashtra
```

---

# call() vs apply()

```
call()

↓

Arguments individually

------------------

apply()

↓

Arguments as array
```

---

# bind()

`bind()` is different.

It does NOT execute the function immediately.

It returns a new function.

Example

```javascript
function greet() {

    console.log(this.name);

}

const user = {

    name: "Kiran"

};

const fn = greet.bind(user);

fn();
```

Output

```
Kiran
```

---

# Internal Flow

```
bind()

↓

Create New Function

↓

Store this=user

↓

Return Function

↓

Later

↓

Execute
```

---

# call vs apply vs bind

| Feature | call | apply | bind |
|----------|------|--------|------|
| Executes immediately | ✅ | ✅ | ❌ |
| Returns function | ❌ | ❌ | ✅ |
| Arguments | Individual | Array | Individual (partial application supported) |

---

# Partial Application using bind()

```javascript
function multiply(a, b) {

    return a * b;

}

const double = multiply.bind(null, 2);

console.log(double(10));
```

Output

```
20
```

Internally

```
double()

↓

multiply(2,10)
```

---

# Hard Binding

Example

```javascript
function show() {

    console.log(this.name);

}

const user = {

    name: "Kiran"

};

const bound = show.bind(user);

bound.call({

    name: "Rahul"

});
```

Output

```
Kiran
```

Why?

Once bound,

```
bind()

↓

Locks this
```

This is called Hard Binding.

---

# 5. New Binding

Example

```javascript
function User(name){

    this.name = name;

}

const u = new User("Kiran");
```

Flow

```
new

↓

Create Empty Object

↓

Assign this

↓

Execute Constructor

↓

Return Object
```

Output

```
User { name: "Kiran" }
```

---

# Binding Priority

Example

```javascript
function User(name){

    this.name=name;

}

const obj={};

const Bound=User.bind(obj);

const u=new Bound("Kiran");
```

Question

Who wins?

```
bind()

OR

new
```

Answer

```
new
```

Priority

```
new

>

bind

>

implicit

>

default
```

---

# 6. `this` in Callbacks

Example

```javascript
const user = {

    name: "Kiran",

    show() {

        setTimeout(function(){

            console.log(this.name);

        },1000);

    }

};

user.show();
```

Output

```
undefined
```

Reason

The callback is a normal function.

```
setTimeout

↓

Normal Function

↓

Default Binding
```

---

# Fix 1 (Arrow Function)

```javascript
setTimeout(() => {

    console.log(this.name);

},1000);
```

Arrow inherits

```
this

↓

user
```

Output

```
Kiran
```

---

# Fix 2 (bind)

```javascript
setTimeout(function(){

    console.log(this.name);

}.bind(this),1000);
```

Also works.

---

# 7. `this` in Event Listeners

Example

```javascript
button.addEventListener("click", function(){

    console.log(this);

});
```

Output

```
<button>
```

Why?

DOM sets

```
this

↓

Clicked Element
```

---

# Arrow Function

```javascript
button.addEventListener("click", ()=>{

    console.log(this);

});
```

Arrow has no own `this`.

It inherits from the surrounding scope.

This is often **not** the button.

---

# 8. `this` in React

Modern React (Functional Components)

```jsx
function App(){

    const handleClick = () => {

        console.log(this);

    }

}
```

Output

```
undefined
```

Functional components do not use `this`.

React Hooks replace class instances.

---

# React Class Component

```jsx
class App extends React.Component{

    handleClick(){

        console.log(this);

    }

}
```

Without binding,

```
this

↓

undefined
```

Common fix

```javascript
this.handleClick = this.handleClick.bind(this);
```

or

```javascript
handleClick = () => {}
```

---

# 9. `this` in Node.js

CommonJS

```javascript
console.log(this);
```

Output

```
module.exports
```

Inside normal functions

```javascript
function show(){

    console.log(this);

}

show();
```

Strict mode

```
undefined
```

---

# Common Interview Traps

## Question 1

```javascript
const obj = {

    name: "Kiran",

    show() {

        return function(){

            console.log(this.name);

        }

    }

};

obj.show()();
```

Output

```
undefined
```

Reason

Returned function is called normally.

---

## Fix

```javascript
const obj = {

    name:"Kiran",

    show(){

        return ()=>{

            console.log(this.name);

        }

    }

};
```

Output

```
Kiran
```

---

## Question 2

```javascript
const person = {

    name:"Kiran",

    greet(){

        console.log(this.name);

    }

};

const fn = person.greet;

fn.call({

    name:"Rahul"

});
```

Output

```
Rahul
```

Because `call()` explicitly sets `this`.

---

# Complete Binding Flow

```
Function Called

↓

Was it called with new?

↓

Yes

↓

this = New Object

↓

No

↓

Was call/apply/bind used?

↓

Yes

↓

Explicit Binding

↓

No

↓

Called through object?

↓

Yes

↓

Object becomes this

↓

No

↓

Default Binding

↓

window / undefined
```

---

# Best Practices

✅ Use arrow functions for callbacks.

✅ Use normal methods for objects.

✅ Prefer class fields (arrow methods) in React class components.

✅ Understand binding instead of memorizing examples.

✅ Avoid unnecessary use of `bind()` in loops.

---

# Senior Interview Questions

### Beginner

- What is implicit binding?
- Difference between `call()` and `apply()`?

---

### Intermediate

- Explain `bind()`.
- Explain binding priority.
- Why is `this` lost in callbacks?

---

### Advanced

- Explain hard binding.
- Why does `new` override `bind()`?
- Explain `this` inside `setTimeout`.
- Explain `this` in event listeners.
- Explain React's use of arrow functions.

---

# Cheat Sheet

| Invocation | `this` Value |
|------------|--------------|
| `show()` | `window` / `undefined` (strict mode) |
| `obj.show()` | `obj` |
| `call(obj)` | `obj` |
| `apply(obj)` | `obj` |
| `bind(obj)` | Permanently bound to `obj` |
| `new User()` | Newly created object |
| Arrow Function | Lexically inherited |
| Event Listener (normal function) | DOM element |
| Event Listener (arrow) | Outer lexical `this` |

---

# Key Takeaways

✅ JavaScript determines `this` using **binding rules**, not where the function is defined.

✅ There are four primary binding rules: **Default**, **Implicit**, **Explicit**, and **New Binding**.

✅ `call()` and `apply()` invoke a function immediately with a specified `this`, while `bind()` returns a new function with a permanently bound `this`.

✅ Arrow functions never create their own `this`; they inherit it from the surrounding lexical scope.

✅ The binding priority is:

```
new
   >
call/apply/bind
   >
implicit binding
   >
default binding
```

✅ Most `this`-related bugs in JavaScript arise from losing implicit binding in callbacks, timers, or event handlers. Understanding the binding rules helps avoid these issues.
