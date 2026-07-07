# Chapter 8 - Authentication & Authorization
# Part 1 - Authentication Fundamentals

> Level: Advanced
> Interview Importance: ⭐⭐⭐⭐⭐
> Estimated Study Time: 8 Hours

---

# Table of Contents

1. Introduction
2. Authentication vs Authorization
3. Identity in Web Applications
4. Authentication Methods
5. Authentication Flow
6. Why Authentication is Required
7. Stateless vs Stateful Authentication
8. Real-world Authentication Architecture
9. Common Authentication Terminology
10. Best Practices
11. Interview Questions
12. Cheat Sheet

---

# 1. Introduction

Almost every application needs to answer one question:

```
Who is making this request?
```

For example,

```
GET /api/orders
```

How does the server know whether the request belongs to

- User A
- User B
- Admin
- Hacker

Authentication solves this problem.

---

Imagine a Bank.

```
Customer

↓

ATM

↓

Insert ATM Card

↓

Enter PIN

↓

Identity Verified

↓

Withdraw Money
```

The ATM first verifies

```
WHO YOU ARE
```

Only then does it allow transactions.

Web applications work exactly the same way.

---

# Authentication Flow

```
User

↓

Login

↓

Verify Credentials

↓

Create Identity

↓

Future Requests

↓

Identity Verification
```

---

# 2. Authentication vs Authorization

This is one of the most common interview questions.

Many developers confuse these two.

---

## Authentication

Authentication answers

```
Who are you?
```

Example

```
Username

+

Password
```

↓

Server verifies

↓

User identified

---

Example

```
Login

↓

Email

↓

Password

↓

JWT

↓

Authenticated
```

---

## Authorization

Authorization answers

```
What are you allowed to do?
```

Example

```
Admin

↓

Delete User

↓

Allowed
```

Customer

↓

Delete User

↓

Denied

---

Visual

```
Authentication

↓

Identity

↓

Authorization

↓

Permissions
```

---

# Easy Way to Remember

Authentication

```
Identity Card
```

Authorization

```
Access Card
```

Example

```
Airport

↓

Passport Check

(Authentication)

↓

Boarding Gate

↓

Boarding Pass

(Authorization)
```

---

# Real Example

User logs in.

```
Email

↓

Password

↓

JWT Generated
```

Now

```
GET /users

↓

Allowed?
```

Depends on

```
Role

Permission
```

Not Authentication.

---

# 3. Identity in Web Applications

A server needs a unique identity.

Usually

```
User ID

Role

Email

Tenant

Permissions
```

Example

```
{

id: 45,

email:"john@gmail.com",

role:"Admin"

}
```

This becomes the authenticated identity.

---

# 4. Authentication Methods

There are several ways to authenticate users.

---

## Method 1

Username + Password

```
User

↓

Login

↓

Database

↓

Password Match

↓

Login Success
```

Still the most common method.

---

## Method 2

OTP

```
Phone

↓

SMS

↓

OTP

↓

Verify

↓

Login
```

Used in

- Banking
- UPI
- E-commerce

---

## Method 3

OAuth

```
Login With Google

↓

Google Verifies

↓

Returns Identity
```

Examples

- Google Login
- GitHub Login
- Microsoft Login

---

## Method 4

Magic Link

```
Email

↓

Link

↓

Click

↓

Login
```

No password required.

---

## Method 5

Biometric

```
Fingerprint

↓

Face Recognition

↓

Login
```

Mostly mobile applications.

---

# Comparison

| Method | Security | User Experience |
|----------|----------|----------------|
| Password | Medium | Medium |
| OTP | High | Good |
| OAuth | High | Excellent |
| Magic Link | High | Excellent |
| Biometric | Very High | Excellent |

---

# 5. Authentication Flow

Let's see what happens internally.

```
Browser

↓

POST /login

↓

Node.js

↓

Check Email

↓

Check Password

↓

Generate Token

↓

Return Token

↓

Browser Stores Token

↓

Future API Calls

↓

Verify Token

↓

Response
```

---

# Detailed Flow

```
Login Request

↓

Express Route

↓

User Service

↓

Database

↓

User Found?

↓

No

↓

401 Unauthorized

-------------------

Yes

↓

Compare Password

↓

Correct?

↓

No

↓

401

-------------------

Yes

↓

Generate JWT

↓

Send Response
```

---

# 6. Why Authentication is Required

Without authentication

```
GET /bank-balance

↓

Anyone Can Access
```

Very dangerous.

---

With Authentication

```
GET /bank-balance

↓

Verify Identity

↓

Return Only User Data
```

---

Real Example

```
Facebook

↓

Profile API

↓

Need Login

↓

Otherwise

401 Unauthorized
```

---

# 7. Stateless vs Stateful Authentication

One of the most important interview topics.

---

## Stateful Authentication

Server stores login session.

```
Browser

↓

Session ID

↓

Server Memory

↓

User Data
```

The server remembers every logged-in user.

---

Advantages

- Easy logout
- Session invalidation
- Traditional applications

Disadvantages

- More memory
- Harder to scale

---

## Stateless Authentication

Server stores nothing.

```
Browser

↓

JWT

↓

Every Request

↓

Verify Signature

↓

Done
```

No session lookup.

---

Advantages

- Highly scalable
- Great for APIs
- Microservices
- Cloud

Disadvantages

- Logout is more complex
- Token revocation requires extra mechanisms

---

Comparison

| Stateful | Stateless |
|-----------|------------|
| Session | JWT |
| Server Memory | No Memory |
| Easy Logout | Harder Logout |
| Scaling Difficult | Scaling Easy |
| Traditional Apps | REST APIs |

---

# 8. Real-world Authentication Architecture

```
                Browser

                    │

                    ▼

               Login Request

                    │

                    ▼

              Express API

                    │

          ┌─────────┴─────────┐

          ▼                   ▼

     User Database      Password Verify

          │                   │

          └─────────┬─────────┘

                    ▼

             JWT Generation

                    │

                    ▼

            Access Token

                    │

                    ▼

              Future Requests

                    │

                    ▼

             JWT Verification

                    │

                    ▼

               Protected APIs
```

---

# 9. Common Authentication Terminology

| Term | Meaning |
|-------|----------|
| Identity | Who the user is |
| Credentials | Username, Password, OTP |
| Principal | Authenticated user |
| Token | Proof of identity |
| Session | Server-side login state |
| Access Token | Short-lived authentication token |
| Refresh Token | Used to obtain new access tokens |
| Claims | Information stored inside a JWT |

---

# HTTP Status Codes

| Code | Meaning |
|------|----------|
| 200 | Success |
| 201 | Resource Created |
| 400 | Bad Request |
| 401 | Authentication Failed |
| 403 | Authenticated but Not Authorized |
| 404 | Resource Not Found |

---

# 10. Best Practices

✅ Always use HTTPS.

✅ Never store plain text passwords.

✅ Use password hashing.

✅ Use short-lived access tokens.

✅ Validate every protected request.

✅ Separate authentication from authorization.

✅ Never trust client-side role information.

---

# Common Mistakes

❌ Authentication == Authorization.

❌ Trusting user role from frontend.

❌ Sending passwords in plain text.

❌ Storing passwords without hashing.

❌ Returning detailed login errors.

Instead of

```
Password Incorrect
```

Return

```
Invalid Credentials
```

---

# Senior Interview Questions

## Beginner

- What is Authentication?
- What is Authorization?
- Difference between Authentication and Authorization?

---

## Intermediate

- Stateful vs Stateless Authentication?
- Why do APIs prefer JWT?
- Why should passwords never be stored directly?

---

## Advanced

- Design authentication for a banking application.
- Explain login flow for a React + Node.js application.
- Why do microservices prefer stateless authentication?
- How would you authenticate WebSocket connections?
- How would you support multiple devices per user?

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Authentication | Verify identity |
| Authorization | Verify permissions |
| Credentials | Username, Password, OTP |
| Session | Server-side login |
| JWT | Stateless authentication |
| Identity | User information |
| Claims | Data inside token |
| 401 | Authentication failed |
| 403 | Permission denied |

---

# Complete Authentication Flow

```
User

↓

Login Page

↓

POST /login

↓

Node.js API

↓

Database

↓

Verify Password

↓

Generate JWT

↓

Browser Stores Token

↓

Future API Calls

↓

JWT Verification

↓

Protected Resource
```

---

# Key Takeaways

✅ Authentication verifies **who the user is**, while Authorization determines **what the user is allowed to do**.

✅ Authentication should always happen **before** authorization.

✅ Stateless authentication (JWT) is preferred for modern REST APIs and microservices because it scales well.

✅ Stateful authentication (sessions) is still widely used in traditional web applications where server-side session management is acceptable.

✅ A secure authentication system includes password hashing, HTTPS, short-lived access tokens, proper error handling, and validation on every protected request.

✅ Understanding the distinction between identity, credentials, tokens, sessions, and permissions is fundamental for designing secure backend systems.
