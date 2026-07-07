# 📘 Authentication Handbook

# Part 5 — Access Token & Refresh Token Deep Dive




# Table of Contents

1. Introduction
2. Traditional Session Authentication
3. Problems with Sessions
4. JWT Authentication
5. Problems with JWT
6. Why Access Tokens Expire
7. Why Refresh Tokens Exist
8. Access Token
9. Refresh Token
10. Two Token Architecture
11. Login Flow
12. Storage Strategy
13. API Request Flow
14. Silent Token Refresh
15. Production Authentication Flow
16. Best Practices
17. Common Mistakes
18. Interview Questions
19. Cheat Sheet
20. Summary

---

# 1. Introduction

Modern web applications rarely rely on session-based authentication anymore.

Applications such as:

- Google
- GitHub
- Microsoft
- Amazon
- Netflix

use **Access Tokens** and **Refresh Tokens** to provide both:

- Better security
- Better user experience

This chapter explains how this architecture works internally.

---

# 2. Traditional Session Authentication

Before JWT became popular, authentication relied on server-side sessions.

## Flow

```text
Browser
   │
Login
   │
   ▼
Server
   │
Create Session
   │
Store Session
   │
Return Session ID
```

Browser stores

```text
Cookie

SESSION_ID = abc123
```

Every request

```text
Browser
   │
Cookie
   │
   ▼
Server
   │
Lookup Session
   │
Find User
   │
Return Response
```

---

## Advantages

- Simple
- Secure
- Easy to invalidate
- Session controlled by server

---

## Disadvantages

- Requires server memory
- Difficult to scale
- Multiple servers require Redis
- Session replication required

---

# 3. Problems with Sessions

Imagine your application has three servers.

```text
             Load Balancer
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
   Server A     Server B     Server C
```

If the session exists only on Server A:

```text
Request

↓

Server B

↓

Session Not Found

↓

User Logged Out
```

Solution:

```text
Redis

↓

Shared Session Store

↓

All Servers Access Same Session
```

Although Redis solves the problem, maintaining session infrastructure increases complexity.

---

# 4. JWT Authentication

JWT (JSON Web Token) stores authentication information inside the token itself.

Instead of storing user information on the server:

```text
Client
   │
JWT
   │
   ▼
Server
   │
Verify Signature
   │
Authenticate User
```

This approach is called **Stateless Authentication**.

---

# 5. Problems with JWT

Suppose a JWT never expires.

```text
JWT

↓

Valid Forever
```

If a hacker steals it:

```text
Attacker

↓

Uses Token

↓

Unlimited Access
```

This creates a serious security vulnerability.

---

# 6. Why Access Tokens Expire

To reduce damage from stolen tokens.

Typical expiration:

| Application | Access Token Lifetime |
|-------------|----------------------|
| Banking | 5 minutes |
| Enterprise Apps | 15 minutes |
| SaaS | 15–30 minutes |
| Social Media | 15–60 minutes |

Short-lived tokens reduce attack windows.

---

# 7. Why Refresh Tokens Exist

Imagine a user watching Netflix.

```text
Login

↓

Watching Movie

↓

15 Minutes

↓

Access Token Expired

↓

User Logged Out
```

This is a poor user experience.

Instead:

```text
Access Token Expires

↓

Refresh Token Generates New Access Token

↓

User Continues Without Logging In Again
```

---

# 8. Access Token

## Purpose

Authenticate API requests.

---

## Used For

- Protected APIs
- Authorization
- User Identity

---

## Example

```http
GET /api/users

Authorization: Bearer eyJhbGciOiJIUzI1NiIsIn...
```

---

## Example Payload

```json
{
    "sub": "1001",
    "email": "john@gmail.com",
    "role": "Admin",
    "exp": 1727000000
}
```

---

## Lifetime

Typical values:

- 5 Minutes
- 10 Minutes
- 15 Minutes
- 30 Minutes

Never:

- 30 Days
- 90 Days
- 1 Year

---

## Advantages

- Lightweight
- Stateless
- Fast
- Easy to verify

---

## Disadvantages

- Cannot be revoked easily
- Must expire quickly

---

# 9. Refresh Token

Purpose:

Generate new Access Tokens.

It is **NOT** used to access protected APIs.

Incorrect:

```text
Refresh Token

↓

GET /users
```

Correct:

```text
Refresh Token

↓

POST /refresh

↓

New Access Token

↓

GET /users
```

---

## Typical Lifetime

- 7 Days
- 15 Days
- 30 Days
- 90 Days

Depends on business requirements.

---

# 10. Two Token Architecture

```text
                 Login
                   │
       ┌───────────┴───────────┐
       ▼                       ▼
Access Token             Refresh Token
```

---

## Responsibilities

### Access Token

- Authentication
- Sent on every request
- Short-lived

### Refresh Token

- Generate new access token
- Long-lived
- Stored securely
- Used occasionally

---

# 11. Complete Login Flow

```text
User

↓

Login

↓

Authentication Server

↓

Verify Username & Password

↓

Generate Access Token

↓

Generate Refresh Token

↓

Return Both Tokens
```

Example response:

```json
{
    "accessToken": "abc123",
    "refreshToken": "xyz789"
}
```

---

# 12. Storage Strategy

## ❌ Bad Practice

```text
localStorage

├── Access Token

└── Refresh Token
```

If an XSS attack occurs:

```text
JavaScript

↓

Read localStorage

↓

Steal Refresh Token

↓

Unlimited Login
```

---

## ✅ Recommended

```text
Access Token

↓

Memory

or

Short-lived Cookie



Refresh Token

↓

HttpOnly Secure Cookie
```

Benefits:

- JavaScript cannot access HttpOnly cookies
- Better protection against XSS

---

# 13. API Request Flow

```text
React

↓

Axios

↓

Access Token

↓

Node API

↓

Is Token Valid?

├── Yes

│      ↓

│   Return Response

│

└── No

       ↓

   Return 401
```

---

# 14. Silent Token Refresh

Axios interceptor detects:

```text
401 Unauthorized
```

Flow:

```text
401

↓

POST /refresh

↓

Validate Refresh Token

↓

Generate New Access Token

↓

Retry Original Request

↓

Return Response
```

User never notices the refresh.

---

# 15. Production Authentication Flow

```text
                User Logs In
                     │
                     ▼
             Authentication API
                     │
      ┌──────────────┴──────────────┐
      ▼                             ▼
 Generate Access Token       Generate Refresh Token
      │                             │
      ▼                             ▼
 React Memory              HttpOnly Secure Cookie
      │
      ▼
 Protected API Request
      │
      ▼
Access Token Expired?
      │
  ┌───┴────┐
  │        │
 No       Yes
  │        │
  ▼        ▼
Response   Call /refresh
           │
           ▼
Validate Refresh Token
      │
 ┌────┴────┐
 │         │
Valid   Invalid
 │         │
 ▼         ▼
Issue New  Force Login
Access Token
      │
      ▼
Retry Original Request
```

---

# 16. Best Practices

✅ Keep access tokens short-lived.

✅ Store refresh tokens in HttpOnly Secure cookies.

✅ Always use HTTPS.

✅ Rotate refresh tokens.

✅ Revoke refresh tokens on logout.

✅ Store refresh tokens hashed in the database.

✅ Detect refresh token reuse.

---

# 17. Common Mistakes

❌ Long-lived access tokens

❌ Refresh token in localStorage

❌ Not rotating refresh tokens

❌ No logout endpoint

❌ No token revocation

❌ Sending refresh token with every request

❌ Not validating JWT signature

---

# 18. Interview Questions

## Beginner

1. What is JWT?
2. Why do access tokens expire?
3. What is a refresh token?
4. Why are two tokens used?
5. What is the purpose of the `exp` claim?

---

## Intermediate

1. Why should refresh tokens not be sent with every API request?
2. Why store refresh tokens in HttpOnly cookies?
3. How do Axios interceptors perform silent refresh?
4. What happens when both tokens expire?
5. How does JWT remain stateless?

---

## Senior

1. How would you implement authentication for millions of users?
2. How do you revoke JWTs?
3. How do you support multiple device logins?
4. How does refresh token rotation work?
5. How do Google and Microsoft detect stolen refresh tokens?

---

# 19. Cheat Sheet

| Access Token | Refresh Token |
|--------------|---------------|
| Short-lived | Long-lived |
| Every Request | Refresh Endpoint Only |
| Authentication | Generate New Access Token |
| Expires Quickly | Expires Slowly |
| Usually Stored in Memory | HttpOnly Cookie |

---

# 20. Summary

- Sessions require server storage.
- JWT enables stateless authentication.
- Access tokens are short-lived for security.
- Refresh tokens improve user experience.
- Access tokens authenticate requests.
- Refresh tokens generate new access tokens.
- Refresh tokens should be stored securely in HttpOnly cookies.
- Silent refresh provides seamless authentication.
- Production applications use both tokens together.

---

# Next Chapter

## Part 6 — Refresh Token Rotation (Advanced)

Topics include:

- Refresh Token Rotation
- One-Time Refresh Tokens
- Refresh Token Families
- Reuse Detection
- Multi-Device Login
- Device Fingerprinting
- Logout from One Device
- Logout from All Devices
- Database Schema Design
- Redis vs Database Storage
- React + Node.js + Express Implementation
- Security Best Practices
- 50+ Advanced Interview Questions
