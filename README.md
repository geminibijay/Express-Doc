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

Alright — here’s your **Day 2 dedicated notes** in the same style as Day 1.

---

## **📒 Day 2 — Working with Routes & HTTP Methods in Express**

### **1️⃣ Concepts Learned**

* **CRUD Operations** with Express routes:

  * `GET` → Read data.
  * `POST` → Create data.
  * `PUT` → Replace an existing record.
  * `PATCH` → Update part of an existing record.
  * `DELETE` → Remove a record.
* **Route Parameters** (`/users/:id`)

  * Accessed via `req.params`.
  * Need to be converted to a number before comparing (`Number()` or `parseInt()`).
* **Query Parameters** (`/search?name=...`)

  * Accessed via `req.query`.
* **Basic Input Validation**

  * Check for missing fields.
  * Return appropriate HTTP status codes:

    * `400` → Bad request (validation failed).
    * `404` → Not found.
    * `201` → Created successfully.
* **404 Fallback Route**

  * Always placed at the end of routes to handle unmatched paths.

---

### **2️⃣ Final Code (Clean Version)**

```js
const express = require("express");
const app = express();
app.use(express.json());

// Fake database
let users = [
  { id: 1, name: "Alice", age: 25 },
  { id: 2, name: "Bob", age: 30 }
];

// GET all users
app.get("/users", (req, res) => {
  res.json(users);
});

// GET single user
app.get("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: "User not found" });
  res.json(user);
});

// POST create new user
app.post("/users", (req, res) => {
  const { name, age } = req.body;
  if (!name || !age) return res.status(400).json({ error: "Name and age are required" });

  const newUser = { id: users.length + 1, name, age };
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT replace entire user
app.put("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  const { name, age } = req.body;
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: "User not found" });
  if (!name || !age) return res.status(400).json({ error: "Name and age are required" });

  user.name = name;
  user.age = age;
  res.json(user);
});

// PATCH update part of user
app.patch("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  const { name, age } = req.body;
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: "User not found" });

  if (name) user.name = name;
  if (age) user.age = age;
  res.json(user);
});

// DELETE user
app.delete("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  const existingLength = users.length;
  users = users.filter(u => u.id !== id);
  if (users.length === existingLength) {
    return res.status(404).json({ error: "User not found" });
  }
  res.json({ message: "User deleted" });
});

// Search users
app.get("/search", (req, res) => {
  const { name } = req.query;
  if (!name) return res.status(400).json({ error: "Name query parameter is required" });

  const result = users.filter(u => u.name.toLowerCase().includes(name.toLowerCase()));
  res.json(result);
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

### **3️⃣ HTTP Status Codes Used**

* **200 OK** → Request was successful.
* **201 Created** → New resource created successfully.
* **400 Bad Request** → Missing or invalid input.
* **404 Not Found** → No matching resource.

---

### **4️⃣ Testing Checklist (Postman / REST Client)**

* [x] GET `/users` → Returns list.
* [x] GET `/users/:id` → Returns user if exists, else `404`.
* [x] POST `/users` → Creates user if valid.
* [x] PUT `/users/:id` → Replaces all fields.
* [x] PATCH `/users/:id` → Updates only sent fields.
* [x] DELETE `/users/:id` → Removes user.
* [x] GET `/search?name=...` → Filters users.
* [x] Invalid route → Returns 404 JSON.

---

* More HTTP methods (`PUT`, `PATCH`, `DELETE`)
* Route parameters (`/users/:id`)
* Query parameters (`/users?age=20`)
* Organizing routes in separate files

---
