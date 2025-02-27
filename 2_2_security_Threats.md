# **Comprehensive Guide to JWT Security Best Practices**

## 1. How do you handle JWT expiration in a distributed system?

### **Problem:**

In a distributed system, multiple services may need to verify JWTs without a centralized session store.

### **Solution:**

Use short-lived access tokens and refresh tokens:

- **Access Token**: Short expiration time (e.g., 15 minutes)
- **Refresh Token**: Longer expiration (e.g., 7 days), stored securely in an HTTP-only cookie or database.
- Implement a centralized authentication service to issue and validate tokens.
- Use a token **blacklist stored** in Redis or a database for revoked tokens.

**Diagram:**

```plaintext
(Client) → [Request with Access Token] → (API Gateway) → [Verify JWT] → (Microservices)
          ↘ [Request New Token with Refresh Token] ↙
```

**Code Implementation in Express.js:**

```js
const jwt = require("jsonwebtoken");

function generateAccessToken(user) {
  return jwt.sign({ userId: user.id }, process.env.ACCESS_SECRET, {
    expiresIn: "15m",
  });
}

function generateRefreshToken(user) {
  return jwt.sign({ userId: user.id }, process.env.REFRESH_SECRET, {
    expiresIn: "7d",
  });
}
```

---

## 2. Can you use JWTs without signing? Why or why not?

### **Problem:**

JWTs can technically be unsigned (`alg: "none"`), but this is a critical security risk.

### **Solution:**

- **Always sign JWTs** using a secure algorithm (HS256, RS256).
- **Reject tokens with `alg: "none"`.**
- **Use asymmetric encryption (RS256)** for better security.

**Example Secure JWT Signing (RS256)**

```js
const privateKey = fs.readFileSync("private.pem");
const token = jwt.sign({ userId: "123" }, privateKey, { algorithm: "RS256" });
```

---

## 3. How do you handle session hijacking when using JWTs?(Preventing JWT Replay Attacks)

**Session hijacking is not a replay attack**. During session hijacking, the attacker takes over a valid user session to **gain access to data and services**. In contrast, a replay attack involves **capturing and resending data to the target network**.

### **Problem:**

Attackers can capture and reuse a JWT to perform unauthorized actions.

### **Mitigation Strategies:**

1. **Use a jti (JWT ID) claim** and store used tokens in a database.
2. **Short expiration times** to minimize token reuse.
3. **Pair JWT with device/session fingerprinting** (bind JWT to IP, device, or user agent).
4. **Log out** users by invalidating refresh tokens.
5. Monitor **unusual login activity** and alert users.
6. Use **OAuth2 PKCE (Proof Key for Code Exchange)** for added security.

**Example: Checking `jti` in a Database**

```js
const usedTokens = new Set();
function verifyToken(token) {
  const decoded = jwt.verify(token, secretKey);
  if (usedTokens.has(decoded.jti)) {
    throw new Error("Replay attack detected!");
  }
  usedTokens.add(decoded.jti);
}
```

---

## 4. How do you secure JWTs when using them in mobile applications?

- Store JWTs in **Secure Enclave (iOS) or Keystore (Android)**.
- Use **OAuth2 PKCE (Proof Key for Code Exchange)** instead of client-side tokens.
- Implement **biometric authentication (Face ID, fingerprint) before sending tokens**.
- Rotate signing keys periodically using **JWKS (JSON Web Key Set)**.

## 5. Can an Attacker Forge a JWT if They Know the Payload?

### **Answer:**

No, an attacker cannot forge a JWT unless they have the signing key.

### **How to Protect Against Forgery?**

- Use **strong secret keys** (HS256) or **asymmetric cryptography** (RS256), or a private key (RS256).
- **Validate token signatures** before trusting claims.
- However, if the JWT uses **"none" algorithm** in the header, an attacker can **forge a token**.

---

## 6. How do you handle JWT token leakage in a public API?

- Store **access tokens in memory** instead of localStorage.
- Use **HTTP-only, Secure, SameSite=strict cookies** for refresh tokens.
- Implement **Token Revocation List (TRL)** in Redis to invalidate leaked tokens.
- Rotate JWT signing keys regularly using **JWKS (JSON Web Key Set)**.

## 7. How do you secure JWTs when working with GraphQL APIs?

### Problem:

GraphQL APIs accept complex queries, making them susceptible to JWT misuse.

### Solution:

1. **Pass JWT in headers** (Authorization: Bearer token) instead of query parameters.
2. **Restrict introspection queries** to prevent schema exposure.
3. **Use middleware to validate JWT before query execution.**
4. Avoid exposing sensitive claims in the JWT payload.

**Example in Apollo Server (GraphQL)**

```js
const { ApolloServer } = require("apollo-server");
const jwt = require("jsonwebtoken");

const context = ({ req }) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) throw new Error("Unauthorized");
  return { user: jwt.verify(token, process.env.JWT_SECRET) };
};

const server = new ApolloServer({ typeDefs, resolvers, context });
```

---

## 8. What are nested JWTs (NJWTs), and when would you use them?

### **Definition:**

- A Nested JWT is a JWT **inside another JWT** (inside the payload), commonly used for additional encryption layers.
- Used when one entity needs to verify the inner JWT while another validates the outer JWT.

### **Use Case:**

- **Secure transmission of sensitive claims.**
- **Multi-layer authentication mechanisms.**
- **Multi-layer authentication**, such as OAuth2 delegation.

**Example:**

```js
const innerJWT = jwt.sign({ data: "Sensitive" }, innerSecret);
const outerJWT = jwt.sign({ jwt: innerJWT }, outerSecret);
```

---

## 9. What are the weakest claims in a JWT that an attacker might exploit?

- **sub:** Attackers may modify the subject (user ID) for privilege escalation.
- **exp:** If missing, tokens never expire, leading to security risks.
- **aud:** If improperly validated, a JWT could be used in unintended services.

## 10. Can JWTs be signed with multiple keys? How would you validate them?

- Yes, using **multiple issuers (multi-tenant systems)** or key rotation with **JWKS (JSON Web Key Set)**.
- The application must fetch the correct public key from the JWKS endpoint before verification.

## 11. What is a JWT injection attack, and how do you mitigate it?

- **JWT injection attack** occurs when an attacker modifies claims like sub or scope to escalate privileges.
- **Mitigation**:
  - Always validate JWT signature before decoding.
  - Never trust user-supplied tokens without server verification.
  - Use role-based access control (RBAC) and strict permission checks.

## 12. Preventing JWT Brute-Force Attacks

### **Problem:**

Attackers may try to guess JWT signatures using brute force.

### **Mitigation Strategies:**

- **Use long, randomly generated secret keys, complex secrets** for HS256.
- Prefer **asymmetric keys (RS256, ES256)** over HS256 to prevent brute-force guessing.
- Set **short expiration times** to reduce attack windows.
- **Implement rate limiting** on authentication endpoints to prevent excessive JWT attempts.
- Use **HSM (Hardware Security Module) or AWS KMS** for key storage.

**Example: Rate Limiting in Express.js**

```js
const rateLimit = require("express-rate-limit");
app.use("/login", rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }));
```

---

## 13. What happens if the signing key of a JWT is compromised? How do you respond?

- Immediately **revoke all tokens signed with the compromised key**.
- Rotate the signing keys and **update the JWKS endpoint**.
- Force users to **re-authenticate and issue new tokens**.
- Log the breach and analyze access logs for suspicious activity.

## 14. How do you prevent a JWT signature confusion attack?

### **Problem:**

- An attacker may switch between symmetric and asymmetric algorithms.
- Attackers can modify the JWT header to change the algorithm to `"none"` or another weak algorithm.

### **Solution:**

- Reject JWTs that do not use expected algorithms.
- Validate **`alg`** before decoding.
- Always **hardcode the expected algorithm** in the server validation logic.
- Reject JWTs where `alg: "none"`.
- Validate the **JWT header, payload, and signature separately** before processing.

**Example:**

```js
if (decodedToken.header.alg !== "RS256") {
  throw new Error("Unexpected algorithm used!");
}
```

---

## 15. How do you prevent JWT downgrade attacks?

- **JWT downgrade attacks** occur when an attacker forces the server to use a weaker signing algorithm (e.g., changing RS256 to HS256).
- **Mitigation:**
  - **Enforce strong algorithms** (e.g., RS256, ES256) in the validation logic.
  - Store allowed algorithms in a **server-side configuration** (not user-controlled).
  - Regularly audit JWT logs for **unexpected algorithm changes**.

---

## 16. The Risk of Using JWTs Without Expiration (`exp` Claim)

### **Problem:**

If a JWT never expires, it remains valid indefinitely, increasing security risks.

### **Solution:**

- **Always set an expiration (`exp`).**
- Use refresh tokens to extend sessions securely.
- Implement token revocation strategies (e.g., JWT blacklists).

**Example:**

```js
jwt.sign({ userId: "123" }, secretKey, { expiresIn: "15m" });
```

---

## **Conclusion**

By implementing these best practices, you can significantly enhance the security of JWTs in your applications.
