# JSON Web Tokens (JWT): Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Understanding Session Tokens vs JWTs](#understanding-session-tokens-vs-jwts)
3. [What are JSON Web Tokens?](#what-are-json-web-tokens)
4. [JWT Structure Deep Dive](#jwt-structure-deep-dive)
5. [JWT Claims Explained](#jwt-claims-explained)
6. [JWT Signature & Security](#jwt-signature--security)
7. [JWTs in Distributed Systems](#jwts-in-distributed-systems)
8. [Implementation Example (Node.js)](#implementation-example-nodejs)
9. [Best Practices & Security Considerations](#best-practices--security-considerations)
10. [Common Use Cases](#common-use-cases)

## Introduction

JSON Web Token (JWT) is an **open standard (RFC 7519)** that defines a compact and secure way of transmitting information between parties. JWTs are revolutionizing how we think about data transmission and authentication in modern applications by providing a stateless, scalable solution for user authentication and authorization.

> 🎯 **Key Concept**: JWT's primary purpose is **not to hide data** but to **ensure the authenticity** of the data through digital signatures.

## Understanding Session Tokens vs JWTs

### Traditional Session Tokens

**How Session Tokens Work:**
Think of calling customer service - you explain your problem, and at the end, you receive a case number. This case number (session token) identifies your conversation, allowing representatives to quickly access your information on future calls.

**Session Token Characteristics:**
- **Server-dependent**: Only valid on the server that issued them
- **Stateful**: Server must store session information
- **Limited scope**: Cannot be used across different systems
- **Database lookup required**: Server queries database to validate each request

**Example Limitation:**
If you log into your bank's website, the session token issued by the web server won't work with the bank's mobile app because it's a different server system.

### JSON Web Tokens (JWTs)

**The Concert Wristband Analogy:**
JWTs are like wristbands at a concert. Once you buy a ticket, you get a wristband that security guards can instantly verify without checking a database. The wristband itself contains all the necessary information.

**JWT Advantages:**
- **Server-independent**: Can be used across multiple servers and services
- **Stateless**: No server-side session storage required
- **Self-contained**: All necessary information is embedded in the token
- **Cross-domain**: Perfect for distributed systems and microservices

## What are JSON Web Tokens?

JSON Web Tokens are a secure method for transmitting information between parties, commonly used for:

### 🔐 **Authentication**
After successful login, the server issues a JWT containing user identity and permissions. Users include this token in subsequent requests to prove their identity.

### 🌐 **Cross-Service Communication**
In microservices architecture, JWTs enable secure communication between different services without requiring centralized session management.

### 📱 **API Authorization**
Frontend applications use JWTs to authorize API requests, allowing servers to quickly verify user permissions without database lookups.

### Typical JWT Flow:
1. User logs in with credentials
2. Server validates credentials and issues JWT
3. Client stores JWT (usually in localStorage or cookies)
4. Client includes JWT in API request headers
5. API verifies JWT signature and extracts user information
6. API processes request based on user permissions

## JWT Structure Deep Dive

A JWT consists of three parts separated by dots (`.`):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 1. Header (Red)
Contains metadata about the token:
```json
{
  "alg": "HS256",    // Algorithm used for signing
  "typ": "JWT"       // Token type
}
```

**Common Algorithms:**
- **HS256**: HMAC with SHA-256 (symmetric key)
- **RS256**: RSA with SHA-256 (asymmetric key)
- **ES256**: ECDSA with SHA-256 (asymmetric key)

### 2. Payload (Purple)
Contains the claims (statements about the user):
```json
{
  "sub": "1234567890",           // Subject (user ID)
  "name": "John Doe",           // Custom claim
  "iat": 1516239022,            // Issued at
  "exp": 1516242622,            // Expiration time
  "role": "admin"               // Custom claim
}
```

### 3. Signature (Blue)
Ensures token integrity:
```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

## JWT Claims Explained

Claims are statements about an entity (typically the user) and additional metadata.

### Registered Claims (Standard)
These are predefined claims with specific meanings:

| Claim | Full Name | Description | Example |
|-------|-----------|-------------|---------|
| **iss** | Issuer | Who issued the token | `"https://myapp.com"` |
| **sub** | Subject | Who the token is about | `"user123"` |
| **aud** | Audience | Who should accept the token | `"api/myapi"` |
| **exp** | Expiration Time | When token expires | `1609459200` |
| **iat** | Issued At | When token was created | `1609372800` |
| **nbf** | Not Before | Token not valid before this time | `1609372800` |
| **jti** | JWT ID | Unique identifier for the token | `"abc123"` |

### Custom Claims (Private)
Application-specific claims for storing additional information:

```json
{
  "role": "admin",
  "permissions": ["read", "write", "delete"],
  "department": "engineering",
  "subscription_tier": "premium"
}
```

### Public Claims
Claims defined by the community or organization, should be collision-resistant:

```json
{
  "https://myapp.com/claims/user_type": "premium",
  "https://myapp.com/claims/features": ["analytics", "export"]
}
```

## JWT Signature & Security

### 🔒 **The Signature Purpose**
The signature is like a tamper-evident seal. It proves:
- The token hasn't been modified since creation
- The token was issued by a trusted source
- The integrity of the header and payload

### **How Signature Verification Works:**

1. **Token Creation:**
   ```
   signature = HMAC-SHA256(base64(header) + "." + base64(payload), secret)
   ```

2. **Token Verification:**
   ```
   calculated_signature = HMAC-SHA256(received_header + "." + received_payload, secret)
   valid = (calculated_signature === received_signature)
   ```

3. **Tamper Detection:**
   If someone changes the payload (e.g., role from "user" to "admin"), the signature won't match, and verification will fail.

### **Public/Private Key Signing (RS256)**

For distributed systems, asymmetric keys provide enhanced security:

- **Private Key**: Used by authentication server to sign tokens
- **Public Key**: Used by API servers to verify tokens
- **Advantage**: API servers don't need the secret key, only the public key

```javascript
// Authentication Server (signs with private key)
const jwt = sign(payload, privateKey, { algorithm: 'RS256' });

// API Server (verifies with public key)
const decoded = verify(jwt, publicKey, { algorithms: ['RS256'] });
```

## JWTs in Distributed Systems

### Microservices Architecture Benefits

**Before JWTs (Session-based):**
```
User → Service A → Database (session lookup)
User → Service B → Database (session lookup)
User → Service C → Database (session lookup)
```

**With JWTs:**
```
User → Service A (verify JWT locally)
User → Service B (verify JWT locally)
User → Service C (verify JWT locally)
```

### Advantages in Distributed Systems:

1. **Reduced Database Load**: No session lookups required
2. **Scalability**: Services can verify tokens independently
3. **Stateless**: Each service can validate requests without shared state
4. **Cross-Domain**: Single token works across multiple domains/services

### Service-to-Service Authentication:
```json
{
  "iss": "user-service",
  "sub": "service-account",
  "aud": "payment-service",
  "scope": ["process_payments", "read_user_data"],
  "exp": 1609459200
}
```

## Implementation Example (Node.js)

Here's a practical implementation of JWT generation and verification:

### `jwt.js` – Utility for generating & verifying JWTs

```javascript
const jwt = require("jsonwebtoken");

// Example permissions resolver
function getUserPermissions(role) {
  if (role === "admin") return ["read", "write", "delete"];
  if (role === "user") return ["read"];
  return [];
}

function generateJWT(userId, role, secretKey) {
  const payload = {
    iss: "myapp.com",            // Issuer
    sub: String(userId),         // Subject (user ID)
    aud: "api/myapi",            // Audience
    role: role,                  // Custom claim
    permissions: getUserPermissions(role), // Custom claim
  };

  const options = {
    algorithm: "HS256",
    expiresIn: "24h",            // Expiration
  };

  return jwt.sign(payload, secretKey, options);
}

function verifyJWT(token, secretKey) {
  try {
    const decoded = jwt.verify(token, secretKey, { audience: "api/myapi" });
    return decoded;
  } catch (err) {
    if (err.name === "TokenExpiredError") {
      throw new Error("Token has expired");
    }
    throw new Error("Invalid token");
  }
}

module.exports = { generateJWT, verifyJWT };
```

---

### `authMiddleware.js` – Express middleware for protecting routes

```javascript
const { verifyJWT } = require("./jwt");

const secretKey = "your-secret-key"; // Move this to .env in production

function jwtRequired(req, res, next) {
  const authHeader = req.headers["authorization"];

  if (!authHeader) {
    return res.status(401).json({ message: "Token is missing!" });
  }

  const token = authHeader.split(" ")[1]; // Remove 'Bearer '
  if (!token) {
    return res.status(401).json({ message: "Invalid Authorization header format" });
  }

  try {
    const decoded = verifyJWT(token, secretKey);
    req.user = decoded; // attach user to request
    next();
  } catch (err) {
    return res.status(401).json({ message: err.message });
  }
}

module.exports = jwtRequired;
```

### `server.js` – Example usage with Express

```javascript
const express = require("express");
const { generateJWT } = require("./jwt");
const jwtRequired = require("./authMiddleware");

const app = express();
const PORT = 3000;
const secretKey = "your-secret-key"; // should be in process.env

// Example login route (mock)
app.get("/login", (req, res) => {
  const token = generateJWT(123, "admin", secretKey);
  res.json({ token });
});

// Protected route
app.get("/api/protected", jwtRequired, (req, res) => {
  res.json({
    user_id: req.user.sub,
    role: req.user.role,
    permissions: req.user.permissions,
    message: "This is a protected endpoint",
  });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### 🔑 Usage

1. Start server:

   ```bash
   node server.js
   ```
2. Get a token:

   ```
   GET http://localhost:3000/login
   ```
3. Call protected endpoint with header:

   ```
   Authorization: Bearer <token>
   ```

---

## Best Practices & Security Considerations

### ✅ **Do's:**

1. **Use HTTPS Always**: JWTs should only be transmitted over secure connections
2. **Set Short Expiration Times**: Limit token lifetime (15-60 minutes for access tokens)
3. **Implement Refresh Tokens**: Use long-lived refresh tokens to get new access tokens
4. **Validate All Claims**: Check issuer, audience, and expiration
5. **Use Strong Secrets**: Minimum 256 bits for HMAC algorithms
6. **Implement Token Blacklisting**: Maintain a list of revoked tokens for critical applications

### ❌ **Don'ts:**

1. **Store Sensitive Data**: Never include passwords, SSNs, or sensitive personal information
2. **Use for Session Storage**: JWTs aren't suitable for storing large session data
3. **Ignore Expiration**: Always check and respect the `exp` claim
4. **Use Weak Algorithms**: Avoid `none` algorithm and weak keys
5. **Trust Client-Side Tokens**: Always verify tokens on the server

### 🔐 **Security Considerations:**

**Token Storage:**
```javascript
// ❌ Bad: Vulnerable to XSS
localStorage.setItem('jwt', token);

// ✅ Better: HttpOnly cookies (CSRF protection needed)
// Set-Cookie: token=jwt; HttpOnly; Secure; SameSite=Strict
```

**Token Refresh Pattern:**
```javascript
// Short-lived access token (15 min) + long-lived refresh token (7 days)
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "expires_in": 900
}
```

## Common Use Cases

### 1. **Single Sign-On (SSO)**
One JWT allows access to multiple applications:
```json
{
  "sub": "user123",
  "aud": ["app1.com", "app2.com", "app3.com"],
  "scp": ["read", "write"]
}
```

### 2. **API Gateway Authentication**
API gateway validates JWT and forwards requests:
```
Client → API Gateway (verify JWT) → Microservice
```

### 3. **Mobile App Authentication**
Mobile apps store JWTs and include them in API calls:
```javascript
// Mobile app request
fetch('/api/data', {
  headers: {
    'Authorization': `Bearer ${jwt}`
  }
});
```

### 4. **Inter-Service Communication**
Services authenticate each other using JWTs:
```json
{
  "iss": "order-service",
  "aud": "inventory-service",
  "sub": "service-account",
  "scope": ["check_inventory", "reserve_items"]
}
```

## Key Takeaways

1. **JWTs are signed, not encrypted** - Data is visible but tamper-evident
2. **Stateless authentication** - No server-side session storage required
3. **Perfect for distributed systems** - Multiple services can validate independently
4. **Security through signatures** - Cryptographic proof of authenticity
5. **Claims-based** - Flexible way to include user information and permissions
6. **Cross-domain capable** - Single token works across multiple services
7. **Scalable solution** - Reduces database load and improves performance

JWTs provide a modern, scalable approach to authentication that's particularly well-suited for today's distributed, microservices-based applications. When implemented correctly with proper security practices, they offer an excellent balance of security, performance, and flexibility.
