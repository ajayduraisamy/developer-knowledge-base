# Express with TS - Typing Request/Response, custom request types, middleware typing, error handling typing

## Introduction

Express.js is a popular web framework for Node.js, and TypeScript enhances it with type safety for request handlers, middleware, and error handling. Proper typing ensures that request and response objects have correctly typed bodies, query parameters, headers, and custom properties. This file covers typing Request and Response, extending requests with custom properties (like currentUser), middleware type signatures, and error handling middleware patterns.

## Request and Response Typing

### What It Is

Express's Request and Response objects have generic type parameters that allow specifying the request body, query parameters, URL parameters, and response body types. Using these generics ensures that handlers access correctly typed data.

### Why It Is Important

Without typing, eq.body, eq.query, and eq.params default to ny, leading to potential runtime errors. Typing these objects provides compile-time safety and documents the expected shapes of incoming and outgoing data.

### How It Works Internally

Express types define Request as Request<Params, ResBody, ReqBody, ReqQuery, Locals>. Each parameter has a default (ParamsDictionary, ny, etc.). By providing concrete types, TypeScript checks that you access only valid properties.

`	ypescript
interface Request<
  P = ParamsDictionary,
  ResBody = any,
  ReqBody = any,
  ReqQuery = ParsedQs,
  Locals extends Record<string, any> = Record<string, any>
> extends http.IncomingMessage {}
`

### Syntax

`	ypescript
import { Request, Response } from 'express';

interface CreateUserBody {
  name: string;
  email: string;
  age: number;
}

interface UserResponse {
  id: string;
  name: string;
  email: string;
}

app.post('/api/users', (req: Request<{}, UserResponse, CreateUserBody>, res: Response<UserResponse>) => {
  const { name, email, age } = req.body; // typed as CreateUserBody
  const user = createUser({ name, email, age });
  res.json({ id: user.id, name: user.name, email: user.email });
});
`

### Beginner Examples

`	ypescript
import { Request, Response } from 'express';

app.get('/hello/:name', (req: Request<{ name: string }>, res: Response) => {
  const { name } = req.params; // typed as string
  res.send(Hello, !);
});
`

### Intermediate Examples

`	ypescript
interface PaginationQuery {
  page?: string;
  limit?: string;
  sort?: string;
  filter?: string;
}

interface ProductBody {
  name: string;
  price: number;
  category: string;
}

interface ProductResponse {
  id: string;
  name: string;
  price: number;
  category: string;
  createdAt: string;
}

app.get('/api/products', (req: Request<{}, ProductResponse[], {}, PaginationQuery>, res: Response<ProductResponse[]>) => {
  const page = parseInt(req.query.page || '1', 10);
  const limit = parseInt(req.query.limit || '20', 10);
  const products = await getProducts({ page, limit, sort: req.query.sort, filter: req.query.filter });
  res.json(products);
});

app.post('/api/products', (req: Request<{}, ProductResponse, ProductBody>, res: Response<ProductResponse>) => {
  const product = await createProduct(req.body);
  res.status(201).json(product);
});
`

### Advanced Examples

`	ypescript
// Typed route handler with async error handling
import { Request, Response, NextFunction } from 'express';

type AsyncRequestHandler<P = {}, ResBody = any, ReqBody = any, ReqQuery = ParsedQs> = (
  req: Request<P, ResBody, ReqBody, ReqQuery>,
  res: Response<ResBody>,
  next: NextFunction
) => Promise<void>;

function asyncHandler<P, ResBody, ReqBody, ReqQuery>(
  fn: AsyncRequestHandler<P, ResBody, ReqBody, ReqQuery>
) {
  return (req: Request<P, ResBody, ReqBody, ReqQuery>, res: Response<ResBody>, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Usage
app.get(
  '/api/users/:id',
  asyncHandler<{ id: string }, UserResponse>(async (req, res) => {
    const user = await findUserById(req.params.id);
    if (!user) {
      res.status(404).json({ error: 'User not found' } as any);
      return;
    }
    res.json(user);
  })
);
`

### Real-World Use Cases

`	ypescript
// Standardized API response type
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  meta?: {
    page: number;
    limit: number;
    total: number;
  };
}

app.get(
  '/api/items',
  async (req: Request<{}, ApiResponse<Item[]>, {}, PaginationQuery>, res: Response<ApiResponse<Item[]>>) => {
    try {
      const { items, total } = await getItems(req.query);
      res.json({
        success: true,
        data: items,
        meta: { page: Number(req.query.page || 1), limit: Number(req.query.limit || 20), total },
      });
    } catch (err) {
      res.status(500).json({ success: false, error: 'Internal server error' });
    }
  }
);
`

### Common Mistakes

`	ypescript
// MISTAKE: Not typing request body
// app.post('/api/data', (req, res) => { req.body.value }); // req.body is any

// MISTAKE: Using type assertions instead of proper generics
// const body = req.body as CreateUserBody; // Less safe than typed req

// MISTAKE: Mixing up the generic parameter order
// Request<ReqBody, ResBody, Params> // Wrong order!
// Correct: Request<Params, ResBody, ReqBody>
`

### Best Practices

- Always type eq.body, eq.params, and eq.query with concrete interfaces.
- Create reusable type aliases for common patterns (e.g., ApiHandler).
- Use syncHandler wrapper to avoid try-catch in every route.
- Define response body types for consistent API responses.

### Performance Considerations

Request/Response typing is compile-time only with zero runtime overhead. However, runtime validation (as in the advanced section) adds minimal overhead and prevents invalid data from reaching your business logic.

### Interview Questions

1. What are the generic type parameters of Express's Request type?
2. How do you type query parameters in Express with TypeScript?
3. What is the correct order of generic parameters for Request<Params, ResBody, ReqBody, ReqQuery>?

### Coding Challenges

1. Create a typed Express route handler for a CRUD API with request validation.
2. Build a generic sendSuccess and sendError response helper with typed responses.

### Related Topics

- Extending Request with custom properties
- Middleware typing
- Error handling middleware

## Extending Request with Custom Properties

### What It Is

Extending the Express Request type allows adding custom properties (like currentUser, correlationId, equestId) that are attached by middleware. TypeScript's declaration merging enables augmenting the Request interface with these properties.

### Why It Is Important

Middleware often attaches properties to the request object (e.g., after authentication). Without proper typing, accessing eq.currentUser would be an error or require unsafe type assertions. Declaration merging provides type-safe access to these custom properties across the entire application.

### How It Works Internally

Express types declare Request as an interface. TypeScript's declaration merging allows multiple declarations of the same interface to be merged. By declaring an interface with the same name in a global module, you add properties to the Request type.

### Syntax

`	ypescript
// types/express.d.ts
declare namespace Express {
  interface Request {
    currentUser?: {
      id: string;
      email: string;
      roles: string[];
    };
    requestId: string;
    correlationId?: string;
  }
}
`

### Beginner Examples

`	ypescript
// types/express.d.ts
export {};

declare global {
  namespace Express {
    interface Request {
      currentUser?: User;
    }
  }
}

// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';

export const authenticate = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Unauthorized' });

  try {
    const decoded = verifyToken(token);
    req.currentUser = { id: decoded.sub, email: decoded.email, roles: decoded.roles };
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// route handler - req.currentUser is now typed
app.get('/api/profile', authenticate, (req, res) => {
  const user = req.currentUser!; // typed as User
  res.json({ id: user.id, email: user.email });
});
`

### Intermediate Examples

`	ypescript
// types/express.d.ts with multiple augmented properties
declare namespace Express {
  interface Request {
    requestId: string;
    startTime: number;
    currentUser?: AuthenticatedUser;
    session?: SessionData;
    tenant?: Tenant;
  }
}

// middleware that sets requestId
import { v4 as uuidv4 } from 'uuid';

export const addRequestId = (req: Request, res: Response, next: NextFunction) => {
  req.requestId = uuidv4();
  res.setHeader('X-Request-Id', req.requestId);
  next();
};

// middleware that measures request duration
export const measureRequest = (req: Request, res: Response, next: NextFunction) => {
  req.startTime = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - req.startTime;
    logger.info({ requestId: req.requestId, method: req.method, path: req.path, duration });
  });
  next();
};
`

### Advanced Examples

`	ypescript
// Generic type for request augmentation
interface AuthenticatedUser {
  id: string;
  email: string;
  name: string;
  roles: string[];
  permissions: string[];
}

declare global {
  namespace Express {
    interface Request {
      currentUser: AuthenticatedUser; // Non-optional when guaranteed by middleware
    }
  }
}

// Middleware that guarantees user is present
export const requireAuth = (req: Request, res: Response, next: NextFunction) => {
  if (!req.currentUser) {
    res.status(401).json({ error: 'Authentication required' });
    return;
  }
  next();
};

// Route with guaranteed auth
app.get('/api/admin/users', requireAuth, (req, res) => {
  // req.currentUser is non-nullable here
  if (!req.currentUser.roles.includes('admin')) {
    res.status(403).json({ error: 'Insufficient permissions' });
    return;
  }
  // ...
});
`

### Real-World Use Cases

`	ypescript
// Multi-tenant request augmentation
declare global {
  namespace Express {
    interface Request {
      tenant: {
        id: string;
        name: string;
        plan: 'free' | 'pro' | 'enterprise';
        features: Set<string>;
      };
      requestId: string;
      session: {
        id: string;
        userId: string;
        expiresAt: Date;
      };
    }
  }
}

export const resolveTenant = async (req: Request, res: Response, next: NextFunction) => {
  const hostname = req.hostname;
  const tenant = await findTenantByHostname(hostname);
  if (!tenant) {
    res.status(404).json({ error: 'Tenant not found' });
    return;
  }
  req.tenant = tenant;
  next();
};
`

### Common Mistakes

`	ypescript
// MISTAKE: Augmenting the wrong interface
// Some versions use express-serve-static-core namespace
// declare namespace Express { ... } // Works for most cases
// declare module 'express-serve-static-core' { interface Request { ... } } // Alternative

// MISTAKE: Not using export {} in the augmentation file
// Without export {}, the file is treated as a script, not a module

// MISTAKE: Making all augmented properties optional
// Properties that middlewares gurarantee should be non-optional
`

### Best Practices

- Create a dedicated 	ypes/express.d.ts file for request augmentations.
- Use declare global { namespace Express { interface Request { ... } } } pattern.
- Keep augmented properties focused on request-scoped data.
- Make properties non-optional when middleware guarantees their presence (with equireAuth).
- Document which middleware sets each property.

### Performance Considerations

Augmenting types has zero runtime impact. However, adding properties to every request object in middleware adds minimal memory overhead. Be mindful of attaching large objects to the request.

### Interview Questions

1. How does declaration merging work with Express's Request type?
2. What is the purpose of declare global in request augmentation files?
3. How do you make augmented request properties optional vs required?

### Coding Challenges

1. Augment the Request type with session and lash properties (flash messages).
2. Create a middleware that adds paginate helper method to Request and type it properly.

### Related Topics

- Declaration merging
- TypeScript module augmentation
- Express middleware types

## Middleware Type Signatures

### What It Is

Middleware are functions that execute during the request-response cycle. Express provides specific type signatures for different kinds of middleware: application-level, router-level, error-handling, and third-party middleware.

### Why It Is Important

Correctly typing middleware ensures that the request, response, and next function are properly typed, and that the middleware's behavior (modifying request/response, ending the cycle, or passing to the next handler) is correctly implemented.

### How It Works Internally

Express types define RequestHandler, ErrorRequestHandler, and RequestParamHandler types. These can be used directly or inferred from inline middleware functions.

`	ypescript
type RequestHandler<P, ResBody, ReqBody, ReqQuery, Locals> = (
  req: Request<P, ResBody, ReqBody, ReqQuery, Locals>,
  res: Response<ResBody, Locals>,
  next: NextFunction
) => void | Promise<void>;

type ErrorRequestHandler<P, ResBody, ReqBody, ReqQuery, Locals> = (
  err: any,
  req: Request<P, ResBody, ReqBody, ReqQuery, Locals>,
  res: Response<ResBody, Locals>,
  next: NextFunction
) => void | Promise<void>;
`

### Syntax

`	ypescript
import { Request, Response, NextFunction, RequestHandler } from 'express';

// Typed middleware
const logger: RequestHandler = (req, res, next) => {
  console.log([]  );
  next();
};

app.use(logger);
`

### Beginner Examples

`	ypescript
import { Request, Response, NextFunction } from 'express';

// Simple logging middleware
const requestLogger = (req: Request, res: Response, next: NextFunction) => {
  console.log(${req.method} );
  next();
};

app.use(requestLogger);

// CORS middleware
const corsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
    return;
  }

  next();
};
`

### Intermediate Examples

`	ypescript
import { Request, Response, NextFunction } from 'express';
import { RateLimiterRedis } from 'rate-limiter-flexible';

// Rate limiting middleware
const rateLimiter = new RateLimiterRedis({
  storeClient: redisClient,
  keyPrefix: 'ratelimit',
  points: 10,
  duration: 1,
});

const rateLimitMiddleware = async (req: Request, res: Response, next: NextFunction) => {
  try {
    await rateLimiter.consume(req.ip);
    next();
  } catch {
    res.status(429).json({ error: 'Too many requests' });
  }
};

// Validation middleware
import { z, ZodSchema } from 'zod';

const validate = (schema: ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      schema.parse(req.body);
      next();
    } catch (err) {
      if (err instanceof z.ZodError) {
        res.status(400).json({ error: 'Validation failed', details: err.errors });
        return;
      }
      next(err);
    }
  };
};
`

### Advanced Examples

`	ypescript
// Generic typed middleware factory
function createMiddleware<T extends Record<string, any>>(
  handler: (req: Request & T, res: Response, next: NextFunction) => void
): RequestHandler {
  return (req, res, next) => handler(req as Request & T, res, next);
}

// Middleware that adds pagination to request
const paginationMiddleware = createMiddleware<{ pagination: { page: number; limit: number; skip: number } }>(
  (req, res, next) => {
    const page = Math.max(1, parseInt(req.query.page as string) || 1);
    const limit = Math.min(100, Math.max(1, parseInt(req.query.limit as string) || 20));
    req.pagination = { page, limit, skip: (page - 1) * limit };
    next();
  }
);

// Middleware that adds response helpers
const responseHelpers = (req: Request, res: Response, next: NextFunction) => {
  res.locals.sendSuccess = (data: any, statusCode = 200) => {
    res.status(statusCode).json({ success: true, data });
  };
  res.locals.sendError = (message: string, statusCode = 500) => {
    res.status(statusCode).json({ success: false, error: message });
  };
  next();
};
`

### Real-World Use Cases

`	ypescript
// Authentication middleware chain
const authenticate: RequestHandler = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    res.status(401).json({ error: 'Missing token' });
    return;
  }
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.currentUser = decoded as User;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};

const authorize = (...allowedRoles: string[]): RequestHandler => {
  return (req, res, next) => {
    if (!req.currentUser) {
      res.status(401).json({ error: 'Not authenticated' });
      return;
    }
    if (!allowedRoles.some(role => req.currentUser!.roles.includes(role))) {
      res.status(403).json({ error: 'Forbidden' });
      return;
    }
    next();
  };
};

// Usage
app.get('/api/admin', authenticate, authorize('admin'), adminHandler);
`

### Common Mistakes

`	ypescript
// MISTAKE: Not calling next() in middleware
// const badMiddleware = (req, res) => { /* never calls next */ }

// MISTAKE: Forgetting the 4th parameter for error middleware
// Express identifies error middleware by checking the arity (4 parameters)

// MISTAKE: Returning a value from middleware instead of calling next
// return next() is fine (next returns void), but actual return value is ignored
`

### Best Practices

- Always call 
ext() to pass control to the next middleware (unless sending a response).
- Use RequestHandler type annotation for clarity.
- Use middleware factories for parameterized middleware.
- Keep middleware focused on a single responsibility.
- Handle errors in middleware by passing them to 
ext(err).

### Performance Considerations

Middleware executes sequentially for every matching request. Keep middleware lightweight. Expensive operations (DB queries, external API calls) should be cached or deferred. The type system has no runtime impact.

### Interview Questions

1. What is the difference between RequestHandler and ErrorRequestHandler?
2. How does Express determine if a middleware is for error handling?
3. What happens if middleware neither sends a response nor calls next()?

### Coding Challenges

1. Create a alidateBody middleware factory that takes a Zod schema and validates eq.body.
2. Build a cacheResponse middleware that caches GET responses in Redis.

### Related Topics

- Error handling middleware
- Request augmentation
- Express middleware patterns

## Error Handling Middleware Types

### What It Is

Error handling middleware in Express has a specific signature with four parameters: (err, req, res, next). TypeScript provides the ErrorRequestHandler type for this purpose, ensuring that the first parameter is an error object and that the other parameters are correctly typed.

### Why It Is Important

Properly typed error handling middleware ensures that errors are caught and formatted consistently. TypeScript catches mistakes like missing parameters or incorrect error object shapes, leading to more robust error handling.

### How It Works Internally

Express identifies error handling middleware by checking the function's .length property (number of parameters). The ErrorRequestHandler type enforces the four-parameter signature. Express skips normal middleware and jumps to the nearest error handler when an error is passed to 
ext(err).

### Syntax

`	ypescript
import { Request, Response, NextFunction, ErrorRequestHandler } from 'express';

const errorHandler: ErrorRequestHandler = (err, req, res, next) => {
  console.error('Error:', err);
  res.status(500).json({ error: 'Internal server error' });
};

app.use(errorHandler);
`

### Beginner Examples

`	ypescript
// Basic error handler
const errorHandler: ErrorRequestHandler = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.statusCode ? err.message : 'Internal server error';

  res.status(statusCode).json({
    error: message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
};

app.use(errorHandler);
`

### Intermediate Examples

`	ypescript
// Custom AppError class
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

// Validation error
export class ValidationError extends AppError {
  constructor(public errors: Array<{ field: string; message: string }>) {
    super(400, 'Validation failed');
  }
}

// Typed error handler
const errorHandler: ErrorRequestHandler = (err, req, res, next) => {
  if (err instanceof ValidationError) {
    res.status(400).json({
      error: 'Validation failed',
      details: err.errors,
    });
    return;
  }

  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: err.message,
      ...(err.isOperational ? {} : { details: 'An unexpected error occurred' }),
    });
    return;
  }

  // Unknown errors
  console.error('Unexpected error:', err);
  res.status(500).json({ error: 'Internal server error' });
};
`

### Advanced Examples

`	ypescript
// Async error wrapper
const asyncErrorWrapper = (fn: RequestHandler): RequestHandler => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Global error handler with multiple error types
interface ErrorResponse {
  error: string;
  details?: unknown;
  stack?: string;
  requestId?: string;
}

const globalErrorHandler: ErrorRequestHandler = (err, req, res, next) => {
  const response: ErrorResponse = {
    error: 'Internal server error',
    requestId: req.requestId,
  };

  // Known error types
  if (err instanceof SyntaxError && 'body' in err) {
    response.error = 'Invalid JSON';
    res.status(400).json(response);
    return;
  }

  if (err.name === 'UnauthorizedError') {
    response.error = 'Invalid or expired token';
    res.status(401).json(response);
    return;
  }

  if (err.code === 'EBADCSRFTOKEN') {
    response.error = 'Invalid CSRF token';
    res.status(403).json(response);
    return;
  }

  // Mongoose errors
  if (err.name === 'ValidationError') {
    response.error = 'Database validation failed';
    response.details = err.errors;
    res.status(400).json(response);
    return;
  }

  if (err.code === 11000) {
    response.error = 'Duplicate key error';
    res.status(409).json(response);
    return;
  }

  // Log unknown errors
  console.error('Unhandled error:', { message: err.message, stack: err.stack, requestId: req.requestId });

  if (process.env.NODE_ENV === 'development') {
    response.stack = err.stack;
  }

  res.status(err.statusCode || 500).json(response);
};
`

### Real-World Use Cases

`	ypescript
// Centralized error handler with logging and monitoring
class ErrorHandler {
  private readonly isProduction = process.env.NODE_ENV === 'production';

  handle: ErrorRequestHandler = (err, req, res, next) => {
    const error = this.normalizeError(err);
    this.logError(error, req);

    if (error.isOperational) {
      res.status(error.statusCode).json({
        error: error.message,
        ...(this.isProduction ? {} : { stack: error.stack }),
      });
    } else {
      res.status(500).json({
        error: 'Internal server error',
        requestId: req.requestId,
      });
    }
  };

  private normalizeError(err: any): AppError {
    if (err instanceof AppError) return err;
    if (err.name === 'CastError') return new AppError(400, 'Invalid ID format');
    if (err.code === 11000) return new AppError(409, 'Duplicate field value');
    if (err.name === 'JsonWebTokenError') return new AppError(401, 'Invalid token');
    if (err.name === 'TokenExpiredError') return new AppError(401, 'Token expired');
    return new AppError(500, 'Internal server error', false);
  }

  private logError(error: AppError, req: Request) {
    logger.error({
      message: error.message,
      stack: error.stack,
      path: req.path,
      method: req.method,
      requestId: req.requestId,
      userId: req.currentUser?.id,
    });
  }
}

app.use(new ErrorHandler().handle);
`

### Common Mistakes

`	ypescript
// MISTAKE: Error handler with wrong number of parameters
// Express uses function.length to detect error handlers
// function handler(err, req, res) {} // 3 params -> treated as normal middleware!
// function handler(err, req, res, next) {} // 4 params -> treated as error handler

// MISTAKE: Not placing error handler after all routes
// app.use(router);
// const handler: ErrorRequestHandler = ...;
// app.use(handler); // Must be after routes

// MISTAKE: Using app.use for error handler without proper type
`

### Best Practices

- Define custom error classes that extend Error with a statusCode property.
- Use ErrorRequestHandler type for error handling middleware.
- Place error handling middleware at the end of the middleware stack, after all routes.
- Differentiate between operational errors (expected) and programmer errors (unexpected).
- Never send stack traces in production.
- Use a centralized error handler for consistent error responses.

### Performance Considerations

Error handling middleware runs only when an error occurs. The performance impact is negligible. However, creating stack traces on every error can be expensive; avoid throwing errors in normal control flow.

### Interview Questions

1. How does Express identify error handling middleware?
2. What are the four parameters of an Express error handler and their types?
3. How do you create custom error classes that work with Express error handlers?

### Coding Challenges

1. Create an AppError class hierarchy with NotFoundError, UnauthorizedError, ForbiddenError, and ValidationError.
2. Build a global error handler that formats errors for JSON API compliance.
