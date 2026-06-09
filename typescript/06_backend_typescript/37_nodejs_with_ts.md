# Node.js with TS - ts-node, nodemon, @types/node, project setup, tsconfig for Node

## Introduction

Node.js with TypeScript combines the runtime flexibility of Node.js with the type safety and developer tooling of TypeScript. This setup enables developers to write server-side code with interfaces, types, and modern ECMAScript features while maintaining compatibility with the Node.js runtime. Proper setup involves configuring 	s-node for execution, 
odemon for auto-restart, @types/node for Node.js API types, and an appropriate 	sconfig.json for Node.js environments.

## ts-node and nodemon Setup

### What It Is

	s-node is a TypeScript execution engine that transpiles TypeScript to JavaScript on-the-fly, allowing you to run .ts files directly without a separate compilation step. 
odemon is a utility that monitors files for changes and automatically restarts the Node.js process, providing a hot-reload development experience.

### Why It Is Important

Together, 	s-node and 
odemon provide a seamless development workflow. You write TypeScript, and the changes are instantly reflected without manual compilation steps. This dramatically improves developer productivity and reduces the edit-compile-run cycle.

### How It Works Internally

	s-node uses the TypeScript compiler API to transpile TypeScript files in memory. It registers a hook with Node.js's module loader so that .ts files are transpiled when equire() or import() is called. 
odemon watches the filesystem for changes and kills/restarts the Node.js process when a file changes.

### Syntax

`json
// package.json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/index.ts",
    "start": "ts-node src/index.ts",
    "build": "tsc",
    "serve": "node dist/index.js"
  }
}
`

### Beginner Examples

`ash
# Installation
npm install --save-dev typescript ts-node nodemon @types/node

# Basic nodemon.json configuration
`

`json
// nodemon.json
{
  "watch": ["src"],
  "ext": "ts,json",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node ./src/index.ts"
}
`

### Intermediate Examples

`json
// nodemon.json with environment and debugging
{
  "watch": ["src"],
  "ext": "ts,json,env",
  "ignore": ["src/**/*.test.ts"],
  "exec": "ts-node --transpile-only --files src/index.ts",
  "env": {
    "NODE_ENV": "development"
  },
  "delay": "2500"
}
`

### Advanced Examples

`json
// nodemon.json with swc for faster compilation
{
  "watch": ["src"],
  "ext": "ts,json",
  "exec": "node --loader ts-node/esm src/index.ts",
  "env": {
    "TS_NODE_PROJECT": "tsconfig.json"
  }
}
`

`json
// package.json with multiple environments
{
  "scripts": {
    "dev": "nodemon",
    "dev:debug": "nodemon --exec 'node --inspect --require ts-node/register src/index.ts'",
    "dev:esm": "nodemon --exec 'node --loader ts-node/esm src/index.ts'",
    "start": "node dist/index.js",
    "build": "tsc",
    "build:watch": "tsc --watch",
    "serve:prod": "node --enable-source-maps dist/index.js"
  }
}
`

### Real-World Use Cases

`	ypescript
// Production startup script with source maps
// package.json
{
  "scripts": {
    "start": "node --enable-source-maps dist/index.js",
    "dev": "nodemon --exec 'ts-node --files src/index.ts'",
    "build": "tsc --sourceMap",
    "docker:start": "node dist/index.js"
  }
}
`

### Common Mistakes

`ash
# MISTAKE: Not installing @types/node
# Results in errors for Node.js globals like Buffer, process, setTimeout

# MISTAKE: Using ts-node in production
# ts-node is slower than compiled JS. Always compile with tsc for production.

# MISTAKE: Not matching tsconfig module resolution with ts-node
# ts-node respects tsconfig.json, but ESM/CJS mismatch causes errors
`

### Best Practices

- Use 	s-node --transpile-only in development for faster execution (skips type checking).
- Use 
odemon with a delay to prevent multiple restarts when saving multiple files.
- Use swc or esbuild for even faster transpilation in development.
- Always compile with 	sc for production deployments.
- Use source maps in production for debugging.

### Performance Considerations

	s-node with --transpile-only is significantly faster than full type checking on every execution. For large projects, consider 	sx (a faster alternative using esbuild) or swc-node. Compilation to JS with 	sc produces the fastest runtime code.

### Interview Questions

1. What is the difference between 	s-node and 	sc?
2. How does 
odemon improve the development workflow?
3. Why should you avoid 	s-node in production?

### Coding Challenges

1. Set up a Node.js TypeScript project with 	s-node, 
odemon, and source maps.
2. Configure a project that supports both ESM and CJS TypeScript modules.

### Related Topics

- tsconfig for Node.js
- @types/node
- Debugging Node.js TypeScript

## @types/node Package

### What It Is

@types/node is the DefinitelyTyped package that provides TypeScript type definitions for Node.js core APIs. It includes types for built-in modules like s, path, http, crypto, stream, process, Buffer, and more.

### Why It Is Important

Without @types/node, TypeScript would not recognize Node.js-specific globals (process, Buffer, __dirname, equire) or core module imports (import * as fs from 'fs'). This package enables type checking and autocompletion for all Node.js APIs.

### How It Works Internally

The package declares global types and module declarations for Node.js namespaces. It uses TypeScript's declare module syntax to type core modules and declare var or declare namespace for globals.

### Syntax

`ash
npm install --save-dev @types/node
`

### Beginner Examples

`	ypescript
import * as fs from 'fs';
import * as path from 'path';

const data = fs.readFileSync(path.join(__dirname, 'data.json'), 'utf-8');
const parsed = JSON.parse(data);
`

### Intermediate Examples

`	ypescript
import { createServer, IncomingMessage, ServerResponse } from 'http';
import { URL } from 'url';

const server = createServer((req: IncomingMessage, res: ServerResponse) => {
  const url = new URL(req.url || '/', http://);
  const name = url.searchParams.get('name') || 'World';

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: Hello, ! }));
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
`

### Advanced Examples

`	ypescript
import { EventEmitter } from 'events';
import { promises as fs } from 'fs';
import { pipeline } from 'stream/promises';
import { createGzip } from 'zlib';
import { createReadStream, createWriteStream } from 'fs';

class FileProcessor extends EventEmitter {
  async compressFile(inputPath: string, outputPath: string): Promise<void> {
    this.emit('start', inputPath);

    try {
      await pipeline(
        createReadStream(inputPath),
        createGzip(),
        createWriteStream(outputPath)
      );
      this.emit('complete', outputPath);
    } catch (err) {
      this.emit('error', err);
    }
  }
}

// Using fs promises API
async function readConfig<T>(configPath: string): Promise<T> {
  const data = await fs.readFile(configPath, 'utf-8');
  return JSON.parse(data) as T;
}
`

### Real-World Use Cases

`	ypescript
import { randomBytes, createHash, scryptSync, timingSafeEqual } from 'crypto';

export function hashPassword(password: string): string {
  const salt = randomBytes(16).toString('hex');
  const hash = scryptSync(password, salt, 64).toString('hex');
  return ${salt}:;
}

export function verifyPassword(password: string, stored: string): boolean {
  const [salt, hash] = stored.split(':');
  const computedHash = scryptSync(password, salt, 64).toString('hex');
  return timingSafeEqual(Buffer.from(hash), Buffer.from(computedHash));
}
`

### Common Mistakes

`	ypescript
// MISTAKE: Using Node.js APIs without @types/node
// const fs = require('fs'); // No type checking

// MISTAKE: Not using the promises API for async operations
// fs.readFileSync is blocking. Use fs.promises.readFile for async.

// MISTAKE: Assuming browser APIs exist in Node.js
// document, window, localStorage are not available
`

### Best Practices

- Always install @types/node with the same major version as your Node.js version.
- Use the 
ode: protocol for importing built-in modules (import * as fs from 'node:fs').
- Prefer the promises API (s.promises) over callback-based APIs.
- Use stream/promises for pipeline operations.

### Performance Considerations

@types/node is purely type information with zero runtime overhead. The types themselves don't affect performance, but using synchronous APIs (like eadFileSync) can block the event loop.

### Interview Questions

1. What does @types/node provide that makes Node.js TypeScript development possible?
2. What is the difference between import * as fs from 'fs' and import { promises as fs } from 'fs'?
3. How do you type Node.js streams with TypeScript?

### Coding Challenges

1. Create a script that watches a directory for new files using s.watch with proper typing.
2. Build a typed wrapper around the child_process exec function with proper error handling.

### Related Topics

- Node.js API
- TypeScript modules
- Stream types

## tsconfig for Node.js Projects

### What It Is

	sconfig.json is the TypeScript configuration file that controls compiler options. For Node.js projects, specific settings are needed to match the Node.js runtime's module system, target ES version, and resolution behavior.

### Why It Is Important

An incorrectly configured 	sconfig.json produces code that Node.js cannot run. Common issues include mismatched module formats (CommonJS vs ESM), incorrect target ES version (using features Node.js doesn't support), and improper source map generation for debugging.

### How It Works Internally

TypeScript reads 	sconfig.json to determine compilation settings. The module setting controls the output module format (CommonJS for Node.js default, ESM for Node.js with "type": "module"). The 	arget setting determines the ES version of the output. The moduleResolution setting determines how modules are resolved.

### Syntax

`json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
`

### Beginner Examples

`json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
`

### Intermediate Examples

`json
// For ESM projects
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "declaration": true,
    "sourceMap": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
`

### Advanced Examples

`json
// Monorepo configuration with project references
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "composite": true,
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
`

### Real-World Use Cases

`json
// Production-optimized tsconfig
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "sourceMap": true,
    "declaration": true,
    "removeComments": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.spec.ts"]
}
`

### Common Mistakes

`json
// MISTAKE: Using target ES5 for Node.js
// Node.js 18+ supports ES2020+. Using ES5 limits language features.

// MISTAKE: Not setting esModuleInterop with CommonJS
// Without it, import * as x from 'y' doesn't work for default exports.

// MISTAKE: Mismatched module formats
// If package.json has "type": "module", use "module": "Node16"
`

### Best Practices

- Set 	arget to match your Node.js version's ES support (ES2020 for Node 14+, ES2022 for Node 18+).
- Use strict: true for full type checking.
- Enable sourceMap: true for debugging.
- Enable declaration: true for library projects.
- Use composite: true and incremental: true for faster builds in large projects.
- Set outDir: "./dist" and ootDir: "./src" for clean project structure.

### Performance Considerations

incremental: true stores compilation info to speed up subsequent builds. skipLibCheck: true skips type checking of .d.ts files, significantly reducing build time. These are recommended for all but the smallest projects.

### Interview Questions

1. What is the difference between module: "commonjs" and module: "Node16" in tsconfig?
2. Why is esModuleInterop important for Node.js projects?
3. What does strict: true enable in TypeScript?

### Coding Challenges

1. Create a tsconfig for an ESM Node.js project that uses top-level await.
2. Configure a tsconfig for a monorepo with shared types across packages.

### Related Topics

- Node.js module systems
- TypeScript compilation
- Module resolution

## Debugging Node.js TypeScript

### What It Is

Debugging Node.js TypeScript involves attaching a debugger to the running Node.js process and mapping the compiled JavaScript back to the original TypeScript source code using source maps. This enables breakpoints, step-through debugging, and variable inspection in TypeScript files.

### Why It Is Important

Debugging TypeScript directly (rather than the compiled JavaScript) is essential for productivity. Source maps bridge the gap between compiled output and source code, allowing developers to use their IDE's debugger with original TypeScript code, complete with accurate line numbers and variable names.

### How It Works Internally

Node.js has a built-in debugger that communicates via the Chrome DevTools Protocol (CDP). The --inspect flag enables the debugger. Source maps (--enable-source-maps) tell the debugger how to map compiled JS lines back to TS source lines.

### Syntax

`ash
# Debugging with ts-node
node --inspect --require ts-node/register src/index.ts

# Debugging compiled code
node --enable-source-maps --inspect dist/index.js

# Debugging with nodemon
nodemon --exec 'node --inspect --require ts-node/register src/index.ts'
`

### Beginner Examples

`json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug TypeScript",
      "type": "node",
      "request": "launch",
      "program": "/src/index.ts",
      "preLaunchTask": "tsc: build - tsconfig.json",
      "outFiles": ["/dist/**/*.js"],
      "sourceMaps": true
    }
  ]
}
`

### Intermediate Examples

`json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug with ts-node",
      "type": "node",
      "request": "launch",
      "runtimeArgs": ["--require", "ts-node/register"],
      "args": ["/src/index.ts"],
      "sourceMaps": true,
      "cwd": "",
      "protocol": "inspector",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    },
    {
      "name": "Debug with nodemon",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "nodemon",
      "args": ["--exec", "node --require ts-node/register", "/src/index.ts"],
      "sourceMaps": true,
      "restart": true,
      "console": "integratedTerminal"
    }
  ]
}
`

### Advanced Examples

`json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Attach to running process",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "restart": true,
      "sourceMaps": true,
      "outFiles": ["/dist/**/*.js"],
      "skipFiles": ["<node_internals>/**", "/node_modules/**"]
    },
    {
      "name": "Debug Mocha Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": ["--require", "ts-node/register"],
      "args": ["/node_modules/.bin/mocha", "--timeout", "999999", "--colors", "/src/**/*.test.ts"],
      "sourceMaps": true,
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
`

### Real-World Use Cases

`ash
# Debugging in production with source maps
NODE_OPTIONS="--enable-source-maps" node dist/index.js

# Chrome DevTools debugging
node --inspect-brk --require ts-node/register src/index.ts
# Then open chrome://inspect in Chrome
`

### Common Mistakes

`ash
# MISTAKE: Not generating source maps (sourceMap: false in tsconfig)
# Without source maps, the debugger shows compiled JS, not TS

# MISTAKE: Using --inspect without --inspect-brk for initial breakpoint
# --inspect-brk pauses on the first line, allowing debugger attachment

# MISTAKE: OutFiles not matching the output directory
# VSCode needs outFiles to map source locations
`

### Best Practices

- Always enable sourceMap: true in tsconfig during development.
- Use --enable-source-maps flag when running compiled Node.js.
- Use --inspect-brk for debugging startup code.
- Configure VSCode .vscode/launch.json with the correct outFiles and sourceMaps.
- Use skipFiles to skip library code and focus on application code.

### Performance Considerations

Source maps add overhead to the initial build (generating them) and to runtime debugging (mapping). They should be enabled in development but can be omitted in production for faster startup, or kept for production debugging with --enable-source-maps.

### Interview Questions

1. How do source maps enable TypeScript debugging in Node.js?
2. What is the difference between --inspect and --inspect-brk?
3. How do you configure VSCode to debug a TypeScript Node.js application?

### Coding Challenges

1. Set up a VSCode launch configuration for debugging TypeScript with ts-node.
2. Configure a debug session for a Node.js Express app with automatic restart on file changes.

### Related Topics

- tsconfig for Node.js
- Source maps
- Chrome DevTools debugging
