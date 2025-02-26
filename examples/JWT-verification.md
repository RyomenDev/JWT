## 🔹 Code Example (JWT Verification in Node.js)

```javascipt
const jwt = require("jsonwebtoken");

const SECRET_KEY = "supersecretkey";

const verifyToken = (token) => {
  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    console.log("Token is valid:", decoded);
  } catch (err) {
    console.log("Invalid Token:", err.message);
  }
};

const token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
verifyToken(token);
```

## 🔹 JWT Verification Flow Diagram**
```
+------------+        +-------------+        +----------------+
|  Client    | -----> |  Server     | -----> |  Verify Token  |
+------------+        +-------------+        +----------------+
                                          |
                                          v
                               +----------------------+
                               |  Valid?  |  Expired? |
                               +----------------------+
```
