# Performance & Optimization of JWTs

## 1. Reducing JWT Size While Maintaining Security

- Use **compact claims** (e.g., `sub` instead of `user_id`).
- Avoid unnecessary claims (store only essential information).
- Use **shortened keys** where possible.
- Prefer **HS256** over **RS256** if asymmetric encryption isn't required.
- Use **DEF encoding** (compressed JWTs) if supported.

### Example: Compact JWT Payload

```json
{
  "sub": "1234567890",
  "iat": 1712147600,
  "exp": 1712151200,
  "role": "admin"
}
```

## 2. Performance Trade-offs Between RS256, ES256, and HS256

- **HS256:** Fastest but needs a shared secret, for internal APIs (fast, single secret key)..
- **RS256:** Secure but slower (uses asymmetric encryption).
- **ES256:** Secure and faster than RS256 but **complex key management**.
- RS256/ES256 for public APIs (higher security, but slower verification).

| Algorithm | Type                        | Security  | Performance |
| --------- | --------------------------- | --------- | ----------- |
| HS256     | Symmetric                   | Moderate  | 🔥 Fast     |
| RS256     | Asymmetric                  | High      | ❄️ Slower   |
| ES256     | Asymmetric (Elliptic Curve) | Very High | ❄️ Slowest  |

## 3. Handling JWT Expiration & Session Persistence in Stateless APIs

- Use **short-lived access tokens** (e.g., 15 minutes).
- Store **refresh tokens in HTTP-only cookies**.
- Rotate refresh tokens upon use to mitigate replay attacks.
- Implement **grace periods** to prevent abrupt session terminations.

### Example: Refresh Token Flow in Express.js

```js
app.post("/refresh", async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (!refreshToken) return res.sendStatus(401);

  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_SECRET);
    const newAccessToken = jwt.sign(
      { userId: decoded.userId },
      process.env.ACCESS_SECRET,
      { expiresIn: "15m" }
    );
    res.json({ accessToken: newAccessToken });
  } catch (err) {
    res.sendStatus(403);
  }
});
```

## 4. Using JWTs with Server-side Caching (Redis, Memcached)

- Store **revoked JWTs** in a Redis blacklist.
- Cache **frequently accessed user claims** to reduce database queries.
- Use **JWT ID (`jti`)** claim to track and invalidate tokens.

### Example: Blacklisting JWTs in Redis

```js
const redisClient = require("redis").createClient();
app.post("/logout", async (req, res) => {
  const token = req.headers.authorization.split(" ")[1];
  const decoded = jwt.decode(token);
  await redisClient.setex(
    decoded.jti,
    decoded.exp - Math.floor(Date.now() / 1000),
    "revoked"
  );
  res.sendStatus(200);
});
```

## 5. JWTs in Serverless Architectures (AWS Lambda, Firebase Functions)

- Use **JWT verification middleware** for each request.
- Avoid storing state; use **refresh tokens** for long sessions.
- Cache verification keys using AWS Secrets Manager or Firebase Authentication.
- Prefer **asymmetric keys (RS256/ES256)** to avoid key sharing issues.

### Example: JWT Middleware in AWS Lambda (Node.js)

```js
const jwt = require("jsonwebtoken");
exports.handler = async (event) => {
  const token = event.headers.Authorization?.split(" ")[1];
  if (!token) return { statusCode: 401, body: "Unauthorized" };

  try {
    const decoded = jwt.verify(token, process.env.PUBLIC_KEY, {
      algorithms: ["RS256"],
    });
    return {
      statusCode: 200,
      body: JSON.stringify({ message: "Success", user: decoded }),
    };
  } catch (error) {
    return { statusCode: 403, body: "Invalid Token" };
  }
};
```

## Flow Diagram: JWT Expiration & Refresh Flow

```
[Client] ---> [Server]
   |             |
   | Access Expired
   |             |
   v             v
[Refresh Token] ---> [Verify & Issue New Access Token]
   |             |
   v             v
[New Access Token]
```

---

By following these best practices, JWT performance and security can be optimized efficiently in high-scale applications.
