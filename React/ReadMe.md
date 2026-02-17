# React Cheatsheet

Welcome to the React cheatsheet! This guide will help you quickly get up to speed with React development.

## Overview

React is a JavaScript library for building user interfaces, particularly single-page applications. Developed and maintained by Meta (Facebook), React allows developers to create reusable UI components and manage application state efficiently.

## Contents

- [Beginner.md](Beginner.md) - Components, JSX, props, and state basics
- [Intermediate.md](Intermediate.md) - Hooks, lifecycle, forms, and routing
- [Advanced.md](Advanced.md) - Performance optimization, Context API, custom hooks, and patterns

## Quick Reference

### Basic Component
```jsx
import React from 'react';

function Welcome({ name }) {
    return <h1>Hello, {name}!</h1>;
}

export default Welcome;
```

### Component with State
```jsx
import { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
        </div>
    );
}
```

### Key Features

- **Component-Based**: Build encapsulated components that manage their own state
- **Declarative**: Design views for each state, React updates efficiently
- **Virtual DOM**: Efficient rendering with minimal DOM manipulation
- **Unidirectional Data Flow**: Predictable data flow from parent to child
- **Rich Ecosystem**: Huge community and extensive third-party libraries

## Getting Started

### Create React App
```bash
npx create-react-app my-app
cd my-app
npm start
```

### Vite (Faster Alternative)
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

## Project Structure
```
my-app/
├── public/
├── src/
│   ├── components/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── package.json
└── vite.config.js
```

## Additional Resources

- [Official React Documentation](https://react.dev/)
- [React Tutorial](https://react.dev/learn)
- [React Patterns](https://reactpatterns.com/)
