# Node.js Intermediate Cheatsheet

## Express.js Framework

### Setting Up Express
```bash
npm install express
```

```javascript
const express = require("express");
const app = express();
const PORT = process.env.PORT || 3000;

// Parse JSON request bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Routing
```javascript
const express = require("express");
const app = express();
app.use(express.json());

// GET request
app.get("/users", (req, res) => {
    res.json({ users: [] });
});

// POST request
app.post("/users", (req, res) => {
    const { name, email } = req.body;
    res.status(201).json({ id: 1, name, email });
});

// PUT request
app.put("/users/:id", (req, res) => {
    const { id } = req.params;
    res.json({ message: `User ${id} updated` });
});

// DELETE request
app.delete("/users/:id", (req, res) => {
    const { id } = req.params;
    res.json({ message: `User ${id} deleted` });
});
```

### Route Parameters and Query Strings
```javascript
// Route parameters (:id)
app.get("/users/:id", (req, res) => {
    const { id } = req.params;  // /users/42 → id = "42"
    res.json({ id });
});

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req, res) => {
    const { userId, postId } = req.params;
    res.json({ userId, postId });
});

// Query strings (?key=value)
app.get("/search", (req, res) => {
    const { q, page = 1, limit = 10 } = req.query;
    // /search?q=nodejs&page=2&limit=5
    res.json({ query: q, page, limit });
});
```

### Express Router
```javascript
// routes/users.js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
    res.json({ users: [] });
});

router.get("/:id", (req, res) => {
    res.json({ id: req.params.id });
});

router.post("/", (req, res) => {
    res.status(201).json({ message: "User created" });
});

module.exports = router;

// app.js
const usersRouter = require("./routes/users");
app.use("/users", usersRouter);
// Routes: GET /users, GET /users/:id, POST /users
```

## Middleware

### Understanding Middleware
```javascript
// Middleware signature: (req, res, next)
function logger(req, res, next) {
    console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
    next();  // Pass to next middleware/route handler
}

// Apply globally
app.use(logger);

// Apply to specific routes
app.get("/protected", logger, (req, res) => {
    res.json({ data: "protected resource" });
});
```

### Error Handling Middleware
```javascript
// Error middleware has 4 parameters
function errorHandler(err, req, res, next) {
    console.error(err.stack);
    
    const statusCode = err.statusCode || 500;
    res.status(statusCode).json({
        error: err.message || "Internal Server Error"
    });
}

// Must be registered after all routes
app.use(errorHandler);

// Triggering the error handler
app.get("/risky", (req, res, next) => {
    try {
        throw new Error("Something went wrong");
    } catch (err) {
        next(err);  // Passes error to error handler
    }
});
```

### Common Middleware Packages
```bash
npm install cors helmet morgan
```

```javascript
const cors = require("cors");
const helmet = require("helmet");
const morgan = require("morgan");

// Security headers
app.use(helmet());

// CORS configuration
app.use(cors({
    origin: "http://localhost:3000",
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"]
}));

// HTTP request logging
app.use(morgan("dev"));
```

## Asynchronous Patterns

### Promises
```javascript
const fs = require("fs").promises;

function readUserFile(userId) {
    return fs.readFile(`users/${userId}.json`, "utf8")
        .then(data => JSON.parse(data))
        .catch(err => {
            throw new Error(`User ${userId} not found`);
        });
}

readUserFile(42)
    .then(user => console.log(user))
    .catch(err => console.error(err));
```

### Async/Await
```javascript
const fs = require("fs").promises;

async function readUserFile(userId) {
    try {
        const data = await fs.readFile(`users/${userId}.json`, "utf8");
        return JSON.parse(data);
    } catch (err) {
        throw new Error(`User ${userId} not found`);
    }
}

// Running multiple async operations in parallel
async function loadDashboard(userId) {
    const [user, posts, comments] = await Promise.all([
        fetchUser(userId),
        fetchPosts(userId),
        fetchComments(userId)
    ]);

    return { user, posts, comments };
}

// Async route handler in Express
app.get("/users/:id", async (req, res, next) => {
    try {
        const user = await readUserFile(req.params.id);
        res.json(user);
    } catch (err) {
        next(err);
    }
});
```

### Promise Utilities
```javascript
// Run all in parallel, fail if any fail
const results = await Promise.all([p1, p2, p3]);

// Run all in parallel, get results even if some fail
const results = await Promise.allSettled([p1, p2, p3]);
results.forEach(result => {
    if (result.status === "fulfilled") {
        console.log(result.value);
    } else {
        console.error(result.reason);
    }
});

// Return first resolved/rejected
const fastest = await Promise.race([p1, p2, p3]);

// Return first fulfilled (ignores rejections)
const first = await Promise.any([p1, p2, p3]);
```

## Working with Databases

### MongoDB with Mongoose
```bash
npm install mongoose
```

```javascript
const mongoose = require("mongoose");

// Connect
async function connectDB() {
    await mongoose.connect(process.env.MONGO_URI);
    console.log("MongoDB connected");
}
connectDB();

// Define schema
const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    age: { type: Number, min: 0 },
    createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model("User", userSchema);

// CRUD Operations
// Create
const user = await User.create({ name: "Alice", email: "alice@example.com" });

// Read
const users = await User.find();
const user = await User.findById(id);
const alice = await User.findOne({ name: "Alice" });

// Update
const updated = await User.findByIdAndUpdate(id, { age: 26 }, { new: true });

// Delete
await User.findByIdAndDelete(id);
```

### SQLite/PostgreSQL with pg (PostgreSQL)
```bash
npm install pg
```

```javascript
const { Pool } = require("pg");

const pool = new Pool({
    connectionString: process.env.DATABASE_URL
});

// Query
async function getUsers() {
    const result = await pool.query("SELECT * FROM users");
    return result.rows;
}

// Parameterized query (prevents SQL injection)
async function getUserById(id) {
    const result = await pool.query(
        "SELECT * FROM users WHERE id = $1",
        [id]
    );
    return result.rows[0];
}

// Transaction
async function transferFunds(fromId, toId, amount) {
    const client = await pool.connect();
    try {
        await client.query("BEGIN");
        await client.query(
            "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
            [amount, fromId]
        );
        await client.query(
            "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
            [amount, toId]
        );
        await client.query("COMMIT");
    } catch (err) {
        await client.query("ROLLBACK");
        throw err;
    } finally {
        client.release();
    }
}
```

## Authentication

### JWT Authentication
```bash
npm install jsonwebtoken bcryptjs
```

```javascript
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");

const SECRET_KEY = process.env.JWT_SECRET;

// Hash password
async function hashPassword(password) {
    const salt = await bcrypt.genSalt(10);
    return bcrypt.hash(password, salt);
}

// Verify password
async function verifyPassword(password, hash) {
    return bcrypt.compare(password, hash);
}

// Generate token
function generateToken(userId) {
    return jwt.sign({ userId }, SECRET_KEY, { expiresIn: "24h" });
}

// Verify token middleware
function authenticate(req, res, next) {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(" ")[1];  // Bearer <token>

    if (!token) {
        return res.status(401).json({ error: "Access token required" });
    }

    try {
        const decoded = jwt.verify(token, SECRET_KEY);
        req.userId = decoded.userId;
        next();
    } catch (err) {
        res.status(403).json({ error: "Invalid or expired token" });
    }
}

// Login route
app.post("/login", async (req, res, next) => {
    try {
        const { email, password } = req.body;
        const user = await User.findOne({ email });

        if (!user || !(await verifyPassword(password, user.password))) {
            return res.status(401).json({ error: "Invalid credentials" });
        }

        const token = generateToken(user._id);
        res.json({ token });
    } catch (err) {
        next(err);
    }
});

// Protected route
app.get("/profile", authenticate, async (req, res, next) => {
    try {
        const user = await User.findById(req.userId);
        res.json(user);
    } catch (err) {
        next(err);
    }
});
```

## Making HTTP Requests

### Using fetch (Node.js 18+)
```javascript
// Built-in fetch (Node.js 18+)
async function fetchUser(id) {
    const response = await fetch(`https://api.example.com/users/${id}`);
    
    if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
    }
    
    return response.json();
}
```

### Using axios
```bash
npm install axios
```

```javascript
const axios = require("axios");

// GET request
async function getUsers() {
    const response = await axios.get("https://api.example.com/users");
    return response.data;
}

// POST request
async function createUser(userData) {
    const response = await axios.post("https://api.example.com/users", userData, {
        headers: { "Content-Type": "application/json" }
    });
    return response.data;
}

// Axios instance with base configuration
const apiClient = axios.create({
    baseURL: "https://api.example.com",
    timeout: 5000,
    headers: { "Authorization": `Bearer ${process.env.API_TOKEN}` }
});

// Use the client
const users = await apiClient.get("/users");
```

## Validation

### Input Validation with express-validator
```bash
npm install express-validator
```

```javascript
const { body, validationResult } = require("express-validator");

// Validation rules
const userValidationRules = [
    body("name").trim().notEmpty().withMessage("Name is required"),
    body("email").isEmail().normalizeEmail().withMessage("Invalid email"),
    body("age").optional().isInt({ min: 0, max: 120 }).withMessage("Invalid age"),
    body("password").isLength({ min: 8 }).withMessage("Password must be at least 8 characters")
];

// Validation middleware
function validate(req, res, next) {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
    }
    next();
}

// Apply to route
app.post("/users", userValidationRules, validate, async (req, res, next) => {
    try {
        const user = await User.create(req.body);
        res.status(201).json(user);
    } catch (err) {
        next(err);
    }
});
```

## Best Practices (Do's)

✅ **Organize routes into separate files**
```javascript
// routes/users.js
const router = require("express").Router();
router.get("/", getAllUsers);
router.post("/", createUser);
module.exports = router;

// app.js
app.use("/users", require("./routes/users"));
app.use("/posts", require("./routes/posts"));
```

✅ **Use async wrapper for Express route handlers**
```javascript
// Utility to catch async errors
const asyncHandler = fn => (req, res, next) =>
    Promise.resolve(fn(req, res, next)).catch(next);

app.get("/users", asyncHandler(async (req, res) => {
    const users = await User.find();
    res.json(users);
}));
```

✅ **Validate and sanitize all inputs**
```javascript
// Always validate before using request data
app.post("/users", [
    body("email").isEmail().normalizeEmail(),
    body("name").trim().escape()
], validate, createUser);
```

## Common Mistakes (Don'ts)

❌ **Don't store secrets in code**
```javascript
// Bad
const secret = "super-secret-key";

// Good
const secret = process.env.JWT_SECRET;
```

❌ **Don't forget to call next() in middleware**
```javascript
// Bad - request hangs
app.use((req, res, next) => {
    console.log(req.method);
    // next() not called!
});

// Good
app.use((req, res, next) => {
    console.log(req.method);
    next();
});
```

❌ **Don't send multiple responses**
```javascript
// Bad - causes "Cannot set headers after they are sent" error
app.get("/user", (req, res) => {
    if (!req.user) {
        res.status(401).json({ error: "Unauthorized" });
    }
    res.json(req.user);  // Runs even if user is not authenticated!
});

// Good
app.get("/user", (req, res) => {
    if (!req.user) {
        return res.status(401).json({ error: "Unauthorized" });
    }
    res.json(req.user);
});
```
