## WebSocket JWT Authentication (Node.js + Socket.io)

```javascript
import { Server } from "socket.io";
import jwt from "jsonwebtoken";

const io = new Server(server, { cors: { origin: "*" } });

io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  if (!token) return next(new Error("Authentication error"));
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.user = decoded;
    next();
  } catch (err) {
    next(new Error("Invalid token"));
  }
});

io.on("connection", (socket) => {
  console.log(`🔌 Client connected: ${socket.user.id}`);
});
```
