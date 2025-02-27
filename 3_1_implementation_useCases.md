# **JWT Authentication: Implementation & Use Cases**

## **1. How can JWTs be used in GraphQL authentication?**

### **Implementation**

JWTs are used in GraphQL authentication by including them in HTTP headers for requests.

- Clients send JWTs in the **Authorization header** of GraphQL queries.
- Middleware validates the token before resolving queries/mutations.

- **Example in Apollo Server (Node.js):**

```javascript
const { verify } = require("jsonwebtoken");
const context = ({ req }) => {
  const token = req.headers.authorization || "";
  return { user: verify(token, "secretKey") };
};
```

#### **GraphQL JWT Flow Diagram**

```
Client  ---> GraphQL Server (JWT Middleware) ---> Database
      <-- Valid JWT (if authenticated) --
```

## **2. How does a JWT middleware work in Express.js or Django?**

#### Express.js Middleware Example:

```javascript
function authenticateJWT(req, res, next) {
  const token = req.header("Authorization");
  if (!token) return res.status(401).send("Access Denied");
  jwt.verify(token, "secretKey", (err, user) => {
    if (err) return res.status(403).send("Invalid Token");
    req.user = user;
    next();
  });
}
```

#### **Django Middleware Example:**

```python
import jwt
from django.http import JsonResponse

def jwt_middleware(get_response):
    def middleware(request):
        token = request.headers.get('Authorization')
        try:
            decoded = jwt.decode(token, 'secretKey', algorithms=['HS256'])
            request.user = decoded
        except:
            return JsonResponse({'error': 'Invalid Token'}, status=403)
        return get_response(request)
    return middleware

```

### **Flow Diagram**

```
Request --> Middleware (Verify JWT) --> Proceed to route
                        \---> Unauthorized (401)
```

## **3. How would you handle JWT authentication in a mobile app?**

- Use JWTs for **access tokens** and store them securely (Keychain for iOS, Encrypted Shared Preferences for Android).
- **Include JWT in API requests**
- Use **refresh tokens** to obtain new access tokens when they expire.
- Secure communication with HTTPS and avoid token storage in localStorage.

## **4. Can JWTs be used for API rate limiting?**

- Yes, the **user ID or IP address in the JWT payload** can be used to enforce rate limits (by embedding usage count in the JWT payload).
- Example:
  - Check how many requests a user has made within a time window.
  - If the limit is exceeded, return a **429 Too Many Requests** response

## **5. How does JWT authentication differ from session cookies in SPAs?**

- **JWTs:** Stateless, stored client-side, scalable, used in RESTful APIs.
- **Session Cookies:** Stateful, stored server-side, less scalable but more secure.
- SPAs often use **JWTs for API authentication** due to their stateless nature.

| Feature         | JWT                        | Session Cookies |
| --------------- | -------------------------- | --------------- |
| Storage         | LocalStorage / HTTP Header | Cookie          |
| Stateless       | Yes                        | No              |
| CSRF Protection | Not Needed                 | Needed          |
| Scaling         | Easy                       | Hard            |

## **6. What is JWT audience (aud) and issuer (iss) claim, and how are they used?**

- **iss (Issuer):** Identifies who issued the token(ensures the token is from a trusted source).
- **aud (Audience):** Specifies who the token is intended for( ensures the token is for a specific API).

### **Example: JWT with aud & iss**

```javascript
const token = jwt.sign({ userId: 1 }, "secret", {
  audience: "api.example.com",
  issuer: "auth.example.com",
});
```

```
{
  "iss": "auth.example.com",
  "aud": "myapp.example.com",
  "userId": "123"
}
```

## **7. How would you implement JWT-based authentication in a microservices environment?**

- Use a central authentication service to issue JWTs
- Use **public-private key (RS256)** signing for added security.
- Issue **JWTs from an authentication service.**
- Each microservice **verifies the JWT** before processing requests.

### **Flow Diagram**

```
Client -> Auth Service (JWT Issued) -> Microservices (Validate JWT)
```

## **8. Can you use JWTs for single sign-on (SSO)?**

- Yes, JWTs are widely used for SSO (Single Sign-On),by using OAuth2 + JWT.
- Steps:
  - The user authenticates with Identity Provider (IdP) (e.g., Google, Auth0).
  - The IdP issues a JWT containing user identity.
  - Other services verify the JWT and grant access.
- **JWT claims for SSO:**

```json
{
  "sub": "user123",
  "iss": "auth.example.com",
  "aud": ["app1.example.com", "app2.example.com"],
  "exp": 1712345678
}
```

## **9. How would you secure JWTs in a serverless (AWS Lambda) architecture?**

- Use AWS Cognito or Lambda Authorizer to verify JWTs
- Store secrets in AWS Secrets Manager

- Verify JWTs **inside each Lambda function** before executing business logic.
- Use **AWS Cognito or Firebase Authentication** for token generation and validation.
- Store **signing keys in AWS Secrets Manager** or Firebase config.
- Restrict token usage with **scopes and audience claims.**

## 10- How do OAuth2 and JWTs work together in third-party authentication systems?

- OAuth2 issues **JWTs as access tokens** to authorize API requests.
- OAuth2 **refresh tokens** allow obtaining new JWTs without re-authenticating.
- Example flow:
  - User logs in via OAuth2 provider (Google, GitHub).
  - The provider issues a JWT access token.
  - The client includes the JWT in API requests (Authorization: Bearer <token>).

## 11- How would you implement device-based authentication using JWTs?

- Store a **device ID** in the JWT payload.
- Use **refresh token**s tied to specific devices.
- Implement a **device approval flow** for new devices.
- Example JWT payload:

```json
{
  "sub": "user123",
  "deviceId": "device-xyz",
  "exp": 1712345678
}
```

## 12- What is the difference between OAuth2 implicit flow vs. authorization code flow with JWTs?

- **Implicit Flow** (Deprecated for security reasons):
  - Access token is returned **directly in the URL.**
  - Vulnerable to **token leakage via browser history.**
- **Authorization Code Flow** (Recommended):
  - Server exchanges a short-lived **authorization code** for an access token.
  - More secure because the token is **not exposed to the frontend.**

## 13- How would you implement multi-tenant authentication using JWTs?

- Include the **tenant ID in the JWT payload.**
- Store tenant-specific keys in a **key store (AWS KMS, JWKS endpoint).**
- Verify that requests are **restricted to the tenant's domain or service.**
- Example JWT payload:

```json
{
  "sub": "user123",
  "tenant": "companyABC",
  "role": "admin"
}
```

---

This document covers a broad range of JWT use cases with implementation examples. Let me know if you need additional details! 🚀
