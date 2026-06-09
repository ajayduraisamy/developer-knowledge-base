# Redux Toolkit - createSlice, configureStore, typed hooks, createAsyncThunk, middleware typing

## Introduction

Redux Toolkit (RTK) is the official, opinionated way to write Redux logic. It simplifies store setup, reduces boilerplate, and provides powerful utilities like createSlice and createAsyncThunk. TypeScript integration with RTK is first-class, with automatic inference of action types, state shapes, and thunk return types. This file covers createSlice and actions typing, configureStore and RootState, typed useDispatch and useSelector, createAsyncThunk typing, and middleware typing patterns.

## createSlice and Actions Type

### What It Is

createSlice is a function that generates reducer functions and action creators based on a configuration object with 
ame, initialState, and educers fields. TypeScript infers the action types and state type from the slice definition, eliminating the need for manual action type constants.

### Why It Is Important

createSlice reduces Redux boilerplate by generating action creators and action types automatically. With TypeScript, the generated actions are fully typed, and reducer state transitions are type-checked, preventing common mistakes like misspelled action types or malformed payloads.

### How It Works Internally

createSlice returns an object with educer, ctions, and caseReducers properties. The ctions object contains action creators that each return a PayloadAction<T> where T is inferred from the reducer function's second parameter type. The educer is a generated reducer that handles all defined cases.

`	ypescript
function createSlice<State, CaseReducers extends SliceCaseReducers<State>, Name extends string>(config: {
  name: Name;
  initialState: State;
  reducers: CaseReducers;
}): Slice<State, CaseReducers, Name>;
`

### Syntax

`	ypescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
  incrementAmount: number;
}

const initialState: CounterState = {
  value: 0,
  incrementAmount: 1,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += state.incrementAmount;
    },
    decrement: (state) => {
      state.value -= state.incrementAmount;
    },
    setValue: (state, action: PayloadAction<number>) => {
      state.value = action.payload;
    },
    setIncrementAmount: (state, action: PayloadAction<number>) => {
      state.incrementAmount = action.payload;
    },
  },
});

export const { increment, decrement, setValue, setIncrementAmount } = counterSlice.actions;
export default counterSlice.reducer;
`

### Beginner Examples

`	ypescript
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

interface TodosState {
  items: Todo[];
  filter: 'all' | 'active' | 'completed';
}

const initialState: TodosState = {
  items: [],
  filter: 'all',
};

const todosSlice = createSlice({
  name: 'todos',
  initialState,
  reducers: {
    addTodo: (state, action: PayloadAction<string>) => {
      state.items.push({ id: crypto.randomUUID(), text: action.payload, completed: false });
    },
    toggleTodo: (state, action: PayloadAction<string>) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },
    removeTodo: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter(t => t.id !== action.payload);
    },
    setFilter: (state, action: PayloadAction<TodosState['filter']>) => {
      state.filter = action.payload;
    },
  },
});
`

### Intermediate Examples

`	ypescript
// Slice with prepare callbacks
interface UserState {
  users: Record<string, { name: string; email: string; addedAt: string }>;
  selectedUserId: string | null;
}

const userSlice = createSlice({
  name: 'users',
  initialState: { users: {}, selectedUserId: null } as UserState,
  reducers: {
    addUser: {
      reducer: (state, action: PayloadAction<{ id: string; name: string; email: string; addedAt: string }>) => {
        state.users[action.payload.id] = {
          name: action.payload.name,
          email: action.payload.email,
          addedAt: action.payload.addedAt,
        };
      },
      prepare: (name: string, email: string) => ({
        payload: { id: crypto.randomUUID(), name, email, addedAt: new Date().toISOString() },
      }),
    },
    selectUser: (state, action: PayloadAction<string | null>) => {
      state.selectedUserId = action.payload;
    },
  },
});

// Slice with extra reducers
interface DataState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

const initialState: DataState<any> = { data: null, loading: false, error: null };
`

### Advanced Examples

`	ypescript
// Reusing reducer logic across slices
const baseUserReducers = {
  setUserName: (state: { name: string }, action: PayloadAction<string>) => {
    state.name = action.payload;
  },
};

const adminSlice = createSlice({
  name: 'admin',
  initialState: { name: '', role: 'admin' as const },
  reducers: {
    ...baseUserReducers,
    setAdminRole: (state, action: PayloadAction<string>) => {
      state.role = action.payload;
    },
  },
});

// Typed slice selectors
const selectTodos = (state: RootState) => state.todos;
const selectFilteredTodos = (state: RootState) => {
  const { items, filter } = state.todos;
  switch (filter) {
    case 'active': return items.filter(t => !t.completed);
    case 'completed': return items.filter(t => t.completed);
    default: return items;
  }
};
`

### Real-World Use Cases

`	ypescript
// Auth slice with complex state
interface AuthState {
  user: { id: string; email: string; name: string; avatarUrl: string } | null;
  token: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    token: null,
    refreshToken: null,
    isAuthenticated: false,
    isLoading: false,
    error: null,
  } as AuthState,
  reducers: {
    loginStart: (state) => {
      state.isLoading = true;
      state.error = null;
    },
    loginSuccess: (state, action: PayloadAction<{ user: AuthState['user']; token: string; refreshToken: string }>) => {
      state.isLoading = false;
      state.isAuthenticated = true;
      state.user = action.payload.user;
      state.token = action.payload.token;
      state.refreshToken = action.payload.refreshToken;
    },
    loginFailure: (state, action: PayloadAction<string>) => {
      state.isLoading = false;
      state.error = action.payload;
    },
    logout: (state) => {
      state.user = null;
      state.token = null;
      state.refreshToken = null;
      state.isAuthenticated = false;
      state.error = null;
    },
    updateToken: (state, action: PayloadAction<{ token: string; refreshToken: string }>) => {
      state.token = action.payload.token;
      state.refreshToken = action.payload.refreshToken;
    },
  },
});
`

### Common Mistakes

`	ypescript
// MISTAKE: Mutating state outside of Immer's allowed operations
// createSlice uses Immer, but direct assignment to state works
// DON'T: return { ...state, value: action.payload } // Unnecessary
// DO: state.value = action.payload

// MISTAKE: Not typing the action payload
// reducers: { add: (state, action) => { } } // action: any

// MISTAKE: Using non-serializable values in state
// Avoid: Date objects, functions, Promises in state
`

### Best Practices

- Define initialState with explicit type annotation for better inference.
- Use PayloadAction<T> for action payloads in reducer functions.
- Leverage Immer's mutable syntax for simpler reducer code.
- Export action creators and individual selectors for use in components.
- Use prepare callbacks when action payloads need transformation.

### Performance Considerations

createSlice uses Immer internally, which provides efficient immutable updates with a mutable API. Immer's performance is excellent for most use cases. For extremely large state trees, consider using createSlice with { reducers: ... } without Immer (using spread syntax).

### Interview Questions

1. How does createSlice generate action types and action creators?
2. What is the purpose of the prepare callback in createSlice?
3. How does Immer enable mutable-looking reducer syntax?
4. How are action types constructed from the slice name and reducer key?

### Coding Challenges

1. Create a productsSlice with reducers for ddProduct, updateProduct, emoveProduct, and setFilter.
2. Build a slice that manages pagination state with 
extPage, prevPage, setPageSize, and setTotal reducers.

### Related Topics

- configureStore
- createAsyncThunk
- Immer
- Redux DevTools

## configureStore and RootState

### What It Is

configureStore is a function that creates a Redux store with good defaults, including middleware setup (thunk by default), DevTools integration, and combined reducers. The RootState type is inferred from the store configuration, providing type-safe access to the entire state tree.

### Why It Is Important

configureStore simplifies store creation while enabling full TypeScript integration. The inferred RootState type eliminates manual type definitions for the state shape and ensures that selectors and dispatches are type-checked.

### How It Works Internally

`	ypescript
function configureStore<S, A, M>(options: ConfigureStoreOptions<S, A, M>): EnhancedStore<S, A, M>;

type ConfigureStoreOptions<S, A, M> = {
  reducer: Reducer<S, A> | ReducersMapObject<S, A>;
  middleware?: Middleware<{}, S, Dispatch>[] | ((getDefaultMiddleware: CurriedGetDefaultMiddleware<S>) => Middleware[]);
  devTools?: boolean | DevToolsOptions;
  preloadedState?: S;
  enhancers?: StoreEnhancer[];
};
`

When educer is a map of slice reducers, configureStore calls combineReducers and infers S as the combined state type.

### Syntax

`	ypescript
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';
import todosReducer from './todosSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    todos: todosReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
`

### Beginner Examples

`	ypescript
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
// RootState is { counter: CounterState }
`

### Intermediate Examples

`	ypescript
// Store with middleware configuration
const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST'],
        ignoredPaths: ['user.timestamp'],
      },
      immutableCheck: { warnAfter: 128 },
    }).concat(loggerMiddleware),
  devTools: process.env.NODE_ENV !== 'production',
  preloadedState: {
    counter: { value: 10, incrementAmount: 1 },
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
`

### Advanced Examples

`	ypescript
// Dynamic reducer injection
const staticReducers = {
  counter: counterReducer,
  auth: authReducer,
};

function createDynamicStore() {
  const store = configureStore({
    reducer: staticReducers,
  });

  type ReducerMap = typeof staticReducers;
  type ReducerKeys = keyof ReducerMap;
  const asyncReducers: Partial<ReducerMap> = {};

  const injectReducer = (key: ReducerKeys, reducer: ReducerMap[ReducerKeys]) => {
    if (!asyncReducers[key]) {
      asyncReducers[key] = reducer;
      store.replaceReducer(combineReducers({ ...staticReducers, ...asyncReducers }));
    }
  };

  return { ...store, injectReducer };
}

export type AppStore = ReturnType<typeof createDynamicStore>;
export type RootState = ReturnType<AppStore['getState']>;
export type AppDispatch = AppStore['dispatch'];
`

### Real-World Use Cases

`	ypescript
// Store with persistence middleware
import { persistStore, persistReducer, FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER } from 'redux-persist';

const persistConfig = { key: 'root', storage: AsyncStorage };

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    }),
});

export const persistor = persistStore(store);
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
`

### Common Mistakes

`	ypescript
// MISTAKE: Manually defining RootState type
// interface RootState { counter: CounterState; user: UserState; }
// BETTER: Infer from store

// MISTAKE: Not exporting RootState and AppDispatch
// These types are essential for typed hooks and components

// MISTAKE: Importing RootState from the wrong location
// export from the store file, import where needed (hooks file, components)
`

### Best Practices

- Always infer RootState and AppDispatch from the store using ReturnType.
- Export RootState and AppDispatch from a dedicated store.ts file.
- Type the global Dispatch and RootState for convenience with useSelector and useDispatch.
- Use getDefaultMiddleware over custom middleware to retain RTK's default middleware.

### Performance Considerations

configureStore itself runs once during app initialization. The middleware configuration affects dispatch performance. RTK's default middleware (thunk, serializableCheck, immutableCheck) is optimized, but the dev checks can be disabled in production.

### Interview Questions

1. How is RootState type inferred in a Redux Toolkit store?
2. What is the purpose of ReturnType<typeof store.getState>?
3. How do you type middleware in configureStore?

### Coding Challenges

1. Create a store with three slices (auth, todos, notifications) and export the proper types.
2. Build a store configuration that adds a custom middleware for logging and error reporting.

### Related Topics

- createSlice
- Typed hooks
- Middleware

## Typed useDispatch and useSelector

### What It Is

Typed versions of useDispatch and useSelector ensure that dispatch actions are type-checked against the store's dispatch type and that selectors return values consistent with the store's state shape. Custom typed hooks eliminate the need to annotate types in every component.

### Why It Is Important

Without typed hooks, each useDispatch call would need type annotation and each useSelector call would need RootState as a generic parameter. Typed hooks reduce boilerplate, prevent errors, and ensure consistency across the codebase.

### How It Works Internally

`	ypescript
// Pre-typed hooks
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
`

TypedUseSelectorHook<RootState> is a generic type that wraps useSelector and automatically provides RootState as the state type parameter.

### Syntax

`	ypescript
// store.ts
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
`

### Beginner Examples

`	ypescript
// Component using typed hooks
import { useAppDispatch, useAppSelector } from './hooks';
import { increment, decrement } from './counterSlice';

const CounterComponent = () => {
  const count = useAppSelector((state) => state.counter.value);
  const dispatch = useAppDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
};
`

### Intermediate Examples

`	ypescript
// Selector with complex derivation
const TodoList = () => {
  const { items, filter } = useAppSelector((state) => state.todos);

  const filteredTodos = useMemo(() => {
    switch (filter) {
      case 'active': return items.filter(t => !t.completed);
      case 'completed': return items.filter(t => t.completed);
      default: return items;
    }
  }, [items, filter]);

  const dispatch = useAppDispatch();

  return (
    <ul>
      {filteredTodos.map(todo => (
        <li key={todo.id}>
          <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            {todo.text}
          </span>
          <button onClick={() => dispatch(toggleTodo(todo.id))}>Toggle</button>
        </li>
      ))}
    </ul>
  );
};
`

### Advanced Examples

`	ypescript
// Selector factory for reusability
const makeSelectItemsByStatus = (status: 'active' | 'completed') => (state: RootState) => {
  return state.todos.items.filter(t => status === 'completed' ? t.completed : !t.completed);
};

const ActiveTodos = () => {
  const items = useAppSelector(makeSelectItemsByStatus('active'));
  return <TodoItems items={items} />;
};

// Multi-selector component
const Dashboard = () => {
  const dispatch = useAppDispatch();

  const counter = useAppSelector((state) => state.counter);
  const todos = useAppSelector((state) => state.todos);
  const user = useAppSelector((state) => state.auth.user);

  // Dispatch with async thunk
  const handleLogin = () => {
    dispatch(loginThunk({ email: 'test@test.com', password: 'password' }));
  };
};

// Creating typed dispatch for specific action types
function useCounterDispatch() {
  const dispatch = useAppDispatch();
  return {
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    setValue: (value: number) => dispatch(setValue(value)),
  };
}
`

### Real-World Use Cases

`	ypescript
// Feature-specific hook with typed dispatch
const useAuthActions = () => {
  const dispatch = useAppDispatch();
  return {
    login: (email: string, password: string) => dispatch(loginThunk({ email, password })),
    logout: () => dispatch(logout()),
    clearError: () => dispatch(clearError()),
  };
};

const useAuthState = () => {
  return useAppSelector((state) => ({
    user: state.auth.user,
    isAuthenticated: state.auth.isAuthenticated,
    isLoading: state.auth.isLoading,
    error: state.auth.error,
  }));
};

// Combined hook
const useAuth = () => {
  return { ...useAuthState(), ...useAuthActions() };
};
`

### Common Mistakes

`	ypescript
// MISTAKE: Not using typed hooks, forcing repeated type annotations
// const count = useSelector<RootState, number>((s) => s.counter.value); // Verbose
// BETTER: const count = useAppSelector((s) => s.counter.value);

// MISTAKE: Destructuring dispatch from non-typed hooks
// const { dispatch } = useDispatch(); // Wrong: useDispatch is a function call

// MISTAKE: Not using AppDispatch type for thunks
// const dispatch = useDispatch();
// dispatch(loginThunk({ email, password })); // May not be type-checked
`

### Best Practices

- Create and export typed useAppDispatch and useAppSelector hooks in a central hooks.ts file.
- Import the typed hooks instead of the untyped versions from eact-redux.
- Create domain-specific hooks that encapsulate selector and dispatch logic.
- Use createSelector from Reselect for memoized selectors.

### Performance Considerations

useSelector by default uses === reference equality to determine if the selected value has changed. For derived values (objects, arrays), use createSelector or shallowEqual to prevent unnecessary re-renders. Typed hooks add no runtime overhead.

### Interview Questions

1. How do you create typed versions of useDispatch and useSelector?
2. What is TypedUseSelectorHook and how does it work?
3. Why should components use useAppSelector instead of plain useSelector?

### Coding Challenges

1. Create a useAppDispatch hook that correctly types async thunk dispatches.
2. Build a useTodos hook that provides typed todo state and actions.

### Related Topics

- Reselect
- createAsyncThunk
- Redux state management patterns

## createAsyncThunk Typing

### What It Is

createAsyncThunk is a function that generates action creators and reducers for handling async request lifecycles (pending, fulfilled, rejected). It accepts a thunk name, an async payload creator function, and optional configuration. TypeScript infers the pending, fulfilled, and rejected action types, as well as the thunk's return type and argument type.

### Why It Is Important

Async thunks are the standard way to handle side effects in Redux Toolkit. Correct typing ensures that thunk arguments, return values, and error handling are type-safe throughout the async lifecycle, eliminating common issues like mismatched payload types in fulfilled/rejected cases.

### How It Works Internally

`	ypescript
function createAsyncThunk<Returned, ThunkArg, ThunkApiConfig extends AsyncThunkConfig>(typePrefix: string, payloadCreator: AsyncThunkPayloadCreator<Returned, ThunkArg, ThunkApiConfig>, options?: AsyncThunkOptions<ThunkArg, ThunkApiConfig>): AsyncThunk<Returned, ThunkArg, ThunkApiConfig>;
`

The generic parameters are Returned (fulfilled payload type), ThunkArg (argument type), and ThunkApiConfig (for customizing dispatch, state, ejectValue, etc.).

### Syntax

`	ypescript
import { createAsyncThunk } from '@reduxjs/toolkit';

interface FetchUserArgs {
  userId: string;
}

interface User {
  id: string;
  name: string;
  email: string;
}

export const fetchUserById = createAsyncThunk<User, FetchUserArgs>(
  'users/fetchById',
  async ({ userId }, thunkAPI) => {
    const response = await fetch(/api/users/);
    if (!response.ok) {
      return thunkAPI.rejectWithValue('Failed to fetch user');
    }
    return (await response.json()) as User;
  }
);
`

### Beginner Examples

`	ypescript
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

export const fetchTodos = createAsyncThunk<Todo[], void>(
  'todos/fetchTodos',
  async (_, thunkAPI) => {
    const response = await fetch('/api/todos');
    return (await response.json()) as Todo[];
  }
);

// In slice
const todosSlice = createSlice({
  name: 'todos',
  initialState: { items: [], loading: false, error: null } as TodosState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message ?? 'Failed to fetch';
      });
  },
});
`

### Intermediate Examples

`	ypescript
// With typed reject value and state
interface UpdateTodoArgs {
  id: string;
  text: string;
}

interface ValidationError {
  field: string;
  message: string;
}

export const updateTodo = createAsyncThunk<
  Todo,
  UpdateTodoArgs,
  { rejectValue: ValidationError[]; state: RootState }
>(
  'todos/update',
  async ({ id, text }, { rejectWithValue, getState }) => {
    const token = getState().auth.token;
    if (!token) {
      return rejectWithValue([{ field: 'auth', message: 'Not authenticated' }]);
    }

    try {
      const response = await fetch(/api/todos/, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json', Authorization: Bearer  },
        body: JSON.stringify({ text }),
      });

      if (!response.ok) {
        const errors: ValidationError[] = await response.json();
        return rejectWithValue(errors);
      }

      return (await response.json()) as Todo;
    } catch (err) {
      return rejectWithValue([{ field: 'network', message: 'Network error' }]);
    }
  }
);
`

### Advanced Examples

`	ypescript
// Chained async thunks
export const createUserAndWorkspace = createAsyncThunk<
  { user: User; workspace: Workspace },
  { email: string; password: string; workspaceName: string }
>(
  'auth/createUserAndWorkspace',
  async (args, { dispatch, rejectWithValue }) => {
    try {
      const userResult = await dispatch(createUser({ email: args.email, password: args.password })).unwrap();
      const workspaceResult = await dispatch(createWorkspace({ name: args.workspaceName, userId: userResult.id })).unwrap();
      return { user: userResult, workspace: workspaceResult };
    } catch (err) {
      return rejectWithValue(err);
    }
  }
);

// Thunk with signal for cancellation
export const searchItems = createAsyncThunk<SearchResult[], string>(
  'items/search',
  async (query, { signal }) => {
    const response = await fetch(/api/items?q=, { signal });
    return (await response.json()) as SearchResult[];
  }
);

// Conditional fetching
export const fetchUserIfNeeded = createAsyncThunk<User, string, { state: RootState }>(
  'users/fetchIfNeeded',
  async (userId, { getState, rejectWithValue }) => {
    const existing = getState().users.cache[userId];
    if (existing && Date.now() - existing.lastFetched < 60000) {
      return existing.data;
    }
    const response = await fetch(/api/users/);
    if (!response.ok) return rejectWithValue('Not found');
    return (await response.json()) as User;
  },
  {
    condition: (userId, { getState }) => {
      const state = getState() as RootState;
      return !state.users.loading[userId];
    },
  }
);
`

### Real-World Use Cases

`	ypescript
// Paginated data loading
interface FetchPageArgs {
  page: number;
  pageSize: number;
  filters?: Record<string, string>;
}

interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

export const fetchProductsPage = createAsyncThunk<
  PaginatedResponse<Product>,
  FetchPageArgs,
  { rejectValue: string }
>(
  'products/fetchPage',
  async ({ page, pageSize, filters }, { rejectWithValue }) => {
    const params = new URLSearchParams({ page: String(page), pageSize: String(pageSize) });
    if (filters) {
      Object.entries(filters).forEach(([key, value]) => params.append(key, value));
    }
    const response = await fetch(/api/products?);
    if (!response.ok) return rejectWithValue('Failed to load products');
    return (await response.json()) as PaginatedResponse<Product>;
  }
);

// File upload
export const uploadFile = createAsyncThunk<UploadedFile, File, { rejectValue: string }>(
  'files/upload',
  async (file, { rejectWithValue }) => {
    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await fetch('/api/files/upload', { method: 'POST', body: formData });
      if (!response.ok) return rejectWithValue('Upload failed');
      return (await response.json()) as UploadedFile;
    } catch (err) {
      return rejectWithValue('Network error during upload');
    }
  }
);
`

### Common Mistakes

`	ypescript
// MISTAKE: Not handling the rejected case properly
// Always handle rejected, fulfilled, and pending in extraReducers

// MISTAKE: Forgetting to use unwrap() when chaining thunks
// dispatch(thunk()).unwrap() unwraps the result or throws

// MISTAKE: Not typing the rejectValue 
// Without rejectValue, action.error is used for all errors
`

### Best Practices

- Always specify ejectValue type for structured error handling.
- Use extraReducers with the builder pattern for type-safe case handling.
- Call .unwrap() on dispatched thunks to get proper Promise resolution.
- Use condition option to prevent duplicate or unnecessary requests.
- Pass {signal} from thunkAPI to fetch for cancellation support.

### Performance Considerations

Async thunks dispatch three actions (pending, fulfilled/rejected). Each dispatch triggers a re-render of connected components. Consider debouncing or using condition to prevent excessive dispatches. The type system adds no runtime overhead.

### Interview Questions

1. What are the three action types generated by createAsyncThunk?
2. How do you type the return value and argument of an async thunk?
3. What is ejectWithValue and why would you use it?
4. How do you handle errors in async thunks with TypeScript?

### Coding Challenges

1. Create an async thunk for user login that handles network errors and validation errors separately.
2. Build a thunk that fetches paginated data and handles race conditions with the condition option.
3. Create a thunk that chains multiple API calls and accumulates errors.

### Related Topics

- createSlice extraReducers
- Middleware typing
- Redux state management patterns
