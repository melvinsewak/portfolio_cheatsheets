# JavaScript Beginner Cheatsheet

## Variables and Data Types

### Variable Declarations
```javascript
// let - block-scoped, can be reassigned
let name = "Alice";
name = "Bob";  // OK

// const - block-scoped, cannot be reassigned
const age = 25;
// age = 26;  // Error!

// var - function-scoped (avoid using)
var oldWay = "deprecated";
```

### Primitive Types
```javascript
// String
let text = "Hello World";
let template = `Hello, ${name}!`;  // Template literal

// Number (both integers and floats)
let integer = 42;
let float = 3.14;
let negative = -10;

// Boolean
let isTrue = true;
let isFalse = false;

// Undefined (declared but not assigned)
let notAssigned;
console.log(notAssigned);  // undefined

// Null (intentionally empty)
let empty = null;

// Symbol (unique identifier)
let sym = Symbol("unique");

// BigInt (large integers)
let big = 123456789012345678901234567890n;
```

### Type Checking
```javascript
typeof "Hello";        // "string"
typeof 42;             // "number"
typeof true;           // "boolean"
typeof undefined;      // "undefined"
typeof null;           // "object" (JavaScript quirk)
typeof Symbol();       // "symbol"
typeof 123n;           // "bigint"
```

## Operators

### Arithmetic Operators
```javascript
let a = 10, b = 3;

console.log(a + b);   // 13 - Addition
console.log(a - b);   // 7  - Subtraction
console.log(a * b);   // 30 - Multiplication
console.log(a / b);   // 3.333... - Division
console.log(a % b);   // 1  - Modulus (remainder)
console.log(a ** b);  // 1000 - Exponentiation

// Increment/Decrement
let x = 5;
x++;  // 6
x--;  // 5
```

### Comparison Operators
```javascript
// Equality
console.log(5 == "5");   // true (loose equality)
console.log(5 === "5");  // false (strict equality)
console.log(5 != "5");   // false
console.log(5 !== "5");  // true

// Comparison
console.log(10 > 5);     // true
console.log(10 < 5);     // false
console.log(10 >= 10);   // true
console.log(10 <= 5);    // false
```

### Logical Operators
```javascript
let a = true, b = false;

console.log(a && b);  // false - AND
console.log(a || b);  // true  - OR
console.log(!a);      // false - NOT

// Short-circuit evaluation
let result = value || "default";  // Use default if value is falsy
let safe = obj && obj.property;   // Access property if obj exists
```

## Control Flow

### If-Else Statements
```javascript
let age = 20;

if (age < 18) {
    console.log("Minor");
} else if (age < 65) {
    console.log("Adult");
} else {
    console.log("Senior");
}

// Ternary operator
let status = age >= 18 ? "Adult" : "Minor";
```

### Switch Statement
```javascript
let day = "Monday";

switch (day) {
    case "Monday":
        console.log("Start of the week");
        break;
    case "Friday":
        console.log("End of the week");
        break;
    case "Saturday":
    case "Sunday":
        console.log("Weekend!");
        break;
    default:
        console.log("Midweek");
}
```

## Loops

### For Loop
```javascript
for (let i = 0; i < 5; i++) {
    console.log(i);  // 0, 1, 2, 3, 4
}

// Loop through array
let fruits = ["Apple", "Banana", "Orange"];
for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i]);
}
```

### While Loop
```javascript
let count = 0;
while (count < 5) {
    console.log(count);
    count++;
}

// Do-while (executes at least once)
let num = 0;
do {
    console.log(num);
    num++;
} while (num < 5);
```

### For...of Loop (arrays)
```javascript
let numbers = [1, 2, 3, 4, 5];

for (let num of numbers) {
    console.log(num);
}
```

### For...in Loop (objects)
```javascript
let person = { name: "Alice", age: 25 };

for (let key in person) {
    console.log(`${key}: ${person[key]}`);
}
```

## Functions

### Function Declaration
```javascript
function greet(name) {
    return `Hello, ${name}!`;
}

console.log(greet("Alice"));
```

### Function Expression
```javascript
const greet = function(name) {
    return `Hello, ${name}!`;
};
```

### Arrow Functions
```javascript
// Single parameter, single expression
const square = x => x * x;

// Multiple parameters
const add = (a, b) => a + b;

// Multiple statements
const multiply = (a, b) => {
    let result = a * b;
    return result;
};

// No parameters
const sayHello = () => console.log("Hello!");
```

### Default Parameters
```javascript
function greet(name = "Guest") {
    return `Hello, ${name}!`;
}

console.log(greet());        // "Hello, Guest!"
console.log(greet("Alice")); // "Hello, Alice!"
```

### Rest Parameters
```javascript
function sum(...numbers) {
    return numbers.reduce((acc, num) => acc + num, 0);
}

console.log(sum(1, 2, 3, 4));  // 10
```

## Arrays

### Creating Arrays
```javascript
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "two", true, null];
let empty = [];
let fromConstructor = new Array(5);  // [empty × 5]
```

### Accessing Elements
```javascript
let fruits = ["Apple", "Banana", "Orange"];

console.log(fruits[0]);     // "Apple"
console.log(fruits[2]);     // "Orange"
console.log(fruits.length); // 3
```

### Modifying Arrays
```javascript
let fruits = ["Apple", "Banana"];

// Add to end
fruits.push("Orange");      // ["Apple", "Banana", "Orange"]

// Remove from end
let last = fruits.pop();    // "Orange"

// Add to beginning
fruits.unshift("Mango");    // ["Mango", "Apple", "Banana"]

// Remove from beginning
let first = fruits.shift(); // "Mango"

// Change element
fruits[0] = "Grape";
```

### Array Methods
```javascript
let numbers = [1, 2, 3, 4, 5];

// Find index
let index = numbers.indexOf(3);  // 2

// Check if includes
let hasThree = numbers.includes(3);  // true

// Join to string
let str = numbers.join("-");  // "1-2-3-4-5"

// Slice (copy portion)
let slice = numbers.slice(1, 3);  // [2, 3]

// Splice (remove/add elements)
numbers.splice(2, 1);  // Remove 1 element at index 2
```

## Objects

### Creating Objects
```javascript
// Object literal
let person = {
    name: "Alice",
    age: 25,
    city: "New York"
};

// Empty object
let empty = {};

// Using new Object()
let obj = new Object();
```

### Accessing Properties
```javascript
let person = { name: "Alice", age: 25 };

// Dot notation
console.log(person.name);  // "Alice"

// Bracket notation
console.log(person["age"]);  // 25

// Useful for dynamic keys
let key = "name";
console.log(person[key]);  // "Alice"
```

### Modifying Objects
```javascript
let person = { name: "Alice" };

// Add property
person.age = 25;

// Modify property
person.name = "Bob";

// Delete property
delete person.age;
```

### Object Methods
```javascript
let person = {
    name: "Alice",
    age: 25,
    greet: function() {
        console.log(`Hi, I'm ${this.name}`);
    },
    // Shorthand method syntax
    introduce() {
        console.log(`I'm ${this.age} years old`);
    }
};

person.greet();      // "Hi, I'm Alice"
person.introduce();  // "I'm 25 years old"
```

## Strings

### String Methods
```javascript
let text = "Hello World";

// Length
console.log(text.length);  // 11

// Convert case
console.log(text.toLowerCase());  // "hello world"
console.log(text.toUpperCase());  // "HELLO WORLD"

// Substring
console.log(text.substring(0, 5));  // "Hello"
console.log(text.slice(6));         // "World"

// Replace
console.log(text.replace("World", "JavaScript"));  // "Hello JavaScript"

// Split
let words = text.split(" ");  // ["Hello", "World"]

// Trim whitespace
let padded = "  hello  ";
console.log(padded.trim());  // "hello"

// Check content
console.log(text.includes("World"));  // true
console.log(text.startsWith("Hello")); // true
console.log(text.endsWith("World"));   // true
```

### Template Literals
```javascript
let name = "Alice";
let age = 25;

// Multi-line strings
let message = `Hello,
my name is ${name}
and I'm ${age} years old.`;

// Expression interpolation
let total = `Total: ${10 + 20}`;  // "Total: 30"
```

## Best Practices (Do's)

✅ **Use const by default, let when reassignment is needed**
```javascript
// Good
const PI = 3.14159;
let count = 0;

// Avoid
var x = 10;
```

✅ **Use strict equality (===) instead of loose equality (==)**
```javascript
// Good
if (value === 5) { }

// Avoid
if (value == 5) { }
```

✅ **Use template literals for string concatenation**
```javascript
// Good
let message = `Hello, ${name}!`;

// Less readable
let message = "Hello, " + name + "!";
```

✅ **Use descriptive variable names**
```javascript
// Good
let userAge = 25;
let isAuthenticated = true;

// Bad
let a = 25;
let flag = true;
```

✅ **Always use semicolons**
```javascript
// Good
let x = 5;
console.log(x);

// Risky (ASI can cause issues)
let x = 5
console.log(x)
```

## Common Mistakes (Don'ts)

❌ **Don't use var (use let or const)**
```javascript
// Bad
var name = "Alice";

// Good
const name = "Alice";
```

❌ **Don't forget to declare variables**
```javascript
// Bad (creates global variable)
name = "Alice";

// Good
let name = "Alice";
```

❌ **Don't compare with == when you mean ===**
```javascript
// Confusing
console.log(0 == false);   // true
console.log("" == false);  // true

// Clear
console.log(0 === false);  // false
```

❌ **Don't modify function parameters with default values**
```javascript
// Problematic
function greet(name = "Guest") {
    name = name.toUpperCase();  // Modifies parameter
    return `Hello, ${name}!`;
}

// Better
function greet(name = "Guest") {
    const upperName = name.toUpperCase();
    return `Hello, ${upperName}!`;
}
```

❌ **Don't forget to return in functions when needed**
```javascript
// Bug - no return
function add(a, b) {
    a + b;  // Result is lost!
}

// Correct
function add(a, b) {
    return a + b;
}
```
