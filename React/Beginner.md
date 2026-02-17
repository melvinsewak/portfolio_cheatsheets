# React Beginner Cheatsheet

## JSX Syntax and Basics

### What is JSX?
JSX is a syntax extension for JavaScript that allows you to write HTML-like code in your React components.

```jsx
// Basic JSX
const element = <h1>Hello, World!</h1>;

// JSX with JavaScript expressions (use curly braces)
const name = "Alice";
const greeting = <h1>Hello, {name}!</h1>;

// JSX with attributes
const image = <img src="photo.jpg" alt="Profile" />;

// JSX must have one parent element
const content = (
    <div>
        <h1>Title</h1>
        <p>Paragraph</p>
    </div>
);

// Using fragments to avoid extra DOM nodes
const contentWithFragment = (
    <>
        <h1>Title</h1>
        <p>Paragraph</p>
    </>
);
```

### JSX Rules
```jsx
// Class becomes className
<div className="container">Content</div>

// For becomes htmlFor
<label htmlFor="email">Email:</label>

// Style must be an object with camelCase properties
<div style={{ backgroundColor: 'blue', fontSize: '16px' }}>
    Styled content
</div>

// Self-closing tags must have closing slash
<img src="image.jpg" />
<input type="text" />
<br />

// JavaScript expressions in JSX
const result = <p>The sum is: {5 + 3}</p>;
const isLoggedIn = true;
const message = <p>{isLoggedIn ? 'Welcome back!' : 'Please log in'}</p>;
```

## Functional Components

### Basic Component
```jsx
// Simple function component
function Welcome() {
    return <h1>Hello, World!</h1>;
}

// Arrow function component
const WelcomeArrow = () => {
    return <h1>Hello, World!</h1>;
};

// Implicit return (one-liner)
const WelcomeShort = () => <h1>Hello, World!</h1>;

// Component with multiple elements
function Card() {
    return (
        <div className="card">
            <h2>Card Title</h2>
            <p>Card description goes here.</p>
            <button>Click Me</button>
        </div>
    );
}
```

### Exporting and Importing Components
```jsx
// Export default (App.jsx)
export default function App() {
    return <h1>My App</h1>;
}

// Named export (Button.jsx)
export function Button() {
    return <button>Click</button>;
}

// Multiple named exports
export function Button() { /* ... */ }
export function Input() { /* ... */ }

// Importing
import App from './App';  // Default import
import { Button } from './Button';  // Named import
import { Button, Input } from './Components';  // Multiple named imports
```

## Props (Passing and Receiving)

### Passing Props
```jsx
// Parent component passing props
function App() {
    return (
        <div>
            <Welcome name="Alice" age={25} />
            <Welcome name="Bob" age={30} />
        </div>
    );
}

// Passing different types of props
function Parent() {
    const user = { name: "Alice", age: 25 };
    
    return (
        <UserCard 
            name="Alice"              // String
            age={25}                  // Number
            isActive={true}           // Boolean
            hobbies={['reading', 'gaming']}  // Array
            user={user}               // Object
            handleClick={() => console.log('clicked')}  // Function
        />
    );
}
```

### Receiving Props
```jsx
// Using props object
function Welcome(props) {
    return <h1>Hello, {props.name}!</h1>;
}

// Destructuring props (preferred)
function Welcome({ name, age }) {
    return (
        <div>
            <h1>Hello, {name}!</h1>
            <p>Age: {age}</p>
        </div>
    );
}

// Default props
function Welcome({ name = "Guest", age = 0 }) {
    return <h1>Hello, {name}, age {age}!</h1>;
}

// Rest operator for additional props
function Button({ label, ...otherProps }) {
    return <button {...otherProps}>{label}</button>;
}

// Usage: <Button label="Click me" onClick={handleClick} disabled={false} />
```

### Children Prop
```jsx
// Children prop allows nesting content
function Card({ children }) {
    return (
        <div className="card">
            {children}
        </div>
    );
}

// Usage
function App() {
    return (
        <Card>
            <h2>Card Title</h2>
            <p>Card content</p>
        </Card>
    );
}
```

## Basic State with useState

### useState Basics
```jsx
import { useState } from 'react';

function Counter() {
    // Declare state variable
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

### Multiple State Variables
```jsx
function Form() {
    const [name, setName] = useState('');
    const [age, setAge] = useState(0);
    const [isSubscribed, setIsSubscribed] = useState(false);
    
    return (
        <form>
            <input 
                value={name} 
                onChange={(e) => setName(e.target.value)} 
            />
            <input 
                type="number"
                value={age} 
                onChange={(e) => setAge(Number(e.target.value))} 
            />
            <input 
                type="checkbox"
                checked={isSubscribed}
                onChange={(e) => setIsSubscribed(e.target.checked)} 
            />
        </form>
    );
}
```

### State with Objects and Arrays
```jsx
function UserProfile() {
    // State with object
    const [user, setUser] = useState({
        name: '',
        email: '',
        age: 0
    });
    
    // Updating object state (must spread existing values)
    const updateName = (newName) => {
        setUser({
            ...user,
            name: newName
        });
    };
    
    return (
        <input 
            value={user.name}
            onChange={(e) => updateName(e.target.value)}
        />
    );
}

function TodoList() {
    const [todos, setTodos] = useState([]);
    
    // Add to array
    const addTodo = (text) => {
        setTodos([...todos, { id: Date.now(), text }]);
    };
    
    // Remove from array
    const removeTodo = (id) => {
        setTodos(todos.filter(todo => todo.id !== id));
    };
    
    return (
        <div>
            {todos.map(todo => (
                <div key={todo.id}>
                    {todo.text}
                    <button onClick={() => removeTodo(todo.id)}>Delete</button>
                </div>
            ))}
        </div>
    );
}
```

### Functional Updates
```jsx
function Counter() {
    const [count, setCount] = useState(0);
    
    // Use functional update when new state depends on previous state
    const increment = () => {
        setCount(prevCount => prevCount + 1);
    };
    
    // Multiple updates in a row
    const incrementByThree = () => {
        setCount(prevCount => prevCount + 1);
        setCount(prevCount => prevCount + 1);
        setCount(prevCount => prevCount + 1);
    };
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>+1</button>
            <button onClick={incrementByThree}>+3</button>
        </div>
    );
}
```

## Event Handling

### Basic Event Handlers
```jsx
function EventExamples() {
    // Click event
    const handleClick = () => {
        console.log('Button clicked!');
    };
    
    // Click with event object
    const handleClickWithEvent = (event) => {
        console.log('Clicked element:', event.target);
    };
    
    // Input change
    const handleChange = (event) => {
        console.log('Input value:', event.target.value);
    };
    
    // Form submit
    const handleSubmit = (event) => {
        event.preventDefault();  // Prevent page reload
        console.log('Form submitted!');
    };
    
    return (
        <div>
            <button onClick={handleClick}>Click Me</button>
            <button onClick={handleClickWithEvent}>Click With Event</button>
            <input onChange={handleChange} />
            <form onSubmit={handleSubmit}>
                <button type="submit">Submit</button>
            </form>
        </div>
    );
}
```

### Passing Arguments to Event Handlers
```jsx
function ItemList() {
    const handleDelete = (id) => {
        console.log('Deleting item:', id);
    };
    
    const handleEdit = (id, name) => {
        console.log('Editing:', id, name);
    };
    
    return (
        <div>
            {/* Using arrow function */}
            <button onClick={() => handleDelete(123)}>
                Delete
            </button>
            
            {/* Multiple arguments */}
            <button onClick={() => handleEdit(123, 'Item Name')}>
                Edit
            </button>
        </div>
    );
}
```

### Common Events
```jsx
function EventTypes() {
    return (
        <div>
            {/* Mouse events */}
            <button onClick={() => console.log('click')}>Click</button>
            <div onMouseEnter={() => console.log('enter')}>Hover Me</div>
            <div onMouseLeave={() => console.log('leave')}>Leave Me</div>
            
            {/* Keyboard events */}
            <input onKeyDown={(e) => console.log('Key:', e.key)} />
            <input onKeyUp={(e) => console.log('Key up:', e.key)} />
            
            {/* Form events */}
            <input onChange={(e) => console.log(e.target.value)} />
            <form onSubmit={(e) => e.preventDefault()}>
                <button type="submit">Submit</button>
            </form>
            
            {/* Focus events */}
            <input onFocus={() => console.log('focused')} />
            <input onBlur={() => console.log('blurred')} />
        </div>
    );
}
```

## Conditional Rendering

### If-Else with Variables
```jsx
function Greeting({ isLoggedIn }) {
    let message;
    
    if (isLoggedIn) {
        message = <h1>Welcome back!</h1>;
    } else {
        message = <h1>Please log in.</h1>;
    }
    
    return <div>{message}</div>;
}
```

### Ternary Operator
```jsx
function Greeting({ isLoggedIn }) {
    return (
        <div>
            {isLoggedIn ? (
                <h1>Welcome back!</h1>
            ) : (
                <h1>Please log in.</h1>
            )}
        </div>
    );
}

// Inline ternary
function Status({ isActive }) {
    return <span>{isActive ? 'Active' : 'Inactive'}</span>;
}
```

### Logical && Operator
```jsx
function Mailbox({ unreadMessages }) {
    return (
        <div>
            <h1>Hello!</h1>
            {unreadMessages.length > 0 && (
                <h2>You have {unreadMessages.length} unread messages.</h2>
            )}
        </div>
    );
}

// Conditional rendering with multiple conditions
function Dashboard({ user }) {
    return (
        <div>
            {user && <h1>Welcome, {user.name}!</h1>}
            {user && user.isAdmin && <button>Admin Panel</button>}
        </div>
    );
}
```

### Conditional Rendering Patterns
```jsx
// Early return
function UserProfile({ user }) {
    if (!user) {
        return <div>Loading...</div>;
    }
    
    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
        </div>
    );
}

// Switch case pattern
function StatusBadge({ status }) {
    const renderBadge = () => {
        switch(status) {
            case 'success':
                return <span className="badge-success">Success</span>;
            case 'error':
                return <span className="badge-error">Error</span>;
            case 'warning':
                return <span className="badge-warning">Warning</span>;
            default:
                return <span className="badge-default">Unknown</span>;
        }
    };
    
    return <div>{renderBadge()}</div>;
}

// Object mapping
function Icon({ type }) {
    const icons = {
        success: '✓',
        error: '✗',
        warning: '⚠',
        info: 'ℹ'
    };
    
    return <span>{icons[type] || '?'}</span>;
}
```

## Lists and Keys

### Rendering Lists
```jsx
function NameList() {
    const names = ['Alice', 'Bob', 'Charlie'];
    
    return (
        <ul>
            {names.map((name, index) => (
                <li key={index}>{name}</li>
            ))}
        </ul>
    );
}
```

### Keys with Unique IDs
```jsx
function TodoList() {
    const todos = [
        { id: 1, text: 'Learn React' },
        { id: 2, text: 'Build a project' },
        { id: 3, text: 'Deploy app' }
    ];
    
    return (
        <ul>
            {todos.map(todo => (
                <li key={todo.id}>{todo.text}</li>
            ))}
        </ul>
    );
}
```

### Lists with Components
```jsx
function UserCard({ user }) {
    return (
        <div className="user-card">
            <h3>{user.name}</h3>
            <p>{user.email}</p>
        </div>
    );
}

function UserList() {
    const users = [
        { id: 1, name: 'Alice', email: 'alice@example.com' },
        { id: 2, name: 'Bob', email: 'bob@example.com' },
        { id: 3, name: 'Charlie', email: 'charlie@example.com' }
    ];
    
    return (
        <div>
            {users.map(user => (
                <UserCard key={user.id} user={user} />
            ))}
        </div>
    );
}
```

### Filtering and Transforming Lists
```jsx
function ProductList({ products }) {
    // Filter items
    const inStockProducts = products.filter(product => product.inStock);
    
    return (
        <div>
            {inStockProducts.map(product => (
                <div key={product.id}>
                    <h3>{product.name}</h3>
                    <p>${product.price}</p>
                </div>
            ))}
        </div>
    );
}

// Sorting lists
function SortedList({ items }) {
    const sortedItems = [...items].sort((a, b) => a.name.localeCompare(b.name));
    
    return (
        <ul>
            {sortedItems.map(item => (
                <li key={item.id}>{item.name}</li>
            ))}
        </ul>
    );
}
```

## Basic Styling

### Inline Styles
```jsx
function StyledComponent() {
    const divStyle = {
        backgroundColor: 'lightblue',
        padding: '20px',
        borderRadius: '8px',
        fontSize: '16px'
    };
    
    return (
        <div style={divStyle}>
            <h1 style={{ color: 'navy', marginBottom: '10px' }}>
                Title
            </h1>
            <p>Content with inline styles</p>
        </div>
    );
}

// Dynamic inline styles
function Alert({ type }) {
    const alertStyle = {
        padding: '15px',
        borderRadius: '4px',
        backgroundColor: type === 'error' ? '#ffebee' : '#e8f5e9',
        color: type === 'error' ? '#c62828' : '#2e7d32'
    };
    
    return (
        <div style={alertStyle}>
            Alert message
        </div>
    );
}
```

### CSS Classes
```jsx
// Regular CSS file (App.css)
/*
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.button:hover {
    background-color: #0056b3;
}
*/

// Component
import './App.css';

function App() {
    return (
        <div className="container">
            <button className="button">Click Me</button>
        </div>
    );
}
```

### CSS Modules
```jsx
// Button.module.css
/*
.button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
}

.primary {
    background-color: #007bff;
}

.secondary {
    background-color: #6c757d;
}
*/

// Button.jsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children }) {
    return (
        <button className={`${styles.button} ${styles[variant]}`}>
            {children}
        </button>
    );
}

// Usage
<Button variant="primary">Primary Button</Button>
<Button variant="secondary">Secondary Button</Button>
```

### Conditional Classes
```jsx
function Alert({ type, message }) {
    // Using template literals
    const className = `alert alert-${type}`;
    
    return <div className={className}>{message}</div>;
}

// Conditional class names
function Card({ isActive }) {
    const cardClass = isActive ? 'card card-active' : 'card';
    
    return <div className={cardClass}>Card Content</div>;
}

// Multiple conditional classes
function Button({ isPrimary, isLarge, isDisabled }) {
    const classes = [
        'button',
        isPrimary && 'button-primary',
        isLarge && 'button-large',
        isDisabled && 'button-disabled'
    ].filter(Boolean).join(' ');
    
    return <button className={classes}>Button</button>;
}
```

## Best Practices (Do's)

✅ **Always use keys when rendering lists**
```jsx
// Good
{items.map(item => <li key={item.id}>{item.name}</li>)}

// Use index only as last resort
{items.map((item, index) => <li key={index}>{item.name}</li>)}
```

✅ **Keep components small and focused**
```jsx
// Good - Single responsibility
function UserCard({ user }) {
    return (
        <div className="card">
            <UserAvatar url={user.avatar} />
            <UserInfo name={user.name} email={user.email} />
        </div>
    );
}

// Not ideal - Too much in one component
function Dashboard() {
    // 200+ lines of code mixing concerns
}
```

✅ **Use destructuring for props**
```jsx
// Good
function Welcome({ name, age }) {
    return <h1>Hello, {name}!</h1>;
}

// Less readable
function Welcome(props) {
    return <h1>Hello, {props.name}!</h1>;
}
```

✅ **Use functional updates for state that depends on previous state**
```jsx
// Good
setCount(prevCount => prevCount + 1);

// Can cause issues
setCount(count + 1);
```

✅ **Name event handlers with "handle" prefix**
```jsx
// Good
const handleClick = () => { /* ... */ };
const handleSubmit = () => { /* ... */ };
const handleChange = () => { /* ... */ };

// Less clear
const click = () => { /* ... */ };
const doSomething = () => { /* ... */ };
```

✅ **Use meaningful component and variable names**
```jsx
// Good
function UserProfile() { /* ... */ }
const [isLoading, setIsLoading] = useState(false);

// Bad
function Component1() { /* ... */ }
const [flag, setFlag] = useState(false);
```

## Common Mistakes (Don'ts)

❌ **Don't modify state directly**
```jsx
// Wrong
const [user, setUser] = useState({ name: 'Alice' });
user.name = 'Bob';  // This won't trigger a re-render!

// Correct
setUser({ ...user, name: 'Bob' });
```

❌ **Don't use array index as key when list can change**
```jsx
// Wrong - Can cause bugs when items are added/removed/reordered
{items.map((item, index) => <li key={index}>{item}</li>)}

// Correct - Use stable unique identifier
{items.map(item => <li key={item.id}>{item}</li>)}
```

❌ **Don't call hooks conditionally**
```jsx
// Wrong
if (condition) {
    const [count, setCount] = useState(0);
}

// Correct - Hooks must be at top level
const [count, setCount] = useState(0);
if (condition) {
    // Use the state here
}
```

❌ **Don't forget to prevent default on form submission**
```jsx
// Wrong - Page will reload
const handleSubmit = () => {
    console.log('Submitting...');
};

// Correct
const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Submitting...');
};
```

❌ **Don't forget parentheses in onClick**
```jsx
// Wrong - Function executes immediately
<button onClick={handleClick()}>Click</button>

// Correct - Pass function reference
<button onClick={handleClick}>Click</button>

// Correct - Use arrow function if you need to pass arguments
<button onClick={() => handleClick(id)}>Click</button>
```

❌ **Don't mutate arrays or objects in state**
```jsx
// Wrong
const [todos, setTodos] = useState([]);
todos.push(newTodo);  // Mutates array
setTodos(todos);

// Correct
setTodos([...todos, newTodo]);

// Wrong
const [user, setUser] = useState({ name: 'Alice', age: 25 });
user.age = 26;  // Mutates object
setUser(user);

// Correct
setUser({ ...user, age: 26 });
```

❌ **Don't use setState in render**
```jsx
// Wrong - Causes infinite loop
function Component() {
    const [count, setCount] = useState(0);
    setCount(count + 1);  // Called on every render!
    
    return <div>{count}</div>;
}

// Correct - Use in event handler
function Component() {
    const [count, setCount] = useState(0);
    
    const handleClick = () => {
        setCount(count + 1);
    };
    
    return (
        <div>
            {count}
            <button onClick={handleClick}>Increment</button>
        </div>
    );
}
```

❌ **Don't forget JSX must return a single parent element**
```jsx
// Wrong
function Component() {
    return (
        <h1>Title</h1>
        <p>Paragraph</p>
    );
}

// Correct - Wrap in div
function Component() {
    return (
        <div>
            <h1>Title</h1>
            <p>Paragraph</p>
        </div>
    );
}

// Correct - Use fragment
function Component() {
    return (
        <>
            <h1>Title</h1>
            <p>Paragraph</p>
        </>
    );
}
```
