# Compiler Internals - Compiler pipeline, parser, binder, checker, emitter, AST

## Introduction

The TypeScript compiler is a sophisticated piece of software that transforms TypeScript code into JavaScript while performing type checking, error reporting, and various code transformations. Understanding the compiler internals is essential for advanced TypeScript developers who want to write custom transpilation tools, create language service plugins, contribute to TypeScript, or simply understand how their code is processed. The compiler pipeline consists of five main phases: Parser, Binder, Checker, Emitter, and the intermediate representation known as the Abstract Syntax Tree (AST).

## Compiler Pipeline Overview

### What It Is

The TypeScript compiler pipeline is a multi-phase process that converts TypeScript source code into JavaScript output while performing type analysis and reporting errors. Each phase has a specific responsibility and produces output consumed by the next phase.

### Why It Is Important

Understanding the compiler pipeline helps developers debug compilation issues, optimize build performance, write custom transformers, and create tools that analyze or transform TypeScript code. It also demystifies how TypeScript's type system works at a fundamental level.

### How It Works Internally

The pipeline follows this sequence:

```
Source Code -> Scanner -> Parser -> AST
                                           |
Binder -> Symbol Table <-> Checker <-> Type Registry
                                           |
Emitter -> JavaScript Output + Declaration Files + Source Maps
```

Each phase: (1) Scanner tokenizes source text into tokens; (2) Parser produces AST from tokens; (3) Binder creates symbols and builds symbol tables; (4) Checker performs type checking and resolution; (5) Emitter generates output files.

### Syntax

```typescript
import * as ts from 'typescript';

function compile(sourceFiles: string[]): void {
  const program = ts.createProgram(sourceFiles, {
    target: ts.ScriptTarget.ES2020,
    module: ts.ModuleKind.CommonJS,
    strict: true,
  });

  const emitResult = program.emit();
  const diagnostics = ts.getPreEmitDiagnostics(program);

  if (diagnostics.length > 0) {
    diagnostics.forEach(diagnostic => {
      const message = ts.flattenDiagnosticMessageText(diagnostic.messageText, '\n');
      console.log(message);
    });
  }
}
```

### Beginner Examples

```typescript
import * as ts from 'typescript';

const source = `
  function greet(name: string): string {
    return 'Hello, ' + name;
  }
`;

const result = ts.transpileModule(source, {
  compilerOptions: { module: ts.ModuleKind.CommonJS, target: ts.ScriptTarget.ES2020 },
});

console.log(result.outputText);
```

### Intermediate Examples

```typescript
import * as ts from 'typescript';

const source = 'const x: number = 42;';

const result = ts.transpileModule(source, {
  compilerOptions: { module: ts.ModuleKind.CommonJS, target: ts.ScriptTarget.ES2020 },
  transformers: {
    before: [removeTypeAnnotations],
    after: [],
  },
});

function removeTypeAnnotations(context: ts.TransformationContext) {
  return (sourceFile: ts.SourceFile) => {
    const visitor = (node: ts.Node): ts.Node => {
      if (ts.isVariableStatement(node)) {
        return ts.factory.updateVariableStatement(
          node,
          node.modifiers,
          ts.factory.createVariableDeclarationList(
            node.declarationList.declarations.map(decl =>
              ts.factory.updateVariableDeclaration(
                decl, decl.name, undefined, undefined, decl.initializer
              )
            ),
            node.declarationList.flags
          )
        );
      }
      return ts.visitEachChild(node, visitor, context);
    };
    return ts.visitEachChild(sourceFile, visitor, context);
  };
}
```

### Advanced Examples

```typescript
import * as ts from 'typescript';

class CustomCompiler {
  private compilerOptions: ts.CompilerOptions;
  private host: ts.CompilerHost;

  constructor(options: ts.CompilerOptions = {}) {
    this.compilerOptions = {
      target: ts.ScriptTarget.ES2020,
      module: ts.ModuleKind.ESNext,
      strict: true,
      ...options,
    };
    this.host = ts.createCompilerHost(this.compilerOptions);
  }

  compile(fileName: string, source: string): ts.EmitResult {
    const sourceFile = ts.createSourceFile(
      fileName, source, this.compilerOptions.target || ts.ScriptTarget.Latest
    );

    const program = ts.createProgram({
      rootNames: [fileName],
      options: this.compilerOptions,
      host: {
        ...this.host,
        getSourceFile: (name) =>
          name === fileName ? sourceFile : this.host.getSourceFile(name, this.compilerOptions.target!),
        fileExists: (name) => name === fileName || this.host.fileExists(name),
      },
    });

    return program.emit();
  }
}
```

### Real-World Use Cases

```typescript
import * as ts from 'typescript';
import { performance } from 'perf_hooks';

function buildWithMetrics(rootFiles: string[], options: ts.CompilerOptions) {
  const start = performance.now();
  const program = ts.createProgram(rootFiles, options);
  const programTime = performance.now() - start;

  const typeCheckStart = performance.now();
  const diagnostics = ts.getPreEmitDiagnostics(program);
  const typeCheckTime = performance.now() - typeCheckStart;

  const emitStart = performance.now();
  const emitResult = program.emit();
  const emitTime = performance.now() - emitStart;

  console.log({
    programCreation: `${programTime.toFixed(0)}ms`,
    typeChecking: `${typeCheckTime.toFixed(0)}ms`,
    emit: `${emitTime.toFixed(0)}ms`,
    total: `${(performance.now() - start).toFixed(0)}ms`,
  });

  return { diagnostics, emitResult };
}
```

### Common Mistakes

- Modifying AST nodes in place instead of creating new ones with `ts.factory`.
- Not handling SourceFile as the root node of traversal.
- Confusing the compiler API's Node type with DOM Node.

### Best Practices

- Use `ts.createProgram` for full compilation with type checking.
- Use `ts.transpileModule` for simple transpilation without type checking.
- Use `ts.factory` API to create new AST nodes instead of constructors.
- Use `ts.visitEachChild` for recursive AST traversal.
- Use `ts.is*` type guard functions for node type checking.

### Performance Considerations

Full program creation is expensive because it parses all source files. For incremental scenarios, use `ts.createIncrementalProgram`. Custom transformers add overhead to the emit phase.

### Interview Questions

1. What are the main phases of the TypeScript compiler pipeline?
2. What is the difference between `ts.createProgram` and `ts.transpileModule`?
3. How does the TypeScript compiler handle type checking across multiple files?

### Coding Challenges

1. Write a custom transformer that removes all console.log calls from the output.
2. Create a script that uses the compiler API to count functions, interfaces, and types in a project.

### Related Topics

- Parser and AST
- Binder and symbol table
- Checker and type checking
- Emitter

## Parser and AST

### What It Is

The parser converts tokens into an Abstract Syntax Tree (AST), which is a tree representation of the source code's syntactic structure. Every TypeScript program, statement, expression, and declaration becomes an AST node.

### Why It Is Important

The AST is the foundation for all subsequent compiler phases. Understanding it is essential for writing code transformations, lint rules, code generators, and language service features.

### How It Works Internally

The parser uses a recursive descent algorithm. It starts with `parseSourceFile` and recursively calls parsing functions for each grammar production. Each function returns an AST node, which becomes a child of the parent node.

### Syntax

```typescript
import * as ts from 'typescript';

const source = `function add(a: number, b: number): number { return a + b; }`;
const sourceFile = ts.createSourceFile('example.ts', source, ts.ScriptTarget.Latest, true);

function visit(node: ts.Node, depth = 0) {
  console.log(' '.repeat(depth * 2) + ts.SyntaxKind[node.kind]);
  ts.forEachChild(node, child => visit(child, depth + 1));
}

visit(sourceFile);
```

### Beginner Examples

```typescript
function findFunctions(node: ts.Node): ts.FunctionDeclaration[] {
  const functions: ts.FunctionDeclaration[] = [];
  function visit(node: ts.Node) {
    if (ts.isFunctionDeclaration(node) && node.name) functions.push(node);
    ts.forEachChild(node, visit);
  }
  visit(node);
  return functions;
}
```

### Intermediate Examples

```typescript
function analyzeAST(node: ts.Node) {
  let declarations = 0, expressions = 0, statements = 0, types = 0;
  function visit(node: ts.Node) {
    if (ts.isDeclaration(node)) declarations++;
    if (ts.isExpression(node)) expressions++;
    if (ts.isStatement(node)) statements++;
    if (ts.isTypeNode(node)) types++;
    ts.forEachChild(node, visit);
  }
  visit(node);
  return { declarations, expressions, statements, types };
}
```

### Advanced Examples

```typescript
function addFunctionLogging(context: ts.TransformationContext) {
  return (sourceFile: ts.SourceFile) => {
    const visitor = (node: ts.Node): ts.Node => {
      if (ts.isFunctionDeclaration(node) && node.body) {
        const logStmt = ts.factory.createExpressionStatement(
          ts.factory.createCallExpression(
            ts.factory.createPropertyAccessExpression(
              ts.factory.createIdentifier('console'),
              ts.factory.createIdentifier('log')
            ), undefined,
            [ts.factory.createStringLiteral(`Entering ${node.name?.text || 'anon'}`)]
          )
        );
        return ts.factory.updateFunctionDeclaration(
          node, node.modifiers, node.asteriskToken, node.name,
          node.typeParameters, node.parameters, node.type,
          ts.factory.createBlock([logStmt, ...node.body.statements])
        );
      }
      return ts.visitEachChild(node, visitor, context);
    };
    return ts.visitEachChild(sourceFile, visitor, context);
  };
}
```

### Real-World Use Cases

```typescript
function renameInterface(source: string, oldName: string, newName: string): string {
  const sourceFile = ts.createSourceFile('temp.ts', source, ts.ScriptTarget.Latest, true);
  const transformer = (context: ts.TransformationContext) => (rootNode: ts.SourceFile) => {
    const visitor = (node: ts.Node): ts.Node => {
      if (ts.isInterfaceDeclaration(node) && node.name.text === oldName) {
        return ts.factory.updateInterfaceDeclaration(
          node, node.modifiers, ts.factory.createIdentifier(newName),
          node.typeParameters, node.heritageClauses, node.members
        );
      }
      if (ts.isIdentifier(node) && node.text === oldName && !ts.isInterfaceDeclaration(node.parent)) {
        return ts.factory.createIdentifier(newName);
      }
      return ts.visitEachChild(node, visitor, context);
    };
    return ts.visitEachChild(rootNode, visitor, context);
  };
  const result = ts.transpileModule(source, {
    compilerOptions: { target: ts.ScriptTarget.ES2020 },
    transformers: { before: [transformer] },
  });
  return result.outputText;
}
```

### Common Mistakes

- Using constructor functions instead of `ts.factory` (deprecated).
- Not setting `setParentNodes: true` when creating SourceFiles for traversal.
- Modifying AST during traversal without proper cloning.

### Best Practices

- Use `ts.createSourceFile` to parse code strings for analysis.
- Use `ts.forEachChild` for recursive traversal.
- Use `ts.is*` functions for type-safe node type checking.
- Use `ts.factory` for creating new AST nodes.

### Performance Considerations

Parsing is one of the most expensive compiler phases. Large files take significant time. TypeScript uses caching and incremental parsing to mitigate this.

### Interview Questions

1. What is an AST and why does TypeScript use one?
2. How do you traverse the TypeScript AST programmatically?

### Coding Challenges

1. Write a function that extracts all type aliases from a source file using the AST.
2. Create a code formatter that uses the AST to ensure consistent import ordering.

### Related Topics

- SyntaxKind enum
- AST node types

## Binder and Symbol Table

### What It Is

The binder creates symbols for each declaration in the AST and builds symbol tables that map names to their declarations. It handles scoping, declaration merging, and tracks relationships between declarations and their containing scopes.

### Why It Is Important

The symbol table is the foundation for name resolution and type checking. Understanding binding helps when writing tools that need to resolve names across files.

### How It Works Internally

The binder walks the AST and for each declaration node creates a `Symbol` object. Each scope has its own symbol table. The binder handles scope nesting, declaration merging, and export/import resolution.

### Syntax

```typescript
const program = ts.createProgram(['example.ts'], { strict: true });
const checker = program.getTypeChecker();

function getSymbolInfo(node: ts.Declaration) {
  const symbol = checker.getSymbolAtLocation(node.name!);
  if (symbol) {
    console.log(`Symbol: ${symbol.name}, Flags: ${symbol.flags}`);
  }
}
```

### Beginner Examples

```typescript
function analyzeExports(program: ts.Program) {
  const checker = program.getTypeChecker();
  for (const sourceFile of program.getSourceFiles()) {
    if (sourceFile.isDeclarationFile) continue;
    const moduleSymbol = checker.getSymbolAtLocation(sourceFile);
    if (!moduleSymbol) continue;
    const exports = checker.getExportsOfModule(moduleSymbol);
    console.log(`${sourceFile.fileName}: ${exports.length} exports`);
  }
}
```

### Advanced Examples

```typescript
function analyzeBindings(program: ts.Program, fileName: string) {
  const checker = program.getTypeChecker();
  const sourceFile = program.getSourceFile(fileName)!;
  const bindings = new Map<string, { declarations: ts.Declaration[]; references: ts.Identifier[] }>();

  function processDeclaration(node: ts.Declaration) {
    if (!node.name || !ts.isIdentifier(node.name)) return;
    const symbol = checker.getSymbolAtLocation(node.name);
    if (!symbol) return;
    const existing = bindings.get(symbol.name) || { declarations: [], references: [] };
    existing.declarations.push(node);
    bindings.set(symbol.name, existing);
  }

  function processIdentifier(node: ts.Identifier) {
    const symbol = checker.getSymbolAtLocation(node);
    if (!symbol) return;
    const existing = bindings.get(symbol.name);
    if (existing) existing.references.push(node);
  }

  function visit(node: ts.Node) {
    if (ts.isDeclaration(node)) processDeclaration(node);
    if (ts.isIdentifier(node)) processIdentifier(node);
    ts.forEachChild(node, visit);
  }

  visit(sourceFile);
  return bindings;
}
```

### Real-World Use Cases

```typescript
function findUnusedExports(program: ts.Program): string[] {
  const checker = program.getTypeChecker();
  const unused: string[] = [];
  for (const sourceFile of program.getSourceFiles()) {
    if (sourceFile.isDeclarationFile || sourceFile.fileName.includes('node_modules')) continue;
    const moduleSymbol = checker.getSymbolAtLocation(sourceFile);
    if (!moduleSymbol) continue;
    const exports = checker.getExportsOfModule(moduleSymbol);
    for (const exportSymbol of exports) {
      if (exportSymbol.declarations) {
        const isUsed = exportSymbol.declarations.some(decl => !ts.isSourceFile(decl.parent));
        if (!isUsed) unused.push(`${sourceFile.fileName}: ${exportSymbol.name}`);
      }
    }
  }
  return unused;
}
```

### Common Mistakes

- Confusing the type checker's symbol resolution with the binder's.
- Assuming all nodes have symbols (only declarations and identifiers).

### Best Practices

- Use `checker.getSymbolAtLocation` to get symbols from identifiers.
- Use `checker.getExportsOfModule` to get module exports.
- Use `symbol.declarations` to find merged declarations.

### Performance Considerations

The binder runs synchronously and processes all files. For large projects, binding can be expensive. TypeScript uses incremental parsing and binding to minimize re-binding.

### Interview Questions

1. What is the binder's role in the TypeScript compiler?
2. How does the binder handle interface merging?

### Coding Challenges

1. Write a function that lists all symbols in a module with their kind.
2. Create a tool that detects naming conflicts across merged declarations.

### Related Topics

- Symbol table
- Scope resolution

## Checker and Type Checking

### What It Is

The checker performs type analysis, resolves types for every expression, checks type compatibility, infers types, and reports type errors.

### Why It Is Important

The checker makes TypeScript "TypeScript." It validates type correctness and provides type information for IDE features.

### How It Works Internally

The checker uses the symbol table and AST to compute types. Key concepts include Type, Type Flow Analysis, Type Compatibility, and Contextual Typing.

### Syntax

```typescript
const program = ts.createProgram(['example.ts'], { strict: true });
const checker = program.getTypeChecker();

function printType(node: ts.Node) {
  const type = checker.getTypeAtLocation(node);
  console.log(checker.typeToString(type));
}
```

### Beginner Examples

```typescript
function areTypesCompatible(checker: ts.TypeChecker, source: ts.Type, target: ts.Type) {
  return checker.isTypeAssignableTo(source, target);
}
```

### Advanced Examples

```typescript
function findAnyUsage(program: ts.Program): ts.Node[] {
  const checker = program.getTypeChecker();
  const anyNodes: ts.Node[] = [];

  function visit(node: ts.Node) {
    if (ts.isTypeNode(node)) {
      const type = checker.getTypeAtLocation(node);
      if (type.flags & ts.TypeFlags.Any) anyNodes.push(node);
    }
    if (ts.isParameter(node) && !node.type) {
      const symbol = checker.getSymbolAtLocation(node.name);
      if (symbol) {
        const type = checker.getTypeOfSymbolAtLocation(symbol, node);
        if (type.flags & ts.TypeFlags.Any) anyNodes.push(node);
      }
    }
    ts.forEachChild(node, visit);
  }

  for (const sf of program.getSourceFiles()) {
    if (!sf.isDeclarationFile) visit(sf);
  }
  return anyNodes;
}
```

### Real-World Use Cases

```typescript
function typeCoverageReport(program: ts.Program) {
  const checker = program.getTypeChecker();
  let total = 0, untyped = 0;

  function visit(node: ts.Node) {
    if (ts.isIdentifier(node) || ts.isPropertyAccessExpression(node)) {
      total++;
      const type = checker.getTypeAtLocation(node);
      if (type.flags & ts.TypeFlags.Any || type.flags & ts.TypeFlags.Unknown) untyped++;
    }
    ts.forEachChild(node, visit);
  }

  for (const sf of program.getSourceFiles()) {
    if (!sf.isDeclarationFile && !sf.fileName.includes('node_modules')) visit(sf);
  }

  return { total, untyped, coverage: `${((total - untyped) / total * 100).toFixed(2)}%` };
}
```

### Common Mistakes

- Calling `getTypeAtLocation` on nodes without types.
- Not handling types that could be undefined.

### Performance Considerations

Type checking is the most expensive compiler phase. Incremental compilation and project references help.

### Interview Questions

1. What is the checker's primary responsibility?
2. How does the checker perform type narrowing?

### Coding Challenges

1. Write a script that identifies all implicit `any` types in a project.

### Related Topics

- Type inference
- Type compatibility

## Emitter

### What It Is

The emitter produces output files: JavaScript, declaration files (`.d.ts`), and source maps. It applies transformations like downlevel compilation and module conversion.

### Why It Is Important

The emitter makes TypeScript usable across JavaScript runtimes. Understanding it helps when debugging output or writing custom transformers.

### How It Works Internally

The emitter walks the AST and produces JavaScript code. It applies transformations (downlevel iteration, decorator lowering) and respects compiler options.

### Syntax

```typescript
const program = ts.createProgram(['input.ts'], {
  target: ts.ScriptTarget.ES5,
  module: ts.ModuleKind.CommonJS,
  declaration: true,
  sourceMap: true,
  outDir: './dist',
});

const emitResult = program.emit();
```

### Beginner Examples

```typescript
function emitASTAsJSON(sourceFile: ts.SourceFile): string {
  function serializeNode(node: ts.Node): Record<string, unknown> {
    const result: Record<string, unknown> = { kind: ts.SyntaxKind[node.kind], pos: node.pos, end: node.end };
    if (ts.isIdentifier(node)) result.text = node.text;
    const children: Record<string, unknown>[] = [];
    ts.forEachChild(node, child => children.push(serializeNode(child)));
    if (children.length > 0) result.children = children;
    return result;
  }
  return JSON.stringify(serializeNode(sourceFile), null, 2);
}
```

### Advanced Examples

```typescript
function customBuild(rootFiles: string[], options: ts.CompilerOptions) {
  const program = ts.createProgram(rootFiles, options);
  const diagnostics = ts.getPreEmitDiagnostics(program);
  if (diagnostics.length > 0) return { success: false, diagnostics };

  const transformers: ts.CustomTransformers = {
    before: [removeTypeAnnotations],
    afterDeclarations: [simplifyDeclarations],
  };

  const emitResult = program.emit(undefined, undefined, undefined, false, transformers);
  return { success: true, emitResult };
}
```

### Real-World Use Cases

Build tools use the emitter programmatically to integrate TypeScript into webpack, rollup, and other bundlers. Custom transformers enable code instrumentation, logging injection, and polyfill insertion.

### Common Mistakes

- Assuming emitter preserves comments by default (set `removeComments: false`).
- Not setting `outDir` when emitting (output goes next to source files).
- Confusing `transpileModule` (no type check) with full emit.

### Best Practices

- Use `program.emit()` for full compilation with type checking.
- Use `ts.transpileModule` for quick transpilation without type checking.
- Enable `declaration: true` for library projects.
- Use custom transformers in `before` to modify the AST before emission.

### Performance Considerations

Emission is faster than type checking. Source map generation adds overhead. Incremental compilation caches emit results.

### Interview Questions

1. What output formats does the TypeScript emitter support?
2. What is the difference between `program.emit()` and `ts.transpileModule()`?

### Coding Challenges

1. Create a custom transformer that inlines constant variables during emission.
2. Write a declaration file generator for a JavaScript library with JSDoc annotations.

### Related Topics

- Custom transformers
- Declaration files
- Source maps
