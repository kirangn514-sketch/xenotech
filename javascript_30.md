# Chapter 8 - Authentication & Authorization
# Part 4 - JWT Internals, Cryptography & Signing Algorithms

> Level: Expert
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 10 Hours

---

# Table of Contents

1. Why Cryptography is Needed
2. JWT Signing Process
3. Hashing vs Encryption vs Encoding
4. HMAC (HS256)
5. Public Key Cryptography
6. RS256
7. ES256
8. JWT Verification Internals
9. Key Rotation
10. Common JWT Attacks
11. Production Best Practices
12. Interview Questions
13. Cheat Sheet

---

# 1. Why Cryptography is Needed

Imagine a JWT without a signature.

Payload

```json
{
    "id": 15,
    "role": "User"
}
```

An attacker can decode it.

```
JWT

↓

Decode

↓

Change

role

↓

Admin

↓

Encode Again
```

Now the payload becomes

```json
{
    "id":15,
    "role":"Admin"
}
```

Without cryptography,

the server has no way to know the payload was modified.

This is why JWT includes a **digital signature**.

---

# Digital Signature

```
Header

+

Payload

+

Secret

↓

Signature
```

Whenever Header or Payload changes,

```
Signature Changes
```

Server immediately detects tampering.

---

# 2. JWT Signing Process

Suppose

Header

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```

Payload

```json
{
  "sub":"45",
  "role":"Admin"
}
```

Step 1

Convert JSON

↓

Base64URL Encode

```
Header

↓

eyJhbGc...
```

```
Payload

↓

eyJzdWI...
```

---

Step 2

Concatenate

```
Header

.

Payload
```

Result

```
xxxxx.yyyyy
```

---

Step 3

Apply HMAC

```
Header.Payload

+

Secret Key

↓

HMACSHA256()

↓

Signature
```

---

Step 4

Final JWT

```
Header

.

Payload

.

Signature
```

---

# Visual

```
Header

↓

Encode

↓

HeaderEncoded

+

PayloadEncoded

↓

HMAC

↓

Signature

↓

JWT
```

---

# 3. Hashing vs Encryption vs Encoding

Interviewers love this question.

---

## Encoding

Purpose

```
Readable Format
```

Example

```
JSON

↓

Base64URL
```

Encoding is reversible.

Anyone can decode.

---

## Encryption

Purpose

```
Hide Data
```

```
Plain Text

↓

Encryption

↓

Cipher Text

↓

Decryption

↓

Original Data
```

Requires encryption key.

---

## Hashing

Purpose

```
Integrity
```

```
Input

↓

SHA256

↓

Hash
```

Cannot recover original data.

One-way.

---

Comparison

| Feature | Encoding | Encryption | Hashing |
|-----------|-----------|------------|----------|
| Reversible | Yes | Yes | No |
| Hides Data | No | Yes | No |
| Detect Changes | No | No | Yes |
| JWT Uses | Base64URL | Rarely | Signature |

---

# 4. HMAC (HS256)

HS256 means

```
HMAC

+

SHA256
```

HMAC

```
Hash-based Message Authentication Code
```

---

Signing Formula

```
Signature =

HMACSHA256(

Header.Payload,

SecretKey

)
```

---

Visual

```
Header.Payload

↓

Secret

↓

HMAC

↓

Signature
```

---

Important

```
Same Secret

↓

Sign

↓

Verify
```

The server must use the **same secret**.

---

Advantages

- Fast
- Simple
- Excellent performance

Disadvantages

Everyone verifying the token also needs the secret.

---

# HS256 Architecture

```
Server

↓

Secret

↓

Generate JWT

↓

Client

↓

JWT

↓

Another Server

↓

Needs Same Secret

↓

Verify
```

---

# 5. Public Key Cryptography

Instead of one secret,

we use

```
Private Key

+

Public Key
```

Like a lock.

```
Private Key

↓

Sign

↓

Public Key

↓

Verify
```

---

Advantages

Verifier never knows the private key.

Only public key is shared.

---

# 6. RS256

RS256

```
RSA

+

SHA256
```

Architecture

```
Authentication Server

↓

Private Key

↓

Sign JWT

↓

JWT

↓

API Server

↓

Public Key

↓

Verify JWT
```

API servers never know the private key.

---

Comparison

| HS256 | RS256 |
|--------|--------|
| One Secret | Public + Private Keys |
| Faster | Slightly Slower |
| Simpler | More Secure for Large Systems |
| Small Apps | Enterprise Systems |

---

Why Google uses RS256

```
Google

↓

Private Key

↓

Millions of APIs

↓

Public Key

↓

Verify
```

Only Google can issue tokens.

Everyone can verify them.

---

# 7. ES256

Uses

```
Elliptic Curve Cryptography
```

Advantages

- Smaller keys
- Faster verification
- Smaller JWT
- Better performance

Used by many modern identity providers.

---

Comparison

| Algorithm | Security | Speed | Token Size |
|-----------|----------|-------|------------|
| HS256 | High | Fastest | Small |
| RS256 | Very High | Moderate | Larger |
| ES256 | Very High | Fast | Smallest |

---

# 8. JWT Verification Internals

Client sends

```
Authorization

Bearer JWT
```

Server performs

```
Split JWT

↓

Header

↓

Payload

↓

Signature
```

---

Recalculate Signature

```
Header.Payload

+

Secret

↓

HMAC

↓

New Signature
```

Compare

```
Incoming Signature

=

Generated Signature ?
```

---

YES

↓

Authenticated

---

NO

↓

401 Unauthorized

---

# Verification Diagram

```
JWT

↓

Split

↓

Header

Payload

Signature

↓

Generate New Signature

↓

Compare

↓

Valid?
```

---

# Why Database Lookup is Not Required

Payload already contains

```
User ID

Role

Permissions

Expiry
```

Verification only requires

```
Secret

or

Public Key
```

No session lookup.

---

# 9. Key Rotation

Secrets should never remain forever.

Old Secret

```
Secret A
```

↓

Rotate

↓

```
Secret B
```

New JWTs use Secret B.

Old tokens expire naturally.

---

Using kid

Header

```json
{
    "alg":"RS256",
    "kid":"key-2026"
}
```

Server

```
kid

↓

Find Matching Public Key

↓

Verify
```

Supports multiple keys during rotation.

---

# 10. Common JWT Attacks

## alg:none Attack

Attacker changes

```json
{
   "alg":"none"
}
```

Poor implementations may skip verification.

Always reject `alg:none`.

---

## Secret Brute Force

Weak secret

```
password123
```

Attackers can guess it.

Use

```
256-bit random secret
```

---

## Token Theft

JWT stolen via XSS.

Attacker sends

```
Bearer JWT
```

Server accepts it.

Prevent with

- HttpOnly Cookies
- CSP
- HTTPS

---

## Long Expiration

```
365 Days
```

Stolen token remains valid.

Use

```
10–15 Minutes
```

---

# 11. Production Best Practices

✅ Use RS256 for distributed systems.

✅ Use HS256 for small internal applications.

✅ Keep secrets outside source code.

```
.env

↓

Vault

↓

AWS Secrets Manager
```

✅ Rotate keys regularly.

✅ Validate

- exp
- iss
- aud
- nbf

✅ Reject malformed tokens.

---

# Common Mistakes

❌ Storing JWT secret in GitHub.

❌ Using weak passwords as secrets.

❌ Ignoring expiration.

❌ Accepting any algorithm.

❌ Using JWT as encrypted storage.

---

# Senior Interview Questions

## Beginner

- What is HS256?
- What is RS256?
- Why is JWT signed?

---

## Intermediate

- Difference between hashing and encryption?
- Why can't users modify JWT payload?
- Why does JWT not need database lookup?

---

## Advanced

- Explain HMAC internally.
- Why does Google use RS256?
- What is key rotation?
- Explain the `kid` header.
- How would you secure JWT in a microservices architecture?
- What are common JWT attacks?

---

# Complete Cryptography Flow

```
Header

↓

Base64URL

↓

Payload

↓

Base64URL

↓

Header.Payload

↓

Secret / Private Key

↓

Digital Signature

↓

JWT

↓

Client

↓

Server

↓

Verify Signature

↓

Authenticated
```

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Base64URL | Encoding format used in JWT |
| HMAC | Symmetric signing algorithm |
| HS256 | HMAC + SHA256 |
| RS256 | RSA + SHA256 |
| ES256 | Elliptic Curve + SHA256 |
| Private Key | Used to sign |
| Public Key | Used to verify |
| kid | Key Identifier |
| Signature | Prevents tampering |

---

# Key Takeaways

✅ JWT integrity is provided through **digital signatures**, not encryption.

✅ HS256 uses a **shared secret** for both signing and verification, while RS256 uses a **private key to sign** and a **public key to verify**.

✅ Base64URL encoding is **not encryption**—anyone can decode the Header and Payload.

✅ During verification, the server recomputes the signature and compares it with the incoming signature. If they differ, the token has been modified or forged.

✅ Key rotation with the `kid` header allows systems to replace signing keys without immediately invalidating all existing tokens.

✅ Production-grade JWT implementations validate **signature, expiration (`exp`), issuer (`iss`), audience (`aud`), and algorithm**, while keeping secrets secure and rotating keys regularly.
