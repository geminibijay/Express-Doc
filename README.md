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

Alright — here’s your **Day 3 dedicated notes** with both the **full code** and a **deep explanation** for each section.
I’ve organized it so you can easily review later and recall **what** we wrote and **why**.

---

# **📓 Day 3 Notes — Middleware & JWT Authentication in Express**

## **Goals**

* Understand **middleware** in Express (global & route-level).
* Implement **logging**, **input validation**, and **auth checks** as middleware.
* Add **JWT login** + protect routes using authentication.
* Use **role-based authorization** (e.g., admin-only routes).
* Handle **404 errors** and **invalid JSON** gracefully.

---

## **1. Full Code**

```js
// ====== Imports & Setup ======
require("dotenv").config();
const express = require("express");
const morgan = require("morgan");
const jwt = require("jsonwebtoken");

const app = express();

// Built-in middleware: parses JSON
app.use(express.json());

// Third-party middleware: logs requests
app.use(morgan("dev"));

// Config
const JWT_SECRET = process.env.JWT_SECRET || "dev-secret-change-me";
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN || "1h";

// ====== Fake Databases ======
let users = [
  { id: 1, name: "Alice", age: 25 },
  { id: 2, name: "Bob", age: 30 }
];

const accounts = [
  { id: 1, username: "alice", password: "alice123", role: "user" },
  { id: 2, username: "admin", password: "admin123", role: "admin" }
];

// ====== Middleware Helpers ======

// Validate required fields
const requireFields = (...fields) => (req, res, next) => {
  for (const f of fields) {
    if (req.body[f] === undefined) {
      return res.status(400).json({ error: `${f} is required` });
    }
  }
  next();
};

// Authenticate using JWT
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization || "";
  const token = authHeader.startsWith("Bearer ") ? authHeader.slice(7) : null;

  if (!token) {
    return res.status(401).json({ error: "Missing or malformed Authorization header" });
  }

  try {
    const payload = jwt.verify(token, JWT_SECRET);
    req.user = payload;
    next();
  } catch {
    return res.status(401).json({ error: "Invalid or expired token" });
  }
}

// Restrict to specific role
const requireRole = (role) => (req, res, next) => {
  if (!req.user || req.user.role !== role) {
    return res.status(403).json({ error: "Forbidden: insufficient permissions" });
  }
  next();
};

// ====== Auth Routes ======
app.post("/auth/login", requireFields("username", "password"), (req, res) => {
  const { username, password } = req.body;
  const account = accounts.find(a => a.username === username && a.password === password);
  if (!account) return res.status(401).json({ error: "Invalid credentials" });

  const token = jwt.sign(
    { sub: account.id, username: account.username, role: account.role },
    JWT_SECRET,
    { expiresIn: JWT_EXPIRES_IN }
  );

  res.json({ token });
});

app.get("/me", authenticate, (req, res) => {
  const account = accounts.find(a => a.id === req.user.sub);
  if (!account) return res.status(404).json({ error: "Account not found" });

  res.json({ id: account.id, username: account.username, role: account.role });
});

// ====== Users CRUD ======
app.get("/users", (req, res) => res.json(users));

app.get("/users/:id", (req, res) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ error: "User not found" });
  res.json(user);
});

app.post("/users", authenticate, requireFields("name", "age"), (req, res) => {
  const { name, age } = req.body;
  const newUser = { id: users.length ? Math.max(...users.map(u => u.id)) + 1 : 1, name, age };
  users.push(newUser);
  res.status(201).json(newUser);
});

app.put("/users/:id", authenticate, requireFields("name", "age"), (req, res) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ error: "User not found" });

  user.name = req.body.name;
  user.age = req.body.age;
  res.json(user);
});

app.patch("/users/:id", authenticate, (req, res) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ error: "User not found" });

  if (req.body.name !== undefined) user.name = req.body.name;
  if (req.body.age !== undefined) user.age = req.body.age;
  res.json(user);
});

app.delete("/users/:id", authenticate, requireRole("admin"), (req, res) => {
  const exists = users.some(u => u.id === Number(req.params.id));
  if (!exists) return res.status(404).json({ error: "User not found" });

  users = users.filter(u => u.id !== Number(req.params.id));
  res.json({ message: "User deleted successfully" });
});

// ====== Search ======
app.get("/search", (req, res) => {
  const { name } = req.query;
  if (!name) return res.status(400).json({ error: "Name query parameter is required" });

  const result = users.filter(u => u.name.toLowerCase().includes(String(name).toLowerCase()));
  res.json(result);
});

// ====== 404 Handler ======
app.use((req, res) => {
  res.status(404).json({ error: "Route not found" });
});

// ====== Error Handler ======
app.use((err, req, res, next) => {
  if (err?.type === "entity.parse.failed") {
    return res.status(400).json({ error: "Invalid JSON in request body" });
  }
  console.error("Unhandled error:", err);
  res.status(err?.status || 500).json({ error: err?.message || "Internal Server Error" });
});

// ====== Start Server ======
app.listen(3002, () => {
  console.log("Server running on http://localhost:3002");
});
```

---

## **2. Concept Breakdown**

### **Middleware**

* Functions that sit **between** the request and the final response.
* Can modify `req`, `res`, or stop the request entirely.
* **Global** middleware: runs for all requests (`app.use(...)`).
* **Route-level** middleware: runs only for specific routes.

---

### **Custom Middleware Functions**

1. **`requireFields`**

   * Checks that certain keys exist in `req.body`.
   * Prevents incomplete data from reaching business logic.
2. **`authenticate`**

   * Reads `Authorization` header.
   * Verifies JWT.
   * Attaches decoded payload (`req.user`) for later use.
3. **`requireRole`**

   * Checks if `req.user.role` matches the required role.
   * Returns `403 Forbidden` if not.

---

### **JWT Authentication Flow**

1. **Login** (`/auth/login`)

   * Verify username/password.
   * If correct, sign a JWT with:

     * `sub`: User ID
     * `username`: Name
     * `role`: Permissions level
   * Send token to client.
2. **Protected routes**

   * Require `Authorization: Bearer <token>`.
   * Token is verified, and payload is attached to `req.user`.
3. **Role-based**

   * Admin-only routes check `req.user.role === "admin"`.

---

### **Error Handling**

* **404 handler**: if no route matches.
* **Invalid JSON**: `express.json()` throws `entity.parse.failed`, caught by error handler.
* **Central error handler**: last middleware in chain, formats all errors in JSON.

---

### **Order of Middleware**

1. `express.json()` – parse JSON first.
2. `morgan("dev")` – log every request.
3. **Routes** – some with extra middlewares.
4. **404 handler** – if no route matched.
5. **Error handler** – final cleanup for any error.

---

## **3. Testing**

You can use:

* **Postman** (GUI)
* **REST Client** or **Postman VS Code Extension** (inline testing)

Example test flow:

1. GET `/users` → public
2. POST `/users` without token → `401`
3. POST `/auth/login` with `alice` → get token
4. POST `/users` with token → `201 Created`
5. DELETE `/users/:id` with normal token → `403`
6. Login as admin → delete works

---

## **4. Key Takeaways**

* Middleware allows clean separation of **concerns** (logging, validation, auth).
* JWTs provide **stateless authentication** — no server session storage.
* Order matters — early middleware preps the request, late middleware handles errors.
* Always sanitize and validate input **before** touching your data.

---

Here’s your **Day 3 Notes** — clean, organized, and ready for reference later.

---

# **Day 3 – Middleware & JWT Authentication in Express**

## **1. Middleware Basics**

* **Definition:** Functions that sit between the request and the response.

* **Types:**

  1. **Application-level**: Runs for all requests.
     Example:

     ```js
     app.use(express.json());
     app.use(morgan('dev'));
     ```
  2. **Route-level**: Runs only for specific routes.
     Example:

     ```js
     app.post('/users', authenticate, handler);
     ```
  3. **Built-in**: Provided by Express (e.g., `express.json()`).
  4. **Third-party**: Installed packages (e.g., `morgan` for logging).
  5. **Custom**: Your own functions (e.g., `requireFields`).

* **Order matters**: Middleware runs in the order you `app.use()` or attach them.

---

## **2. Request Lifecycle with Middleware**

1. Request comes in.
2. Passes through global middleware (e.g., JSON parser, logger).
3. Hits route handler (with optional route-level middleware).
4. If no route matches → 404 handler.
5. If an error is thrown/passed → central error handler.

---

## **3. JWT Authentication**

* **JWT** = JSON Web Token — a secure, signed string encoding user data.

* **Flow:**

  1. **Login** (`/auth/login`)

     * Verify username/password.
     * Generate token with `jwt.sign(payload, secret, options)`.
  2. **Send token to client** (e.g., `{"token": "..."}`).
  3. **Client stores token** (e.g., in memory, secure cookie).
  4. **Protected route**:

     * Client sends header:
       `Authorization: Bearer <token>`.
     * Server verifies with `jwt.verify(token, secret)`.
     * If valid → `req.user` contains decoded payload.
     * If invalid/expired → 401 Unauthorized.

* **Token claims**:

  * `sub`: Subject (usually user ID).
  * `role`: Role of user (`user`, `admin`).
  * `iat`: Issued At timestamp.
  * `exp`: Expiry timestamp.

---

## **4. Common Custom Middleware in Day 3**

1. **`requireFields(...fields)`**

   * Checks if certain fields are in `req.body`.
   * If missing → `400 Bad Request`.
2. **`authenticate`**

   * Verifies JWT token from `Authorization` header.
   * If valid → attaches payload to `req.user`.
   * If invalid → `401 Unauthorized`.
3. **`requireRole(role)`**

   * Checks `req.user.role`.
   * If not matching → `403 Forbidden`.

---

## **5. Error Handling**

* **404 Handler**: Runs if no route matches.
* **Central Error Handler**:

  ```js
  app.use((err, req, res, next) => {
    if (err.type === 'entity.parse.failed') {
      return res.status(400).json({ error: 'Invalid JSON' });
    }
    res.status(500).json({ error: 'Internal Server Error' });
  });
  ```
* Handles:

  * Invalid JSON body.
  * Uncaught exceptions in routes.
  * Sends **consistent JSON error format**.

---

## **6. Day 3 API Features**

* **Public**:

  * `GET /users`
  * `GET /users/:id`
  * `GET /search?name=...`
* **Protected (need token)**:

  * `POST /users` (create)
  * `PUT /users/:id` (replace)
  * `PATCH /users/:id` (update)
  * `GET /me` (current user info)
* **Admin-only**:

  * `DELETE /users/:id`

---

## **7. Tools Used**

* `express` → Web framework.
* `dotenv` → Load `.env` config.
* `morgan` → HTTP request logging.
* `jsonwebtoken` → JWT creation & verification.

---

## **8. Testing**

* Use **Postman** or **VS Code REST Client**.
* Test both:

  * Missing/invalid tokens (should fail).
  * Valid tokens (should pass).
* Try sending **invalid JSON** to see error handler work.

---

✅ **Day 3 Skill Gains:**

* Write and use middleware correctly.
* Implement JWT login flow.
* Protect routes based on authentication & role.
* Handle errors cleanly.
* Test APIs for both happy-path and failure cases.

---


                ┌────────────────────────────┐
                │        Incoming Request     │
                └──────────────┬─────────────┘
                               │
               ┌───────────────▼──────────────┐
               │  1. Global Middleware Layer  │
               ├──────────────────────────────┤
               │ express.json() → parse body  │
               │ morgan("dev") → log request  │
               └───────────────┬──────────────┘
                               │
                   ┌───────────▼───────────────┐
                   │        ROUTES             │
                   ├───────────────────────────┤
                   │ Public Routes:            │
                   │   GET /users               │
                   │   GET /users/:id           │
                   │   GET /search              │
                   │                            │
                   │ Auth Routes:               │
                   │   POST /auth/login         │
                   │   GET /me                  │
                   │                            │
                   │ Protected Routes:          │
                   │   POST /users              │
                   │   PUT /users/:id           │
                   │   PATCH /users/:id         │
                   │   DELETE /users/:id (admin)│
                   └───────────┬───────────────┘
                               │
                ┌──────────────▼───────────────┐
                │   Middleware per Route       │
                ├─────────────────────────────┤
                │ requireFields(...)           │
                │ authenticate (JWT verify)    │
                │ requireRole("admin")         │
                └──────────────┬───────────────┘
                               │
                ┌──────────────▼───────────────┐
                │ Controller Logic             │
                │ - Access DB / modify users   │
                │ - Send JSON response         │
                └──────────────┬───────────────┘
                               │
           ┌───────────────────▼────────────────────┐
           │            If No Route Match            │
           │ → 404 Handler: { error: "Route not found"} │
           └───────────────────┬────────────────────┘
                               │
           ┌───────────────────▼────────────────────┐
           │        Central Error Handler            │
           │ Handles: invalid JSON, uncaught errors  │
           │ Returns: { error: "message" }           │
           └───────────────────┬────────────────────┘
                               │
                ┌──────────────▼───────────────┐
                │       Response Sent          │
                └─────────────────────────────┘


---
