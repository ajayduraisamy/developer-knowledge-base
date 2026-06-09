# Mapped Types - { [P in K]: T }, key remapping, mapped modifiers, template literal mapping

## Introduction

Mapped types are a fundamental feature of TypeScript's type system that allow you to create new types by transforming all properties of an existing type. The basic syntax `{ [P in K]: T }` iterates over each key in the key set `K` and assigns a value type `T` (which may depend on `P`). Mapped types are the mechanism behind utility types like `Partial`, `Required`, `Readonly`, `Pick`, and `Record`.

Mapped types can include additional clauses for key remapping (the `as` clause, introduced in TypeScript 4.1), modifiers like `readonly` and `?` with `+`/`-` prefix, and template literal mapping for transforming property names. Together, these features make mapped types the most versatile tool for type transformation in TypeScript.

## Mapped type syntax

### What It Is

A mapped type iterates over a union of keys `K` and creates a property for each key with a value type `T`. The most common form is `{ [P in keyof T]: T[P] }`, which creates an exact copy of the original type.

```typescript
type MyPartial<T> = { [P in keyof T]?: T[P] };
type MyReadonly<T> = { readonly [P in keyof T]: T[P] };
```

### Why It Is Important

Mapped types eliminate the need to manually rewrite types when you want to apply transformations like making properties optional, readonly, or changing their types. They enable the creation of flexible, reusable type transformations that work with any input type.

### How It Works Internally

The compiler iterates over each member of the key union `K`. For each key `P`, it creates a property in the resulting type. If `T` depends on `P` (as in `T[P]`), the compiler evaluates the indexed access. The result is a new object type with the transformed properties.

```typescript
// { [P in keyof { a: string; b: number }]: T[P] }
// → { a: string; b: number }
// { [P in keyof { a: string; b: number }]?: T[P] }
// → { a?: string; b?: number }
```

### Syntax

```typescript
// Basic mapping over an object's keys
type MapToBoolean<T> = { [P in keyof T]: boolean };

// Mapping over a union of keys
type StringMap<K extends string> = { [P in K]: string };

// Mapping with type transformation
type NullableProperties<T> = { [P in keyof T]: T[P] | null };

// Mapping with conditional value types
type StringifyValues<T> = { [P in keyof T]: T[P] extends string ? T[P] : string };

// Mapping with key filtering
type NonFunctionKeys<T> = {
  [P in keyof T as T[P] extends Function ? never : P]: T[P];
};
```

### Beginner Examples

```typescript
// Simple mapping: all properties become strings
interface User {
  id: number;
  name: string;
  age: number;
}

type Stringified<T> = { [P in keyof T]: string };
type StringUser = Stringified<User>;
// { id: string; name: string; age: string }

// Mapping: all properties become optional
type Optional<T> = { [P in keyof T]?: T[P] };
type OptionalUser = Optional<User>;
// { id?: number; name?: string; age?: number }

// Mapping: all properties become nullable
type Nullable<T> = { [P in keyof T]: T[P] | null };
type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; age: number | null }

// Mapping with constant value type
type Flags<T> = { [P in keyof T]: boolean };
type UserFlags = Flags<User>;
// { id: boolean; name: boolean; age: boolean }

// Mapping over a union of string literals
type HTTPHeaders = { [P in "content-type" | "accept" | "authorization"]: string };
// { "content-type": string; accept: string; authorization: string }
```

### Intermediate Examples

```typescript
// Mapping that transforms value types
type Wrapped<T> = { [P in keyof T]: { value: T[P] } };
type WrappedUser = Wrapped<User>;
// { id: { value: number }; name: { value: string }; age: { value: number } }

// Mapping with conditional value types
type PrimitiveOnly<T> = {
  [P in keyof T]: T[P] extends object ? never : T[P];
};

// Mapping that adds methods
type WithGetters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};
type UserGetters = WithGetters<User>;
// { getId: () => number; getName: () => string; getAge: () => number }

// Mapping with filtering by value type
type StringKeysOnly<T> = {
  [P in keyof T as T[P] extends string ? P : never]: T[P];
};
type UserStrings = StringKeysOnly<User>; // { name: string }

// Mapping with both key and value transformations
type Serialized<T> = {
  [P in keyof T]: T[P] extends Date ? string : T[P];
};
```

### Advanced Examples

```typescript
// Recursive mapped type
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};

// Mapped type with conditional key filtering
type FunctionPropertyNames<T> = {
  [P in keyof T]: T[P] extends Function ? P : never;
}[keyof T];

type UserMethods = FunctionPropertyNames<User>;

// Mapped type over tuple/array
type TupleToUnion<T extends readonly unknown[]> = T[number];
type ArrayToObject<T extends readonly (keyof any)[]> = {
  [P in T[number]]: P;
};

// Mapped type with template literal value types
type CSSProperties<T extends string> = {
  [P in T as `--${P}`]: string | number;
};

// Mapped type for validation schema
type ValidationSchema<T> = {
  [P in keyof T]: {
    required?: boolean;
    type: "string" | "number" | "boolean" | "object";
    validate?: (value: T[P]) => boolean;
  };
};

// Mapped type with function signatures
type Promisify<T> = {
  [P in keyof T]: T[P] extends (...args: infer A) => infer R
    ? (...args: A) => Promise<R>
    : Promise<T[P]>;
};

// Mapped type with index signature preservation
type PreserveIndex<T> = {
  [P in keyof T]: T[P];
} & {
  [key: string]: unknown;
};
```

### Real-World Use Cases

- **API client generation**: Mapping REST endpoints to typed functions.
- **ORM entity transformation**: Converting entity types to DTOs or views.
- **State management**: Deriving action types from reducer shapes.
- **Form validation**: Generating validation schemas from model types.
- **Serialization**: Creating serialized versions of complex types.
- **Testing utilities**: Generating mock types from real types.

```typescript
// Real-world: API response transformation
type APIResponse<T> = {
  [P in keyof T]: {
    data: T[P];
    loading: boolean;
    error: Error | null;
  };
};

interface UserData {
  profile: { name: string; email: string };
  posts: { id: number; title: string }[];
  settings: { theme: string; notifications: boolean };
}

type UserAPIState = APIResponse<UserData>;
// {
//   profile: { data: { name: string; email: string }; loading: boolean; error: ... };
//   posts: { data: { id: number; title: string }[]; loading: boolean; error: ... };
//   settings: { data: { theme: string; notifications: boolean }; loading: boolean; error: ... };
// }
```

### Common Mistakes

1. **Using `[P in keyof T]` where `T` might not be an object type**—it evaluates to `never`.
2. **Forgetting the `in` keyword**: The syntax is `[P in keyof T]`, not `[P: keyof T]`.
3. **Not handling the `readonly` modifier**: By default, mapped types don't copy readonly.
4. **Creating overly complex mapped types**—sometimes a simple union or interface is clearer.
5. **Assuming mapped types copy optionality**: You need `?` to add optionality, and `-?` to remove it.

### Best Practices

- Use mapped types for bulk property transformations.
- Combine with conditional types for value-dependent transformations.
- Use key remapping (`as`) for property name transformations.
- Prefer built-in utility types (`Partial`, `Readonly`) over writing your own.
- Test mapped types with simple interfaces first.
- Use `keyof T` as the key source for object-type mapping.

### Performance Considerations

Mapped types are efficient for typical object sizes. Very large objects (100+ properties) with recursive mapped types can impact compile time. Mapped types over unions with thousands of members can also be slow.

### Interview Questions

1. **Q**: What is a mapped type?
   **A**: A mapped type creates a new object type by iterating over a union of keys and transforming the property values. Syntax: `{ [P in K]: T }`.

2. **Q**: How do you make all properties of a type optional using a mapped type?
   **A**: `type MyPartial<T> = { [P in keyof T]?: T[P] }`

### Coding Challenges

1. Implement `Partial<T>`, `Required<T>`, and `Readonly<T>` using mapped types.
2. Write a mapped type that converts all property values to their JSON-safe equivalents.

### Related Topics

- Key remapping
- Mapped modifiers
- Template literal mapping
- Utility types

## Key remapping (as clause)

### What It Is

Key remapping allows you to transform the keys themselves in a mapped type using the `as` clause. The syntax `[P in K as NewKey]: T` lets you filter, rename, or restructure the keys of the resulting type.

```typescript
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};
```

### Why It Is Important

Key remapping enables creating entirely new property names based on existing ones—essential for getter/setter generation, prefix/suffix patterns, and key filtering based on value types. Before TypeScript 4.1, key remapping required complex type gymnastics.

### How It Works Internally

The `as` clause specifies a new key name for each `P`. The new key can be a string literal, a template literal, or `never` (to exclude the property). The compiler evaluates the key expression for each property and builds the resulting type with the transformed keys.

```typescript
// [P in keyof T as `get${Capitalize<P>}`]: ()
// For P = "name": new key = "getName"
// For P = "age": new key = "getAge"
```

### Syntax

```typescript
// Key renaming with template literals
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

// Key filtering (exclude by key)
type WithoutId<T> = {
  [P in keyof T as P extends "id" ? never : P]: T[P];
};

// Key filtering (exclude by value type)
type FunctionsOnly<T> = {
  [P in keyof T as T[P] extends Function ? P : never]: T[P];
};

// Key remapping with constant
type Prefix<T, Pref extends string> = {
  [P in keyof T as `${Pref}${Capitalize<string & P>}`]: T[P];
};
```

### Beginner Examples

```typescript
interface Person {
  name: string;
  age: number;
  email: string;
}

// Add "get" prefix to all keys
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number; getEmail: () => string }

// Remove "email" key
type WithoutEmail<T> = {
  [P in keyof T as P extends "email" ? never : P]: T[P];
};

type PersonWithoutEmail = WithoutEmail<Person>;
// { name: string; age: number }

// Keep only string-valued keys
type StringKeys<T> = {
  [P in keyof T as T[P] extends string ? P : never]: T[P];
};

type PersonStrings = StringKeys<Person>;
// { name: string; email: string }
```

### Intermediate Examples

```typescript
// Key remapping with suffix
type Setters<T> = {
  [P in keyof T as `set${Capitalize<string & P>}`]: (value: T[P]) => void;
};

// Exclude function-typed keys
type NonFunctions<T> = {
  [P in keyof T as T[P] extends Function ? never : P]: T[P];
};

// Map keys to their string literal representations
type StringLiteralKeys<T> = {
  [P in keyof T as P extends string ? P : never]: T[P];
};

// Double the keys (original + prefixed)
type WithPrefix<T, Pref extends string> = T & {
  [P in keyof T as `${Pref}${Capitalize<string & P>}`]: T[P];
};

type PrefixedPerson = WithPrefix<Person, "original">;
// Person & { OriginalName: string; OriginalAge: number; OriginalEmail: string }

// Filter by optionality
type RequiredKeys<T> = {
  [P in keyof T as undefined extends T[P] ? never : P]: T[P];
};
```

### Advanced Examples

```typescript
// Conditional key remapping with multiple branches
type TransformKeys<T> = {
  [P in keyof T as P extends string
    ? `transformed_${P}`
    : P]: T[P];
};

// Key remapping for API paths
type APIRoutes = {
  users: { list: "GET"; create: "POST" };
  posts: { list: "GET"; byId: "GET" };
};

type ExpandAPI<T> = {
  [Module in keyof T]: {
    [Action in keyof T[Module] as `/${string & Module}/${string & Action}`]: T[Module][Action];
  };
}[keyof T];

type Routes = ExpandAPI<APIRoutes>;
// { "/users/list": "GET" } | { "/users/create": "POST" } | { "/posts/list": "GET" } | { "/posts/byId": "GET" }

// Key remapping with complex filtering
type OmitByType<T, Type> = {
  [P in keyof T as T[P] extends Type ? never : P]: T[P];
};

type PickByType<T, Type> = {
  [P in keyof T as T[P] extends Type ? P : never]: T[P];
};

// Key remapping with conditional types for recursive types
type DeepOmitByType<T, Type> = T extends object
  ? {
      [P in keyof T as T[P] extends Type ? never : P]: DeepOmitByType<T[P], Type>;
    }
  : T;

// Key remapping with branded keys
type Brand<T, B extends string> = T & { __brand: B };
type RemoveBrand<T> = {
  [P in keyof T as P extends "__brand" ? never : P]: T[P];
};

// Key remapping to convert snake_case to camelCase
type SnakeToCamel<S extends string> =
  S extends `${infer T}_${infer U}` ? `${T}${Capitalize<SnakeToCamel<U>>}` : S;

type CamelCaseKeys<T> = {
  [P in keyof T as SnakeToCamel<string & P>]: T[P];
};

type SnakeCase = { user_name: string; user_age: number };
type CamelCase = CamelCaseKeys<SnakeCase>;
// { userName: string; userAge: number }
```

### Real-World Use Cases

- **API client generation**: Creating typed methods from entity names.
- **ORM query builders**: Converting entity field names to query methods.
- **Form validation**: Creating validation rules from model fields.
- **Serialization**: Renaming fields for JSON output.
- **Permissions**: Transforming permission names into check functions.
- **State management**: Creating action creator types from state shapes.

```typescript
// Real-world: Form validation schema generator
type ValidationSchema<T> = {
  [P in keyof T as `validate${Capitalize<string & P>}`]: (value: T[P]) => string | null;
};

interface LoginForm {
  email: string;
  password: string;
}

type LoginValidation = ValidationSchema<LoginForm>;
// {
//   validateEmail: (value: string) => string | null;
//   validatePassword: (value: string) => string | null;
// }
```

### Common Mistakes

1. **Returning a non-string type from `as`**: The key must be a `string | number | symbol` or `never`.
2. **Using `as` without template literals**: For simple renaming, you need string manipulation.
3. **Filtering all keys**: If all keys become `never`, the result is `{}`.
4. **Not handling `string & P` for template literals**: Template literals require `string` type, but `P` might be `number` or `symbol`.
5. **Forgetting that `as` is evaluated after the mapping**: You can use `T[P]` in the `as` expression.

### Best Practices

- Use `string & P` in template literals to handle potential `number`/`symbol` keys.
- Use `never` in the `as` clause to filter out unwanted properties.
- Combine with conditional types for value-based key filtering.
- Keep key remapping expressions simple—extract complex logic into helper types.
- Use `Capitalize`, `Lowercase`, `Uppercase`, `Uncapitalize` for case transformations.

### Performance Considerations

Key remapping adds a small overhead compared to simple mapping. Complex template literal expressions in key remapping can increase compile time. Filtering with `never` is efficient.

### Interview Questions

1. **Q**: What does `never` do in the `as` clause of a mapped type?
   **A**: It excludes that property from the resulting type.

2. **Q**: How do you rename properties in a mapped type?
   **A**: Use the `as` clause with template literals: `[P in keyof T as `new_${P}`]: T[P]`.

### Coding Challenges

1. Write a mapped type that converts all property names from camelCase to snake_case.
2. Create a type that adds a `get` prefix to every property name and wraps the value in `() => T`.

### Related Topics

- Template literal types
- `Capitalize`, `Lowercase`, etc.
- Conditional types
- Mapped modifiers

## Modifiers (+/- readonly, +/- ?)

### What It Is

Mapped types can apply or remove the `readonly` and optional (`?`) modifiers using `+` and `-` prefixes. By default, adding `readonly` or `?` applies the modifier (`+readonly`, `+?`). Using `-readonly` or `-?` removes the modifier.

```typescript
type Mutable<T> = { -readonly [P in keyof T]: T[P] };
type Required<T> = { [P in keyof T]-?: T[P] };
```

### Why It Is Important

Modifier control is essential for implementing `Partial`, `Required`, `Readonly`, and `Mutable` utility types. Without `-?` and `-readonly`, you could only add modifiers, not remove them. These operators give you complete control over the resulting type's structure.

### How It Works Internally

The `+` and `-` prefixes control whether the modifier is added or removed. `+readonly` adds the readonly modifier (same as just `readonly`). `-readonly` removes it. Same for `?` (optional). If neither `+` nor `-` is specified, `readonly` and `?` are applied (equivalent to `+`).

```typescript
// { [P in keyof T]+?: T[P] } → adds optional (same as { [P in keyof T]?: T[P] })
// { [P in keyof T]-?: T[P] } → removes optional
// { +readonly [P in keyof T]: T[P] } → adds readonly (same as { readonly [P in keyof T]: T[P] })
// { -readonly [P in keyof T]: T[P] } → removes readonly
```

### Syntax

```typescript
// Adding modifiers
type Partial<T> = { [P in keyof T]?: T[P] };           // +? (implied)
type Readonly<T> = { readonly [P in keyof T]: T[P] }; // +readonly (implied)

// Removing modifiers
type Required<T> = { [P in keyof T]-?: T[P] };       // -?
type Mutable<T> = { -readonly [P in keyof T]: T[P] }; // -readonly

// Both
type MutableRequired<T> = { -readonly [P in keyof T]-?: T[P] };
type PartialReadonly<T> = { readonly [P in keyof T]?: T[P] };
```

### Beginner Examples

```typescript
interface Config {
  readonly host: string;
  readonly port: number;
  timeout?: number;
  retries?: number;
}

// Remove readonly
type MutableConfig = { -readonly [P in keyof Config]: Config[P] };
// { host: string; port: number; timeout?: number; retries?: number }

// Remove optional
type RequiredConfig = { [P in keyof Config]-?: Config[P] };
// { readonly host: string; readonly port: number; timeout: number; retries: number }

// Remove both
type PlainConfig = { -readonly [P in keyof Config]-?: Config[P] };
// { host: string; port: number; timeout: number; retries: number }

// Add both
type FrozenOptional = { readonly [P in keyof Config]?: Config[P] };
// { readonly host?: string; readonly port?: number; readonly timeout?: number; readonly retries?: number }
```

### Intermediate Examples

```typescript
// Mapped type that preserves original modifiers
type Identity<T> = { [P in keyof T]: T[P] };

// Selective modifier removal
type MutableKeys<T, K extends keyof T> = 
  Omit<T, K> & { -readonly [P in K]: T[P] };

// Non-nullable properties (remove both optional and null/undefined)
type Concrete<T> = {
  [P in keyof T]-?: NonNullable<T[P]>;
};

// Toggle modifiers based on value type
type StringOptionalOthersRequired<T> = {
  [P in keyof T]: T[P] extends string ? T[P] | undefined : T[P];
};

// Combining multiple modifiers
type ApiConfig = {
  readonly baseUrl: string;
  readonly apiKey?: string;
  timeout: number;
};

type EditableConfig = {
  -readonly [P in keyof ApiConfig]-?: ApiConfig[P];
};
// { baseUrl: string; apiKey: string; timeout: number }
```

### Advanced Examples

```typescript
// Conditional modifier application
type CustomPartial<T, K extends keyof T = keyof T> = {
  [P in keyof T as P extends K ? P : never]?: T[P];
} & {
  [P in keyof T as P extends K ? never : P]: T[P];
};

// Mapped type with modifiers and key remapping
type ReadonlyGetters<T> = {
  readonly [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

// Deep modifier application
type DeepMutable<T> = T extends object
  ? { -readonly [P in keyof T]: DeepMutable<T[P]> }
  : T;

type DeepRequired<T> = T extends object
  ? { [P in keyof T]-?: DeepRequired<T[P]> }
  : T;

// Modifiers with index signatures
type NormalizeIndex<T> = {
  readonly [P in keyof T]-?: T[P];
} & {
  [key: string]: unknown;
};

// Modifier mapping with conditional types
type Modify<T, Modifier extends "partial" | "required" | "readonly" | "mutable"> =
  Modifier extends "partial" ? { [P in keyof T]?: T[P] } :
  Modifier extends "required" ? { [P in keyof T]-?: T[P] } :
  Modifier extends "readonly" ? { readonly [P in keyof T]: T[P] } :
  Modifier extends "mutable" ? { -readonly [P in keyof T]: T[P] } :
  T;

// Specifying modifiers via template literal
type ApplyModifier<T, M extends string> = {
  [P in keyof T as `${M}${Capitalize<string & P>}`]: T[P];
};
```

### Real-World Use Cases

- **DTO generation**: Converting entity types (often readonly) to mutable DTOs.
- **Form state**: Making model fields optional for partial form state.
- **API responses**: Converting optional fields to required after validation.
- **Configuration merging**: Making all config properties required after defaults are applied.
- **State management**: Creating mutable local state from readonly global state.
- **Testing**: Creating mutable mock objects from readonly interfaces.

```typescript
// Real-world: Form state initialization
interface UserEntity {
  readonly id: string;
  readonly createdAt: Date;
  name?: string;
  email?: string;
}

type FormState = { [P in keyof UserEntity]-?: UserEntity[P] };
// { id: string; createdAt: Date; name: string; email: string }

function createForm(user: UserEntity): FormState {
  return {
    id: user.id,
    createdAt: user.createdAt,
    name: user.name ?? "",
    email: user.email ?? "",
  };
}
```

### Common Mistakes

1. **Using `-?` on a type where no properties are optional**: Has no effect but doesn't error.
2. **Using `-readonly` on a type with no readonly properties**: Similarly no effect.
3. **Forgetting that `?` and `readonly` are the only modifiable modifiers**.
4. **Applying `-readonly` without removing `?` first**: Order doesn't matter—both are applied.
5. **Assuming modifier removal works on nested objects**: It only affects the top level.

### Best Practices

- Use `-?` for `Required<T>` and `-readonly` for `Mutable<T>`.
- Use `?` for `Partial<T>` and `readonly` for `Readonly<T>`.
- Combine modifier removal with key remapping for fine-grained control.
- Use `DeepRequired` and `DeepMutable` for nested type transformations.
- Always test modifier removal on a sample type to ensure it works as expected.

### Performance Considerations

Modifier application and removal is efficient. The compiler simply toggles flags on properties. Deep modifier application with recursive types can increase compile time for deeply nested types.

### Interview Questions

1. **Q**: What does `-?` do in a mapped type?
   **A**: It removes the optional modifier from properties, making them required.

2. **Q**: What is the difference between `readonly` and `+readonly`?
   **A**: There is no difference. `+` is implied when adding a modifier. `-readonly` is used to remove it.

### Coding Challenges

1. Implement `Required<T>` and `Mutable<T>` using mapped type modifiers.
2. Create a `DeepMutable` type that recursively removes readonly from nested objects.

### Related Topics

- `Partial`, `Required`, `Readonly` utility types
- `-?` and `-readonly` syntax
- Mapped types

## Template literal mapping

### What It Is

Template literal mapping combines mapped types with template literal types to create new property names based on existing ones. By using template literal types in the key position of a mapped type, you can generate property names that follow naming conventions.

```typescript
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};
```

### Why It Is Important

Template literal mapping enables the creation of API client methods, ORM query builders, form validation schemas, and any type transformation that follows naming conventions. It eliminates manual repetition and ensures consistency between original and generated property names.

### How It Works Internally

The compiler evaluates the template literal for each key `P`. The `Capitalize` intrinsic type transforms the first character of the string representation of `P`. The result becomes the new property name. The compiler tracks the correspondence between original and generated names.

```typescript
// P = "name"
// `get${Capitalize<string & P>}` → `get${Capitalize<"name">}` → "getName"
```

### Syntax

```typescript
// Add prefix
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};

// Add suffix
type Suffixed<T, S extends string> = {
  [K in keyof T as `${string & K}${Capitalize<S>}`]: T[K];
};

// Multiple transformations
type GettersAndSetters<T> = Getters<T> & Setters<T>;

// Key pattern matching with infer
type StripPrefix<T, P extends string> = {
  [K in keyof T as K extends `${P}${infer Rest}`
    ? Uncapitalize<Rest>
    : K]: T[K];
};
```

### Beginner Examples

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

// Create getter types
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number; getEmail: () => string }

// Create setter types
type Setters<T> = {
  [P in keyof T as `set${Capitalize<string & P>}`]: (value: T[P]) => void;
};

type UserSetters = Setters<User>;
// { setName: (value: string) => void; setAge: (value: number) => void; setEmail: (value: string) => void }

// Combine getters and setters
type FullAccess<T> = Getters<T> & Setters<T>;
type UserAccess = FullAccess<User>;

// Add common prefix
type APIEndpoints = {
  [P in "users" | "posts" | "comments" as `/api/${P}`]: {
    list: string;
    create: string;
  };
};
```

### Intermediate Examples

```typescript
// Mapping events to handlers
type EventHandlers<T extends Record<string, unknown>> = {
  [P in keyof T & string as `on${Capitalize<P>}`]: (payload: T[P]) => void;
};

interface AppEvents {
  click: { x: number; y: number };
  focus: void;
  blur: void;
}

type Handlers = EventHandlers<AppEvents>;
// { onClick: (payload: { x: number; y: number }) => void; onFocus: (payload: void) => void; ... }

// CSS variable generation
type CSSVariables<T extends Record<string, string>> = {
  [P in keyof T & string as `--${P}`]: T[P];
};

type ThemeVars = CSSVariables<{ primary: "#007bff"; secondary: "#6c757d" }>;
// { "--primary": "#007bff"; "--secondary": "#6c757d" }

// Snake to camel case keys
type SnakeToCamel<S extends string> =
  S extends `${infer T}_${infer U}` ? `${T}${Capitalize<SnakeToCamel<U>>}` : S;

type CamelCaseKeys<T> = {
  [P in keyof T as SnakeToCamel<string & P>]: T[P];
};

type SnakeData = { user_name: string; user_age: number };
type CamelData = CamelCaseKeys<SnakeData>;
// { userName: string; userAge: number }
```

### Advanced Examples

```typescript
// Deep template literal mapping
type DeepCamelCaseKeys<T> = T extends object
  ? {
      [P in keyof T as SnakeToCamel<string & P>]: DeepCamelCaseKeys<T[P]>;
    }
  : T;

// Bidirectional mapping (prefix + suffix)
type FullTransformer<T> = {
  [P in keyof T as `get${Capitalize<string & P>}Value`]: () => T[P];
};

// Conditional naming based on value type
type TypedNaming<T> = {
  [P in keyof T as T[P] extends string
    ? `str${Capitalize<string & P>}`
    : T[P] extends number
      ? `num${Capitalize<string & P>}`
      : P]: T[P];
};

// API path generation with parameters
type APIPaths<T extends Record<string, Record<string, string>>> = {
  [Module in keyof T]: {
    [Action in keyof T[Module] as `/${string & Module}/${string & Action}`]: 
      T[Module][Action];
  };
}[keyof T];

// Creating utility types with template literal mapping
type PluckResult<T, K extends string> = {
  [P in keyof T as P extends `${K}_${infer Rest}` ? Rest : never]: T[P];
};

// Event emitter typing
type EventEmitter<T extends Record<string, unknown>> = {
  [P in keyof T as `emit${Capitalize<string & P>}`]: (payload: T[P]) => void;
} & {
  [P in keyof T as `on${Capitalize<string & P>}`]: (handler: (payload: T[P]) => void) => void;
} & {
  [P in keyof T as `off${Capitalize<string & P>}`]: (handler: (payload: T[P]) => void) => void;
};
```

### Real-World Use Cases

- **API client libraries**: Generating typed methods like `getUser`, `createPost` from entity names.
- **ORM query builders**: Creating methods like `whereName`, `orderByAge` from field names.
- **Form validation**: Creating methods like `validateEmail`, `validatePassword`.
- **Event emitters**: Typed `onClick`, `emitClick`, `offClick` methods.
- **CSS-in-JS**: Converting theme objects to CSS variable names.
- **State management**: Creating action creators named after actions.
- **Configuration**: Transforming config keys to environment variable names.

```typescript
// Real-world: Type-safe API client generator
type APIClient<T extends Record<string, Record<string, { req: unknown; res: unknown }>>> = {
  [Module in keyof T]: {
    [Action in keyof T[Module] as `${string & Action}${Capitalize<string & Module>}`]: (
      req: T[Module][Action]["req"]
    ) => Promise<T[Module][Action]["res"]>;
  };
}[keyof T];

interface API {
  users: {
    list: { req: void; res: { id: number; name: string }[] };
    create: { req: { name: string; email: string }; res: { id: number } };
  };
}

type Client = APIClient<API>;
// { listUsers: () => Promise<{ id: number; name: string }[]> }
// { createUsers: (req: { name: string; email: string }) => Promise<{ id: number }> }
```

### Common Mistakes

1. **Not converting `P` to `string` in template literals**: `P` might be `number | symbol`, causing errors.
2. **Using `Capitalize` on `P` directly**: You need `Capitalize<string & P>`.
3. **Creating unnecessary complex names**: Sometimes a simple suffix is enough.
4. **Forgetting that template literal keys must be valid identifiers** (for dot access).
5. **Not handling edge cases**: What happens when all keys are filtered out?

### Best Practices

- Always use `string & P` in template literals for key remapping.
- Use `Capitalize`, `Lowercase`, `Uppercase`, `Uncapitalize` for case transformations.
- Combine with key filtering (`never`) to exclude unwanted keys.
- Extract complex template literal patterns into helper types.
- Test template literal mapping with a small sample type first.
- Document the naming convention being generated.

### Performance Considerations

Template literal mapping involves string manipulation at the type level, which can be slower than simple key mapping. The compiler must evaluate each template literal expression for each property. For types with many properties (50+), this can noticeably increase compile time.

### Interview Questions

1. **Q**: How do you create a mapped type that adds a prefix to all property names?
   **A**: `type Prefixed<T, P extends string> = { [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K] };`

2. **Q**: Why do you need `string & P` in template literal key remapping?
   **A**: Because `P` might be a number or symbol, and template literals only work with strings. `string & P` narrows to the string-like subset.

### Coding Challenges

1. Write a mapped type that converts all property names to UPPER_SNAKE_CASE.
2. Create a type that generates both getters and setters for all properties of an interface.
3. Implement a type that transforms a flat object into nested getter objects like `user.getName()`.

### Related Topics

- Template literal types
- Key remapping
- `Capitalize`, `Lowercase`, `Uppercase`, `Uncapitalize`
- Mapped type modifiers
