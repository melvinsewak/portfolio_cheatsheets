# TypeScript Advanced Cheatsheet

## Advanced Types

### Intersection Types
```typescript
interface Nameable {
    name: string;
}

interface Ageable {
    age: number;
}

// Combine types
type Person = Nameable & Ageable;

const person: Person = {
    name: "Alice",
    age: 25
};

// Complex intersection
type Employee = Person & {
    employeeId: number;
    department: string;
};
```

### Union Types with Type Guards
```typescript
type Result<T> = 
    | { success: true; data: T }
    | { success: false; error: string };

function handleResult<T>(result: Result<T>): void {
    if (result.success) {
        console.log("Data:", result.data);
    } else {
        console.log("Error:", result.error);
    }
}
```

### Discriminated Unions
```typescript
interface Circle {
    kind: "circle";
    radius: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

interface Triangle {
    kind: "triangle";
    base: number;
    height: number;
}

type Shape = Circle | Rectangle | Triangle;

function calculateArea(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "rectangle":
            return shape.width * shape.height;
        case "triangle":
            return (shape.base * shape.height) / 2;
        default: {
            const _exhaustiveCheck: never = shape;
            throw new Error(`Unhandled shape kind: ${JSON.stringify(_exhaustiveCheck)}`);
        }
    }
}
```

### Conditional Types
```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;   // true
type B = IsString<number>;   // false

// Practical example - Custom NonNullable implementation
type MyNonNullable<T> = T extends null | undefined ? never : T;

type C = MyNonNullable<string | null>;  // string
type D = MyNonNullable<number | undefined>;  // number
```

### Mapped Types
```typescript
// Custom Partial implementation (for demonstration)
type MyPartial<T> = {
    [P in keyof T]?: T[P];
};

// Custom Readonly implementation (for demonstration)
type MyReadonly<T> = {
    readonly [P in keyof T]: T[P];
};

// Custom mapped type
type Nullable<T> = {
    [P in keyof T]: T[P] | null;
};

interface User {
    name: string;
    age: number;
}

type NullableUser = Nullable<User>;
// { name: string | null; age: number | null }
```

### Template Literal Types
```typescript
// String literal types
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type ApiEndpoint = `/api/${string}`;

const endpoint: ApiEndpoint = "/api/users";  // OK
// const bad: ApiEndpoint = "api/users";     // Error

// Template literals in types
type EventName<T extends string> = `on${Capitalize<T>}`;

type ClickEvent = EventName<"click">;  // "onClick"
type HoverEvent = EventName<"hover">;  // "onHover"

// Complex template types
type PropEventSource<T> = {
    on<K extends string & keyof T>(
        eventName: `${K}Changed`,
        callback: (newValue: T[K]) => void
    ): void;
};
```

### Index Accessed Types
```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

// Access type of a property
type UserId = User["id"];  // number
type UserName = User["name"];  // string

// Access multiple properties
type UserInfo = User["name" | "email"];  // string | string = string

// Get all property types
type UserValue = User[keyof User];  // number | string
```

## Advanced Generics

### Generic Constraints
```typescript
// Ensure type has length property
function logLength<T extends { length: number }>(arg: T): void {
    console.log(arg.length);
}

logLength("Hello");      // OK
logLength([1, 2, 3]);    // OK
// logLength(123);       // Error

// Using keyof in constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const person = { name: "Alice", age: 25 };
const name = getProperty(person, "name");  // string
const age = getProperty(person, "age");    // number
```

### Generic Parameter Defaults
```typescript
interface Container<T = string> {
    value: T;
}

const stringContainer: Container = { value: "Hello" };  // Defaults to string
const numberContainer: Container<number> = { value: 42 };
```

### Conditional Type Inference
```typescript
// Infer types within conditional types
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser(): User {
    return { name: "Alice", age: 25 };
}

type UserReturn = ReturnType<typeof getUser>;  // User

// Extract array element type
type ElementType<T> = T extends (infer E)[] ? E : T;

type Numbers = ElementType<number[]>;  // number
type Single = ElementType<string>;     // string
```

### Recursive Types
```typescript
// JSON type
type JSONValue = 
    | string
    | number
    | boolean
    | null
    | JSONValue[]
    | { [key: string]: JSONValue };

const data: JSONValue = {
    name: "Alice",
    age: 25,
    hobbies: ["reading", "coding"],
    address: {
        city: "New York",
        zip: 10001
    }
};

// Tree structure
interface TreeNode<T> {
    value: T;
    children?: TreeNode<T>[];
}
```

## Decorators

### Class Decorators
```typescript
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@sealed
class BugReport {
    type = "report";
    title: string;
    
    constructor(title: string) {
        this.title = title;
    }
}
```

### Method Decorators
```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with args:`, args);
        const result = originalMethod.apply(this, args);
        console.log(`Result:`, result);
        return result;
    };
    
    return descriptor;
}

class Calculator {
    @log
    add(a: number, b: number): number {
        return a + b;
    }
}
```

### Property Decorators
```typescript
function required(target: any, propertyKey: string) {
    let value: any;
    
    const getter = () => {
        return value;
    };
    
    const setter = (newVal: any) => {
        if (!newVal) {
            throw new Error(`${propertyKey} is required`);
        }
        value = newVal;
    };
    
    Object.defineProperty(target, propertyKey, {
        get: getter,
        set: setter,
        enumerable: true,
        configurable: true
    });
}

class User {
    @required
    name: string;
}
```

### Parameter Decorators
```typescript
function validate(target: any, propertyKey: string, parameterIndex: number) {
    console.log(`Validating parameter ${parameterIndex} of ${propertyKey}`);
}

class UserService {
    create(@validate name: string, @validate age: number): void {
        console.log(`Creating user: ${name}, ${age}`);
    }
}
```

## Advanced Patterns

### Builder Pattern
```typescript
class UserBuilder {
    private user: Partial<User> = {};
    
    setName(name: string): this {
        this.user.name = name;
        return this;
    }
    
    setAge(age: number): this {
        this.user.age = age;
        return this;
    }
    
    setEmail(email: string): this {
        this.user.email = email;
        return this;
    }
    
    build(): User {
        if (!this.user.name || !this.user.age || !this.user.email) {
            throw new Error("Missing required fields");
        }
        return this.user as User;
    }
}

const user = new UserBuilder()
    .setName("Alice")
    .setAge(25)
    .setEmail("alice@example.com")
    .build();
```

### Factory Pattern with Generics
```typescript
interface Animal {
    speak(): void;
}

class Dog implements Animal {
    speak(): void {
        console.log("Woof!");
    }
}

class Cat implements Animal {
    speak(): void {
        console.log("Meow!");
    }
}

class AnimalFactory {
    static create<T extends Animal>(type: new () => T): T {
        return new type();
    }
}

const dog = AnimalFactory.create(Dog);
const cat = AnimalFactory.create(Cat);
```

### Singleton Pattern
```typescript
class Database {
    private static instance: Database;
    private connection: any;
    
    private constructor() {
        // Private constructor prevents instantiation
        this.connection = this.connect();
    }
    
    static getInstance(): Database {
        if (!Database.instance) {
            Database.instance = new Database();
        }
        return Database.instance;
    }
    
    private connect(): any {
        console.log("Connecting to database...");
        return {};
    }
    
    query(sql: string): void {
        console.log(`Executing: ${sql}`);
    }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2);  // true
```

### Observer Pattern
```typescript
interface Observer<T> {
    update(data: T): void;
}

class Subject<T> {
    private observers: Observer<T>[] = [];
    
    subscribe(observer: Observer<T>): void {
        this.observers.push(observer);
    }
    
    unsubscribe(observer: Observer<T>): void {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
            this.observers.splice(index, 1);
        }
    }
    
    notify(data: T): void {
        this.observers.forEach(observer => observer.update(data));
    }
}

class DataLogger implements Observer<string> {
    update(data: string): void {
        console.log(`Logged: ${data}`);
    }
}

const subject = new Subject<string>();
const logger = new DataLogger();
subject.subscribe(logger);
subject.notify("Event occurred");
```

## Type Manipulation

### keyof and typeof
```typescript
interface User {
    name: string;
    age: number;
    email: string;
}

// keyof - get union of keys
type UserKeys = keyof User;  // "name" | "age" | "email"

// typeof - get type of a value
const user = { name: "Alice", age: 25 };
type UserType = typeof user;  // { name: string; age: number }

// Combine them
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}
```

### ReturnType and Parameters
```typescript
function createUser(name: string, age: number): User {
    return { name, age, email: "" };
}

// Get return type
type UserReturnType = ReturnType<typeof createUser>;  // User

// Get parameters type
type UserParams = Parameters<typeof createUser>;  // [string, number]
```

### Awaited Type
```typescript
// Extract type from Promise
type Result = Awaited<Promise<string>>;  // string

// Nested promises
type NestedResult = Awaited<Promise<Promise<number>>>;  // number

// Useful with async functions
async function fetchUser(): Promise<User> {
    return { name: "Alice", age: 25, email: "alice@example.com" };
}

type FetchedUser = Awaited<ReturnType<typeof fetchUser>>;  // User
```

## Module Augmentation

### Extending External Modules
```typescript
// Extend express Request interface
declare global {
    namespace Express {
        interface Request {
            user?: User;
        }
    }
}

// Now TypeScript knows about req.user
app.get('/profile', (req, res) => {
    if (req.user) {
        res.json(req.user);
    }
});
```

### Ambient Declarations
```typescript
// Declare types for external libraries without types
declare module 'my-library' {
    export function doSomething(value: string): number;
    export const VERSION: string;
}

// Declare global variables
declare const GLOBAL_CONFIG: {
    apiUrl: string;
    timeout: number;
};
```

## Best Practices (Do's)

✅ **Use discriminated unions for state management**
```typescript
type RequestState<T> =
    | { status: "idle" }
    | { status: "loading" }
    | { status: "success"; data: T }
    | { status: "error"; error: string };

function renderState<T>(state: RequestState<T>) {
    switch (state.status) {
        case "idle":
            return "Not started";
        case "loading":
            return "Loading...";
        case "success":
            return `Data: ${state.data}`;
        case "error":
            return `Error: ${state.error}`;
    }
}
```

✅ **Use const assertions for literal types**
```typescript
// Good - inferred as specific literals
const config = {
    host: "localhost",
    port: 3000
} as const;

type Config = typeof config;
// { readonly host: "localhost"; readonly port: 3000 }
```

✅ **Leverage utility types for transformations**
```typescript
// Good
type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
type RequiredUser = Required<User>;

// Avoid recreating manually
```

✅ **Use branded types for type safety**
```typescript
type UserId = string & { __brand: "UserId" };
type ProductId = string & { __brand: "ProductId" };

function getUser(id: UserId): User {
    // ...
}

// Prevents mixing up IDs
const userId = "123" as UserId;
const productId = "456" as ProductId;

getUser(userId);       // OK
// getUser(productId); // Error
```

## Common Mistakes (Don'ts)

❌ **Don't use type assertions to bypass type checking**
```typescript
// Bad
const user = {} as User;  // Missing properties!

// Good
const user: User = {
    name: "Alice",
    age: 25,
    email: "alice@example.com"
};
```

❌ **Don't overuse any in generic contexts**
```typescript
// Bad
function process<T>(data: any): any {
    return data;
}

// Good
function process<T>(data: T): T {
    return data;
}
```

❌ **Don't forget to handle all discriminated union cases**
```typescript
// Bad - missing case
function handle(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "rectangle":
            return shape.width * shape.height;
        // Missing triangle case!
    }
}

// Good - exhaustive check
function handle(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "rectangle":
            return shape.width * shape.height;
        case "triangle":
            return (shape.base * shape.height) / 2;
        default:
            const _exhaustive: never = shape;
            return _exhaustive;
    }
}
```

❌ **Don't mutate readonly types**
```typescript
interface Config {
    readonly apiUrl: string;
}

const config: Config = { apiUrl: "https://api.example.com" };

// Bad
// config.apiUrl = "https://new-api.example.com";  // Error

// Good - create new object
const newConfig = { ...config, apiUrl: "https://new-api.example.com" };
```
