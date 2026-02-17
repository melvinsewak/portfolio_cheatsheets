# JavaScript Intermediate Cheatsheet

## ES6+ Features

### Destructuring

#### Array Destructuring
```javascript
// Basic
const [a, b, c] = [1, 2, 3];

// Skip elements
const [first, , third] = [1, 2, 3];

// Rest operator
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head);  // 1
console.log(tail);  // [2, 3, 4, 5]

// Default values
const [x = 0, y = 0] = [10];
console.log(x, y);  // 10, 0
```

#### Object Destructuring
```javascript
const person = { name: "Alice", age: 25, city: "NYC" };

// Basic
const { name, age } = person;

// Rename variables
const { name: userName, age: userAge } = person;

// Default values
const { country = "USA" } = person;

// Nested destructuring
const user = {
    name: "Bob",
    address: { city: "NYC", zip: "10001" }
};
const { address: { city, zip } } = user;

// Function parameters
function greet({ name, age }) {
    console.log(`${name} is ${age} years old`);
}
greet(person);
```

### Spread Operator
```javascript
// Arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];  // [1, 2, 3, 4, 5, 6]

// Copy array
const copy = [...arr1];

// Objects
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 };

// Copy and override
const original = { name: "Alice", age: 25 };
const updated = { ...original, age: 26 };
```

### Enhanced Object Literals
```javascript
const name = "Alice";
const age = 25;

// Property shorthand
const person = { name, age };

// Method shorthand
const calculator = {
    add(a, b) {
        return a + b;
    },
    multiply(a, b) {
        return a * b;
    }
};

// Computed property names
const key = "dynamicKey";
const obj = {
    [key]: "value",
    [`${key}2`]: "value2"
};
```

## Advanced Array Methods

### Map
```javascript
const numbers = [1, 2, 3, 4, 5];

// Transform array
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8, 10]

// With objects
const users = [
    { name: "Alice", age: 25 },
    { name: "Bob", age: 30 }
];
const names = users.map(user => user.name);
// ["Alice", "Bob"]
```

### Filter
```javascript
const numbers = [1, 2, 3, 4, 5, 6];

// Filter even numbers
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4, 6]

// Filter objects
const adults = users.filter(user => user.age >= 18);
```

### Reduce
```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((acc, num) => acc + num, 0);
// 15

// Find max
const max = numbers.reduce((max, num) => num > max ? num : max);

// Group by property
const users = [
    { name: "Alice", role: "admin" },
    { name: "Bob", role: "user" },
    { name: "Charlie", role: "admin" }
];

const grouped = users.reduce((acc, user) => {
    const { role } = user;
    if (!acc[role]) acc[role] = [];
    acc[role].push(user);
    return acc;
}, {});
```

### Find and FindIndex
```javascript
const users = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
];

// Find first match
const user = users.find(u => u.id === 2);

// Find index
const index = users.findIndex(u => u.id === 2);
```

### Some and Every
```javascript
const numbers = [1, 2, 3, 4, 5];

// Check if any element matches
const hasEven = numbers.some(n => n % 2 === 0);  // true

// Check if all elements match
const allPositive = numbers.every(n => n > 0);   // true
```

## Asynchronous JavaScript

### Callbacks
```javascript
function fetchData(callback) {
    setTimeout(() => {
        callback("Data loaded");
    }, 1000);
}

fetchData((data) => {
    console.log(data);
});
```

### Promises
```javascript
// Creating a promise
const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        const success = true;
        if (success) {
            resolve("Success!");
        } else {
            reject("Error!");
        }
    }, 1000);
});

// Using promises
promise
    .then(result => {
        console.log(result);
        return "Next step";
    })
    .then(result => {
        console.log(result);
    })
    .catch(error => {
        console.error(error);
    })
    .finally(() => {
        console.log("Done");
    });
```

### Async/Await
```javascript
// Async function
async function fetchData() {
    return "Data";
}

// Returns a Promise
fetchData().then(data => console.log(data));

// Using await
async function loadData() {
    try {
        const data = await fetchData();
        console.log(data);
    } catch (error) {
        console.error(error);
    }
}

// Multiple async operations
async function loadMultiple() {
    const [data1, data2] = await Promise.all([
        fetchData1(),
        fetchData2()
    ]);
    console.log(data1, data2);
}
```

### Fetch API
```javascript
// GET request
fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));

// With async/await
async function getData() {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error(error);
    }
}

// POST request
async function postData(payload) {
    const response = await fetch('https://api.example.com/data', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(payload)
    });
    return response.json();
}
```

## Classes (ES6)

### Basic Class
```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        console.log(`Hi, I'm ${this.name}`);
    }
    
    get info() {
        return `${this.name}, ${this.age}`;
    }
    
    set age(value) {
        if (value < 0) throw new Error("Age cannot be negative");
        this._age = value;
    }
}

const person = new Person("Alice", 25);
person.greet();
```

### Inheritance
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        console.log(`${this.name} makes a sound`);
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);  // Call parent constructor
        this.breed = breed;
    }
    
    speak() {
        console.log(`${this.name} barks`);
    }
    
    fetch() {
        console.log(`${this.name} fetches the ball`);
    }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.speak();
dog.fetch();
```

### Static Methods
```javascript
class MathUtils {
    static add(a, b) {
        return a + b;
    }
    
    static PI = 3.14159;
}

console.log(MathUtils.add(5, 3));
console.log(MathUtils.PI);
```

## Modules

### Export
```javascript
// named exports
export const PI = 3.14159;
export function add(a, b) {
    return a + b;
}

// or
const PI = 3.14159;
function add(a, b) {
    return a + b;
}
export { PI, add };

// default export
export default class Calculator {
    multiply(a, b) {
        return a * b;
    }
}
```

### Import
```javascript
// named imports
import { PI, add } from './math.js';

// import all
import * as Math from './math.js';

// default import
import Calculator from './calculator.js';

// rename imports
import { add as addition } from './math.js';

// mixed
import Calculator, { PI, add } from './math.js';
```

## Error Handling

### Try-Catch-Finally
```javascript
try {
    // Code that might throw an error
    const result = riskyOperation();
    console.log(result);
} catch (error) {
    // Handle error
    console.error("Error:", error.message);
} finally {
    // Always executes
    console.log("Cleanup");
}
```

### Throwing Errors
```javascript
function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero");
    }
    return a / b;
}

// Custom error
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = "ValidationError";
    }
}

function validateAge(age) {
    if (age < 0) {
        throw new ValidationError("Age cannot be negative");
    }
}
```

## JSON

### Parse and Stringify
```javascript
// Object to JSON string
const obj = { name: "Alice", age: 25 };
const json = JSON.stringify(obj);
// '{"name":"Alice","age":25}'

// JSON string to object
const parsed = JSON.parse(json);
console.log(parsed.name);  // "Alice"

// Pretty print
const pretty = JSON.stringify(obj, null, 2);

// With replacer
const filtered = JSON.stringify(obj, ['name']);
// '{"name":"Alice"}'
```

## Array and Object Operations

### Array Methods
```javascript
// flat - flatten nested arrays
const nested = [1, [2, 3], [4, [5, 6]]];
const flat = nested.flat();      // [1, 2, 3, 4, [5, 6]]
const flatDeep = nested.flat(2); // [1, 2, 3, 4, 5, 6]

// flatMap - map then flatten
const numbers = [1, 2, 3];
const doubled = numbers.flatMap(n => [n, n * 2]);
// [1, 2, 2, 4, 3, 6]

// from - create array from iterable
const str = "hello";
const chars = Array.from(str);  // ['h', 'e', 'l', 'l', 'o']
```

### Object Methods
```javascript
const person = { name: "Alice", age: 25, city: "NYC" };

// Keys
const keys = Object.keys(person);
// ["name", "age", "city"]

// Values
const values = Object.values(person);
// ["Alice", 25, "NYC"]

// Entries
const entries = Object.entries(person);
// [["name", "Alice"], ["age", 25], ["city", "NYC"]]

// From entries
const obj = Object.fromEntries(entries);

// Assign (merge objects)
const merged = Object.assign({}, person, { country: "USA" });

// Freeze (make immutable)
const frozen = Object.freeze({ value: 42 });
// frozen.value = 10;  // Error in strict mode
```

## Best Practices (Do's)

✅ **Use async/await instead of promise chains**
```javascript
// Good
async function loadData() {
    const data = await fetchData();
    const processed = await processData(data);
    return processed;
}

// Less readable
function loadData() {
    return fetchData()
        .then(data => processData(data))
        .then(processed => processed);
}
```

✅ **Use array methods instead of loops**
```javascript
// Good
const doubled = numbers.map(n => n * 2);

// Less elegant
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
    doubled.push(numbers[i] * 2);
}
```

✅ **Use destructuring to extract properties**
```javascript
// Good
const { name, age } = person;

// Verbose
const name = person.name;
const age = person.age;
```

✅ **Use default parameters**
```javascript
// Good
function greet(name = "Guest") {
    return `Hello, ${name}`;
}

// Avoid
function greet(name) {
    name = name || "Guest";
    return `Hello, ${name}`;
}
```

## Common Mistakes (Don'ts)

❌ **Don't forget to handle promise rejections**
```javascript
// Bad
async function loadData() {
    const data = await fetchData();  // Unhandled error
}

// Good
async function loadData() {
    try {
        const data = await fetchData();
    } catch (error) {
        console.error(error);
    }
}
```

❌ **Don't mutate original arrays when you don't need to**
```javascript
// Bad
const numbers = [1, 2, 3];
numbers.push(4);  // Mutates original

// Good
const numbers = [1, 2, 3];
const extended = [...numbers, 4];
```

❌ **Don't mix callbacks and promises**
```javascript
// Confusing
function getData(callback) {
    return fetch('/api/data')
        .then(response => callback(response));
}

// Clear - use one pattern
async function getData() {
    return await fetch('/api/data');
}
```

❌ **Don't use forEach when you need the result**
```javascript
// Bad
let doubled = [];
numbers.forEach(n => doubled.push(n * 2));

// Good
const doubled = numbers.map(n => n * 2);
```
