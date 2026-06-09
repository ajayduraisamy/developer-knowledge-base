# Arrays - Array<T> syntax, tuple arrays, readonly arrays, array methods typing

## Introduction

Arrays are one of the most fundamental data structures in TypeScript. TypeScript enhances JavaScript arrays with a rich type system that allows precise specification of element types, immutability constraints, and method safety. This guide covers array type syntax, readonly arrays, type-safe method usage, and multidimensional arrays.

## Array type syntax (T[] and Array<T>)

### What It Is

TypeScript provides two equivalent syntaxes for typing arrays: the shorthand T[] and the generic Array<T>. Both represent an array where every element is of type T.

### Why It Is Important

Understanding array typing is essential for working with collections of data. Array types are the foundation for lists, buffers, and data sets, and choosing the right syntax affects readability and consistency in your codebase.

### How It Works Internally

Both T[] and Array<T> compile to the same JavaScript arrays. TypeScript treats them as identical in the type system. The Array<T> type is a generic interface defined in lib.es5.d.ts (or later) that includes method signatures and index access types.

### Syntax

`	ypescript
// T[] syntax (shorthand)
let numbers: number[] = [1, 2, 3];
let strings: string[] = ["a", "b", "c"];

// Array<T> syntax (generic)
let numbers2: Array<number> = [1, 2, 3];
let strings2: Array<string> = ["a", "b", "c"];

// Mixed
let mixed: (string | number)[] = [1, "two", 3];
let mixed2: Array<string | number> = [1, "two", 3];
`

### Beginner Examples

`	ypescript
// Declaring typed arrays
const fruits: string[] = ["apple", "banana", "orange"];
const scores: number[] = [95, 87, 92, 78];
const flags: boolean[] = [true, false, true];

// Array of objects
interface Product {
  id: number;
  name: string;
  price: number;
}

const products: Product[] = [
  { id: 1, name: "Laptop", price: 999 },
  { id: 2, name: "Mouse", price: 29 },
];

// Accessing elements
const firstFruit: string = fruits[0];
const product: Product = products[1];
`

### Intermediate Examples

`	ypescript
// Array of unions
const data: (string | number)[] = [1, "hello", 42, "world"];

// Nested array types
const matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

// Array of generics
function cloneArray<T>(items: T[]): T[] {
  return [...items];
}

const cloned = cloneArray([1, 2, 3]);

// Mapping arrays with type safety
const doubled: number[] = [1, 2, 3].map((x) => x * 2);
const lengths: number[] = ["hi", "hello"].map((s) => s.length);
`

### Advanced Examples

`	ypescript
// Array with discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rect"; width: number; height: number };

const shapes: Shape[] = [
  { kind: "circle", radius: 5 },
  { kind: "rect", width: 3, height: 4 },
];

const areas = shapes.map((s) => {
  if (s.kind === "circle") return Math.PI * s.radius ** 2;
  return s.width * s.height;
});

// Generic array constraints
function getLengths<T extends { length: number }>(items: T[]): number[] {
  return items.map((item) => item.length);
}

getLengths(["hello", "world"]);
getLengths(["a", "b", "c"]);

// Fixed-length array types
type RGB = [number, number, number];
const color: RGB = [255, 0, 0];

// ReadonlyMap from array entries
const entries: readonly (readonly [string, number])[] = [
  ["a", 1],
  ["b", 2],
] as const;
`

### Real-World Use Cases

**API Response Lists**: Arrays of typed objects representing database records.

**UI Component Lists**: Arrays of data fed into list renderers (React FlatList, etc.).

**Data Processing Pipelines**: Arrays passed through map/filter/reduce chains.

### Common Mistakes

**Using any[]**: Prefer unknown[] when the element type is unknown.

**Mutable Array Parameters**: Functions should accept eadonly T[] when they don't modify the array.

**Assuming Arrays Are Homogeneous**: Arrays can hold union types for heterogeneous data.

### Best Practices

1. Prefer T[] shorthand for simplicity
2. Use eadonly T[] for immutable array parameters
3. Use Array<T> for alignment with other generic types
4. Avoid ny[] — use unknown[] and narrow

### Performance Considerations

Array type annotations are erased at compile time. No runtime overhead.

### Interview Questions

1. What is the difference between T[] and Array<T>?
2. How do you type a multidimensional array?
3. How does TypeScript's structural typing apply to arrays?

### Coding Challenges

1. Write a function that accepts both 
umber[] and eadonly number[]
2. Implement a type-safe zip function for two arrays

### Related Topics

- Tuple Types
- ReadonlyArray
- Array Method Typing

## ReadonlyArray

### What It Is

ReadonlyArray<T> (or eadonly T[]) is a TypeScript type representing an array whose elements cannot be mutated. It prevents operations like push, pop, splice, and index assignment.

### Why It Is Important

Readonly arrays enforce immutability at the type level, preventing accidental mutations. They signal intent clearly — a function accepting eadonly T[] promises not to modify the array, making code safer and easier to reason about.

### How It Works Internally

ReadonlyArray<T> is an interface defined in TypeScript's standard library. It includes all the read-only methods of arrays (like map, ilter, educe, concat, slice) but omits mutating methods (like push, pop, splice, sort). The eadonly T[] syntax is syntactic sugar for ReadonlyArray<T>.

### Syntax

`	ypescript
// Two equivalent syntaxes
const arr1: readonly number[] = [1, 2, 3];
const arr2: ReadonlyArray<number> = [1, 2, 3];

// arr1.push(4); // Error
// arr1[0] = 10; // Error

// Valid operations (non-mutating)
const doubled = arr1.map((x) => x * 2);
const filtered = arr1.filter((x) => x > 1);
const sliced = arr1.slice(0, 2);
`

### Beginner Examples

`	ypescript
// Function accepts readonly — signals it won't mutate
function sum(numbers: readonly number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

const scores: readonly number[] = [90, 85, 92];
const total = sum(scores);

// Cannot mutate
// scores.push(95); // Error
// scores[0] = 100; // Error

// Can create new arrays
const updated = [...scores, 95]; // OK — creates new array
`

### Intermediate Examples

`	ypescript
// Readonly in generic functions
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}

function last<T>(items: readonly T[]): T | undefined {
  return items[items.length - 1];
}

// Readonly with class fields
class ImmutableCollection<T> {
  constructor(private readonly items: readonly T[]) {}

  get(index: number): T | undefined {
    return this.items[index];
  }

  toArray(): readonly T[] {
    return this.items;
  }
}

// ReadonlyArray with spread
function merge<T>(a: readonly T[], b: readonly T[]): T[] {
  return [...a, ...b];
}
`

### Advanced Examples

`	ypescript
// Deep readonly arrays
type DeepReadonlyArray<T> = readonly (T extends (infer U)[]
  ? DeepReadonlyArray<U>
  : T)[];

type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Readonly array of readonly objects
interface User {
  readonly id: number;
  name: string;
}

const users: readonly User[] = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
];

// users[0].id = 5; // Error: id is readonly
// users[0] = { id: 3, name: "Charlie" }; // Error: readonly array

// Readonly tuple
type Point = readonly [number, number];
const origin: Point = [0, 0];
// origin[0] = 5; // Error
`

### Real-World Use Cases

**Reducer State**: Redux reducers should treat state as immutable — eadonly enforces this.

**Configuration Arrays**: Lists of configuration values that should not change at runtime.

**Pure Function Parameters**: Functions that should not mutate their input arrays.

### Common Mistakes

**Mutating Through Type Assertion**: Using s to bypass readonly checks defeats the purpose.

**Not Using readonly for Parameters**: Functions that don't mutate should accept eadonly T[].

### Best Practices

1. Use eadonly T[] for function parameters that won't mutate the array
2. Use ReadonlyArray<T> when you need the generic syntax for consistency
3. Prefer eadonly over manual immutability conventions
4. Use s const for literal array immutability

### Performance Considerations

Zero runtime cost — eadonly is purely a compile-time check.

### Interview Questions

1. What is the difference between eadonly T[] and T[]?
2. How do you create a deep readonly array type?
3. Can a eadonly T[] be assigned to T[]?

### Coding Challenges

1. Create a function that safely merges multiple readonly arrays
2. Implement a ReadonlyArray wrapper class

### Related Topics

- Readonly Properties
- as const Assertions
- Immutability Patterns

## Array method type safety

### What It Is

TypeScript provides full type information for all array methods (map, ilter, educe, ind, orEach, etc.), ensuring that callbacks and return values are properly typed.

### Why It Is Important

Type-safe array methods prevent common bugs like incorrect callback parameters, wrong return types from map, or missing undefined checks from ind. They also enable better autocompletion and inline documentation.

### How It Works Internally

Array methods are defined in lib.es5.d.ts (and extended in later ES versions) as generic methods on the Array<T> interface. Each method has type parameters that capture the element type and relationship between input and output types.

### Syntax

`	ypescript
const numbers: number[] = [1, 2, 3, 4, 5];

// map — transforms each element
const doubled: number[] = numbers.map((n) => n * 2);

// filter — narrows type
const evens: number[] = numbers.filter((n) => n % 2 === 0);

// reduce — accumulates
const sum: number = numbers.reduce((acc, n) => acc + n, 0);

// find — returns T | undefined
const firstEven: number | undefined = numbers.find((n) => n % 2 === 0);

// forEach — side effects
numbers.forEach((n) => console.log(n));
`

### Beginner Examples

`	ypescript
const fruits: string[] = ["apple", "banana", "cherry"];

// map
const uppercased: string[] = fruits.map((f) => f.toUpperCase());

// filter
const longFruits: string[] = fruits.filter((f) => f.length > 5);

// find
const apple: string | undefined = fruits.find((f) => f.startsWith("a"));

// includes
const hasBanana: boolean = fruits.includes("banana");

// indexOf
const index: number = fruits.indexOf("banana");
`

### Intermediate Examples

`	ypescript
interface User {
  id: number;
  name: string;
  age: number;
  isActive: boolean;
}

const users: User[] = [
  { id: 1, name: "Alice", age: 30, isActive: true },
  { id: 2, name: "Bob", age: 25, isActive: false },
  { id: 3, name: "Charlie", age: 35, isActive: true },
];

// Filter with type predicate
function isActiveUser(user: User): user is User {
  return user.isActive;
}
const activeUsers: User[] = users.filter(isActiveUser);

// Map to a different type
const names: string[] = users.map((u) => u.name);

// Sort (mutates!)
const sortedByAge: User[] = [...users].sort((a, b) => a.age - b.age);

// Complex reduce
const groupedByActive: Record<string, User[]> = users.reduce(
  (acc, user) => {
    const key = user.isActive ? "active" : "inactive";
    acc[key].push(user);
    return acc;
  },
  { active: [], inactive: [] } as Record<string, User[]>
);

// every and some
const allActive: boolean = users.every((u) => u.isActive);
const anyActive: boolean = users.some((u) => u.isActive);
`

### Advanced Examples

`	ypescript
// Reduce with different accumulator type
interface UserSummary {
  totalAge: number;
  activeCount: number;
  names: string[];
}

const summary: UserSummary = users.reduce<UserSummary>(
  (acc, user) => ({
    totalAge: acc.totalAge + user.age,
    activeCount: acc.activeCount + (user.isActive ? 1 : 0),
    names: [...acc.names, user.name],
  }),
  { totalAge: 0, activeCount: 0, names: [] }
);

// flatMap (ES2019+)
const nested: number[][] = [[1, 2], [3, 4], [5]];
const flat: number[] = nested.flatMap((arr) => arr);

// Type-safe sort with comparator
function sortBy<T, K extends string | number>(
  items: readonly T[],
  keyFn: (item: T) => K
): T[] {
  return [...items].sort((a, b) => {
    const ka = keyFn(a);
    const kb = keyFn(b);
    if (ka < kb) return -1;
    if (ka > kb) return 1;
    return 0;
  });
}

const sortedByName = sortBy(users, (u) => u.name);

// Chaining with type safety
const result: string[] = users
  .filter((u) => u.isActive)
  .map((u) => u.name.toUpperCase())
  .sort();

// Group by with Map
function groupBy<T, K>(items: readonly T[], keyFn: (item: T) => K): Map<K, T[]> {
  return items.reduce((map, item) => {
    const key = keyFn(item);
    const group = map.get(key) ?? [];
    group.push(item);
    map.set(key, group);
    return map;
  }, new Map<K, T[]>());
}
`

### Real-World Use Cases

**Data Transformation Pipelines**: Chaining ilter -> map -> educe for ETL operations.

**UI Data Formatting**: Transforming API responses into UI-ready data structures.

**Validation**: Using every and some for array validation checks.

### Common Mistakes

**Ignoring undefined from find**: Array.find returns T | undefined — must check before use.

**Calling sort on readonly arrays**: sort mutates in place. Copy first with spread.

**Assuming filter narrows types**: ilter does not narrow the element type unless a type predicate is used.

### Best Practices

1. Always handle undefined from ind
2. Copy arrays before sort to avoid mutation
3. Use type predicates in ilter for type narrowing
4. Prefer latMap over map + lat
5. Use educe with explicit type parameter for non-matching accumulator types

### Performance Considerations

Array methods have normal JavaScript performance characteristics. educe is generally efficient. Chaining creates intermediate arrays — for large arrays, consider a single loop or transducers.

### Interview Questions

1. How does filter affect the return type?
2. Why does find return T | undefined?
3. How do you properly type reduce for complex accumulators?

### Coding Challenges

1. Implement a type-safe groupBy function
2. Create a pipe function that chains array operations with proper typing

### Related Topics

- Array Methods Reference
- Functional Programming in TypeScript
- Type Predicates

## Multidimensional arrays

### What It Is

Multidimensional arrays are arrays of arrays. TypeScript allows typing them with nested brackets: T[][], T[][][], etc., representing arrays at each nesting level.

### Why It Is Important

Multidimensional arrays represent grids, matrices, tables, and other multi-index data structures. Proper typing ensures correct access patterns and prevents mixing dimensions.

### How It Works Internally

Each nested [] represents another array level. 
umber[][] is Array<Array<number>>. TypeScript checks access at each level: matrix[0] returns 
umber[], matrix[0][1] returns 
umber.

### Syntax

`	ypescript
// Two-dimensional array
const matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
];

// Three-dimensional
const cube: number[][][] = [
  [[1, 2], [3, 4]],
  [[5, 6], [7, 8]],
];

// Generic syntax
const matrix2: Array<Array<number>> = matrix;

// Mixed dimensions
const table: (string | number)[][] = [
  ["Name", "Age", "City"],
  ["Alice", 30, "NYC"],
];
`

### Beginner Examples

`	ypescript
// 2D grid
const grid: boolean[][] = [
  [true, false, true],
  [false, true, false],
  [true, true, false],
];

// Access
const firstRow: boolean[] = grid[0];
const cell: boolean = grid[0][1];

// Iterate
for (const row of grid) {
  for (const cell of row) {
    console.log(cell);
  }
}

// Create empty 2D array
const empty: number[][] = Array(3).fill(null).map(() => Array(4).fill(0));
`

### Intermediate Examples

`	ypescript
// Matrix operations
function transpose<T>(matrix: T[][]): T[][] {
  if (matrix.length === 0) return [];
  const rows = matrix.length;
  const cols = matrix[0].length;
  const result: T[][] = [];
  for (let c = 0; c < cols; c++) {
    result[c] = [];
    for (let r = 0; r < rows; r++) {
      result[c][r] = matrix[r][c];
    }
  }
  return result;
}

const transposed = transpose([
  [1, 2, 3],
  [4, 5, 6],
]);

// Jagged arrays (different row lengths)
const jagged: number[][] = [
  [1, 2],
  [3, 4, 5],
  [6],
];

// Typed cell values
type Cell = { value: number; visited: boolean };

const grid2: Cell[][] = [
  [{ value: 1, visited: false }, { value: 2, visited: true }],
  [{ value: 3, visited: false }, { value: 4, visited: false }],
];
`

### Advanced Examples

`	ypescript
// Generic matrix class
class Matrix<T> {
  constructor(
    private readonly data: T[][],
    public readonly rows: number,
    public readonly cols: number
  ) {}

  get(row: number, col: number): T {
    return this.data[row][col];
  }

  map<U>(fn: (value: T, row: number, col: number) => U): Matrix<U> {
    const newData = this.data.map((row, r) =>
      row.map((cell, c) => fn(cell, r, c))
    );
    return new Matrix(newData, this.rows, this.cols);
  }

  toArray(): T[][] {
    return this.data.map((row) => [...row]);
  }
}

// 3D tensor representation
type Tensor3D<T> = T[][][];

function createTensor3D<T>(
  d1: number,
  d2: number,
  d3: number,
  fill: T
): Tensor3D<T> {
  return Array.from({ length: d1 }, () =>
    Array.from({ length: d2 }, () => Array(d3).fill(fill))
  );
}

// Deep readonly multidimensional array
type DeepReadonlyArray2<T> = readonly (T extends readonly any[]
  ? DeepReadonlyArray2<T[number]>
  : T)[];

type ReadonlyMatrix = DeepReadonlyArray2<number[][]>;
`

### Real-World Use Cases

**Image Processing**: 2D arrays of pixels (grayscale) or 3D arrays (RGB).

**Spreadsheets**: 2D arrays for cell data with mixed types.

**Game Boards**: Grid-based games (chess, tic-tac-toe, minesweeper).

**Machine Learning**: Tensors of arbitrary dimensions.

### Common Mistakes

**Confusing Dimensions**: Accidentally swapping row and column indices.

**Creating Sparse Arrays**: 
ew Array(3).fill(new Array(3).fill(0)) shares the same inner array.

**Type Widening in Mixed Arrays**: Without explicit typing, [[1, "a"], [2, "b"]] is (string | number)[][].

### Best Practices

1. Use type aliases for dimension-specific types: 	ype Matrix = number[][]
2. Be explicit with jagged array types
3. Avoid mutating multidimensional arrays in place
4. Use classes for complex matrix operations

### Performance Considerations

Nested loops over multidimensional arrays have typical O(n*m) performance. For large grids, consider typed arrays (Float64Array, etc.).

### Interview Questions

1. How do you type a 2D array in TypeScript?
2. What is the difference between 
umber[][] and Array<Array<number>>?
3. How do you transpose a matrix with proper typing?

### Coding Challenges

1. Implement a generic Matrix class with type-safe operations
2. Write a function that flattens a multidimensional array of any depth
3. Create a type-safe function to rotate a 2D matrix 90 degrees

### Related Topics

- Typed Arrays
- Generic Classes
- Recursive Types
