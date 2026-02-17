# JavaScript Cheatsheet

Welcome to the JavaScript cheatsheet! This guide will help you quickly get up to speed with JavaScript programming.

## Overview

JavaScript is a versatile, high-level programming language that powers the interactive web. Originally created for browsers, it now runs everywhere - from servers (Node.js) to mobile apps, desktop applications, and even IoT devices.

## Contents

- [Beginner.md](Beginner.md) - Basics, variables, functions, and fundamental concepts
- [Intermediate.md](Intermediate.md) - Objects, arrays, ES6+ features, and async programming
- [Advanced.md](Advanced.md) - Closures, prototypes, design patterns, and performance optimization

## Quick Reference

### Basic Syntax
```javascript
// Variables
let name = "Alice";
const age = 25;
var oldStyle = "deprecated";

// Function
function greet(name) {
    return `Hello, ${name}!`;
}

// Arrow function
const add = (a, b) => a + b;

// Object
const person = {
    name: "Bob",
    age: 30,
    greet() {
        console.log(`Hi, I'm ${this.name}`);
    }
};

// Array
const numbers = [1, 2, 3, 4, 5];
```

### Key Features

- **Dynamic Typing**: Variables can hold any type of value
- **First-Class Functions**: Functions are values that can be passed around
- **Prototype-Based**: Object inheritance through prototypes
- **Event-Driven**: Perfect for handling user interactions
- **Asynchronous**: Built-in support for async operations

## Running JavaScript

### In Browser
```html
<script>
    console.log("Hello, World!");
</script>
```

### With Node.js
```bash
# Install Node.js first
node script.js
```

### Interactive Console
- Browser: Open DevTools (F12) → Console tab
- Node.js: Run `node` in terminal

## Additional Resources

- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [JavaScript.info](https://javascript.info/)
- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)
