# TypeScript Cheatsheet

Welcome to the TypeScript cheatsheet! This guide will help you quickly get up to speed with TypeScript programming.

## Overview

TypeScript is a strongly typed superset of JavaScript that compiles to plain JavaScript. Developed by Microsoft, it adds optional static typing, classes, and modules to JavaScript, making it easier to build and maintain large-scale applications.

## Contents

- [Beginner.md](Beginner.md) - Basic types, functions, and fundamental concepts
- [Intermediate.md](Intermediate.md) - Interfaces, classes, generics, and modules
- [Advanced.md](Advanced.md) - Advanced types, decorators, and type manipulation

## Quick Reference

### Basic Syntax
```typescript
// Variable declarations with types
let name: string = "Alice";
let age: number = 25;
let isActive: boolean = true;

// Function with type annotations
function greet(name: string): string {
    return `Hello, ${name}!`;
}

// Interface
interface User {
    name: string;
    age: number;
}

const user: User = {
    name: "Bob",
    age: 30
};
```

### Key Features

- **Static Typing**: Catch errors at compile time
- **Type Inference**: Smart type detection
- **Object-Oriented**: Full support for classes and interfaces
- **Modern JavaScript**: ES6+ features with backward compatibility
- **Tooling Support**: Excellent IDE integration and IntelliSense

## Getting Started

1. Install TypeScript: `npm install -g typescript`
2. Create a TypeScript file: `touch app.ts`
3. Compile to JavaScript: `tsc app.ts`
4. Or use `ts-node` for direct execution: `npx ts-node app.ts`

## Configuration

Create a `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true
  }
}
```

## Additional Resources

- [Official TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [TypeScript Playground](https://www.typescriptlang.org/play)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
