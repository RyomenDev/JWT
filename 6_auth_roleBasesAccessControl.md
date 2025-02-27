# Role-Based Access Control (RBAC) Using JWTs

## Overview
RBAC assigns roles to users, which define their access permissions. JWTs can store role information in the payload.

### JWT Payload Example for RBAC:
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "roles": ["admin", "editor"],
  "iat": 1618822997,
  "exp": 1618826597
}
```

### **Implementation in Express.js**
```js
const authorizeRole = (roles) => (req, res, next) => {
  if (!req.user || !roles.includes(req.user.role)) {
    return res.status(403).json({ message: "Access Denied" });
  }
  next();
};

app.get("/admin", authenticateToken, authorizeRole(["admin"]), (req, res) => {
  res.json({ message: "Welcome, Admin!" });
});
```

### **Diagram:**
```
[User] ---> [JWT with Role] ---> [Protected Route] ---> [Role-Based Access Granted]
```

---

# Attribute-Based Access Control (ABAC) Using JWTs

## Overview
ABAC grants access based on user attributes, resource context, and actions.

### JWT Payload Example for ABAC:
```json
{
  "sub": "1234567890",
  "name": "Jane Doe",
  "department": "finance",
  "clearance": "high"
}
```

### **Implementation Example**
```js
const authorizeABAC = (req, res, next) => {
  const { department, clearance } = req.user;
  if (department === "finance" && clearance === "high") {
    return next();
  }
  return res.status(403).json({ message: "Access Denied" });
};
```

### **Diagram:**
```
[User] ---> [JWT with Attributes] ---> [ABAC Policy Engine] ---> [Access Granted]
```

---

# Hierarchical Roles and Permissions

## Overview
Permissions are inherited from higher-level roles.

### **Example Role Hierarchy:**
```
Admin > Manager > Employee
```

### **Implementation:**
```js
const roleHierarchy = {
  admin: ["manager", "employee"],
  manager: ["employee"],
  employee: []
};

const authorizeHierarchy = (role) => (req, res, next) => {
  const userRole = req.user.role;
  if (!roleHierarchy[userRole]?.includes(role) && userRole !== role) {
    return res.status(403).json({ message: "Access Denied" });
  }
  next();
};
```

---

# Multi-Factor Authentication (MFA) Using JWTs

## Overview
MFA requires an additional verification step, such as an OTP.

### **Flow Diagram:**
```
[User Login] ---> [JWT Issued] ---> [MFA Required] ---> [Second Factor Verified] ---> [Access Granted]
```

### **Implementation:**
```js
const generateMfaToken = (userId) => jwt.sign({ userId, mfa: true }, process.env.MFA_SECRET, { expiresIn: "5m" });
```

---

# JWT with Multiple Audiences (aud)

## Overview
A JWT can have multiple audiences by setting the `aud` claim as an array.

### **Example JWT Payload:**
```json
{
  "sub": "1234567890",
  "aud": ["service1", "service2"],
  "exp": 1618826597
}
```

### **Implementation:**
```js
const verifyAudience = (token, expectedAudiences) => {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  if (!expectedAudiences.includes(decoded.aud)) {
    throw new Error("Invalid Audience");
  }
};
```

---
This guide provides secure JWT-based access control solutions for different use cases.

