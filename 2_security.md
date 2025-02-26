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

- The `kid` parameter in the header helps in identifying the correct signing key when verifying JWTs.
- The kid (Key ID) helps identify which key was used to sign the JWT.
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

- **Stateless:** jWTs are stateless by design, The server does not store session data. The client sends JWT with each request,  meaning no session is stored on the server..
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

---

These best practices help mitigate common JWT security risks and ensure safe authentication in web applications.
