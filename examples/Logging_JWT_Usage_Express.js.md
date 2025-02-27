## Logging JWT Usage in Express.js

```js
const winston = require("winston");
const logger = winston.createLogger({
  transports: [new winston.transports.File({ filename: "jwt-audit.log" })],
});

app.use((req, res, next) => {
  if (req.headers.authorization) {
    const decoded = jwt.decode(req.headers.authorization.split(" ")[1]);
    logger.info({
      event: "JWT Used",
      user: decoded.user,
      timestamp: Date.now(),
    });
  }
  next();
});
```