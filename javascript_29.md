# Chapter 8 - Authentication & Authorization
# Part 3 - JWT (JSON Web Token) Deep Dive

> Level: Advanced → Expert
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 10 Hours

---

# Table of Contents

1. Why JWT Was Created
2. What is JWT?
3. JWT Architecture
4. JWT Structure
5. Header
6. Payload
7. Signature
8. JWT Generation Process
9. JWT Verification Process
10. Signed vs Encrypted Tokens
11. JWT Claims
12. JWT Lifecycle
13. Stateless Authentication
14. Common Mistakes
15. Interview Questions
16. Cheat Sheet

---

# 1. Why JWT Was Created

Before JWT, most applications used Sessions.

```
Browser

↓

Session ID

↓

Server

↓

Session Store

↓

User
```

Problem

If we have

```
10 Servers
```

all servers must access the same session store.

```
Server A

↓

Redis

↑

Server B

↑

Server C
```

More infrastructure is needed.

---

JWT solves this.

Instead of storing user data on the server,

the server stores it **inside the token**.

```
Browser

↓

JWT

↓

Server

↓

Verify Signature

↓

Done
```

No database lookup is required for authentication.

---

# 2. What is JWT?

JWT stands for

```
JSON Web Token
```

It is a compact, URL-safe token used to securely transmit information between two parties.

A JWT is:

- Self-contained
- Digitally signed
- Stateless
- Easy to transmit

---

Example

```
eyJhbGciOiJIUzI1Ni...

eyJzdWIiOjQ1...

P8c1JrF...
```

Looks random,

but internally it has three parts.

---

# 3. JWT Architecture

```
Browser

↓

Login

↓

Node.js

↓

Verify Password

↓

Generate JWT

↓

Browser Stores JWT

↓

Future Requests

↓

Authorization:

Bearer JWT

↓

Verify Signature

↓

Return Data
```

---

# 4. JWT Structure

A JWT has three parts.

```
Header

.

Payload

.

Signature
```

Example

```
xxxxx.yyyyy.zzzzz
```

Think of it as

```
Envelope

↓

Letter

↓

Seal
```

---

Diagram

```
┌──────────┐

 Header

└──────────┘

      .

┌──────────┐

 Payload

└──────────┘

      .

┌──────────┐

Signature

└──────────┘
```

---

# 5. Header

The Header describes

- Token type
- Signing algorithm

Example

```json
{
    "alg":"HS256",
    "typ":"JWT"
}
```

Meaning

```
Algorithm

↓

HS256

Token Type

↓

JWT
```

The header is Base64URL encoded.

---

Encoded Header

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

---

# 6. Payload

The payload contains claims.

Example

```json
{
    "sub":"45",
    "name":"John",
    "role":"Admin",
    "exp":1750000000
}
```

Important

Payload is

```
NOT ENCRYPTED
```

Anyone can decode it.

Never store

❌ Password

❌ Credit Card

❌ API Keys

inside a JWT.

---

Payload Diagram

```
User

↓

ID

Role

Email

Expiry

↓

Payload
```

---

# 7. Signature

This is the most important part.

Signature ensures

```
Integrity
```

It proves

```
Token Not Modified
```

---

Formula

```
HMACSHA256(

Header

+

Payload

+

Secret Key

)
```

Diagram

```
Header

+

Payload

+

Secret

↓

Hash

↓

Signature
```

---

If someone changes

```
Role

↓

User

to

Admin
```

the signature becomes invalid.

Server rejects the token.

---

# 8. JWT Generation Process

Step 1

User logs in.

```
Email

↓

Password
```

---

Step 2

Server verifies credentials.

```
Database

↓

Password Match

↓

Success
```

---

Step 3

Server creates payload.

```json
{
    "sub":"45",
    "role":"Admin"
}
```

---

Step 4

Header is created.

```json
{
    "alg":"HS256"
}
```

---

Step 5

Header + Payload

↓

Base64URL Encode

↓

Sign

↓

JWT Created

---

Final Token

```
xxxxx.yyyyy.zzzzz
```

Returned to browser.

---

# Node.js Example

```javascript
const jwt = require("jsonwebtoken");

const token = jwt.sign(

    {
        id:45,
        role:"Admin"
    },

    process.env.JWT_SECRET,

    {
        expiresIn:"15m"
    }

);
```

---

# 9. JWT Verification Process

Every protected request sends

```
Authorization:

Bearer JWT
```

---

Verification

```
Request

↓

Extract JWT

↓

Decode Header

↓

Decode Payload

↓

Generate Signature Again

↓

Compare Signatures

↓

Match?

↓

YES

↓

Authenticated
```

---

Node.js Example

```javascript
const decoded = jwt.verify(

    token,

    process.env.JWT_SECRET

);
```

If signature doesn't match

```
401 Unauthorized
```

---

# 10. Signed vs Encrypted

One of the most common interview questions.

JWT is

```
SIGNED

NOT

ENCRYPTED
```

---

Signed

```
Anyone Can Read

↓

Nobody Can Modify
```

---

Encrypted

```
Nobody Can Read

↓

Need Key To Decrypt
```

JWT provides integrity,

not confidentiality.

---

Analogy

```
Letter

↓

Transparent Envelope

↓

Wax Seal
```

Everyone can read the letter,

but if someone opens or changes it,

the seal breaks.

---

# 11. JWT Claims

Claims are pieces of information stored in the payload.

---

## Registered Claims

| Claim | Meaning |
|--------|----------|
| sub | Subject (User ID) |
| iss | Issuer |
| aud | Audience |
| exp | Expiration Time |
| iat | Issued At |
| nbf | Not Before |
| jti | JWT ID |

---

Example

```json
{
    "sub":"45",
    "iss":"my-api",
    "aud":"web-app",
    "exp":1750000000,
    "iat":1749990000
}
```

---

Custom Claims

```json
{
    "role":"Admin",
    "department":"HR",
    "tenant":"CompanyA"
}
```

---

# 12. JWT Lifecycle

```
Login

↓

Generate JWT

↓

Store in Browser

↓

API Request

↓

Verify JWT

↓

Return Response

↓

Expire

↓

Login Again

or

Refresh Token
```

---

# 13. Stateless Authentication

With Sessions

```
Token

↓

Database

↓

User
```

With JWT

```
Token

↓

Verify Signature

↓

User Information Already Inside
```

No lookup required for authentication.

This makes JWT ideal for

- REST APIs
- Microservices
- Cloud
- Kubernetes

---

# Security Considerations

Never store

❌ Password

❌ PAN Card

❌ Aadhaar Number

❌ Secret Keys

inside JWT.

Always

✅ Use HTTPS

✅ Use short expiration

✅ Validate issuer and audience

✅ Rotate signing keys periodically

---

# Common Mistakes

❌ Storing sensitive information in payload.

❌ Using very long expiration times.

❌ Accepting tokens without verifying signatures.

❌ Using weak secret keys.

❌ Forgetting to validate `exp`.

---

# Best Practices

✅ Use HS256 or RS256.

✅ Keep access tokens short-lived (10–15 minutes).

✅ Store only minimal claims.

✅ Validate every incoming token.

✅ Keep signing keys secure.

---

# Senior Interview Questions

## Beginner

- What is JWT?
- Why was JWT created?
- What are the three parts of JWT?

---

## Intermediate

- Difference between Header, Payload, and Signature?
- Why is JWT stateless?
- Why is JWT signed instead of encrypted?

---

## Advanced

- Explain JWT verification internally.
- Why can anyone decode a JWT?
- Can a user change the payload?
- What happens if the payload changes?
- Why shouldn't passwords be stored in JWT?
- Why is JWT suitable for microservices?

---

# Complete JWT Flow

```
Browser

↓

POST /login

↓

Node.js

↓

Verify Password

↓

Generate JWT

↓

Browser Stores JWT

↓

Authorization:

Bearer JWT

↓

Node.js

↓

Verify Signature

↓

Read Claims

↓

Protected Resource
```

---

# Cheat Sheet

| Component | Purpose |
|-----------|----------|
| Header | Algorithm & Token Type |
| Payload | Claims |
| Signature | Prevent Tampering |
| HS256 | Symmetric Signing |
| RS256 | Asymmetric Signing |
| exp | Expiration |
| sub | User ID |
| iss | Issuer |
| aud | Audience |

---

# Key Takeaways

✅ JWT is a **self-contained, stateless authentication token**.

✅ A JWT has three parts: **Header**, **Payload**, and **Signature**.

✅ The **Header** specifies the signing algorithm, the **Payload** contains claims, and the **Signature** ensures integrity.

✅ JWTs are **signed, not encrypted**, so their contents are readable but cannot be modified without invalidating the signature.

✅ Authentication with JWT requires **no server-side session lookup**, making it highly scalable for REST APIs and microservices.

✅ JWT payloads should contain only the minimum information necessary and should never include sensitive data.
