# Security & Best Practices

# Best Practices for Securing JWTs in Web Applications

## 1- What are the best practices for securing JWTs in web applications?

#### **Do's:**

- Store JWTs in **HTTP-only cookies** instead of localStorage to prevent XSS attacks.
- Use **Secure and SameSite flags** to prevent CSRF.
- Use **short-lived access tokens** with refresh tokens.
- Sign **JWTs with RS256** (asymmetric key pair) instead of HS256 when possible.
- Validate issuer (iss), audience (aud), and expiration (exp) claims.
- Implement **token revocation strategies** (blacklisting, database tracking).

### **Don'ts:**

- Do not store JWTs in **localStorage** or **sessionStorage** due to XSS risks.
- Do not expose JWTs in URLs.

## 2- How do you prevent Cross-Site Request Forgery (CSRF) when using JWTs?

- Store JWTs in \*HTTP-only, Secure cookies\*\* instead of localStorage.
- Implement **C**SRF tokens\*\* for state-changing requests.
- Use **SameSite=strict or SameSite=lax cookie attributes**.
  JWTs are **stateless**, so they don’t inherently prevent CSRF. Here’s how you can mitigate it:

### **Solutions:**

- **Use SameSite Cookies:**
  ```js
  res.cookie("token", jwtToken, {
    httpOnly: true,
    secure: true,
    sameSite: "Strict",
  });
  ```
- **Use CSRF tokens in Forms** (for additional security):
  ```html
  <input type="hidden" name="csrf_token" value="{{csrf_token}}" />
  ```

## 3- How do you prevent Cross-Site Scripting (XSS) attacks when handling JWTs?

### **Mitigation Strategies:**

- **Sanitize user input** before rendering it in HTML.
- **Use Content Security Policy (CSP)** headers To mitigate script injection:
  ```js
  app.use((req, res, next) => {
    res.setHeader("Content-Security-Policy", "default-src 'self'");
    next();
  });
  ```
- **Avoid storing JWTs in localStorage.**

## 4- What is the role of the kid (Key ID) in JWT headers?

- The `kid` parameter in the header helps in identifying the correct signing key when verifying JWTs (when using multiple signing keys).
- The server retrieves the correct key from a **JWKS (JSON Web Key Set)** and verifies the signature.
- This allows **key rotation** without breaking existing JWTs.
- Useful when rotating signing keys in production environments.
- The authentication server looks up the corresponding key using kid.

Example JWT Header:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "12345"
}
```

On the server, retrieve the correct public key using `kid` to verify the token.

## 5- How can JWTs be used with role-based access control (RBAC)?

- Store user roles (admin, user, moderator, etc.) in the JWT payload.
- Verify the role in middleware before processing API requests.
- Check role permissions before granting access to specific resources.

Assign user roles inside the JWT payload:

```json
{
  "sub": "user123",
  "role": "admin"
}
```

Verify the role in middleware:

```js
function authorize(roles) {
  return (req, res, next) => {
    const token = req.cookies.token;
    const decoded = jwt.verify(token, secret);
    if (!roles.includes(decoded.role)) {
      return res.status(403).send("Access Denied");
    }
    next();
  };
}
```

Usage in routes:

```js
app.get("/admin", authorize(["admin"]), (req, res) => {
  res.send("Welcome Admin");
});
```

## 6- Can JWTs be used for stateful authentication? Why or why not?

- **Stateless:** jWTs are stateless by design, The server does not store session data. The client sends JWT with each request, meaning no session is stored on the server..
- **Stateful:** Store JWTs in a database and check their validity before processing requests (**A token blacklist or database tracking system can make JWTs behave statefully**).

### **When to Use Stateful JWTs?**

- When needing immediate revocation of JWTs.
- When implementing fine-grained session control.

## 7- How does JWS (JSON Web Signature) differ from JWE (JSON Web Encryption)?

- JWS (Signed JWTs) ensure integrity but do not encrypt data.
- JWE (Encrypted JWTs) encrypt the payload, ensuring confidentiality.

| Feature           | JWS                     | JWE                              |
| ----------------- | ----------------------- | -------------------------------- |
| Purpose           | Data integrity (signed) | Data confidentiality (encrypted) |
| Readability       | Readable                | Encrypted                        |
| Example Algorithm | HS256, RS256            | RSA-OAEP, AES-GCM                |

Example JWS:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Example JWE:

```json
{
  "alg": "RSA-OAEP",
  "enc": "A256GCM"
}
```

## 8- Why should you never store sensitive user information inside a JWT payload?

- JWT payloads are **base64-encoded**, not encrypted, only signed.(By default)
- Anyone with access to the token can **decode and read** the payload.
- Instead, store sensitive information in **database sessions** and reference the user ID in the JWT.

### **Example of Insecure JWT Payload:**

```json
{
  "user": "john_doe",
  "password": "supersecret"
}
```

### **Secure Alternative:**

```json
{
  "sub": "user123",
  "sessionId": "a1b2c3d4"
}
```

## 9- How do you implement JWT revocation in a stateless system?

Since JWTs are stateless, revocation is challenging. Possible solutions:

- **Blacklist Approach:** Maintain a database of revoked tokens and check on every request.
- **Short-Lived Tokens:** Use short expiration times and refresh tokens.
- **Token Versioning:** Store a version number in the JWT payload and check it against the database.
- Use a **blocklist stored in Redis** for real-time revocation.

Example Blacklist:

```js
const revokedTokens = new Set();
app.post("/logout", (req, res) => {
  revokedTokens.add(req.cookies.token);
  res.send("Logged out");
});
```

## 10- What is token poisoning in JWTs, and how can you prevent it?

Token poisoning occurs when an attacker modifies or misuses JWT's payload.

- **Prevention:** Always validate the signature before processing JWTs.
- **Example Validation:**
  ```js
  try {
    const decoded = jwt.verify(token, secretKey);
  } catch (err) {
    return res.status(401).send("Invalid token");
  }
  ```
- Use **RS256 (asymmetric keys)** instead of HS256 (shared secret).
- Always verify iss, aud, and exp claims.
- Never store JWTs in **localStorage** (use HTTP-only, Secure cookies).
- Implement **rate limiting** to detect token replay attacks.

## 10- How does key rotation work with JWTs, and why is it important?

Key rotation ensures compromised keys don’t cause long-term damage.

- **Use `kid` field** to track multiple keys.
- **Revoke old tokens when rotating keys.** : key rotation means periodically changing the signing key to reduce exposure in case of compromise.
- Use **JWKS (JSON Web Key Set)** for automatic key rotation.
- Include a kid (Key ID) in JWT headers to identify the correct key.

Example JWT header:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "key123"
}
```

Example Key Rotation Handling:

```js
const keys = {
  oldKeyId: "oldSecret",
  newKeyId: "newSecret",
};
const tokenHeader = JSON.parse(
  Buffer.from(token.split(".")[0], "base64").toString()
);
const secret = keys[tokenHeader.kid];
jwt.verify(token, secret);
```

## 11- Can a JWT be used across multiple domains? How do you handle CORS issues?

- Yes, JWTs are self-contained, so they can be used across multiple domains.
- **CORS-related solutions:**
  - Use CORS headers (Access-Control-Allow-Origin) to specify allowed domains.
  - Store JWTs in Secure, HTTP-only cookies with the SameSite=None attribute.
  - Implement token forwarding via a central authentication service.

When using JWTs across multiple domains:

- **Enable CORS properly:**
  ```js
  app.use(
    cors({
      origin: "https://trusted-domain.com",
      credentials: true,
    })
  );
  ```
- **Set proper cookie attributes:**
  ```js
  res.cookie("token", jwtToken, { sameSite: "None", secure: true });
  ```

## 12- What are some alternatives to JWTs for authentication?

- **Session-based Authentication** (using Redis or DB)
- **OAuth 2.0** (Token-based delegation) , session cookies (stateful authentication).
- **SAML (Security Assertion Markup Language)** (XML-based authentication) for enterprise SSO.
- **Paseto (Platform-Agnostic Security Tokens)** as a JWT alternative.
- **Opaque tokens** (server-verified tokens, better for security-sensitive applications).

## 13- How do you prevent brute force attacks on JWTs?

- **Use Strong Signing Keys**
- **Rate Limit Authentication Endpoints**
  ```js
  const rateLimit = require("express-rate-limit");
  app.use("/login", rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }));
  ```
- **Monitor Unusual Login Attempts**
- Use **strong secret keys** (256-bit or higher for HMAC, RSA keys for RS256).
- Implement **rate limiting** on authentication endpoints.
- Enforce **account lockouts** after multiple failed login attempts.
- Use **refresh tokens** instead of long-lived access tokens.

---

These best practices help mitigate common JWT security risks and ensure safe authentication in web applications.
