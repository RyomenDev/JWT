## Multi-Tenant JWT Issuance

```js
const tenants = {
  tenantA: { secret: "secretA" },
  tenantB: { secret: "secretB" },
};

app.post("/login", (req, res) => {
  const { username, tenant } = req.body;
  const token = jwt.sign({ user: username, tenant }, tenants[tenant].secret, {
    expiresIn: "1h",
  });
  res.json({ token });
});
```
