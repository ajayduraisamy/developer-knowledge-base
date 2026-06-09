# Proxy

## Introduction

The Proxy object enables custom behavior for fundamental operations on objects. It acts as a wrapper that intercepts and redefines operations like property access, assignment, enumeration, function invocation, and more. Proxies are the most powerful meta-programming tool in JavaScript, enabling features like reactivity, validation, logging, and access control.

---

## Proxy constructor

### What It Is

The `Proxy` constructor creates a proxy object that wraps a target object. It takes two arguments: the target object and a handler object that defines traps (interceptors) for operations. Every operation on the proxy is forwarded to the corresponding handler trap if defined, or passed through to the target if not.

```javascript
const target = { message: "Hello" };
const handler = {
  get(obj, prop) {
    return prop in obj ? obj[prop] : `Property ${prop} not found`;
  },
};

const proxy = new Proxy(target, handler);
console.log(proxy.message); // "Hello"
console.log(proxy.unknown); // "Property unknown not found"
```

### Why It Is Important

Proxies enable capabilities that were previously impossible or required complex workarounds:
- Intercept and customize object operations.
- Implement reactive data binding without dirty checking.
- Create virtualized objects (properties that don't exist yet).
- Add validation on property writes.
- Hide or protect properties dynamically.
- Create sandboxes for untrusted code.

### How It Works Internally

When an operation (e.g., `proxy.prop`) occurs, the engine checks if the proxy has a handler trap for that operation (e.g., `get`). If yes, it calls the trap with the relevant arguments. If not, it forwards the operation directly to the target. The proxy internally stores references to the target and handler, which cannot be changed after creation.

### Syntax

```javascript
const proxy = new Proxy(target, handler);

// target: The original object to wrap
// handler: Object with trap methods

// Revocable proxy (can be revoked)
const { proxy, revoke } = Proxy.revocable(target, handler);
revoke(); // Proxy becomes unusable
```

### Beginner Examples

```javascript
// Example 1: Default pass-through proxy
const target = { x: 1, y: 2 };
const proxy = new Proxy(target, {});
console.log(proxy.x); // 1
proxy.z = 3;
console.log(target.z); // 3 (proxied operations affect target)

// Example 2: Logging proxy
const logger = {
  get(target, prop) {
    console.log(`Accessing property "${String(prop)}"`);
    return target[prop];
  },
};
const logged = new Proxy({ a: 10, b: 20 }, logger);
console.log(logged.a); // Logs: Accessing property "a" then 10

// Example 3: Revocable proxy
const { proxy: revocable, revoke } = Proxy.revocable(
  { data: "secret" },
  {}
);
console.log(revocable.data); // "secret"
revoke();
// console.log(revocable.data); // TypeError: Cannot perform 'get' on a proxy that has been revoked
```

### Intermediate Examples

```javascript
// Example 1: Virtual properties
const virtual = new Proxy(
  {},
  {
    get(target, prop) {
      if (prop === "currentTime") return Date.now();
      if (prop === "random") return Math.random();
      return undefined;
    },
  }
);
console.log(virtual.currentTime); // Current timestamp
console.log(virtual.random); // Random number

// Example 2: Default values
function withDefaults(target, defaults) {
  return new Proxy(target, {
    get(target, prop) {
      if (prop in target) return target[prop];
      return defaults[prop];
    },
  });
}
const config = withDefaults(
  { port: 3000 },
  { host: "localhost", debug: false }
);
console.log(config.port); // 3000
console.log(config.host); // "localhost" (default)

// Example 3: Array negative indexing
const negativeArray = (arr) =>
  new Proxy(arr, {
    get(target, prop) {
      const index = Number(prop);
      if (index < 0) {
        return target[target.length + index];
      }
      return target[prop];
    },
    set(target, prop, value) {
      const index = Number(prop);
      if (index < 0) {
        target[target.length + index] = value;
      } else {
        target[prop] = value;
      }
      return true;
    },
  });

const arr = negativeArray([1, 2, 3]);
console.log(arr[-1]); // 3
console.log(arr[-2]); // 2
```

### Advanced Examples

```javascript
// Example 1: Deep proxy for reactive state
function deepProxy(target, onChange) {
  const handler = {
    get(target, prop, receiver) {
      const value = Reflect.get(target, prop, receiver);
      if (typeof value === "object" && value !== null) {
        return deepProxy(value, onChange);
      }
      return value;
    },
    set(target, prop, value, receiver) {
      const oldValue = target[prop];
      const result = Reflect.set(target, prop, value, receiver);
      onChange(prop, value, oldValue);
      return result;
    },
  };
  return new Proxy(target, handler);
}

const state = deepProxy({ user: { name: "Alice", age: 30 } }, (prop, value, old) => {
  console.log(`Changed: ${String(prop)} from ${old} to ${value}`);
});
state.user.name = "Bob"; // Changed: name from Alice to Bob

// Example 2: Singleton proxy
function singleton(Class) {
  let instance;
  return new Proxy(Class, {
    construct(target, args) {
      if (!instance) {
        instance = new target(...args);
      }
      return instance;
    },
  });
}

// Example 3: Sandbox proxy
function createSandbox(context) {
  return new Proxy(
    {},
    {
      has(target, prop) {
        return prop in context || prop === "console";
      },
      get(target, prop) {
        if (prop === "console") return console;
        if (prop in context) return context[prop];
        return undefined;
      },
      set(target, prop, value) {
        context[prop] = value;
        return true;
      },
    }
  );
}
```

### Real-World Use Cases

- **Vue 3 reactivity** — Proxies power the reactive system.
- **MobX** — Observable state via proxies.
- **Immer** — Draft proxies for immutable state.
- **Testing mocks** — Jest, Sinon use proxies for spies.
- **ORM lazy loading** — Models that fetch data on property access.

### Common Mistakes

- **Assuming proxy === target** — `proxy === target` is false.
- **Forgetting that proxies affect the target** — Mutations flow through.
- **Creating proxies inside tight loops** — Each proxy creation has overhead.
- **Not handling all necessary traps** — Missing traps may expose the target.
- **Using `this` in traps without caution** — `this` in traps refers to the handler.

### Best Practices

- Use the simplest proxy that achieves your goal.
- Always use `Reflect.*` inside traps for correct forwarding.
- Use `Proxy.revocable` when you need to clean up.
- Document what each trap does.
- Profile proxy usage in performance-critical paths.

### Performance Considerations

- Each intercepted operation has overhead (function call to the trap).
- Creating proxies is relatively cheap.
- Heavy use of get/set traps can slow down property access by 10-50x.
- V8 optimizes proxies with simple handlers better than complex ones.
- Measure before optimizing — proxies are often fast enough.

### Interview Questions

1. **What is a Proxy?**
   An object that wraps another object and intercepts operations via handler traps.

2. **What is the difference between Proxy and Object.defineProperty?**
   Proxies intercept ALL operations on the entire object. defineProperty works on single properties.

3. **How do you revoke a Proxy?**
   Using `Proxy.revocable()` which returns `{ proxy, revoke }`. Call `revoke()` to disable.

4. **Can a Proxy intercept function calls?**
   Yes, using the `apply` trap for functions and `construct` trap for constructors.

### Coding Challenges

1. **Create a proxy that logs all property access.**
2. **Build a validation proxy for a user object (age > 0, name required).**
3. **Implement an observable proxy that notifies subscribers on change.**
4. **Create a debouncing proxy for expensive property access.**
5. **Build a simple ORM with lazy-loaded relations using Proxy.**

### Related Topics

- Reflect API
- Handler traps
- Proxy.revocable()
- Meta-programming
- Reactive programming

---

## Handler traps (get, set, has, deleteProperty)

### What It Is

Handler traps are methods in the Proxy handler object that intercept specific operations. Each trap corresponds to a fundamental operation: `get` for property access, `set` for property assignment, `has` for the `in` operator, `deleteProperty` for `delete`, and many more. There are 13 available traps in total.

```javascript
const handler = {
  get(target, prop, receiver) { /* intercept reading */ },
  set(target, prop, value, receiver) { /* intercept writing */ },
  has(target, prop) { /* intercept 'in' operator */ },
  deleteProperty(target, prop) { /* intercept 'delete' */ },
};
```

### Why It Is Important

Handler traps provide fine-grained control over object behavior:
- `get` — Implement virtual properties, logging, access control.
- `set` — Validate, transform, or reject property values.
- `has` — Hide properties from the `in` operator.
- `deleteProperty` — Prevent deletion of protected properties.
- They form the building blocks of all Proxy-based patterns.

### How It Works Internally

Each trap is called by the engine when the corresponding operation is performed on the proxy. The trap receives the original target and operation-specific arguments. If the trap returns a value, the engine uses it. If the trap doesn't exist, the engine performs the default behavior on the target.

### Syntax

```javascript
const handler = {
  // Property access: proxy.prop
  get(target, property, receiver) { return value; },

  // Property assignment: proxy.prop = value
  set(target, property, value, receiver) { return boolean; },

  // in operator: "prop" in proxy
  has(target, property) { return boolean; },

  // delete operator: delete proxy.prop
  deleteProperty(target, property) { return boolean; },

  // Additional traps:
  apply(target, thisArg, args) { },
  construct(target, args, newTarget) { },
  getPrototypeOf(target) { },
  setPrototypeOf(target, proto) { },
  isExtensible(target) { },
  preventExtensions(target) { },
  getOwnPropertyDescriptor(target, prop) { },
  defineProperty(target, prop, desc) { },
  ownKeys(target) { },
};
```

### Beginner Examples

```javascript
// Example 1: get trap - virtual properties and logging
const handler1 = {
  get(target, prop) {
    if (prop === "timestamp") return Date.now();
    if (prop in target) return target[prop];
    return `Property "${String(prop)}" does not exist`;
  },
};
const p1 = new Proxy({ name: "Alice" }, handler1);
console.log(p1.name); // "Alice"
console.log(p1.timestamp); // Current timestamp
console.log(p1.xyz); // "Property "xyz" does not exist"

// Example 2: set trap - validation
const handler2 = {
  set(target, prop, value) {
    if (prop === "age") {
      if (typeof value !== "number") throw new TypeError("Age must be a number");
      if (value < 0 || value > 150) throw new RangeError("Invalid age");
    }
    target[prop] = value;
    return true;
  },
};
const p2 = new Proxy({}, handler2);
p2.age = 25; // OK
// p2.age = -5; // RangeError

// Example 3: has trap - hiding properties
const handler3 = {
  has(target, prop) {
    if (prop.startsWith("_")) return false;
    return prop in target;
  },
};
const p3 = new Proxy({ _secret: "hidden", public: "visible" }, handler3);
console.log("public" in p3); // true
console.log("_secret" in p3); // false

// Example 4: deleteProperty trap - protecting properties
const handler4 = {
  deleteProperty(target, prop) {
    if (prop === "protected") {
      throw new Error(`Cannot delete ${String(prop)}`);
    }
    return delete target[prop];
  },
};
const p4 = new Proxy({ protected: "safe", free: "removable" }, handler4);
delete p4.free; // OK
// delete p4.protected; // Error
```

### Intermediate Examples

```javascript
// Example 1: Read-only proxy
function readonly(obj) {
  return new Proxy(obj, {
    set() { return false; },
    deleteProperty() { return false; },
    defineProperty() { return false; },
    setPrototypeOf() { return false; },
  });
}
const config = readonly({ api: "https://api.example.com", key: "secret" });
// config.api = "other"; // silently fails

// Example 2: Property renaming
function rename(obj, mapping) {
  return new Proxy(obj, {
    get(target, prop) {
      const mapped = mapping[prop] || prop;
      return target[mapped];
    },
    set(target, prop, value) {
      const mapped = mapping[prop] || prop;
      target[mapped] = value;
      return true;
    },
    has(target, prop) {
      const mapped = mapping[prop] || prop;
      return mapped in target;
    },
  });
}
const user = rename(
  { first_name: "John", last_name: "Doe" },
  { firstName: "first_name", lastName: "last_name" }
);
console.log(user.firstName); // "John"

// Example 3: Memoization proxy
function memoized(target) {
  const cache = new Map();
  return new Proxy(target, {
    get(target, prop) {
      if (typeof target[prop] !== "function") return target[prop];
      return function (...args) {
        const key = `${String(prop)}:${JSON.stringify(args)}`;
        if (cache.has(key)) return cache.get(key);
        const result = target[prop](...args);
        cache.set(key, result);
        return result;
      };
    },
  });
}
```

### Advanced Examples

```javascript
// Example 1: Computed properties with get/set
function computedProps(obj, computed) {
  return new Proxy(obj, {
    get(target, prop, receiver) {
      if (prop in computed) {
        return Reflect.apply(computed[prop].get, receiver, []);
      }
      return Reflect.get(target, prop, receiver);
    },
    set(target, prop, value, receiver) {
      if (prop in computed && computed[prop].set) {
        Reflect.apply(computed[prop].set, receiver, [value]);
        return true;
      }
      return Reflect.set(target, prop, value, receiver);
    },
    has(target, prop) {
      return prop in computed || prop in target;
    },
  });
}

const user = computedProps(
  { firstName: "John", lastName: "Doe" },
  {
    fullName: {
      get() { return `${this.firstName} ${this.lastName}`; },
      set(val) { [this.firstName, this.lastName] = val.split(" "); },
    },
  }
);
console.log(user.fullName); // "John Doe"
user.fullName = "Jane Smith";
console.log(user.firstName); // "Jane"

// Example 2: Schema validation with all traps
function schemaValidator(schema) {
  return new Proxy({}, {
    set(target, prop, value) {
      if (prop in schema) {
        const rules = schema[prop];
        if (rules.type && typeof value !== rules.type) {
          throw new TypeError(`${String(prop)} must be ${rules.type}`);
        }
        if (rules.min != null && value < rules.min) {
          throw new RangeError(`${String(prop)} must be >= ${rules.min}`);
        }
        if (rules.max != null && value > rules.max) {
          throw new RangeError(`${String(prop)} must be <= ${rules.max}`);
        }
        if (rules.required && (value == null || value === "")) {
          throw new Error(`${String(prop)} is required`);
        }
      }
      target[prop] = value;
      return true;
    },
    has(target, prop) {
      return prop in schema || prop in target;
    },
    deleteProperty(target, prop) {
      if (schema[prop]?.required) {
        throw new Error(`Cannot delete required property ${String(prop)}`);
      }
      return delete target[prop];
    },
  });
}

const user = schemaValidator({
  name: { type: "string", required: true },
  age: { type: "number", min: 0, max: 150 },
});
user.name = "Alice"; // OK
user.age = 25; // OK

// Example 3: Observable proxy with all traps
function observable(obj, onChange) {
  return new Proxy(obj, {
    get(target, prop, receiver) {
      const value = Reflect.get(target, prop, receiver);
      onChange("get", prop, value);
      return value;
    },
    set(target, prop, value, receiver) {
      const old = target[prop];
      const result = Reflect.set(target, prop, value, receiver);
      onChange("set", prop, value, old);
      return result;
    },
    deleteProperty(target, prop) {
      const old = target[prop];
      const result = Reflect.deleteProperty(target, prop);
      onChange("delete", prop, undefined, old);
      return result;
    },
    has(target, prop) {
      const result = Reflect.has(target, prop);
      onChange("has", prop, result);
      return result;
    },
  });
}
```

### Real-World Use Cases

- **Reactive state** — Vue 3 uses get/set traps for reactivity.
- **Form validation** — set trap validates form fields on assignment.
- **Access control** — get/has traps for role-based access.
- **API clients** — get trap for lazy resource loading.
- **Database models** — set trap for change tracking.

### Common Mistakes

- **Not returning boolean from set** — Must return `true` for success.
- **Not handling receiver in get** — Breaks prototype getter context.
- **Forgetting `Reflect.*` forwarding** — Loses default behavior.
- **Making traps too complex** — Each trap should do one thing.
- **Not throwing proper errors** — Validation traps should throw TypeError/RangeError.

### Best Practices

- Always return proper types from traps (boolean for set/deleteProperty, etc.).
- Use `Reflect.*` for default forwarding in traps.
- Keep trap logic minimal and focused.
- Document what each trap does and why.
- Test edge cases (non-existent properties, prototype chain).

### Performance Considerations

- Each trap invocation adds function call overhead.
- get/set traps are the most frequently called (wire them carefully).
- Complex traps (heavy computation) degrade performance.
- V8 can optimize simple get/set traps.
- Consider using a single trap for all properties vs per-property logic.

### Interview Questions

1. **What traps does a Proxy handler support?** 13 traps: get, set, has, deleteProperty, apply, construct, getPrototypeOf, setPrototypeOf, isExtensible, preventExtensions, getOwnPropertyDescriptor, defineProperty, ownKeys.

2. **What must the set trap return?** A boolean — `true` if the set succeeded, `false` otherwise.

3. **What arguments does the get trap receive?** `target`, `property`, `receiver` (the proxy or object that received the original call).

4. **How does the has trap differ from `Object.hasOwn`?** The has trap intercepts the `in` operator (which checks the prototype chain). It's analogous to `Reflect.has`.

### Coding Challenges

1. **Implement a `range` proxy where `proxy[5]` returns 5.**
2. **Create a `defaults` proxy that returns default values for undefined properties.**
3. **Build an `accessLog` proxy that records all property access.**
4. **Implement a cascade proxy (properties like `a.b.c` create intermediate objects).**
5. **Create a `lazyInit` proxy that initializes properties on first access.**

### Related Topics

- Reflect API
- Proxy constructor
- Revocable proxies
- Property descriptors
- Meta-programming

---

## Revocable proxies

### What It Is

`Proxy.revocable()` creates a proxy that can be revoked (disabled) later. It returns an object with `proxy` and `revoke` properties. After calling `revoke()`, any operation on the proxy throws a TypeError.

```javascript
const { proxy, revoke } = Proxy.revocable(
  { data: "sensitive" },
  {
    get(target, prop) {
      return target[prop];
    },
  }
);

console.log(proxy.data); // "sensitive"
revoke();
// console.log(proxy.data); // TypeError: Cannot perform 'get' on a proxy that has been revoked
```

### Why It Is Important

Revocable proxies provide a security and resource management mechanism:
- Grant temporary access to an object, then revoke it.
- Prevent further access after a timeout or event.
- Implement "one-time use" patterns.
- Clean up proxy references when no longer needed.
- Control API access lifecycle.

### How It Works Internally

The engine maintains a `[[Revoked]]` internal slot on the proxy. When `revoke()` is called, it sets this flag to `true`. Any subsequent trap invocation checks this flag and throws a TypeError before executing the trap. The revoke function can only be called once — subsequent calls are no-ops.

### Syntax

```javascript
const { proxy, revoke } = Proxy.revocable(target, handler);

// When done:
revoke();

// After revocation:
// proxy.anyOperation throws TypeError
// revoke() again is a no-op
```

### Beginner Examples

```javascript
// Example 1: Time-limited access
function createTimedAccess(obj, timeoutMs) {
  const { proxy, revoke } = Proxy.revocable(obj, {});
  setTimeout(revoke, timeoutMs);
  return proxy;
}

const temp = createTimedAccess({ secret: "hidden" }, 2000);
console.log(temp.secret); // "hidden" (within 2 seconds)
// After 2 seconds: temp.secret throws

// Example 2: One-time use proxy
function oneTime(obj) {
  const { proxy, revoke } = Proxy.revocable(obj, {
    get(target, prop) {
      const value = target[prop];
      revoke();
      return value;
    },
  });
  return proxy;
}

const single = oneTime({ msg: "You can only read me once" });
console.log(single.msg); // "You can only read me once"
// console.log(single.msg); // TypeError

// Example 3: Revocable for resource cleanup
function createResource(resource) {
  const handlers = [];
  const { proxy, revoke } = Proxy.revocable(resource, {
    get(target, prop) {
      handlers.push(prop);
      return target[prop];
    },
  });

  return {
    resource: proxy,
    release() {
      console.log(`Access log:`, handlers);
      revoke();
    },
  };
}
```

### Intermediate Examples

```javascript
// Example 1: Session-based access
class SecureSession {
  constructor(data) {
    const { proxy, revoke } = Proxy.revocable(data, {
      get(target, prop) {
        if (this.expired) throw new Error("Session expired");
        return target[prop];
      },
    });
    this.proxy = proxy;
    this.revoke = revoke;
    this.expired = false;
  }

  expire() {
    this.expired = true;
    this.revoke();
  }
}

const session = new SecureSession({ userId: 123, role: "admin" });
console.log(session.proxy.userId); // 123
session.expire();
// console.log(session.proxy.userId); // Error

// Example 2: Limited API access
function createLimitedAPI(api, allowedMethods) {
  const { proxy, revoke } = Proxy.revocable(api, {
    get(target, prop) {
      if (!allowedMethods.includes(prop)) {
        throw new Error(`Method ${String(prop)} not allowed`);
      }
      return target[prop];
    },
  });
  return { api: proxy, revoke };
}

// Example 3: Usage quota proxy
function quotaProxy(obj, maxAccess) {
  let count = 0;
  const { proxy, revoke } = Proxy.revocable(obj, {
    get(target, prop) {
      if (count >= maxAccess) {
        revoke();
        throw new Error("Quota exceeded");
      }
      count++;
      return target[prop];
    },
    set(target, prop, value) {
      if (count >= maxAccess) {
        revoke();
        throw new Error("Quota exceeded");
      }
      count++;
      target[prop] = value;
      return true;
    },
  });
  return { proxy, revoke, getCount: () => count };
}
```

### Advanced Examples

```javascript
// Example 1: Composable revocable proxies
function revocableChain(target, ...handlers) {
  let current = target;
  const revokers = [];

  for (const handler of handlers) {
    const { proxy, revoke } = Proxy.revocable(current, handler);
    revokers.push(revoke);
    current = proxy;
  }

  return {
    proxy: current,
    revokeAll() {
      revokers.forEach(r => r());
    },
  };
}

// Example 2: Self-revoking after operation
function selfRevoking(fn) {
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      const { proxy, revoke } = Proxy.revocable(target, {});
      const result = Reflect.apply(target, thisArg, args);
      revoke();
      return result;
    },
  });
}

// Example 3: Revocable with timeout and fallback
function withFallback(obj, fallback, timeoutMs) {
  const { proxy, revoke } = Proxy.revocable(obj, {
    get(target, prop) {
      if (prop in target) return target[prop];
      return fallback[prop];
    },
    set(target, prop, value) {
      target[prop] = value;
      return true;
    },
  });
  setTimeout(revoke, timeoutMs);
  return proxy;
}
```

### Real-World Use Cases

- **Token-based API access** — Revoke proxy when token expires.
- **Plugin sandboxes** — Grant temporary access to host APIs.
- **WebWorker communication** — Transfer proxy ownership with revoke.
- **One-time tokens** — Secure single-use credentials.
- **Test fixtures** — Mock objects that self-destruct after tests.

### Common Mistakes

- **Calling revoke multiple times** — Second call is a no-op, safe.
- **Trying to access proxy after revocation** — Must handle the TypeError.
- **Forgetting to revoke** — Resources leak if revoke is never called.
- **Storing reference to target** — Bypasses the proxy entirely.
- **Not handling revoked state gracefully** — Operations throw unexpectedly.

### Best Practices

- Always call `revoke()` when the proxy is no longer needed.
- Use try/catch around revoked proxy access if it may be accessed later.
- Store `revoke` in a place where it's guaranteed to be called (finally, cleanup).
- Use revocable proxies for temporary grants of access.
- Document that a proxy may be revoked and how to handle the error.

### Performance Considerations

- Revocable proxies have slight overhead (tracking revoked state).
- The `revoke` function is lightweight.
- Checking revoked state on each operation adds minimal cost.
- Unrevoked proxies that go out of scope are garbage collected normally.
- Revoking is one-way — no way to re-enable.

### Interview Questions

1. **What does `Proxy.revocable()` return?**
   An object with `{ proxy, revoke }` properties.

2. **What happens after calling `revoke()`?**
   Any operation on the proxy throws a TypeError.

3. **When would you use a revocable proxy?**
   Temporary access grants, session management, security contexts, resource cleanup.

4. **Can a revoked proxy be re-enabled?**
   No. Revocation is permanent and one-way.

### Coding Challenges

1. **Create a proxy that auto-revokes after N property accesses.**
2. **Implement a session manager using revocable proxies.**
3. **Build a function that returns a one-time-use revocable API wrapper.**
4. **Create a revocable proxy chain with cascading revoke.**

### Related Topics

- Proxy constructor
- Handler traps
- Object lifecycle
- Security patterns
- Resource management

---

## Practical Proxy use cases

### What It Is

Proxies enable practical patterns that solve real-world problems elegantly. From reactive state management to lazy loading, validation to logging, proxies provide a layer of indirection that can augment, restrict, or virtualize object behavior without modifying the original object.

### Why It Is Important

Practical proxy use cases demonstrate the power of meta-programming:
- **Reactivity** — Automatic UI updates on state changes.
- **Validation** — Runtime type checking and constraints.
- **Lazy loading** — Fetch data only when accessed.
- **Logging/Debugging** — Comprehensive access tracing.
- **Caching/Memoization** — Automatic result caching.
- **Access control** — Role-based property visibility.

### How It Works Internally

Each pattern leverages specific traps:
- Reactive: get (track), set (trigger updates).
- Validate: set (check values).
- Lazy: get (initialize on first access).
- Cache: get (return cached or compute).
- Access control: get/has (check permissions).
- Logging: all traps (record operations).

### Beginner Examples

```javascript
// Use Case 1: Default values
function withDefaults(defaults) {
  return new Proxy({}, {
    get(target, prop) {
      if (prop in target) return target[prop];
      if (prop in defaults) return defaults[prop];
      return undefined;
    },
    set(target, prop, value) {
      target[prop] = value;
      return true;
    },
  });
}

const settings = withDefaults({ theme: "dark", lang: "en" });
console.log(settings.theme); // "dark"
console.log(settings.lang); // "en"
console.log(settings.timeout); // undefined (no default)

// Use Case 2: Enum validation
function enumProxy(values) {
  return new Proxy({}, {
    set(target, prop, value) {
      if (!values.includes(value)) {
        throw new Error(`Value must be one of: ${values.join(", ")}`);
      }
      target[prop] = value;
      return true;
    },
  });
}

const status = enumProxy(["pending", "active", "completed"]);
status.state = "active"; // OK
// status.state = "invalid"; // Error

// Use Case 3: Property deprecation warnings
function deprecated(obj, deprecatedMap) {
  return new Proxy(obj, {
    get(target, prop) {
      if (prop in deprecatedMap) {
        console.warn(`Warning: ${String(prop)} is deprecated. Use ${deprecatedMap[prop]} instead.`);
        return target[deprecatedMap[prop]];
      }
      return target[prop];
    },
  });
}
```

### Intermediate Examples

```javascript
// Use Case 1: Lazy initialization
function lazyInit(factory) {
  const instance = { _initialized: false, _data: null };
  return new Proxy(instance, {
    get(target, prop) {
      if (!target._initialized) {
        console.log("Initializing...");
        Object.assign(target, factory());
        target._initialized = true;
      }
      return target[prop];
    },
  });
}

const config = lazyInit(() => ({
  host: "localhost",
  port: 3000,
  _initialized: true,
}));
console.log(config.host); // Logs "Initializing..." then "localhost"
console.log(config.host); // Just "localhost"

// Use Case 2: Auto-binding methods
function autoBind(obj) {
  return new Proxy(obj, {
    get(target, prop) {
      const value = target[prop];
      if (typeof value === "function") {
        return value.bind(target);
      }
      return value;
    },
  });
}

const actions = autoBind({
  name: "action",
  greet() { console.log(`Hello from ${this.name}`); },
});
const { greet } = actions;
greet(); // "Hello from action" (this preserved without explicit bind)

// Use Case 3: Indexed collections
function createCollection() {
  const items = [];
  return new Proxy(items, {
    get(target, prop) {
      if (prop === "first") return target[0];
      if (prop === "last") return target[target.length - 1];
      if (prop === "isEmpty") return target.length === 0;
      return target[prop];
    },
    set(target, prop, value) {
      if (prop === "first") {
        target[0] = value;
        return true;
      }
      if (prop === "last") {
        target[target.length - 1] = value;
        return true;
      }
      target[prop] = value;
      return true;
    },
  });
}

const col = createCollection();
col.push(1, 2, 3, 4, 5);
console.log(col.first); // 1
console.log(col.last); // 5
console.log(col.isEmpty); // false
```

### Advanced Examples

```javascript
// Use Case 1: Full reactive state management
function createStore(initialState) {
  const state = { ...initialState };
  const listeners = new Map();

  return {
    state: new Proxy(state, {
      set(target, prop, value) {
        const oldValue = target[prop];
        target[prop] = value;
        if (listeners.has(prop)) {
          listeners.get(prop).forEach(fn => fn(value, oldValue));
        }
        return true;
      },
    }),
    subscribe(prop, fn) {
      if (!listeners.has(prop)) listeners.set(prop, new Set());
      listeners.get(prop).add(fn);
      return () => listeners.get(prop).delete(fn);
    },
  };
}

// Use Case 2: API client with lazy endpoints
function createClient(baseUrl) {
  return new Proxy({}, {
    get(target, endpoint) {
      return async (params = {}) => {
        const query = Object.entries(params)
          .map(([k, v]) => `${k}=${encodeURIComponent(v)}`)
          .join("&");
        const url = `${baseUrl}/${String(endpoint)}${query ? `?${query}` : ""}`;
        const res = await fetch(url);
        return res.json();
      };
    },
  });
}

const api = createClient("https://api.example.com");
// No method definition needed — just access:
// const users = await api.users({ page: 1 });
// const posts = await api.posts({ limit: 10 });

// Use Case 3: Secure sandbox for untrusted code
function createSandbox(context) {
  const safeGlobals = new Set(["Object", "Array", "String", "Number", "Boolean",
    "Map", "Set", "Promise", "Math", "JSON", "parseInt", "parseFloat", "console"]);
  return new Proxy(context || {}, {
    has(target, prop) {
      if (prop === "eval" || prop === "Function") {
        throw new Error("Access denied");
      }
      if (!safeGlobals.has(prop) && !(prop in target)) {
        return false;
      }
      return prop in target || safeGlobals.has(prop);
    },
    get(target, prop) {
      if (prop === "eval" || prop === "Function" || prop === "Proxy") {
        throw new Error("Access denied");
      }
      if (prop in target) return target[prop];
      if (safeGlobals.has(prop)) return globalThis[prop];
      return undefined;
    },
    set(target, prop, value) {
      target[prop] = value;
      return true;
    },
  });
}

// Use Case 4: Schema-based data validation
function createModel(schema) {
  return new Proxy({}, {
    set(target, prop, value) {
      const rules = schema[prop];
      if (!rules) {
        target[prop] = value;
        return true;
      }
      if (rules.type && typeof value !== rules.type) {
        throw new TypeError(`${String(prop)} must be of type ${rules.type}`);
      }
      if (rules.enum && !rules.enum.includes(value)) {
        throw new Error(`${String(prop)} must be one of ${rules.enum.join(", ")}`);
      }
      if (rules.pattern && !rules.pattern.test(value)) {
        throw new Error(`${String(prop)} does not match pattern ${rules.pattern}`);
      }
      if (rules.minLength && String(value).length < rules.minLength) {
        throw new Error(`${String(prop)} must be at least ${rules.minLength} chars`);
      }
      target[prop] = value;
      return true;
    },
  });
}

const userSchema = createModel({
  email: { type: "string", pattern: /^.+@.+\..+$/ },
  role: { type: "string", enum: ["admin", "user", "guest"] },
  age: { type: "number", min: 0, max: 150 },
});
```

### Real-World Use Cases

- **Vue 3** — Reactive data system using Proxy.
- **MobX** — Observable observables via Proxy.
- **Immer** — Draft proxies for immutable state updates.
- **Onnx Runtime** — Tensor access via Proxy.
- **JSDOM** — Simulated browser APIs via Proxy.

### Common Mistakes

- **Over-engineering** — Using Proxy when a simple function would do.
- **Performance blind spots** — Not measuring proxy overhead in hot paths.
- **Leaking the target** — Keeping references to the original object.
- **Ignoring receiver** — Breaking prototype chain behavior.
- **Not handling all traps** — Incomplete proxy can leak through.

### Best Practices

- Start with the simplest proxy that solves your problem.
- Use TypeScript for proxy-based APIs (better type safety).
- Measure performance before optimizing proxy code.
- Always forward with Reflect in production Proxies.
- Document proxy behavior clearly, especially side effects.

### Performance Considerations

- Each proxy operation has overhead — use sparingly in hot loops.
- Reactive systems batch updates to minimize proxy calls.
- V8 improves proxy performance with each version.
- Nested proxies (proxy wrapping proxy) add cumulative overhead.
- Consider using classes or simple functions for non-interception requirements.

### Interview Questions

1. **How would you implement a reactive state system using Proxy?**
   Use get trap to track dependencies, set trap to trigger subscribers.

2. **How can you implement lazy loading with Proxy?**
   In the get trap, check if the property exists; if not, load/fetch it on first access.

3. **Can Proxy help with API versioning?**
   Yes — a proxy can map old property names to new ones using get/set traps.

4. **How do you build a sandbox for untrusted code?**
   Use a proxy with restrictive get/set/has traps that whitelist allowed globals.

### Coding Challenges

1. **Create a caching proxy that stores function results by arguments.**
2. **Build a retry proxy that retries failed async operations.**
3. **Implement a `throttle` proxy that limits how often a function can be called.**
4. **Create a `debounce` proxy that delays function execution.**
5. **Build a change-detection proxy for forms that tracks dirty fields.**

### Related Topics

- Reflect API
- Reactive programming
- Validation patterns
- Lazy loading
- Caching strategies
- Access control
