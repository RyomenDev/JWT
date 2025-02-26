## **Example: API Rate Limiting with JWT**

```javascript
const rateLimitMiddleware = (req, res, next) => {
  const token = req.header("Authorization").split(" ")[1];
  const decoded = jwt.verify(token, "secret");

  if (decoded.requests >= 100) {
    return res.status(429).json({ message: "Rate limit exceeded" });
  }

  decoded.requests += 1;
  next();
};
```
