# React Components - FC type, function components, JSX types, component props typing

## Introduction

React components are the fundamental building blocks of any React application. In TypeScript-powered React projects, components benefit from static type checking, which catches prop mismatches, incorrect state usage, and missing handlers at compile time rather than at runtime. TypeScript provides several ways to type components, including the `React.FC` (or `React.FunctionComponent`) type, function component syntax with explicit return types, JSX type representations like `JSX.Element` and `React.ReactNode`, and robust prop typing patterns. Mastering these concepts ensures that your React codebase remains maintainable, self-documenting, and resilient to refactoring.

## FC (FunctionComponent) Type

### What It Is

`React.FC` (also available as `React.FunctionComponent`) is a generic type provided by `@types/react` that describes a function component. It accepts a props type parameter and automatically includes `children` in the props definition.

```typescript
import React from 'react';

type GreetingProps = {
  name: string;
};

const Greeting: React.FC<GreetingProps> = ({ name }) => {
  return <div>Hello, {name}!</div>;
};
```

### Why It Is Important

`React.FC` offers a concise way to annotate function components, making the intent clear. It also handles edge cases like returning `null` or `undefined` correctly, as it knows the component's return type is `React.ReactElement | null`. However, it has fallen out of favor in many modern codebases because it adds an implicit `children` prop, which may not always be desired.

### How It Works Internally

The `@types/react` package defines `React.FC` as:

```typescript
type FC<P = {}> = FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  (props: P, context?: any): ReactElement<any, any> | null;
  displayName?: string;
  contextTypes?: ValidationMap<any>;
  propTypes?: WeakValidationMap<P>;
  defaultProps?: Partial<P>;
}
```

This means a value of type `React.FC<Props>` is a callable function that takes `props` (and optionally `context`) and returns either a `ReactElement` or `null`. The interface also includes optional static properties like `displayName`, `propTypes`, and `defaultProps`.

### Syntax

```typescript
// Basic FC
const Header: React.FC = () => <header>Logo</header>;

// FC with props
type Props = { title: string; count: number };
const Item: React.FC<Props> = ({ title, count }) => (
  <div>{title}: {count}</div>
);

// Without FC (preferred in many modern codebases)
const ItemAlt = ({ title, count }: Props): JSX.Element => (
  <div>{title}: {count}</div>
);
```

### Beginner Examples

```typescript
// Simple component using FC
type WelcomeProps = {
  username: string;
};

const Welcome: React.FC<WelcomeProps> = ({ username }) => {
  return <h1>Welcome back, {username}!</h1>;
};

// Usage
<Welcome username="alice" />
```

### Intermediate Examples

```typescript
// Component with optional props and default values
type CardProps = {
  title: string;
  description?: string;
  variant?: 'primary' | 'secondary';
};

const Card: React.FC<CardProps> = ({
  title,
  description,
  variant = 'primary'
}) => {
  const className = `card card--${variant}`;
  return (
    <div className={className}>
      <h2>{title}</h2>
      {description && <p>{description}</p>}
    </div>
  );
};
```

### Advanced Examples

```typescript
// Generic component using FC pattern
type ListProps<T> = {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
};

const List: React.FC<ListProps<any>> = ({ items, renderItem }) => {
  return <ul>{items.map(renderItem)}</ul>;
};

// More type-safe generic component (without FC)
function TypedList<T>({ items, renderItem }: ListProps<T>): JSX.Element {
  return <ul>{items.map(renderItem)}</ul>;
}
```

### Real-World Use Cases

In large-scale applications, typing components prevents entire categories of bugs. For example, a design system component library uses `React.FC` to ensure every component exposes a consistent interface:

```typescript
// Design system button
type ButtonProps = {
  label: string;
  onClick: () => void;
  variant: 'solid' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  fullWidth?: boolean;
};

const Button: React.FC<ButtonProps> = ({
  label,
  onClick,
  variant,
  size = 'md',
  disabled = false,
  fullWidth = false,
}) => {
  return (
    <button
      className={`btn btn--${variant} btn--${size} ${fullWidth ? 'btn--full' : ''}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

### Common Mistakes

```typescript
// MISTAKE: Assuming FC adds no children
// The following accepts children even though the type doesn't define them
type TitleProps = { text: string };
const Title: React.FC<TitleProps> = ({ text }) => <h1>{text}</h1>;

// This compiles without error, which may be unintended
<Title text="Hi"><span>extra</span></Title>

// BETTER: Explicitly omit children or avoid FC
const TitleAlt = ({ text }: TitleProps): JSX.Element => <h1>{text}</h1>;
// Now <TitleAlt text="Hi"><span>extra</span></Title> would be an error
```

### Best Practices

- Prefer explicit prop typing over `React.FC` in new codebases for better control over `children`.
- Use `React.FC` in legacy codebases for consistency if the pattern is already established.
- Always provide meaningful names to prop types instead of using inline types.
- Use `interface` for public API prop types that may be extended, `type` for internal or union-based props.

### Performance Considerations

`React.FC` has no runtime performance impact; it is purely a compile-time annotation. However, typing components correctly can prevent expensive React re-renders by catching unnecessary prop changes early:

```typescript
type ExpensiveListProps = {
  data: readonly Item[];
  onSelect: (id: string) => void;
};

// Correct typing helps useMemo and useCallback work correctly
const ExpensiveList: React.FC<ExpensiveListProps> = React.memo(({ data, onSelect }) => {
  return data.map(item => (
    <div key={item.id} onClick={() => onSelect(item.id)}>{item.name}</div>
  ));
});
```

### Interview Questions

1. What is the difference between `React.FC` and a regular function component with typed props?
2. Why did the React team advise against using `React.FC` in newer versions?
3. How does `React.FC` handle the `children` prop implicitly?
4. What is the return type of a function component that can return `null`?

### Coding Challenges

1. Create a generic `DataFetcher` component using `React.FC` that accepts a `url` prop and a `render` prop (children as function).
2. Type a `FormField` component that accepts `name`, `label`, `type` (union of input types), and renders the appropriate input element.

### Related Topics

- JSX.Element vs ReactNode vs ReactElement
- Component props typing
- Generic components
- Higher-order components typing

## Function Component Syntax

### What It Is

Function component syntax refers to writing React components as plain JavaScript/TypeScript functions rather than ES6 classes. TypeScript enhances this pattern by allowing explicit type annotations on props, return types, and internal state.

### Why It Is Important

Function components are the modern standard for React development. They are simpler, easier to test, and work seamlessly with React Hooks. TypeScript makes them robust by enforcing prop contracts and return types.

### How It Works Internally

TypeScript compiles function components to JavaScript functions. The JSX returned is transpiled to `React.createElement` calls. TypeScript's JSX support ensures that the returned values conform to expected types.

### Syntax

```typescript
// Arrow function
const MyComponent: React.FC<Props> = (props) => JSX;

// Named function
function MyComponent(props: Props): JSX.Element { return JSX; }

// Inline type
const MyComponent = ({ name, age }: { name: string; age: number }) => JSX;
```

### Beginner Examples

```typescript
const Greeting = ({ name }: { name: string }) => {
  return <p>Hello, {name}!</p>;
};
```

### Intermediate Examples

```typescript
const UserCard = ({ user, onEdit }: UserCardProps) => {
  const [expanded, setExpanded] = useState(false);
  return (
    <div onClick={() => setExpanded(!expanded)}>
      <h3>{user.name}</h3>
      {expanded && <p>{user.bio}</p>}
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
};
```

### Advanced Examples

```typescript
function withAuth<T extends object>(
  Component: React.ComponentType<T>
): React.FC<T & { isAuthenticated: boolean }> {
  return ({ isAuthenticated, ...props }) => {
    if (!isAuthenticated) return <Redirect to="/login" />;
    return <Component {...(props as T)} />;
  };
}
```

### Real-World Use Cases

Large enterprise applications standardize on function components with TypeScript to enforce consistency across teams. The type system serves as living documentation.

### Common Mistakes

```typescript
// MISTAKE: Not typing return value
const Component = () => {
  // TypeScript infers return type, but explicit annotation catches errors
  return null; // or JSX.Element
};
```

### Best Practices

- Prefer named function declarations for root-level components (better stack traces).
- Use arrow functions for inline or anonymous components.
- Always type props explicitly, never rely on inference for boundaries.

### Performance Considerations

Function components without `React.memo` re-render whenever their parent re-renders. Proper typing enables effective use of `React.memo`, `useMemo`, and `useCallback`.

### Interview Questions

1. What is the difference between function and class components in React with TypeScript?
2. How do you type the return of a function component?
3. Can a function component return `undefined`? How does TypeScript handle this?

### Coding Challenges

1. Convert a class component with complex state to a typed function component.
2. Write a higher-order component that logs props and renders the wrapped component, fully typed.

### Related Topics

- FC type vs function component
- Component lifecycle with hooks
- React.memo typing

## JSX.Element and ReactNode Types

### What It Is

`JSX.Element` and `React.ReactNode` are two commonly confused types in React TypeScript. `JSX.Element` represents the return type of a React component (a single React element), while `React.ReactNode` is broader and includes strings, numbers, booleans, `null`, `undefined`, fragments, and arrays of these.

### Why It Is Important

Choosing the correct type prevents type errors when rendering content. A component that returns `ReactNode` is more flexible than one that returns `JSX.Element`, but may require additional handling.

### How It Works Internally

`JSX.Element` is defined in TypeScript's JSX namespace:

```typescript
declare namespace JSX {
  interface Element {
    type: any;
    props: any;
    key: string | null;
  }
}
```

`React.ReactNode` is defined in React types as:

```typescript
type ReactNode = ReactElement | string | number | ReactFragment | ReactPortal | boolean | null | undefined;
```

### Syntax

```typescript
import React, { ReactNode } from 'react';

type ContainerProps = {
  children: ReactNode;
};

const Container = ({ children }: ContainerProps): JSX.Element => {
  return <div>{children}</div>;
};
```

### Beginner Examples

```typescript
type MessageProps = {
  content: ReactNode;
};
const Message = ({ content }: MessageProps) => <div>{content}</div>;
// Accepts: <Message content={<span>Hi</span>} />
// Accepts: <Message content="Hello" />
// Accepts: <Message content={42} />
```

### Intermediate Examples

```typescript
type TabProps = {
  title: ReactNode;
  children: ReactNode;
  disabled?: boolean;
};

const Tab = ({ title, children, disabled }: TabProps) => (
  <div className={`tab ${disabled ? 'tab--disabled' : ''}`}>
    <div className="tab__header">{title}</div>
    {!disabled && <div className="tab__body">{children}</div>}
  </div>
);
```

### Advanced Examples

```typescript
type RenderPropProps = {
  render: (value: number) => ReactNode;
};

const NumberDisplay = ({ render }: RenderPropProps) => {
  const [count, setCount] = useState(0);
  return <div onClick={() => setCount(c => c + 1)}>{render(count)}</div>;
};

// Usage
<NumberDisplay render={(n) => <strong>Count: {n}</strong>} />
```

### Real-World Use Cases

Modal components, layout wrappers, and polymorphic components all benefit from using `ReactNode` for maximum flexibility in what they render.

### Common Mistakes

```typescript
// MISTAKE: Using JSX.Element when ReactNode is needed
// const Wrap = ({ children }: { children: JSX.Element }) => ...
// This fails if children is a string

// CORRECT
const Wrap = ({ children }: { children: ReactNode }) => ...
```

### Best Practices

- Use `ReactNode` for children and render props.
- Use `JSX.Element` for component return types that must be a single element.
- Use `ReactElement` when you need to clone the element or check its type.

### Performance Considerations

The type choice has no runtime cost. However, using `ReactNode` for props can reduce unnecessary type assertions.

### Interview Questions

1. What is the difference between `React.ReactNode`, `React.ReactElement`, and `JSX.Element`?
2. Why does `React.FC` use `ReactElement | null` as its return type rather than `ReactNode`?
3. Can `ReactNode` be used as a return type for a component? Why or why not?

### Coding Challenges

1. Write a `SafeRender` component that accepts `ReactNode` but renders nothing for boolean or undefined values.
2. Create a typed `Switch` component where each `Case` child must pass type-checking based on the `Switch`'s value type.

### Related Topics

- Children prop typing
- Component composition patterns
- React Portal types

## Component Props Typing

### What It Is

Component props typing is the practice of defining TypeScript types or interfaces for the props that a React component accepts. This ensures that consumers of the component pass the correct values with the correct shapes.

### Why It Is Important

Props typing acts as a contract between a component and its consumers. It enables IDE autocompletion, compile-time error checking, and documents the component's API without separate documentation.

### How It Works Internally

TypeScript resolves props types at the JSX level through its JSX type checking mechanism. When you write `<Component prop={value} />`, TypeScript checks that the JSX attributes conform to the component's props type using structural typing.

### Syntax

```typescript
// Interface for component props
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

// Union type for props
type StatusProps = {
  status: 'loading' | 'success' | 'error';
  message?: string;
};
```

### Beginner Examples

```typescript
interface UserDisplayProps {
  name: string;
  age: number;
  isOnline: boolean;
}

const UserDisplay = ({ name, age, isOnline }: UserDisplayProps) => (
  <div>
    <span>{name} ({age})</span>
    <span>{isOnline ? 'Online' : 'Offline'}</span>
  </div>
);
```

### Intermediate Examples

```typescript
// Discriminated union for variant props
type NotificationProps =
  | { type: 'success'; message: string; duration?: number }
  | { type: 'error'; message: string; onRetry?: () => void }
  | { type: 'info'; message: string };

const Notification = (props: NotificationProps) => {
  switch (props.type) {
    case 'success':
      return <SuccessBanner message={props.message} duration={props.duration} />;
    case 'error':
      return <ErrorBanner message={props.message} onRetry={props.onRetry} />;
    case 'info':
      return <InfoBanner message={props.message} />;
  }
};
```

### Advanced Examples

```typescript
// Polymorphic component with "as" prop
type PolymorphicProps<T extends React.ElementType> = {
  as?: T;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<T>;

const Box = <T extends React.ElementType = 'div'>({
  as,
  children,
  ...rest
}: PolymorphicProps<T>) => {
  const Component = as || 'div';
  return <Component {...rest}>{children}</Component>;
};

// Usage
<Box as="button" onClick={() => alert('clicked')}>Click</Box>
<Box as="a" href="https://example.com">Link</Box>
```

### Real-World Use Cases

Complex form components with validation, data table components with sorting/filtering props, and design system primitives all rely on sophisticated prop typing to ensure correct usage.

```typescript
// Data table props
interface Column<T> {
  key: keyof T;
  header: string;
  sortable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string | number;
}

interface DataTableProps<T extends Record<string, unknown>> {
  columns: Column<T>[];
  data: T[];
  sortColumn?: keyof T;
  sortDirection?: 'asc' | 'desc';
  onSort?: (column: keyof T) => void;
  pageSize?: number;
}
```

### Common Mistakes

```typescript
// MISTAKE: Mutating props
interface BadProps {
  items: string[];
}
const BadList = ({ items }: BadProps) => {
  items.push('extra'); // This mutates the prop!
};

// CORRECT: Use readonly
interface GoodProps {
  readonly items: readonly string[];
}
```

### Best Practices

- Use `interface` for props that may be extended (e.g., by styled-components or HOCs).
- Use `type` for union-based props (discriminated unions, conditional props).
- Prefer `readonly` for arrays and objects in props to prevent mutation.
- Use `React.ComponentPropsWithoutRef<'button'>` for wrapping native elements.

### Performance Considerations

Typed props do not affect runtime performance. However, they enable TypeScript to optimize error messages and enable tooling that can identify prop-related performance issues.

### Interview Questions

1. How do you handle conditional props where one prop depends on another?
2. What is the difference between `type` and `interface` for props?
3. How would you type a component that accepts an "as" prop to render as different HTML elements?

### Coding Challenges

1. Create a `FormField` component with conditional props where `type: "select"` requires `options` but `type: "text"` does not.
2. Build a typed `Table` component that infers column types from the data array's generic type.

### Related Topics

- Generic components
- Discriminated unions
- Prop spreading and Omit
