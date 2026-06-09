# Hooks Typing - useEffect, useRef, useMemo, useCallback, custom hooks typing

## Introduction

React Hooks revolutionized how developers manage component logic by enabling stateful and effectful behavior in function components. TypeScript adds a layer of safety to hooks by ensuring that dependencies are correctly specified, that refs are properly typed, that memoized values retain their types, and that custom hooks expose predictable interfaces. This file covers typing `useEffect`, `useRef` (including the `MutableRefObject` vs `RefObject` distinction), `useMemo`, `useCallback`, and patterns for typing custom hooks with generics.

## useEffect Typing

### What It Is

`useEffect` lets you synchronize a component with an external system (e.g., API calls, subscriptions, DOM manipulation). TypeScript's typing for `useEffect` focuses on ensuring that the effect function and its cleanup function are properly typed, and that the dependency array is correct through lint rules rather than type checking alone.

### Why It Is Important

While `useEffect`'s type signature is relatively simple, correct typing matters for the return value (the cleanup function) and for ensuring that values used inside the effect are properly captured. TypeScript helps catch errors like returning the wrong type from an effect or using values that may be stale.

### How It Works Internally

`useEffect` is declared in `@types/react` as:

```typescript
function useEffect(
  effect: EffectCallback,
  deps?: DependencyList
): void;

type EffectCallback = () => void | Destructor;
type Destructor = () => void | { [UNDEFINED_VOID_ONLY]: never };
type DependencyList = readonly unknown[];
```

The effect function must return either `void` or a cleanup function (a function that returns `void`). The dependency list is `readonly unknown[]`, meaning any array of values is accepted, but the `react-hooks/exhaustive-deps` ESLint rule enforces completeness.

### Syntax

```typescript
import React, { useEffect } from 'react';

// Basic effect
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);

// Effect with cleanup
useEffect(() => {
  const subscription = someAPI.subscribe(id, (data) => {
    setData(data);
  });
  return () => {
    subscription.unsubscribe();
  };
}, [id]);

// Effect without dependencies (runs once)
useEffect(() => {
  fetchInitialData();
}, []);
```

### Beginner Examples

```typescript
// Timer effect
const Timer = () => {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  }, []);

  return <div>Elapsed: {seconds}s</div>;
};

// Document title updater
const DocumentTitleUpdater = ({ title }: { title: string }) => {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = title;
    return () => {
      document.title = previousTitle;
    };
  }, [title]);

  return null;
};
```

### Intermediate Examples

```typescript
// Event listener with cleanup
const useWindowSize = () => {
  const [size, setSize] = useState({ width: window.innerWidth, height: window.innerHeight });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return size;
};

// Async effect pattern
const UserProfile = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;

    const fetchUser = async () => {
      try {
        setError(null);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const data: User = await response.json();
        if (!cancelled) {
          setUser(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err.message : 'Unknown error');
        }
      }
    };

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
};
```

### Advanced Examples

```typescript
// Custom hook with typed useEffect
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Effect with multiple subscriptions
function useWebSocket(url: string) {
  const [isConnected, setIsConnected] = useState(false);
  const [lastMessage, setLastMessage] = useState<MessageEvent | null>(null);

  useEffect(() => {
    const ws = new WebSocket(url);

    ws.onopen = () => setIsConnected(true);
    ws.onclose = () => setIsConnected(false);
    ws.onmessage = (event) => setLastMessage(event);

    return () => {
      ws.close();
    };
  }, [url]);

  return { isConnected, lastMessage };
}

// Effect with ref for cleanup
function useIntersectionObserver(
  elementRef: React.RefObject<HTMLElement>,
  options?: IntersectionObserverInit
) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);

    observer.observe(element);

    return () => {
      observer.disconnect();
    };
  }, [elementRef, options]);

  return isIntersecting;
}
```

### Real-World Use Cases

```typescript
// Analytics tracking effect
const usePageView = (path: string) => {
  useEffect(() => {
    analytics.track('page_view', { path });
  }, [path]);
};

// Keyboard shortcut handler
const useKeyboardShortcut = (
  key: string,
  callback: () => void,
  modifiers?: { ctrl?: boolean; shift?: boolean; alt?: boolean }
) => {
  useEffect(() => {
    const handler = (event: KeyboardEvent) => {
      if (
        event.key === key &&
        (!modifiers?.ctrl || event.ctrlKey) &&
        (!modifiers?.shift || event.shiftKey) &&
        (!modifiers?.alt || event.altKey)
      ) {
        event.preventDefault();
        callback();
      }
    };

    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, [key, callback, modifiers]);
};
```

### Common Mistakes

```typescript
// MISTAKE: Returning a Promise from useEffect
// useEffect(async () => { await fetch() }); // BAD - returns Promise, not cleanup

// CORRECT: Use an async IIFE inside
useEffect(() => {
  const fetchData = async () => {
    const result = await fetch('/api/data');
    // ...
  };
  fetchData();
}, []);

// MISTAKE: Missing dependencies
useEffect(() => {
  setCount(count + 1); // Missing `count` in deps
}, []); // BAD: stale closure

// MISTAKE: Including unnecessary dependencies
useEffect(() => {
  const interval = setInterval(() => setCount((c) => c + 1), 1000);
  return () => clearInterval(interval);
}, [setCount]); // setCount is stable, but doesn't need to be in deps
```

### Best Practices

- Always include all reactive values in the dependency array.
- Use the `exhaustive-deps` ESLint rule to enforce complete dependencies.
- For async operations, use the `cancelled` flag pattern to prevent state updates on unmounted components.
- Keep effects focused on a single concern; split complex logic into multiple `useEffect` calls.
- Return cleanup functions for subscriptions, timers, and event listeners.

### Performance Considerations

Effects run after every render by default. The dependency array controls re-execution. Omitting dependencies causes stale closures, while including unnecessary dependencies causes excessive re-runs. The type system doesn't enforce dependency completeness; this is the domain of lint rules.

### Interview Questions

1. How does `useEffect` determine when to re-run the effect?
2. What is the return type of the effect function in `useEffect`?
3. How do you properly type an async function inside `useEffect`?
4. Why does the `eslint-plugin-react-hooks` rule `exhaustive-deps` exist?

### Coding Challenges

1. Create a `useInterval` hook that accepts a callback and a delay, properly typed.
2. Implement a `useDocumentTitle` hook that restores the previous title on cleanup.
3. Build a `useOnlineStatus` hook that tracks network connectivity with proper effect cleanup.

### Related Topics

- useLayoutEffect
- useInsertionEffect
- Custom hooks
- ESLint rules for hooks

## useRef (MutableRefObject vs RefObject)

### What It Is

`useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument. The ref object persists for the lifetime of the component. TypeScript distinguishes between `MutableRefObject<T>` (when `.current` can be reassigned) and `RefObject<T>` (when `.current` is read-only, typically for DOM refs).

### Why It Is Important

Choosing the correct ref type prevents bugs. A DOM ref should not have its `.current` property reassigned after attachment (React manages this). A mutable ref, used for storing arbitrary values, must allow reassignment. Using the wrong type can either prevent legitimate mutations or allow incorrect DOM manipulations.

### How It Works Internally

`useRef` has three overloads in `@types/react`:

```typescript
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

- When initialized with a non-null value, returns `MutableRefObject<T>`.
- When initialized with `null` (commonly for DOM refs), returns `RefObject<T>` where `.current` is read-only.
- When called with no arguments, returns `MutableRefObject<T | undefined>`.

`MutableRefObject<T>` has `{ current: T }` (writable), while `RefObject<T>` has `{ readonly current: T | null }` (read-only).

### Syntax

```typescript
import React, { useRef } from 'react';

// DOM ref (read-only)
const inputRef = useRef<HTMLInputElement>(null);
// Type: RefObject<HTMLInputElement>

// Mutable ref for values
const countRef = useRef(0);
// Type: MutableRefObject<number>

// Mutable ref with explicit type
const dataRef = useRef<string[]>([]);
// Type: MutableRefObject<string[]>
```

### Beginner Examples

```typescript
// DOM focus management
const AutoFocusInput = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} type="text" />;
};

// Storing interval ID
const TimerWithRef = () => {
  const intervalRef = useRef<number | null>(null);

  const startTimer = () => {
    intervalRef.current = window.setInterval(() => {
      console.log('Tick');
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current !== null) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  return (
    <div>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
};

// Previous value tracker
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```

### Intermediate Examples

```typescript
// Video player with refs
const VideoPlayer = ({ src }: { src: string }) => {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [isPlaying, setIsPlaying] = useState(false);

  const play = () => {
    videoRef.current?.play();
    setIsPlaying(true);
  };

  const pause = () => {
    videoRef.current?.pause();
    setIsPlaying(false);
  };

  const skip = (seconds: number) => {
    if (videoRef.current) {
      videoRef.current.currentTime += seconds;
    }
  };

  return (
    <div>
      <video ref={videoRef} src={src} />
      <button onClick={isPlaying ? pause : play}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <button onClick={() => skip(-10)}>-10s</button>
      <button onClick={() => skip(10)}>+10s</button>
    </div>
  );
};

// Canvas ref with typed context
const Canvas = () => {
  const canvasRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;

    const ctx = canvas.getContext('2d');
    if (!ctx) return;

    // ctx is CanvasRenderingContext2D
    ctx.fillStyle = 'red';
    ctx.fillRect(0, 0, 100, 100);
  }, []);

  return <canvas ref={canvasRef} width={200} height={200} />;
};
```

### Advanced Examples

```typescript
// Forward ref with typing
interface FancyInputProps {
  label: string;
  error?: string;
}

const FancyInput = React.forwardRef<HTMLInputElement, FancyInputProps>(
  ({ label, error }, ref) => {
    return (
      <div className="fancy-input">
        <label>{label}</label>
        <input ref={ref} className={error ? 'error' : ''} />
        {error && <span className="error-message">{error}</span>}
      </div>
    );
  }
);

FancyInput.displayName = 'FancyInput';

// Parent usage
const Form = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return (
    <form>
      <FancyInput ref={inputRef} label="Username" />
      <button type="submit">Submit</button>
    </form>
  );
};

// Callback ref pattern
function useCallbackRef<T extends HTMLElement>(
  callback: (element: T | null) => void
) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  return useCallback((element: T | null) => {
    callbackRef.current(element);
  }, []);
}

// Ref for measuring element dimensions
function useDimensions<T extends HTMLElement>() {
  const [dimensions, setDimensions] = useState<{ width: number; height: number } | null>(null);
  const observedRef = useRef<T>(null);

  useEffect(() => {
    const element = observedRef.current;
    if (!element) return;

    const observer = new ResizeObserver((entries) => {
      const { width, height } = entries[0].contentRect;
      setDimensions({ width, height });
    });

    observer.observe(element);
    return () => observer.disconnect();
  }, []);

  return [observedRef, dimensions] as const;
}
```

### Real-World Use Cases

```typescript
// Infinite scroll with intersection observer
function useInfiniteScroll(onLoadMore: () => Promise<void>) {
  const sentinelRef = useRef<HTMLDivElement>(null);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    const sentinel = sentinelRef.current;
    if (!sentinel) return;

    const observer = new IntersectionObserver(
      async (entries) => {
        if (entries[0].isIntersecting && !isLoading) {
          setIsLoading(true);
          await onLoadMore();
          setIsLoading(false);
        }
      },
      { threshold: 0.1 }
    );

    observer.observe(sentinel);
    return () => observer.disconnect();
  }, [onLoadMore, isLoading]);

  return sentinelRef;
}

// Drag and drop with refs
function useDrag<T extends HTMLElement>() {
  const elementRef = useRef<T>(null);
  const [isDragging, setIsDragging] = useState(false);
  const positionRef = useRef({ x: 0, y: 0 });
  const offsetRef = useRef({ x: 0, y: 0 });

  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const handleMouseDown = (e: MouseEvent) => {
      offsetRef.current = {
        x: e.clientX - positionRef.current.x,
        y: e.clientY - positionRef.current.y,
      };
      setIsDragging(true);
    };

    element.addEventListener('mousedown', handleMouseDown);
    return () => element.removeEventListener('mousedown', handleMouseDown);
  }, []);

  useEffect(() => {
    if (!isDragging) return;

    const handleMouseMove = (e: MouseEvent) => {
      positionRef.current = {
        x: e.clientX - offsetRef.current.x,
        y: e.clientY - offsetRef.current.y,
      };
      if (elementRef.current) {
        elementRef.current.style.transform = `translate(${positionRef.current.x}px, ${positionRef.current.y}px)`;
      }
    };

    const handleMouseUp = () => setIsDragging(false);

    window.addEventListener('mousemove', handleMouseMove);
    window.addEventListener('mouseup', handleMouseUp);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
      window.removeEventListener('mouseup', handleMouseUp);
    };
  }, [isDragging]);

  return elementRef;
}
```

### Common Mistakes

```typescript
// MISTAKE: Trying to reassign RefObject.current
// const ref = useRef<HTMLDivElement>(null);
// ref.current = document.createElement('div'); // Error: cannot assign to readonly

// MISTAKE: Not handling null check on ref.current
// ref.current.focus(); // Error: Object is possibly null
// CORRECT: ref.current?.focus();

// MISTAKE: Using RefObject when MutableRefObject is needed
// const countRef = useRef(0);
// countRef.current = 5; // Works with MutableRefObject

// MISTAKE: Passing ref to custom component without forwardRef
// const Component = () => { return <div />; }
// <Component ref={myRef} /> // Error: ref is not a prop

// CORRECT: Use React.forwardRef
```

### Best Practices

- Use `useRef<T>(null)` for DOM refs (returns `RefObject<T>`).
- Use `useRef(initialValue)` for mutable values (returns `MutableRefObject<T>`).
- Always check `ref.current` for null before accessing DOM methods.
- Use `React.forwardRef` to pass refs to custom components.
- Prefer callback refs for dynamic ref assignment when needed.
- Use refs for values that should not trigger re-renders when changed.

### Performance Considerations

Refs are essential for performance because changing `ref.current` does not cause re-renders. This makes them ideal for storing values that change frequently (e.g., scroll positions, interval IDs, animation frame IDs). The ref object itself is stable across renders.

### Interview Questions

1. What is the difference between `MutableRefObject` and `RefObject`?
2. How does TypeScript determine which ref type to return from `useRef`?
3. How do you type a forwarded ref with `React.forwardRef`?
4. When would you use a callback ref instead of `useRef`?

### Coding Challenges

1. Implement a `useEventListener` hook that uses a ref to store the handler and attaches/detaches the event listener.
2. Create a `useHover` hook that returns a ref and a boolean indicating whether the element is hovered.
3. Build a `usePrevious` hook that tracks the previous value of a state or prop, properly typed.

### Related Topics

- ForwardRef
- Callback refs
- DOM type declarations in TypeScript

## useMemo and useCallback

### What It Is

`useMemo` memoizes the result of a computation, recomputing only when dependencies change. `useCallback` memoizes a function, returning the same function reference across renders unless dependencies change. Both optimize performance by preventing unnecessary recalculations and re-renders.

### Why It Is Important

Typing `useMemo` and `useCallback` ensures that the memoized value or function has the correct type, and that the factory function or callback receives properly typed arguments. TypeScript infers the return type from the factory function, making explicit annotations often unnecessary.

### How It Works Internally

```typescript
function useMemo<T>(factory: () => T, deps: DependencyList): T;
function useCallback<T extends Function>(callback: T, deps: DependencyList): T;
```

`useMemo` infers `T` from the return type of the factory function. `useCallback` preserves the exact function type `T`, including its parameter and return types.

### Syntax

```typescript
import React, { useMemo, useCallback } from 'react';

// useMemo
const sortedList = useMemo(() => {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}, [items]);

// useCallback
const handleClick = useCallback((id: string) => {
  setSelectedId(id);
}, []);
```

### Beginner Examples

```typescript
// Memoizing a filtered list
const TodoList = ({ todos, filter }: { todos: Todo[]; filter: string }) => {
  const filteredTodos = useMemo(() => {
    return todos.filter((todo) =>
      todo.title.toLowerCase().includes(filter.toLowerCase())
    );
  }, [todos, filter]);

  return (
    <ul>
      {filteredTodos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
};

// Memoizing a callback for child component
const Parent = () => {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []);

  return <Child onIncrement={increment} />;
};
```

### Intermediate Examples

```typescript
// Complex computation
const useStatistics = (data: number[]) => {
  const stats = useMemo(() => {
    const sorted = [...data].sort((a, b) => a - b);
    const sum = data.reduce((acc, val) => acc + val, 0);
    const mean = sum / data.length;
    const median = sorted.length % 2 === 0
      ? (sorted[sorted.length / 2 - 1] + sorted[sorted.length / 2]) / 2
      : sorted[Math.floor(sorted.length / 2)];
    const variance = data.reduce((acc, val) => acc + (val - mean) ** 2, 0) / data.length;
    const stdDev = Math.sqrt(variance);
    return { sum, mean, median, variance, stdDev, min: sorted[0], max: sorted[sorted.length - 1] };
  }, [data]);

  return stats;
};

// Callback with stable reference
const SearchComponent = ({ onSearch }: { onSearch: (query: string) => Promise<Result[]> }) => {
  const [query, setQuery] = useState('');

  const debouncedSearch = useCallback(
    debounce(async (q: string) => {
      const results = await onSearch(q);
      setResults(results);
    }, 300),
    [onSearch]
  );

  const handleChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
    debouncedSearch(e.target.value);
  }, [debouncedSearch]);

  return <input value={query} onChange={handleChange} />;
};
```

### Advanced Examples

```typescript
// useMemo with generic factory
function useCreate<T>(factory: () => T): T {
  return useMemo(factory, []);
}

// Memoizing JSX
const TableSection = ({ rows, columns }: TableProps) => {
  const headerRow = useMemo(() => (
    <tr>
      {columns.map((col) => <th key={col.key}>{col.header}</th>)}
    </tr>
  ), [columns]);

  const bodyRows = useMemo(() => (
    rows.map((row) => (
      <tr key={row.id}>
        {columns.map((col) => <td key={col.key}>{row[col.key]}</td>)}
      </tr>
    ))
  ), [rows, columns]);

  return (
    <table>
      <thead>{headerRow}</thead>
      <tbody>{bodyRows}</tbody>
    </table>
  );
};

// Callback with strict typing
interface Callbacks {
  onAdd: (item: Item) => void;
  onRemove: (id: string) => void;
  onUpdate: (id: string, changes: Partial<Item>) => void;
}

function useItemManager(items: Item[]): { items: Item[]; callbacks: Callbacks } {
  const [state, setState] = useState(items);

  const callbacks = useMemo<Callbacks>(() => ({
    onAdd: (item) => setState((prev) => [...prev, item]),
    onRemove: (id) => setState((prev) => prev.filter((i) => i.id !== id)),
    onUpdate: (id, changes) =>
      setState((prev) =>
        prev.map((i) => (i.id === id ? { ...i, ...changes } : i))
      ),
  }), []);

  return { items: state, callbacks };
}

// Avoiding useCallback for simple cases
// Instead of:
const handleClick = useCallback(() => doSomething(id), [id, doSomething]);
// Consider if the child is memoized. If not, useCallback adds overhead.
```

### Real-World Use Cases

```typescript
// Chart data processing
const useChartData = (rawData: DataPoint[], timeRange: TimeRange) => {
  const filteredData = useMemo(() => {
    const now = Date.now();
    const range = timeRange === '1h' ? 3600000
      : timeRange === '24h' ? 86400000
      : 604800000;
    return rawData.filter((point) => now - point.timestamp < range);
  }, [rawData, timeRange]);

  const aggregatedData = useMemo(() => {
    const grouped = new Map<string, DataPoint[]>();
    filteredData.forEach((point) => {
      const key = new Date(point.timestamp).toISOString().slice(0, 13);
      const group = grouped.get(key) || [];
      group.push(point);
      grouped.set(key, group);
    });
    return Array.from(grouped.entries()).map(([hour, points]) => ({
      hour,
      avg: points.reduce((s, p) => s + p.value, 0) / points.length,
      max: Math.max(...points.map((p) => p.value)),
      min: Math.min(...points.map((p) => p.value)),
    }));
  }, [filteredData]);

  return aggregatedData;
};

// Form submission handler
const useFormSubmit = <T extends Record<string, unknown>>(
  schema: ZodSchema<T>,
  onSubmit: (data: T) => Promise<void>
) => {
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = useCallback(
    async (rawData: unknown) => {
      setIsSubmitting(true);
      setError(null);
      try {
        const parsed = schema.parse(rawData);
        await onSubmit(parsed);
      } catch (err) {
        if (err instanceof ZodError) {
          setError(err.errors.map((e) => e.message).join(', '));
        } else {
          setError('An unexpected error occurred');
        }
      } finally {
        setIsSubmitting(false);
      }
    },
    [schema, onSubmit]
  );

  return { handleSubmit, isSubmitting, error };
};
```

### Common Mistakes

```typescript
// MISTAKE: Using useMemo for simple values
const value = useMemo(() => 42, []); // Overhead > benefit

// MISTAKE: Missing dependencies
const fullName = useMemo(() => `${firstName} ${lastName}`, []); // BAD: missing firstName, lastName

// MISTAKE: Unnecessary useCallback
// If the child is not memoized, useCallback has no benefit
const handleChange = useCallback((e) => setValue(e.target.value), []);

// MISTAKE: Inline function in JSX with memoized child
<MemoizedChild onClick={() => doSomething(id)} /> // BAD: new function every render
// CORRECT: const handleClick = useCallback(() => doSomething(id), [id, doSomething]);
```

### Best Practices

- Profile before optimizing: only add `useMemo`/`useCallback` when there is a measured performance issue.
- Use `useMemo` for expensive computations (filtering, sorting, transforming large arrays).
- Use `useCallback` for callbacks passed to memoized children or as effect dependencies.
- Always include all reactive values in the dependency array.
- Prefer `useMemo<Type>(...)` for explicit return type when inference isn't clear.

### Performance Considerations

`useMemo` and `useCallback` have overhead: the hook itself costs memory and the dependency comparison costs time. Overusing them can degrade performance. React compares dependencies with `Object.is`, which is cheap but not free. Only use these hooks when the computation cost outweighs the comparison cost.

### Interview Questions

1. What is the difference between `useMemo` and `useCallback`?
2. How does TypeScript infer the return type of `useMemo`?
3. When is it appropriate to use `useCallback`?
4. How do `useMemo` and `useCallback` help with referential equality?

### Coding Challenges

1. Create a `useSorted` hook that memoizes a sorted array with a custom comparator.
2. Implement a `useAsyncCallback` hook that wraps an async function and returns a memoized callback with loading/error state.
3. Build a `useFormValidator` hook that memoizes validation functions based on a schema.

### Related Topics

- React.memo
- Referential equality
- Performance optimization in React

## Custom Hooks Generics

### What It Is

Custom hooks are JavaScript functions that use React hooks internally. TypeScript generics allow custom hooks to be type-safe while working with any data type. By making a custom hook generic, it becomes reusable across different contexts without sacrificing type safety.

### Why It Is Important

Generic custom hooks eliminate the need for type assertions or `any` in reusable hook logic. A hook like `useLocalStorage<T>` preserves the type of stored data, and `useApi<T>` ensures that fetched data matches the expected shape.

### How It Works Internally

Generic hooks follow the same type parameter rules as any generic TypeScript function. The type parameter is typically inferred from the arguments, though it can also be explicitly specified. The generic parameter flows through the internal `useState`, `useEffect`, and other hooks.

### Syntax

```typescript
// Basic generic hook
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    const item = window.localStorage.getItem(key);
    return item ? JSON.parse(item) as T : initialValue;
  });

  const setValue = (value: T | ((prev: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    window.localStorage.setItem(key, JSON.stringify(valueToStore));
  };

  return [storedValue, setValue];
}
```

### Beginner Examples

```typescript
// Generic toggle hook
function useToggle<T>(initialValue: T, alternateValue: T): [T, () => void] {
  const [value, setValue] = useState(initialValue);
  const toggle = useCallback(() => {
    setValue((prev) => (prev === initialValue ? alternateValue : initialValue));
  }, [initialValue, alternateValue]);

  return [value, toggle];
}

// Usage
const [theme, toggleTheme] = useToggle<'light' | 'dark'>('light', 'dark');

// Generic counter hook
function useCounter<T extends number>(initialValue: T) {
  const [count, setCount] = useState(initialValue);
  const increment = useCallback(() => setCount((c) => c + 1), []);
  const decrement = useCallback(() => setCount((c) => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset } as const;
}
```

### Intermediate Examples

```typescript
// Generic form field hook
function useFormField<T>(initialValue: T) {
  const [value, setValue] = useState<T>(initialValue);
  const [error, setError] = useState<string | null>(null);
  const [touched, setTouched] = useState(false);

  const onChange = useCallback((newValue: T) => {
    setValue(newValue);
    setError(null);
  }, []);

  const onBlur = useCallback(() => {
    setTouched(true);
  }, []);

  const reset = useCallback(() => {
    setValue(initialValue);
    setError(null);
    setTouched(false);
  }, [initialValue]);

  return { value, error, touched, onChange, onBlur, reset, setError };
}

// Generic API hook
interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

function useApi<T>(url: string, options?: RequestInit): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [refreshKey, setRefreshKey] = useState(0);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);

    fetch(url, options)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json() as Promise<T>;
      })
      .then((result) => {
        if (!cancelled) {
          setData(result);
          setError(null);
        }
      })
      .catch((err) => {
        if (!cancelled) {
          setError(err.message);
          setData(null);
        }
      })
      .finally(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; };
  }, [url, refreshKey, ...(options ? [options] : [])]);

  const refetch = useCallback(() => setRefreshKey((k) => k + 1), []);

  return { data, loading, error, refetch };
}
```

### Advanced Examples

```typescript
// Generic hook with multiple type parameters
function useMap<K, V>(initialEntries?: readonly (readonly [K, V])[]) {
  const [map, setMap] = useState(() => new Map(initialEntries));

  const set = useCallback((key: K, value: V) => {
    setMap((prev) => {
      const next = new Map(prev);
      next.set(key, value);
      return next;
    });
  }, []);

  const get = useCallback((key: K) => map.get(key), [map]);

  const remove = useCallback((key: K) => {
    setMap((prev) => {
      const next = new Map(prev);
      next.delete(key);
      return next;
    });
  }, []);

  const clear = useCallback(() => setMap(new Map()), []);

  const has = useCallback((key: K) => map.has(key), [map]);

  return { map: map as ReadonlyMap<K, V>, set, get, remove, clear, has };
}

// Generic hook with constrained type parameter
interface Identifiable {
  id: string | number;
}

function useSelection<T extends Identifiable>(items: T[]) {
  const [selectedIds, setSelectedIds] = useState<Set<string | number>>(new Set());

  const toggle = useCallback((item: T) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(item.id)) {
        next.delete(item.id);
      } else {
        next.add(item.id);
      }
      return next;
    });
  }, []);

  const selectAll = useCallback(() => {
    setSelectedIds(new Set(items.map((item) => item.id)));
  }, [items]);

  const deselectAll = useCallback(() => {
    setSelectedIds(new Set());
  }, []);

  const isSelected = useCallback((item: T) => selectedIds.has(item.id), [selectedIds]);

  const selectedItems = useMemo(
    () => items.filter((item) => selectedIds.has(item.id)),
    [items, selectedIds]
  );

  return { selectedIds, selectedItems, toggle, selectAll, deselectAll, isSelected };
}

// Generic hook returning multiple refs
function useMediaStream<T extends MediaStream>() {
  const [stream, setStream] = useState<T | null>(null);
  const [error, setError] = useState<string | null>(null);
  const videoRef = useRef<HTMLVideoElement>(null);

  const start = useCallback(async (constraints?: MediaStreamConstraints) => {
    try {
      const mediaStream = await navigator.mediaDevices.getUserMedia(constraints) as T;
      setStream(mediaStream);
      if (videoRef.current) {
        videoRef.current.srcObject = mediaStream;
      }
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to access media');
    }
  }, []);

  const stop = useCallback(() => {
    if (stream) {
      stream.getTracks().forEach((track) => track.stop());
      setStream(null);
    }
  }, [stream]);

  useEffect(() => {
    return () => {
      if (stream) {
        stream.getTracks().forEach((track) => track.stop());
      }
    };
  }, [stream]);

  return { stream, error, videoRef, start, stop };
}

// Generic hook with tuple return type using `as const`
function useStep<T extends string>(steps: readonly T[], initialStep?: T) {
  const [currentStep, setCurrentStep] = useState<T>(initialStep ?? steps[0]);
  const currentIndex = steps.indexOf(currentStep);

  const next = useCallback(() => {
    const nextIndex = currentIndex + 1;
    if (nextIndex < steps.length) {
      setCurrentStep(steps[nextIndex]);
    }
  }, [currentIndex, steps]);

  const prev = useCallback(() => {
    const prevIndex = currentIndex - 1;
    if (prevIndex >= 0) {
      setCurrentStep(steps[prevIndex]);
    }
  }, [currentIndex, steps]);

  const goTo = useCallback((step: T) => {
    if (steps.includes(step)) {
      setCurrentStep(step);
    }
  }, [steps]);

  const isFirst = currentIndex === 0;
  const isLast = currentIndex === steps.length - 1;

  return { currentStep, currentIndex, next, prev, goTo, isFirst, isLast, steps } as const;
}

// Usage
const { currentStep, next, prev, isFirst, isLast } = useStep(['shipping', 'payment', 'review'] as const);
```

### Real-World Use Cases

```typescript
// Generic pagination hook
function usePagination<T>(items: T[], pageSize: number) {
  const [currentPage, setCurrentPage] = useState(1);
  const totalPages = Math.ceil(items.length / pageSize);

  const pageItems = useMemo(
    () => items.slice((currentPage - 1) * pageSize, currentPage * pageSize),
    [items, currentPage, pageSize]
  );

  const goToPage = useCallback((page: number) => {
    setCurrentPage(Math.max(1, Math.min(page, totalPages)));
  }, [totalPages]);

  const nextPage = useCallback(() => goToPage(currentPage + 1), [currentPage, goToPage]);
  const prevPage = useCallback(() => goToPage(currentPage - 1), [currentPage, goToPage]);

  return { pageItems, currentPage, totalPages, goToPage, nextPage, prevPage } as const;
}

// Generic undo/redo hook
function useUndoRedo<T>(initial: T) {
  const [past, setPast] = useState<T[]>([]);
  const [present, setPresent] = useState<T>(initial);
  const [future, setFuture] = useState<T[]>([]);

  const set = useCallback((newPresent: T) => {
    setPast((prev) => [...prev, present]);
    setPresent(newPresent);
    setFuture([]);
  }, [present]);

  const undo = useCallback(() => {
    if (past.length === 0) return;
    const previous = past[past.length - 1];
    setPast((prev) => prev.slice(0, -1));
    setFuture((prev) => [present, ...prev]);
    setPresent(previous);
  }, [past, present]);

  const redo = useCallback(() => {
    if (future.length === 0) return;
    const next = future[0];
    setFuture((prev) => prev.slice(1));
    setPast((prev) => [...prev, present]);
    setPresent(next);
  }, [future, present]);

  const reset = useCallback((newInitial: T) => {
    setPast([]);
    setPresent(newInitial);
    setFuture([]);
  }, []);

  return {
    state: present,
    set,
    undo,
    redo,
    reset,
    canUndo: past.length > 0,
    canRedo: future.length > 0,
  };
}
```

### Common Mistakes

```typescript
// MISTAKE: Not providing a type parameter when the initial value is too broad
// const [value] = useLocalStorage('key', null); // T is inferred as null
// CORRECT: useLocalStorage<string | null>('key', null);

// MISTAKE: Using `any` instead of a generic
// function useStorage(key: string, initial: any) { ... } // BAD

// MISTAKE: Not constraining the generic when needed
// function useSorter<T>(items: T[]) { ... } // T is unconstrained
// function useSorter<T extends { id: string }>(items: T[]) { ... } // Better if we need id

// MISTAKE: Overly complex generics that confuse inference
```

### Best Practices

- Use generics when the hook works with multiple data types.
- Constrain type parameters with `extends` when the hook needs specific properties.
- Provide default type parameters where sensible.
- Use `as const` for tuple return types to preserve literal types.
- Document the type parameter expectations in comments.
- Prefer inference over explicit type annotations for ergonomics.

### Performance Considerations

Generic type parameters are erased at compile time and have zero runtime cost. However, complex generic type definitions can slow down TypeScript's type checker. Keep generic constraints simple and avoid deeply nested conditional types inside hooks.

### Interview Questions

1. How do you create a generic custom hook in React with TypeScript?
2. When should you constrain a generic type parameter with `extends`?
3. How does `as const` help with typing custom hooks that return tuples?
4. What is the difference between generic hooks and hooks that return `any`?

### Coding Challenges

1. Create a generic `useQueue<T>` hook that implements a FIFO queue with `enqueue`, `dequeue`, `peek`, and `size`.
2. Implement a generic `useAsyncReducer<T, A>` that wraps `useReducer` with async action dispatching.
3. Build a generic `useTreeData<T>` hook for managing tree structures with `addChild`, `removeNode`, and `moveNode` operations.

### Related Topics

- TypeScript generics
- useState typing
- useReducer typing
- Custom hooks patterns
