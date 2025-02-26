## Implementing JWT Authentication in Node.js/Express

```js
const express = require("express");
const jwt = require("jsonwebtoken");

const app = express();
const SECRET_KEY = "your_secret_key";

app.post("/login", (req, res) => {
  const user = { id: 1, username: "john" };
  const token = jwt.sign(user, SECRET_KEY, { expiresIn: "1h" });
  res.json({ token });
});

const authenticateJWT = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.sendStatus(403);

  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};

app.get("/protected", authenticateJWT, (req, res) => {
  res.json({ message: "Protected data" });
});

app.listen(3000, () => console.log("Server running on port 3000"));
```