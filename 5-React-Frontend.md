# React & Frontend

## Question 101: Virtual DOM

**Answer:**
React's Virtual DOM is a lightweight JavaScript representation of the actual DOM. When state changes, React:
1. Creates new Virtual DOM tree
2. Diffs against old Virtual DOM (reconciliation)
3. Updates only changed elements in real DOM

**Benefits:**
- Batches DOM updates (faster)
- Minimizes reflows/repaints
- Abstraction layer for different platforms (React Native)

**Performance:**
```javascript
// Naive: 1000 direct DOM updates
for (let i = 0; i < 1000; i++) {
    document.getElementById('list').appendChild(createItem(i));
    // Reflow/repaint 1000 times
}

// React: Batches updates
state.items = Array.from({length: 1000}, (_, i) => i);
// Virtual DOM: 1 render pass
// Real DOM: 1 update
```

---

## Question 102: Reconciliation

**Answer:**
React's diffing algorithm compares old and new Virtual DOM trees:

1. **Type checking**: Different component types = replace
2. **Key matching**: Lists use keys for identity
3. **Props/state**: Only update changed props

**Example:**
```jsx
// Old: <div><p>Hello</p></div>
// New: <div><p>Hello</p><span>World</span></div>
// React identifies: Keep <p>, Add <span> (efficient)

// Keys for lists:
const items = data.map(item => <li key={item.id}>{item.name}</li>);
// Without key: reuses DOM nodes, may reorder wrongly
```

---

## Question 103: Hooks rules

**Answer:**
1. **Only call in React components** - Not regular functions
2. **Only call at top level** - Not in loops/conditions
3. **Cannot call in event handlers** - Hook state must be stable

**Why?**
React identifies hooks by call order:
```javascript
// Wrong - hook in condition
if (condition) {
    const [state, setState] = useState(0); // Order changes!
}

// Correct
const [state, setState] = useState(0);
if (condition) {
    // Use state
}
```

---

## Question 104: useEffect pitfalls

**Answer:**
```javascript
// ❌ Infinite loop - no dependency array
useEffect(() => {
    setState(count + 1);
});

// ❌ Stale closure
useEffect(() => {
    const timer = setInterval(() => {
        console.log(count); // Always logs initial count
    }, 1000);
}, []); // Missing count dependency

// ✅ Correct
useEffect(() => {
    const timer = setInterval(() => {
        console.log(count);
    }, 1000);
}, [count]);

// ❌ Memory leak
useEffect(() => {
    const subscription = subscribe();
    // No cleanup!
});

// ✅ Cleanup
useEffect(() => {
    const subscription = subscribe();
    return () => subscription.unsubscribe();
}, []);
```

---

## Question 105: useMemo/useCallback trade-offs

**Answer:**
```javascript
// useMemo - Memoize computation
const expensiveValue = useMemo(() => {
    return computeExpensiveValue(a, b);
}, [a, b]); // Only recompute if a or b changes

// useCallback - Memoize function
const memoCallback = useCallback(
    () => { doSomething(a, b); },
    [a, b]
);

// When to use:
// - If you pass to React.memo child
// - If used as dependency in another hook
// - NOT for micro-optimizations
```

---

## Question 106: Context vs Redux

**Answer:**
**Context** - Local state sharing:
```javascript
const ThemeContext = createContext();
<ThemeProvider value={theme}>
    <App /> // All children access theme
</ThemeProvider>
```

**Redux** - Global state management:
```javascript
const store = createStore(rootReducer);
<Provider store={store}>
    <App />
</Provider>

// Dispatch actions
dispatch(setTheme('dark'));
```

**When to use:**
- **Context**: Theme, auth, user settings
- **Redux**: Complex app, many state updates, time-travel debugging

---

## Question 107: Redux Toolkit

**Answer:**
Modern Redux with less boilerplate:
```javascript
import { createSlice, configureStore } from '@reduxjs/toolkit';

const userSlice = createSlice({
    name: 'user',
    initialState: { name: '', loading: false },
    reducers: {
        setUser: (state, action) => {
            state.name = action.payload; // Immer handles immutability
        },
        setLoading: (state, action) => {
            state.loading = action.payload;
        }
    },
    extraReducers: (builder) => {
        builder.addCase(fetchUser.fulfilled, (state, action) => {
            state.user = action.payload;
        });
    }
});

const store = configureStore({
    reducer: { user: userSlice.reducer }
});
```

---

## Question 108: React Query

**Answer:**
Handles server state (data from API):
```javascript
import { useQuery, useMutation } from '@tanstack/react-query';

// Fetch with caching
const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5000, // Cache for 5s
    retry: 2 // Retry failed requests
});

// Mutation with cache update
const { mutate: updateUser } = useMutation({
    mutationFn: (newUser) => api.updateUser(newUser),
    onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: ['user'] });
    }
});

return user ? <div>{user.name}</div> : <div>Loading...</div>;
```

---

## Question 109: SSR vs CSR vs SSG

**Answer:**
**CSR (Client-Side Rendering)**:
- Server sends empty HTML, JS renders on client
- Slower initial load, interactive after loaded
- Best for: Apps, dynamic content

**SSR (Server-Side Rendering)**:
- Server sends fully rendered HTML
- Fast initial load, interactive
- Best for: SEO, initial paint speed

**SSG (Static Site Generation)**:
- Build-time rendering
- HTML pre-generated, served instantly
- Best for: Blogs, marketing sites

```javascript
// Next.js examples
// CSR
export default function Home() { ... }

// SSR
export async function getServerProps() {
    const data = await fetchData();
    return { props: { data } };
}

// SSG
export async function getStaticProps() {
    const data = await fetchData();
    return { props: { data }, revalidate: 3600 };
}
```

---

## Question 110: Code splitting

**Answer:**
Load only needed code per route:
```javascript
import { lazy, Suspense } from 'react';

// Lazy load component
const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <AdminPanel />
        </Suspense>
    );
}
// AdminPanel code only loaded when needed
```

---

## Question 111: Lazy loading

**Answer:**
Defer loading resources:
```javascript
// Images
<img loading="lazy" src="image.jpg" />

// Components (already covered in code splitting)

// Virtual scrolling (large lists)
import { FixedSizeList } from 'react-window';
<FixedSizeList
    height={600}
    itemCount={10000}
    itemSize={35}
>
    {Row}
</FixedSizeList>
```

---

## Question 112: XSS prevention

**Answer:**
React escapes JSX by default:
```javascript
// Safe - React escapes HTML
<div>{userInput}</div> // <script> shows as text

// Dangerous - dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// Content Security Policy
// <meta http-equiv="Content-Security-Policy" content="script-src 'self'">
```

---

## Question 113: Token storage

**Answer:**
```javascript
// ❌ localStorage - XSS vulnerable
localStorage.setItem('token', token);

// ✅ HttpOnly cookie (server sets)
// Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict
// Automatically sent in requests, not accessible to JS

// ✅ In-memory (best, lost on refresh)
const [token, setToken] = useState(null);
// Store refresh token in HttpOnly cookie
```

---

## Question 114: Form management

**Answer:**
```javascript
// Controlled inputs
const [email, setEmail] = useState('');
<input value={email} onChange={e => setEmail(e.target.value)} />

// Form library (React Hook Form)
import { useForm } from 'react-hook-form';

const { register, handleSubmit, formState: { errors } } = useForm();

<form onSubmit={handleSubmit(onSubmit)}>
    <input {...register('email', { required: true })} />
    {errors.email && <p>Required</p>}
</form>
```

---

## Question 115: Error boundaries

**Answer:**
Catch component errors:
```javascript
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false };
    }
    
    static getDerivedStateFromError(error) {
        return { hasError: true };
    }
    
    componentDidCatch(error, errorInfo) {
        logToService(error, errorInfo);
    }
    
    render() {
        if (this.state.hasError) {
            return <h1>Something went wrong</h1>;
        }
        return this.props.children;
    }
}

<ErrorBoundary>
    <ComponentThatMayError />
</ErrorBoundary>
```

---

## Question 116: Micro frontends

**Answer:**
Split large frontend into independent modules:
```javascript
// Module Federation (Webpack 5)
// shell/webpack.config.js
new ModuleFederationPlugin({
    name: 'shell',
    remotes: {
        admin: 'admin@http://localhost:3001/remoteEntry.js'
    }
});

// admin/webpack.config.js
new ModuleFederationPlugin({
    name: 'admin',
    filename: 'remoteEntry.js',
    exposes: {
        './AdminPanel': './src/AdminPanel'
    }
});

// shell/src/App.js
const AdminPanel = React.lazy(() => import('admin/AdminPanel'));
```

---

## Question 117: State normalization

**Answer:**
Flatten nested state for easier updates:
```javascript
// ❌ Nested (hard to update)
state = {
    users: {
        1: { id: 1, name: 'John', orders: [1, 2, 3] }
    }
};

// ✅ Normalized
state = {
    users: { 1: { id: 1, name: 'John' } },
    userOrders: { 1: [1, 2, 3] },
    orders: {
        1: { id: 1, total: 100 },
        2: { id: 2, total: 200 }
    }
};

// Selectors
const getUserOrders = (state, userId) =>
    state.userOrders[userId].map(id => state.orders[id]);
```

---

## Question 118: Performance profiling

**Answer:**
```javascript
// React DevTools Profiler
// Measure component render time

// React.ProfilerAPI
<Profiler id="page" onRender={onRenderCallback}>
    <Page />
</Profiler>

// Web Vitals
import { getCLS, getFID, getLCP } from 'web-vitals';
getCLS(console.log); // Cumulative Layout Shift
getFID(console.log); // First Input Delay
getLCP(console.log); // Largest Contentful Paint
```

---

## Question 119: Accessibility essentials

**Answer:**
```javascript
// ARIA labels
<button aria-label="Close menu">✕</button>

// Semantic HTML
<nav>, <main>, <article>, <button> (not <div onClick>)

// Focus management
<input autoFocus />
ref.current?.focus();

// Color contrast
// AA standard: 4.5:1 for text

// Keyboard navigation
// Tab, Enter, Escape should work
```

---

## Question 120: Testing React apps

**Answer:**
```javascript
import { render, screen, fireEvent } from '@testing-library/react';

test('updates name on input change', () => {
    render(<MyComponent />);
    const input = screen.getByRole('textbox');
    fireEvent.change(input, { target: { value: 'John' } });
    expect(input.value).toBe('John');
});

// Test async
test('fetches user', async () => {
    render(<UserProfile userId={1} />);
    const name = await screen.findByText('John');
    expect(name).toBeInTheDocument();
});
```

---

## Question 121: Component composition

**Answer:**
```javascript
// Composition > Inheritance
function ExpandablePanel({ title, children }) {
    return (
        <div>
            <h2>{title}</h2>
            {children}
        </div>
    );
}

// Reusable
<ExpandablePanel title="Settings">
    <div>Content</div>
</ExpandablePanel>
```

---

## Question 122: Custom hooks

**Answer:**
```javascript
function useWindowSize() {
    const [size, setSize] = useState({ width: 0, height: 0 });
    
    useEffect(() => {
        const handler = () => {
            setSize({ width: window.innerWidth, height: window.innerHeight });
        };
        window.addEventListener('resize', handler);
        return () => window.removeEventListener('resize', handler);
    }, []);
    
    return size;
}

// Usage
const { width, height } = useWindowSize();
```

---

## Question 123: WebSocket integration

**Answer:**
```javascript
function useWebSocket(url) {
    const [message, setMessage] = useState(null);
    
    useEffect(() => {
        const ws = new WebSocket(url);
        ws.onmessage = (event) => setMessage(event.data);
        return () => ws.close();
    }, [url]);
    
    return message;
}
```

---

## Question 124: Frontend observability

**Answer:**
```javascript
// Error tracking
import Sentry from "@sentry/react";
Sentry.init({ dsn: "..." });

// Analytics
gtag.event('page_view', { page_path: location.pathname });

// Performance monitoring
const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        console.log('Performance:', entry);
    }
});
observer.observe({ entryTypes: ['navigation', 'resource', 'paint'] });
```

---

## Question 125: Design a scalable React architecture

**Answer:**
```
/src
  /components      # Dumb components
  /containers      # Smart components (hooks)
  /hooks          # Custom hooks
  /services       # API calls
  /store          # Redux/Context
  /utils          # Helpers
  /styles         # Global styles
  /pages          # Route pages
```

**Key Practices:**
- Use hooks over HOCs
- Code split by route
- Normalize state
- Memoize with React.memo
- Use lazy loading
- Separate concerns

