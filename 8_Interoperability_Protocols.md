# Interoperability & Protocols in JWT Authentication

## 1. How do JWTs differ from SAML tokens in authentication workflows?

- **JWTs:** Lightweight, JSON format, commonly used in REST APIs.
- **SAML:** XML-based, used for enterprise Single Sign-On (SSO).

### **Key Differences:**

| Feature                | JWT (JSON Web Token)             | SAML (Security Assertion Markup Language) |
| ---------------------- | -------------------------------- | ----------------------------------------- |
| Format                 | JSON                             | XML                                       |
| Transport              | HTTP Headers (Bearer Token)      | SAML Assertions via POST/SOAP             |
| Use Case               | Modern APIs, SPAs, Microservices | Enterprise SSO, Legacy Apps               |
| Signature & Encryption | JWS (Signing), JWE (Encryption)  | XML Signature & Encryption                |

### **Authentication Flow:**

1. User logs in and receives a JWT.
2. The JWT is sent in API requests (`Authorization: Bearer <token>`).
3. The server verifies the token and grants access.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622
}
```

---

## 2. How does OpenID Connect (OIDC) use JWTs for authentication?

### **How OIDC Uses JWTs for Authentication:**

- OIDC issues **ID tokens (JWTs)** after successful login, which contain user profile info.
- OIDC is built on top of OAuth2 and uses JWTs as **ID Tokens**.
- The **ID Token** contains user identity claims (e.g., email, name, roles).

### **OIDC Flow Diagram:**

```
[User] ---> [Client App] ---> [Authorization Server]
                        |---> Returns JWT (ID Token + Access Token)
```

### **OIDC ID Token Example:**

```json
{
  "iss": "https://auth.example.com",
  "sub": "user123",
  "aud": "client_id",
  "iat": 1616239022,
  "exp": 1616242622,
  "email": "user@example.com"
}
```

---

## 3. What are self-contained vs. reference tokens, and how do JWTs fit in?

### **Self-Contained Tokens (JWTs):**

- All necessary information is embedded within the token (stateless).
- **Pros**: No database lookup required.
- **Cons**: Cannot be revoked easily.

### **Reference Tokens:**

- Contains only a unique identifier (requires database lookup).
- **Pros**: Can be revoked or updated.
- **Cons**: Extra DB calls.

```json
{
  "token": "random-uuid-123456"
}
```

---

## 4. How does JWT authentication work with gRPC microservices?

- JWTs are passed in **metadata headers** `(authorization: Bearer <token>)`.

### **Flow Diagram:**

```
[Client] ---> [gRPC API Gateway] ---> [Microservice 1]
                              |---> [Microservice 2]
```

### **Example: JWT Validation in gRPC (Node.js)**

```js
const jwt = require("jsonwebtoken");
const grpc = require("@grpc/grpc-js");

function authenticate(call, callback) {
  const token = call.metadata.get("authorization")[0].split(" ")[1];
  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) return callback({ code: grpc.status.UNAUTHENTICATED });
    callback(null, { userId: decoded.sub });
  });
}
```

---

## 5. How do OAuth2 scopes work with JWTs?

### **OAuth2 Scope Usage:**

- JWTs include `scope` claims to define user permissions.
- Example: `scope: "read:users write:posts"`.

### **Example JWT with Scopes:**

```json
{
  "sub": "user123",
  "scope": "read:users write:posts",
  "exp": 1616242622
}
```

### **Validating Scopes in Express.js:**

```js
function checkScope(requiredScope) {
  return (req, res, next) => {
    const token = req.headers.authorization.split(" ")[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    if (!decoded.scope.includes(requiredScope)) return res.sendStatus(403);
    next();
  };
}
```

---

These strategies ensure secure and scalable JWT authentication across different authentication protocols and systems.
