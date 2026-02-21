# Node.js Beginner Cheatsheet

## Modules and require

### CommonJS Modules (require/module.exports)
```javascript
// math.js - Exporting
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

module.exports = { add, subtract };

// app.js - Importing
const { add, subtract } = require("./math");

console.log(add(5, 3));       // 8
console.log(subtract(10, 4)); // 6
```

### ES Modules (import/export)
```javascript
// math.mjs or with "type": "module" in package.json
// math.js - Exporting
export function add(a, b) {
    return a + b;
}

export default function multiply(a, b) {
    return a * b;
}

// app.js - Importing
import multiply, { add } from "./math.js";

console.log(add(5, 3));       // 8
console.log(multiply(4, 3));  // 12
```

### Built-in Modules
```javascript
// No installation needed - these come with Node.js
const path = require("path");
const fs = require("fs");
const os = require("os");
const http = require("http");
const url = require("url");
const crypto = require("crypto");
```

## npm and package.json

### Initializing a Project
```bash
# Create package.json interactively
npm init

# Create package.json with defaults
npm init -y
```

### package.json Structure
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.0.0"
  }
}
```

### Managing Packages
```bash
# Install all dependencies from package.json
npm install

# Install a specific package (saved to dependencies)
npm install express

# Install a dev dependency
npm install --save-dev nodemon

# Uninstall a package
npm uninstall express

# List installed packages
npm list

# Check for outdated packages
npm outdated

# Update packages
npm update
```

## File System (fs module)

### Reading Files
```javascript
const fs = require("fs");

// Asynchronous (non-blocking) - preferred
fs.readFile("data.txt", "utf8", (err, data) => {
    if (err) {
        console.error("Error reading file:", err);
        return;
    }
    console.log(data);
});

// Synchronous (blocking) - use sparingly
try {
    const data = fs.readFileSync("data.txt", "utf8");
    console.log(data);
} catch (err) {
    console.error("Error:", err);
}

// Promise-based (modern approach)
const fsPromises = require("fs").promises;

async function readFile() {
    try {
        const data = await fsPromises.readFile("data.txt", "utf8");
        console.log(data);
    } catch (err) {
        console.error("Error:", err);
    }
}
readFile();
```

### Writing Files
```javascript
const fs = require("fs");

// Write (creates or overwrites)
fs.writeFile("output.txt", "Hello, Node.js!", "utf8", (err) => {
    if (err) throw err;
    console.log("File written successfully!");
});

// Append to file
fs.appendFile("output.txt", "\nAppended line", "utf8", (err) => {
    if (err) throw err;
    console.log("Content appended!");
});
```

### Working with Directories
```javascript
const fs = require("fs");

// Create directory
fs.mkdir("new-folder", { recursive: true }, (err) => {
    if (err) throw err;
    console.log("Directory created!");
});

// Read directory contents
fs.readdir(".", (err, files) => {
    if (err) throw err;
    files.forEach(file => console.log(file));
});

// Check if path exists
fs.access("data.txt", fs.constants.F_OK, (err) => {
    console.log(err ? "File does not exist" : "File exists");
});
```

## Path Module

### Working with File Paths
```javascript
const path = require("path");

// Join paths (cross-platform safe)
const filePath = path.join(__dirname, "data", "users.json");
console.log(filePath);  // /project/data/users.json

// Get file extension
console.log(path.extname("index.html"));  // ".html"

// Get filename
console.log(path.basename("/folder/file.txt"));       // "file.txt"
console.log(path.basename("/folder/file.txt", ".txt")); // "file"

// Get directory name
console.log(path.dirname("/folder/file.txt"));  // "/folder"

// Resolve absolute path
console.log(path.resolve("data", "file.txt"));  // /current/dir/data/file.txt

// __dirname and __filename (CommonJS)
console.log(__dirname);   // Current directory
console.log(__filename);  // Current file path
```

## Events (EventEmitter)

### Creating and Using Event Emitters
```javascript
const EventEmitter = require("events");

// Create an event emitter
const emitter = new EventEmitter();

// Register an event listener
emitter.on("greet", (name) => {
    console.log(`Hello, ${name}!`);
});

// Emit the event
emitter.emit("greet", "Alice");  // Hello, Alice!

// One-time listener
emitter.once("connect", () => {
    console.log("Connected! This runs only once.");
});

// Remove a listener
const handler = () => console.log("data received");
emitter.on("data", handler);
emitter.off("data", handler);  // or emitter.removeListener("data", handler)
```

### Custom EventEmitter Class
```javascript
const EventEmitter = require("events");

class Logger extends EventEmitter {
    log(message) {
        console.log(message);
        this.emit("log", { message, timestamp: new Date() });
    }
}

const logger = new Logger();

logger.on("log", (data) => {
    console.log(`Event logged at ${data.timestamp}`);
});

logger.log("Server started");
```

## HTTP Server

### Creating a Basic HTTP Server
```javascript
const http = require("http");

const server = http.createServer((req, res) => {
    // Set response header
    res.writeHead(200, { "Content-Type": "text/html" });
    
    // Send response body
    res.end("<h1>Hello, World!</h1>");
});

server.listen(3000, "localhost", () => {
    console.log("Server running at http://localhost:3000/");
});
```

### Routing Requests
```javascript
const http = require("http");

const server = http.createServer((req, res) => {
    const { method, url } = req;

    if (url === "/" && method === "GET") {
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("Home Page");
    } else if (url === "/about" && method === "GET") {
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("About Page");
    } else {
        res.writeHead(404, { "Content-Type": "text/plain" });
        res.end("404 Not Found");
    }
});

server.listen(3000, () => console.log("Listening on port 3000"));
```

### Sending JSON Responses
```javascript
const http = require("http");

const server = http.createServer((req, res) => {
    const data = { message: "Hello", status: "OK" };

    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify(data));
});

server.listen(3000);
```

## Environment Variables

### Using process.env
```javascript
// Access environment variables
const port = process.env.PORT || 3000;
const nodeEnv = process.env.NODE_ENV || "development";

console.log(`Running on port ${port} in ${nodeEnv} mode`);
```

### Using dotenv Package
```bash
npm install dotenv
```

```javascript
// .env file (never commit to version control!)
// PORT=3000
// DB_URL=mongodb://localhost:27017/mydb
// SECRET_KEY=mysecretkey

// app.js
require("dotenv").config();

const port = process.env.PORT;
const dbUrl = process.env.DB_URL;

console.log(`Connecting to database at ${dbUrl}`);
```

## Process Object

### Useful process Properties and Methods
```javascript
// Command-line arguments
const args = process.argv.slice(2);
console.log(args);  // node app.js arg1 arg2 → ["arg1", "arg2"]

// Current working directory
console.log(process.cwd());

// Node.js version
console.log(process.version);

// Platform
console.log(process.platform);  // "linux", "darwin", "win32"

// Exit the process
process.exit(0);  // 0 = success, 1 = failure

// Handle process events
process.on("exit", (code) => {
    console.log(`Process exiting with code ${code}`);
});

process.on("uncaughtException", (err) => {
    console.error("Uncaught exception:", err);
    process.exit(1);
});
```

## Best Practices (Do's)

✅ **Use async/await instead of callbacks**
```javascript
// Good
const fs = require("fs").promises;

async function readConfig() {
    const data = await fs.readFile("config.json", "utf8");
    return JSON.parse(data);
}

// Avoid
fs.readFile("config.json", "utf8", (err, data) => {
    if (err) throw err;
    const config = JSON.parse(data);
});
```

✅ **Use path.join() for file paths**
```javascript
// Good - works on all operating systems
const filePath = path.join(__dirname, "data", "file.txt");

// Bad - may break on Windows
const filePath = __dirname + "/data/file.txt";
```

✅ **Always handle errors**
```javascript
// Good
fs.readFile("file.txt", "utf8", (err, data) => {
    if (err) {
        console.error("Failed to read file:", err.message);
        return;
    }
    console.log(data);
});

// Bad - unhandled error
fs.readFile("file.txt", "utf8", (err, data) => {
    console.log(data);  // Could throw if err is set
});
```

✅ **Use environment variables for configuration**
```javascript
// Good
const port = process.env.PORT || 3000;
const dbUrl = process.env.DB_URL;

// Bad - hardcoded values
const port = 3000;
const dbUrl = "mongodb://localhost:27017/mydb";
```

## Common Mistakes (Don'ts)

❌ **Don't block the event loop with synchronous code**
```javascript
const express = require("express");
const fs = require("fs");
const app = express();

// Bad - blocks other requests
app.get("/data", (req, res) => {
    const data = fs.readFileSync("large-file.txt");  // Blocks!
    res.send(data);
});

// Good - non-blocking
app.get("/data", async (req, res) => {
    const data = await fs.promises.readFile("large-file.txt");
    res.send(data);
});

app.listen(3000);
```

❌ **Don't commit .env files to version control**
```bash
# .gitignore
.env
node_modules/
```

❌ **Don't ignore errors in async code**
```javascript
// Bad
async function fetchData() {
    const data = await riskyOperation();  // Unhandled rejection!
    return data;
}

// Good
async function fetchData() {
    try {
        const data = await riskyOperation();
        return data;
    } catch (err) {
        console.error("Failed:", err.message);
        throw err;
    }
}
```

❌ **Don't use require() inside loops or conditionals for performance-sensitive code**
```javascript
// Bad - require is called repeatedly
for (let i = 0; i < 100; i++) {
    const helper = require("./helper");  // Cached after first call, but still bad practice
    helper.doWork(i);
}

// Good - require at the top of the file
const helper = require("./helper");
for (let i = 0; i < 100; i++) {
    helper.doWork(i);
}
```
