# TypeScript Intermediate Cheatsheet

## Interfaces

### Basic Interface
```typescript
interface User {
    name: string;
    age: number;
    email: string;
}

const user: User = {
    name: "Alice",
    age: 25,
    email: "alice@example.com"
};
```

### Optional and Readonly Properties
```typescript
interface Product {
    id: number;
    name: string;
    price: number;
    description?: string;  // Optional
    readonly sku: string;  // Cannot be modified
}

const product: Product = {
    id: 1,
    name: "Laptop",
    price: 999,
    sku: "LAP-001"
};

// product.sku = "LAP-002";  // Error: readonly
```

### Function Interfaces
```typescript
interface MathOperation {
    (a: number, b: number): number;
}

const add: MathOperation = (a, b) => a + b;
const multiply: MathOperation = (a, b) => a * b;

// Method signatures
interface Calculator {
    add(a: number, b: number): number;
    subtract(a: number, b: number): number;
}
```

### Extending Interfaces
```typescript
interface Person {
    name: string;
    age: number;
}

interface Employee extends Person {
    employeeId: number;
    department: string;
}

const employee: Employee = {
    name: "Bob",
    age: 30,
    employeeId: 1001,
    department: "IT"
};

// Multiple inheritance
interface Manager extends Employee, Person {
    teamSize: number;
}
```

### Index Signatures
```typescript
interface StringDictionary {
    [key: string]: string;
}

const translations: StringDictionary = {
    hello: "Hola",
    goodbye: "Adiós"
};

// Numeric index
interface NumberArray {
    [index: number]: number;
}
```

## Classes

### Basic Class
```typescript
class Person {
    // Properties
    name: string;
    age: number;
    
    // Constructor
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
    
    // Method
    introduce(): string {
        return `Hi, I'm ${this.name} and I'm ${this.age} years old.`;
    }
}

const person = new Person("Alice", 25);
console.log(person.introduce());
```

### Access Modifiers
```typescript
class BankAccount {
    public accountNumber: string;      // Accessible everywhere
    private balance: number;           // Only within class
    protected owner: string;           // Within class and subclasses
    
    constructor(accountNumber: string, owner: string, initialBalance: number) {
        this.accountNumber = accountNumber;
        this.owner = owner;
        this.balance = initialBalance;
    }
    
    public deposit(amount: number): void {
        this.balance += amount;
    }
    
    public getBalance(): number {
        return this.balance;
    }
}
```

### Property Shorthand
```typescript
class User {
    // Shorthand: declare and initialize in constructor
    constructor(
        public name: string,
        public age: number,
        private email: string
    ) {}
    
    getEmail(): string {
        return this.email;
    }
}

const user = new User("Alice", 25, "alice@example.com");
```

### Getters and Setters
```typescript
class Circle {
    private _radius: number = 0;
    
    get radius(): number {
        return this._radius;
    }
    
    set radius(value: number) {
        if (value < 0) {
            throw new Error("Radius cannot be negative");
        }
        this._radius = value;
    }
    
    get area(): number {
        return Math.PI * this._radius ** 2;
    }
}

const circle = new Circle();
circle.radius = 5;
console.log(circle.area);
```

### Inheritance
```typescript
class Animal {
    constructor(public name: string) {}
    
    makeSound(): void {
        console.log("Some generic sound");
    }
}

class Dog extends Animal {
    constructor(name: string, public breed: string) {
        super(name);  // Call parent constructor
    }
    
    makeSound(): void {
        console.log("Woof!");
    }
    
    fetch(): void {
        console.log(`${this.name} is fetching`);
    }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.makeSound();
dog.fetch();
```

### Abstract Classes
```typescript
abstract class Shape {
    abstract calculateArea(): number;
    
    // Concrete method
    describe(): void {
        console.log(`Area: ${this.calculateArea()}`);
    }
}

class Rectangle extends Shape {
    constructor(public width: number, public height: number) {
        super();
    }
    
    calculateArea(): number {
        return this.width * this.height;
    }
}

// const shape = new Shape();  // Error: cannot instantiate abstract class
const rectangle = new Rectangle(10, 5);
rectangle.describe();
```

### Implementing Interfaces
```typescript
interface Drawable {
    draw(): void;
}

interface Resizable {
    resize(width: number, height: number): void;
}

class Canvas implements Drawable, Resizable {
    draw(): void {
        console.log("Drawing on canvas");
    }
    
    resize(width: number, height: number): void {
        console.log(`Resizing to ${width}x${height}`);
    }
}
```

## Generics

### Generic Functions
```typescript
// Generic function
function identity<T>(arg: T): T {
    return arg;
}

let result1 = identity<string>("Hello");  // Explicit type
let result2 = identity(42);               // Type inference

// Generic with constraints
function getLength<T extends { length: number }>(arg: T): number {
    return arg.length;
}

getLength("Hello");      // OK
getLength([1, 2, 3]);    // OK
// getLength(123);       // Error: number doesn't have length
```

### Generic Interfaces
```typescript
interface Box<T> {
    value: T;
}

let numberBox: Box<number> = { value: 42 };
let stringBox: Box<string> = { value: "Hello" };

// Generic function interface
interface GenericIdentityFn<T> {
    (arg: T): T;
}

let myIdentity: GenericIdentityFn<number> = (arg) => arg;
```

### Generic Classes
```typescript
class DataStore<T> {
    private data: T[] = [];
    
    add(item: T): void {
        this.data.push(item);
    }
    
    get(index: number): T {
        return this.data[index];
    }
    
    getAll(): T[] {
        return this.data;
    }
}

const numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);

const stringStore = new DataStore<string>();
stringStore.add("Hello");
stringStore.add("World");
```

### Multiple Type Parameters
```typescript
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second];
}

let result = pair<string, number>("Age", 25);

// Generic constraints with multiple types
function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
    return { ...obj1, ...obj2 };
}

const merged = merge({ name: "Alice" }, { age: 25 });
```

## Modules

### Export
```typescript
// math.ts
export function add(a: number, b: number): number {
    return a + b;
}

export function subtract(a: number, b: number): number {
    return a - b;
}

export const PI = 3.14159;

// Default export
export default class Calculator {
    multiply(a: number, b: number): number {
        return a * b;
    }
}
```

### Import
```typescript
// app.ts
import Calculator, { add, subtract, PI } from './math';

console.log(add(5, 3));
console.log(PI);

const calc = new Calculator();

// Import all
import * as Math from './math';
Math.add(1, 2);

// Import with alias
import { add as addition } from './math';
addition(1, 2);
```

### Re-exporting
```typescript
// index.ts
export { add, subtract } from './math';
export { default as Calculator } from './calculator';
export * from './utils';
```

## Utility Types

### Partial<T>
```typescript
interface User {
    name: string;
    age: number;
    email: string;
}

// Makes all properties optional
function updateUser(user: User, updates: Partial<User>): User {
    return { ...user, ...updates };
}

updateUser(user, { age: 26 });  // Only update age
```

### Required<T>
```typescript
interface Config {
    host?: string;
    port?: number;
}

// Makes all properties required
const config: Required<Config> = {
    host: "localhost",  // Must provide
    port: 3000          // Must provide
};
```

### Readonly<T>
```typescript
interface Point {
    x: number;
    y: number;
}

const point: Readonly<Point> = { x: 10, y: 20 };
// point.x = 5;  // Error: readonly
```

### Pick<T, K>
```typescript
interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

// Pick specific properties
type UserPreview = Pick<User, 'id' | 'name'>;

const preview: UserPreview = {
    id: 1,
    name: "Alice"
};
```

### Omit<T, K>
```typescript
// Omit specific properties
type UserWithoutEmail = Omit<User, 'email'>;

const user: UserWithoutEmail = {
    id: 1,
    name: "Alice",
    age: 25
};
```

### Record<K, T>
```typescript
// Create object type with specific keys and value type
type Roles = 'admin' | 'user' | 'guest';

const permissions: Record<Roles, string[]> = {
    admin: ['read', 'write', 'delete'],
    user: ['read', 'write'],
    guest: ['read']
};
```

## Type Guards

### typeof Type Guards
```typescript
function printValue(value: string | number): void {
    if (typeof value === "string") {
        console.log(value.toUpperCase());
    } else {
        console.log(value.toFixed(2));
    }
}
```

### instanceof Type Guards
```typescript
class Dog {
    bark(): void {
        console.log("Woof!");
    }
}

class Cat {
    meow(): void {
        console.log("Meow!");
    }
}

function makeSound(animal: Dog | Cat): void {
    if (animal instanceof Dog) {
        animal.bark();
    } else {
        animal.meow();
    }
}
```

### Custom Type Guards
```typescript
interface Fish {
    swim(): void;
}

interface Bird {
    fly(): void;
}

// Type predicate
function isFish(animal: Fish | Bird): animal is Fish {
    return (animal as Fish).swim !== undefined;
}

function move(animal: Fish | Bird): void {
    if (isFish(animal)) {
        animal.swim();
    } else {
        animal.fly();
    }
}
```

## Best Practices (Do's)

✅ **Prefer interfaces over type aliases for object shapes**
```typescript
// Good
interface User {
    name: string;
    age: number;
}

// Less preferred (unless needed for unions/intersections)
type User = {
    name: string;
    age: number;
};
```

✅ **Use generics to create reusable components**
```typescript
// Good
function wrapInArray<T>(value: T): T[] {
    return [value];
}

// Less flexible
function wrapNumberInArray(value: number): number[] {
    return [value];
}
```

✅ **Use utility types to transform existing types**
```typescript
// Good - reuses existing type
type UserUpdate = Partial<User>;

// Avoid - duplicates structure
type UserUpdate = {
    name?: string;
    age?: number;
};
```

✅ **Use readonly for immutable data**
```typescript
interface Config {
    readonly apiUrl: string;
    readonly timeout: number;
}
```

## Common Mistakes (Don'ts)

❌ **Don't use public keyword unnecessarily (it's default)**
```typescript
// Redundant
class User {
    public name: string;
}

// Better
class User {
    name: string;
}
```

❌ **Don't forget to implement all interface members**
```typescript
interface Drawable {
    draw(): void;
    clear(): void;
}

// Error: missing 'clear' method
class Canvas implements Drawable {
    draw(): void {
        console.log("Drawing");
    }
}
```

❌ **Don't use any in generic functions**
```typescript
// Bad
function identity<T>(arg: any): any {
    return arg;
}

// Good
function identity<T>(arg: T): T {
    return arg;
}
```

❌ **Don't forget super() in derived class constructors**
```typescript
class Animal {
    constructor(public name: string) {}
}

class Dog extends Animal {
    constructor(name: string, public breed: string) {
        super(name);  // Required!
    }
}
```
