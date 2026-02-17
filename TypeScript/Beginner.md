# TypeScript Beginner Cheatsheet

## Basic Types

### Primitive Types
```typescript
// String
let name: string = "Alice";
let message: string = `Hello, ${name}!`;

// Number (integers and floats)
let age: number = 25;
let price: number = 19.99;
let hex: number = 0xf00d;

// Boolean
let isActive: boolean = true;
let isCompleted: boolean = false;

// Undefined and Null
let nothing: undefined = undefined;
let empty: null = null;
```

### Arrays
```typescript
// Array of numbers
let numbers: number[] = [1, 2, 3, 4, 5];
let scores: Array<number> = [85, 90, 95];

// Array of strings
let names: string[] = ["Alice", "Bob", "Charlie"];

// Mixed array (use union types)
let mixed: (string | number)[] = ["Alice", 25, "Bob", 30];
```

### Tuples
```typescript
// Fixed-length array with specific types
let person: [string, number] = ["Alice", 25];

// Accessing elements
let name: string = person[0];
let age: number = person[1];

// Optional tuple elements
let optional: [string, number?] = ["Bob"];
```

### Enums
```typescript
// Numeric enum
enum Direction {
    Up,      // 0
    Down,    // 1
    Left,    // 2
    Right    // 3
}

let dir: Direction = Direction.Up;

// String enum
enum Status {
    Active = "ACTIVE",
    Inactive = "INACTIVE",
    Pending = "PENDING"
}

let status: Status = Status.Active;
```

### Any and Unknown
```typescript
// Any - disables type checking
let anything: any = "Hello";
anything = 42;
anything = true;

// Unknown - safer than any
let value: unknown = "Hello";

if (typeof value === "string") {
    console.log(value.toUpperCase());  // OK after type check
}
```

## Type Annotations

### Variables
```typescript
// Explicit type annotation
let username: string = "Alice";
let count: number = 10;

// Type inference (TypeScript infers the type)
let age = 25;  // Inferred as number
let name = "Bob";  // Inferred as string
```

### Functions
```typescript
// Function with type annotations
function add(a: number, b: number): number {
    return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => {
    return a * b;
};

// Function with no return value
function logMessage(message: string): void {
    console.log(message);
}

// Optional parameters
function greet(name: string, greeting?: string): string {
    return `${greeting || "Hello"}, ${name}!`;
}

// Default parameters
function createUser(name: string, age: number = 18): void {
    console.log(`${name} is ${age} years old`);
}

// Rest parameters
function sum(...numbers: number[]): number {
    return numbers.reduce((acc, num) => acc + num, 0);
}
```

## Type Assertions

```typescript
// Type assertion (telling TypeScript the type)
let value: any = "Hello World";
let length: number = (value as string).length;

// Alternative syntax (not recommended in JSX)
let length2: number = (<string>value).length;

// Non-null assertion
let name: string | null = "Alice";
let nameLength: number = name!.length;  // Asserting it's not null
```

## Union Types

```typescript
// Variable can be multiple types
let id: number | string;
id = 101;      // OK
id = "A101";   // OK

// Function with union type
function printId(id: number | string): void {
    if (typeof id === "string") {
        console.log(`ID: ${id.toUpperCase()}`);
    } else {
        console.log(`ID: ${id}`);
    }
}

// Array with union types
let mixed: (number | string)[] = [1, "two", 3, "four"];
```

## Type Aliases

```typescript
// Create custom type names
type StringOrNumber = string | number;
type User = {
    name: string;
    age: number;
};

let id: StringOrNumber = 123;
let user: User = {
    name: "Alice",
    age: 25
};

// Function type alias
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const multiply: MathOperation = (a, b) => a * b;
```

## Literal Types

```typescript
// Specific string values
let direction: "up" | "down" | "left" | "right";
direction = "up";    // OK
// direction = "forward";  // Error

// Numeric literals
let diceRoll: 1 | 2 | 3 | 4 | 5 | 6;
diceRoll = 3;  // OK

// Boolean literal
let success: true = true;
```

## Objects

```typescript
// Object type
let person: { name: string; age: number } = {
    name: "Alice",
    age: 25
};

// Optional properties
let user: { name: string; age?: number } = {
    name: "Bob"
};

// Readonly properties
let config: { readonly apiKey: string } = {
    apiKey: "abc123"
};
// config.apiKey = "xyz";  // Error: readonly

// Index signatures
let scores: { [subject: string]: number } = {
    math: 90,
    english: 85,
    science: 95
};
```

## Type Narrowing

```typescript
// Type guards with typeof
function printValue(value: string | number): void {
    if (typeof value === "string") {
        console.log(value.toUpperCase());
    } else {
        console.log(value.toFixed(2));
    }
}

// Truthiness narrowing
function printLength(text: string | null): void {
    if (text) {
        console.log(text.length);
    } else {
        console.log("No text provided");
    }
}

// Equality narrowing
function compare(x: string | number, y: string | boolean): void {
    if (x === y) {
        // x and y are both string
        console.log(x.toUpperCase(), y.toUpperCase());
    }
}
```

## Working with Arrays

```typescript
// Array methods with types
let numbers: number[] = [1, 2, 3, 4, 5];

// Map
let doubled: number[] = numbers.map(n => n * 2);

// Filter
let evens: number[] = numbers.filter(n => n % 2 === 0);

// Reduce
let sum: number = numbers.reduce((acc, n) => acc + n, 0);

// Find
let found: number | undefined = numbers.find(n => n > 3);

// Some and Every
let hasEven: boolean = numbers.some(n => n % 2 === 0);
let allPositive: boolean = numbers.every(n => n > 0);
```

## Type Inference

```typescript
// TypeScript infers types
let message = "Hello";  // string
let count = 42;         // number
let isActive = true;    // boolean

// Function return type inference
function add(a: number, b: number) {
    return a + b;  // Inferred as number
}

// Array type inference
let numbers = [1, 2, 3];  // number[]
let mixed = [1, "two", 3];  // (string | number)[]

// Best of both worlds
const person = {
    name: "Alice",
    age: 25
};  // Inferred as { name: string; age: number }
```

## Best Practices (Do's)

✅ **Use explicit types for function parameters and return types**
```typescript
// Good
function calculateTotal(price: number, quantity: number): number {
    return price * quantity;
}

// Avoid (parameters should be typed)
function calculateTotal(price, quantity) {
    return price * quantity;
}
```

✅ **Enable strict mode in tsconfig.json**
```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

✅ **Use const for values that won't change**
```typescript
// Good
const API_URL = "https://api.example.com";

// Less ideal
let API_URL = "https://api.example.com";
```

✅ **Prefer interfaces or type aliases over inline types**
```typescript
// Good
type User = {
    name: string;
    email: string;
};

function createUser(user: User): void {
    // ...
}

// Less maintainable
function createUser(user: { name: string; email: string }): void {
    // ...
}
```

✅ **Use union types instead of any when possible**
```typescript
// Good
function printId(id: string | number): void {
    console.log(id);
}

// Avoid
function printId(id: any): void {
    console.log(id);
}
```

## Common Mistakes (Don'ts)

❌ **Don't overuse 'any' type**
```typescript
// Bad - defeats the purpose of TypeScript
let data: any = fetchData();

// Good - use proper types
interface ApiResponse {
    status: number;
    data: User[];
}

let data: ApiResponse = fetchData();
```

❌ **Don't ignore TypeScript errors**
```typescript
// Bad - using @ts-ignore to suppress errors
// @ts-ignore
let result = someFunction();

// Good - fix the underlying issue
let result: ExpectedType = someFunction();
```

❌ **Don't forget to check for null/undefined**
```typescript
// Risky
function getNameLength(name: string | null): number {
    return name.length;  // Error if name is null
}

// Safe
function getNameLength(name: string | null): number {
    return name?.length || 0;
}
```

❌ **Don't use type assertions unnecessarily**
```typescript
// Bad - lying to TypeScript
let value = "123" as number;  // Wrong!

// Good - proper conversion
let value = parseInt("123");
```

❌ **Don't mix types in arrays without union types**
```typescript
// Confusing
let data = [];
data.push(1);
data.push("two");

// Clear
let data: (number | string)[] = [];
data.push(1);
data.push("two");
```
