# Advanced React Concepts

## Table of Contents
- [Performance Optimization](#performance-optimization)
- [Custom Hooks](#custom-hooks)
- [Higher-Order Components](#higher-order-components)
- [Render Props](#render-props)
- [Code Splitting](#code-splitting)
- [Advanced Patterns](#advanced-patterns)
- [Testing](#testing)

## Performance Optimization
```jsx
// React.memo prevents re-renders when props haven't changed
// Useful for optimizing performance in child components
const MemoizedComponent = React.memo(function MyComponent(props) {
    // Only re-renders if props.value changes
    return <div>{props.value}</div>;
});

// useMemo caches expensive calculations between renders
// Great for heavy computations or complex data transformations
function ExpensiveCalculation({ data }) {
    const memoizedValue = useMemo(() => {
        // This calculation only runs when data changes
        return data.reduce((acc, item) => acc + item, 0);
    }, [data]); // Dependency array - controls when to recalculate

    return <div>Result: {memoizedValue}</div>;
}

// useCallback memoizes functions to maintain reference equality
// Important when passing callbacks to optimized child components
function ParentComponent() {
    const [count, setCount] = useState(0);

    // This function maintains the same reference between renders
    const handleClick = useCallback(() => {
        setCount(c => c + 1);
    }, []); // Empty array means function never changes

    return <ChildComponent onClick={handleClick} />;
}
```

## Custom Hooks
```jsx
// Custom hooks let you extract component logic into reusable functions
// This hook tracks window size changes
function useWindowSize() {
    // Initialize state with current window dimensions
    const [size, setSize] = useState({
        width: window.innerWidth,
        height: window.innerHeight
    });

    useEffect(() => {
        // Handler to call on window resize
        const handleResize = () => {
            setSize({
                width: window.innerWidth,
                height: window.innerHeight
            });
        };

        // Subscribe to window resize events
        window.addEventListener('resize', handleResize);
        
        // Cleanup function removes event listener
        return () => window.removeEventListener('resize', handleResize);
    }, []); // Empty array means effect runs once on mount

    return size; // Return current dimensions
}

// Custom hook for form handling
// Simplifies form state management and input handling
function useForm(initialState = {}) {
    const [values, setValues] = useState(initialState);

    // Generic change handler for any form input
    const handleChange = (event) => {
        const { name, value } = event.target;
        setValues(prev => ({ ...prev, [name]: value }));
    };

    // Reset form to initial state
    const resetForm = () => setValues(initialState);

    return { values, handleChange, resetForm };
}

// Example of using the custom form hook
function SignupForm() {
    // Hook provides form state and handlers
    const { values, handleChange, resetForm } = useForm({
        username: '',
        email: '',
        password: ''
    });

    return (
        <form>
            <input
                name="username"
                value={values.username}
                onChange={handleChange}
            />
            {/* Other form fields */}
        </form>
    );
}
```

## Higher-Order Components (HOCs)
```jsx
// HOCs are functions that take a component and return an enhanced version
// This HOC adds loading state handling
function withLoader(WrappedComponent) {
    return function WithLoaderComponent({ isLoading, ...props }) {
        // Show loading indicator when isLoading is true
        if (isLoading) {
            return <div>Loading...</div>;
        }
        // Otherwise, render the wrapped component with its props
        return <WrappedComponent {...props} />;
    };
}

// HOC for protecting routes that require authentication
function withAuth(WrappedComponent) {
    return function WithAuthComponent(props) {
        const isAuthenticated = useAuth(); // Custom authentication hook

        // Redirect to login if not authenticated
        if (!isAuthenticated) {
            return <Navigate to="/login" />;
        }
        // Render component if authenticated
        return <WrappedComponent {...props} />;
    };
}

// Compose multiple HOCs together
const ProtectedComponent = withAuth(withLoader(UserDashboard));
```

## Render Props
```jsx
// Render props pattern shares code between components
// This component tracks mouse position and shares it
class MouseTracker extends React.Component {
    state = { x: 0, y: 0 };

    // Update state with current mouse position
    handleMouseMove = (event) => {
        this.setState({
            x: event.clientX,
            y: event.clientY
        });
    };

    render() {
        // Call the render prop with current state
        return (
            <div onMouseMove={this.handleMouseMove}>
                {this.props.render(this.state)}
            </div>
        );
    }
}

// Example usage showing mouse coordinates
<MouseTracker
    render={({ x, y }) => (
        <div>Mouse position: {x}, {y}</div>
    )}
/>
```

## Code Splitting
```jsx
// Code splitting helps reduce bundle size
// Lazy loading loads components only when needed
const LazyComponent = React.lazy(() => import('./LazyComponent'));

// Suspense provides a loading state while component loads
function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <LazyComponent />
        </Suspense>
    );
}

// Route-based code splitting
// Each route loads its component separately
const Dashboard = React.lazy(() => import('./Dashboard'));
const Settings = React.lazy(() => import('./Settings'));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <Routes>
                <Route path="/dashboard" element={<Dashboard />} />
                <Route path="/settings" element={<Settings />} />
            </Routes>
        </Suspense>
    );
}
```

## Advanced Patterns
```jsx
// Compound Components Pattern
// Creates related components that work together
function Menu({ children }) {
    const [activeIndex, setActiveIndex] = useState(0);
    
    // Clone and enhance child elements with additional props
    return React.Children.map(children, (child, index) => {
        return React.cloneElement(child, {
            isActive: index === activeIndex,
            onActivate: () => setActiveIndex(index)
        });
    });
}

// Subcomponent for Menu items
Menu.Item = function MenuItem({ isActive, onActivate, children }) {
    return (
        <div className={isActive ? 'active' : ''} onClick={onActivate}>
            {children}
        </div>
    );
};

// State Reducer Pattern
// Allows customization of state logic
function useCounter({ initialState = 0, reducer }) {
    const [count, dispatch] = useReducer(reducer, initialState);

    const increment = () => dispatch({ type: 'INCREMENT' });
    const decrement = () => dispatch({ type: 'DECREMENT' });

    return { count, increment, decrement };
}

// Example implementation with custom state logic
function Counter() {
    // Custom reducer for counter behavior
    const reducer = (state, action) => {
        switch (action.type) {
            case 'INCREMENT':
                return state + 1;
            case 'DECREMENT':
                return state > 0 ? state - 1 : 0; // Prevent negative values
            default:
                return state;
        }
    };

    const { count, increment, decrement } = useCounter({
        initialState: 0,
        reducer
    });

    return (
        <div>
            <button onClick={decrement}>-</button>
            <span>{count}</span>
            <button onClick={increment}>+</button>
        </div>
    );
}
```

## Testing
```jsx
// React Testing Library focuses on testing user interactions
import { render, fireEvent, screen } from '@testing-library/react';

// Basic component rendering test
test('renders counter', () => {
    render(<Counter />);
    // Check if element exists in document
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
});

// Testing user interactions
test('increments counter', () => {
    render(<Counter />);
    // Simulate click event
    fireEvent.click(screen.getByText('+'));
    // Verify counter increased
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

// Testing asynchronous operations
test('loads user data', async () => {
    render(<UserProfile userId="123" />);
    // Check loading state
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    // Wait for data to load
    await screen.findByText('User: John');
    // Verify loaded content
    expect(screen.getByText('User: John')).toBeInTheDocument();
});

// Testing with mock functions
test('calls onSubmit with form data', () => {
    // Create mock function
    const handleSubmit = jest.fn();
    render(<Form onSubmit={handleSubmit} />);
    
    // Simulate form input
    fireEvent.change(screen.getByLabelText('Username'), {
        target: { value: 'testuser' }
    });
    // Simulate form submission
    fireEvent.click(screen.getByText('Submit'));
    
    // Verify mock function was called with correct data
    expect(handleSubmit).toHaveBeenCalledWith({
        username: 'testuser'
    });
});
```
