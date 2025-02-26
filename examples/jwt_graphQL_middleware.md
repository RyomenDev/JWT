# GraphQL Middleware in Express.js

```javascript
import express from "express";
import { ApolloServer, gql } from "apollo-server-express";
import jwt from "jsonwebtoken";

const JWT_SECRET = "your_secret_key";

const typeDefs = gql`
  type Query {
    currentUser: String
  }
`;

const resolvers = {
  Query: {
    currentUser: (_, __, { user }) => {
      if (!user) throw new Error("Not authenticated");
      return user.username;
    },
  },
};

const getUser = (token) => {
  try {
    return jwt.verify(token, JWT_SECRET);
  } catch (error) {
    return null;
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization?.split(" ")[1];
    const user = token ? getUser(token) : null;
    return { user };
  },
});

const app = express();
server.applyMiddleware({ app });
app.listen(4000, () => console.log("Server running"));
```