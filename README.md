# Express-Doc
Learn Node (Express.js)

Alright — here’s your **Dedicated Day 1 Notes** for Week 1 Express learning.
You can keep this as a reference while practicing.

---

# **📒 Week 1 – Day 1 Notes (Express Basics)**

## **1. What is Express.js?**

* A **framework** for building web servers and APIs in Node.js.
* Makes it easy to define **routes**, use **middleware**, and handle **HTTP requests**.

---

## **2. REST API Basics**

**REST** = Representational State Transfer
A set of rules for building APIs.

* **Resources** = The things your API manages (e.g., users, products).
* Each resource has an **endpoint** (URL).
* Uses **HTTP methods** to define actions:

| Method     | Meaning                             | Example    |
| ---------- | ----------------------------------- | ---------- |
| **GET**    | Read data                           | `/users`   |
| **POST**   | Create new data                     | `/users`   |
| **PUT**    | Replace an existing resource        | `/users/5` |
| **PATCH**  | Update part of an existing resource | `/users/5` |
| **DELETE** | Remove resource                     | `/users/5` |

---

## **3. Setting Up an Express Server**

```js
const express = require("express");
const app = express();

// Middleware to parse JSON
app.use(express.json());

// GET route
app.get("/", (req, res) => {
  res.json({ message: "Hello World from Express!" });
});

// POST route
app.post("/users", (req, res) => {
  const { name } = req.body;
  if (!name) return res.status(400).json({ error: "Name is required" });
  res.json({ message: `User ${name} created successfully!` });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: "Route not found" });
});

// Start server
app.listen(3002, () => {
  console.log("Server running on http://localhost:3002");
});
```

---

## **4. Understanding the Code**

* **`const express = require("express");`** → Import Express.
* **`const app = express();`** → Create Express app object.
* **`app.use(express.json());`** → Middleware to parse JSON body into `req.body`.
* **`app.get()`** → Handle GET requests.
* **`app.post()`** → Handle POST requests.
* **`app.use((req, res) => {...})`** → Catch-all for undefined routes (404).
* **`app.listen()`** → Start server on a given port.

---

## **5. Middleware**

* Functions that run **before** your route handler.
* Can:

  * Parse data (e.g., JSON)
  * Log requests
  * Authenticate users
  * Handle errors

Example logging middleware:

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // continue to next handler
});
```

---

## **6. Testing API in VS Code with REST Client**

1. Install **REST Client** extension.
2. Create `requests.http`:

```http
### GET request
GET http://localhost:3002/
Accept: application/json

###

### POST request
POST http://localhost:3002/users
Content-Type: application/json

{
  "name": "Alice"
}
```

3. Click **Send Request** inside VS Code.

---

## **7. Day 1 Practice Checklist**

✅ Install Node.js & Express
✅ Create GET and POST routes
✅ Learn REST & HTTP methods
✅ Add JSON parsing middleware
✅ Test with Postman or REST Client
✅ Add a 404 handler

---

If you save this note, you’ll have everything you need to review **before** Day 2, where we’ll cover:

* More HTTP methods (`PUT`, `PATCH`, `DELETE`)
* Route parameters (`/users/:id`)
* Query parameters (`/users?age=20`)
* Organizing routes in separate files

---
