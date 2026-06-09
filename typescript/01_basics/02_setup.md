# Setup - Installing TS, tsc compiler, tsconfig.json, VS Code setup

## Introduction

Setting up a proper TypeScript development environment is the first step toward productive development. From installing the compiler to configuring your IDE, each component plays a role in creating a smooth development experience. This guide covers everything from the initial `npm install` to advanced `tsconfig.json` configurations and VS Code integration that will supercharge your workflow.

## Installing TypeScript (npm)

### What It Is

TypeScript can be installed globally or locally in your project via npm (Node Package Manager). The package `typescript` includes the TypeScript compiler (`tsc`), language services, and standard library type definitions.

### Why It Is Important

Proper installation ensures you have access to the TypeScript compiler and language services. Local installation (per project) is recommended over global installation to ensure consistent versions across development environments and CI/CD pipelines.

### How It Works Internally

The `typescript` npm package ships as a Node.js package containing the TypeScript compiler (`bin/tsc`), the language service for IDE integration, and type definitions for the JavaScript standard library (`lib.d.ts`). When you run `npx tsc`, Node.js resolves the locally installed package and executes the compiler, which reads your source files and `tsconfig.json` to orchestrate compilation.

### Syntax

```bash
# Global installation (not recommended for projects)
npm install -g typescript

# Local installation (recommended per project)
npm install --save-dev typescript

# Verify installation
npx tsc --version

# Check TypeScript version
npx tsc -v
```

### Beginner Examples

```bash
# Create a new project directory
mkdir my-ts-project
cd my-ts-project

# Initialize npm
npm init -y

# Install TypeScript locally
npm install --save-dev typescript

# Create your first TypeScript file
echo 'const greeting: string = "Hello, TypeScript!"; console.log(greeting);' > index.ts

# Compile it
npx tsc index.ts

# Run the output
node index.js
```

### Intermediate Examples

```typescript
// package.json with TypeScript scripts
{
  "name": "my-ts-project",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc",
    "watch": "tsc --watch",
    "type-check": "tsc --noEmit",
    "clean": "rimraf dist"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "rimraf": "^5.0.0"
  }
}
```

### Advanced Examples

```bash
# Using TypeScript with different module systems
# ESM setup
npm install --save-dev typescript @types/node

# tsconfig.json for ESM
# {
#   "compilerOptions": {
#     "module": "NodeNext",
#     "moduleResolution": "NodeNext",
#     "target": "ESNext",
#     "outDir": "./dist",
#     "rootDir": "./src"
#   }
# }

# Installing multiple TS versions for testing
npm install --save-dev typescript@5.3 typescript@5.4

# Using TypeScript nightly builds
npm install --save-dev typescript@next
```

### Real-World Use Cases

**Monorepo Setup**: Large organizations install TypeScript at the root of a monorepo (e.g., Nx, Turborepo) with a single version shared across all packages.

```json
// Root package.json
{
  "devDependencies": {
    "typescript": "^5.4.0"
  },
  "workspaces": ["packages/*"]
}
```

**Dockerized Development**: TypeScript can be installed in Docker containers for consistent build environments.

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

### Common Mistakes

**Using Different TS Versions**: Different packages in a project using different TypeScript versions can cause type incompatibilities and confusing errors.

**Skipping `@types/node`**: For Node.js projects, forgetting to install `@types/node` means missing type definitions for Node.js APIs like `process`, `fs`, and `Buffer`.

**Installing Globally for Projects**: Global TypeScript installation can lead to version mismatches across different projects and environments.

### Best Practices

1. Always install TypeScript as a `devDependency` locally
2. Pin a specific TypeScript version for stability
3. Use `npx tsc` instead of global `tsc` to guarantee local version usage
4. Include TypeScript in `engines` field of `package.json`
5. Use `.nvmrc` to ensure consistent Node.js version across environments

### Performance Considerations

Local installation adds to `node_modules` size (TypeScript is ~60MB). CI/CD pipelines should cache `node_modules` to speed up installations. For monorepos, hoisting TypeScript to the root reduces duplication.

### Interview Questions

1. Why is local TypeScript installation preferred over global?
2. What does the `@types/node` package provide?
3. How do you ensure the same TypeScript version across your team?
4. What is the purpose of `npx tsc --noEmit`?
5. How does TypeScript interact with different Node.js versions?

### Coding Challenges

1. Set up a TypeScript project from scratch with `npm init` and local TypeScript
2. Create a script that compiles TypeScript and runs it in one command
3. Configure a monorepo with shared TypeScript configuration

### Related Topics

- npm and Yarn package managers
- Node.js version management (nvm)
- Monorepo tools (Nx, Turborepo)
- ESBuild and SWC as TypeScript alternatives

## tsc compiler

### What It Is

The TypeScript compiler (`tsc`) is the official tool that compiles TypeScript code to JavaScript. It performs type checking, syntax validation, and code transformation according to compiler options specified in `tsconfig.json` or command-line arguments.

### Why It Is Important

`tsc` is the primary tool for transforming TypeScript into executable JavaScript. Understanding its options and behavior is essential for configuring builds, debugging compilation issues, and optimizing development workflows.

### How It Works Internally

`tsc` processes files through multiple phases:
1. **Program Creation**: Reads `tsconfig.json` (or command-line args), creates a map of all input files and their dependencies
2. **Parsing and Binding**: Parses files into AST, creates symbols for declarations, resolves module references
3. **Type Checking**: Performs type checking across the entire program, reporting diagnostics
4. **Emission**: Generates JavaScript output, declaration files, and source maps

The compiler uses a `CompilerHost` interface for file system interactions and a `Program` object that manages the compilation context.

### Syntax

```bash
# Basic compilation
tsc

# Compile specific files
tsc file1.ts file2.ts

# Specify output
tsc --outDir ./dist

# Watch mode — recompile on changes
tsc --watch

# Emit declaration files only
tsc --declaration --emitDeclarationOnly

# Build mode (project references)
tsc --build

# Compile with specific config
tsc -p tsconfig.prod.json

# Generate config file
tsc --init

# List all compiler options
tsc --all

# Show compiled files
tsc --listFiles

# Show file emission
tsc --listEmittedFiles
```

### Beginner Examples

```typescript
// hello.ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

const message = greet("TypeScript");
console.log(message);
```

```bash
# Compile
tsc hello.ts
# Output: hello.js

# Run
node hello.js
# Hello, TypeScript!
```

### Intermediate Examples

```typescript
// src/index.ts
export interface Config {
  port: number;
  host: string;
}

export function createServer(config: Config): void {
  console.log(`Starting server on ${config.host}:${config.port}`);
}
```

```bash
# Use --outDir to organize output
tsc --outDir dist --rootDir src --declaration --sourceMap

# Output structure:
# dist/
#   index.js
#   index.d.ts
#   index.js.map

# Watch mode for development
tsc --watch --outDir dist
```

### Advanced Examples

```bash
# Project references build
tsc --build

# With verbose output
tsc --build --verbose

# Clean build
tsc --build --clean

# Force rebuild all
tsc --build --force

# Generate declaration files separately
tsc --declaration --emitDeclarationOnly --outDir types

# Use with source maps for debugging
tsc --sourceMap --inlineSources

# Custom output file (for legacy systems)
tsc --outFile bundle.js --module system
```

### Real-World Use Cases

**CI Type Checking**: Running `tsc --noEmit` in CI pipelines catches type errors before deployment without generating output files.

```yaml
# GitHub Actions CI
- name: Type Check
  run: npx tsc --noEmit
```

**Incremental Builds**: Large projects use `tsc --build` with project references to only rebuild changed projects.

**Custom Transformers**: Advanced projects use the TypeScript compiler API to create custom transformers for code generation or instrumentation.

### Common Mistakes

**Not Using `--noEmit`**: Running `tsc` without `--noEmit` in CI generates output files that are typically handled by bundlers, causing confusion.

**Ignoring Watch Mode**: Running `tsc` manually after every change is inefficient. Use `--watch` during development.

**Confusing `--outDir` and `--outFile`**: `--outDir` preserves directory structure; `--outFile` concatenates to a single file.

### Best Practices

1. Use `tsc --noEmit` for CI type checking
2. Enable `--watch` during development for instant feedback
3. Use `--strict` mode for maximum safety
4. Configure `rootDir` and `outDir` for clean output organization
5. Enable `sourceMap` for debugging production issues
6. Use `tsc --build` for monorepos

### Performance Considerations

- First compilation: Full type checking of all files
- Subsequent compilations (watch mode): Only recheck changed files
- `incremental: true` saves type-checking results to `.tsbuildinfo`
- Project references enable parallel compilation of independent projects
- `--skipLibCheck` skips type checking of `.d.ts` files for faster compilation

### Interview Questions

1. What is the difference between `tsc` and `tsc --noEmit`?
2. How does `tsc --watch` differ from `tsc --build --watch`?
3. What is the purpose of `--declaration` and `--declarationMap`?
4. How does the `--strict` flag affect compilation?
5. Explain the difference between `--outDir` and `--outFile`.

### Coding Challenges

1. Write a batch file that compiles TypeScript, runs tests, and reports compilation time
2. Set up a watch mode compilation that outputs to a specific directory
3. Create a custom build pipeline with TypeScript, source maps, and declaration files

### Related Topics

- TypeScript compiler API
- SWC and esbuild as alternative compilers
- TypeScript configuration reference
- Babel TypeScript support

## tsconfig.json configuration

### What It Is

`tsconfig.json` is the configuration file for TypeScript projects. It specifies compiler options, file inclusion/exclusion rules, project references, and various settings that control TypeScript's behavior. It serves as the single source of truth for how TypeScript processes your code.

### Why It Is Important

Without `tsconfig.json`, `tsc` uses a minimal set of defaults. Proper configuration ensures consistent compilation across environments, enables strict type checking, and optimizes build performance. Misconfiguration is a common source of bugs and confusion.

### How It Works Internally

When `tsc` runs, it searches for `tsconfig.json` in the current directory, traversing up the directory tree until one is found. The file is parsed as JSON (with optional comments and trailing commas), validated against the TypeScript configuration schema, and merged with any command-line arguments. Configuration inheritance (`extends`) allows sharing base configurations across projects.

### Syntax

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Beginner Examples

```json
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "commonjs",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true
  },
  "include": ["src/**/*"]
}
```

### Intermediate Examples

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.spec.ts"]
}
```

### Advanced Examples

```json
// tsconfig.base.json — shared base config
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}

// packages/core/tsconfig.json — extends base
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "composite": true
  },
  "references": [
    { "path": "../utils" }
  ],
  "include": ["src/**/*"]
}

// packages/utils/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "composite": true
  },
  "include": ["src/**/*"]
}
```

### Real-World Use Cases

**Multi-environment Configuration**: Different `tsconfig` files for development (source maps, watch mode), production (minification-ready), and testing (different module system).

```json
// tsconfig.prod.json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "sourceMap": false,
    "declaration": false
  },
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.spec.ts"]
}
```

**Framework Configurations**: Next.js, Angular, and other frameworks generate their own `tsconfig.json` with framework-specific settings.

### Common Mistakes

**Not Enabling `strict`**: Without `strict: true`, many common type errors (including null checks) pass through.

**Wrong `moduleResolution`**: Using `"moduleResolution": "node"` with `"module": "ESNext"` can cause module resolution failures.

**Forgetting `resolveJsonModule`**: Trying to import `.json` files without `resolveJsonModule: true` causes compilation errors.

**Using `paths` Without `baseUrl`**: The `paths` option requires `baseUrl` to be set.

### Best Practices

1. Always start with `strict: true`
2. Use `extends` for shared configurations in monorepos
3. Set `outDir` and `rootDir` for clean output
4. Enable `esModuleInterop` for easier module imports
5. Use `skipLibCheck` for faster builds in development
6. Enable `noUnusedLocals` and `noUnusedParameters` for code quality
7. Set `forceConsistentCasingInFileNames` for cross-platform compatibility

### Performance Considerations

- `skipLibCheck: true` significantly reduces compilation time by skipping `.d.ts` file checking
- `incremental: true` stores compilation info for faster subsequent builds
- `composite: true` enables project references for parallel builds
- `isolatedModules: true` ensures each file can be transpiled independently (compatible with esbuild/SWC)

### Interview Questions

1. What does `strict: true` actually enable?
2. Explain the difference between `include` and `files` in tsconfig.json
3. What is the purpose of `extends` in tsconfig.json?
4. How does `moduleResolution` affect module imports?
5. What does `esModuleInterop` do?
6. When would you use `paths` in tsconfig.json?

### Coding Challenges

1. Create a base tsconfig.json and extend it in two sub-projects
2. Configure a project that supports both CommonJS and ESM output
3. Set up tsconfig.json for a monorepo with three interdependent packages

### Related Topics

- TypeScript compiler options reference
- Path mapping with TypeScript
- TypeScript project references
- Integrating TypeScript with Webpack, Vite, and esbuild

## VS Code integration

### What It Is

Visual Studio Code provides first-class TypeScript support through its built-in TypeScript language service. This integration includes syntax highlighting, IntelliSense, type checking on file save, code navigation, refactoring tools, and debugging — all powered by the TypeScript compiler itself.

### Why It Is Important

VS Code's TypeScript integration is one of the primary reasons developers choose TypeScript. The immediate feedback loop (errors shown in real-time), intelligent autocompletion, and powerful refactoring capabilities dramatically improve developer productivity and code quality.

### How It Works Internally

VS Code uses the TypeScript language service as a separate process (tsserver). When you open a TypeScript file, VS Code spawns `tsserver`, which:
1. Parses the project's `tsconfig.json`
2. Creates a TypeScript Program that includes all files
3. Provides real-time diagnostics, completions, and navigation
4. Sends responses via JSON-RPC protocol

The language service runs independently from the editor, allowing non-blocking operations. VS Code communicates with `tsserver` via a `TypeScriptServiceClient` that handles all requests and responses.

### Syntax

```json
// .vscode/settings.json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "typescript.preferences.quoteStyle": "single",
  "typescript.format.semicolons": "insert",
  "typescript.referencesCodeLens.enabled": true,
  "typescript.implementationsCodeLens.enabled": true,
  "typescript.suggest.completeFunctionCalls": true,
  "typescript.updateImportsOnFileMove.enabled": "always"
}
```

### Beginner Examples

```json
// .vscode/extensions.json — recommended extensions
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next"
  ]
}

// .vscode/settings.json — basic setup
{
  "typescript.validate.enable": true,
  "typescript.suggest.autoImports": true,
  "editor.formatOnSave": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### Intermediate Examples

```json
// Advanced VS Code TypeScript settings
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "typescript.surveys.enabled": false,
  "typescript.tsserver.maxTsServerMemory": 8192,
  "typescript.tsserver.log": "verbose",
  "typescript.disableAutomaticTypeAcquisition": false,
  "typescript.suggest.paths": true,
  "typescript.suggest.autoImports": true,
  "typescript.suggest.completeFunctionCalls": true,
  "typescript.format.enable": true,
  "typescript.format.insertSpaceAfterCommaDelimiter": true,
  "typescript.format.insertSpaceAfterSemicolonInForStatements": true,
  "typescript.format.insertSpaceBeforeAndAfterBinaryOperators": true,
  "typescript.format.insertSpaceAfterKeywordsInControlFlowStatements": true,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingNonemptyParenthesis": false,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingNonemptyBrackets": false,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingNonemptyBraces": false,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingTemplateStringBraces": false,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingJsxExpressionInsideBraces": false,
  "typescript.format.insertSpaceAfterTypeAssertion": false,
  "typescript.format.placeOpenBraceOnNewLineForFunctions": false,
  "typescript.format.placeOpenBraceOnNewLineForControlBlocks": false,
  "typescript.referencesCodeLens.enabled": true,
  "typescript.implementationsCodeLens.enabled": true,
  "typescript.codeLens.testReferences": true
}
```

### Advanced Examples

```typescript
// Using TypeScript snippets in VS Code
// Create .vscode/typescript.code-snippets
{
  "Functional Component": {
    "prefix": "tsfc",
    "body": [
      "import React from 'react';",
      "",
      "interface ${1:Component}Props {",
      "  ${2}",
      "}",
      "",
      "export const ${1:Component}: React.FC<${1:Component}Props> = ({${3}}) => {",
      "  return <div>${4}</div>;",
      "};",
      "",
      "export default ${1:Component};"
    ],
    "description": "React Functional Component with TypeScript"
  },
  "Interface": {
    "prefix": "tsif",
    "body": [
      "interface ${1:Name} {",
      "  ${2}",
      "}"
    ],
    "description": "TypeScript Interface"
  }
}
```

```json
// Launch configuration for TypeScript debugging
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "program": "${workspaceFolder}/src/index.ts",
      "preLaunchTask": "tsc: build - tsconfig.json",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "sourceMaps": true
    },
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Process",
      "port": 9229
    }
  ]
}
```

### Real-World Use Cases

**Team Standardization**: Teams share `.vscode/settings.json` via version control to ensure consistent editor settings across all developers.

**TypeScript Strictness Workflow**: VS Code shows errors inline as developers type, enabling strict TypeScript adoption without slowing down the development process.

**Debugging TypeScript Directly**: With source maps configured, developers can set breakpoints directly in `.ts` files in VS Code and debug in Node.js or Chrome.

### Common Mistakes

**Not Using Workspace TS Version**: VS Code may use a global TypeScript version instead of the project-local one. Use `typescript.tsdk` to point to the project's version.

**Ignoring TSServer Memory**: Large projects can cause tsserver to run out of memory. Increase `typescript.tsserver.maxTsServerMemory` if you experience crashes.

**Not Sharing Settings via .vscode**: Each developer configuring VS Code independently leads to inconsistent formatting and behavior.

### Best Practices

1. Always set `typescript.tsdk` to the project's local TypeScript
2. Share `.vscode/settings.json` via version control
3. Enable `typescript.updateImportsOnFileMove.enabled` for automatic import updates
4. Use TypeScript-specific settings for formatting over generic editor settings
5. Install the `TypeScript Next` extension for the latest features
6. Configure `.vscode/launch.json` for debugging TypeScript directly

### Performance Considerations

- TSServer memory can be a bottleneck; increase `typescript.tsserver.maxTsServerMemory` for large projects
- Disable unused features like TypeScript surveys for slightly better performance
- Use `typescript.tsserver.experimental.enableProjectDiagnostics` for larger workspaces
- If tsserver becomes slow, try `TypeScript: Restart TS Server` command

### Interview Questions

1. How does VS Code's TypeScript support work under the hood?
2. What is tsserver and what role does it play?
3. How do you force VS Code to use the project's TypeScript version?
4. What VS Code settings help with TypeScript development productivity?
5. How do source maps enable debugging TypeScript directly?

### Coding Challenges

1. Configure a VS Code workspace with three TypeScript projects and shared settings
2. Set up debugging for a TypeScript Node.js application with source maps
3. Create custom TypeScript snippets for common patterns in your framework of choice

### Related Topics

- VS Code TypeScript debugging guide
- TypeScript Language Service API
- IntelliSense customization
- ESLint integration with TypeScript
- Prettier with TypeScript
