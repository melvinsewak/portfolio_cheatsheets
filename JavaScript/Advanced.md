# JavaScript Advanced Cheatsheet

## Closures

### Understanding Closures
```javascript
function outer() {
    let count = 0;
    
    return function inner() {
        count++;
        console.log(count);
    };
}

const counter = outer();
counter();  // 1
counter();  // 2
counter();  // 3
```

### Practical Closures
```javascript
// Private variables
function createBankAccount(initialBalance) {
    let balance = initialBalance;
    
    return {
        deposit(amount) {
            balance += amount;
            return balance;
        },
        withdraw(amount) {
            if (amount > balance) {
                throw new Error("Insufficient funds");
            }
            balance -= amount;
            return balance;
        },
        getBalance() {
            return balance;
        }
    };
}

const account = createBankAccount(100);
account.deposit(50);
console.log(account.getBalance());  // 150
```

### Module Pattern
```javascript
const Calculator = (function() {
    // Private variables and functions
    let result = 0;
    
    function log(operation, value) {
        console.log(`${operation}: ${value}`);
    }
    
    // Public API
    return {
        add(value) {
            result += value;
            log('Added', value);
            return this;
        },
        subtract(value) {
            result -= value;
            log('Subtracted', value);
            return this;
        },
        getResult() {
            return result;
        },
        reset() {
            result = 0;
            return this;
        }
    };
})();

Calculator.add(10).subtract(3).add(5);
console.log(Calculator.getResult());  // 12
```

## Prototypes and Inheritance

### Prototype Chain
```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
}

// Add method to prototype
Person.prototype.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
};

const person = new Person("Alice", 25);
person.greet();

// Check prototype
console.log(person.__proto__ === Person.prototype);  // true
```

### Prototypal Inheritance
```javascript
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
    Animal.call(this, name);  // Call parent constructor
    this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    console.log(`${this.name} barks!`);
};

const dog = new Dog("Buddy", "Golden Retriever");
dog.speak();  // Inherited from Animal
dog.bark();   // Own method
```

### Object.create
```javascript
const personPrototype = {
    greet() {
        console.log(`Hi, I'm ${this.name}`);
    }
};

const person = Object.create(personPrototype);
person.name = "Alice";
person.greet();
```

## Advanced Functions

### Higher-Order Functions
```javascript
// Function that returns a function
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// Function that takes a function
function applyOperation(arr, operation) {
    return arr.map(operation);
}

const numbers = [1, 2, 3, 4];
const squared = applyOperation(numbers, x => x * x);
```

### Function Composition
```javascript
const compose = (...fns) => x =>
    fns.reduceRight((acc, fn) => fn(acc), x);

const add5 = x => x + 5;
const multiply2 = x => x * 2;
const subtract3 = x => x - 3;

const combined = compose(subtract3, multiply2, add5);
console.log(combined(10));  // (10 + 5) * 2 - 3 = 27
```

### Currying
```javascript
// Regular function
function add(a, b, c) {
    return a + b + c;
}

// Curried version
function curriedAdd(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

const add5 = curriedAdd(5);
const add5and10 = add5(10);
console.log(add5and10(15));  // 30

// With arrow functions
const curriedMultiply = a => b => c => a * b * c;
```

### Memoization
```javascript
function memoize(fn) {
    const cache = {};
    
    return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) {
            console.log('From cache');
            return cache[key];
        }
        
        console.log('Calculating...');
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}

const fibonacci = memoize(function(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10));  // Calculates once
console.log(fibonacci(10));  // From cache
```

## This Keyword

### Understanding 'this'
```javascript
// Global context
console.log(this);  // window (browser) or global (Node.js)

// Object method
const person = {
    name: "Alice",
    greet() {
        console.log(this.name);  // "Alice"
    }
};

// Arrow functions don't have their own 'this'
const person2 = {
    name: "Bob",
    greet: () => {
        console.log(this.name);  // undefined or global
    }
};
```

### Binding 'this'
```javascript
function greet() {
    console.log(`Hello, ${this.name}`);
}

const person = { name: "Alice" };

// call - invoke immediately
greet.call(person);  // "Hello, Alice"

// apply - invoke with array of arguments
greet.apply(person, []);

// bind - create new function with bound 'this'
const boundGreet = greet.bind(person);
boundGreet();  // "Hello, Alice"
```

## Symbols

### Creating Symbols
```javascript
// Unique identifiers
const sym1 = Symbol();
const sym2 = Symbol();
console.log(sym1 === sym2);  // false

// With description
const id = Symbol('id');
console.log(id.description);  // "id"

// Well-known symbols
const obj = {
    [Symbol.iterator]: function*() {
        yield 1;
        yield 2;
        yield 3;
    }
};

for (let value of obj) {
    console.log(value);
}
```

## Generators and Iterators

### Generators
```javascript
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
}

const gen = numberGenerator();
console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 3, done: false }
console.log(gen.next());  // { value: undefined, done: true }

// Infinite generator
function* infiniteSequence() {
    let i = 0;
    while (true) {
        yield i++;
    }
}

// Generator delegation
function* generator1() {
    yield 1;
    yield 2;
}

function* generator2() {
    yield* generator1();
    yield 3;
    yield 4;
}
```

### Custom Iterators
```javascript
const range = {
    from: 1,
    to: 5,
    
    [Symbol.iterator]() {
        return {
            current: this.from,
            last: this.to,
            
            next() {
                if (this.current <= this.last) {
                    return { value: this.current++, done: false };
                }
                return { done: true };
            }
        };
    }
};

for (let num of range) {
    console.log(num);  // 1, 2, 3, 4, 5
}
```

## Proxy and Reflect

### Proxy
```javascript
const target = {
    name: "Alice",
    age: 25
};

const handler = {
    get(target, prop) {
        console.log(`Getting ${prop}`);
        return target[prop];
    },
    set(target, prop, value) {
        console.log(`Setting ${prop} to ${value}`);
        if (prop === 'age' && typeof value !== 'number') {
            throw new TypeError('Age must be a number');
        }
        target[prop] = value;
        return true;
    }
};

const proxy = new Proxy(target, handler);
console.log(proxy.name);  // Logs "Getting name", returns "Alice"
proxy.age = 26;           // Logs "Setting age to 26"
```

### Validation with Proxy
```javascript
function createValidator(target, validations) {
    return new Proxy(target, {
        set(target, prop, value) {
            if (validations[prop]) {
                if (!validations[prop](value)) {
                    throw new Error(`Invalid value for ${prop}`);
                }
            }
            target[prop] = value;
            return true;
        }
    });
}

const user = createValidator({}, {
    age: value => typeof value === 'number' && value > 0,
    email: value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
});

user.age = 25;           // OK
// user.age = -5;        // Error
user.email = "a@b.com";  // OK
// user.email = "invalid"; // Error
```

## Design Patterns

### Singleton
```javascript
const Singleton = (function() {
    let instance;
    
    function createInstance() {
        return {
            config: {},
            method() {
                console.log("Singleton method");
            }
        };
    }
    
    return {
        getInstance() {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();

const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2);  // true
```

### Observer Pattern
```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    }
    
    off(event, listenerToRemove) {
        if (!this.events[event]) return;
        
        this.events[event] = this.events[event].filter(
            listener => listener !== listenerToRemove
        );
    }
    
    emit(event, ...args) {
        if (!this.events[event]) return;
        
        this.events[event].forEach(listener => {
            listener(...args);
        });
    }
}

const emitter = new EventEmitter();
emitter.on('data', data => console.log(`Received: ${data}`));
emitter.emit('data', 'Hello World');
```

### Factory Pattern
```javascript
class Car {
    constructor(type) {
        this.type = type;
    }
}

class CarFactory {
    createCar(type) {
        switch(type) {
            case 'sedan':
                return new Car('Sedan');
            case 'suv':
                return new Car('SUV');
            default:
                throw new Error('Unknown car type');
        }
    }
}

const factory = new CarFactory();
const sedan = factory.createCar('sedan');
```

## Performance Optimization

### Debouncing
```javascript
function debounce(func, delay) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            func.apply(this, args);
        }, delay);
    };
}

// Usage
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce(query => {
    console.log(`Searching for: ${query}`);
}, 300);

searchInput.addEventListener('input', e => {
    debouncedSearch(e.target.value);
});
```

### Throttling
```javascript
function throttle(func, limit) {
    let inThrottle;
    
    return function(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            
            setTimeout(() => {
                inThrottle = false;
            }, limit);
        }
    };
}

// Usage
const throttledScroll = throttle(() => {
    console.log('Scroll event');
}, 1000);

window.addEventListener('scroll', throttledScroll);
```

## WeakMap and WeakSet

### WeakMap
```javascript
// Keys must be objects, garbage collected when no other references
const weakMap = new WeakMap();

let obj = { name: "Alice" };
weakMap.set(obj, "some value");

console.log(weakMap.get(obj));  // "some value"

// When obj is garbage collected, entry is automatically removed
obj = null;
```

### WeakSet
```javascript
const weakSet = new WeakSet();

let obj1 = { id: 1 };
let obj2 = { id: 2 };

weakSet.add(obj1);
weakSet.add(obj2);

console.log(weakSet.has(obj1));  // true

obj1 = null;  // Garbage collected
```

## Best Practices (Do's)

✅ **Use closures for data privacy**
```javascript
// Good
function createCounter() {
    let count = 0;
    return {
        increment: () => ++count,
        getCount: () => count
    };
}
```

✅ **Use generators for lazy evaluation**
```javascript
// Good - memory efficient
function* largeDataSet() {
    for (let i = 0; i < 1000000; i++) {
        yield processData(i);
    }
}
```

✅ **Use proxy for validation and logging**
```javascript
// Good
const validatedUser = new Proxy(user, {
    set(target, prop, value) {
        if (prop === 'age' && value < 0) {
            throw new Error('Invalid age');
        }
        target[prop] = value;
        return true;
    }
});
```

✅ **Debounce expensive operations**
```javascript
// Good
const debouncedSave = debounce(saveToDatabase, 500);
```

## Common Mistakes (Don'ts)

❌ **Don't create closures in loops without proper scoping**
```javascript
// Bad
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);  // Prints 3, 3, 3
}

// Good
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);  // Prints 0, 1, 2
}
```

❌ **Don't forget 'new' with constructor functions**
```javascript
// Bad
const person = Person("Alice");  // Returns undefined

// Good
const person = new Person("Alice");
```

❌ **Don't modify prototypes of built-in objects**
```javascript
// Bad
Array.prototype.myMethod = function() { };

// Good - use utility functions instead
function myArrayMethod(arr) { }
```

❌ **Don't use synchronous operations for heavy tasks**
```javascript
// Bad - blocks event loop
const data = fs.readFileSync('large-file.txt');

// Good - non-blocking
const data = await fs.promises.readFile('large-file.txt');
```
