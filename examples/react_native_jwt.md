## **Example: React Native JWT Authentication**

```javascript
import AsyncStorage from "@react-native-async-storage/async-storage";

const login = async (email, password) => {
  const res = await fetch("https://api.example.com/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password }),
  });
  const data = await res.json();
  if (data.token) {
    await AsyncStorage.setItem("token", data.token);
  }
};
```
