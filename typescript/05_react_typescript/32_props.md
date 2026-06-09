# Props - Typing props, interface for props, optional/default props, children prop, component generics

## Introduction

Props are the mechanism by which data flows from parent to child components in React. TypeScript brings rigor to this data flow by allowing developers to define precise contracts for what each component expects. From basic optional props marked with `?` to sophisticated generic components that preserve type relationships across boundaries, TypeScript's type system transforms React props from runtime assumptions into compile-time guarantees. This file covers the props interface pattern, optional and default props, the children prop, and generic components that operate on parameterized types.

## Props Interface Pattern

### What It Is

The props interface pattern is the practice of defining a dedicated TypeScript `interface` (or `type`) that describes all the props a component accepts. This interface is then used as the type annotation for the component's first parameter.

### Why It Is Important

Separating the props type from the component body creates a clear API contract. It makes the component self-documenting, enables IDE autocompletion, and allows the type to be reused or extended elsewhere. It also keeps component signatures clean, especially when props grow beyond a few fields.

### How It Works Internally

When TypeScript compiles JSX, it resolves the component type and extracts its props type. For function components, TypeScript infers the props type from the parameter type annotation. The JSX checking phase then validates each attribute against the props type, performing excess property checks (unless the props are spread from an object with an index signature).

### Syntax

```typescript
import React from 'react';

// Define the props interface
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant: 'primary' | 'secondary';
}

// Use it as the parameter type
const Button = ({ label, onClick, disabled, variant }: ButtonProps) => {
  return (
    <button
      className={`btn btn--${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

### Beginner Examples

```typescript
interface UserAvatarProps {
  username: string;
  size: number;
  avatarUrl?: string;
}

const UserAvatar = ({ username, size, avatarUrl }: UserAvatarProps) => (
  <img
    src={avatarUrl || `https://ui-avatars.com/api/?name=${username}`}
    alt={username}
    width={size}
    height={size}
    className="avatar"
  />
);
```

### Intermediate Examples

```typescript
// Extending an interface
interface BaseInputProps {
  name: string;
  label: string;
  error?: string;
  disabled?: boolean;
}

interface TextInputProps extends BaseInputProps {
  type?: 'text' | 'email' | 'password';
  placeholder?: string;
  maxLength?: number;
}

interface SelectInputProps extends BaseInputProps {
  options: readonly { value: string; label: string }[];
  multiple?: boolean;
}

const TextInput = (props: TextInputProps) => {
  const { name, label, error, disabled, type = 'text', placeholder, maxLength } = props;
  return (
    <div className="field">
      <label htmlFor={name}>{label}</label>
      <input
        id={name}
        name={name}
        type={type}
        placeholder={placeholder}
        maxLength={maxLength}
        disabled={disabled}
      />
      {error && <span className="field__error">{error}</span>}
    </div>
  );
};
```

### Advanced Examples

```typescript
// Omit utility for wrapping native elements
interface StyledButtonProps extends Omit<React.ComponentPropsWithoutRef<'button'>, 'style'> {
  variant?: 'primary' | 'secondary';
  fullWidth?: boolean;
  customStyle?: React.CSSProperties;
}

const StyledButton = ({ variant = 'primary', fullWidth, customStyle, ...nativeProps }: StyledButtonProps) => (
  <button
    {...nativeProps}
    className={`btn btn--${variant} ${fullWidth ? 'btn--full' : ''}`}
    style={customStyle}
  />
);

// Pick utility for restricting props
type ColorAndSize = Pick<Theme, 'primaryColor' | 'fontSize'>;

interface ThemedBoxProps extends ColorAndSize {
  children: React.ReactNode;
}
```

### Real-World Use Cases

Enterprise design systems rely on the props interface pattern to enforce consistent component APIs across dozens of teams. Each component in the system has its props interface documented and versioned:

```typescript
// @package @acme/ui
export interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl';
  closeOnOverlayClick?: boolean;
  showCloseButton?: boolean;
  footer?: React.ReactNode;
}
```

### Common Mistakes

```typescript
// MISTAKE: Inline props type for complex props
const BadComponent = ({ items, onSelect }: { items: string[]; onSelect: (id: string) => void }) => {};

// MISTAKE: Not using interface for exported components
// Interface allows declaration merging and extension

// MISTAKE: Overly permissive types
interface LooseProps {
  config: object; // Avoid: use a specific shape
  data: any;      // Avoid: use unknown or a specific type
}
```

### Best Practices

- Always define a named interface for exported components.
- Use `type` for union props, `interface` for object props.
- Keep interfaces focused; extract reusable sub-types.
- Use `extends` to compose complex interfaces from simpler ones.
- Prefer `interface` over `type` when the props may be extended by consumers.

### Performance Considerations

Props interfaces are erased at compile time with zero runtime impact. However, excessively large interfaces can slow down TypeScript's type-checking performance. Break large interfaces into smaller, focused types.

### Interview Questions

1. When would you use `type` vs `interface` for React props?
2. How does excess property checking work with props interfaces?
3. What is the purpose of `React.ComponentPropsWithoutRef` and when would you use it?

### Coding Challenges

1. Create a `Form` component with a props interface that accepts a validation schema type and infers field types.
2. Build a `Steps` wizard component using an interface that describes each step's props with `next` and `back` callbacks.

### Related Topics

- Discriminated unions for props
- Component composition
- HOC prop injection

## Optional and Default Props

### What It Is

Optional props are props that may or may not be provided by the consumer. They are marked with `?` in the type definition. Default props are values assigned to optional props when they are not provided, usually via destructuring defaults.

### Why It Is Important

Optional props make components flexible without burdening consumers with mandatory configuration. Combined with default values, they provide sensible behavior out of the box while allowing customization when needed.

### How It Works Internally

TypeScript marks optional properties with `| undefined` in the type. When a prop is optional, TypeScript allows omitting it in JSX. The runtime behavior relies on JavaScript's destructuring defaults, which are evaluated at runtime if the value is `undefined`.

### Syntax

```typescript
interface AlertProps {
  message: string;        // Required
  type?: 'info' | 'warning' | 'error';  // Optional
  dismissible?: boolean;  // Optional with default
  onDismiss?: () => void; // Optional callback
}

const Alert = ({
  message,
  type = 'info',
  dismissible = true,
  onDismiss,
}: AlertProps) => (
  <div className={`alert alert--${type}`}>
    {message}
    {dismissible && onDismiss && (
      <button onClick={onDismiss}>X</button>
    )}
  </div>
);
```

### Beginner Examples

```typescript
interface GreetingProps {
  name: string;
  greeting?: string;
}

const Greeting = ({ name, greeting = 'Hello' }: GreetingProps) => (
  <h1>{greeting}, {name}!</h1>
);

// Usage
<Greeting name="Alice" />                    // "Hello, Alice!"
<Greeting name="Bob" greeting="Hi" />       // "Hi, Bob!"
```

### Intermediate Examples

```typescript
interface PaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  siblingCount?: number;
  showFirstLast?: boolean;
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
}

const Pagination = ({
  currentPage,
  totalPages,
  onPageChange,
  siblingCount = 1,
  showFirstLast = true,
  size = 'md',
  disabled = false,
}: PaginationProps) => {
  const pages = generatePageNumbers(currentPage, totalPages, siblingCount);
  return (
    <nav className={`pagination pagination--${size}`}>
      {showFirstLast && (
        <button disabled={disabled || currentPage === 1} onClick={() => onPageChange(1)}>
          First
        </button>
      )}
      {pages.map(page => (
        <button
          key={page}
          disabled={disabled}
          className={page === currentPage ? 'active' : ''}
          onClick={() => onPageChange(page)}
        >
          {page}
        </button>
      ))}
    </nav>
  );
};
```

### Advanced Examples

```typescript
// Conditional required props based on value
interface ConfirmDialogPropsBase {
  isOpen: boolean;
  title: string;
  message: string;
}

// Confirm dialog with optional cancel
interface ConfirmDialogWithCancel extends ConfirmDialogPropsBase {
  showCancel?: boolean;
  cancelLabel?: string;
  onCancel: () => void;
}

// Confirm dialog without cancel
interface ConfirmDialogWithoutCancel extends ConfirmDialogPropsBase {
  showCancel: false;
  cancelLabel?: never;
  onCancel?: never;
}

type ConfirmDialogProps = ConfirmDialogWithCancel | ConfirmDialogWithoutCancel;

const ConfirmDialog = (props: ConfirmDialogProps) => {
  const { isOpen, title, message, showCancel = true, cancelLabel = 'Cancel', onCancel } = props;
  if (!isOpen) return null;
  return (
    <div className="dialog-overlay">
      <div className="dialog">
        <h2>{title}</h2>
        <p>{message}</p>
        <div className="dialog__actions">
          {showCancel && onCancel && (
            <button onClick={onCancel}>{cancelLabel}</button>
          )}
          <button className="primary">Confirm</button>
        </div>
      </div>
    </div>
  );
};
```

### Real-World Use Cases

UI libraries heavily use optional and default props to provide sensible defaults:

```typescript
interface ToastOptions {
  message: string;
  duration?: number;        // default: 3000
  position?: 'top-right' | 'top-left' | 'bottom-right' | 'bottom-left'; // default: 'bottom-right'
  type?: 'success' | 'error' | 'info' | 'warning'; // default: 'info'
  animation?: 'slide' | 'fade' | 'bounce'; // default: 'slide'
  showIcon?: boolean;       // default: true
}
```

### Common Mistakes

```typescript
// MISTAKE: Using defaultProps static property
// interface Props { name: string; count?: number; }
// Component.defaultProps = { count: 0 }; // Deprecated

// CORRECT: Use destructuring defaults
// const Component = ({ name, count = 0 }: Props) => ...

// MISTAKE: Optional boolean with default true
interface BadToggle {
  isOn?: boolean;
}
// Default should be true but undefined is falsy
// const Toggle = ({ isOn = true }: BadToggle) => ...

// MISTAKE: Undefined vs omitted in callbacks
interface CallbackExample {
  onChange?: (value: string) => void;
}
// Consumer must check: if (onChange) onChange(value)
```

### Best Practices

- Use destructuring defaults instead of `defaultProps` (deprecated in React 18+).
- Place required props first in the interface, optional props after.
- Use `?` for optional props, never `| undefined` directly (they are equivalent but `?` is more idiomatic).
- Be explicit about boolean defaults: don't rely on truthiness of optional booleans.

### Performance Considerations

Destructuring defaults add minimal runtime overhead. They are evaluated only when the prop is `undefined`. There is no performance reason to avoid them.

### Interview Questions

1. How do default props work with destructuring in TypeScript?
2. Why is `defaultProps` deprecated and what should you use instead?
3. How do you handle a prop that is optional but required when another prop has a specific value?

### Coding Challenges

1. Create a `Tooltip` component where `position` defaults to `'top'`, `delay` to `500ms`, and `color` to `'dark'`.
2. Build a `Table` component that accepts optional `sortable`, `filterable`, and `pageable` props, each defaulting to `false`.

### Related Topics

- Optional chaining in TypeScript
- Conditional props patterns
- Props with defaults in generic components

## Children Prop Typing

### What It Is

The `children` prop is a special prop in React that represents the content between opening and closing tags of a component. In TypeScript, it must be explicitly typed and is not automatically included in the props type (unless using `React.FC`).

### Why It Is Important

Correctly typing `children` determines what kind of content a component accepts. A component can accept a single string, a single JSX element, multiple elements, a function (render prop), or nothing at all. Getting the type right prevents rendering errors and documents the component's composition contract.

### How It Works Internally

React treats `children` as a regular prop at runtime, but JSX transpilers automatically insert it from the nested content. In `@types/react`, `ReactNode` is the most general children type, while more specific types like `ReactElement` or callbacks can be used for narrower contracts.

### Syntax

```typescript
// Basic children typing
interface BoxProps {
  children: React.ReactNode;
}

const Box = ({ children }: BoxProps) => <div className="box">{children}</div>;

// Strict children (single element)
interface StrictBoxProps {
  children: React.ReactElement;
}

// No children allowed
interface VoidBoxProps {
  children?: never;
}
```

### Beginner Examples

```typescript
interface CardProps {
  title: string;
  children: React.ReactNode;
}

const Card = ({ title, children }: CardProps) => (
  <div className="card">
    <div className="card__header">{title}</div>
    <div className="card__body">{children}</div>
  </div>
);

// Usage
<Card title="My Card">
  <p>This is the card content.</p>
  <button>Action</button>
</Card>
```

### Intermediate Examples

```typescript
// Multiple children slots
interface LayoutProps {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  children: React.ReactNode;
  footer?: React.ReactNode;
}

const Layout = ({ header, sidebar, children, footer }: LayoutProps) => (
  <div className="layout">
    <header>{header}</header>
    <aside>{sidebar}</aside>
    <main>{children}</main>
    {footer && <footer>{footer}</footer>}
  </div>
);

// Render prop pattern
interface DataProviderProps<T> {
  url: string;
  children: (data: T | null, isLoading: boolean, error: Error | null) => React.ReactNode;
}
```

### Advanced Examples

```typescript
// Compound component pattern with typed children
interface TabsComposition {
  Tab: typeof Tab;
  Panel: typeof Panel;
}

interface TabsProps {
  children: React.ReactElement<TabProps | PanelProps> | React.ReactElement<TabProps | PanelProps>[];
  defaultActiveTab?: string;
}

const Tabs: React.FC<TabsProps> & TabsComposition = ({ children, defaultActiveTab }) => {
  const [activeTab, setActiveTab] = useState(defaultActiveTab);
  const tabs = React.Children.toArray(children).filter(
    (child): child is React.ReactElement<TabProps> =>
      React.isValidElement(child) && child.type === Tab
  );
  // Implementation...
};

// Restricting children to specific component types
interface StepperProps {
  children: React.ReactElement<StepProps> | React.ReactElement<StepProps>[];
  activeStep: number;
}

const Stepper = ({ children, activeStep }: StepperProps) => {
  const steps = React.Children.toArray(children);
  // Only React.ReactElement<StepProps> elements are accepted
};
```

### Real-World Use Cases

Compound components like `Select` with `Option`, `Menu` with `MenuItem`, and `Table` with `Column` all rely on typed children to enforce correct usage:

```typescript
interface SelectProps {
  children: React.ReactElement<OptionProps> | React.ReactElement<OptionProps>[];
  value: string;
  onChange: (value: string) => void;
}

interface OptionProps {
  value: string;
  children: React.ReactNode;
}

const Select = ({ children, value, onChange }: SelectProps) => {
  // Implementation
};

const Option = ({ value, children }: OptionProps) => {
  return <option value={value}>{children}</option>;
};

// Usage - TypeScript ensures only Option components are children
<Select value="a" onChange={handleChange}>
  <Option value="a">Option A</Option>
  <Option value="b">Option B</Option>
</Select>
```

### Common Mistakes

```typescript
// MISTAKE: Not typing children at all
// const Component = (props) => ... // children is implicitly any

// MISTAKE: Too restrictive with JSX.Element
interface Restrictive {
  children: JSX.Element; // Does not accept strings, numbers, or arrays
}

// MISTAKE: Assuming children is always present
interface BadProps {
  children: React.ReactNode; // children could be undefined
}
// Correct: use children?: React.ReactNode or handle undefined

// MISTAKE: Forgetting to type render props properly
interface RenderProp {
  children: (count: number) => React.ReactNode;
}
```

### Best Practices

- Default to `React.ReactNode` for general-purpose children.
- Use `children?: React.ReactNode` if children are optional.
- Use `never` to explicitly forbid children.
- Use function types for render prop patterns.
- Use `React.ReactElement<SpecificProps>` for compound component patterns.
- Validate children at runtime with `React.Children.map` and `React.isValidElement`.

### Performance Considerations

Children typing is erased at compile time. However, using `React.Children.map` or `React.Children.toArray` adds runtime overhead. For high-frequency re-renders, consider using direct prop access instead of children traversal.

### Interview Questions

1. How do you type children in React with TypeScript?
2. What is the difference between `React.ReactNode` and `React.ReactElement` for children?
3. How do you type a component that only accepts specific child component types?

### Coding Challenges

1. Create an `Accordion` component where each child must be an `AccordionPanel` component with `title` and `content` props.
2. Build a `Form` component that only accepts `Field` and `Button` components as children, with type checking.

### Related Topics

- Compound components
- Render props pattern
- React.Children utilities

## Generic Components

### What It Is

Generic components are React components that accept type parameters, allowing them to work with a variety of types while maintaining type safety. The type parameter is inferred from the props passed by the consumer.

### Why It Is Important

Generic components eliminate the need for type assertions or the `any` type when creating reusable data-driven components. A `List<T>` component, for example, ensures that the `items` array and the `onSelect` callback operate on the same type, and that the `renderItem` function receives correctly typed elements.

### How It Works Internally

TypeScript's generic type parameters are resolved at compile time. When JSX is transpiled, TypeScript infers the concrete type for the generic parameter based on the props. For example, `<List items={stringArray} />` resolves `T` to `string`. The inference works through the same mechanisms as generic function calls.

### Syntax

```typescript
// Basic generic component
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  onSelect?: (item: T) => void;
}

function List<T>({ items, renderItem, onSelect }: ListProps<T>): JSX.Element {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index} onClick={() => onSelect?.(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}
```

### Beginner Examples

```typescript
interface DropdownProps<T extends string | number> {
  options: T[];
  value: T;
  onChange: (value: T) => void;
  label: string;
}

function Dropdown<T extends string | number>({
  options,
  value,
  onChange,
  label,
}: DropdownProps<T>): JSX.Element {
  return (
    <div>
      <label>{label}</label>
      <select value={value} onChange={(e) => onChange(e.target.value as T)}>
        {options.map((opt) => (
          <option key={String(opt)} value={opt}>
            {String(opt)}
          </option>
        ))}
      </select>
    </div>
  );
}

// Usage - T is inferred as string
<Dropdown
  options={['a', 'b', 'c']}
  value="a"
  onChange={(val) => console.log(val)}
  label="Choose"
/>
```

### Intermediate Examples

```typescript
// Generic table component
interface Column<T> {
  key: keyof T;
  header: string;
  sortable?: boolean;
  render?: (value: T[keyof T], item: T) => React.ReactNode;
}

interface TableProps<T extends Record<string, unknown>> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (item: T) => void;
  keyExtractor: (item: T) => string | number;
}

function Table<T extends Record<string, unknown>>({
  data,
  columns,
  onRowClick,
  keyExtractor,
}: TableProps<T>): JSX.Element {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((col) => (
            <th key={String(col.key)}>{col.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((item) => (
          <tr key={keyExtractor(item)} onClick={() => onRowClick?.(item)}>
            {columns.map((col) => (
              <td key={String(col.key)}>
                {col.render ? col.render(item[col.key], item) : String(item[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Advanced Examples

```typescript
// Generic component with multiple type parameters
interface SelectProps<T, K extends keyof T> {
  items: T[];
  valueKey: K;
  labelKey: keyof T;
  value: T[K];
  onChange: (value: T[K]) => void;
}

function Select<T extends Record<string, unknown>, K extends keyof T>({
  items,
  valueKey,
  labelKey,
  value,
  onChange,
}: SelectProps<T, K>): JSX.Element {
  return (
    <select value={String(value)} onChange={(e) => onChange(e.target.value as T[K])}>
      {items.map((item, index) => (
        <option key={index} value={String(item[valueKey])}>
          {String(item[labelKey])}
        </option>
      ))}
    </select>
  );
}

// Generic component with constraints and default type parameter
interface ApiListProps<T = Record<string, unknown>> {
  url: string;
  children: (state: { data: T[]; loading: boolean; error: Error | null }) => React.ReactNode;
}

function ApiList<T = Record<string, unknown>>({
  url,
  children,
}: ApiListProps<T>): JSX.Element {
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((json: T[]) => {
        setData(json);
        setLoading(false);
      })
      .catch((err: Error) => {
        setError(err);
        setLoading(false);
      });
  }, [url]);

  return <>{children({ data, loading, error })}</>;
}

// Forwarding refs on generic components
import React from 'react';

interface InputProps<T> {
  value: T;
  onChange: (value: T) => void;
  parse: (raw: string) => T;
}

const GenericInput = React.forwardRef(
  <T,>(props: InputProps<T>, ref: React.Ref<HTMLInputElement>) => {
    const { value, onChange, parse, ...rest } = props;
    return (
      <input
        ref={ref}
        value={String(value)}
        onChange={(e) => onChange(parse(e.target.value))}
        {...rest}
      />
    );
  }
) as <T>(props: InputProps<T> & { ref?: React.Ref<HTMLInputElement> }) => JSX.Element;
```

### Real-World Use Cases

Data-heavy applications like dashboards, admin panels, and analytics tools use generic components extensively:

```typescript
// Data fetching hook with generic type
function useApi<T>(url: string): { data: T | null; loading: boolean; error: Error | null } {
  const [state, setState] = useState<{ data: T | null; loading: boolean; error: Error | null }>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    fetch(url)
      .then((res) => res.json() as Promise<T>)
      .then((data) => setState({ data, loading: false, error: null }))
      .catch((error) => setState({ data: null, loading: false, error }));
  }, [url]);

  return state;
}

// Generic data table component
interface DataTableProps<T extends { id: string | number }> {
  data: T[];
  columns: ColumnDef<T>[];
  pageSize?: number;
  sortable?: boolean;
  filterable?: boolean;
}

function DataTable<T extends { id: string | number }>(props: DataTableProps<T>): JSX.Element {
  // Generic table implementation with sorting, filtering, pagination
}
```

### Common Mistakes

```typescript
// MISTAKE: Using arrow function with generics in JSX
// const List = <T,>(props: ListProps<T>) => {} // Syntax error in some TSX configurations
// CORRECT: function List<T>(props: ListProps<T>) ...

// MISTAKE: Not constraining the generic
interface BadListProps<T> {
  items: T[];
}
// function BadList<T>(props: BadListProps<T>) {} // T is completely unconstrained

// CORRECT: Add constraint
interface GoodListProps<T extends { id: string | number }> {
  items: T[];
}

// MISTAKE: Using interface for generic function type
// interface ListFn { <T>(props: ListProps<T>): JSX.Element } // OK
// But generic component is preferred
```

### Best Practices

- Use `function` keyword for generic components (better inference than arrow functions).
- Constrain type parameters with `extends` when you need specific properties.
- Use meaningful type parameter names (`TItem`, `TData`, `TValue`) instead of single letters for clarity.
- Provide default type parameters for maximum flexibility.
- Use `forwardRef` with generic components carefully, often requiring type assertions.

### Performance Considerations

Generics are purely compile-time and have no runtime cost. However, deeply nested generic types in large projects can slow down TypeScript's type checker. Keep generic components reasonably scoped.

### Interview Questions

1. How do you create a generic component in React with TypeScript?
2. How does TypeScript infer generic type parameters from JSX props?
3. How do you forward refs with generic components?
4. What is the purpose of constraining generic type parameters?

### Coding Challenges

1. Create a generic `Form` component where each field's value type is inferred from a schema type.
2. Build a generic `VirtualList` component that accepts items of type `T`, a `renderItem` function, and a `keyExtractor`.
3. Create a generic `Select` component that infers the option type from the `options` array and provides type-safe `value` and `onChange`.

### Related Topics

- TypeScript generics fundamentals
- Polymorphic components
- Render props with generics
- ForwardRef typing
