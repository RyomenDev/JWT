# Implementing JWT Authentication (Node.js + Express)

### Step 1: Install Dependencies
```
npm install express jsonwebtoken dotenv
```

### Step 2: JWT Authentication Code
```js
const express = require("express");
const jwt = require("jsonwebtoken");
const dotenv = require("dotenv");

dotenv.config();
const app = express();
app.use(express.json());

const SECRET_KEY = process.env.JWT_SECRET || "supersecretkey";

// Dummy User Database
const users = [{ id: 1, username: "john", password: "1234" }];

// Generate JWT Token
const generateToken = (user) => {
  return jwt.sign({ userId: user.id, role: "user" }, SECRET_KEY, {
    expiresIn: "1h",
  });
};

// Login Route
app.post("/login", (req, res) => {
  const { username, password } = req.body;
  const user = users.find(
    (u) => u.username === username && u.password === password
  );
  if (!user) return res.status(401).json({ message: "Invalid credentials" });

  const token = generateToken(user);
  res.json({ token });
});

// Middleware for JWT Authentication
const authenticateJWT = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.sendStatus(403);

  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};

// Protected Route
app.get("/protected", authenticateJWT, (req, res) => {
  res.json({ message: "Welcome to the protected route!", user: req.user });
});

app.listen(3000, () => console.log("Server running on port 3000"));
```
