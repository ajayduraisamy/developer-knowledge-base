# Setup Environment - Browser console, Node.js, VS Code, devtools

## Introduction
Setting up a proper JavaScript development environment is the first step to becoming productive. Modern JavaScript development relies on a browser with developer tools, Node.js for server-side execution and package management, and a code editor like VS Code. This guide covers each component in depth.

## Browser console

### What It Is
The browser console is a built-in developer tool in web browsers that provides a JavaScript REPL (Read-Eval-Print Loop) environment. It allows developers to execute JavaScript code, inspect DOM elements, view logs, profile performance, and debug applications in real-time.

### Why It Is Important
The console is the fastest way to test JavaScript snippets, debug issues, inspect network requests, and understand how code behaves in the browser. It is an essential tool for every web developer, providing immediate feedback without needing to set up a full development environment.

### How It Works Internally
The console is part of the browser's DevTools, which runs in a separate process from the web page. When you type code in the console, it gets evaluated by the JavaScript engine (V8 in Chrome) in the context of the current page's global scope. The console API (`console.log`, `console.error`, etc.) is implemented as a Web API that the browser exposes to JavaScript.

### Syntax
```javascript
// Basic console methods
console.log("Standard log message");
console.info("Informational message");
console.warn("Warning message");
console.error("Error message");
console.debug("Debug message");

// Advanced console methods
console.table([{ name: "Alice", age: 30 }, { name: "Bob", age: 25 }]);
console.time("timerLabel");
console.timeEnd("timerLabel");
console.group("Group Label");
console.groupEnd();
console.trace("Trace label");
console.count("Counter");
console.clear();
```

### Beginner Examples
```javascript
// Opening the console
// Chrome: F12 or Ctrl+Shift+I (Windows/Linux) | Cmd+Option+I (Mac)

// Simple logging
let x = 10;
let y = 20;
console.log("The sum is:", x + y);

// Inspecting objects
const user = { name: "John", age: 30, address: { city: "NYC" } };
console.log(user);
console.dir(user); // Shows interactive property list

// Evaluating expressions
// Type directly in console:
// document.title
// window.innerWidth
// 2 + 2 === 4
```

### Intermediate Examples
```javascript
// Performance measurement
console.time("fetchData");
fetch("https://jsonplaceholder.typicode.com/todos/1")
  .then(res => res.json())
  .then(data => {
    console.timeEnd("fetchData");
    console.log(data);
  });

// Styling console output
console.log("%cStyled message", "color: blue; font-size: 20px; font-weight: bold;");

// Conditional logging
const debug = true;
debug && console.log("Debug info:", someVariable);

// Grouping logs
console.group("User Details");
console.log("Name: Alice");
console.log("Age: 30");
console.groupCollapsed("Address");
console.log("Street: 123 Main St");
console.log("City: Wonderland");
console.groupEnd();
console.groupEnd();
```

### Advanced Examples
```javascript
// Console with custom formatters (Chrome)
class CustomFormatter {
  header(obj) {
    if (obj instanceof CustomClass) {
      return ["div", {}, `CustomClass: ${obj.value}`];
    }
    return null;
  }
  hasBody() { return true; }
  body(obj) {
    return ["div", {}, JSON.stringify(obj.details)];
  }
}
// Register with: devtoolsFormatters

// Async console debugging
async function debugAsync() {
  console.log("Start");
  await new Promise(r => setTimeout(r, 1000));
  console.log("After 1s");
  // Breakpoints in Sources tab work with async code
}

// Memory profiling with console
const leaks = [];
for (let i = 0; i < 100000; i++) {
  leaks.push(new Array(1000).fill("*"));
}
console.log("Check Memory tab for heap snapshot");
```

### Real-World Use Cases
- Debugging API responses by logging fetch results
- Inspecting DOM element properties during development
- Profiling function execution times
- Monitoring event handler execution
- Testing CSS selectors with `document.querySelector`
- Debugging WebSocket messages
- Analyzing network request waterfalls

### Common Mistakes
- Leaving `console.log` statements in production code
- Logging sensitive data (passwords, tokens)
- Overusing console.log instead of breakpoints
- Expecting console.log of objects to show current state (objects are live references)
- Using console.log in performance-critical code paths

### Best Practices
- Use `console.warn` and `console.error` appropriately
- Remove console statements before committing (use ESLint `no-console` rule in production)
- Use structured logging with `console.table` and `console.group`
- Use `debugger` statement or breakpoints instead of log-only debugging
- Use `console.time` / `console.timeEnd` for performance analysis

### Performance Considerations
- `console.log` blocks the main thread in some browsers
- Heavy logging in loops can freeze the UI
- Use conditional logging with flags to enable/disable in production
- `console.table` is slower than `console.log` for large datasets
- Persistent console logs can cause memory retention

### Interview Questions
1. What is the difference between `console.log` and `console.dir`?
2. How can you style console output?
3. What is the purpose of `console.time` and `console.timeEnd`?
4. How does `console.trace` help in debugging?
5. Can you use the console to profile memory?
6. How do you preserve logs across page navigations?

### Coding Challenges
1. Create a custom console wrapper that prefixes all logs with timestamps.
2. Write a function that logs only in development mode.
3. Build a simple console-based task timer.
4. Implement a logger that supports log levels (info, warn, error).

### Related Topics
- Browser DevTools
- Debugger statement
- Source maps
- Breakpoints

## Node.js setup

### What It Is
Node.js is an open-source, cross-platform JavaScript runtime built on V8 that executes JavaScript outside the browser. It uses an event-driven, non-blocking I/O model, making it efficient for building scalable network applications. Node.js also includes npm, the largest package ecosystem.

### Why It Is Important
Node.js enables JavaScript to be used for server-side development, CLI tools, build systems, and more. It is essential for modern front-end tooling (Webpack, Vite) and backend development. Understanding Node.js setup is foundational for any JavaScript developer.

### How It Works Internally
Node.js combines the V8 JavaScript engine with libuv, a C library that provides the event loop, thread pool, and async I/O operations. JavaScript code runs in a single thread, while libuv handles file system operations, networking, and DNS lookups via its thread pool. The event loop (phases: timers, I/O callbacks, idle/prepare, poll, check, close callbacks) processes callbacks in order.

### Syntax
```javascript
// Running Node.js scripts from terminal
// node script.js
// node --version
// node -e "console.log('inline script')"

// CommonJS modules (default)
const fs = require('fs');
const path = require('path');

// ES modules (with "type": "module" in package.json)
import fs from 'fs';
import path from 'path';
```

### Beginner Examples
```javascript
// Hello World HTTP server
const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(3000, '127.0.0.1', () => {
  console.log('Server running at http://127.0.0.1:3000/');
});

// Reading a file
const fs = require('fs');
fs.readFile('example.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

### Intermediate Examples
```javascript
// Using npm for package management
// npm init -y (creates package.json)
// npm install express
// npm install -D nodemon

// Express server
const express = require('express');
const app = express();
app.use(express.json());

app.get('/api/hello', (req, res) => {
  res.json({ message: 'Hello from Express!' });
});

app.post('/api/data', (req, res) => {
  console.log(req.body);
  res.status(201).json({ received: true });
});

app.listen(3001);

// Environment variables
require('dotenv').config();
console.log(process.env.NODE_ENV); // 'development'
console.log(process.env.PORT); // '3000'

// File system with promises
const fs = require('fs/promises');
async function readConfig() {
  try {
    const data = await fs.readFile('./config.json', 'utf8');
    return JSON.parse(data);
  } catch (err) {
    console.error('Config not found, using defaults');
    return { port: 3000 };
  }
}
```

### Advanced Examples
```javascript
// Child processes for parallel execution
const { spawn } = require('child_process');
const child = spawn('python', ['script.py']);
child.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});
child.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});

// Cluster module for multi-core
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);
  console.log(`Worker ${process.pid} started`);
}

// Streams for large data
const { createReadStream, createWriteStream } = require('fs');
const { Transform } = require('stream');

const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

createReadStream('input.txt')
  .pipe(upperCaseTransform)
  .pipe(createWriteStream('output.txt'))
  .on('finish', () => console.log('Done'));
```

### Real-World Use Cases
- Building REST APIs and GraphQL servers
- Creating real-time applications (chat, gaming) with WebSockets
- Building CLI tools (npm packages, scaffolding)
- Running build tools and task runners (Webpack, Gulp)
- Server-side rendering (Next.js, Nuxt)
- Microservices architecture
- Proxy servers and API gateways
- Automation scripts and data processing pipelines

### Common Mistakes
- Blocking the event loop with synchronous operations (e.g., using `fs.readFileSync` in a server)
- Not handling uncaught exceptions and promise rejections
- Forgetting to call `next()` in Express middleware
- Using global state that gets shared across requests
- Not setting `NODE_ENV=production` in production
- Running CPU-intensive tasks without worker threads

### Best Practices
- Always handle errors with proper error boundaries
- Use `process.on('uncaughtException')` and `process.on('unhandledRejection')`
- Prefer asynchronous APIs over synchronous ones
- Use environment variables for configuration
- Implement graceful shutdown (SIGTERM handler)
- Use npm scripts for common tasks
- Pin dependency versions in package.json
- Use `.env` files for local development

### Performance Considerations
- Node.js is single-threaded; use worker threads for CPU-bound tasks
- Stream data instead of buffering large files in memory
- Use connection pooling for databases
- Enable gzip compression for HTTP responses
- Cluster mode for multi-core utilization
- Profile memory usage and look for leaks
- Use `--inspect` flag for debugging performance issues

### Interview Questions
1. How does the Node.js event loop work?
2. What is the difference between CommonJS and ES modules?
3. How do you handle uncaught exceptions in Node.js?
4. Explain the cluster module and when to use it.
5. What are streams and why are they important?
6. How does npm handle dependency resolution?
7. What is the purpose of `package-lock.json`?
8. Explain the libuv library's role in Node.js.

### Coding Challenges
1. Create a CLI tool that reads a CSV file and outputs JSON.
2. Build a file watcher that logs file changes.
3. Implement a simple HTTP router without Express.
4. Create a program that spawns a child process and communicates via IPC.
5. Build a streaming file compression/decompression tool.

### Related Topics
- npm and yarn package managers
- CommonJS vs ES modules
- Event loop and libuv
- Node.js streams
- Worker threads

## VS Code

### What It Is
Visual Studio Code (VS Code) is a free, open-source code editor developed by Microsoft. It is the most popular editor for JavaScript development due to its extensive extension ecosystem, built-in debugging, IntelliSense, Git integration, and terminal support.

### Why It Is Important
A good editor dramatically improves developer productivity. VS Code provides intelligent code completion, real-time error detection, refactoring tools, and seamless debugging, all of which are essential for modern JavaScript development.

### How It Works Internally
VS Code is built on Electron (Chromium + Node.js). The editor UI is rendered with web technologies. Extensions run in separate processes (extension hosts). Language services (IntelliSense) communicate via the Language Server Protocol (LSP). Debugging uses the Debug Adapter Protocol (DAP). Git integration spawns child processes for git commands.

### Syntax
```javascript
// VS Code settings.json (Ctrl+,)
{
  "editor.fontSize": 14,
  "editor.tabSize": 2,
  "files.autoSave": "onFocusChange",
  "javascript.suggest.autoImports": true,
  "editor.formatOnSave": true
}

// .vscode/launch.json for debugging
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/app.js"
    }
  ]
}

// .vscode/tasks.json for build tasks
{
  "version": "2.0.0",
  "tasks": [{
    "label": "Run Tests",
    "type": "shell",
    "command": "npm test",
    "group": "test"
  }]
}
```

### Beginner Examples
```javascript
// Essential VS Code Extensions for JavaScript
// 1. ESLint (dbaeumer.vscode-eslint)
// 2. Prettier (esbenp.prettier-vscode)
// 3. JavaScript (ES6) code snippets (xabikos.javascript)
// 4. Path Intellisense (christian-kohler.path-intellisense)
// 5. Live Server (ritwickdey.LiveServer)
// 6. npm Intellisense (christian-kohler.npm-intellisense)
// 7. GitHub Copilot (GitHub.copilot)

// Keyboard shortcuts
// Ctrl+P: Quick open file
// Ctrl+Shift+P: Command palette
// Ctrl+D: Select next occurrence
// Ctrl+/: Toggle comment
// Alt+Up/Down: Move line
// Ctrl+Shift+L: Select all occurrences
// F5: Start debugging
```

### Intermediate Examples
```javascript
// Multi-cursor editing
// 1. Place cursor
// 2. Ctrl+Alt+Down to add cursors below
// 3. Type to edit all at once

// Snippets (create .vscode/javascript.code-snippets)
{
  "Console Log": {
    "prefix": "cl",
    "body": ["console.log('$1');", "$2"],
    "description": "Log to console"
  },
  "Arrow Function": {
    "prefix": "af",
    "body": ["const ${1:name} = (${2:params}) => {", "\t${3}", "};"],
    "description": "Arrow function expression"
  }
}

// Debugging configuration for Node.js
// Add breakpoint by clicking gutter (F9)
// Step over: F10
// Step into: F11
// Step out: Shift+F11
// Continue: F5
// Watch variables in left panel
// Debug console for REPL during break
```

### Advanced Examples
```javascript
// Launch configurations for different scenarios
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Current File",
      "program": "${file}",
      "runtimeArgs": ["--experimental-modules"]
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Jest Tests",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["--runInBand", "--testPathPattern=${fileBasename}"],
      "console": "integratedTerminal"
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug in Chrome",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/src"
    },
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Process",
      "port": 9229
    }
  ]
}

// Tasks for automation
// .vscode/tasks.json with compound tasks
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Start Dev Server",
      "type": "shell",
      "command": "npm run dev",
      "isBackground": true,
      "problemMatcher": []
    },
    {
      "label": "Run Linter",
      "type": "shell",
      "command": "npm run lint"
    }
  ]
}
```

### Real-World Use Cases
- Writing and debugging Node.js applications
- Developing front-end frameworks (React, Angular, Vue)
- Collaborative development with Live Share extension
- Remote development via SSH or containers
- Jupyter notebook support for data science
- Docker integration for containerized apps
- Database management with SQL extensions
- API testing with Thunder Client or REST Client

### Common Mistakes
- Not using source control integration
- Ignoring error squiggles from ESLint
- Modifying node_modules content
- Over-relying on extensions, causing slow startup
- Not customizing workspace settings per project
- Forgetting to save before debugging

### Best Practices
- Use workspace settings (`.vscode/settings.json`) per project
- Set up a consistent formatter (Prettier with `formatOnSave`)
- Configure ESLint to auto-fix on save
- Use integrated terminal instead of external terminals
- Learn keyboard shortcuts progressively
- Use GitLens for enhanced Git visualization
- Enable auto-save to prevent data loss

### Performance Considerations
- Disable unused extensions to reduce memory usage
- Exclude large directories (node_modules, dist) from file watcher
- Use `files.watcherExclude` in settings
- Consider using the "Remote" extensions for heavy projects
- Use `search.exclude` and `files.exclude` for ignored paths
- For large projects, use TypeScript project references

### Interview Questions
1. What is the Language Server Protocol and how does VS Code use it?
2. How does VS Code's debugging work?
3. What is the difference between workspace and user settings?
4. How do you create custom snippets in VS Code?
5. What is the Debug Adapter Protocol?

### Coding Challenges
1. Create a VS Code snippet for a React functional component.
2. Write a launch configuration to debug a Mocha test suite.
3. Configure Prettier and ESLint to work together.
4. Create a multi-root workspace for a monorepo.

### Related Topics
- Language Server Protocol (LSP)
- Debug Adapter Protocol (DAP)
- Prettier and ESLint
- Git integration

## DevTools

### What It Is
Browser Developer Tools (DevTools) are a suite of debugging and profiling tools built into modern browsers. They include elements inspection, console, sources/ debugger, network monitoring, performance profiler, memory analyzer, application storage viewer, and Lighthouse audits.

### Why It Is Important
DevTools are essential for understanding how web pages work, diagnosing layout issues, optimizing performance, debugging JavaScript, analyzing network requests, and measuring accessibility. Professional developers spend significant time in DevTools.

### How It Works Internally
DevTools run in a separate process from the web page to avoid interference. They communicate with the browser's internal APIs via the Chrome DevTools Protocol (CDP). The Elements tab walks the DOM tree; the Sources tab maps to V8's debugging interface; the Performance tab uses the V8 profiler and browser tracing; the Network tab intercepts HTTP traffic at the network stack level.

### Syntax
```javascript
// Elements tab
// Inspect HTML structure, modify CSS in real-time
// $0 references the currently selected element in console
// $1, $2, $3 are previously selected elements

// Console shortcut references
$('selector');       // document.querySelector()
$$('selector');      // document.querySelectorAll()
$x('//xpath');       // XPath evaluation

// Copying from console
copy(someObject);    // Copies to clipboard

// Monitoring events
monitorEvents(window, 'resize');
unmonitorEvents(window);
```

### Beginner Examples
```javascript
// Opening DevTools
// Chrome: F12 or Ctrl+Shift+I (Windows/Linux) | Cmd+Option+I (Mac)

// Inspecting elements
// 1. Right-click element > Inspect
// 2. Elements tab shows DOM tree
// 3. Styles pane shows computed CSS
// 4. Modify CSS directly in Styles pane

// Console basics
// Type: document.title to see page title
// Type: window.scrollY to see scroll position
// Type: typeof navigator to see type

// Network tab
// Reload page with Network tab open
// See waterfall of requests
// Filter by XHR, JS, CSS, Img, etc.
// Click a request to see headers, response, timing

// Sources tab
// Find JavaScript files in the file tree
// Click line number to set breakpoint
// Hover variables to see current value
// Use Watch panel to track expressions
```

### Intermediate Examples
```javascript
// Using breakpoints effectively
// Regular breakpoint: Click line number
// Conditional breakpoint: Right-click line number
// DOM breakpoint: Right-click element > Break on > subtree modifications
// XHR breakpoint: Sources > XHR Breakpoints > Add URL pattern
// Event listener breakpoint: Sources > Event Listener Breakpoints

// Snippets in DevTools
// Sources > Snippets > New snippet
// Useful for reusable debugging scripts
(function() {
  console.clear();
  console.log('All links on page:');
  document.querySelectorAll('a').forEach(a => {
    console.log(`${a.textContent.trim()}: ${a.href}`);
  });
})();

// Console utilities
// $() - alias for document.querySelector
// $$() - alias for document.querySelectorAll
// $x() - XPath selector
// inspect(obj) - switches to Elements tab and selects element
// getEventListeners(element) - shows bound event listeners
// keys(obj) - returns property names of object
// values(obj) - returns values of object
```

### Advanced Examples
```javascript
// Performance profiling
// 1. Performance tab > Record button
// 2. Interact with the page
// 3. Stop recording
// 4. Analyze the flame chart
// Look for: Long tasks (red), Layout shifts, FPS drops
// Identify: Forced reflows, memory leaks, jank

// Memory analysis
// Memory tab > Heap snapshot > Take snapshot
// Compare snapshots to find memory leaks
// Allocation instrumentation timeline to see allocations over time
// Look for detached DOM trees (elements removed but still referenced)

// Lighthouse audits
// Lighthouse tab > Generate report
// Scores for: Performance, Accessibility, Best Practices, SEO, PWA
// Get actionable recommendations

// Chrome DevTools Protocol (CDP) usage
// Used by Puppeteer, Playwright, Selenium
// const { ChromeLauncher } = require('chrome-launcher');
// Access via WebSocket at chrome://inspect
```

### Real-World Use Cases
- Debugging layout issues with Flexbox/Grid inspectors
- Tracking down memory leaks with heap snapshots
- Analyzing Core Web Vitals (LCP, FID, CLS)
- Debugging network request waterfalls
- Testing responsive designs with device emulation
- Auditing accessibility issues
- Inspecting WebSocket communication
- Debugging service workers and cache storage
- Analyzing CSS specificity conflicts

### Common Mistakes
- Not using conditional breakpoints, adding manual console.log instead
- Modifying CSS in DevTools and forgetting to copy changes to source files
- Not understanding the difference between :hover states and actual styles
- Relying solely on console.log when breakpoints are more effective
- Ignoring performance panel when debugging jank
- Not clearing console between debugging sessions

### Best Practices
- Use `debugger;` statement in code to trigger breakpoints
- Set conditional breakpoints instead of adding if-blocks
- Use the Network panel's "Disable cache" checkbox when debugging
- Use the Sources panel's "Overrides" feature to test persistent changes
- Take heap snapshots before and after suspect operations to find leaks
- Use the Coverage tab to find unused CSS/JS code
- Use the Console's "Preserve log" checkbox for navigation debugging

### Performance Considerations
- Performance tab recording adds overhead; stop recording promptly
- Heap snapshots can be large; take targeted snapshots
- Network throttling in DevTools simulates slow connections accurately
- CPU throttling helps test on slower devices
- The Performance Monitor (real-time metrics) is low-overhead
- Animations tab shows FPS and GPU usage

### Interview Questions
1. What is the difference between a regular breakpoint and a conditional breakpoint?
2. How do you debug memory leaks using DevTools?
3. How does the Network tab help in optimizing page load?
4. What is the Coverage tab and how is it useful?
5. Explain how you would debug a layout shift issue.
6. What is the Chrome DevTools Protocol?

### Coding Challenges
1. Use DevTools to find and fix a memory leak in a sample application.
2. Record a performance profile and identify the longest task.
3. Use the Coverage tab to find unused CSS rules.
4. Debug an XHR/fetch request by intercepting it with a breakpoint.

### Related Topics
- Chrome DevTools Protocol (CDP)
- Puppeteer and Playwright
- Web performance optimization
- Memory management in browsers
