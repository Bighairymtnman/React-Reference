# React Basics Quick Reference

## Table of Contents
- [Project Setup](#project-setup)
- [React Hooks](#react-hooks)
- [Component Lifecycle](#component-lifecycle)
- [Context API](#context-api)
- [Routing](#routing)
- [Data Fetching](#data-fetching)

## Project Setup
```bash
# Create new React project
npx create-react-app my-app

# Start development server
cd my-app
npm start
```

```jsx
// Basic App Structure
// src/App.js - Main application component
import React from 'react';

function App() {
    return (
        <div className="app">
            <h1>My React App</h1>
        </div>
    );
}

export default App;

// src/index.js - Entry point
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

## React Hooks
```jsx
// useState - Manage component state
function Counter() {
    // State declaration: [value, setValue]
    const [count, setCount] = useState(0);
    
    return (
        <button onClick={() => setCount(count + 1)}>
            Count: {count}
        </button>
    );
}

// useEffect - Handle side effects
function UserStatus({ id }) {
    const [isOnline, setIsOnline] = useState(false);

    useEffect(() => {
        // Runs on mount and when id changes
        checkUserStatus(id);
        
        return () => {
            // Cleanup function runs before next effect
            cleanupStatusCheck(id);
        };
    }, [id]); // Dependency array

    return <div>User is: {isOnline ? 'Online' : 'Offline'}</div>;
}

// useRef - Persist values between renders
function TextInput() {
    // Create a ref to store DOM element
    const inputRef = useRef(null);

    const focusInput = () => {
        inputRef.current.focus();
    };

    return (
        <>
            <input ref={inputRef} />
            <button onClick={focusInput}>Focus Input</button>
        </>
    );
}
```

## Component Lifecycle
```jsx
// Modern lifecycle using hooks
function LifecycleDemo() {
    // Component Mount
    useEffect(() => {
        console.log('Component mounted');
        
        // Component Unmount
        return () => {
            console.log('Component will unmount');
        };
    }, []); // Empty array = run once on mount

    // Component Update
    useEffect(() => {
        console.log('Data updated');
    }, [/* dependencies */]);

    return <div>Lifecycle Demo</div>;
}

// Error Boundary
class ErrorBoundary extends React.Component {
    state = { hasError: false };

    static getDerivedStateFromError(error) {
        return { hasError: true };
    }

    componentDidCatch(error, info) {
        console.log('Error caught:', error, info);
    }

    render() {
        if (this.state.hasError) {
            return <h1>Something went wrong.</h1>;
        }
        return this.props.children;
    }
}
```

## Context API
```jsx
// Create context for global state
const ThemeContext = React.createContext('light');

// Provider component
function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');

    return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

// Consumer component using useContext
function ThemedButton() {
    const { theme, setTheme } = useContext(ThemeContext);

    return (
        <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
            Current theme: {theme}
        </button>
    );
}

// Usage
function App() {
    return (
        <ThemeProvider>
            <ThemedButton />
        </ThemeProvider>
    );
}
```

## Routing
```jsx
// Basic routing setup using react-router-dom
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Navigation component
function Navigation() {
    return (
        <nav>
            <Link to="/">Home</Link>
            <Link to="/about">About</Link>
            <Link to="/users">Users</Link>
        </nav>
    );
}

// Route setup
function App() {
    return (
        <BrowserRouter>
            <Navigation />
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                <Route path="/users/:id" element={<UserProfile />} />
            </Routes>
        </BrowserRouter>
    );
}

// Route with parameters
function UserProfile() {
    // Get route parameters
    const { id } = useParams();
    return <div>User Profile: {id}</div>;
}
```

## Data Fetching
```jsx
// Fetch data with useEffect
function UserList() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        // Fetch users when component mounts
        const fetchUsers = async () => {
            try {
                const response = await fetch('https://api.example.com/users');
                const data = await response.json();
                setUsers(data);
            } catch (error) {
                console.error('Error fetching users:', error);
            } finally {
                setLoading(false);
            }
        };

        fetchUsers();
    }, []);

    if (loading) return <div>Loading...</div>;
    
    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}

// Custom fetch hook
function useDataFetching(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        const fetchData = async () => {
            try {
                const response = await fetch(url);
                const result = await response.json();
                setData(result);
            } catch (err) {
                setError(err);
            } finally {
                setLoading(false);
            }
        };

        fetchData();
    }, [url]);

    return { data, loading, error };
}

// Usage of custom hook
function UserData({ userId }) {
    const { data, loading, error } = useDataFetching(
        `https://api.example.com/users/${userId}`
    );

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;
    
    return <div>User: {data.name}</div>;
}
```
