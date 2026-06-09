# Validation - Zod schemas, runtime validation, type inference from schemas, class-validator

## Introduction

Validation is a critical layer in any application that accepts external input. TypeScript provides compile-time type checking, but types are erased at runtime, making runtime validation essential. Zod is a TypeScript-first schema validation library that combines runtime validation with type inference, providing a single source of truth for data shapes. Class-validator and class-transformer offer an alternative decorator-based approach. This file covers Zod schemas, type inference from schemas, class-validator with class-transformer, and the relationship between runtime and compile-time validation.

## Zod Schemas

### What It Is

Zod is a TypeScript-first schema declaration and validation library. Schemas define the shape, type, and constraints of data at runtime, and TypeScript types can be inferred from schemas automatically.

### Why It Is Important

Zod eliminates duplication between runtime validation and TypeScript types. Instead of maintaining separate validation logic and type definitions, Zod schemas serve as the single source of truth. This ensures that validation logic and type information never drift apart.

### How It Works Internally

Zod creates a schema object for each type. The schema's .parse() method validates input against the schema, returning the typed value or throwing a ZodError. .safeParse() returns a discriminated union { success: true; data: T } | { success: false; error: ZodError }.

### Syntax

`	ypescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(2).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
  tags: z.array(z.string()).max(10).default([]),
  metadata: z.record(z.string(), z.unknown()).optional(),
});

type User = z.infer<typeof UserSchema>;
`

### Beginner Examples

`	ypescript
import { z } from 'zod';

// String validation
const NameSchema = z.string().min(1).max(50);
NameSchema.parse('Alice'); // OK
NameSchema.parse(''); // Throws: must be at least 1 character

// Number validation
const AgeSchema = z.number().int().positive().max(150);
AgeSchema.parse(25); // OK
AgeSchema.parse(-1); // Throws: must be positive

// Boolean
const IsActiveSchema = z.boolean();

// Enum
const ColorSchema = z.enum(['red', 'green', 'blue']);

// Optional and defaults
const ConfigSchema = z.object({
  host: z.string().default('localhost'),
  port: z.number().default(3000),
});
`

### Intermediate Examples

`	ypescript
import { z } from 'zod';

// Union types
const ResultSchema = z.union([
  z.object({ status: z.literal('success'), data: z.unknown() }),
  z.object({ status: z.literal('error'), message: z.string() }),
]);

// Discriminated union
const EventSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number(), y: z.number() }),
  z.object({ type: z.literal('scroll'), position: z.number() }),
  z.object({ type: z.literal('keypress'), key: z.string() }),
]);

// Refinements
const PasswordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .refine((val) => /[A-Z]/.test(val), 'Must contain an uppercase letter')
  .refine((val) => /[0-9]/.test(val), 'Must contain a number');

// Transformations
const DateStringSchema = z.string().datetime().transform((val) => new Date(val));
const NumberStringSchema = z.string().transform((val) => parseFloat(val));

// Recursive schemas
interface Category {
  name: string;
  children: Category[];
}
const CategorySchema: z.ZodType<Category> = z.object({
  name: z.string(),
  children: z.lazy(() => CategorySchema.array()),
});
`

### Advanced Examples

`	ypescript
import { z } from 'zod';

// Generic constraint
function createPaginatedSchema<T extends z.ZodTypeAny>(itemSchema: T) {
  return z.object({
    data: z.array(itemSchema),
    meta: z.object({
      total: z.number().int().nonnegative(),
      page: z.number().int().positive(),
      limit: z.number().int().positive(),
      hasMore: z.boolean(),
    }),
  });
}

const UserPaginatedSchema = createPaginatedSchema(z.object({
  id: z.string().uuid(),
  name: z.string(),
}));

type UserPaginated = z.infer<typeof UserPaginatedSchema>;

// Pipeline validation
const OrderSchema = z.object({
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
  couponCode: z.string().optional(),
}).superRefine((data, ctx) => {
  // Cross-field validation: ensure items exist in inventory
  // Add issues to context instead of throwing
});

// Branded types
type UserId = z.Branded<string, 'UserId'>;
const UserIdSchema = z.string().uuid().brand<'UserId'>();

function getUser(id: UserId) {
  // id is branded as UserId, not just any string
}
`

### Real-World Use Cases

`	ypescript
// Environment variable validation
const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.coerce.number().int().positive().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  REDIS_URL: z.string().url().optional(),
  SENTRY_DSN: z.string().url().optional(),
});

const env = EnvSchema.parse(process.env);
export default env;

// Form validation
const ContactFormSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email'),
  subject: z.string().min(5, 'Subject must be at least 5 characters'),
  message: z.string().min(10, 'Message must be at least 10 characters'),
}).refine((data) => !data.subject.includes(data.name), {
  message: 'Subject should not contain your name',
  path: ['subject'],
});
`

### Common Mistakes

`	ypescript
// MISTAKE: Not handling ZodError properly
// try { schema.parse(data) } catch (err) { console.log(err); }
// err is unknown - must check instanceof ZodError

// MISTAKE: Using .parse() when .safeParse() is more appropriate
// .parse() throws - use .safeParse() for expected validation

// MISTAKE: Not using .coerce for string inputs
// req.query.page is a string. z.number() fails.
// Use z.coerce.number() to convert
`

### Best Practices

- Use Zod as the single source of truth for data shapes, deriving TypeScript types with z.infer.
- Use .safeParse() for expected validation scenarios (form submission, API requests).
- Use .parse() for initialization validation (environment variables, config files).
- Use .brand() for type-safe identifiers.
- Use .superRefine() for complex cross-field validation.
- Co-locate schemas with their usage context (near routes, near forms).

### Performance Considerations

Zod is well-optimized for validation performance. Schema compilation happens once at module load time. .parse() and .safeParse() have linear complexity relative to data size. For high-throughput validation, consider caching schemas (Zod does this natively).

### Interview Questions

1. How does Zod differ from TypeScript's type system for validation?
2. What is the purpose of .safeParse() vs .parse()?
3. How do you handle cross-field validation in Zod?
4. What are branded types in Zod and when would you use them?

### Coding Challenges

1. Create a Zod schema for a multi-step checkout form with cross-field validation.
2. Build a generic validation utility that uses Zod schemas to validate API responses.

### Related Topics

- Type inference from Zod
- Runtime vs compile-time validation
- Zod with Express middleware

## Type Inference from Zod

### What It Is

Zod schemas can be used to generate TypeScript types using the z.infer utility type. This eliminates the need to maintain separate type definitions and validation schemas, ensuring they stay in sync.

### Why It Is Important

Type inference from Zod schemas ensures that the types your code uses are always consistent with the validation logic. Changing the schema automatically updates both the runtime validation and the compile-time type, eliminating a class of bugs where types and validation drift apart.

### How It Works Internally

z.infer<typeof schema> traverses the Zod schema and produces a TypeScript type that matches the schema's output shape. For example, z.string().min(1) produces string, z.object({ name: z.string() }) produces { name: string }.

### Syntax

`	ypescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

type User = z.infer<typeof UserSchema>;
// Equivalent to:
// type User = {
//   id: string;
//   name: string;
//   email: string;
//   age?: number;
// }
`

### Beginner Examples

`	ypescript
const ProductSchema = z.object({
  id: z.string(),
  name: z.string(),
  price: z.number().positive(),
  inStock: z.boolean(),
  tags: z.array(z.string()),
});

type Product = z.infer<typeof ProductSchema>;

// Usage
function formatProduct(product: Product): string {
  return ${product.name}: {product.price.toFixed(2)} ();
}
`

### Intermediate Examples

`	ypescript
// Nested inference
const OrderSchema = z.object({
  orderId: z.string().uuid(),
  customer: z.object({
    id: z.string(),
    email: z.string().email(),
  }),
  items: z.array(z.object({
    productId: z.string(),
    quantity: z.number().int().positive(),
    unitPrice: z.number().positive(),
  })),
  total: z.number().positive(),
  createdAt: z.date(),
});

type Order = z.infer<typeof OrderSchema>;
// type Order = {
//   orderId: string;
//   customer: { id: string; email: string };
//   items: { productId: string; quantity: number; unitPrice: number }[];
//   total: number;
//   createdAt: Date;
// }

// Extracting nested types
type OrderItem = z.infer<typeof OrderSchema>['items'][number];
// { productId: string; quantity: number; unitPrice: number }

// Input vs output types
type OrderInput = z.input<typeof OrderSchema>;
// The type expected by .parse() - may differ from output due to transforms
type OrderOutput = z.output<typeof OrderSchema>;
// The type returned by .parse() - matches z.infer
`

### Advanced Examples

`	ypescript
// Using inferred types in generic functions
function createEntity<T extends z.ZodTypeAny>(
  schema: T,
  data: z.input<T>
): z.output<T> {
  return schema.parse(data);
}

const user = createEntity(UserSchema, { name: 'Alice', email: 'alice@test.com' });
// user is typed as z.output<typeof UserSchema>

// Combining schemas
const BaseEntitySchema = z.object({
  id: z.string().uuid(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

const UserSchema2 = BaseEntitySchema.extend({
  name: z.string(),
  email: z.string().email(),
});

type User2 = z.infer<typeof UserSchema2>;
// { id: string; createdAt: Date; updatedAt: Date; name: string; email: string }

// Schema pick/omit
const PublicUserSchema = UserSchema2.omit({ updatedAt: true });
type PublicUser = z.infer<typeof PublicUserSchema>;
// { id: string; createdAt: Date; name: string; email: string }

// Branded type inference
const UserIdSchema = z.string().uuid().brand();
type UserId = z.infer<typeof UserIdSchema>;
`

### Real-World Use Cases

`	ypescript
// Repository layer with inferred types
interface Repository<T extends z.ZodTypeAny> {
  schema: T;
  create(data: z.input<T>): Promise<z.output<T>>;
  findById(id: string): Promise<z.output<T> | null>;
  update(id: string, data: Partial<z.input<T>>): Promise<z.output<T>>;
}

class UserRepository implements Repository<typeof UserSchema> {
  schema = UserSchema;

  async create(data: z.input<typeof UserSchema>): Promise<z.output<typeof UserSchema>> {
    const validated = this.schema.parse(data);
    return db.insert('users', validated);
  }

  async findById(id: string): Promise<z.output<typeof UserSchema> | null> {
    return db.find('users', id);
  }

  async update(id: string, data: Partial<z.input<typeof UserSchema>>): Promise<z.output<typeof UserSchema>> {
    const validated = this.schema.partial().parse(data);
    return db.update('users', id, validated);
  }
}
`

### Common Mistakes

`	ypescript
// MISTAKE: Manually duplicating types alongside Zod schemas
interface User { name: string; email: string; }
const UserSchema = z.object({ name: z.string(), email: z.string().email() });
// If you change one, the other drifts!

// MISTAKE: Using z.infer on the wrong thing
type T = z.infer<z.ZodString>; // string - but why not just use string?

// MISTAKE: Forgetting that transforms change the output type
const NumberSchema = z.string().transform(Number);
type Num = z.infer<typeof NumberSchema>; // number, not string
type In = z.input<typeof NumberSchema>; // string
`

### Best Practices

- Always use z.infer<typeof schema> instead of manually defining types that match schemas.
- Use z.input<typeof schema> for the type accepted by .parse() (may differ from output due to transforms).
- Co-locate schemas with their usage, exporting both the schema and the inferred type.
- Use schema composition (extend, pick, omit) to derive related schemas instead of repeating field definitions.
- Export branded type aliases for identifier types.

### Performance Considerations

z.infer is a pure type-level operation with zero runtime cost. The type is computed at compile time during type checking. Complex inferred types may slightly increase type-checking time but have no runtime impact.

### Interview Questions

1. What is z.infer and how does it relate Zod schemas to TypeScript types?
2. What is the difference between z.input and z.output in Zod?
3. How do you derive a partial version of a Zod schema's inferred type?

### Coding Challenges

1. Create a Zod schema for a configuration object and export both the schema and inferred type.
2. Build a generic validation middleware that infers types from Zod schemas.

### Related Topics

- Zod schemas
- Runtime validation
- class-validator

## class-validator and class-transformer

### What It Is

class-validator is a decorator-based validation library that uses TypeScript decorators to define validation rules on class properties. class-transformer transforms plain objects to class instances. Together, they provide an alternative to Zod for runtime validation.

### Why It Is Important

For codebases that use classes extensively, class-validator provides a natural validation approach using decorators. It integrates well with NestJS and frameworks that rely on class-based DTOs. class-transformer ensures that incoming JSON is transformed to proper class instances with typed properties.

### How It Works Internally

class-validator uses decorators to register validation metadata on class properties. The alidate() function reads this metadata and checks each property against the defined rules. class-transformer uses plainToClass() to convert plain objects to class instances, applying type transformations.

### Syntax

`	ypescript
import { IsString, IsEmail, MinLength, MaxLength, IsInt, Min, Max, IsOptional } from 'class-validator';
import { Type } from 'class-transformer';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  name!: string;

  @IsEmail()
  email!: string;

  @IsString()
  @MinLength(8)
  password!: string;

  @IsOptional()
  @IsInt()
  @Min(18)
  @Max(150)
  @Type(() => Number)
  age?: number;
}
`

### Beginner Examples

`	ypescript
import { IsString, IsNotEmpty, IsEmail, MinLength } from 'class-validator';
import { plainToClass } from 'class-transformer';

class LoginDto {
  @IsEmail()
  @IsNotEmpty()
  email!: string;

  @IsString()
  @MinLength(6)
  @IsNotEmpty()
  password!: string;
}

// Usage in Express
app.post('/login', async (req, res) => {
  const dto = plainToClass(LoginDto, req.body);
  const errors = await validate(dto);

  if (errors.length > 0) {
    res.status(400).json({
      error: 'Validation failed',
      details: errors.map(e => ({
        property: e.property,
        constraints: e.constraints,
      })),
    });
    return;
  }

  // dto is a validated LoginDto instance
  const user = await authenticate(dto.email, dto.password);
  res.json(user);
});
`

### Intermediate Examples

`	ypescript
import {
  IsString, IsEmail, IsOptional, IsEnum, IsArray,
  ValidateNested, MinLength, MaxLength, IsUUID, ArrayMinSize
} from 'class-validator';
import { Type } from 'class-transformer';

enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
}

class OrderItemDto {
  @IsUUID()
  productId!: string;

  @Type(() => Number)
  quantity!: number;

  @Type(() => Number)
  unitPrice!: number;
}

class CreateOrderDto {
  @IsUUID()
  customerId!: string;

  @IsArray()
  @ValidateNested({ each: true })
  @ArrayMinSize(1)
  @Type(() => OrderItemDto)
  items!: OrderItemDto[];

  @IsOptional()
  @IsString()
  couponCode?: string;

  @IsOptional()
  @IsEnum(OrderStatus)
  status?: OrderStatus;
}

// Nested validation
const body = req.body;
const orderDto = plainToClass(CreateOrderDto, body);
const errors = await validate(orderDto, { whitelist: true, forbidNonWhitelisted: true });
`

### Advanced Examples

`	ypescript
import {
  ValidatorConstraint, ValidatorConstraintInterface,
  ValidationArguments, registerDecorator, ValidationOptions,
  validate, ValidationError
} from 'class-validator';
import { plainToClass, Transform } from 'class-transformer';

// Custom validator
@ValidatorConstraint({ name: 'isStrongPassword', async: false })
class IsStrongPasswordConstraint implements ValidatorConstraintInterface {
  validate(password: string, args: ValidationArguments) {
    return (
      password.length >= 8 &&
      /[a-z]/.test(password) &&
      /[A-Z]/.test(password) &&
      /[0-9]/.test(password) &&
      /[!@#$%^&*]/.test(password)
    );
  }

  defaultMessage(args: ValidationArguments) {
    return 'Password must contain lowercase, uppercase, number, and special character';
  }
}

function IsStrongPassword(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsStrongPasswordConstraint,
    });
  };
}

// Custom transformer
function Trim(): PropertyDecorator {
  return Transform(({ value }) => typeof value === 'string' ? value.trim() : value);
}

function ToLowerCase(): PropertyDecorator {
  return Transform(({ value }) => typeof value === 'string' ? value.toLowerCase() : value);
}

class RegisterDto {
  @Trim()
  @ToLowerCase()
  @IsEmail()
  email!: string;

  @Trim()
  @IsString()
  @MinLength(2)
  name!: string;

  @IsStrongPassword()
  password!: string;

  @Transform(({ value }) => new Date(value))
  @IsOptional()
  dateOfBirth?: Date;
}

// Validation groups
class UserDto {
  @IsString({ groups: ['create', 'update'] })
  name!: string;

  @IsEmail({ groups: ['create'] })
  email!: string;

  @IsString({ groups: ['create'] })
  password!: string;

  @IsOptional()
  @IsString({ groups: ['update'] })
  bio?: string;
}

// Usage with groups
const errors = await validate(userDto, { groups: ['create'] });
`

### Real-World Use Cases

`	ypescript
// NestJS integration
import { Controller, Post, Body, UsePipes, ValidationPipe } from '@nestjs/common';

class CreateUserDto {
  @IsString()
  @MinLength(2)
  name!: string;

  @IsEmail()
  email!: string;

  @IsString()
  @MinLength(8)
  password!: string;
}

@Controller('users')
export class UserController {
  @Post()
  @UsePipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))
  async create(@Body() createUserDto: CreateUserDto) {
    // createUserDto is validated and transformed
    return this.userService.create(createUserDto);
  }
}
`

### Common Mistakes

`	ypescript
// MISTAKE: Forgetting to use @Type() decorator for nested objects
// Without @Type(() => NestedDto), nested objects remain plain objects

// MISTAKE: Not using ! (definite assignment assertion) on class properties
// TypeScript strict mode requires properties to be initialized

// MISTAKE: Forgetting whitelist: true option
// Without it, extra properties are not stripped, potentially passing through unintended data
`

### Best Practices

- Use plainToClass to transform incoming data before validation.
- Enable whitelist: true and orbidNonWhitelisted: true for security.
- Use @Type() decorator for nested object and array transformations.
- Create custom validators for complex business rules.
- Use validation groups for different operation contexts (create vs update).
- Combine with class-transformer for type coercion.

### Performance Considerations

class-validator validation is generally fast but uses reflection metadata, which adds overhead. For high-throughput endpoints, consider caching validated instances or using lighter-weight validation. The metadata reflection (eflect-metadata) must be imported once at application entry.

### Interview Questions

1. How does class-validator differ from Zod in approach?
2. What is the purpose of plainToClass from class-transformer?
3. How do you create custom validators in class-validator?
4. What does the whitelist option do in class-validator?

### Coding Challenges

1. Create a custom validator that checks if a date is in the future.
2. Build a DTO class hierarchy for a blog system with CreatePost, UpdatePost, and PostResponse DTOs.

### Related Topics

- Zod schemas
- Runtime vs compile-time validation
- NestJS validation pipes

## Runtime vs Compile-Time Validation

### What It Is

Compile-time validation (TypeScript types) checks correctness during development. Runtime validation (Zod, class-validator) checks data during execution. Understanding the distinction and knowing when to use each is essential for building robust applications.

### Why It Is Important

Relying solely on TypeScript types for validation gives a false sense of security because types are erased at runtime. External data (API requests, file reads, user input) bypasses TypeScript's type system entirely. Runtime validation is the safety net that catches invalid data before it reaches business logic.

### How It Works Internally

TypeScript types are checked by the compiler and erased in the output JavaScript. Runtime validation libraries store validation rules as data structures (Zod schemas) or metadata (class-validator decorators) that persist at runtime and execute validation logic.

### Syntax

`	ypescript
// Compile-time only - ERASED AT RUNTIME
interface UserInput {
  name: string;
  email: string;
  age: number;
}

// Runtime validation - PERSISTS AT RUNTIME
const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().int().positive(),
});

// Both combined
type ValidatedUser = z.infer<typeof UserSchema>;

function processUser(data: unknown): ValidatedUser {
  return UserSchema.parse(data); // Runtime validation produces typed output
}
`

### Beginner Examples

`	ypescript
// BAD: Trusting TypeScript for external data
function createUser(input: UserInput) {
  // input.name is typed as string, but could be null at runtime!
  return db.insert(input);
}

// GOOD: Validating at runtime
function createUser(input: unknown) {
  const validated = UserSchema.parse(input);
  return db.insert(validated);
}
`

### Intermediate Examples

`	ypescript
// Defense in depth: types + validation + sanitization
interface SafeUser {
  id: string;
  name: string;
  email: string;
}

const UserSchema = z.object({
  name: z.string().min(1).max(100).transform(sanitizeHtml),
  email: z.string().email().transform(s => s.toLowerCase()),
  password: z.string().min(8),
});

class UserService {
  createUser(rawData: unknown): SafeUser {
    const validated = UserSchema.parse(rawData);
    const user = this.userRepository.create(validated);
    return { id: user.id, name: user.name, email: user.email };
  }
}
`

### Advanced Examples

`	ypescript
// Runtime type guards
function isUser(obj: unknown): obj is User {
  try {
    UserSchema.parse(obj);
    return true;
  } catch {
    return false;
  }
}

// Assertion function
function assertUser(obj: unknown): asserts obj is User {
  UserSchema.parse(obj);
}

// Usage
function handleRequest(data: unknown) {
  assertUser(data); // Throws if invalid
  // data is now typed as User
}
`

### Real-World Use Cases

`	ypescript
// Multi-layer validation strategy
// 1. Transport layer: Parse JSON body
// 2. Validation layer: Zod/class-validator
// 3. Domain layer: TypeScript types (compile-time)
// 4. Database layer: Schema constraints (runtime)

// Example
app.post('/api/users', async (req, res) => {
  // Layer 1: JSON parsing (Express does this)
  // Layer 2: Runtime validation
  const result = UserSchema.safeParse(req.body);
  if (!result.success) {
    res.status(400).json({ errors: result.error.flatten() });
    return;
  }

  // Layer 3: result.data is fully typed
  const user = await userService.create(result.data);

  // Layer 4: Database will enforce constraints
  res.json(user);
});
`

### Common Mistakes

`	ypescript
// MISTAKE: Assuming TypeScript types protect against invalid external data
// Input from API, files, or user always needs runtime validation

// MISTAKE: Only validating at one layer
// Validate at the boundary (API request), not deep in business logic

// MISTAKE: Using s type assertions as a substitute for validation
// const data = req.body as UserInput; // Unsafe! No runtime check
`

### Best Practices

- Validate all external input at the system boundary.
- Use TypeScript types for internal consistency (function signatures, return types).
- Use runtime validation for external data (API requests, file reads, environment variables).
- Combine both: define schemas and derive types from them.
- Validate early, validate once at the boundary, then use typed data internally.
- Use assertion functions or type guards for safe narrowing.

### Performance Considerations

Runtime validation adds overhead proportional to data complexity. For high-throughput systems, consider:
- Validating only at API boundaries, not internally.
- Using cached schema instances (Zod compiles once).
- Skipping validation in trusted internal communication.
- Using lighter validation for performance-critical paths while keeping full validation for mutation operations.

### Interview Questions

1. Why is runtime validation necessary when using TypeScript?
2. What is the difference between type checking and validation?
3. How do you balance between compile-time and runtime validation?
4. What is a type guard and how does it relate to runtime validation?

### Coding Challenges

1. Create a validation pipeline that validates, sanitizes, and types API request data.
2. Build a safe JSON parser that validates unknown JSON against a Zod schema.

### Related Topics

- Zod schemas
- class-validator
- Type guards and assertions
