# Caching JWT validation in Redis

```js
// Example: Caching JWT validation in Redis
const redisClient = require("redis").createClient();
const jwt = require("jsonwebtoken");

const verifyToken = async (token) => {
  const cachedToken = await redisClient.get(token);
  if (cachedToken) return JSON.parse(cachedToken);

  const decoded = jwt.verify(token, process.env.PUBLIC_KEY);
  await redisClient.set(token, JSON.stringify(decoded), "EX", 900); // Cache for 15 min
  return decoded;
};
```
