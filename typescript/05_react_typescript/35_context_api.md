# Context API - createContext typing, useContext, context providers, context with reducer

## Introduction

React Context API provides a way to share values across the component tree without manually passing props through every level. TypeScript enhances the Context API by ensuring that the context value has the correct shape, that consumers access only available properties, and that providers supply the required values. From basic createContext typing with default values to complex state management with useReducer, TypeScript makes context usage robust against common runtime errors.

## createContext with Default Value

### What It Is

createContext is a function that creates a context object. It accepts a default value that is used when a component consumes the context without a matching provider ancestor. In TypeScript, the generic type parameter specifies the shape of the context value.

### Why It Is Important

Correctly typing createContext ensures that consumers know exactly what properties are available on the context value. The default value must match the type, which is why common patterns use 
ull as the default with a type that excludes 
ull for the actual value, forcing consumers to handle the unprovided case.

### How It Works Internally

createContext in @types/react is declared as:

`	ypescript
function createContext<T>(defaultValue: T): Context<T>;

interface Context<T> {
  Provider: Provider<T>;
  Consumer: Consumer<T>;
  displayName?: string;
}
`

The type T is used to type the alue prop of Provider and the return value of useContext. The default value must satisfy T, which is why initialization patterns often use type assertions or dual types.

### Syntax

`	ypescript
import React, { createContext } from 'react';

const ThemeContext = createContext('light');

interface UserContextValue {
  user: { name: string; email: string } | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const UserContext = createContext<UserContextValue | null>(null);
`

### Beginner Examples

`	ypescript
type Theme = 'light' | 'dark';
const ThemeContext = createContext<Theme>('light');

const ThemedButton = () => {
  const theme = useContext(ThemeContext);
  return <button className={'btn btn--' + theme}>Themed Button</button>;
};

const App = () => (
  <ThemeContext.Provider value="dark">
    <ThemedButton />
  </ThemeContext.Provider>
);
`

### Intermediate Examples

`	ypescript
interface LanguageContextType {
  locale: string;
  translations: Record<string, string>;
  setLocale: (locale: string) => void;
}

const defaultLanguage: LanguageContextType = {
  locale: 'en',
  translations: { hello: 'Hello', goodbye: 'Goodbye' },
  setLocale: () => {},
};

const LanguageContext = createContext<LanguageContextType>(defaultLanguage);

const Greeting = () => {
  const { translations } = useContext(LanguageContext);
  return <h1>{translations.hello}</h1>;
};

const LanguageProvider = ({ children }: { children: React.ReactNode }) => {
  const [locale, setLocale] = useState('en');
  const translations = getTranslations(locale);
  return (
    <LanguageContext.Provider value={{ locale, translations, setLocale }}>
      {children}
    </LanguageContext.Provider>
  );
};
`

### Advanced Examples

`	ypescript
interface AuthContextType {
  user: { id: string; name: string; roles: string[] };
  isAuthenticated: boolean;
  login: (token: string) => void;
  logout: () => void;
  hasRole: (role: string) => boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}

const UserProfile = () => {
  const { user, hasRole, logout } = useAuth();
  return (
    <div>
      <h2>{user.name}</h2>
      {hasRole('admin') && <AdminPanel />}
      <button onClick={logout}>Logout</button>
    </div>
  );
};
`

### Real-World Use Cases

`	ypescript
interface FeatureFlags {
  newDashboard: boolean;
  darkMode: boolean;
  betaFeatures: boolean;
  exportCSV: boolean;
}

const defaultFlags: FeatureFlags = {
  newDashboard: false, darkMode: false, betaFeatures: false, exportCSV: true,
};

const FeatureFlagContext = createContext<FeatureFlags>(defaultFlags);

const useFeatureFlag = (flag: keyof FeatureFlags): boolean => {
  const flags = useContext(FeatureFlagContext);
  return flags[flag];
};

interface AppConfig {
  apiUrl: string;
  environment: 'development' | 'staging' | 'production';
  version: string;
}

const ConfigContext = createContext<AppConfig | null>(null);

function useConfig(): AppConfig {
  const config = useContext(ConfigContext);
  if (!config) throw new Error('useConfig must be used within a ConfigProvider');
  return config;
}
`

### Common Mistakes

`	ypescript
// MISTAKE: Using a nonsense default value just to satisfy the type
// const Context = createContext<ExpensiveObject>({} as ExpensiveObject); // BAD
// BETTER: createContext<ExpensiveObject | null>(null);

// MISTAKE: Not using the context consumer guard pattern
const BadComponent = () => {
  const ctx = useContext(MyContext);
  return <div>{ctx.someProp}</div>; // Runtime error if no provider!
};
`

### Best Practices

- Default to createContext<T | undefined>(undefined) and provide a custom hook with guard.
- Create a custom hook for each context that includes the null check and throws a descriptive error.
- Provide a meaningful default value when the context always has a provider.
- Use interface for complex context values to allow extension.
- Keep context values focused; avoid putting everything in a single context.

### Performance Considerations

Creating a context is cheap. When the context value changes, all consumers re-render. Use useMemo for the context value to prevent unnecessary re-renders. Split large contexts into smaller, focused contexts to minimize re-render scope.

### Interview Questions

1. How does createContext determine the type of the context value?
2. Why is the default value for createContext often 
ull?
3. How do you create a custom hook that safely accesses context?
4. What happens when you call useContext without a matching Provider?

### Coding Challenges

1. Create a typed ToastContext that allows any component to show toasts with success, error, and info variants.
2. Build a PermissionContext where roles and permissions are typed, with a usePermission hook.

### Related Topics

- useContext hook
- Context providers
- Context with useReducer

## useContext Hook

### What It Is

useContext is a React Hook that subscribes to a context and returns the current context value. It accepts a context object (created by createContext) and returns the current value from the nearest matching Provider above the component in the tree.

### Why It Is Important

useContext eliminates prop drilling, allowing deeply nested components to access shared values. TypeScript ensures that the returned value has the correct type based on the context's generic parameter, enabling autocompletion and preventing access to non-existent properties.

### How It Works Internally

`	ypescript
function useContext<T>(context: Context<T>): T;
`

The return type T is extracted from the Context<T> interface. If the context was created with createContext<string | null>(null), useContext returns string | null.

### Syntax

`	ypescript
import React, { useContext } from 'react';

const ThemeContext = createContext('light');

const Button = () => {
  const theme = useContext(ThemeContext);
  return <button className={'btn-' + theme}>Click</button>;
};
`

### Beginner Examples

`	ypescript
interface CounterContextType {
  count: number;
  increment: () => void;
  decrement: () => void;
}

const CounterContext = createContext<CounterContextType | undefined>(undefined);

const CounterDisplay = () => {
  const context = useContext(CounterContext);
  if (!context) throw new Error('Missing CounterContext Provider');
  return <p>Count: {context.count}</p>;
};

const CounterControls = () => {
  const { increment, decrement } = useContext(CounterContext)!;
  return (
    <div>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
};
`

### Intermediate Examples

`	ypescript
interface NotificationContextType {
  notifications: Notification[];
  unreadCount: number;
  markAsRead: (id: string) => void;
  clearAll: () => void;
}

const NotificationContext = createContext<NotificationContextType | null>(null);

const NotificationBadge = () => {
  const ctx = useContext(NotificationContext);
  if (!ctx) return null;
  return (
    <div className="notification-badge">
      <span>Notifications</span>
      {ctx.unreadCount > 0 && <span className="badge">{ctx.unreadCount}</span>}
    </div>
  );
};
`

### Advanced Examples

`	ypescript
function useScopedContext<T>(
  contexts: Context<T>[],
  selector?: (value: T) => boolean
): T | null {
  for (const context of contexts) {
    const value = useContext(context);
    if (selector ? selector(value) : value !== null && value !== undefined) {
      return value;
    }
  }
  return null;
}

function useContextSelector<T, K>(
  context: Context<T>,
  selector: (value: T) => K
): K {
  const value = useContext(context);
  return selector(value);
}
`

### Real-World Use Cases

`	ypescript
interface BreakpointState {
  isMobile: boolean;
  isTablet: boolean;
  isDesktop: boolean;
  breakpoint: 'mobile' | 'tablet' | 'desktop';
  width: number;
}

const BreakpointContext = createContext<BreakpointState | null>(null);

const useBreakpoint = () => {
  const ctx = useContext(BreakpointContext);
  if (!ctx) throw new Error('useBreakpoint requires BreakpointProvider');
  return ctx;
};

interface RouterContextType {
  currentPath: string;
  params: Record<string, string>;
  navigate: (path: string) => void;
}

const RouterContext = createContext<RouterContextType | undefined>(undefined);

const useRouter = () => {
  const router = useContext(RouterContext);
  if (!router) throw new Error('useRouter requires RouterProvider');
  return router;
};
`

### Common Mistakes

`	ypescript
// MISTAKE: Calling useContext outside of a React component
// function helperFunction() { useContext(MyContext) } // Error

// MISTAKE: Not handling null/undefined context value
const value = useContext(MyContext);
value.doSomething(); // Runtime error if no provider

// MISTAKE: Using useContext inside conditional rendering
// if (condition) { useContext(MyContext) } // Error: Hooks must be called unconditionally
`

### Best Practices

- Always call useContext at the top level of a React function component or custom hook.
- Create a custom hook wrapper for each context with null-checking and error throwing.
- Use non-null assertion (!) only when you are certain a Provider exists above.
- Destructure context values for cleaner access.

### Performance Considerations

Every component that calls useContext re-renders when the context value changes. To optimize, split contexts by responsibility (e.g., separate auth context from theme context). Consider useMemo for the context value object to prevent unnecessary re-renders.

### Interview Questions

1. How does useContext determine what value to return?
2. What happens to a component using useContext when the context value changes?
3. How can you optimize components that use useContext to prevent excessive re-renders?

### Coding Challenges

1. Create a useTheme hook that returns the current theme and a 	oggleTheme function from a typed context.
2. Build a NotificationCenter using context where components can showNotification with a message and type.

### Related Topics

- createContext
- Context providers
- React.memo and context

## Context Provider Typing

### What It Is

Context providers are React components that supply a value to the context tree. Typing context providers involves ensuring that the alue prop matches the context's type and that the provider component itself is correctly structured.

### Why It Is Important

Typing the provider component ensures that the context value is always in the correct shape, preventing consumers from receiving malformed data. It also documents what the provider requires and provides intellisense when creating provider components.

### How It Works Internally

The Context.Provider component has a alue prop typed as T from Context<T>. TypeScript checks that the value passed to the provider matches this type at compile time.

### Syntax

`	ypescript
interface ThemeProviderProps {
  children: React.ReactNode;
  initialTheme?: Theme;
}

const ThemeProvider = ({ children, initialTheme = 'light' }: ThemeProviderProps) => {
  const [theme, setTheme] = useState<Theme>(initialTheme);
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
`

### Beginner Examples

`	ypescript
type User = { id: string; name: string } | null;

interface UserProviderProps {
  children: React.ReactNode;
}

const UserProvider = ({ children }: UserProviderProps) => {
  const [user, setUser] = useState<User>(null);

  const login = async (email: string, password: string) => {
    const userData = await api.login(email, password);
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
};
`

### Intermediate Examples

`	ypescript
interface Config {
  apiUrl: string;
  environment: string;
}

interface ConfigProviderProps {
  children: React.ReactNode;
  config: Config;
}

const ConfigProvider = ({ children, config }: ConfigProviderProps) => {
  const value = useMemo(() => config, [config]);
  return (
    <ConfigContext.Provider value={value}>
      {children}
    </ConfigContext.Provider>
  );
};
`

### Advanced Examples

`	ypescript
// Generic provider
interface ProviderProps<T> {
  children: React.ReactNode;
  initialData?: T;
}

function GenericDataProvider<T>({ children, initialData }: ProviderProps<T>) {
  const [data, setData] = useState<T | null>(initialData ?? null);
  const updateData = useCallback((newData: T) => setData(newData), []);

  return (
    <DataContext.Provider value={{ data, updateData }}>
      {children}
    </DataContext.Provider>
  );
}

// Provider with reducer integration
interface ProviderWithReducerProps {
  children: React.ReactNode;
}

const AppStateProvider = ({ children }: ProviderWithReducerProps) => {
  const [state, dispatch] = useReducer(appReducer, initialState);
  const value = useMemo(() => ({ state, dispatch }), [state]);

  return (
    <AppStateContext.Provider value={value}>
      {children}
    </AppStateContext.Provider>
  );
};
`

### Real-World Use Cases

`	ypescript
// Composed providers
const AppProviders = ({ children }: { children: React.ReactNode }) => {
  return (
    <ThemeProvider>
      <AuthProvider>
        <ConfigProvider>
          <NotificationProvider>
            <RouterProvider>
              {children}
            </RouterProvider>
          </NotificationProvider>
        </ConfigProvider>
      </AuthProvider>
    </ThemeProvider>
  );
};
`

### Common Mistakes

`	ypescript
// MISTAKE: Creating a new value object on every render without useMemo
const BadProvider = ({ children }: { children: React.ReactNode }) => {
  return (
    <MyContext.Provider value={{ data, updateData }}>
      {children}
    </MyContext.Provider>
  ); // Every render creates a new object -> all consumers re-render
};

// CORRECT: Use useMemo
const GoodProvider = ({ children }: { children: React.ReactNode }) => {
  const value = useMemo(() => ({ data, updateData }), [data]);
  return <MyContext.Provider value={value}>{children}</MyContext.Provider>;
};
`

### Best Practices

- Always memoize the context value object with useMemo.
- Provide a named export for the provider component.
- Include the provider in a custom hook's documentation.
- Keep provider components focused on a single context concern.

### Performance Considerations

The most critical performance consideration is value memoization. Every re-render of the provider creates a new value object reference, causing all consumers to re-render even if the actual data hasn't changed. Always use useMemo for object context values.

### Interview Questions

1. How do you prevent unnecessary re-renders in context consumers?
2. What happens if you don't memoize the context value in a provider?
3. Why would you use multiple small providers instead of one large provider?

### Coding Challenges

1. Create a StoreProvider that uses useReducer and provides both state and dispatch via context.
2. Build a WebSocketProvider that manages a WebSocket connection and provides messages via context.

### Related Topics

- useMemo for context values
- Provider composition patterns
- State management with context

## createContext with useReducer

### What It Is

Combining createContext with useReducer provides a lightweight state management solution that resembles Redux but uses only built-in React APIs. The context provides both state and dispatch, allowing any consumer to read state or dispatch actions.

### Why It Is Important

This pattern provides a predictable state container with reducer-based logic (like Redux) without external dependencies. TypeScript ensures that dispatch actions are correctly typed and that state access is type-safe across the entire component tree.

### How It Works Internally

The provider component uses useReducer internally and passes the state and dispatch to the context value. Consumers use the context to access state and dispatch typed actions.

### Syntax

`	ypescript
type Action = { type: 'increment' } | { type: 'decrement' } | { type: 'reset' };
type State = { count: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    case 'reset': return { count: 0 };
  }
}

const CounterContext = createContext<{ state: State; dispatch: React.Dispatch<Action> } | undefined>(undefined);

const CounterProvider = ({ children }: { children: React.ReactNode }) => {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  const value = useMemo(() => ({ state, dispatch }), [state]);
  return <CounterContext.Provider value={value}>{children}</CounterContext.Provider>;
};
`

### Beginner Examples

`	ypescript
type TodoAction =
  | { type: 'add'; payload: string }
  | { type: 'toggle'; payload: number }
  | { type: 'remove'; payload: number };

type TodoState = { todos: { id: number; text: string; done: boolean }[] };

function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'add':
      return { todos: [...state.todos, { id: Date.now(), text: action.payload, done: false }] };
    case 'toggle':
      return { todos: state.todos.map(t => t.id === action.payload ? { ...t, done: !t.done } : t) };
    case 'remove':
      return { todos: state.todos.filter(t => t.id !== action.payload) };
  }
}

const TodoContext = createContext<{ state: TodoState; dispatch: React.Dispatch<TodoAction> } | undefined>(undefined);

const TodoProvider = ({ children }: { children: React.ReactNode }) => {
  const [state, dispatch] = useReducer(todoReducer, { todos: [] });
  const value = useMemo(() => ({ state, dispatch }), [state]);
  return <TodoContext.Provider value={value}>{children}</TodoContext.Provider>;
};
`

### Intermediate Examples

`	ypescript
interface AuthState {
  user: { id: string; name: string; email: string } | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: { user: AuthState['user']; token: string } }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'CLEAR_ERROR' };

const initialAuthState: AuthState = {
  user: null,
  token: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
};

function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { user: action.payload.user, token: action.payload.token, isAuthenticated: true, isLoading: false, error: null };
    case 'LOGIN_FAILURE':
      return { ...initialAuthState, error: action.payload };
    case 'LOGOUT':
      return { ...initialAuthState };
    case 'CLEAR_ERROR':
      return { ...state, error: null };
  }
}

const AuthContext = createContext<{ state: AuthState; dispatch: React.Dispatch<AuthAction> } | undefined>(undefined);

const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [state, dispatch] = useReducer(authReducer, initialAuthState);
  const value = useMemo(() => ({ state, dispatch }), [state]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
`

### Advanced Examples

`	ypescript
// Generic context with reducer
interface StoreContextType<S, A> {
  state: S;
  dispatch: React.Dispatch<A>;
}

function createStoreContext<S, A>() {
  const StoreContext = createContext<StoreContextType<S, A> | undefined>(undefined);

  const Provider = ({ children, reducer, initialState }: {
    children: React.ReactNode;
    reducer: React.Reducer<S, A>;
    initialState: S;
  }) => {
    const [state, dispatch] = useReducer(reducer, initialState);
    const value = useMemo(() => ({ state, dispatch }), [state]);
    return <StoreContext.Provider value={value}>{children}</StoreContext.Provider>;
  };

  function useStore(): StoreContextType<S, A> {
    const context = useContext(StoreContext);
    if (!context) throw new Error('useStore must be used within StoreProvider');
    return context;
  }

  return { Provider, useStore, StoreContext };
}

// Usage
interface CartItem { id: string; name: string; price: number; quantity: number; }
type CartAction =
  | { type: 'add'; payload: CartItem }
  | { type: 'remove'; payload: string }
  | { type: 'clear' };
type CartState = { items: CartItem[]; total: number };

const {
  Provider: CartProvider,
  useStore: useCart,
} = createStoreContext<CartState, CartAction>();
`

### Real-World Use Cases

`	ypescript
// Theme with persisted state
type ThemeAction = { type: 'set'; payload: 'light' | 'dark' } | { type: 'toggle' };
type ThemeState = { mode: 'light' | 'dark' };

function themeReducer(state: ThemeState, action: ThemeAction): ThemeState {
  switch (action.type) {
    case 'set': return { mode: action.payload };
    case 'toggle': return { mode: state.mode === 'light' ? 'dark' : 'light' };
  }
}

const ThemeContext = createContext<{ state: ThemeState; dispatch: React.Dispatch<ThemeAction> } | undefined>(undefined);

const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  const [state, dispatch] = useReducer(themeReducer, { mode: 'light' });

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', state.mode);
  }, [state.mode]);

  const value = useMemo(() => ({ state, dispatch }), [state]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};
`

### Common Mistakes

`	ypescript
// MISTAKE: Not memoizing the context value
// const value = { state, dispatch }; // New object every render

// MISTAKE: Exposing dispatch directly without action constraints
// Action types should be restricted, not any

// MISTAKE: Putting too much state in a single reducer
// Split into domain-specific reducers with separate contexts
`

### Best Practices

- Always memoize the context value with useMemo.
- Create custom hooks (useYourContext) that include the null check and throw meaningful errors.
- Use discriminated unions for action types for exhaustive checking.
- Split state into multiple contexts based on update frequency to minimize re-renders.
- Consider using a generic createStoreContext factory for consistent patterns.

### Performance Considerations

Context with useReducer has the same performance characteristics as plain context. All consumers re-render when the state reference changes. Use useMemo for the context value and consider splitting state that changes at different frequencies.

### Interview Questions

1. Why would you use useReducer with context instead of Redux?
2. How do you type the dispatch function when using context?
3. How do you prevent all consumers from re-rendering when only part of the state changes?

### Coding Challenges

1. Create a useStore hook that provides typed getState and dispatch from a context-backed store.
2. Build a NotificationProvider using useReducer that supports dd, dismiss, and clearAll actions.
3. Implement a CounterApp with increment, decrement, reset, and undo functionality using context + useReducer.

### Related Topics

- useReducer
- Redux comparison
- State management patterns
