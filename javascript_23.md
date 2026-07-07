# Chapter 7 - Streams, Buffers & Backpressure
# Part 2 - Streams Deep Dive

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 7-8 Hours

---

# Table of Contents

1. Why Streams?
2. What is a Stream?
3. Stream Lifecycle
4. Stream Types
5. Readable Streams
6. Writable Streams
7. Duplex Streams
8. Transform Streams
9. Stream Events
10. pipe() Internals
11. pipeline()
12. Real-world Examples
13. Best Practices
14. Interview Questions
15. Cheat Sheet

---

# 1. Why Streams?

Suppose you need to send a **10 GB** video file to a client.

Option 1

```javascript
const fs = require("fs");

const data = fs.readFileSync("movie.mp4");

res.send(data);
```

Problem

```
10 GB File

↓

Load Entire File

↓

Memory Explosion

↓

Server Crash
```

---

Option 2

```javascript
const stream = fs.createReadStream("movie.mp4");

stream.pipe(res);
```

Execution

```
10 GB File

↓

64 KB Chunk

↓

Client

↓

Next 64 KB

↓

Client

↓

Repeat
```

Memory remains almost constant.

---

# Memory Comparison

Without Stream

```
File

10 GB

↓

RAM

10 GB
```

---

With Stream

```
File

10 GB

↓

64 KB

↓

64 KB

↓

64 KB

↓

RAM ≈ 64 KB
```

This is why Streams are memory efficient.

---

# 2. What is a Stream?

Definition

A Stream is a sequence of data that becomes available over time instead of all at once.

Think of water flowing through a pipe.

```
Water Tank

↓

Pipe

↓

Glass
```

You don't wait for the whole tank to empty.

Data behaves similarly.

```
Disk

↓

Chunk

↓

Application

↓

Chunk

↓

Application
```

---

# Examples of Streams

- Reading files
- Writing files
- HTTP request body
- HTTP response body
- TCP socket
- Video streaming
- Audio streaming
- Database export
- CSV processing

---

# 3. Stream Lifecycle

```
Source

↓

Open

↓

Read Chunk

↓

Process Chunk

↓

Read Next Chunk

↓

...

↓

End

↓

Close
```

The application processes data continuously.

---

# 4. Types of Streams

Node.js provides four stream types.

| Stream | Purpose |
|---------|----------|
| Readable | Read data |
| Writable | Write data |
| Duplex | Read + Write |
| Transform | Read, modify, write |

---

# Diagram

```
Readable

Disk

↓

Application

--------------------

Writable

Application

↓

Disk

--------------------

Duplex

Client

⇅

Server

--------------------

Transform

Input

↓

Modify

↓

Output
```

---

# 5. Readable Stream

Readable streams produce data.

Example

```javascript
const fs = require("fs");

const stream = fs.createReadStream("users.csv");
```

---

Internal Flow

```
Disk

↓

Chunk

↓

Readable Stream

↓

Application
```

---

Listening to Data

```javascript
stream.on("data",(chunk)=>{

    console.log(chunk.length);

});
```

Output

```
65536

65536

65536

...
```

Each chunk is typically **64 KB** for file streams.

---

End Event

```javascript
stream.on("end",()=>{

    console.log("Finished");

});
```

Flow

```
Chunk

↓

Chunk

↓

Chunk

↓

End Event
```

---

# 6. Writable Stream

Writable streams consume data.

Example

```javascript
const writeStream =
fs.createWriteStream("output.txt");
```

Writing

```javascript
writeStream.write("Hello\n");

writeStream.write("Node.js\n");

writeStream.end();
```

Execution

```
Application

↓

Writable Stream

↓

Disk
```

---

# 7. Duplex Stream

Duplex streams can read and write.

Examples

- TCP Socket
- WebSocket
- HTTP/2
- TLS Connection

Diagram

```
Client

↓

Server

↑

Response
```

Both directions remain open.

---

Socket Example

```javascript
socket.write("Hello");

socket.on("data",(data)=>{

    console.log(data);

});
```

```
Read

+

Write
```

at the same time.

---

# 8. Transform Stream

Transform streams modify data while passing it through.

Examples

- gzip compression
- encryption
- CSV → JSON
- image resizing
- uppercase conversion

---

Example

```
Input

↓

Transform

↓

Output
```

---

Compression

```javascript
fs.createReadStream("video.mp4")

.pipe(zlib.createGzip())

.pipe(fs.createWriteStream("video.gz"));
```

Execution

```
Read

↓

Compress

↓

Write
```

---

# 9. Stream Events

Important events

| Event | Description |
|---------|-------------|
| data | Chunk received |
| readable | Data ready |
| end | Reading complete |
| finish | Writing complete |
| close | Resource closed |
| error | Error occurred |

---

Flow

```
Open

↓

data

↓

data

↓

data

↓

end

↓

close
```

---

# 10. pipe() Internals

Without pipe()

```javascript
read.on("data",(chunk)=>{

    write.write(chunk);

});
```

You manually transfer chunks.

---

With pipe()

```javascript
read.pipe(write);
```

Internally

```
Readable

↓

Chunk

↓

Writable

↓

Repeat
```

Much simpler.

---

# Internal Diagram

```
Readable

↓

Chunk

↓

pipe()

↓

Writable
```

`pipe()` also coordinates flow control automatically.

---

# 11. pipeline()

`pipe()` doesn't automatically destroy every stream on failure.

Use `pipeline()` for production.

```javascript
const { pipeline } = require("stream");

pipeline(

    fs.createReadStream("input.txt"),

    fs.createWriteStream("output.txt"),

    (err)=>{

        if(err){

            console.log(err);

        }

    }

);
```

Benefits

- Automatic cleanup
- Better error handling
- Prevents resource leaks

---

# 12. Real-world Examples

## Video Streaming

```
Disk

↓

Readable Stream

↓

HTTP Response

↓

Browser
```

No need to load the entire movie into memory.

---

## CSV Processing

```
CSV

↓

Readable

↓

Parse

↓

Database
```

Millions of rows can be processed efficiently.

---

## AWS S3 Upload

```
Client Upload

↓

Readable Stream

↓

S3 SDK

↓

Cloud Storage
```

---

## Express Download API

```javascript
app.get("/download",(req,res)=>{

    fs.createReadStream("report.pdf")

    .pipe(res);

});
```

Memory usage remains low regardless of file size.

---

# Stream Chaining

Streams are often chained.

```javascript
fs.createReadStream("input.txt")

.pipe(zlib.createGzip())

.pipe(fs.createWriteStream("output.gz"));
```

Flow

```
Disk

↓

Read

↓

Compress

↓

Write
```

---

# Common Mistakes

❌ Using `fs.readFile()` for huge files.

❌ Ignoring `"error"` events.

❌ Forgetting to close writable streams.

❌ Manually copying chunks when `pipe()` is sufficient.

---

# Best Practices

✅ Use streams for files larger than a few MB.

✅ Prefer `pipeline()` in production.

✅ Always handle `"error"` events.

✅ Chain streams with `pipe()`.

✅ Process data incrementally.

---

# Senior Interview Questions

## Beginner

- What is a Stream?
- Why are Streams memory efficient?

---

## Intermediate

- Difference between Readable and Writable streams?
- What is `pipe()`?

---

## Advanced

- Difference between `pipe()` and `pipeline()`?
- Explain Transform streams.
- How would you stream a 50 GB file?
- Why is streaming faster than `readFile()`?
- How does Node.js stream video to browsers?

---

# Cheat Sheet

| Stream Type | Purpose |
|--------------|---------|
| Readable | Produce data |
| Writable | Consume data |
| Duplex | Read + Write |
| Transform | Modify data |

---

# Complete Stream Flow

```
Disk

↓

Readable Stream

↓

Chunk

↓

Transform (optional)

↓

Writable Stream

↓

Disk / Network / Browser
```

---

# Key Takeaways

✅ Streams process data **incrementally**, not all at once.

✅ Streaming large files dramatically reduces memory usage.

✅ Readable streams generate data, Writable streams consume it, Duplex streams do both, and Transform streams modify data in transit.

✅ `pipe()` automatically transfers chunks from a readable stream to a writable stream.

✅ `pipeline()` is preferred in production because it provides better error handling and resource cleanup.

✅ Streams are the foundation of file downloads, uploads, HTTP communication, video streaming, compression, cloud storage integrations, and high-performance Node.js applications.
