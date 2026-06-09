# APIs - REST API typing, request validation, response types, OpenAPI generation

## Introduction

Building REST APIs with TypeScript provides type safety across the entire request-response lifecycle, from incoming request validation to outgoing response serialization. This file covers REST endpoint typing with Express, request validation using Zod, typed response objects, and generating OpenAPI specifications from TypeScript types.

## REST Endpoint Types

### What It Is

REST endpoint typing involves defining TypeScript types for request parameters, query strings, request bodies, response bodies, and headers for each API endpoint. These types serve as a contract between the server implementation and API consumers.

### Why It Is Important

Typed endpoints provide compile-time safety for request handling, ensure consistent response shapes, and serve as living documentation. They enable IDE autocompletion for API consumers and catch breaking changes during refactoring.

### How It Works Internally

Endpoint types are typically defined as interfaces or types, then used as generic parameters to Express's Request and Response types. These types are erased at runtime but checked during compilation.

### Syntax

`	ypescript
// Endpoint type definitions
interface GetUserParams {
  id: string;
}

interface GetUserResponse {
  id: string;
  name: string;
  email: string;
  createdAt: string;
}

// Usage in route handler
app.get('/api/users/:id', (
  req: Request<GetUserParams, GetUserResponse>,
  res: Response<GetUserResponse>
) => {
  const user = await findUserById(req.params.id);
  if (!user) {
    res.status(404).json({ error: 'User not found' } as any);
    return;
  }
  res.json(user);
});
`

### Beginner Examples

`	ypescript
interface ListItemsQuery {
  page?: string;
  limit?: string;
  search?: string;
}

interface ListItemsResponse {
  items: Item[];
  total: number;
  page: number;
  limit: number;
}

app.get('/api/items', async (
  req: Request<{}, ListItemsResponse, {}, ListItemsQuery>,
  res: Response<ListItemsResponse>
) => {
  const page = Number(req.query.page || '1');
  const limit = Number(req.query.limit || '20');
  const { items, total } = await findItems({ page, limit, search: req.query.search });
  res.json({ items, total, page, limit });
});
`

### Intermediate Examples

`	ypescript
// CRUD endpoint types
interface CreateUserBody {
  name: string;
  email: string;
  password: string;
}

interface UpdateUserBody {
  name?: string;
  email?: string;
}

interface UserResponse {
  id: string;
  name: string;
  email: string;
  createdAt: string;
  updatedAt: string;
}

// Create
app.post('/api/users', async (
  req: Request<{}, UserResponse, CreateUserBody>,
  res: Response<UserResponse>
) => {
  const user = await createUser(req.body);
  res.status(201).json(user);
});

// Update (partial)
app.patch('/api/users/:id', async (
  req: Request<{ id: string }, UserResponse, UpdateUserBody>,
  res: Response<UserResponse>
) => {
  const user = await updateUser(req.params.id, req.body);
  if (!user) {
    res.status(404).json({ error: 'User not found' } as any);
    return;
  }
  res.json(user);
});

// Delete
app.delete('/api/users/:id', async (
  req: Request<{ id: string }, { message: string }>,
  res: Response<{ message: string }>
) => {
  await deleteUser(req.params.id);
  res.json({ message: 'User deleted' });
});
`

### Advanced Examples

`	ypescript
// Generic API response wrapper
interface ApiResponse<T> {
  data: T;
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
    hasMore?: boolean;
  };
}

interface ApiError {
  error: string;
  details?: unknown;
  code?: string;
}

type ApiResult<T> = ApiResponse<T> | ApiError;

function sendSuccess<T>(res: Response, data: T, meta?: ApiResponse<T>['meta']) {
  const response: ApiResponse<T> = { data };
  if (meta) response.meta = meta;
  res.json(response);
}

function sendError(res: Response, status: number, error: string, code?: string, details?: unknown) {
  res.status(status).json({ error, code, details } as ApiError);
}

// Usage
app.get('/api/products', async (req, res) => {
  try {
    const { products, total } = await getProducts();
    sendSuccess(res, products, { total, page: 1, limit: products.length });
  } catch (err) {
    sendError(res, 500, 'Internal server error', 'INTERNAL_ERROR');
  }
});
`

### Real-World Use Cases

`	ypescript
// Versioned API types
namespace API {
  export namespace V1 {
    export interface User {
      id: string;
      name: string;
      email: string;
    }

    export interface CreateUserBody {
      name: string;
      email: string;
      password: string;
    }

    export type CreateUserResponse = User;
  }

  export namespace V2 {
    export interface User {
      id: string;
      name: string;
      email: string;
      profile: {
        avatar: string;
        bio: string;
      };
    }

    export interface CreateUserBody {
      name: string;
      email: string;
      password: string;
      profile: {
        avatar?: string;
        bio?: string;
      };
    }

    export type CreateUserResponse = User;
  }
}

// V1 routes
app.post('/api/v1/users', async (
  req: Request<{}, API.V1.CreateUserResponse, API.V1.CreateUserBody>,
  res: Response<API.V1.CreateUserResponse>
) => { /* v1 implementation */ });

// V2 routes
app.post('/api/v2/users', async (
  req: Request<{}, API.V2.CreateUserResponse, API.V2.CreateUserBody>,
  res: Response<API.V2.CreateUserResponse>
) => { /* v2 implementation */ });
`

### Common Mistakes

`	ypescript
// MISTAKE: Not typing the response body
// res.json({ id: '123' }); // Response body is implicitly any

// MISTAKE: Using the same type for request body and response
// Request body and response body often have different shapes

// MISTAKE: Not typing error responses
// Error responses should be typed separately from success responses
`

### Best Practices

- Define types for request body, response body, params, and query for each endpoint.
- Use a generic ApiResponse<T> wrapper for consistent API responses.
- Keep endpoint types co-located with their route handlers or in a dedicated 	ypes/api folder.
- Use Pick, Omit, and Partial to derive types and avoid duplication.
- Version your API types explicitly using namespaces or separate modules.

### Performance Considerations

Endpoint types are erased at compile time. However, runtime validation (see next sections) is necessary to ensure that incoming data matches the expected types, providing defense in depth.

### Interview Questions

1. How would you type a REST API endpoint that accepts query parameters and returns a paginated response?
2. What is the advantage of using a generic ApiResponse<T> type?
3. How do you handle different API versions with TypeScript types?

### Coding Challenges

1. Define types for a blog API with endpoints for creating, reading, updating, and deleting posts and comments.
2. Create a typed API client that consumes the blog API with full type safety.

### Related Topics

- Request validation with Zod
- Response typing
- OpenAPI generation

## Request Validation with Zod

### What It Is

Request validation ensures that incoming data (body, params, query) matches expected schemas before it reaches business logic. Zod is a TypeScript-first schema validation library that provides runtime validation with automatic type inference.

### Why It Is Important

TypeScript types are erased at runtime, so they cannot validate external input. Zod bridges this gap by providing runtime validation that produces typed results. Combined with TypeScript, Zod ensures that validated data is correctly typed throughout the application.

### How It Works Internally

Zod schemas produce a type that can be extracted with z.infer<typeof schema>. The .parse() method validates data against the schema and returns the typed data or throws a ZodError. The .safeParse() method returns a result object without throwing.

### Syntax

`	ypescript
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

type UserInput = z.infer<typeof UserSchema>;
// { name: string; email: string; age?: number }
`

### Beginner Examples

`	ypescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

app.post('/api/users', (req, res) => {
  try {
    const data = CreateUserSchema.parse(req.body);
    // data is typed as { name: string; email: string; password: string }
    const user = await createUser(data);
    res.status(201).json(user);
  } catch (err) {
    if (err instanceof z.ZodError) {
      res.status(400).json({ error: 'Validation failed', details: err.errors });
      return;
    }
    res.status(500).json({ error: 'Internal server error' });
  }
});
`

### Intermediate Examples

`	ypescript
import { z } from 'zod';

// Param validation
const GetUserParamsSchema = z.object({
  id: z.string().uuid('Invalid user ID'),
});

// Query validation
const ListUsersQuerySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.enum(['name', 'email', 'createdAt']).optional(),
  order: z.enum(['asc', 'desc']).default('asc'),
});

// Body validation for update
const UpdateUserBodySchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional(),
  password: z.string().min(8).optional(),
}).refine(data => Object.keys(data).length > 0, {
  message: 'At least one field must be provided',
});

app.get('/api/users/:id', async (req, res) => {
  const params = GetUserParamsSchema.parse(req.params);
  const query = ListUsersQuerySchema.parse(req.query);
  const user = await getUserById(params.id);
  res.json(user);
});

app.patch('/api/users/:id', async (req, res) => {
  const params = GetUserParamsSchema.parse(req.params);
  const body = UpdateUserBodySchema.parse(req.body);
  const user = await updateUser(params.id, body);
  res.json(user);
});
`

### Advanced Examples

`	ypescript
// Validation middleware factory
import { Request, Response, NextFunction } from 'express';
import { z, ZodSchema } from 'zod';

interface ValidationSchemas {
  body?: ZodSchema;
  query?: ZodSchema;
  params?: ZodSchema;
}

function validate(schemas: ValidationSchemas) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      if (schemas.body) req.body = schemas.body.parse(req.body);
      if (schemas.query) req.query = schemas.query.parse(req.query) as any;
      if (schemas.params) req.params = schemas.params.parse(req.params) as any;
      next();
    } catch (err) {
      if (err instanceof z.ZodError) {
        res.status(400).json({
          error: 'Validation failed',
          details: err.errors.map(e => ({
            path: e.path.join('.'),
            message: e.message,
          })),
        });
        return;
      }
      next(err);
    }
  };
}

// Usage with full validation
const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1),
  tags: z.array(z.string()).max(10).default([]),
  published: z.boolean().default(false),
});

const PostParamsSchema = z.object({
  id: z.string().uuid(),
});

app.post(
  '/api/posts',
  validate({ body: CreatePostSchema }),
  async (req, res) => {
    // req.body is fully typed from schema inference
    const post = await createPost(req.body);
    res.status(201).json(post);
  }
);

app.get(
  '/api/posts/:id',
  validate({ params: PostParamsSchema }),
  async (req, res) => {
    const post = await getPostById(req.params.id);
    if (!post) {
      res.status(404).json({ error: 'Post not found' });
      return;
    }
    res.json(post);
  }
);
`

### Real-World Use Cases

`	ypescript
// Complex nested validation
const CreateOrderSchema = z.object({
  customerId: z.string().uuid(),
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
    price: z.number().positive(),
  })).min(1, 'Order must have at least one item'),
  shippingAddress: z.object({
    street: z.string().min(1),
    city: z.string().min(1),
    state: z.string().length(2),
    zipCode: z.string().regex(/^\d{5}(-\d{4})?$/),
    country: z.string().length(2),
  }),
  couponCode: z.string().optional(),
  notes: z.string().max(500).optional(),
});

type CreateOrderInput = z.infer<typeof CreateOrderSchema>;
// Full type inference for the complex nested object
`

### Common Mistakes

`	ypescript
// MISTAKE: Relying solely on TypeScript types for validation
// Types are erased at runtime - Zod is needed for runtime safety

// MISTAKE: Not using coerce for query/param strings
// z.number() // Fails for '123' (string from query)
// z.coerce.number() // Converts '123' to 123

// MISTAKE: Throwing ZodErrors instead of using safeParse for expected failures
// For expected failures (validation), use safeParse
`

### Best Practices

- Use Zod's z.infer to extract TypeScript types from schemas (single source of truth).
- Use z.coerce for query and param parsing (they come as strings).
- Create a validation middleware factory for reusability.
- Use .safeParse() instead of .parse() when you want to handle errors without try-catch.
- Define schemas in a dedicated schemas/ directory, separate from route handlers.

### Performance Considerations

Zod validation adds runtime overhead proportional to the size and complexity of the data. For high-throughput endpoints, consider:
- Caching compiled schemas (Zod caches them by default).
- Validating only at the service boundary, not internally.
- Using safeParse() to avoid try-catch overhead.

### Interview Questions

1. Why is runtime validation necessary even with TypeScript?
2. What is the difference between z.infer<typeof schema> and the schema itself?
3. How does Zod's coerce work for query string parsing?

### Coding Challenges

1. Create a Zod schema for a user registration endpoint with password confirmation validation.
2. Build a validation middleware that validates body, params, and query simultaneously.

### Related Topics

- Zod schemas
- Type inference from schemas
- Runtime vs compile-time validation

## Response Typing

### What It Is

Response typing involves defining TypeScript types for API responses, ensuring that every endpoint returns consistently shaped data. This includes success responses, error responses, and paginated responses.

### Why It Is Important

Typed responses provide a contract for API consumers, enable automatic documentation generation, and prevent accidental exposure of internal data (like passwords). They also make refactoring safer by ensuring response shapes are updated consistently.

### How It Works Internally

Response types are used as the second generic parameter to Express's Response type. While the type system doesn't enforce that es.json() actually returns data matching the type, it ensures that the code composes correctly.

### Syntax

`	ypescript
interface SuccessResponse<T> {
  success: true;
  data: T;
}

interface ErrorResponse {
  success: false;
  error: string;
  code?: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

// Usage
function success<T>(res: Response, data: T, status = 200) {
  const response: SuccessResponse<T> = { success: true, data };
  res.status(status).json(response);
}

function error(res: Response, message: string, status = 400, code?: string) {
  const response: ErrorResponse = { success: false, error: message, code };
  res.status(status).json(response);
}
`

### Beginner Examples

`	ypescript
interface UserResponse {
  id: string;
  name: string;
  email: string;
}

interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

app.get('/api/users', async (req, res: Response<PaginatedResponse<UserResponse>>) => {
  const users = await getUsers();
  res.json({
    data: users,
    meta: { total: users.length, page: 1, limit: 20, totalPages: 1 },
  });
});
`

### Intermediate Examples

`	ypescript
// Resource-specific responses
interface PostResponse {
  id: string;
  title: string;
  content: string;
  author: {
    id: string;
    name: string;
  };
  tags: string[];
  createdAt: string;
  updatedAt: string;
}

interface PostListResponse {
  posts: PostResponse[];
  total: number;
  page: number;
  limit: number;
}

// Mapped types for consistent responses
type SanitizedUser = Omit<User, 'password' | 'refreshToken'>;

app.get('/api/users/me', async (req, res: Response<SanitizedUser>) => {
  const user = await findUserById(req.currentUser!.id);
  const { password, refreshToken, ...safeUser } = user;
  res.json(safeUser);
});
`

### Advanced Examples

`	ypescript
// Response builder pattern
class ResponseBuilder<T> {
  private statusCode = 200;
  private headers: Record<string, string> = {};
  private meta?: Record<string, unknown>;

  constructor(private data: T) {}

  status(code: number): this {
    this.statusCode = code;
    return this;
  }

  header(name: string, value: string): this {
    this.headers[name] = value;
    return this;
  }

  withMeta(meta: Record<string, unknown>): this {
    this.meta = meta;
    return this;
  }

  send(res: Response): void {
    const body: Record<string, unknown> = { data: this.data };
    if (this.meta) body.meta = this.meta;

    res.status(this.statusCode);
    Object.entries(this.headers).forEach(([key, value]) => res.setHeader(key, value));
    res.json(body);
  }
}

// Usage
app.get('/api/products', async (req, res) => {
  const products = await getProducts();
  new ResponseBuilder(products)
    .status(200)
    .withMeta({ total: products.length })
    .header('X-Cache', 'HIT')
    .send(res);
});
`

### Real-World Use Cases

`	ypescript
// JSON:API compliant responses
interface JsonApiResource<T, R = Record<string, unknown>> {
  type: string;
  id: string;
  attributes: T;
  relationships?: R;
}

interface JsonApiResponse<T, R = Record<string, unknown>> {
  data: JsonApiResource<T, R> | JsonApiResource<T, R>[];
  included?: unknown[];
  meta?: {
    count?: number;
    [key: string]: unknown;
  };
  links?: {
    self: string;
    first?: string;
    prev?: string;
    next?: string;
    last?: string;
  };
}

// User resource response
type UserResource = JsonApiResource<{
  name: string;
  email: string;
}, {
  posts: { data: { type: 'posts'; id: string }[] };
}>;

app.get('/api/users/:id', async (req, res: Response<JsonApiResponse<UserResource>>) => {
  const user = await getUserWithPosts(req.params.id);
  res.json({
    data: {
      type: 'users',
      id: user.id,
      attributes: { name: user.name, email: user.email },
      relationships: {
        posts: { data: user.posts.map(p => ({ type: 'posts', id: p.id })) },
      },
    },
  });
});
`

### Common Mistakes

`	ypescript
// MISTAKE: Returning internal database objects directly
// res.json(user) // Might expose password, __v, etc.

// MISTAKE: Inconsistent error response shapes
// Sometimes { error: 'msg' }, sometimes { message: 'msg' }, sometimes { errors: [] }

// MISTAKE: Not typing the response body generic
// res: Response // Response body is any
`

### Best Practices

- Use a consistent response envelope ({ success, data, meta } or JSON:API).
- Never return database objects directly; always transform them.
- Use Omit, Pick, or Mapped Types to derive response types from entity types.
- Type the response generic parameter for compile-time checking.
- Use response builder classes or helper functions for consistency.

### Performance Considerations

Response transformation adds minimal overhead. However, serializing large response bodies can be expensive. Consider pagination, field selection (?fields=name,email), and compression.

### Interview Questions

1. How do you type a paginated API response in TypeScript?
2. What is the advantage of using a consistent response envelope?
3. How would you derive a response type from an entity type while excluding sensitive fields?

### Coding Challenges

1. Create a generic PaginatedResponse<T> type and a helper function to build paginated responses.
2. Build a typed response system that automatically strips sensitive fields based on user roles.

### Related Topics

- REST API typing
- OpenAPI generation
- Response serialization

## OpenAPI from TypeScript

### What It Is

OpenAPI (Swagger) is a specification for describing REST APIs. Generating OpenAPI specifications from TypeScript types ensures that documentation stays in sync with implementation. Tools like zod-to-openapi, 	s-to-openapi, and 	soa automate this process.

### Why It Is Important

Manually maintaining OpenAPI specs is error-prone and leads to drift between implementation and documentation. Auto-generating from TypeScript types ensures that the API documentation is always accurate and provides type-safe contract testing for API consumers.

### How It Works Internally

Tools like zod-to-openapi traverse Zod schemas and generate OpenAPI schema objects. Tools like 	soa use TypeScript decorators and the compiler API to generate OpenAPI specs from runtime metadata. These tools produce JSON/YAML files that can be served by Swagger UI.

### Syntax

`	ypescript
// Using zod-to-openapi
import { OpenAPIRegistry, OpenApiGeneratorV3 } from '@asteasolutions/zod-to-openapi';
import { z } from 'zod';

const registry = new OpenAPIRegistry();

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  email: z.string().email(),
});

registry.registerPath({
  method: 'get',
  path: '/api/users/{id}',
  summary: 'Get user by ID',
  request: {
    params: z.object({ id: z.string().uuid() }),
  },
  responses: {
    200: {
      description: 'User found',
      content: { 'application/json': { schema: UserSchema } },
    },
    404: {
      description: 'User not found',
      content: { 'application/json': { schema: z.object({ error: z.string() }) } },
    },
  },
});

const generator = new OpenApiGeneratorV3(registry.definitions);
const spec = generator.generateDocument({
  openapi: '3.0.3',
  info: { title: 'My API', version: '1.0.0' },
});
`

### Beginner Examples

`	ypescript
// Using tsoa (decorator-based)
import { Get, Route, Path, Response } from 'tsoa';

@Route('users')
export class UserController {
  @Get('{userId}')
  @Response(404, 'User not found')
  async getUser(@Path() userId: string): Promise<User> {
    return await findUserById(userId);
  }
}
`

### Intermediate Examples

`	ypescript
import { OpenAPIRegistry, OpenApiGeneratorV3 } from '@asteasolutions/zod-to-openapi';
import { z } from 'zod';

const registry = new OpenAPIRegistry();

// Reusable schemas
const ErrorResponse = z.object({
  error: z.string(),
  code: z.string().optional(),
});

const PaginationParams = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
});

const UserResponse = z.object({
  id: z.string().uuid(),
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().datetime(),
});

const CreateUserBody = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string().min(8),
});

// Register paths
registry.registerPath({
  method: 'get',
  path: '/api/users',
  summary: 'List users',
  request: { query: PaginationParams },
  responses: {
    200: {
      description: 'Paginated list of users',
      content: {
        'application/json': {
          schema: z.object({
            data: z.array(UserResponse),
            meta: z.object({ total: z.number(), page: z.number(), limit: z.number() }),
          }),
        },
      },
    },
  },
});

registry.registerPath({
  method: 'post',
  path: '/api/users',
  summary: 'Create user',
  request: { body: { content: { 'application/json': { schema: CreateUserBody } } } },
  responses: {
    201: { description: 'User created', content: { 'application/json': { schema: UserResponse } } },
    400: { description: 'Validation error', content: { 'application/json': { schema: ErrorResponse } } },
  },
});

const generator = new OpenApiGeneratorV3(registry.definitions);
export const openApiSpec = generator.generateDocument({
  openapi: '3.0.3',
  info: { title: 'User API', version: '1.0.0' },
  servers: [{ url: 'http://localhost:3000' }],
});
`

### Advanced Examples

`	ypescript
// Full API documentation with security and tags
registry.registerComponent('securitySchemes', 'bearerAuth', {
  type: 'http',
  scheme: 'bearer',
  bearerFormat: 'JWT',
});

registry.registerPath({
  method: 'get',
  path: '/api/admin/users',
  description: 'Admin-only user list',
  security: [{ bearerAuth: [] }],
  tags: ['Admin'],
  request: {
    query: z.object({
      role: z.enum(['user', 'admin', 'moderator']).optional(),
      status: z.enum(['active', 'suspended', 'deleted']).optional(),
    }),
  },
  responses: {
    200: {
      description: 'Admin user list',
      content: { 'application/json': { schema: z.array(UserResponse) } },
    },
    401: { description: 'Unauthorized' },
    403: { description: 'Forbidden' },
  },
});

// Serve the spec
app.get('/api-docs/openapi.json', (req, res) => {
  res.json(openApiSpec);
});

// Swagger UI
import swaggerUi from 'swagger-ui-express';
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(openApiSpec));
`

### Real-World Use Cases

`	ypescript
// Monorepo: Shared API types between frontend and backend
// packages/shared/src/api/users.ts
export const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  email: z.string().email(),
});

export type User = z.infer<typeof UserSchema>;

// Backend: Uses schema for validation + generates OpenAPI
// Frontend: Uses User type for API client

import { UserSchema } from '@shared/api/users';
import { createApiClient } from '@shared/api-client';

const api = createApiClient<User>('/api/users');
const users = await api.list(); // Fully typed
`

### Common Mistakes

`	ypescript
// MISTAKE: Manual OpenAPI specs that drift from implementation
// Always auto-generate from types/schemas

// MISTAKE: Not including error responses in the spec
// Error responses are part of the API contract

// MISTAKE: Overly complex generated specs
// Keep schemas focused and reuse components
`

### Best Practices

- Generate OpenAPI specs from the same Zod schemas used for validation.
- Use egistry.registerComponent for reusable schema components.
- Include security schemes, tags, and descriptions for completeness.
- Serve the generated spec via a route and use Swagger UI for exploration.
- Use shared type packages in monorepos for frontend/backend type consistency.

### Performance Considerations

OpenAPI generation typically happens at build time or startup time, not on each request. The generation overhead is incurred once, making it free at runtime. Serving the spec via an endpoint adds negligible overhead.

### Interview Questions

1. What tools can generate OpenAPI specs from TypeScript types?
2. How does zod-to-openapi generate OpenAPI schemas from Zod schemas?
3. Why is auto-generating OpenAPI specs better than maintaining them manually?

### Coding Challenges

1. Create a Zod schema for a blog post API and generate an OpenAPI specification from it.
2. Build a middleware that automatically registers routes and generates OpenAPI documentation.

### Related Topics

- Zod validation
- API typing
- Contract testing
