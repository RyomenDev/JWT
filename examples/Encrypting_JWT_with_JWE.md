## Encrypting JWT with JWE

```js
const jose = require("node-jose");
const key = await jose.JWK.createKey("RSA", 2048, {
  alg: "RSA-OAEP",
  use: "enc",
});
const encryptedJWT = await jose.JWE.createEncrypt({ format: "compact" }, key)
  .update(Buffer.from(JSON.stringify(payload)))
  .final();
```