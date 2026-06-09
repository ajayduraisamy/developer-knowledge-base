# Namespaces - namespace keyword, nested namespaces, aliases, vs ES modules

## Introduction
Namespaces (formerly called internal modules) are a TypeScript-specific feature for organizing code into logical groups within a single file or across files. They predate ES modules and provide a way to avoid global scope pollution by creating named containers for related declarations. While ES modules are now the recommended approach, namespaces remain useful for certain scenarios like packaging global libraries, organizing declaration files, and maintaining legacy codebases.

## namespace keyword

### What It Is
The `namespace` keyword defines a named scope that groups related types, functions, classes, and variables together. Members are accessed via dot notation from the namespace name.

### Why It Is Important
Namespaces prevent naming collisions in the global scope and logically group related code. They are particularly useful in scenarios where ES module loading is not available or when building self-contained script bundles.

### How It Works Internally
TypeScript compiles namespaces into JavaScript IIFE (Immediately Invoked Function Expression) patterns that assign members to an object. Nested namespaces become nested objects.

### Syntax
```typescript
namespace MyNamespace {
  export interface MyInterface { }
  export function myFunction(): void { }
}
```

### Beginner Examples
```typescript
namespace MathUtils {
  export const PI = 3.14159;
  export function add(a: number, b: number): number {
    return a + b;
  }
  export function multiply(a: number, b: number): number {
    return a * b;
  }
}
console.log(MathUtils.PI);
console.log(MathUtils.add(2, 3));
```

### Intermediate Examples
```typescript
namespace Validation {
  export interface Rule {
    validate(value: unknown): boolean;
    message: string;
  }
  export class Required implements Rule {
    message = "This field is required";
    validate(value: unknown): boolean {
      return value !== null && value !== undefined && value !== "";
    }
  }
  export class Email implements Rule {
    message = "Invalid email";
    validate(value: unknown): boolean {
      const str = String(value);
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(str);
    }
  }
  export function validateAll(value: unknown, rules: Rule[]): string[] {
    return rules.filter(r => !r.validate(value)).map(r => r.message);
  }
}
const errors = Validation.validateAll("test", [new Validation.Required(), new Validation.Email()]);
```

### Advanced Examples
```typescript
namespace Database {
  export namespace Pool {
    export interface Config {
      max: number;
      idleTimeout: number;
    }
    export function create(config: Config): Pool {
      return new PoolImpl(config);
    }
    class PoolImpl {
      constructor(private config: Config) { }
      async query(sql: string): Promise<unknown[]> {
        return [];
      }
    }
    export function execute(pool: PoolImpl, sql: string): Promise<unknown[]> {
      return pool.query(sql);
    }
  }
  export namespace Migration {
    export interface Migration {
      version: number;
      up(): Promise<void>;
      down(): Promise<void>;
    }
    export async function run(migrations: Migration[]): Promise<void> {
      for (const m of migrations) {
        await m.up();
      }
    }
  }
  export type Dialect = "postgres" | "mysql" | "sqlite";
  export const SUPPORTED_DIALECTS: Dialect[] = ["postgres", "mysql", "sqlite"];
}
```

### Real-World Use Cases
- Packaging utility functions for browser scripts without bundlers
- Organizing large declaration files (.d.ts)
- Grouping related types and interfaces in legacy codebases
- Creating logical boundaries in single-file modules

### Common Mistakes
- Forgetting the `export` keyword on members that should be accessible outside the namespace
- Using namespaces in ES module files (mix of paradigms)
- Creating deeply nested namespaces that are hard to navigate

### Best Practices
- Use ES modules for new projects; use namespaces only when necessary
- Always export members that need to be accessible outside the namespace
- Keep namespaces flat and focused

### Performance Considerations
Namespaces compile to JavaScript objects with IIFE initialization. There is a small performance cost for the IIFE execution at load time, but it is negligible.

### Interview Questions
- Q: What is the difference between a namespace and a module?
  A: Namespaces are TypeScript-specific and compile to nested objects; modules (ES modules) are the JavaScript standard for code organization.

### Coding Challenges
- Create a `StringUtils` namespace with functions for `capitalize`, `truncate`, and `toKebabCase`.
- Build a `Geometry` namespace with sub-namespaces for `Shapes` and `Calculations`.

### Related Topics
- Nested namespaces
- Namespace aliases
- Namespaces vs ES modules

---

## Nested namespaces

### What It Is
Nested namespaces are namespaces declared inside another namespace, creating a hierarchical organization structure accessed via chained dot notation.

### Why It Is Important
Nested namespaces allow for multi-level logical grouping, reflecting complex domain hierarchies or layered architectures within a single namespace tree.

### How It Works Internally
The compiler generates nested JavaScript objects corresponding to the namespace hierarchy. Each nested namespace becomes a property of its parent namespace object.

### Syntax
```typescript
namespace Outer {
  export namespace Inner {
    export const value = 42;
  }
}
// Access: Outer.Inner.value
```

### Beginner Examples
```typescript
namespace App {
  export namespace Config {
    export const PORT = 3000;
    export const HOST = "localhost";
    export function getURL(): string {
      return "http://" + HOST + ":" + PORT;
    }
  }
  export namespace Utils {
    export function delay(ms: number): Promise<void> {
      return new Promise(resolve => setTimeout(resolve, ms));
    }
  }
}
console.log(App.Config.getURL());
await App.Utils.delay(1000);
```

### Intermediate Examples
```typescript
namespace Company {
  export namespace HR {
    export interface Employee {
      id: string;
      name: string;
      department: string;
    }
    export class EmployeeService {
      private employees: Employee[] = [];
      add(emp: Employee): void {
        this.employees.push(emp);
      }
      findByDepartment(dept: string): Employee[] {
        return this.employees.filter(e => e.department === dept);
      }
    }
  }
  export namespace Finance {
    export interface Invoice {
      id: string;
      amount: number;
      issuedAt: Date;
    }
    export class BillingService {
      private invoices: Invoice[] = [];
      createInvoice(amount: number): Invoice {
        const inv: Invoice = {
          id: crypto.randomUUID(),
          amount,
          issuedAt: new Date(),
        };
        this.invoices.push(inv);
        return inv;
      }
    }
  }
  export namespace Common {
    export type ID = string;
    export function generateId(): ID {
      return crypto.randomUUID();
    }
  }
}
```

### Advanced Examples
```typescript
namespace API {
  export namespace V1 {
    export namespace Auth {
      export interface LoginRequest {
        username: string;
        password: string;
      }
      export interface LoginResponse {
        token: string;
        expiresIn: number;
      }
      export async function login(req: LoginRequest): Promise<LoginResponse> {
        const res = await fetch("/api/v1/auth/login", {
          method: "POST",
          body: JSON.stringify(req),
        });
        return res.json();
      }
    }
    export namespace Users {
      export interface User {
        id: string;
        name: string;
        email: string;
      }
      export async function getById(id: string): Promise<User> {
        const res = await fetch("/api/v1/users/" + id);
        return res.json();
      }
      export async function list(): Promise<User[]> {
        const res = await fetch("/api/v1/users");
        return res.json();
      }
    }
    export namespace Products {
      export interface Product {
        id: string;
        title: string;
        price: number;
      }
      export namespace Categories {
        export interface Category {
          id: string;
          name: string;
        }
        export async function list(): Promise<Category[]> {
          const res = await fetch("/api/v1/products/categories");
          return res.json();
        }
      }
    }
  }
}
// Usage
const token = await API.V1.Auth.login({ username: "admin", password: "secret" });
const user = await API.V1.Users.getById("123");
const categories = await API.V1.Products.Categories.list();
```

### Real-World Use Cases
- Organizing API client libraries with versioned endpoints
- Structuring large frameworks with layered namespaces
- Grouping domain models by bounded context

### Common Mistakes
- Creating deeply nested (3+ levels) namespaces that are cumbersome to use
- Repeating the full namespace path when importing
- Mixing nested namespaces with ES modules in the same project

### Best Practices
- Limit nesting to 2-3 levels maximum
- Use aliases (`import Name = Namespace.Name`) to reduce verbosity
- Consider whether a separate module would be clearer than a nested namespace

### Performance Considerations
Deeply nested namespaces generate deeply nested JavaScript objects. The initialization cost scales with nesting depth but remains negligible.

### Interview Questions
- Q: How do you access a member of a nested namespace from outside?
  A: Using chained dot notation: `Outer.Inner.InnerMost.value`.

### Coding Challenges
- Create a `MyApp` namespace with nested `Services`, `Models`, and `Utils` namespaces.
- Build a `Shipping` namespace with nested `Rates`, `Tracking`, and `Labels` namespaces.

### Related Topics
- namespace keyword
- Namespace aliases
- Namespaces vs ES modules

---

## Namespace aliases

### What It Is
Namespace aliases provide a shorthand name for accessing a deeply nested namespace member using the `import Alias = Namespace.Path` syntax.

### Why It Is Important
Aliases reduce verbosity when repeatedly accessing deeply nested namespace members, improving code readability and maintainability.

### How It Works Internally
TypeScript compiles the alias into a variable assignment that references the nested namespace object. It does not create a copy — it is a reference.

### Syntax
```typescript
import Alias = Namespace.NestedNamespace;
```

### Beginner Examples
```typescript
namespace App {
  export namespace Config {
    export namespace Database {
      export const HOST = "localhost";
      export const PORT = 5432;
      export const NAME = "mydb";
    }
  }
}
import DB = App.Config.Database;
console.log(DB.HOST);
console.log(DB.PORT);
```

### Intermediate Examples
```typescript
namespace Geometry {
  export namespace ThreeD {
    export namespace VectorOps {
      export function dot(a: number[], b: number[]): number {
        return a.reduce((sum, v, i) => sum + v * b[i], 0);
      }
      export function cross(a: number[], b: number[]): number[] {
        return [
          a[1] * b[2] - a[2] * b[1],
          a[2] * b[0] - a[0] * b[2],
          a[0] * b[1] - a[1] * b[0],
        ];
      }
      export function magnitude(v: number[]): number {
        return Math.sqrt(v.reduce((s, x) => s + x * x, 0));
      }
    }
  }
}
import Vec3 = Geometry.ThreeD.VectorOps;
const a = [1, 0, 0];
const b = [0, 1, 0];
console.log(Vec3.dot(a, b));
console.log(Vec3.cross(a, b));
```

### Advanced Examples
```typescript
namespace MyEnterprise {
  export namespace Infrastructure {
    export namespace Persistence {
      export namespace ORM {
        export interface Entity {
          id: string;
        }
        export class Repository<T extends Entity> {
          protected items = new Map<string, T>();
          find(id: string): T | undefined {
            return this.items.get(id);
          }
          save(entity: T): void {
            this.items.set(entity.id, entity);
          }
          delete(id: string): void {
            this.items.delete(id);
          }
        }
      }
    }
  }
  export namespace Domain {
    export namespace UserManagement {
      export interface User extends Infrastructure.Persistence.ORM.Entity {
        name: string;
        email: string;
      }
      export class UserRepository extends Infrastructure.Persistence.ORM.Repository<User> {
        findByEmail(email: string): User | undefined {
          for (const user of this.items.values()) {
            if (user.email === email) return user;
          }
          return undefined;
        }
      }
    }
  }
}
import ORM = MyEnterprise.Infrastructure.Persistence.ORM;
import UserDomain = MyEnterprise.Domain.UserManagement;
const repo = new UserDomain.UserRepository();
repo.save({ id: "1", name: "Alice", email: "alice@example.com" });
```

### Real-World Use Cases
- Large enterprise applications with deeply namespaced architectures
- Libraries that provide lengthy fully-qualified namespace paths
- Migration code where both old and new namespace paths are used

### Common Mistakes
- Using aliases for single-access patterns (just use the full path)
- Confusing namespace aliases with ES module imports
- Forgetting that aliases are references, not copies (mutations affect the original)

### Best Practices
- Use aliases when a deeply nested namespace member is accessed 3+ times
- Name aliases to be shorter but still descriptive
- Place aliases at the top of the file for visibility

### Performance Considerations
Aliases are just local variables referencing the namespace object. They have no meaningful performance impact.

### Interview Questions
- Q: Can you create an alias for an interface inside a namespace?
  A: Yes: `import MyInterface = Namespace.MyInterface`.

### Coding Challenges
- Create a deeply nested namespace (4 levels) and use aliases to simplify access.
- Refactor a codebase of repetitive full namespace paths into clean aliases.

### Related Topics
- namespace keyword
- Nested namespaces
- Namespaces vs ES modules

---

## Namespaces vs ES modules

### What It Is
Namespaces are a TypeScript-specific code organization feature that predates ES modules. ES modules are the standardized JavaScript module system. Both serve to organize code but differ fundamentally in semantics and use cases.

### Why It Is Important
Understanding the differences helps developers choose the right tool. ES modules are the recommended approach for modern TypeScript projects, while namespaces remain useful in specific scenarios.

### How It Works Internally
Namespaces compile to IIFE-assigned objects attached to the global scope or parent namespace. ES modules compile to `import` and `export` statements that the runtime or bundler resolves.

### Syntax
```typescript
// Namespace
namespace Foo {
  export const x = 1;
}

// ES Module
// foo.ts
export const x = 1;
// bar.ts
import { x } from "./foo.js";
```

### Beginner Examples
```typescript
// Namespace approach
namespace Calculator {
  export function add(a: number, b: number): number {
    return a + b;
  }
}
console.log(Calculator.add(2, 3));

// ES Module approach
// calculator.ts
export function add(a: number, b: number): number {
  return a + b;
}
// app.ts
import { add } from "./calculator.js";
console.log(add(2, 3));
```

### Intermediate Examples
```typescript
// Namespace: all code in one file or referenced via triple-slash directives
namespace App.Models {
  export interface User { name: string; }
}
namespace App.Services {
  export class UserService { }
}

// ES Modules: each file is its own module
// models/user.ts
export interface User { name: string; }
// services/user-service.ts
import type { User } from "../models/user.js";
export class UserService { }
```

### Advanced Examples
```typescript
// Namespace: global library distribution (single bundle)
namespace MyLib {
  export function init(): void { }
  export namespace Components {
    export class Button { }
    export class Input { }
  }
}
// Accessible globally after script load
MyLib.init();
new MyLib.Components.Button();

// ES Module: import-based consumption
// index.ts
export function init(): void { }
export * as Components from "./components/index.js";

// consumer.ts
import { init, Components } from "./my-lib/index.js";
init();
new Components.Button();
```

### Real-World Use Cases
- Namespaces: browser globals, legacy codebases, declaration files (.d.ts) for libraries
- ES modules: modern applications, libraries published to npm, code-split bundles

### Common Mistakes
- Mixing namespaces and ES modules in the same project without clear boundaries
- Using namespaces when ES modules would provide better tooling and tree-shaking
- Assuming namespaces support hot-module replacement or code splitting

### Best Practices
- Default to ES modules for all new code
- Use namespaces only in `.d.ts` files or when bundling for the browser without a bundler
- Never mix `namespace` and `export` at the top level of an ES module file

### Performance Considerations
ES modules enable tree-shaking (dead code elimination) and lazy loading. Namespaces cannot be tree-shaken as effectively because they are bundled as single objects.

### Interview Questions
- Q: Why did TypeScript deprecate the `module` keyword for internal modules?
  A: To avoid confusion with ES modules. The `namespace` keyword was introduced to clarify the distinction.

### Coding Challenges
- Convert a project using namespaces to use ES modules instead.
- Create a library that exposes both a namespace-based global script and ES module entry points.

### Related Topics
- ES Modules
- Declaration files
- Triple-slash directives
