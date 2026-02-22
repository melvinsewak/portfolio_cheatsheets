# Node.js Cheatsheet

Welcome to the Node.js cheatsheet! This guide will help you quickly get up to speed with Node.js development.

## Overview

Node.js is a cross-platform, open-source JavaScript runtime environment built on Chrome's V8 engine. It enables JavaScript to run on the server side, making it possible to build fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient.

## Contents

- [Beginner.md](Beginner.md) - Modules, npm, file system, events, and HTTP basics
- [Intermediate.md](Intermediate.md) - Express.js, REST APIs, middleware, databases, and async patterns
- [Advanced.md](Advanced.md) - Streams, clustering, performance, testing, and security

## Quick Reference

### Basic Setup
```bash
# Check Node.js version
node --version

# Run a script
node app.js

# Start interactive REPL
node

# Initialize a new project
npm init -y
```

### Hello World
```javascript
// app.js
console.log("Hello, World!");

// Simple HTTP server
const http = require("http");

const server = http.createServer((req, res) => {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello, World!\n");
});

server.listen(3000, () => {
    console.log("Server running at http://localhost:3000/");
});
```

### Key Features

- **Non-blocking I/O**: Handles many connections concurrently without threads
- **Event-driven**: Uses an event loop for asynchronous operations
- **npm Ecosystem**: Access to the world's largest software registry
- **Single Language**: Use JavaScript for both frontend and backend
- **Fast Execution**: Built on V8, Google's high-performance JavaScript engine

## Running Node.js

### Installation
```bash
# Download from https://nodejs.org
# Or use a version manager (recommended)

# Using nvm (Node Version Manager)
nvm install --lts
nvm use --lts
```

### npm Basics
```bash
# Install a package
npm install express

# Install as dev dependency
npm install --save-dev nodemon

# Install globally
npm install -g pm2

# Run scripts defined in package.json
npm start
npm test
npm run dev
```

## Additional Resources

- [Node.js Official Docs](https://nodejs.org/en/docs/)
- [npm Documentation](https://docs.npmjs.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
