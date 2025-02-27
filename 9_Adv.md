# Advanced JWT Interview Questions for Deep Technical Discussions

# <u>Security & Threats</u>

### 1. What are JWT kid (Key ID) headers, and how do they improve security?

- The `kid` header in JWTs helps identify the key used for signing, enabling key rotation and management.
- Example:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "12345"
}
```

- This allows clients to fetch the correct public key for verification.
- The server retrieves the correct key from a **JWKS (JSON Web Key Set)** and verifies the signature.
- This allows **key rotation** without breaking existing JWTs.

### 2. How do you mitigate JWT brute-force attacks?

- Use strong, long secrets for HMAC algorithms.
- Set **short expiration times** to reduce attack windows.
- Implement rate limiting and monitoring.
- Use asymmetric keys (RS256, ES256) instead of HS256.
- Use **HSM (Hardware Security Module)** or **AWS KMS** for key storage.
- Example using rate limiting in Express:

```js
const rateLimit = require("express-rate-limit");

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: "Too many login attempts. Try again later.",
});
```

### 3. What happens if the signing key of a JWT is compromised? How do you respond?

- Immediate key rotation using JWKS.
- Invalidate affected JWTs via short expiration.
- **Revoke** refresh tokens if applicable. Immediately revoke all tokens signed with the compromised key.
- **Rotate the signing keys** and update the JWKS endpoint.
- Force users to **re-authenticate and issue new tokens**.
- Log the breach and analyze access logs for suspicious activity.

- Example key rotation flow:

```
[Old JWT] → [Key Rotation] → [New JWT with new kid] → [Clients fetch new key]
```

### 4. How do you prevent a JWT signature confusion attack?

- Ensure algorithms are explicitly defined and restricted.
- Example: Reject HS256 JWTs when using RS256:
- Always **hardcode the expected algorithm** in the server validation logic.
- Reject JWTs where alg: "none".
- **Validate the JWT header, payload, and signature** separately before processing.

```js
if (decodedToken.header.alg !== "RS256") {
  throw new Error("Invalid algorithm");
}
```

### 5. How do you prevent JWT downgrade attacks (e.g., forcing HS256 over RS256)?

- JWT downgrade attacks occur when an attacker forces the server to use a **weaker signing algorithm** (e.g., changing RS256 to HS256).
- **Mitigation:**
  - **Enforce strong algorithms** (e.g., RS256, ES256) in the validation logic.
  - Store allowed algorithms in a **server-side configuration** (not user-controlled).
  - Regularly audit JWT logs for **unexpected algorithm changes**.
  - **Validate JWT algorithm** on the server, Use **`alg` whitelisting** in verification.

```js
jwt.verify(token, publicKey, { algorithms: ["RS256"] });
```

### 6. What is the risk of using JWTs without expiration (`exp` claim)?

- Tokens remain valid indefinitely if not explicitly revoked.
- **Best practice:**
  - Use short-lived access tokens and refresh tokens.
  - Implement **token revocation strategies** (e.g., JWT blacklists).

# <u>JWT Best Practices & Management</u>

### 7. How do you handle JWTs in a serverless environment (AWS Lambda, Firebase)?

- Validate JWTs in API Gateway or edge functions before reaching Lambda.
- Use a **caching mechanism** for public keys to avoid excessive key fetch requests.
- **Store public keys in AWS Parameter Store or Secrets Manager** for validation.
- Use **JWT verification middleware** in AWS Lambda or Firebase Functions.
- Leverage **API Gateway authentication** to handle JWT verification before invoking Lambda.

### 8. What is the role of the `iss` (issuer) claim, and why is it important?

- Helps verify token authenticity and prevent token reuse across different applications.

- **Never log the full JWT** (especially the signature).
- Log only **header and claims** (excluding sensitive data).
- Use **structured logging tools** like ELK Stack, Datadog, or CloudWatch.

```json
{
  "iss": "https://auth.example.com"
}
```

### 9. How do you securely log JWTs without exposing sensitive data?

- Log only header and claims, excluding sensitive information.

```js
console.log(JSON.stringify(jwt.decode(token, { complete: true }).header));
```

### 10. How does JWT introspection work in OAuth2, and when would you use it?

- Introspection allows servers to check token validity against an authorization server.

```http
POST /introspect HTTP/1.1
Authorization: Basic client_id:client_secret
Content-Type: application/x-www-form-urlencoded

{ "token": "JWT_TOKEN_HERE" }
```

# <u>JWT in Distributed Systems & Microservices</u>

### 11. How do you handle JWT validation in a multi-tenant SaaS application?

- Use tenant-specific signing keys.

```json
{
  "sub": "user123",
  "tenant_id": "company_xyz",
  "role": "admin"
}
```

- Store `tenant_id` in claims and verify it before authorization.

### 12. How do you implement cross-service authentication using JWTs in a microservice architecture?

- Use API Gateway for centralized authentication.
- Each **microservice validates JWTs** before processing requests.
- Use **JWKS (JSON Web Key Set)** for dynamic key management.

```
[Client] → [API Gateway] → [Microservices: Auth, Orders, Payments]
```

### 13. How do you handle JWT revocation in a stateless system?

- Maintain a revocation list in a cache like Redis.
- Use **short-lived JWTs** with refresh tokens.
- Implement a **JWT blacklist** in Redis or a database.

### 14. How do you share JWTs securely across multiple domains?

- Use **CORS with proper settings**.
- Implement secure token exchange using OAuth2.

### 15. How do you prevent single JWTs from being used across multiple regions in a distributed system?

- Use region-specific signing keys and validate `iss` claim.
- Example:

```json
{
  "iss": "https://us-east.auth.example.com"
}
```

- Include a **region claim** inside the JWT:

```json
{
  "sub": "user123",
  "region": "us-west-1"
}
```

- Verify that the **token matches the current region** before processing.

# <u>Performance & Optimization</u>

### 16. How do you reduce JWT parsing time in high-performance APIs?

- Use **lightweight libraries** and avoid unnecessary claims.

### 17. How does using JWTs impact database queries and performance?

- JWTs eliminate the need for frequent database lookups for session validation.
- Use **compact JWTs** (remove unnecessary claims).
- Use **HS256 instead of RS256** when asymmetric keys are unnecessary.
- Cache **public keys** instead of fetching them from a remote JWKS endpoint.

### 18. How do you implement lazy validation of JWT claims to optimize performance?

- Validate only critical claims at request entry.

### 19. How does JWT compression (e.g., using Gzip or Brotli) impact security?

- WT compression can reduce payload size **but introduces security risks** like **CRIME and BREACH attacks**.
- Prefer using **short claim names** to reduce token size instead of compression.

# <u>Advanced JWT Usage & Interoperability</u>

### 20. What are self-issued OpenID Connect (OIDC) JWTs, and how do they work?

- Self-issued JWTs are signed by the entity itself.

### 21. Can JWTs be used for delegated authorization between third-party services?

- Yes, through **delegation tokens** with limited scopes.

### 22. How do you implement JWT-based Single Sign-On (SSO) across multiple applications?

- Use a centralized authentication server that issues JWTs recognized by all services.
- Use **OAuth2 + OpenID Connect (OIDC)** for federated authentication.
- Issue an **ID Token (JWT)** from a central Identity Provider (IdP).

### 23. How do you handle JWTs when dealing with IoT devices?

- Use **lightweight JWT formats** like **CWT (CBOR Web Tokens)**.
- ssue **short-lived JWTs** to minimize risk.
- Use **mutual TLS authentication** along with JWTs.

### 24. How do you verify JWTs in a Zero Trust security model?

- Validate every request with **continuous verification mechanisms**.
- Validate **JWT signature and claims on every request**.
- Implement **Continuous Access Evaluation (CAE)** to revoke active tokens dynamically.

---

These advanced JWT topics prepare candidates for deep technical discussions in interviews. Let me know if you need additional refinements! 🚀
