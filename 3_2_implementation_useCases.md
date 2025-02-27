# Implementation And UseCases

## 1. How do you manage JWTs in a multi-device login scenario?

- Each device gets a unique **refresh token** upon login.
- Access tokens are short-lived, while refresh tokens help obtain new access tokens.
- Store refresh tokens securely using **HTTP-only cookies**.
- Implement an **active session list** to track device logins.
- Provide an option for users to **log out from specific devices**.

### **Flow Diagram:**

1. User logs in on multiple devices.
2. Each device gets a separate refresh token.
3. When the access token expires, the refresh token is used to get a new one.
4. If a device logs out, revoke its refresh token only.

```js
// Example: Refresh Token Handling in Express.js
app.post("/refresh", async (req, res) => {
  const { refreshToken } = req.cookies;
  if (!refreshToken) return res.status(401).json({ error: "Unauthorized" });

  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_SECRET);
    const newAccessToken = jwt.sign(
      { userId: decoded.userId },
      process.env.ACCESS_SECRET,
      { expiresIn: "15m" }
    );
    res.json({ accessToken: newAccessToken });
  } catch (err) {
    res.status(403).json({ error: "Invalid Refresh Token" });
  }
});
```

## 2. Handling JWT Authentication in Microservices Architecture

- **Decentralized approach:** Each microservice validates JWTs independently using a shared public key.
- **Centralized approach:** Use an API Gateway (e.g., Kong, Nginx) for token validation before forwarding requests.
- Use **JWKS endpoint** for key management.
- Consider using a **JWT verification service** for consistency.

### **Diagram:**

```
[Client] ---> [API Gateway] ---> [Microservice 1]
                        |----> [Microservice 2]
                        |----> [Microservice 3]
```

```js
// Example: JWT Middleware for Microservices (Express.js)
const authenticateToken = (req, res, next) => {
  const token = req.headers["authorization"]?.split(" ")[1];
  if (!token) return res.sendStatus(401);

  jwt.verify(token, process.env.ACCESS_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};
```

## 3. Storing JWTs: Cookies vs. LocalStorage

| Feature                  | HTTP-only Cookies | LocalStorage |
| ------------------------ | ----------------- | ------------ |
| Secure Against XSS       | ✅ Yes            | ❌ No        |
| Secure Against CSRF      | ❌ No             | ✅ Yes       |
| Auto-sent with requests  | ✅ Yes            | ❌ No        |
| Requires manual handling | ❌ No             | ✅ Yes       |

**Best Practice:** Store **access tokens in memory** and **refresh tokens in HTTP-only cookies**.

## 4. JWT Authentication with API Gateways (Kong, Nginx, AWS API Gateway)

- Offload JWT validation to **API Gateway plugins (e.g., Kong JWT Plugin)**.
- Use **AWS Cognito or Auth0** for managed authentication.
- Configure **JWT verification at the edge** using Cloudflare or AWS Lambda@Edge.
- **Kong API Gateway**: Use the `jwt` plugin for token verification.
- **Nginx**: Configure JWT validation in the `nginx.conf`.
- **AWS API Gateway**: Use Cognito or a Lambda function to validate tokens.

```yaml
# Example: Kong API Gateway JWT Plugin
plugins:
  - name: jwt
    config:
      key_claim_name: iss
      secret_is_base64: true
```

### **Flow:**

1. The client sends a JWT in the `Authorization` header.
2. The API Gateway verifies the token.
3. If valid, the request is forwarded to the microservice.
4. If invalid, an error is returned.

---

These best practices ensure secure JWT management in different authentication scenarios.
