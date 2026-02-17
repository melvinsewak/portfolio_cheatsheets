# React Advanced Cheatsheet

## Context API and useContext

### Creating and Using Context
```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();

// Provider component
function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');
    
    const toggleTheme = () => {
        setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
    };
    
    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

// Custom hook for using context
function useTheme() {
    const context = useContext(ThemeContext);
    if (context === undefined) {
        throw new Error('useTheme must be used within ThemeProvider');
    }
    return context;
}

// Usage in component
function ThemedButton() {
    const { theme, toggleTheme } = useTheme();
    
    return (
        <button 
            onClick={toggleTheme}
            style={{ 
                background: theme === 'light' ? '#fff' : '#333',
                color: theme === 'light' ? '#333' : '#fff'
            }}
        >
            Toggle Theme
        </button>
    );
}

// App with provider
function App() {
    return (
        <ThemeProvider>
            <ThemedButton />
        </ThemeProvider>
    );
}
```

### Multiple Contexts
```jsx
const ThemeContext = createContext();
const AuthContext = createContext();
const LanguageContext = createContext();

function App() {
    return (
        <ThemeProvider>
            <AuthProvider>
                <LanguageProvider>
                    <MainApp />
                </LanguageProvider>
            </AuthProvider>
        </ThemeProvider>
    );
}

// Using multiple contexts
function UserProfile() {
    const { theme } = useTheme();
    const { user } = useAuth();
    const { language } = useLanguage();
    
    return (
        <div className={`profile theme-${theme} lang-${language}`}>
            <h1>{user.name}</h1>
        </div>
    );
}
```

### Context with Complex State
```jsx
const UserContext = createContext();

function UserProvider({ children }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    const login = async (credentials) => {
        setLoading(true);
        setError(null);
        try {
            const response = await fetch('/api/login', {
                method: 'POST',
                body: JSON.stringify(credentials)
            });
            const data = await response.json();
            setUser(data.user);
            return true;
        } catch (err) {
            setError(err.message);
            return false;
        } finally {
            setLoading(false);
        }
    };
    
    const logout = () => {
        setUser(null);
    };
    
    const value = {
        user,
        loading,
        error,
        login,
        logout,
        isAuthenticated: !!user
    };
    
    return (
        <UserContext.Provider value={value}>
            {children}
        </UserContext.Provider>
    );
}

function useUser() {
    const context = useContext(UserContext);
    if (context === undefined) {
        throw new Error('useUser must be used within UserProvider');
    }
    return context;
}
```

### Context Performance Optimization
```jsx
// Split context by update frequency
const UserStateContext = createContext();
const UserActionsContext = createContext();

function UserProvider({ children }) {
    const [user, setUser] = useState(null);
    
    // Actions don't change
    const actions = useMemo(() => ({
        login: (userData) => setUser(userData),
        logout: () => setUser(null),
        updateUser: (updates) => setUser(prev => ({ ...prev, ...updates }))
    }), []);
    
    return (
        <UserStateContext.Provider value={user}>
            <UserActionsContext.Provider value={actions}>
                {children}
            </UserActionsContext.Provider>
        </UserStateContext.Provider>
    );
}

// Separate hooks
function useUserState() {
    return useContext(UserStateContext);
}

function useUserActions() {
    return useContext(UserActionsContext);
}

// Components only re-render when needed
function UserDisplay() {
    const user = useUserState(); // Re-renders when user changes
    return <div>{user?.name}</div>;
}

function LogoutButton() {
    const { logout } = useUserActions(); // Never re-renders
    return <button onClick={logout}>Logout</button>;
}
```

## useReducer for Complex State

### Basic useReducer
```jsx
import { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
    switch (action.type) {
        case 'INCREMENT':
            return { count: state.count + 1 };
        case 'DECREMENT':
            return { count: state.count - 1 };
        case 'RESET':
            return { count: 0 };
        default:
            throw new Error(`Unknown action: ${action.type}`);
    }
}

function Counter() {
    const [state, dispatch] = useReducer(counterReducer, { count: 0 });
    
    return (
        <div>
            <p>Count: {state.count}</p>
            <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
            <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
            <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
        </div>
    );
}
```

### useReducer with Payload
```jsx
function todoReducer(state, action) {
    switch (action.type) {
        case 'ADD_TODO':
            return {
                ...state,
                todos: [...state.todos, {
                    id: Date.now(),
                    text: action.payload,
                    completed: false
                }]
            };
        case 'TOGGLE_TODO':
            return {
                ...state,
                todos: state.todos.map(todo =>
                    todo.id === action.payload
                        ? { ...todo, completed: !todo.completed }
                        : todo
                )
            };
        case 'DELETE_TODO':
            return {
                ...state,
                todos: state.todos.filter(todo => todo.id !== action.payload)
            };
        case 'SET_FILTER':
            return {
                ...state,
                filter: action.payload
            };
        default:
            return state;
    }
}

function TodoApp() {
    const initialState = {
        todos: [],
        filter: 'all'
    };
    
    const [state, dispatch] = useReducer(todoReducer, initialState);
    
    const addTodo = (text) => {
        dispatch({ type: 'ADD_TODO', payload: text });
    };
    
    const toggleTodo = (id) => {
        dispatch({ type: 'TOGGLE_TODO', payload: id });
    };
    
    const deleteTodo = (id) => {
        dispatch({ type: 'DELETE_TODO', payload: id });
    };
    
    return (
        <div>
            {/* Todo UI */}
        </div>
    );
}
```

### useReducer with Context (Redux Pattern)
```jsx
const StateContext = createContext();
const DispatchContext = createContext();

function appReducer(state, action) {
    switch (action.type) {
        case 'SET_USER':
            return { ...state, user: action.payload };
        case 'SET_THEME':
            return { ...state, theme: action.payload };
        case 'ADD_NOTIFICATION':
            return {
                ...state,
                notifications: [...state.notifications, action.payload]
            };
        case 'REMOVE_NOTIFICATION':
            return {
                ...state,
                notifications: state.notifications.filter(n => n.id !== action.payload)
            };
        default:
            return state;
    }
}

function AppProvider({ children }) {
    const initialState = {
        user: null,
        theme: 'light',
        notifications: []
    };
    
    const [state, dispatch] = useReducer(appReducer, initialState);
    
    return (
        <StateContext.Provider value={state}>
            <DispatchContext.Provider value={dispatch}>
                {children}
            </DispatchContext.Provider>
        </StateContext.Provider>
    );
}

function useAppState() {
    const context = useContext(StateContext);
    if (context === undefined) {
        throw new Error('useAppState must be used within AppProvider');
    }
    return context;
}

function useAppDispatch() {
    const context = useContext(DispatchContext);
    if (context === undefined) {
        throw new Error('useAppDispatch must be used within AppProvider');
    }
    return context;
}

// Usage
function UserProfile() {
    const { user } = useAppState();
    const dispatch = useAppDispatch();
    
    const updateTheme = (theme) => {
        dispatch({ type: 'SET_THEME', payload: theme });
    };
    
    return <div>{user?.name}</div>;
}
```

### Async Actions with useReducer
```jsx
function dataReducer(state, action) {
    switch (action.type) {
        case 'FETCH_START':
            return { ...state, loading: true, error: null };
        case 'FETCH_SUCCESS':
            return { ...state, loading: false, data: action.payload };
        case 'FETCH_ERROR':
            return { ...state, loading: false, error: action.payload };
        default:
            return state;
    }
}

function DataFetcher() {
    const [state, dispatch] = useReducer(dataReducer, {
        data: null,
        loading: false,
        error: null
    });
    
    const fetchData = async () => {
        dispatch({ type: 'FETCH_START' });
        try {
            const response = await fetch('/api/data');
            const data = await response.json();
            dispatch({ type: 'FETCH_SUCCESS', payload: data });
        } catch (error) {
            dispatch({ type: 'FETCH_ERROR', payload: error.message });
        }
    };
    
    useEffect(() => {
        fetchData();
    }, []);
    
    if (state.loading) return <div>Loading...</div>;
    if (state.error) return <div>Error: {state.error}</div>;
    
    return <div>{JSON.stringify(state.data)}</div>;
}
```

## Custom Hooks

### Basic Custom Hook
```jsx
// Custom hook for form input
function useInput(initialValue = '') {
    const [value, setValue] = useState(initialValue);
    
    const reset = () => setValue(initialValue);
    
    const bind = {
        value,
        onChange: (e) => setValue(e.target.value)
    };
    
    return [value, bind, reset];
}

// Usage
function LoginForm() {
    const [email, emailBind, resetEmail] = useInput('');
    const [password, passwordBind, resetPassword] = useInput('');
    
    const handleSubmit = (e) => {
        e.preventDefault();
        console.log('Email:', email, 'Password:', password);
        resetEmail();
        resetPassword();
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input type="email" {...emailBind} placeholder="Email" />
            <input type="password" {...passwordBind} placeholder="Password" />
            <button type="submit">Login</button>
        </form>
    );
}
```

### Custom Hook for API Calls
```jsx
function useFetch(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        let cancelled = false;
        
        async function fetchData() {
            try {
                setLoading(true);
                const response = await fetch(url);
                const result = await response.json();
                
                if (!cancelled) {
                    setData(result);
                    setError(null);
                }
            } catch (err) {
                if (!cancelled) {
                    setError(err.message);
                }
            } finally {
                if (!cancelled) {
                    setLoading(false);
                }
            }
        }
        
        fetchData();
        
        return () => {
            cancelled = true;
        };
    }, [url]);
    
    return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
    const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
    
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    
    return <div>{user.name}</div>;
}
```

### Custom Hook with Refetch
```jsx
function useApi(url, options = {}) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    const fetchData = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const response = await fetch(url, options);
            const result = await response.json();
            setData(result);
            return result;
        } catch (err) {
            setError(err.message);
            throw err;
        } finally {
            setLoading(false);
        }
    }, [url, options]);
    
    useEffect(() => {
        fetchData();
    }, [fetchData]);
    
    return { data, loading, error, refetch: fetchData };
}

// Usage
function Posts() {
    const { data: posts, loading, error, refetch } = useApi('/api/posts');
    
    return (
        <div>
            <button onClick={refetch}>Refresh</button>
            {loading && <div>Loading...</div>}
            {error && <div>Error: {error}</div>}
            {posts && posts.map(post => <Post key={post.id} {...post} />)}
        </div>
    );
}
```

### Custom Hook for Local Storage
```jsx
function useLocalStorage(key, initialValue) {
    const [storedValue, setStoredValue] = useState(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            console.error(error);
            return initialValue;
        }
    });
    
    const setValue = (value) => {
        try {
            const valueToStore = value instanceof Function 
                ? value(storedValue) 
                : value;
            
            setStoredValue(valueToStore);
            window.localStorage.setItem(key, JSON.stringify(valueToStore));
        } catch (error) {
            console.error(error);
        }
    };
    
    return [storedValue, setValue];
}

// Usage
function Settings() {
    const [theme, setTheme] = useLocalStorage('theme', 'light');
    const [language, setLanguage] = useLocalStorage('language', 'en');
    
    return (
        <div>
            <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
                Toggle Theme
            </button>
            <select value={language} onChange={(e) => setLanguage(e.target.value)}>
                <option value="en">English</option>
                <option value="es">Spanish</option>
            </select>
        </div>
    );
}
```

### Custom Hook for Window Dimensions
```jsx
function useWindowSize() {
    const [windowSize, setWindowSize] = useState({
        width: undefined,
        height: undefined
    });
    
    useEffect(() => {
        function handleResize() {
            setWindowSize({
                width: window.innerWidth,
                height: window.innerHeight
            });
        }
        
        window.addEventListener('resize', handleResize);
        handleResize(); // Initial call
        
        return () => window.removeEventListener('resize', handleResize);
    }, []);
    
    return windowSize;
}

// Usage
function ResponsiveComponent() {
    const { width } = useWindowSize();
    
    return (
        <div>
            {width < 768 ? <MobileView /> : <DesktopView />}
        </div>
    );
}
```

### Custom Hook for Debouncing
```jsx
function useDebounce(value, delay) {
    const [debouncedValue, setDebouncedValue] = useState(value);
    
    useEffect(() => {
        const handler = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);
        
        return () => {
            clearTimeout(handler);
        };
    }, [value, delay]);
    
    return debouncedValue;
}

// Usage
function SearchComponent() {
    const [searchTerm, setSearchTerm] = useState('');
    const debouncedSearchTerm = useDebounce(searchTerm, 500);
    
    useEffect(() => {
        if (debouncedSearchTerm) {
            // API call with debounced value
            fetch(`/api/search?q=${debouncedSearchTerm}`)
                .then(res => res.json())
                .then(data => setResults(data));
        }
    }, [debouncedSearchTerm]);
    
    return (
        <input 
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            placeholder="Search..."
        />
    );
}
```

## Performance Optimization

### React.memo
```jsx
import { memo } from 'react';

// Without memo - re-renders on every parent render
function ExpensiveComponent({ data }) {
    console.log('Rendering ExpensiveComponent');
    return <div>{data.name}</div>;
}

// With memo - only re-renders when props change
const MemoizedComponent = memo(function ExpensiveComponent({ data }) {
    console.log('Rendering MemoizedComponent');
    return <div>{data.name}</div>;
});

// Custom comparison function
const MemoizedWithComparison = memo(
    function ExpensiveComponent({ data }) {
        return <div>{data.name}</div>;
    },
    (prevProps, nextProps) => {
        // Return true if props are equal (don't re-render)
        return prevProps.data.id === nextProps.data.id;
    }
);

// Usage
function Parent() {
    const [count, setCount] = useState(0);
    const [user] = useState({ id: 1, name: 'Alice' });
    
    return (
        <div>
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
            <MemoizedComponent data={user} /> {/* Won't re-render */}
        </div>
    );
}
```

### useMemo for Expensive Calculations
```jsx
function DataTable({ data, filters }) {
    // Without useMemo - recalculates on every render
    const filteredData = data.filter(item => 
        item.category === filters.category
    ).sort((a, b) => a.price - b.price);
    
    // With useMemo - only recalculates when dependencies change
    const optimizedData = useMemo(() => {
        console.log('Filtering and sorting data...');
        return data
            .filter(item => item.category === filters.category)
            .sort((a, b) => a.price - b.price);
    }, [data, filters.category]);
    
    return (
        <table>
            {optimizedData.map(item => (
                <tr key={item.id}>
                    <td>{item.name}</td>
                    <td>{item.price}</td>
                </tr>
            ))}
        </table>
    );
}
```

### useCallback for Stable Function References
```jsx
function ParentComponent() {
    const [count, setCount] = useState(0);
    const [other, setOther] = useState(0);
    
    // Without useCallback - new function on every render
    const handleClick = () => {
        console.log('Clicked');
    };
    
    // With useCallback - same function unless dependencies change
    const handleClickMemo = useCallback(() => {
        console.log('Clicked with count:', count);
    }, [count]);
    
    return (
        <div>
            <MemoizedChild onClick={handleClickMemo} />
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
            <button onClick={() => setOther(other + 1)}>Other: {other}</button>
        </div>
    );
}

const MemoizedChild = memo(({ onClick }) => {
    console.log('Child rendered');
    return <button onClick={onClick}>Click Me</button>;
});
```

### Virtualization for Long Lists
```jsx
// Using react-window (install: npm install react-window)
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
    const Row = ({ index, style }) => (
        <div style={style}>
            {items[index].name}
        </div>
    );
    
    return (
        <FixedSizeList
            height={600}
            itemCount={items.length}
            itemSize={50}
            width="100%"
        >
            {Row}
        </FixedSizeList>
    );
}

// Manual windowing concept
function WindowedList({ items, itemHeight = 50, containerHeight = 500 }) {
    const [scrollTop, setScrollTop] = useState(0);
    
    const startIndex = Math.floor(scrollTop / itemHeight);
    const endIndex = Math.ceil((scrollTop + containerHeight) / itemHeight);
    const visibleItems = items.slice(startIndex, endIndex);
    
    return (
        <div 
            style={{ height: containerHeight, overflow: 'auto' }}
            onScroll={(e) => setScrollTop(e.target.scrollTop)}
        >
            <div style={{ height: items.length * itemHeight, position: 'relative' }}>
                {visibleItems.map((item, index) => (
                    <div
                        key={item.id}
                        style={{
                            position: 'absolute',
                            top: (startIndex + index) * itemHeight,
                            height: itemHeight
                        }}
                    >
                        {item.name}
                    </div>
                ))}
            </div>
        </div>
    );
}
```

### Avoid Unnecessary Re-renders
```jsx
// Problem: Parent state change causes child re-render
function Parent() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
            <ExpensiveChild /> {/* Re-renders unnecessarily */}
        </div>
    );
}

// Solution 1: Move state down
function Parent() {
    return (
        <div>
            <Counter /> {/* State is local */}
            <ExpensiveChild /> {/* Won't re-render */}
        </div>
    );
}

function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}

// Solution 2: Lift content up
function Parent({ children }) {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
            {children} {/* Won't re-render - created outside */}
        </div>
    );
}

// Usage
<Parent>
    <ExpensiveChild />
</Parent>
```

## Code Splitting and Lazy Loading

### React.lazy and Suspense
```jsx
import { lazy, Suspense } from 'react';

// Lazy load component
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

function App() {
    return (
        <div>
            <Suspense fallback={<div>Loading...</div>}>
                <HeavyComponent />
            </Suspense>
        </div>
    );
}
```

### Route-based Code Splitting
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { lazy, Suspense } from 'react';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
    return (
        <BrowserRouter>
            <Suspense fallback={<LoadingSpinner />}>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="/about" element={<About />} />
                    <Route path="/dashboard" element={<Dashboard />} />
                    <Route path="/profile" element={<Profile />} />
                </Routes>
            </Suspense>
        </BrowserRouter>
    );
}
```

### Conditional Loading
```jsx
function App() {
    const [showAdmin, setShowAdmin] = useState(false);
    
    // Only load admin panel when needed
    const AdminPanel = lazy(() => import('./AdminPanel'));
    
    return (
        <div>
            <button onClick={() => setShowAdmin(true)}>
                Show Admin Panel
            </button>
            
            {showAdmin && (
                <Suspense fallback={<div>Loading admin panel...</div>}>
                    <AdminPanel />
                </Suspense>
            )}
        </div>
    );
}
```

### Preloading Components
```jsx
const OtherComponent = lazy(() => import('./OtherComponent'));

// Preload on hover
function MyComponent() {
    const preload = () => {
        const component = import('./OtherComponent');
    };
    
    return (
        <div>
            <button onMouseEnter={preload}>
                Show Other Component
            </button>
        </div>
    );
}
```

### Error Boundaries with Lazy Loading
```jsx
class LazyLoadErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false };
    }
    
    static getDerivedStateFromError(error) {
        return { hasError: true };
    }
    
    render() {
        if (this.state.hasError) {
            return <div>Failed to load component</div>;
        }
        return this.props.children;
    }
}

function App() {
    const HeavyComponent = lazy(() => import('./HeavyComponent'));
    
    return (
        <LazyLoadErrorBoundary>
            <Suspense fallback={<div>Loading...</div>}>
                <HeavyComponent />
            </Suspense>
        </LazyLoadErrorBoundary>
    );
}
```

## Advanced Patterns

### Higher-Order Components (HOC)
```jsx
// Basic HOC
function withLoading(Component) {
    return function WithLoadingComponent({ isLoading, ...props }) {
        if (isLoading) {
            return <div>Loading...</div>;
        }
        return <Component {...props} />;
    };
}

// Usage
function DataDisplay({ data }) {
    return <div>{data}</div>;
}

const DataDisplayWithLoading = withLoading(DataDisplay);

// In parent
<DataDisplayWithLoading isLoading={loading} data={data} />

// HOC with additional props
function withAuth(Component) {
    return function WithAuthComponent(props) {
        const { user, loading } = useAuth();
        
        if (loading) return <div>Loading...</div>;
        if (!user) return <div>Please log in</div>;
        
        return <Component {...props} user={user} />;
    };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);

// Composing HOCs
const EnhancedComponent = withAuth(withLoading(withTheme(Component)));
```

### Render Props Pattern
```jsx
// Basic render prop
function Mouse({ render }) {
    const [position, setPosition] = useState({ x: 0, y: 0 });
    
    useEffect(() => {
        const handleMouseMove = (e) => {
            setPosition({ x: e.clientX, y: e.clientY });
        };
        
        window.addEventListener('mousemove', handleMouseMove);
        return () => window.removeEventListener('mousemove', handleMouseMove);
    }, []);
    
    return render(position);
}

// Usage
<Mouse render={({ x, y }) => (
    <div>Mouse position: {x}, {y}</div>
)} />

// Using children as render prop
function Toggle({ children }) {
    const [on, setOn] = useState(false);
    
    const toggle = () => setOn(!on);
    
    return children({ on, toggle });
}

// Usage
<Toggle>
    {({ on, toggle }) => (
        <div>
            <button onClick={toggle}>
                {on ? 'ON' : 'OFF'}
            </button>
            {on && <div>Content is visible</div>}
        </div>
    )}
</Toggle>
```

### Compound Components Pattern
```jsx
// Create context for internal state
const TabsContext = createContext();

function Tabs({ children, defaultValue }) {
    const [activeTab, setActiveTab] = useState(defaultValue);
    
    return (
        <TabsContext.Provider value={{ activeTab, setActiveTab }}>
            <div className="tabs">{children}</div>
        </TabsContext.Provider>
    );
}

function TabList({ children }) {
    return <div className="tab-list">{children}</div>;
}

function Tab({ value, children }) {
    const { activeTab, setActiveTab } = useContext(TabsContext);
    const isActive = activeTab === value;
    
    return (
        <button
            className={isActive ? 'tab active' : 'tab'}
            onClick={() => setActiveTab(value)}
        >
            {children}
        </button>
    );
}

function TabPanel({ value, children }) {
    const { activeTab } = useContext(TabsContext);
    
    if (activeTab !== value) return null;
    
    return <div className="tab-panel">{children}</div>;
}

// Attach sub-components
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage
function App() {
    return (
        <Tabs defaultValue="home">
            <Tabs.List>
                <Tabs.Tab value="home">Home</Tabs.Tab>
                <Tabs.Tab value="profile">Profile</Tabs.Tab>
                <Tabs.Tab value="settings">Settings</Tabs.Tab>
            </Tabs.List>
            
            <Tabs.Panel value="home">
                <h2>Home Content</h2>
            </Tabs.Panel>
            <Tabs.Panel value="profile">
                <h2>Profile Content</h2>
            </Tabs.Panel>
            <Tabs.Panel value="settings">
                <h2>Settings Content</h2>
            </Tabs.Panel>
        </Tabs>
    );
}
```

### State Reducer Pattern
```jsx
function useToggle(reducer = (state, action) => action) {
    const [on, setOn] = useState(false);
    
    const toggle = () => setOn(reducer(!on, { type: 'toggle' }));
    const setOnState = () => setOn(reducer(true, { type: 'on' }));
    const setOffState = () => setOn(reducer(false, { type: 'off' }));
    
    return { on, toggle, setOn: setOnState, setOff: setOffState };
}

// User can control state changes
function App() {
    const { on, toggle } = useToggle((state, action) => {
        if (action.type === 'toggle' && state) {
            // Prevent toggling off
            return state;
        }
        return !state;
    });
    
    return <button onClick={toggle}>{on ? 'ON' : 'OFF'}</button>;
}
```

## Refs and Forwarding Refs

### useRef for DOM Access
```jsx
function TextInputWithFocus() {
    const inputRef = useRef(null);
    
    const focusInput = () => {
        inputRef.current.focus();
    };
    
    const selectText = () => {
        inputRef.current.select();
    };
    
    return (
        <div>
            <input ref={inputRef} type="text" />
            <button onClick={focusInput}>Focus</button>
            <button onClick={selectText}>Select</button>
        </div>
    );
}
```

### Forwarding Refs
```jsx
import { forwardRef } from 'react';

// Forward ref to child component
const FancyInput = forwardRef((props, ref) => {
    return (
        <div className="fancy-input">
            <input ref={ref} {...props} />
        </div>
    );
});

// Usage in parent
function App() {
    const inputRef = useRef(null);
    
    const focusInput = () => {
        inputRef.current.focus();
    };
    
    return (
        <div>
            <FancyInput ref={inputRef} placeholder="Enter text" />
            <button onClick={focusInput}>Focus Input</button>
        </div>
    );
}
```

### useImperativeHandle
```jsx
import { forwardRef, useImperativeHandle, useRef } from 'react';

const VideoPlayer = forwardRef((props, ref) => {
    const videoRef = useRef(null);
    
    // Expose only specific methods
    useImperativeHandle(ref, () => ({
        play: () => {
            videoRef.current.play();
        },
        pause: () => {
            videoRef.current.pause();
        },
        reset: () => {
            videoRef.current.currentTime = 0;
            videoRef.current.pause();
        }
    }));
    
    return <video ref={videoRef} src={props.src} />;
});

// Usage
function App() {
    const playerRef = useRef(null);
    
    return (
        <div>
            <VideoPlayer ref={playerRef} src="video.mp4" />
            <button onClick={() => playerRef.current.play()}>Play</button>
            <button onClick={() => playerRef.current.pause()}>Pause</button>
            <button onClick={() => playerRef.current.reset()}>Reset</button>
        </div>
    );
}
```

### Callback Refs
```jsx
function MeasureComponent() {
    const [height, setHeight] = useState(0);
    
    const measuredRef = useCallback(node => {
        if (node !== null) {
            setHeight(node.getBoundingClientRect().height);
        }
    }, []);
    
    return (
        <div>
            <div ref={measuredRef}>
                <p>This div's height is measured</p>
            </div>
            <p>Height: {height}px</p>
        </div>
    );
}
```

## Portals

### Basic Portal
```jsx
import { createPortal } from 'react-dom';

function Modal({ isOpen, children }) {
    if (!isOpen) return null;
    
    return createPortal(
        <div className="modal-overlay">
            <div className="modal">
                {children}
            </div>
        </div>,
        document.getElementById('modal-root')
    );
}

// In index.html, add: <div id="modal-root"></div>

// Usage
function App() {
    const [isOpen, setIsOpen] = useState(false);
    
    return (
        <div>
            <button onClick={() => setIsOpen(true)}>Open Modal</button>
            
            <Modal isOpen={isOpen}>
                <h2>Modal Title</h2>
                <p>Modal content</p>
                <button onClick={() => setIsOpen(false)}>Close</button>
            </Modal>
        </div>
    );
}
```

### Portal with Event Bubbling
```jsx
function ModalWithPortal() {
    const [isOpen, setIsOpen] = useState(false);
    
    // Click event bubbles up through portal
    const handleClick = () => {
        console.log('Parent clicked');
    };
    
    return (
        <div onClick={handleClick}>
            <button onClick={() => setIsOpen(true)}>Open</button>
            
            {isOpen && createPortal(
                <div className="modal">
                    <button onClick={() => setIsOpen(false)}>
                        Close (event bubbles to parent)
                    </button>
                </div>,
                document.body
            )}
        </div>
    );
}
```

### Tooltip with Portal
```jsx
function Tooltip({ children, content }) {
    const [show, setShow] = useState(false);
    const [position, setPosition] = useState({ x: 0, y: 0 });
    const triggerRef = useRef(null);
    
    const handleMouseEnter = () => {
        const rect = triggerRef.current.getBoundingClientRect();
        setPosition({
            x: rect.left + rect.width / 2,
            y: rect.top
        });
        setShow(true);
    };
    
    return (
        <>
            <span
                ref={triggerRef}
                onMouseEnter={handleMouseEnter}
                onMouseLeave={() => setShow(false)}
            >
                {children}
            </span>
            
            {show && createPortal(
                <div 
                    className="tooltip"
                    style={{
                        position: 'fixed',
                        left: position.x,
                        top: position.y - 30,
                        transform: 'translateX(-50%)'
                    }}
                >
                    {content}
                </div>,
                document.body
            )}
        </>
    );
}

// Usage
<Tooltip content="This is a tooltip">
    Hover over me
</Tooltip>
```

## Best Practices (Do's)

✅ **Use Context for truly global state**
```jsx
// Good - Theme, auth, language settings
<ThemeProvider>
    <AuthProvider>
        <App />
    </AuthProvider>
</ThemeProvider>
```

✅ **Split Context by update frequency**
```jsx
// Good - Separate state and actions
<StateContext.Provider value={state}>
    <ActionsContext.Provider value={actions}>
        {children}
    </ActionsContext.Provider>
</StateContext.Provider>
```

✅ **Create custom hooks for reusable logic**
```jsx
// Good - Encapsulated, testable, reusable
function useAuth() {
    // Auth logic
    return { user, login, logout };
}
```

✅ **Use React.memo for expensive components**
```jsx
// Good - Prevents unnecessary re-renders
const ExpensiveList = memo(function ExpensiveList({ items }) {
    return items.map(item => <ExpensiveItem key={item.id} {...item} />);
});
```

✅ **Code split by route**
```jsx
// Good - Faster initial load
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));
```

✅ **Use useImperativeHandle sparingly**
```jsx
// Good - Only expose what's needed
useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => setvalue('')
}));
```

## Common Mistakes (Don'ts)

❌ **Don't overuse Context**
```jsx
// Wrong - Context for every small state
<CounterContext.Provider value={count}>
    <TimerContext.Provider value={timer}>
        <ToggleContext.Provider value={toggle}>
            {/* Too many contexts */}
        </ToggleContext.Provider>
    </TimerContext.Provider>
</CounterContext.Provider>

// Better - Use props or local state
function Parent() {
    const [count, setCount] = useState(0);
    return <Child count={count} />;
}
```

❌ **Don't create context without default value check**
```jsx
// Wrong - Can cause errors
const MyContext = createContext();
function useMyContext() {
    return useContext(MyContext);
}

// Correct - Validate context is provided
function useMyContext() {
    const context = useContext(MyContext);
    if (context === undefined) {
        throw new Error('useMyContext must be used within MyProvider');
    }
    return context;
}
```

❌ **Don't mutate state in reducer**
```jsx
// Wrong
function reducer(state, action) {
    state.count++; // Mutation!
    return state;
}

// Correct
function reducer(state, action) {
    return { ...state, count: state.count + 1 };
}
```

❌ **Don't forget to memoize Context values**
```jsx
// Wrong - New object every render
function Provider({ children }) {
    const [state, setState] = useState(initialState);
    return (
        <Context.Provider value={{ state, setState }}>
            {children}
        </Context.Provider>
    );
}

// Correct
function Provider({ children }) {
    const [state, setState] = useState(initialState);
    const value = useMemo(() => ({ state, setState }), [state]);
    return <Context.Provider value={value}>{children}</Context.Provider>;
}
```

❌ **Don't use index as key in dynamic lists**
```jsx
// Wrong - Causes bugs when items reorder
{items.map((item, index) => <Item key={index} {...item} />)}

// Correct - Use stable identifier
{items.map(item => <Item key={item.id} {...item} />)}
```

❌ **Don't lazy load components that are immediately needed**
```jsx
// Wrong - Header is always needed
const Header = lazy(() => import('./Header'));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <Header /> {/* Always visible, no need to lazy load */}
        </Suspense>
    );
}

// Correct - Load header normally
import Header from './Header';
```

❌ **Don't forward refs to function components without forwardRef**
```jsx
// Wrong - Won't work
function Input({ ref }) {
    return <input ref={ref} />;
}

// Correct
const Input = forwardRef((props, ref) => {
    return <input ref={ref} {...props} />;
});
```

❌ **Don't create Portals without cleanup**
```jsx
// Wrong - Creates multiple portal roots
function Modal() {
    return createPortal(
        <div>Modal</div>,
        document.createElement('div') // New div every time!
    );
}

// Correct - Use existing element
function Modal() {
    return createPortal(
        <div>Modal</div>,
        document.getElementById('modal-root')
    );
}
```
