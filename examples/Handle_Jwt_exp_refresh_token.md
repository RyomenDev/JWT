# Handling JWT Expiration and Refresh Tokens Securely

## Implementation (Node.js + Express)

```javascript
const jwt = require("jsonwebtoken");

const ACCESS_SECRET = "your_access_secret";
const REFRESH_SECRET = "your_refresh_secret";

// Generate access token (valid for 15 min)
const generateAccessToken = (user) => {
  return jwt.sign({ id: user.id, role: user.role }, ACCESS_SECRET, { expiresIn: "15m" });
};

// Generate refresh token (valid for 7 days)
const generateRefreshToken = (user) => {
  return jwt.sign({ id: user.id }, REFRESH_SECRET, { expiresIn: "7d" });
};
```

## Refresh Flow

```javascript
app.post("/refresh", (req, res) => {
  const refreshToken = req.body.token;
  if (!refreshToken) return res.sendStatus(401);

  jwt.verify(refreshToken, REFRESH_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);

    const newAccessToken = generateAccessToken(user);
    res.json({ accessToken: newAccessToken });
  });
});
```

**Diagram: Refresh Token Flow**
```plaintext
Client       ->   Server (JWT expired)
Client       <-   401 Unauthorized
Client       ->   Sends Refresh Token
Server       ->   Validates Refresh Token
Server       ->   Issues New Access Token
Client       <-   Receives New Access Token
```