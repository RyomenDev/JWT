

**🔹 Code Example (Node.js - Refresh Token Flow)**

```js
const express = require("express");
const jwt = require("jsonwebtoken");

const SECRET_KEY = "supersecretkey";
const REFRESH_SECRET = "refreshsecret";

let refreshTokens = [];

const app = express();
app.use(express.json());

const generateAccessToken = (user) => {
  return jwt.sign(user, SECRET_KEY, { expiresIn: "15m" });
};

const generateRefreshToken = (user) => {
  const refreshToken = jwt.sign(user, REFRESH_SECRET);
  refreshTokens.push(refreshToken);
  return refreshToken;
};

// Login Route
app.post("/login", (req, res) => {
  const user = { id: 1, role: "user" };
  const accessToken = generateAccessToken(user);
  const refreshToken = generateRefreshToken(user);
  res.json({ accessToken, refreshToken });
});

// Refresh Token Route
app.post("/token", (req, res) => {
  const { token } = req.body;
  if (!token || !refreshTokens.includes(token)) return res.sendStatus(403);

  jwt.verify(token, REFRESH_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    const newAccessToken = generateAccessToken({ id: user.id, role: user.role });
    res.json({ accessToken: newAccessToken });
  });
});

app.listen(3000, () => console.log("Server running on port 3000"));
```