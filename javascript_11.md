# Chapter 4 - Closures (Part 2 - Advanced Closures & Real-World Applications)

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐

---

# Table of Contents

1. Function Factory
2. Data Encapsulation (Private Variables)
3. Module Pattern
4. Memoization
5. Currying
6. Once Function
7. Closures in Event Listeners
8. Closures in setTimeout
9. Closures in Promises
10. Closures in Async/Await
11. Closures in Node.js
12. Closures in Express Middleware
13. Closures in React
14. Stale Closures
15. Memory Leaks
16. Best Practices
17. Interview Questions
18. Cheat Sheet

---

# 1. Function Factory

A Function Factory is simply a function that creates another function.

Example

```javascript
function multiplier(x) {

    return function(y) {

        return x * y;

    }

}

const double = multiplier(2);

const triple = multiplier(3);

console.log(double(10));

console.log(triple(10));
```

Output

```
20

30
```

Memory

```
double

↓

Closure

↓

x = 2

-------------------

triple

↓

Closure

↓

x = 3
```

Each returned function has its own Lexical Environment.

---

# Why Function Factories?

Instead of writing

```javascript
function double(x){

return x*2;

}

function triple(x){

return x*3;

}
```

We create one reusable function.

```
multiplier()

↓

Creates

↓

Custom Function
```

---

# Real Interview Question

Create a greeting generator.

Solution

```javascript
function greeting(language){

    return function(name){

        console.log(`${language}: ${name}`);

    }

}

const english = greeting("Hello");

const hindi = greeting("Namaste");

english("Kiran");

hindi("Kiran");
```

Output

```
Hello: Kiran

Namaste: Kiran
```

---

# 2. Private Variables (Data Encapsulation)

JavaScript does not have traditional private variables (before ES2022 private fields).

Closures provide privacy.

Example

```javascript
function createBankAccount(){

    let balance = 0;

    return {

        deposit(amount){

            balance += amount;

        },

        withdraw(amount){

            balance -= amount;

        },

        getBalance(){

            return balance;

        }

    }

}
```

Usage

```javascript
const account = createBankAccount();

account.deposit(1000);

account.withdraw(300);

console.log(account.getBalance());
```

Output

```
700
```

Question

Can anyone access

```javascript
balance
```

No.

```
account.balance
```

Output

```
undefined
```

Why?

Because balance exists only inside the closure.

---

# Memory Diagram

```
account

↓

Closure

↓

balance

↓

700
```

Only methods can access it.

---

# 3. Module Pattern

Before ES Modules, Closures were used to create modules.

Example

```javascript
const Counter = (function(){

    let count = 0;

    return {

        increment(){

            count++;

        },

        get(){

            return count;

        }

    }

})();
```

Usage

```javascript
Counter.increment();

Counter.increment();

console.log(Counter.get());
```

Output

```
2
```

Private

```
count
```

Public

```
increment()

get()
```

---

# Module Pattern Structure

```
IIFE

↓

Private Variables

↓

Public Methods

↓

Return Object
```

---

# 4. Memoization

Memoization stores expensive computation results.

Without Memoization

```javascript
function square(x){

console.log("Calculating");

return x*x;

}

square(10);

square(10);
```

Output

```
Calculating

Calculating
```

Repeated work.

---

# Memoized Version

```javascript
function memoSquare(){

    const cache={};

    return function(num){

        if(cache[num]){

            return cache[num];

        }

        console.log("Calculating");

        cache[num]=num*num;

        return cache[num];

    }

}
```

Usage

```javascript
const square=memoSquare();

square(5);

square(5);

square(5);
```

Output

```
Calculating

25

25

25
```

Memory

```
Closure

↓

cache

↓

{

5:25

}
```

---

# Why Memoization?

Benefits

✔ Faster

✔ Avoid duplicate computation

✔ Widely used in React

Example

```
useMemo()

↓

Closure

↓

Cached Value
```

---

# 5. Currying

Currying converts

```
f(a,b,c)
```

into

```
f(a)(b)(c)
```

Example

```javascript
function add(a){

    return function(b){

        return function(c){

            return a+b+c;

        }

    }

}
```

Usage

```javascript
console.log(add(10)(20)(30));
```

Output

```
60
```

Closures preserve

```
a

↓

b
```

until the final function executes.

---

# 6. Once Function

Sometimes a function should execute only once.

Example

```javascript
function once(fn){

    let executed=false;

    return function(){

        if(executed) return;

        executed=true;

        return fn();

    }

}
```

Usage

```javascript
const init=once(()=>{

console.log("Initialized");

});

init();

init();

init();
```

Output

```
Initialized
```

Only once.

---

# 7. Closures in Event Listeners

Example

```javascript
function attach(){

    let clicks=0;

    document
    .getElementById("btn")
    .addEventListener("click",()=>{

        clicks++;

        console.log(clicks);

    });

}
```

Question

Why does

```
clicks
```

remember its value?

Because the event listener callback forms a closure.

---

# Memory

```
Button

↓

Callback

↓

Closure

↓

clicks
```

---

# 8. Closures in setTimeout

Example

```javascript
function start(){

    let message="Hello";

    setTimeout(()=>{

        console.log(message);

    },2000);

}

start();
```

Question

After

```
start()

↓

Finished
```

How does timer still print

```
Hello
```

Because the callback holds a closure.

---

# 9. Closures in Promises

```javascript
function fetchData(){

    let token="ABC123";

    return Promise.resolve().then(()=>{

        console.log(token);

    });

}

fetchData();
```

Output

```
ABC123
```

Promise callbacks preserve variables using closures.

---

# 10. Closures in Async/Await

```javascript
async function login(){

    let user="Kiran";

    await Promise.resolve();

    console.log(user);

}
```

Question

How does

```
user
```

still exist after

```
await
```

Because the async function resumes with the same lexical environment.

---

# 11. Closures in Node.js

Express Middleware

```javascript
function authorize(role){

    return function(req,res,next){

        if(req.user.role===role){

            next();

        }else{

            res.send("Forbidden");

        }

    }

}
```

Usage

```javascript
app.get(
"/admin",
authorize("admin"),
handler
);
```

Memory

```
Middleware

↓

Closure

↓

role

↓

"admin"
```

---

# 12. Database Example

```javascript
function createRepository(db){

    return{

        find(){

            return db.query(...);

        }

    }

}
```

Repository methods remember

```
db
```

through closure.

---

# 13. Closures in React

One of the biggest interview topics.

Example

```jsx
function Counter(){

    const [count,setCount]=useState(0);

    return(

        <button

        onClick={()=>{

            setCount(count+1);

        }}

        >

        {count}

        </button>

    );

}
```

Question

How does the event handler know

```
count
```

Answer

Closure.

The handler captures variables from the render in which it was created.

---

# React Render

```
Render

↓

count=0

↓

Button Handler

↓

Closure

↓

count
```

Every render creates new closures.

---

# 14. Stale Closures

One of the most asked React interview questions.

Example

```jsx
useEffect(()=>{

    console.log(count);

},[]);
```

Question

Why doesn't

```
count
```

update?

Because the effect captured the first render's closure.

This is called a

```
Stale Closure
```

---

# Fix

```jsx
useEffect(()=>{

console.log(count);

},[count]);
```

Always include dependencies.

---

# 15. Memory Leaks

Closures themselves are not memory leaks.

Problem

```javascript
function test(){

    let hugeArray=new Array(1000000);

    return function(){

        console.log(hugeArray.length);

    }

}
```

Even if only

```
length
```

is needed,

Entire array stays alive.

Memory

```
Closure

↓

Huge Array

↓

1,000,000 Items
```

GC cannot remove it.

---

# Optimization

Instead

```javascript
let size=hugeArray.length;

hugeArray=null;
```

Now only

```
size
```

is preserved.

---

# Best Practices

✅ Use closures for encapsulation.

✅ Avoid retaining huge objects.

✅ Remove event listeners.

✅ Clear timers.

✅ Understand React dependency arrays.

✅ Prefer WeakMap for object metadata.

---

# Senior Interview Questions

### Beginner

- What is a closure?
- Why are closures useful?

---

### Intermediate

- Explain memoization.
- Explain the Module Pattern.
- Explain private variables using closures.

---

### Advanced

1. Explain stale closures in React.

2. Why do React Hooks depend on closures?

3. Explain closures in async/await.

4. Can closures cause memory leaks?

5. Explain closure memory using Heap and Garbage Collection.

6. How does Express middleware use closures?

---

# Cheat Sheet

| Pattern | Closure Usage |
|----------|---------------|
| Function Factory | Create customized functions |
| Private Variables | Hide state |
| Module Pattern | Encapsulation |
| Memoization | Cache results |
| Currying | Preserve arguments |
| Once Function | Execute once |
| Event Listener | Preserve state |
| setTimeout | Preserve callback variables |
| Promise | Preserve async state |
| Async/Await | Resume with same lexical environment |
| React Hooks | Capture render state |
| Express Middleware | Capture configuration |

---

# Key Takeaways

✅ Closures are the foundation of many JavaScript design patterns.

✅ Function factories, currying, memoization, and module patterns all rely on closures.

✅ React event handlers, `useEffect`, `useCallback`, and `useMemo` depend heavily on closures.

✅ Node.js middleware and asynchronous callbacks use closures to preserve configuration and execution state.

✅ Closures themselves are **not** memory leaks, but they can keep large objects alive if references are retained unnecessarily.

✅ Mastering closures is essential for understanding modern JavaScript frameworks, asynchronous programming, and senior-level interview questions.
