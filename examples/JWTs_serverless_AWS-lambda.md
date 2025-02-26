## Secure JWTs in a serverless (AWS Lambda) architecture**

- Use AWS Cognito or Lambda Authorizer to verify JWTs
- Store secrets in AWS Secrets Manager

```javascript
exports.handler = async (event) => {
  const token = event.headers.Authorization.split(" ")[1];
  try {
    jwt.verify(token, process.env.JWT_SECRET);
    return { statusCode: 200, body: "Authorized" };
  } catch (err) {
    return { statusCode: 401, body: "Unauthorized" };
  }
};
```