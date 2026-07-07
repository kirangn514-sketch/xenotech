# Chapter 7 - Streams, Buffers & Backpressure
# Part 4 - Building Custom Streams

> Level: Expert
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 8 Hours

---

# Table of Contents

1. Why Build Custom Streams?
2. Stream Base Classes
3. Creating a Readable Stream
4. Creating a Writable Stream
5. Creating a Duplex Stream
6. Creating a Transform Stream
7. Object Mode
8. Stream Lifecycle
9. Internal Methods
10. Real-world Examples
11. Best Practices
12. Interview Questions
13. Cheat Sheet

---

# 1. Why Build Custom Streams?

Node.js already provides streams for:

- Files
- HTTP
- TCP
- zlib
- Crypto

So why create our own?

Because sometimes the data source isn't a file.

Examples:

- Database rows
- Kafka messages
- Redis Pub/Sub
- AI token streaming
- CSV parser
- Excel parser
- XML parser
- API pagination

Example

```
Database

↓

Custom Readable

↓

Application
```

---

# 2. Stream Base Classes

Node.js provides four base classes.

```javascript
const {

Readable,

Writable,

Duplex,

Transform

} = require("stream");
```

Everything is built on top of these.

```
Stream

│

├── Readable

├── Writable

├── Duplex

└── Transform
```

---

# 3. Creating a Readable Stream

A Readable stream **produces data**.

You must implement

```
_read(size)
```

---

Example

```javascript
const { Readable } = require("stream");

class NumberStream extends Readable{

    constructor(){

        super();

        this.current = 1;

    }

    _read(){

        if(this.current > 5){

            this.push(null);

            return;

        }

        this.push(String(this.current++));

    }

}

const stream = new NumberStream();

stream.on("data",(chunk)=>{

    console.log(chunk.toString());

});
```

Output

```
1

2

3

4

5
```

---

# Internal Flow

```
Application

↓

Needs Data

↓

_read()

↓

push(chunk)

↓

Readable Queue

↓

data Event

↓

Consumer
```

---

# What is push()?

`push()` places data into the readable stream's internal buffer.

```
_read()

↓

push(chunk)

↓

Internal Buffer

↓

Consumer
```

Special value

```javascript
this.push(null);
```

Means

```
End Of Stream
```

---

# 4. Creating a Writable Stream

Writable streams consume data.

Implement

```
_write(chunk, encoding, callback)
```

Example

```javascript
const { Writable } = require("stream");

class Logger extends Writable{

    _write(chunk, encoding, callback){

        console.log(chunk.toString());

        callback();

    }

}

const logger = new Logger();

logger.write("Hello");

logger.write("Node");

logger.end();
```

Output

```
Hello

Node
```

---

# Why callback()?

Node.js must know

```
Current Write Finished

↓

Ready For Next Chunk
```

If callback isn't called

```
Stream Stuck
```

---

# Writable Flow

```
write()

↓

Internal Buffer

↓

_write()

↓

callback()

↓

Next Chunk
```

---

# 5. Creating a Duplex Stream

A Duplex stream can both read and write.

Implement

```
_read()

+

_write()
```

Example

```javascript
const { Duplex } = require("stream");

class Echo extends Duplex{

    _read(){}

    _write(chunk, encoding, callback){

        this.push(chunk);

        callback();

    }

}

const echo = new Echo();

echo.on("data",(data)=>{

    console.log(data.toString());

});

echo.write("Hello");
```

Output

```
Hello
```

---

# Duplex Diagram

```
Write

↓

_write()

↓

push()

↓

Read
```

---

# Real Examples

- TCP Socket
- WebSocket
- SSH
- HTTP/2

All are Duplex streams.

---

# 6. Creating a Transform Stream

Transform streams modify data.

You implement

```
_transform()
```

Example

```javascript
const { Transform } = require("stream");

class UpperCase extends Transform{

    _transform(chunk, encoding, callback){

        this.push(

            chunk

            .toString()

            .toUpperCase()

        );

        callback();

    }

}

const upper = new UpperCase();

upper.on("data",(data)=>{

    console.log(data.toString());

});

upper.write("hello");

upper.end();
```

Output

```
HELLO
```

---

# Internal Flow

```
Input

↓

_transform()

↓

Modify

↓

push()

↓

Output
```

---

# Multiple Chunks

```
hello

↓

HELLO

world

↓

WORLD
```

Every chunk passes through `_transform()`.

---

# 7. Object Mode

Normally streams work with

```
Buffer

or

String
```

Sometimes we want

```
JavaScript Objects
```

Enable Object Mode

```javascript
const stream = new Readable({

    objectMode: true

});
```

---

Example

```javascript
const { Readable } = require("stream");

const users = [

    {id:1},

    {id:2},

    {id:3}

];

const stream = Readable.from(users,{
    objectMode:true
});

stream.on("data",(user)=>{

    console.log(user);

});
```

Output

```javascript
{ id:1 }

{ id:2 }

{ id:3 }
```

---

# Why Object Mode?

Useful for

- Database records
- JSON objects
- CSV rows
- Kafka messages
- Event processing

---

# Buffer Mode vs Object Mode

| Buffer Mode | Object Mode |
|--------------|-------------|
| Buffers | JS Objects |
| Byte-based | Object-based |
| Default | Must enable |
| File Streaming | Data Pipelines |

---

# 8. Stream Lifecycle

Readable

```
Constructor

↓

_read()

↓

push()

↓

data

↓

end

↓

close
```

Writable

```
Constructor

↓

write()

↓

_write()

↓

callback()

↓

finish

↓

close
```

Transform

```
Input

↓

_transform()

↓

push()

↓

Output

↓

finish
```

---

# 9. Internal Methods

| Method | Purpose |
|----------|---------|
| _read() | Produce data |
| _write() | Consume data |
| _transform() | Modify data |
| _flush() | Final cleanup before stream ends |
| push() | Send data to readable side |
| callback() | Signal completion |

---

# _flush()

Called after all chunks have been processed.

Example

```javascript
_flush(callback){

    console.log("Finished");

    callback();

}
```

Useful for

- Writing trailers
- Final aggregation
- Closing resources

---

# 10. Real-world Examples

## CSV Parser

```
CSV File

↓

Readable

↓

Transform

↓

JSON Objects

↓

Application
```

---

## Encryption Stream

```
Input

↓

Encrypt

↓

Encrypted Output
```

---

## AI Streaming

```
LLM Tokens

↓

Transform

↓

HTTP Response

↓

Browser
```

---

## Log Processor

```
Log File

↓

Readable

↓

Transform

↓

Filter

↓

Database
```

---

# 11. Common Mistakes

❌ Forgetting `callback()` in `_write()` or `_transform()`.

❌ Never calling `push(null)` in a Readable stream.

❌ Mixing Object Mode and Buffer Mode incorrectly.

❌ Ignoring `"error"` events.

❌ Doing heavy synchronous work inside `_transform()`.

---

# 12. Best Practices

✅ Use Transform streams for data processing.

✅ Enable `objectMode` when working with JavaScript objects.

✅ Keep `_transform()` lightweight.

✅ Always call `callback()` exactly once.

✅ Always end Readable streams with `push(null)`.

---

# Senior Interview Questions

## Beginner

- What is a custom stream?
- Difference between Readable and Writable?

---

## Intermediate

- What does `_read()` do?
- What is `push()`?
- What is Object Mode?

---

## Advanced

- Explain the lifecycle of a Transform stream.
- Difference between Duplex and Transform?
- What happens if `_write()` never calls `callback()`?
- Why is `push(null)` required?
- When would you use `_flush()`?

---

# Internal Architecture

```
             Readable

          _read()

              │

         push(chunk)

              │

      Internal Buffer

              │

          Consumer

--------------------------------

             Writable

      write(chunk)

              │

          _write()

              │

        callback()

--------------------------------

            Transform

      Input Chunk

              │

        _transform()

              │

         push(chunk)

              │

        Output Chunk
```

---

# Cheat Sheet

| Class | Required Method | Purpose |
|--------|-----------------|---------|
| Readable | `_read()` | Generate data |
| Writable | `_write()` | Consume data |
| Duplex | `_read()` + `_write()` | Read and Write |
| Transform | `_transform()` | Modify data |
| Transform | `_flush()` | Final cleanup |

---

# Key Takeaways

✅ Custom streams allow you to stream data from **any source**, not just files.

✅ `Readable` streams generate data using `_read()` and `push()`.

✅ `Writable` streams consume data using `_write()` and must call `callback()` when processing is complete.

✅ `Duplex` streams combine reading and writing, making them ideal for sockets and bidirectional communication.

✅ `Transform` streams modify data as it flows through the pipeline and are the foundation of compression, encryption, parsing, and many open-source libraries.

✅ `objectMode` allows streams to process JavaScript objects instead of raw bytes, making it ideal for ETL pipelines, database records, and event-driven systems.
