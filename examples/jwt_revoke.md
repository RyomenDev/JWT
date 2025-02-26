## Implementation (Blacklist with Redis)

```js
const redisClient = require("redis").createClient();

// Revoke a token by storing its `jti`
app.post("/logout", (req, res) => {
  const { token } = req.body;
  const decoded = jwt.decode(token);
  redisClient.set(
    decoded.jti,
    "revoked",
    "EX",
    decoded.exp - Math.floor(Date.now() / 1000)
  );
  res.json({ message: "Logged out" });
});

// Middleware to check if token is revoked
const checkRevokedToken = async (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.sendStatus(403);

  const decoded = jwt.decode(token);
  redisClient.get(decoded.jti, (err, data) => {
    if (data) return res.status(401).json({ message: "Token revoked" });
    next();
  });
};
```
