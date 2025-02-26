## **Google OAuth with JWT**

```javascript
passport.use(
  new GoogleStrategy(
    {
      clientID: GOOGLE_CLIENT_ID,
      clientSecret: GOOGLE_CLIENT_SECRET,
      callbackURL: "https://example.com/auth/google/callback",
    },
    (accessToken, refreshToken, profile, done) => {
      const token = jwt.sign({ user: profile }, "secret");
      done(null, token);
    }
  )
);
```