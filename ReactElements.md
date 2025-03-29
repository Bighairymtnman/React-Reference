# React Elements Quick Reference

## Table of Contents
- [JSX Basics](#jsx-basics)
- [Components](#components)
- [Props](#props)
- [State](#state)
- [Events](#events)
- [Lists & Keys](#lists--keys)
- [Conditional Rendering](#conditional-rendering)
- [Forms](#forms)

## JSX Basics
```jsx
// JSX is React's syntax extension for JavaScript
// It allows you to write HTML-like code in JavaScript

// Basic JSX - Simple element creation
const element = <h1>Hello, React!</h1>;

// JSX with JavaScript Expressions - Use curly braces {} to embed JavaScript
const name = 'User';
const greeting = <h1>Hello, {name}!</h1>;  // Will display: Hello, User!

// JSX Attributes - Use className instead of class for CSS classes
const button = <button className="btn">Click me</button>;

// Multi-line JSX - Must be wrapped in parentheses
// This creates a nested structure of elements
const container = (
    <div>
        <h1>Title</h1>
        <p>Content</p>
    </div>
);
```

## Components
```jsx
// Components are reusable pieces of UI
// They can be created in several ways:

// Function Component - The simplest way to create a component
// Takes props as an argument and returns React elements
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}

// Arrow Function Component - Modern alternative syntax
// Functionally identical to regular function components
const Welcome = (props) => {
    return <h1>Hello, {props.name}</h1>;
};

// Class Component - Traditional way to create components
// Provides additional features like lifecycle methods
class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}

// Using Components - Components can be nested and reused
// Each component instance can receive different props
const element = (
    <div>
        <Welcome name="Alice" />  {/* Renders: Hello, Alice */}
        <Welcome name="Bob" />    {/* Renders: Hello, Bob */}
    </div>
);
```

## Props
```jsx
// Props (properties) are how components receive data
// They are read-only and help make components reusable

// Basic Props - Passing data to a component
function Greeting(props) {
    return <h1>Hello, {props.name}!</h1>;
}

// Props with Destructuring - Cleaner way to access props
// Directly extract the values you need
function Greeting({ name, age }) {
    return (
        <div>
            <h1>Hello, {name}!</h1>
            <p>Age: {age}</p>
        </div>
    );
}

// Default Props - Fallback values if props aren't provided
function Greeting({ name = 'Guest' }) {
    return <h1>Hello, {name}!</h1>;  // Uses 'Guest' if name prop is missing
}

// Children Props - Special prop for nested content
// Allows components to wrap other elements
function Container({ children }) {
    return <div className="container">{children}</div>;
}
```

## State
```jsx
// State manages data that can change over time
// When state updates, React re-renders the component

// useState Hook - Most common way to manage state
// Returns current state and a function to update it
function Counter() {
    const [count, setCount] = useState(0);  // [currentValue, updateFunction]
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
        </div>
    );
}

// Multiple State Values - Components can have multiple states
// Each useState call manages an independent piece of state
function UserForm() {
    const [name, setName] = useState('');    // State for name input
    const [age, setAge] = useState(0);       // State for age input
    
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
        </form>
    );
}
```

## Events
```jsx
// React events handle user interactions
// They use camelCase naming (onClick, onChange, etc.)

// Event Handling - Basic click event
function Button() {
    const handleClick = () => {
        console.log('Button clicked!');
    };
    
    return <button onClick={handleClick}>Click Me</button>;
}

// Event with Parameters - Passing additional data to event handlers
function Button({ id }) {
    const handleClick = (event, id) => {
        console.log(`Button ${id} clicked!`, event);
        // event contains information about the event
    };
    
    return (
        <button onClick={(e) => handleClick(e, id)}>
            Click Me
        </button>
    );
}

// Form Events - Handling form submissions
function Form() {
    const handleSubmit = (event) => {
        event.preventDefault();  // Prevents page reload
        // Form handling logic here
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <button type="submit">Submit</button>
        </form>
    );
}
```

## Lists & Keys
```jsx
// React needs keys to efficiently update lists
// Keys help React identify which items changed

// Rendering Lists - Basic list rendering with keys
function ItemList({ items }) {
    return (
        <ul>
            {items.map(item => (
                // key prop helps React track each item
                <li key={item.id}>{item.name}</li>
            ))}
        </ul>
    );
}

// Complex List Items - Rendering lists with more structure
function UserList({ users }) {
    return (
        <div>
            {users.map(user => (
                // Each top-level element in map needs a key
                <div key={user.id} className="user-card">
                    <h3>{user.name}</h3>
                    <p>{user.email}</p>
                </div>
            ))}
        </div>
    );
}
```

## Conditional Rendering
```jsx
// Conditional rendering shows different content based on conditions

// If Statement - Basic conditional rendering
function Greeting({ isLoggedIn }) {
    if (isLoggedIn) {
        return <h1>Welcome back!</h1>;
    }
    return <h1>Please log in</h1>;
}

// Ternary Operator - Inline conditional rendering
// Shorter syntax for simple conditions
function Greeting({ isLoggedIn }) {
    return (
        <div>
            {isLoggedIn ? (
                <h1>Welcome back!</h1>
            ) : (
                <h1>Please log in</h1>
            )}
        </div>
    );
}

// Logical && Operator - Conditional rendering for single outcomes
// Shows element only if condition is true
function Notification({ message }) {
    return (
        <div>
            {message && <p>{message}</p>}  // Shows message only if it exists
        </div>
    );
}
```

## Forms
```jsx
// React forms use controlled components to manage form data
// Form elements are controlled by React state

// Controlled Component - React manages form data
function LoginForm() {
    // State for each form field
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();  // Prevent default form submission
        console.log('Submit:', { username, password });
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                value={username}  // Value controlled by state
                onChange={(e) => setUsername(e.target.value)}  // Update state on change
            />
            <input
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
            />
            <button type="submit">Login</button>
        </form>
    );
}

// Multiple Input Handler - Managing multiple inputs efficiently
function SignupForm() {
    // Single state object for all form fields
    const [values, setValues] = useState({
        username: '',
        email: '',
        password: ''
    });

    // Generic handler for all inputs
    const handleChange = (e) => {
        const { name, value } = e.target;
        setValues(prev => ({
            ...prev,           // Spread existing values
            [name]: value      // Update only changed field
        }));
    };

    return (
        <form>
            <input
                name="username"  // name attribute matches state property
                value={values.username}
                onChange={handleChange}
            />
            <input
                name="email"
                type="email"
                value={values.email}
                onChange={handleChange}
            />
            <input
                name="password"
                type="password"
                value={values.password}
                onChange={handleChange}
            />
        </form>
    );
}
```
