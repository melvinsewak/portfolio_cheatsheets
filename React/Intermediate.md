# React Intermediate Cheatsheet

## useEffect Hook

### Basic useEffect
```jsx
import { useState, useEffect } from 'react';

function Component() {
    const [count, setCount] = useState(0);
    
    // Runs after every render
    useEffect(() => {
        console.log('Component rendered or updated');
    });
    
    return <div>Count: {count}</div>;
}
```

### useEffect with Dependency Array
```jsx
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    
    // Runs only when userId changes
    useEffect(() => {
        fetch(`/api/users/${userId}`)
            .then(response => response.json())
            .then(data => setUser(data));
    }, [userId]);
    
    return <div>{user?.name}</div>;
}
```

### useEffect with Cleanup
```jsx
function Timer() {
    const [seconds, setSeconds] = useState(0);
    
    useEffect(() => {
        const interval = setInterval(() => {
            setSeconds(prev => prev + 1);
        }, 1000);
        
        // Cleanup function
        return () => {
            clearInterval(interval);
        };
    }, []);
    
    return <div>Seconds: {seconds}</div>;
}

// Event listener cleanup
function WindowSize() {
    const [size, setSize] = useState({ width: 0, height: 0 });
    
    useEffect(() => {
        const handleResize = () => {
            setSize({
                width: window.innerWidth,
                height: window.innerHeight
            });
        };
        
        window.addEventListener('resize', handleResize);
        handleResize(); // Initial call
        
        // Cleanup
        return () => {
            window.removeEventListener('resize', handleResize);
        };
    }, []);
    
    return <div>Width: {size.width}, Height: {size.height}</div>;
}
```

### Common useEffect Patterns
```jsx
// Run once on mount (empty dependency array)
useEffect(() => {
    console.log('Component mounted');
}, []);

// Run on every render (no dependency array)
useEffect(() => {
    console.log('Component rendered');
});

// Run when specific values change
useEffect(() => {
    console.log('Count changed:', count);
}, [count]);

// Multiple dependencies
useEffect(() => {
    console.log('User or theme changed');
}, [user, theme]);

// Async operations in useEffect
useEffect(() => {
    let cancelled = false;
    
    async function fetchData() {
        try {
            const response = await fetch('/api/data');
            const data = await response.json();
            
            if (!cancelled) {
                setData(data);
            }
        } catch (error) {
            if (!cancelled) {
                setError(error);
            }
        }
    }
    
    fetchData();
    
    return () => {
        cancelled = true;
    };
}, []);
```

## useRef Hook

### Basic useRef
```jsx
import { useRef } from 'react';

function TextInput() {
    const inputRef = useRef(null);
    
    const focusInput = () => {
        inputRef.current.focus();
    };
    
    return (
        <div>
            <input ref={inputRef} type="text" />
            <button onClick={focusInput}>Focus Input</button>
        </div>
    );
}
```

### useRef for Storing Values
```jsx
function Component() {
    const renderCount = useRef(0);
    const previousValue = useRef(null);
    
    useEffect(() => {
        renderCount.current += 1;
        console.log('Render count:', renderCount.current);
    });
    
    const [value, setValue] = useState('');
    
    useEffect(() => {
        previousValue.current = value;
    }, [value]);
    
    return (
        <div>
            <input 
                value={value} 
                onChange={(e) => setValue(e.target.value)} 
            />
            <p>Current: {value}</p>
            <p>Previous: {previousValue.current}</p>
        </div>
    );
}
```

### useRef vs useState
```jsx
function Comparison() {
    // useState - triggers re-render
    const [stateCount, setStateCount] = useState(0);
    
    // useRef - does NOT trigger re-render
    const refCount = useRef(0);
    
    const incrementState = () => {
        setStateCount(stateCount + 1); // Component re-renders
    };
    
    const incrementRef = () => {
        refCount.current += 1; // No re-render
        console.log('Ref count:', refCount.current);
    };
    
    return (
        <div>
            <p>State count: {stateCount}</p>
            <button onClick={incrementState}>Increment State</button>
            <button onClick={incrementRef}>Increment Ref (check console)</button>
        </div>
    );
}
```

## useCallback Hook

### Basic useCallback
```jsx
import { useState, useCallback } from 'react';

function ParentComponent() {
    const [count, setCount] = useState(0);
    const [other, setOther] = useState(0);
    
    // Without useCallback - new function on every render
    const handleClick = () => {
        console.log('Clicked');
    };
    
    // With useCallback - same function reference unless dependencies change
    const handleClickMemo = useCallback(() => {
        console.log('Clicked');
    }, []);
    
    return (
        <div>
            <ChildComponent onClick={handleClickMemo} />
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
        </div>
    );
}
```

### useCallback with Dependencies
```jsx
function SearchComponent() {
    const [query, setQuery] = useState('');
    const [filter, setFilter] = useState('all');
    
    // Recreated only when query or filter changes
    const handleSearch = useCallback(() => {
        console.log('Searching for:', query, 'with filter:', filter);
        // Perform search...
    }, [query, filter]);
    
    return (
        <div>
            <input 
                value={query} 
                onChange={(e) => setQuery(e.target.value)} 
            />
            <SearchButton onClick={handleSearch} />
        </div>
    );
}
```

### useCallback with Async Operations
```jsx
function DataFetcher({ userId }) {
    const [data, setData] = useState(null);
    
    const fetchData = useCallback(async () => {
        try {
            const response = await fetch(`/api/users/${userId}`);
            const result = await response.json();
            setData(result);
        } catch (error) {
            console.error('Error fetching data:', error);
        }
    }, [userId]);
    
    useEffect(() => {
        fetchData();
    }, [fetchData]);
    
    return <div>{data?.name}</div>;
}
```

## useMemo Hook

### Basic useMemo
```jsx
import { useState, useMemo } from 'react';

function ExpensiveComponent({ numbers }) {
    const [count, setCount] = useState(0);
    
    // Expensive calculation without useMemo - runs on every render
    const sum = numbers.reduce((acc, num) => acc + num, 0);
    
    // With useMemo - only recalculates when numbers changes
    const memoizedSum = useMemo(() => {
        console.log('Calculating sum...');
        return numbers.reduce((acc, num) => acc + num, 0);
    }, [numbers]);
    
    return (
        <div>
            <p>Sum: {memoizedSum}</p>
            <button onClick={() => setCount(count + 1)}>
                Count: {count}
            </button>
        </div>
    );
}
```

### useMemo for Filtering and Sorting
```jsx
function UserList({ users, searchQuery }) {
    // Only recompute when users or searchQuery changes
    const filteredUsers = useMemo(() => {
        console.log('Filtering users...');
        return users.filter(user => 
            user.name.toLowerCase().includes(searchQuery.toLowerCase())
        );
    }, [users, searchQuery]);
    
    const sortedUsers = useMemo(() => {
        console.log('Sorting users...');
        return [...filteredUsers].sort((a, b) => 
            a.name.localeCompare(b.name)
        );
    }, [filteredUsers]);
    
    return (
        <ul>
            {sortedUsers.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}
```

### useMemo vs useCallback
```jsx
function Comparison() {
    // useMemo - memoizes the RESULT of a function
    const memoizedValue = useMemo(() => {
        return computeExpensiveValue(a, b);
    }, [a, b]);
    
    // useCallback - memoizes the FUNCTION itself
    const memoizedCallback = useCallback(() => {
        doSomething(a, b);
    }, [a, b]);
    
    // useCallback is equivalent to:
    const memoizedCallback2 = useMemo(() => {
        return () => doSomething(a, b);
    }, [a, b]);
}
```

## Component Lifecycle

### Lifecycle Equivalents with Hooks
```jsx
function LifecycleComponent() {
    const [data, setData] = useState(null);
    
    // ComponentDidMount - runs once after initial render
    useEffect(() => {
        console.log('Component mounted');
        // Fetch initial data, set up subscriptions, etc.
    }, []);
    
    // ComponentDidUpdate - runs after every update
    useEffect(() => {
        console.log('Component updated');
    });
    
    // ComponentWillUnmount - cleanup function
    useEffect(() => {
        return () => {
            console.log('Component will unmount');
            // Clean up subscriptions, timers, etc.
        };
    }, []);
    
    // Specific dependency updates
    useEffect(() => {
        console.log('Data changed:', data);
    }, [data]);
    
    return <div>{data}</div>;
}
```

### Combining Lifecycle Behaviors
```jsx
function CompleteLifecycle({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        console.log('Mount: Setting up subscription');
        let cancelled = false;
        
        async function loadUser() {
            setLoading(true);
            try {
                const response = await fetch(`/api/users/${userId}`);
                const data = await response.json();
                
                if (!cancelled) {
                    setUser(data);
                    setLoading(false);
                }
            } catch (error) {
                if (!cancelled) {
                    console.error('Error:', error);
                    setLoading(false);
                }
            }
        }
        
        loadUser();
        
        // Cleanup
        return () => {
            console.log('Unmount: Cleaning up');
            cancelled = true;
        };
    }, [userId]); // Runs when userId changes
    
    if (loading) return <div>Loading...</div>;
    return <div>{user?.name}</div>;
}
```

## Forms and Controlled Components

### Basic Controlled Input
```jsx
function LoginForm() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    
    const handleSubmit = (e) => {
        e.preventDefault();
        console.log('Email:', email, 'Password:', password);
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                placeholder="Email"
            />
            <input
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                placeholder="Password"
            />
            <button type="submit">Login</button>
        </form>
    );
}
```

### Form with Multiple Inputs
```jsx
function RegistrationForm() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: '',
        age: ''
    });
    
    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };
    
    const handleSubmit = (e) => {
        e.preventDefault();
        console.log('Form data:', formData);
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input
                name="username"
                value={formData.username}
                onChange={handleChange}
                placeholder="Username"
            />
            <input
                name="email"
                type="email"
                value={formData.email}
                onChange={handleChange}
                placeholder="Email"
            />
            <input
                name="password"
                type="password"
                value={formData.password}
                onChange={handleChange}
                placeholder="Password"
            />
            <input
                name="age"
                type="number"
                value={formData.age}
                onChange={handleChange}
                placeholder="Age"
            />
            <button type="submit">Register</button>
        </form>
    );
}
```

### Checkbox and Radio Inputs
```jsx
function PreferencesForm() {
    const [preferences, setPreferences] = useState({
        newsletter: false,
        notifications: true,
        theme: 'light'
    });
    
    const handleCheckboxChange = (e) => {
        const { name, checked } = e.target;
        setPreferences(prev => ({
            ...prev,
            [name]: checked
        }));
    };
    
    const handleRadioChange = (e) => {
        setPreferences(prev => ({
            ...prev,
            theme: e.target.value
        }));
    };
    
    return (
        <form>
            <label>
                <input
                    type="checkbox"
                    name="newsletter"
                    checked={preferences.newsletter}
                    onChange={handleCheckboxChange}
                />
                Subscribe to newsletter
            </label>
            
            <label>
                <input
                    type="checkbox"
                    name="notifications"
                    checked={preferences.notifications}
                    onChange={handleCheckboxChange}
                />
                Enable notifications
            </label>
            
            <div>
                <label>
                    <input
                        type="radio"
                        value="light"
                        checked={preferences.theme === 'light'}
                        onChange={handleRadioChange}
                    />
                    Light Theme
                </label>
                <label>
                    <input
                        type="radio"
                        value="dark"
                        checked={preferences.theme === 'dark'}
                        onChange={handleRadioChange}
                    />
                    Dark Theme
                </label>
            </div>
        </form>
    );
}
```

### Select Dropdown
```jsx
function CountrySelector() {
    const [country, setCountry] = useState('');
    
    const countries = [
        { code: 'us', name: 'United States' },
        { code: 'uk', name: 'United Kingdom' },
        { code: 'ca', name: 'Canada' }
    ];
    
    return (
        <select 
            value={country} 
            onChange={(e) => setCountry(e.target.value)}
        >
            <option value="">Select a country</option>
            {countries.map(c => (
                <option key={c.code} value={c.code}>
                    {c.name}
                </option>
            ))}
        </select>
    );
}
```

### Form Validation
```jsx
function ValidatedForm() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [errors, setErrors] = useState({});
    
    const validate = () => {
        const newErrors = {};
        
        if (!email) {
            newErrors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(email)) {
            newErrors.email = 'Email is invalid';
        }
        
        if (!password) {
            newErrors.password = 'Password is required';
        } else if (password.length < 6) {
            newErrors.password = 'Password must be at least 6 characters';
        }
        
        return newErrors;
    };
    
    const handleSubmit = (e) => {
        e.preventDefault();
        const validationErrors = validate();
        
        if (Object.keys(validationErrors).length === 0) {
            console.log('Form is valid:', { email, password });
            setErrors({});
        } else {
            setErrors(validationErrors);
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <div>
                <input
                    type="email"
                    value={email}
                    onChange={(e) => setEmail(e.target.value)}
                    placeholder="Email"
                />
                {errors.email && <span className="error">{errors.email}</span>}
            </div>
            
            <div>
                <input
                    type="password"
                    value={password}
                    onChange={(e) => setPassword(e.target.value)}
                    placeholder="Password"
                />
                {errors.password && <span className="error">{errors.password}</span>}
            </div>
            
            <button type="submit">Submit</button>
        </form>
    );
}
```

## React Router Basics

### Setting Up Router
```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
    return (
        <BrowserRouter>
            <nav>
                <Link to="/">Home</Link>
                <Link to="/about">About</Link>
                <Link to="/contact">Contact</Link>
            </nav>
            
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                <Route path="/contact" element={<Contact />} />
                <Route path="*" element={<NotFound />} />
            </Routes>
        </BrowserRouter>
    );
}
```

### Route Parameters
```jsx
import { useParams, useNavigate } from 'react-router-dom';

function UserProfile() {
    const { userId } = useParams();
    const [user, setUser] = useState(null);
    
    useEffect(() => {
        fetch(`/api/users/${userId}`)
            .then(res => res.json())
            .then(data => setUser(data));
    }, [userId]);
    
    return <div>{user?.name}</div>;
}

// Route definition
<Route path="/users/:userId" element={<UserProfile />} />
```

### Programmatic Navigation
```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
    const navigate = useNavigate();
    
    const handleLogin = async (credentials) => {
        const success = await login(credentials);
        
        if (success) {
            navigate('/dashboard');
            // Or with replace (no history entry)
            // navigate('/dashboard', { replace: true });
        }
    };
    
    return (
        <form onSubmit={handleLogin}>
            {/* form fields */}
        </form>
    );
}
```

### Nested Routes
```jsx
function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Layout />}>
                    <Route index element={<Home />} />
                    <Route path="about" element={<About />} />
                    <Route path="users" element={<Users />}>
                        <Route index element={<UserList />} />
                        <Route path=":userId" element={<UserProfile />} />
                    </Route>
                </Route>
            </Routes>
        </BrowserRouter>
    );
}

// Layout component with Outlet
import { Outlet } from 'react-router-dom';

function Layout() {
    return (
        <div>
            <Header />
            <main>
                <Outlet /> {/* Child routes render here */}
            </main>
            <Footer />
        </div>
    );
}
```

### Query Parameters
```jsx
import { useSearchParams } from 'react-router-dom';

function SearchResults() {
    const [searchParams, setSearchParams] = useSearchParams();
    
    const query = searchParams.get('q');
    const page = searchParams.get('page') || '1';
    
    const updateSearch = (newQuery) => {
        setSearchParams({ q: newQuery, page: '1' });
    };
    
    return (
        <div>
            <p>Searching for: {query}</p>
            <p>Page: {page}</p>
        </div>
    );
}

// URL: /search?q=react&page=2
```

## Lifting State Up

### Sharing State Between Components
```jsx
// Parent component holds shared state
function TemperatureConverter() {
    const [celsius, setCelsius] = useState('');
    
    const handleCelsiusChange = (value) => {
        setCelsius(value);
    };
    
    const handleFahrenheitChange = (value) => {
        setCelsius(((value - 32) * 5) / 9);
    };
    
    const fahrenheit = celsius ? (celsius * 9) / 5 + 32 : '';
    
    return (
        <div>
            <TemperatureInput
                scale="Celsius"
                temperature={celsius}
                onTemperatureChange={handleCelsiusChange}
            />
            <TemperatureInput
                scale="Fahrenheit"
                temperature={fahrenheit}
                onTemperatureChange={handleFahrenheitChange}
            />
        </div>
    );
}

// Child component receives state and callback
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
    return (
        <div>
            <label>Temperature in {scale}:</label>
            <input
                value={temperature}
                onChange={(e) => onTemperatureChange(e.target.value)}
            />
        </div>
    );
}
```

### Sibling Component Communication
```jsx
function TodoApp() {
    const [todos, setTodos] = useState([]);
    
    const addTodo = (text) => {
        setTodos([...todos, { id: Date.now(), text, completed: false }]);
    };
    
    const toggleTodo = (id) => {
        setTodos(todos.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        ));
    };
    
    return (
        <div>
            <TodoInput onAddTodo={addTodo} />
            <TodoList todos={todos} onToggleTodo={toggleTodo} />
        </div>
    );
}

function TodoInput({ onAddTodo }) {
    const [text, setText] = useState('');
    
    const handleSubmit = (e) => {
        e.preventDefault();
        if (text.trim()) {
            onAddTodo(text);
            setText('');
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input 
                value={text} 
                onChange={(e) => setText(e.target.value)} 
            />
            <button type="submit">Add</button>
        </form>
    );
}

function TodoList({ todos, onToggleTodo }) {
    return (
        <ul>
            {todos.map(todo => (
                <li key={todo.id}>
                    <input
                        type="checkbox"
                        checked={todo.completed}
                        onChange={() => onToggleTodo(todo.id)}
                    />
                    {todo.text}
                </li>
            ))}
        </ul>
    );
}
```

## Component Composition

### Children Props Pattern
```jsx
function Card({ children, title }) {
    return (
        <div className="card">
            <h2>{title}</h2>
            <div className="card-body">
                {children}
            </div>
        </div>
    );
}

// Usage
function App() {
    return (
        <Card title="User Info">
            <p>Name: John Doe</p>
            <p>Email: john@example.com</p>
            <button>Edit Profile</button>
        </Card>
    );
}
```

### Slot Pattern with Named Children
```jsx
function Dialog({ header, body, footer }) {
    return (
        <div className="dialog">
            <div className="dialog-header">{header}</div>
            <div className="dialog-body">{body}</div>
            <div className="dialog-footer">{footer}</div>
        </div>
    );
}

// Usage
function App() {
    return (
        <Dialog
            header={<h2>Confirm Action</h2>}
            body={<p>Are you sure you want to continue?</p>}
            footer={
                <>
                    <button>Cancel</button>
                    <button>Confirm</button>
                </>
            }
        />
    );
}
```

### Container/Presenter Pattern
```jsx
// Container (logic)
function UserProfileContainer({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        fetch(`/api/users/${userId}`)
            .then(res => res.json())
            .then(data => {
                setUser(data);
                setLoading(false);
            });
    }, [userId]);
    
    if (loading) return <div>Loading...</div>;
    
    return <UserProfilePresenter user={user} />;
}

// Presenter (UI)
function UserProfilePresenter({ user }) {
    return (
        <div className="profile">
            <h1>{user.name}</h1>
            <p>{user.email}</p>
            <p>{user.bio}</p>
        </div>
    );
}
```

### Wrapper Components
```jsx
function ErrorBoundaryWrapper({ children }) {
    return (
        <ErrorBoundary>
            <Suspense fallback={<Loading />}>
                {children}
            </Suspense>
        </ErrorBoundary>
    );
}

// Usage
function App() {
    return (
        <ErrorBoundaryWrapper>
            <UserDashboard />
        </ErrorBoundaryWrapper>
    );
}
```

## Error Boundaries

### Basic Error Boundary
```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }
    
    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }
    
    componentDidCatch(error, errorInfo) {
        console.error('Error caught by boundary:', error, errorInfo);
        // Log to error reporting service
    }
    
    render() {
        if (this.state.hasError) {
            return (
                <div className="error-boundary">
                    <h1>Something went wrong</h1>
                    <p>{this.state.error?.message}</p>
                    <button onClick={() => this.setState({ hasError: false })}>
                        Try again
                    </button>
                </div>
            );
        }
        
        return this.props.children;
    }
}

// Usage
function App() {
    return (
        <ErrorBoundary>
            <MyComponent />
        </ErrorBoundary>
    );
}
```

### Custom Error Fallback
```jsx
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }
    
    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }
    
    componentDidCatch(error, errorInfo) {
        this.props.onError?.(error, errorInfo);
    }
    
    render() {
        if (this.state.hasError) {
            return this.props.fallback ? (
                this.props.fallback(this.state.error)
            ) : (
                <div>An error occurred</div>
            );
        }
        
        return this.props.children;
    }
}

// Usage with custom fallback
function App() {
    return (
        <ErrorBoundary 
            fallback={(error) => (
                <div className="custom-error">
                    <h2>Oops! {error.message}</h2>
                    <button onClick={() => window.location.reload()}>
                        Reload Page
                    </button>
                </div>
            )}
            onError={(error, errorInfo) => {
                logErrorToService(error, errorInfo);
            }}
        >
            <App />
        </ErrorBoundary>
    );
}
```

### Multiple Error Boundaries
```jsx
function App() {
    return (
        <ErrorBoundary fallback={<div>App-level error</div>}>
            <Header />
            
            <ErrorBoundary fallback={<div>Sidebar error</div>}>
                <Sidebar />
            </ErrorBoundary>
            
            <ErrorBoundary fallback={<div>Content error</div>}>
                <MainContent />
            </ErrorBoundary>
            
            <Footer />
        </ErrorBoundary>
    );
}
```

## Best Practices (Do's)

✅ **Use useEffect cleanup for subscriptions and timers**
```jsx
// Good
useEffect(() => {
    const interval = setInterval(() => {
        // Do something
    }, 1000);
    
    return () => clearInterval(interval);
}, []);
```

✅ **Memoize expensive calculations with useMemo**
```jsx
// Good
const sortedData = useMemo(() => {
    return data.sort((a, b) => a.value - b.value);
}, [data]);
```

✅ **Use useCallback for callbacks passed to child components**
```jsx
// Good
const handleClick = useCallback(() => {
    doSomething(id);
}, [id]);

return <ChildComponent onClick={handleClick} />;
```

✅ **Validate forms before submission**
```jsx
// Good
const handleSubmit = (e) => {
    e.preventDefault();
    const errors = validateForm(formData);
    
    if (Object.keys(errors).length === 0) {
        submitForm(formData);
    } else {
        setErrors(errors);
    }
};
```

✅ **Keep forms controlled**
```jsx
// Good
const [value, setValue] = useState('');

<input 
    value={value} 
    onChange={(e) => setValue(e.target.value)} 
/>
```

✅ **Use Error Boundaries to catch errors**
```jsx
// Good
<ErrorBoundary>
    <MyComponent />
</ErrorBoundary>
```

## Common Mistakes (Don'ts)

❌ **Don't forget cleanup in useEffect**
```jsx
// Wrong - memory leak
useEffect(() => {
    const interval = setInterval(() => {
        setCount(prev => prev + 1);
    }, 1000);
    // Missing cleanup!
}, []);

// Correct
useEffect(() => {
    const interval = setInterval(() => {
        setCount(prev => prev + 1);
    }, 1000);
    
    return () => clearInterval(interval);
}, []);
```

❌ **Don't use async functions directly in useEffect**
```jsx
// Wrong
useEffect(async () => {
    const data = await fetchData();
    setData(data);
}, []);

// Correct
useEffect(() => {
    async function loadData() {
        const data = await fetchData();
        setData(data);
    }
    
    loadData();
}, []);
```

❌ **Don't omit dependencies in useEffect**
```jsx
// Wrong - can cause bugs
useEffect(() => {
    console.log(count);
}, []); // Missing 'count' dependency

// Correct
useEffect(() => {
    console.log(count);
}, [count]);
```

❌ **Don't use inline objects/arrays as dependencies**
```jsx
// Wrong - creates new reference every render
useEffect(() => {
    // Do something
}, [{ id: userId }]); // New object each time

// Correct - use primitive values
useEffect(() => {
    // Do something
}, [userId]);
```

❌ **Don't forget to preventDefault on form submission**
```jsx
// Wrong - page refreshes
const handleSubmit = () => {
    submitForm();
};

// Correct
const handleSubmit = (e) => {
    e.preventDefault();
    submitForm();
};
```

❌ **Don't create functions inside JSX for event handlers**
```jsx
// Wrong - new function on every render
<button onClick={() => handleClick(id)}>Click</button>

// Better for frequently rendering components
const handleButtonClick = useCallback(() => {
    handleClick(id);
}, [id]);

<button onClick={handleButtonClick}>Click</button>
```

❌ **Don't use wrong ref types**
```jsx
// Wrong - useRef for component state that affects UI
const [count, setCount] = useRef(0); // Won't trigger re-render

// Correct
const [count, setCount] = useState(0); // Triggers re-render

// Wrong - useState for values that don't affect UI
const [intervalId, setIntervalId] = useState(null);

// Correct
const intervalId = useRef(null);
```

❌ **Don't directly mutate ref.current in render**
```jsx
// Wrong
function Component() {
    const count = useRef(0);
    count.current++; // Don't mutate during render
    return <div>{count.current}</div>;
}

// Correct - mutate in effect or event handler
function Component() {
    const count = useRef(0);
    
    useEffect(() => {
        count.current++;
    });
    
    return <div>Component rendered {count.current} times</div>;
}
```
