# State - useState typing, useReducer typing, state management types, discriminated unions for state

## Introduction

State management lies at the heart of every React application. TypeScript enhances state management by ensuring that state values, updater functions, and state transitions are type-safe. From the simplicity of `useState` with its automatic type inference to the power of `useReducer` with complex state shapes and discriminated unions, TypeScript provides compile-time guarantees that prevent entire categories of bugs. This file explores how to type `useState`, `useReducer`, complex state management patterns, and discriminated unions for modeling distinct state shapes.

## useState Type Inference and Annotations

### What It Is

`useState` is a React Hook that lets you add state to function components. TypeScript can often infer the type of the state from the initial value, but explicit type annotations can be provided when inference is insufficient or when the state type is a union.

### Why It Is Important

Correct typing of `useState` ensures that state updates are type-safe, preventing accidental assignment of wrong types. It also enables IDE autocompletion on state values and ensures that updater functions receive correctly typed previous state.

### How It Works Internally

`useState` is declared in `@types/react` as a generic function:

```typescript
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>];

type SetStateAction<S> = S | ((prevState: S) => S);
type Dispatch<A> = (value: A) => void;
```

When called with an initial value, TypeScript infers `S` from that value. The returned tuple contains the state value and a dispatch function that accepts either a new value or a function from previous state to new state.

### Syntax

```typescript
// Inferred type (string)
const [name, setName] = useState('Alice');

// Explicit type annotation (number | null)
const [count, setCount] = useState<number | null>(null);

// Union type
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');

// Complex object with type argument
const [user, setUser] = useState<User | null>(null);
```

### Beginner Examples

```typescript
import React, { useState } from 'react';

// Simple counter - type inferred as number
const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount((prev) => prev - 1)}>Decrement</button>
    </div>
  );
};

// Toggle - type inferred as boolean
const Toggle = () => {
  const [isOn, setIsOn] = useState(false);
  return <button onClick={() => setIsOn(!isOn)}>{isOn ? 'ON' : 'OFF'}</button>;
};

// Text input - type inferred as string
const TextInput = () => {
  const [text, setText] = useState('');
  return <input value={text} onChange={(e) => setText(e.target.value)} />;
};
```

### Intermediate Examples

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

const UserProfile = () => {
  const [user, setUser] = useState<User | null>(null);
  const [isEditing, setIsEditing] = useState(false);

  // Update specific properties while keeping others
  const updateName = (name: string) => {
    setUser((prev) => prev ? { ...prev, name } : null);
  };

  if (!user) return <p>Loading...</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <p>Role: {user.role}</p>
      <button onClick={() => setIsEditing(!isEditing)}>
        {isEditing ? 'Cancel' : 'Edit'}
      </button>
    </div>
  );
};
```

### Advanced Examples

```typescript
// Lazy initialization with a function
const ExpensiveInitialization = () => {
  const [data, setData] = useState(() => {
    const initial = performExpensiveComputation();
    return initial;
  });
  // The function is only called on the initial render
};

// useState with callback setters and complex updates
type Action = 'increment' | 'decrement' | 'reset' | { type: 'set'; value: number };

const ComplexCounter = () => {
  const [count, setCount] = useState(0);

  const dispatch = (action: Action) => {
    switch (action) {
      case 'increment':
        setCount((c) => c + 1);
        break;
      case 'decrement':
        setCount((c) => c - 1);
        break;
      case 'reset':
        setCount(0);
        break;
      default:
        setCount(action.value);
        break;
    }
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch('increment')}>+</button>
      <button onClick={() => dispatch('decrement')}>-</button>
      <button onClick={() => dispatch('reset')}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', value: 100 })}>Set to 100</button>
    </div>
  );
};

// useState with generic function for array state
function useArrayState<T>(initial: T[]) {
  const [array, setArray] = useState<T[]>(initial);

  const push = (item: T) => setArray((prev) => [...prev, item]);
  const remove = (index: number) => setArray((prev) => prev.filter((_, i) => i !== index));
  const update = (index: number, item: T) =>
    setArray((prev) => prev.map((existing, i) => (i === index ? item : existing)));
  const clear = () => setArray([]);

  return { array, setArray, push, remove, update, clear };
}
```

### Real-World Use Cases

```typescript
// Form state management with useState
interface FormState {
  values: Record<string, string>;
  errors: Record<string, string | undefined>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
}

const useFormState = (initialValues: Record<string, string>) => {
  const [state, setState] = useState<FormState>({
    values: initialValues,
    errors: {},
    touched: {},
    isSubmitting: false,
  });

  const setFieldValue = (name: string, value: string) => {
    setState((prev) => ({
      ...prev,
      values: { ...prev.values, [name]: value },
    }));
  };

  const setFieldTouched = (name: string) => {
    setState((prev) => ({
      ...prev,
      touched: { ...prev.touched, [name]: true },
    }));
  };

  const setFieldError = (name: string, error: string | undefined) => {
    setState((prev) => ({
      ...prev,
      errors: { ...prev.errors, [name]: error },
    }));
  };

  return { state, setFieldValue, setFieldTouched, setFieldError };
};
```

### Common Mistakes

```typescript
// MISTAKE: TypeScript infers too broad a type
const [value, setValue] = useState(''); // inferred as string
setValue(42); // Error: Argument of type 'number' not assignable to 'string' - GOOD

// MISTAKE: Not providing type argument when initial value is null
const [data, setData] = useState(null); // inferred as null, not DataType | null
// CORRECT: useState<DataType | null>(null)

// MISTAKE: Mutating state directly
const [items, setItems] = useState([1, 2, 3]);
items.push(4); // Mutation!
setItems(items); // React won't re-render because reference hasn't changed
// CORRECT: setItems([...items, 4])
```

### Best Practices

- Rely on type inference when the initial value clearly defines the state type.
- Use explicit type annotations with `| null` when starting from `null`.
- Use lazy initialization for expensive computations.
- Never mutate state directly; always return new references.
- Use updater functions (`setState((prev) => ...)`) when the new state depends on the previous state.

### Performance Considerations

`useState` is highly optimized by React. Setting state triggers a re-render, but the type system has no runtime cost. Using multiple `useState` calls vs a single state object is a trade-off between readability and performance; object state with `useState` can cause unnecessary re-renders if not managed carefully.

### Interview Questions

1. How does TypeScript infer the type for `useState`?
2. When would you need to explicitly annotate the type argument for `useState`?
3. What is the difference between `setState(newValue)` and `setState((prev) => newValue)` in terms of typing?
4. How do you type `useState` for an array of objects?

### Coding Challenges

1. Implement a typed `useLocalStorage` hook that uses `useState` internally and syncs with `localStorage`.
2. Create a `useFormState` hook that manages form values, errors, and touched state with full type safety.
3. Build a `useSelection` hook that manages a set of selected items with typed `select`, `deselect`, and `toggle` operations.

### Related Topics

- useReducer
- Custom hooks
- TypeScript generics with hooks

## useReducer Typing

### What It Is

`useReducer` is a React Hook that manages complex state logic through a reducer function. It is an alternative to `useState` that is preferable when state transitions depend on complex logic or when the next state depends on the previous state in non-trivial ways.

### Why It Is Important

Typing `useReducer` ensures that every action dispatched to the reducer is valid, that the reducer returns the correct state shape, and that the dispatch function can only be called with defined action types. This eliminates runtime errors caused by misspelled action types or malformed action payloads.

### How It Works Internally

`useReducer` is declared in `@types/react` as:

```typescript
function useReducer<R extends Reducer<any, any>, I>(
  reducer: R,
  initializerArg: I,
  initializer: (arg: I) => ReducerState<R>
): [ReducerState<R>, Dispatch<ReducerAction<R>>];

type Reducer<S, A> = (prevState: S, action: A) => S;
type ReducerState<R extends Reducer<any, any>> = R extends Reducer<infer S, any> ? S : never;
type ReducerAction<R extends Reducer<any, any>> = R extends Reducer<any, infer A> ? A : never;
```

The state type `S` and action type `A` are inferred from the reducer function signature. The `dispatch` function is typed to only accept actions of type `A`.

### Syntax

```typescript
import React, { useReducer } from 'react';

// Define action types
type CounterAction =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'set'; payload: number };

// Define state type
type CounterState = { count: number };

// Reducer function
function counterReducer(state: CounterState, action: CounterAction): CounterState {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      return state;
  }
}

// Component
const Counter = () => {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 100 })}>Set to 100</button>
    </div>
  );
};
```

### Beginner Examples

```typescript
// Simple toggle reducer
type ToggleAction = { type: 'toggle' } | { type: 'set'; value: boolean };
type ToggleState = { isOn: boolean };

function toggleReducer(state: ToggleState, action: ToggleAction): ToggleState {
  switch (action.type) {
    case 'toggle':
      return { isOn: !state.isOn };
    case 'set':
      return { isOn: action.value };
  }
}

const Toggle = () => {
  const [state, dispatch] = useReducer(toggleReducer, { isOn: false });
  return (
    <button onClick={() => dispatch({ type: 'toggle' })}>
      {state.isOn ? 'ON' : 'OFF'}
    </button>
  );
};

// Input state reducer
type InputAction =
  | { type: 'change'; payload: string }
  | { type: 'reset' }
  | { type: 'setError'; payload: string };

type InputState = {
  value: string;
  error: string | null;
  isDirty: boolean;
};

function inputReducer(state: InputState, action: InputAction): InputState {
  switch (action.type) {
    case 'change':
      return { ...state, value: action.payload, isDirty: true, error: null };
    case 'reset':
      return { value: '', error: null, isDirty: false };
    case 'setError':
      return { ...state, error: action.payload };
  }
}
```

### Intermediate Examples

```typescript
// Fetch state reducer with discriminated union
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

type FetchAction<T> =
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: T }
  | { type: 'FETCH_ERROR'; payload: string };

function fetchReducer<T>(state: FetchState<T>, action: FetchAction<T>): FetchState<T> {
  switch (action.type) {
    case 'FETCH_START':
      return { status: 'loading' };
    case 'FETCH_SUCCESS':
      return { status: 'success', data: action.payload };
    case 'FETCH_ERROR':
      return { status: 'error', error: action.payload };
  }
}

// Generic useFetch hook
function useFetch<T>(url: string) {
  const [state, dispatch] = useReducer(fetchReducer<T>, { status: 'idle' });

  useEffect(() => {
    dispatch({ type: 'FETCH_START' });
    fetch(url)
      .then((res) => res.json() as Promise<T>)
      .then((data) => dispatch({ type: 'FETCH_SUCCESS', payload: data }))
      .catch((err) => dispatch({ type: 'FETCH_ERROR', payload: err.message }));
  }, [url]);

  return state;
}
```

### Advanced Examples

```typescript
// Complex form state reducer
interface FormField<T = string> {
  value: T;
  error: string | null;
  touched: boolean;
  validating: boolean;
}

type FormState<T extends Record<string, unknown>> = {
  [K in keyof T]: FormField<T[K]>;
};

type FormAction<T extends Record<string, unknown>> =
  | { type: 'SET_FIELD'; field: keyof T; value: T[keyof T] }
  | { type: 'SET_ERROR'; field: keyof T; error: string | null }
  | { type: 'TOUCH_FIELD'; field: keyof T }
  | { type: 'START_VALIDATING'; field: keyof T }
  | { type: 'STOP_VALIDATING'; field: keyof T }
  | { type: 'RESET'; initialState: FormState<T> };

function formReducer<T extends Record<string, unknown>>(
  state: FormState<T>,
  action: FormAction<T>
): FormState<T> {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        [action.field]: {
          ...state[action.field],
          value: action.value,
          error: null,
        },
      };
    case 'SET_ERROR':
      return {
        ...state,
        [action.field]: {
          ...state[action.field],
          error: action.error,
        },
      };
    case 'TOUCH_FIELD':
      return {
        ...state,
        [action.field]: {
          ...state[action.field],
          touched: true,
        },
      };
    case 'START_VALIDATING':
      return {
        ...state,
        [action.field]: {
          ...state[action.field],
          validating: true,
        },
      };
    case 'STOP_VALIDATING':
      return {
        ...state,
        [action.field]: {
          ...state[action.field],
          validating: false,
        },
      };
    case 'RESET':
      return action.initialState;
  }
}

// Usage with specific form shape
type LoginForm = { email: string; password: string };
const initialLoginState: FormState<LoginForm> = {
  email: { value: '', error: null, touched: false, validating: false },
  password: { value: '', error: null, touched: false, validating: false },
};

const LoginForm = () => {
  const [state, dispatch] = useReducer(formReducer<LoginForm>, initialLoginState);

  const handleSubmit = () => {
    dispatch({ type: 'TOUCH_FIELD', field: 'email' });
    dispatch({ type: 'TOUCH_FIELD', field: 'password' });
    // Validation logic...
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={state.email.value}
        onChange={(e) => dispatch({ type: 'SET_FIELD', field: 'email', value: e.target.value })}
      />
      {state.email.error && <span>{state.email.error}</span>}
      <input
        type="password"
        value={state.password.value}
        onChange={(e) => dispatch({ type: 'SET_FIELD', field: 'password', value: e.target.value })}
      />
      {state.password.error && <span>{state.password.error}</span>}
      <button type="submit">Login</button>
    </form>
  );
};
```

### Real-World Use Cases

```typescript
// Shopping cart reducer
interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
}

type CartState = {
  items: CartItem[];
  isOpen: boolean;
  couponCode: string | null;
  discount: number;
};

type CartAction =
  | { type: 'ADD_ITEM'; payload: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { productId: string; quantity: number } }
  | { type: 'TOGGLE_CART' }
  | { type: 'APPLY_COUPON'; payload: string }
  | { type: 'REMOVE_COUPON' }
  | { type: 'CLEAR_CART' };

function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find((item) => item.productId === action.payload.productId);
      if (existing) {
        return {
          ...state,
          items: state.items.map((item) =>
            item.productId === action.payload.productId
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
      };
    }
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter((item) => item.productId !== action.payload),
      };
    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map((item) =>
          item.productId === action.payload.productId
            ? { ...item, quantity: action.payload.quantity }
            : item
        ),
      };
    case 'TOGGLE_CART':
      return { ...state, isOpen: !state.isOpen };
    case 'APPLY_COUPON':
      return { ...state, couponCode: action.payload, discount: 0.1 };
    case 'REMOVE_COUPON':
      return { ...state, couponCode: null, discount: 0 };
    case 'CLEAR_CART':
      return { ...state, items: [] };
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Not handling all action types in the switch
// TypeScript helps here if the return type is explicitly stated

// MISTAKE: Mutating state in reducer
// function reducer(state, action) { state.count += 1; return state; } // BAD
// CORRECT: return { ...state, count: state.count + 1 };

// MISTAKE: Using string literals instead of union types for action.type
// type Action = { type: string; payload: any }; // BAD - no type safety

// MISTAKE: Not using the `never` type in default case
// default: const _exhaustiveCheck: never = action; return state; // Ensures all cases handled
```

### Best Practices

- Define action types as discriminated unions with a literal `type` property.
- Use `interface` for state and `type` for action unions.
- Always return new state objects, never mutate.
- Include a `default` case with an exhaustiveness check.
- Use action creators or action helper functions for complex payload construction.
- Keep reducers pure and side-effect free.

### Performance Considerations

`useReducer` has similar performance characteristics to `useState`. The reducer function should be pure and fast. For very large state objects, consider using `useImmerReducer` from `use-immer` for simpler immutable updates. The typing has no runtime impact.

### Interview Questions

1. How does `useReducer` typing differ from `useState` typing?
2. What is the purpose of discriminated unions in reducer actions?
3. How do you implement an exhaustiveness check in a reducer?
4. When would you choose `useReducer` over `useState`?

### Coding Challenges

1. Implement a generic `useAsyncReducer` that handles async actions with `loading`, `success`, and `error` states.
2. Create a `useUndoReducer` that extends a base reducer with undo/redo functionality.
3. Build a todo list with `useReducer` where actions include `add`, `toggle`, `remove`, `edit`, and `filter`.

### Related Topics

- useState
- Context API with reducer
- Redux concepts
- State machines

## Complex State with Discriminated Unions

### What It Is

A discriminated union (also called a tagged union) is a type pattern where each member of a union has a common property (the discriminant) with a literal type. TypeScript can narrow the type based on the discriminant value, providing type-safe access to properties that exist only on specific members.

### Why It Is Important

Discriminated unions are the most powerful pattern for modeling state machines and complex state in TypeScript. They ensure that only the properties relevant to the current state are accessible, preventing impossible states and reducing the need for runtime checks.

### How It Works Internally

TypeScript's control flow analysis can narrow a discriminated union type based on checking the discriminant property. When you check `state.status === 'loading'`, TypeScript knows that within that branch, the state is the specific member of the union with `status: 'loading'`, and only the properties of that member are accessible.

### Syntax

```typescript
// Define a discriminated union
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T; timestamp: number }
  | { status: 'error'; error: Error; retryCount: number };
```

### Beginner Examples

```typescript
type AuthState =
  | { phase: 'unauthenticated' }
  | { phase: 'authenticating'; provider: 'google' | 'github' | 'email' }
  | { phase: 'authenticated'; user: { id: string; name: string; email: string }; token: string }
  | { phase: 'error'; message: string };

function AuthStatus({ state }: { state: AuthState }) {
  switch (state.phase) {
    case 'unauthenticated':
      return <button>Login</button>;
    case 'authenticating':
      return <Spinner text={`Signing in with ${state.provider}`} />;
    case 'authenticated':
      return <div>Welcome, {state.user.name}! Token: {state.token.slice(0, 10)}...</div>;
    case 'error':
      return <ErrorBanner message={state.message} />;
  }
}
```

### Intermediate Examples

```typescript
type PaymentState =
  | { status: 'pending'; amount: number; currency: string }
  | { status: 'processing'; amount: number; currency: string; transactionId: string }
  | { status: 'completed'; amount: number; currency: string; transactionId: string; completedAt: Date }
  | { status: 'failed'; amount: number; currency: string; error: string; retryAllowed: boolean }
  | { status: 'refunded'; amount: number; currency: string; transactionId: string; refundedAt: Date; reason: string };

function PaymentSummary({ payment }: { payment: PaymentState }) {
  const baseInfo = <span>{payment.amount} {payment.currency}</span>;

  switch (payment.status) {
    case 'pending':
      return <div>{baseInfo} - Awaiting payment</div>;
    case 'processing':
      return <div>{baseInfo} - Processing (ID: {payment.transactionId})</div>;
    case 'completed':
      return <div>{baseInfo} - Completed at {payment.completedAt.toLocaleDateString()}</div>;
    case 'failed':
      return (
        <div>
          {baseInfo} - Failed: {payment.error}
          {payment.retryAllowed && <button onClick={retry}>Retry</button>}
        </div>
      );
    case 'refunded':
      return (
        <div>
          {baseInfo} - Refunded: {payment.reason}
        </div>
      );
  }
}
```

### Advanced Examples

```typescript
// Nested discriminated unions
type Notification =
  | {
      type: 'message';
      payload: {
        kind: 'text'; content: string;
      } | {
        kind: 'image'; url: string; alt: string; width?: number; height?: number;
      } | {
        kind: 'file'; name: string; size: number; mimeType: string;
      };
      metadata: { sentAt: Date; read: boolean };
    }
  | {
      type: 'alert';
      severity: 'info' | 'warning' | 'critical';
      message: string;
      action?: { label: string; handler: () => void };
    }
  | {
      type: 'system';
      code: number;
      description: string;
      autoDismiss: boolean;
    };

function renderNotification(n: Notification) {
  switch (n.type) {
    case 'message': {
      switch (n.payload.kind) {
        case 'text':
          return <p>{n.payload.content}</p>;
        case 'image':
          return <img src={n.payload.url} alt={n.payload.alt} width={n.payload.width} />;
        case 'file':
          return <FileBadge name={n.payload.name} size={n.payload.size} type={n.payload.mimeType} />;
      }
      break;
    }
    case 'alert':
      return (
        <AlertBanner severity={n.severity}>
          {n.message}
          {n.action && <button onClick={n.action.handler}>{n.action.label}</button>}
        </AlertBanner>
      );
    case 'system':
      return <SystemNotification code={n.code}>{n.description}</SystemNotification>;
  }
}

// State machine with transitions
type LightState =
  | { color: 'red'; timer: number }
  | { color: 'yellow'; timer: number }
  | { color: 'green'; timer: number };

type LightAction =
  | { type: 'TICK' }
  | { type: 'RESET' };

function trafficLightReducer(state: LightState, action: LightAction): LightState {
  switch (action.type) {
    case 'TICK':
      switch (state.color) {
        case 'red':
          return state.timer <= 1
            ? { color: 'green', timer: 30 }
            : { ...state, timer: state.timer - 1 };
        case 'yellow':
          return state.timer <= 1
            ? { color: 'red', timer: 60 }
            : { ...state, timer: state.timer - 1 };
        case 'green':
          return state.timer <= 1
            ? { color: 'yellow', timer: 5 }
            : { ...state, timer: state.timer - 1 };
      }
      break;
    case 'RESET':
      return { color: 'red', timer: 60 };
  }
}
```

### Real-World Use Cases

```typescript
// Data fetching with full type safety
type AsyncData<T, E = Error> =
  | { kind: 'idle' }
  | { kind: 'loading' }
  | { kind: 'success'; data: T }
  | { kind: 'error'; error: E };

function useAsync<T, E = Error>(
  asyncFn: () => Promise<T>
): AsyncData<T, E> & { execute: () => void; reset: () => void } {
  const [state, setState] = useState<AsyncData<T, E>>({ kind: 'idle' });

  const execute = useCallback(() => {
    setState({ kind: 'loading' });
    asyncFn()
      .then((data) => setState({ kind: 'success', data }))
      .catch((error: E) => setState({ kind: 'error', error }));
  }, [asyncFn]);

  const reset = useCallback(() => setState({ kind: 'idle' }), []);

  return { ...state, execute, reset };
}

// UI state machine for a modal
type ModalState<T = unknown> =
  | { phase: 'closed' }
  | { phase: 'opening'; animation: 'fade' | 'slide' }
  | { phase: 'open'; data?: T }
  | { phase: 'closing'; animation: 'fade' | 'slide' }
  | { phase: 'submitting'; data: T }
  | { phase: 'error'; data: T; error: string };

function Modal<T>({ state, onClose, onSubmit }: {
  state: ModalState<T>;
  onClose: () => void;
  onSubmit: (data: T) => Promise<void>;
}) {
  switch (state.phase) {
    case 'closed':
      return null;
    case 'opening':
    case 'closing':
      return <AnimatedOverlay animation={state.animation}>...</AnimatedOverlay>;
    case 'open':
      return (
        <ModalOverlay onClose={onClose}>
          <ModalContent data={state.data} />
          <button onClick={() => state.data && onSubmit(state.data)}>Submit</button>
        </ModalOverlay>
      );
    case 'submitting':
      return <ModalOverlay><Spinner /></ModalOverlay>;
    case 'error':
      return (
        <ModalOverlay>
          <ErrorDisplay message={state.error} />
          <button onClick={() => state.data && onSubmit(state.data)}>Retry</button>
        </ModalOverlay>
      );
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Not using a literal type for the discriminant
type BadState = { status: string; data?: unknown }; // BAD - no narrowing
// CORRECT: type GoodState = { status: 'idle' } | { status: 'loading' } | ...

// MISTAKE: Discriminant property with the same name but different types
// TypeScript requires the discriminant to be a literal type

// MISTAKE: Overly complex discriminated unions
// Break down into smaller helper types
```

### Best Practices

- Always use a literal string or number type for the discriminant property.
- Name the discriminant property consistently across your codebase (`status`, `phase`, `kind`, `type`).
- Use exhaustive switch statements to ensure all cases are handled.
- Combine discriminated unions with `useReducer` for state machine patterns.
- Keep discriminated unions focused; avoid deeply nesting more than 2-3 levels.

### Performance Considerations

Discriminated unions are erased at compile time. However, runtime switch statements over the discriminant add minimal overhead. The type-level benefits far outweigh any runtime cost.

### Interview Questions

1. What is a discriminated union and how does TypeScript narrow types with it?
2. How would you model a loading/success/error state with a discriminated union?
3. What is the `never` type used for in exhaustive switch statements?
4. How do you nest discriminated unions for complex state machines?

### Coding Challenges

1. Model a video player state with discriminated unions for `playing`, `paused`, `buffering`, `ended`, and `error` states.
2. Create a `useStateMachine` hook that takes a discriminated union state and transitions, enforcing valid transitions at compile time.
3. Build a multi-step checkout wizard where each step is a discriminated union member with its own data and validations.

### Related Topics

- Discriminated unions in TypeScript
- useReducer with discriminated unions
- State machines (XState)
- Pattern matching
