# Node.js Advanced Cheatsheet

## Streams

### Understanding Streams
```javascript
// Streams process data in chunks instead of loading it all into memory
// Types: Readable, Writable, Duplex, Transform

const fs = require("fs");

// Readable stream
const readStream = fs.createReadStream("large-file.txt", {
    encoding: "utf8",
    highWaterMark: 64 * 1024  // 64KB chunks
});

readStream.on("data", (chunk) => {
    console.log(`Received ${chunk.length} bytes`);
});

readStream.on("end", () => {
    console.log("Stream finished");
});

readStream.on("error", (err) => {
    console.error("Stream error:", err);
});
```

### Piping Streams
```javascript
const fs = require("fs");
const zlib = require("zlib");

// Pipe: readable → writable
fs.createReadStream("input.txt")
    .pipe(fs.createWriteStream("output.txt"));

// Chain transforms
fs.createReadStream("input.txt")
    .pipe(zlib.createGzip())
    .pipe(fs.createWriteStream("input.txt.gz"));
```

### Transform Streams
```javascript
const { Transform } = require("stream");

const upperCaseTransform = new Transform({
    transform(chunk, encoding, callback) {
        this.push(chunk.toString().toUpperCase());
        callback();
    }
});

process.stdin
    .pipe(upperCaseTransform)
    .pipe(process.stdout);
```

### Pipeline (Error-Safe Piping)
```javascript
const { pipeline } = require("stream/promises");
const fs = require("fs");
const zlib = require("zlib");

async function compressFile(input, output) {
    await pipeline(
        fs.createReadStream(input),
        zlib.createGzip(),
        fs.createWriteStream(output)
    );
    console.log(`Compressed ${input} → ${output}`);
}

compressFile("data.txt", "data.txt.gz");
```

## Clustering and Worker Threads

### Cluster Module
```javascript
const cluster = require("cluster");
const http = require("http");
const os = require("os");

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
    console.log(`Primary process ${process.pid} running`);

    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    cluster.on("exit", (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died. Restarting...`);
        cluster.fork();  // Restart the worker
    });
} else {
    // Worker processes share the same port
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end(`Handled by worker ${process.pid}`);
    }).listen(3000);

    console.log(`Worker ${process.pid} started`);
}
```

### Worker Threads
```javascript
const { Worker, isMainThread, parentPort, workerData } = require("worker_threads");

if (isMainThread) {
    // Main thread
    function runWorker(data) {
        return new Promise((resolve, reject) => {
            const worker = new Worker(__filename, { workerData: data });

            worker.on("message", resolve);
            worker.on("error", reject);
            worker.on("exit", (code) => {
                if (code !== 0) reject(new Error(`Worker stopped with exit code ${code}`));
            });
        });
    }

    // Run CPU-intensive task in a worker thread
    async function main() {
        const result = await runWorker({ n: 40 });
        console.log(`Fibonacci result: ${result}`);
    }
    main();

} else {
    // Worker thread - compute Fibonacci
    function fibonacci(n) {
        if (n <= 1) return n;
        return fibonacci(n - 1) + fibonacci(n - 2);
    }

    parentPort.postMessage(fibonacci(workerData.n));
}
```

## Caching

### In-Memory Caching
```javascript
class Cache {
    constructor(ttlSeconds = 60) {
        this.cache = new Map();
        this.ttl = ttlSeconds * 1000;
    }

    set(key, value) {
        const expires = Date.now() + this.ttl;
        this.cache.set(key, { value, expires });
    }

    get(key) {
        const item = this.cache.get(key);
        if (!item) return null;
        if (Date.now() > item.expires) {
            this.cache.delete(key);
            return null;
        }
        return item.value;
    }

    delete(key) {
        this.cache.delete(key);
    }
}

// Usage
const cache = new Cache(300);  // 5-minute TTL

app.get("/users/:id", async (req, res) => {
    const { id } = req.params;
    let user = cache.get(`user:${id}`);

    if (!user) {
        user = await User.findById(id);
        cache.set(`user:${id}`, user);
    }

    res.json(user);
});
```

### Redis Caching
```bash
npm install ioredis
```

```javascript
const Redis = require("ioredis");
const redis = new Redis(process.env.REDIS_URL);

// Cache with expiration
async function cacheUser(userId) {
    const cacheKey = `user:${userId}`;
    
    // Try cache first
    const cached = await redis.get(cacheKey);
    if (cached) {
        return JSON.parse(cached);
    }

    // Fetch from database
    const user = await User.findById(userId);

    // Store in cache for 1 hour
    await redis.setex(cacheKey, 3600, JSON.stringify(user));
    return user;
}

// Invalidate cache
async function updateUser(userId, data) {
    const user = await User.findByIdAndUpdate(userId, data, { new: true });
    await redis.del(`user:${userId}`);  // Invalidate cache
    return user;
}
```

## Performance Optimization

### Profiling with Node.js Built-ins
```javascript
// CPU profiling
const { Session } = require("inspector");
const fs = require("fs");

const session = new Session();
session.connect();

// Start profiling
session.post("Profiler.enable", () => {
    session.post("Profiler.start", () => {
        // Run your code here
        heavyOperation();

        // Stop and save profile
        session.post("Profiler.stop", (err, { profile }) => {
            fs.writeFileSync("profile.cpuprofile", JSON.stringify(profile));
            session.disconnect();
        });
    });
});
```

### Memory Management
```javascript
// Monitor memory usage
function logMemoryUsage() {
    const { rss, heapUsed, heapTotal, external } = process.memoryUsage();
    console.log({
        rss: `${Math.round(rss / 1024 / 1024)} MB`,
        heapUsed: `${Math.round(heapUsed / 1024 / 1024)} MB`,
        heapTotal: `${Math.round(heapTotal / 1024 / 1024)} MB`
    });
}

setInterval(logMemoryUsage, 5000);

// Use streaming for large data instead of loading into memory
async function processLargeFile(filePath) {
    const rl = require("readline").createInterface({
        input: require("fs").createReadStream(filePath),
        crlfDelay: Infinity
    });

    for await (const line of rl) {
        await processLine(line);  // Process one line at a time
    }
}
```

### Connection Pooling
```javascript
const { Pool } = require("pg");

// Use a connection pool instead of individual connections
const pool = new Pool({
    connectionString: process.env.DATABASE_URL,
    max: 20,           // Maximum connections in pool
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000
});

// Pool automatically manages connections
async function queryDatabase(sql, params) {
    const client = await pool.connect();
    try {
        const result = await client.query(sql, params);
        return result.rows;
    } finally {
        client.release();  // Always release back to pool
    }
}
```

## Testing

### Unit Testing with Jest
```bash
npm install --save-dev jest
```

```javascript
// math.js
function add(a, b) {
    return a + b;
}

function divide(a, b) {
    if (b === 0) throw new Error("Cannot divide by zero");
    return a / b;
}

module.exports = { add, divide };

// math.test.js
const { add, divide } = require("./math");

describe("Math functions", () => {
    test("adds two numbers", () => {
        expect(add(2, 3)).toBe(5);
        expect(add(-1, 1)).toBe(0);
    });

    test("divides two numbers", () => {
        expect(divide(10, 2)).toBe(5);
    });

    test("throws on division by zero", () => {
        expect(() => divide(10, 0)).toThrow("Cannot divide by zero");
    });
});
```

### Testing Express APIs with Supertest
```bash
npm install --save-dev supertest
```

```javascript
// app.js
const express = require("express");
const app = express();
app.use(express.json());

app.get("/users", (req, res) => {
    res.json([{ id: 1, name: "Alice" }]);
});

app.post("/users", (req, res) => {
    const { name } = req.body;
    if (!name) {
        return res.status(400).json({ error: "Name is required" });
    }
    res.status(201).json({ id: 2, name });
});

module.exports = app;

// app.test.js
const request = require("supertest");
const app = require("./app");

describe("Users API", () => {
    test("GET /users returns user list", async () => {
        const res = await request(app).get("/users");
        expect(res.status).toBe(200);
        expect(res.body).toHaveLength(1);
        expect(res.body[0].name).toBe("Alice");
    });

    test("POST /users creates a user", async () => {
        const res = await request(app)
            .post("/users")
            .send({ name: "Bob" });
        expect(res.status).toBe(201);
        expect(res.body.name).toBe("Bob");
    });

    test("POST /users returns 400 without name", async () => {
        const res = await request(app)
            .post("/users")
            .send({});
        expect(res.status).toBe(400);
    });
});
```

### Mocking with Jest
```javascript
// userService.js
const axios = require("axios");

async function fetchUser(id) {
    const { data } = await axios.get(`https://api.example.com/users/${id}`);
    return data;
}

module.exports = { fetchUser };

// userService.test.js
const axios = require("axios");
const { fetchUser } = require("./userService");

jest.mock("axios");

test("fetchUser returns user data", async () => {
    const mockUser = { id: 1, name: "Alice" };
    axios.get.mockResolvedValue({ data: mockUser });

    const user = await fetchUser(1);
    expect(user).toEqual(mockUser);
    expect(axios.get).toHaveBeenCalledWith("https://api.example.com/users/1");
});
```

## Security Best Practices

### Helmet and Rate Limiting
```bash
npm install helmet express-rate-limit
```

```javascript
const helmet = require("helmet");
const rateLimit = require("express-rate-limit");

// Security headers
app.use(helmet());

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100,                    // Max 100 requests per window
    message: { error: "Too many requests, please try again later" }
});

app.use("/api/", limiter);

// Stricter limit for auth routes
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    message: { error: "Too many login attempts" }
});

app.use("/api/auth/", authLimiter);
```

### Input Sanitization
```javascript
// Prevent NoSQL injection in MongoDB queries
app.get("/users", async (req, res) => {
    const name = String(req.query.name);  // Force string conversion

    // Bad - allows injection: ?name[$ne]=
    // const users = await User.find({ name: req.query.name });

    // Good - sanitized
    const users = await User.find({ name: name });
    res.json(users);
});

// Use parameterized queries for SQL databases
// Never interpolate user input into SQL strings
async function getUserByEmail(email) {
    // Good - parameterized
    const result = await pool.query(
        "SELECT * FROM users WHERE email = $1",
        [email]
    );

    // Bad - SQL injection risk!
    // const result = await pool.query(`SELECT * FROM users WHERE email = '${email}'`);
    
    return result.rows[0];
}
```

### Secure Cookie Configuration
```bash
npm install express-session
```

```javascript
const session = require("express-session");

app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === "production",  // HTTPS only in production
        httpOnly: true,   // Prevent JavaScript access
        sameSite: "strict",  // CSRF protection
        maxAge: 24 * 60 * 60 * 1000  // 24 hours
    }
}));
```

## Logging

### Structured Logging with Winston
```bash
npm install winston
```

```javascript
const winston = require("winston");

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || "info",
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: "logs/error.log", level: "error" }),
        new winston.transports.File({ filename: "logs/combined.log" })
    ]
});

if (process.env.NODE_ENV !== "production") {
    logger.add(new winston.transports.Console({
        format: winston.format.simple()
    }));
}

// Usage
logger.info("Server started", { port: 3000 });

try {
    // Simulated error
} catch (err) {
    logger.error("Database connection failed", { error: err.message });
}

// Express request logging middleware
app.use((req, res, next) => {
    logger.warn("Deprecated endpoint accessed", { path: req.path });
    const start = Date.now();
    res.on("finish", () => {
        logger.info("HTTP Request", {
            method: req.method,
            path: req.path,
            status: res.statusCode,
            duration: `${Date.now() - start}ms`
        });
    });
    next();
});
```

## Graceful Shutdown

### Handling Process Signals
```javascript
const http = require("http");
const express = require("express");
const mongoose = require("mongoose");
const { Pool } = require("pg");

// Minimal Express app for the HTTP server
const app = express();
app.get("/", (req, res) => {
    res.send("Server is running");
});

// Example PostgreSQL pool (configure as needed for your environment)
const pool = new Pool({
    connectionString: process.env.DATABASE_URL
});

// Assume mongoose.connect(...) is called during app startup

const server = http.createServer(app);
server.listen(3000);

// Graceful shutdown
async function shutdown(signal) {
    console.log(`\n${signal} received. Shutting down gracefully...`);

    // Stop accepting new connections
    server.close(async () => {
        console.log("HTTP server closed");

        try {
            // Close database connections
            await mongoose.connection.close();
            await pool.end();
            console.log("Database connections closed");
            process.exit(0);
        } catch (err) {
            console.error("Error during shutdown:", err);
            process.exit(1);
        }
    });

    // Force shutdown after 30 seconds
    setTimeout(() => {
        console.error("Forced shutdown after timeout");
        process.exit(1);
    }, 30000);
}

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));
```

## Best Practices (Do's)

✅ **Use streams for large data**
```javascript
// Good - memory efficient
app.get("/download", (req, res) => {
    const fileStream = fs.createReadStream("large-file.csv");
    res.setHeader("Content-Type", "text/csv");
    fileStream.pipe(res);
});

// Bad - loads entire file into memory
app.get("/download", async (req, res) => {
    const data = await fs.promises.readFile("large-file.csv");
    res.send(data);
});
```

✅ **Implement proper health check endpoints**
```javascript
app.get("/health", (req, res) => {
    res.json({
        status: "healthy",
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        memory: process.memoryUsage()
    });
});
```

✅ **Use a process manager in production**
```bash
# Install PM2
npm install -g pm2

# Start application
pm2 start app.js --name "my-app" -i max  # Cluster mode

# Monitor
pm2 monit

# Auto-restart on reboot
pm2 startup
pm2 save
```

## Common Mistakes (Don'ts)

❌ **Don't use synchronous methods in request handlers**
```javascript
// Bad - blocks the event loop for all requests
app.get("/data", (req, res) => {
    const data = fs.readFileSync("large-file.txt");  // Blocks!
    res.send(data);
});

// Good
app.get("/data", async (req, res, next) => {
    try {
        const data = await fs.promises.readFile("large-file.txt");
        res.send(data);
    } catch (err) {
        next(err);
    }
});
```

❌ **Don't let unhandled promise rejections crash your server**
```javascript
// Bad - unhandled rejection
async function getUser(id) {
    return User.findById(id);  // If this rejects, it's unhandled!
}

// Good
process.on("unhandledRejection", (reason, promise) => {
    logger.error("Unhandled Rejection:", { reason, promise });
    // Gracefully shut down
    shutdown("UNHANDLED_REJECTION");
});
```

❌ **Don't expose stack traces in production**
```javascript
// Bad
app.use((err, req, res, next) => {
    res.status(500).json({ error: err.stack });  // Exposes internals!
});

// Good
app.use((err, req, res, next) => {
    logger.error(err);
    const statusCode = err.statusCode || 500;
    res.status(statusCode).json({
        error: process.env.NODE_ENV === "production"
            ? "Internal Server Error"
            : err.message
    });
});
```
