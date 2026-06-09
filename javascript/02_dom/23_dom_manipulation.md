# DOM Manipulation - innerHTML, textContent, createElement, appendChild, classList

## Introduction

DOM manipulation is the process of programmatically changing the structure, content, and styling of web pages. It encompasses creating new elements, modifying existing content, adjusting visual presentation through CSS classes, and managing the hierarchical relationships between nodes. Mastering DOM manipulation is essential for building dynamic, interactive web applications that respond to user actions and data changes.

Modern web development frameworks like React, Vue, and Angular abstract away direct DOM manipulation through virtual DOM and reactive data binding, but they all ultimately compile to DOM manipulation operations. Understanding the underlying DOM APIs enables developers to debug framework issues, optimize performance, and work effectively without framework dependencies.

## innerHTML and textContent

### What It Is

`innerHTML` is a property on `Element` nodes that gets or sets the HTML markup contained within an element. When set, the browser parses the provided HTML string, constructs a DOM subtree, and replaces the element's existing content. `textContent` is a property on `Node` that gets or sets the textual content of a node and all its descendants, treating the content as plain text rather than HTML.

```javascript
// innerHTML - parses HTML
element.innerHTML = '<strong>Bold text</strong>';

// textContent - treats as plain text
element.textContent = '<strong>Not bold text</strong>'; // Displays "<strong>..." literally
```

### Why It Is Important

`innerHTML` provides a convenient way to render HTML content from strings, making it useful for templating, rendering API responses that contain HTML, and dynamic content insertion. `textContent` is essential for safely injecting user-generated content without XSS (Cross-Site Scripting) vulnerabilities, and it is also the most performant way to read or write text content because it bypasses HTML parsing.

### How It Works Internally

When `innerHTML` is set, the browser:
1. Removes existing child nodes
2. Parses the HTML string into tokens
3. Constructs a DOM fragment from the tokens
4. Inserts the fragment as children of the element

When `innerHTML` is read, the browser serializes the element's child nodes back into an HTML string, which involves tree traversal and stringification.

`textContent` simply concatenates all `Text` node values in the subtree. Setting it replaces all child nodes with a single `Text` node.

```javascript
// Internal behavior of textContent setter (conceptual)
// set textContent(value) {
//   this.removeChild(this.firstChild);
//   const textNode = document.createTextNode(String(value));
//   this.appendChild(textNode);
// }
```

### Syntax

```javascript
// Getting/setting innerHTML
const htmlContent = element.innerHTML;
element.innerHTML = '<p>New HTML content</p>';

// Getting/setting textContent
const text = element.textContent;
element.textContent = 'Plain text content';
```

### Beginner Examples

```javascript
// Example 1: Setting innerHTML
const container = document.getElementById('content');
container.innerHTML = '<h1>Welcome</h1><p>This is dynamic content</p>';

// Example 2: Safe text insertion
const userComment = '<script>alert("xss")</script>';
const commentDiv = document.getElementById('comment');
commentDiv.textContent = userComment; // Shows the script tag as text, not executed

// Example 3: Reading text content
const article = document.querySelector('article');
const articleText = article.textContent;
console.log(articleText); // All text without HTML tags

// Example 4: Clearing content
element.innerHTML = ''; // Clear all child nodes
element.textContent = ''; // Same result, slightly faster

// Example 5: Appending HTML
element.innerHTML += '<li>New item</li>';
```

### Intermediate Examples

```javascript
// Example 1: Safe HTML templating
function renderTemplate(data) {
  return `
    <div class="card">
      <h2>${escapeHtml(data.title)}</h2>
      <p>${escapeHtml(data.description)}</p>
      <span class="meta">${escapeHtml(data.date)}</span>
    </div>
  `;
}

function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

document.getElementById('cards').innerHTML = data.map(renderTemplate).join('');

// Example 2: Extracting text for search indexing
function getSearchableText(root) {
  // Get clean text, script/style content excluded
  const clone = root.cloneNode(true);
  clone.querySelectorAll('script, style, noscript').forEach(el => el.remove());
  return clone.textContent.replace(/\s+/g, ' ').trim();
}

// Example 3: Measuring content size
function getContentStats(element) {
  return {
    htmlLength: element.innerHTML.length,
    textLength: element.textContent.length,
    charCount: element.textContent.replace(/\s/g, '').length,
    wordCount: element.textContent.split(/\s+/).filter(Boolean).length,
    childElements: element.children.length
  };
}

// Example 4: Progressive content rendering
function progressiveRender(container, items, batchSize = 50) {
  let index = 0;

  function renderNextBatch() {
    const fragment = document.createDocumentFragment();
    const batch = items.slice(index, index + batchSize);

    for (const item of batch) {
      const div = document.createElement('div');
      div.textContent = item.name;
      fragment.appendChild(div);
    }

    container.appendChild(fragment);
    index += batchSize;

    if (index < items.length) {
      requestAnimationFrame(renderNextBatch);
    }
  }

  renderNextBatch();
}
```

### Advanced Examples

```javascript
// Example 1: HTML sanitizer using innerHTML
function sanitizeHTML(dirty) {
  // Create a template element that cannot execute scripts
  const template = document.createElement('template');
  template.innerHTML = dirty;

  // Remove all script and event handlers
  const walker = document.createTreeWalker(
    template.content,
    NodeFilter.SHOW_ELEMENT
  );

  let node;
  while (node = walker.nextNode()) {
    // Remove unsafe tags
    if (['SCRIPT', 'STYLE', 'IFRAME', 'OBJECT', 'EMBED'].includes(node.tagName)) {
      node.remove();
      continue;
    }

    // Remove event handler attributes
    const attrs = Array.from(node.attributes);
    for (const attr of attrs) {
      if (attr.name.startsWith('on') || attr.name === 'href' && attr.value.startsWith('javascript:')) {
        node.removeAttribute(attr.name);
      }
    }
  }

  return template.innerHTML;
}

// Example 2: innerHTML-based template engine
class TemplateEngine {
  constructor() {
    this.cache = new Map();
  }

  compile(template) {
    if (this.cache.has(template)) {
      return this.cache.get(template);
    }

    const fn = new Function('data', `
      const parts = [];
      ${this.parse(template)}
      return parts.join('');
    `);

    this.cache.set(template, fn);
    return fn;
  }

  parse(template) {
    let code = 'let $ = data;';
    let buffer = '';
    let i = 0;

    while (i < template.length) {
      if (template[i] === '{' && template[i + 1] === '{') {
        if (buffer) {
          code += `parts.push(${JSON.stringify(buffer)});`;
          buffer = '';
        }
        i += 2;
        const end = template.indexOf('}}', i);
        const expr = template.slice(i, end).trim();
        code += `parts.push($.${expr});`;
        i = end + 2;
      } else {
        buffer += template[i];
        i++;
      }
    }

    if (buffer) {
      code += `parts.push(${JSON.stringify(buffer)});`;
    }

    return code;
  }

  render(template, data) {
    const fn = this.compile(template);
    return fn(data);
  }
}

const engine = new TemplateEngine();
const html = engine.render(
  '<h1>{{title}}</h1><p>{{description}}</p>',
  { title: 'Hello', description: 'World' }
);
document.body.innerHTML = html;

// Example 3: Observing innerHTML changes
function watchElement(element) {
  const originalDescriptor = Object.getOwnPropertyDescriptor(
    Element.prototype,
    'innerHTML'
  );

  Object.defineProperty(element, 'innerHTML', {
    get() {
      return originalDescriptor.get.call(this);
    },
    set(value) {
      console.log('innerHTML changed:', value.substring(0, 100) + '...');
      console.trace('Stack trace:');
      return originalDescriptor.set.call(this, value);
    }
  });
}

const target = document.getElementById('watched');
watchElement(target);

// Example 4: Structural diff via innerHTML comparison
function detectDOMChanges(element, pollInterval = 1000) {
  let lastHTML = element.innerHTML;

  return setInterval(() => {
    const currentHTML = element.innerHTML;
    if (currentHTML !== lastHTML) {
      console.log('DOM changed');
      console.log('Previous length:', lastHTML.length);
      console.log('Current length:', currentHTML.length);
      lastHTML = currentHTML;
    }
  }, pollInterval);
}

const watcher = detectDOMChanges(document.getElementById('dynamic-content'));
```

### Real-World Use Cases

- **Rich Text Editors**: Quill, TinyMCE use innerHTML to render and read editor content
- **Server-Side Rendering**: Hydrating server-rendered HTML into interactive components
- **Content Management**: Rendering CMS content stored as HTML strings
- **Email Clients**: Rendering HTML email content safely
- **Template Systems**: Rendering template strings with data bindings
- **Document Preview**: Generating document preview from markup

### Common Mistakes

```javascript
// Mistake 1: XSS vulnerability with innerHTML
const userInput = '<img src=x onerror=alert(1)>';
element.innerHTML = userInput; // Executes the script!

// Mistake 2: Performance issue with += innerHTML
list.innerHTML += '<li>Item</li>'; // Serializes entire content, appends, re-parses
// Better: Use insertAdjacentHTML or createElement

// Mistake 3: Forgetting innerHTML returns the current state (includes changes)
element.innerHTML = '<p>First</p>';
element.innerHTML = '<p>Second</p>';
console.log(element.innerHTML); // '<p>Second</p>'

// Mistake 4: Confusing textContent with innerText
element.textContent; // All text, includes hidden text, no layout cost
element.innerText; // Visible text only, triggers layout, slower
```

### Best Practices

- Always use `textContent` for user-generated or untrusted content
- Use `insertAdjacentHTML` instead of `innerHTML +=` for better performance
- Sanitize HTML before using `innerHTML` with any external content
- Prefer `textContent` over `innerText` for reading text (faster, no layout trigger)
- Use `innerHTML` for initial render, then manipulate specific parts for updates
- Consider `template` literals with proper escaping for HTML construction
- Avoid `innerHTML` in loops; build fragments or arrays and set once

### Performance Considerations

`textContent` is significantly faster than `innerHTML` because it doesn't invoke the HTML parser. Setting `innerHTML` destroys all existing child nodes and creates new ones, which is O(n) for parsing and O(m) for node creation. Reading `innerHTML` involves serialization which is also O(n).

```javascript
// Fastest: textContent for plain text
element.textContent = 'Text';

// 2-3x slower: innerHTML for the same text
element.innerHTML = 'Text';

// Worst: innerHTML += (serialize + parse entire content)
element.innerHTML += '<p>New</p>'; // Destroys and recreates all existing nodes

// Better: insertAdjacentHTML (doesn't destroy existing nodes)
element.insertAdjacentHTML('beforeend', '<p>New</p>');
```

### Interview Questions

1. What is the difference between `innerHTML` and `textContent`?
2. Why is `innerHTML` dangerous with user input?
3. How would you sanitize HTML before using `innerHTML`?
4. What is the difference between `textContent` and `innerText`?
5. How does `insertAdjacentHTML` differ from `innerHTML`?
6. What happens to event listeners on children when you set `innerHTML`?
7. Why is `+=` with `innerHTML` a performance anti-pattern?
8. How does the browser parse HTML in `innerHTML`?

### Coding Challenges

1. **HTML Sanitizer**: Build a whitelist-based HTML sanitizer
2. **Template Engine**: Implement a simple template engine using innerHTML
3. **Content Extractor**: Extract clean text content from HTML, removing scripts and styles
4. **HTML Diff**: Compare two HTML strings and report structural differences
5. **Safe Renderer**: Create a function that renders data safely using textContent

### Related Topics

- [insertAdjacentHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML)
- [DOMParser](https://developer.mozilla.org/en-US/docs/Web/API/DOMParser)
- [XMLSerializer](https://developer.mozilla.org/en-US/docs/Web/API/XMLSerializer)
- [XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

## createElement() and appendChild()

### What It Is

`document.createElement()` creates a new HTML element node with the specified tag name. The created element is detached from the DOM (not yet visible) and can be configured before insertion. `appendChild()` inserts a node as the last child of a parent element. Together, they form the programmatic way to build DOM structures without using HTML strings.

```javascript
const div = document.createElement('div');
div.textContent = 'Created dynamically';
document.body.appendChild(div);
```

### Why It Is Important

Programmatic element creation is fundamental to dynamic web applications. Unlike `innerHTML`, `createElement` provides type safety (the element is created as a specific HTMLElement subclass with all its properties), better performance for complex structures, and allows attaching event listeners before insertion. It is the preferred approach for framework internals and performance-critical code.

### How It Works Internally

`createElement` creates an instance of the appropriate `HTMLElement` subclass (e.g., `HTMLDivElement` for `div`, `HTMLInputElement` for `input`). The element is initialized with default properties but is not connected to the DOM tree (`isConnected` returns `false`). `appendChild` moves the node from its current position (or from detached state) to the end of the parent's child list, updating all parent/sibling references and triggering reflow if the element was already connected.

```javascript
// Element lifecycle
// 1. Create: document.createElement('div')
//    - element.isConnected === false
//    - element.parentNode === null
// 2. Configure: Set properties, add listeners
// 3. Insert: parent.appendChild(element)
//    - element.isConnected === true
//    - Reflow triggered if in visible document
// 4. Remove: parent.removeChild(element)
//    - element.isConnected === false
```

### Syntax

```javascript
// Creating elements
const element = document.createElement(tagName);
const element = document.createElementNS(namespaceURI, qualifiedName);

// Appending children
parent.appendChild(child);

// Parameters
// tagName: string - HTML tag name (case-insensitive)
// child: Node - The node to append

// Returns
// createElement: Element
// appendChild: Node (the appended child)
```

### Beginner Examples

```javascript
// Example 1: Creating and appending elements
const list = document.getElementById('list');
const item = document.createElement('li');
item.textContent = 'New list item';
list.appendChild(item);

// Example 2: Building a card structure
const card = document.createElement('div');
card.className = 'card';

const title = document.createElement('h2');
title.textContent = 'Card Title';

const body = document.createElement('p');
body.textContent = 'Card description text';

card.appendChild(title);
card.appendChild(body);
document.getElementById('cards').appendChild(card);

// Example 3: Creating input elements
const input = document.createElement('input');
input.type = 'text';
input.placeholder = 'Enter your name';
input.id = 'name-input';
input.required = true;
document.querySelector('form').appendChild(input);

// Example 4: Creating image elements
const img = document.createElement('img');
img.src = 'https://example.com/image.jpg';
img.alt = 'Description';
img.width = 300;
img.height = 200;
document.body.appendChild(img);
```

### Intermediate Examples

```javascript
// Example 1: Batch element creation with DocumentFragment
function createList(items) {
  const fragment = document.createDocumentFragment();

  for (const item of items) {
    const li = document.createElement('li');
    li.textContent = item;
    li.className = 'list-item';
    fragment.appendChild(li);
  }

  return fragment;
}

document.getElementById('list').appendChild(createList(['A', 'B', 'C']));

// Example 2: Element factory with configuration
function createElement(tag, config = {}) {
  const el = document.createElement(tag);

  if (config.className) el.className = config.className;
  if (config.id) el.id = config.id;
  if (config.text) el.textContent = config.text;
  if (config.html) el.innerHTML = config.html;
  if (config.attributes) {
    for (const [key, value] of Object.entries(config.attributes)) {
      el.setAttribute(key, value);
    }
  }
  if (config.dataset) {
    for (const [key, value] of Object.entries(config.dataset)) {
      el.dataset[key] = value;
    }
  }
  if (config.children) {
    for (const child of config.children) {
      el.appendChild(
        typeof child === 'string'
          ? document.createTextNode(child)
          : createElement(child.tag, child)
      );
    }
  }
  if (config.listeners) {
    for (const [event, handler] of Object.entries(config.listeners)) {
      el.addEventListener(event, handler);
    }
  }

  return el;
}

const button = createElement('button', {
  className: 'btn btn-primary',
  text: 'Click me',
  attributes: { 'data-action': 'submit' },
  listeners: {
    click: () => alert('Clicked!')
  }
});

document.body.appendChild(button);

// Example 3: Moving elements vs cloning
const source = document.getElementById('source');
const target = document.getElementById('target');

// Move (detach from source, attach to target)
target.appendChild(source); // source is removed from its original position

// Clone and append
const clone = source.cloneNode(true); // deep clone
target.appendChild(clone); // original remains in place

// Example 4: Conditional element insertion
function insertIf(condition, parent, createFn) {
  if (condition) {
    parent.appendChild(createFn());
  }
}

const sidebar = document.getElementById('sidebar');
insertIf(user.isAdmin, sidebar, () => {
  const link = document.createElement('a');
  link.href = '/admin';
  link.textContent = 'Admin Panel';
  return link;
});
```

### Advanced Examples

```javascript
// Example 1: Virtual element reconciliation (minimal)
class VElement {
  constructor(tag, props, ...children) {
    this.tag = tag;
    this.props = props || {};
    this.children = children.flat();
    this.el = null;
  }

  mount(parent) {
    this.el = document.createElement(this.tag);

    for (const [key, value] of Object.entries(this.props)) {
      if (key.startsWith('on')) {
        this.el.addEventListener(key.slice(2).toLowerCase(), value);
      } else if (key === 'className') {
        this.el.className = value;
      } else if (key === 'style' && typeof value === 'object') {
        Object.assign(this.el.style, value);
      } else {
        this.el.setAttribute(key, value);
      }
    }

    for (const child of this.children) {
      if (child instanceof VElement) {
        child.mount(this.el);
      } else {
        this.el.appendChild(document.createTextNode(String(child)));
      }
    }

    parent.appendChild(this.el);
    return this;
  }

  update(props, ...children) {
    // Simplified update - in practice, would diff
    const newEl = new VElement(this.tag, props, ...children);
    this.el.parentNode.replaceChild(newEl.mount(), this.el);
    return newEl;
  }
}

const app = new VElement('div', { className: 'app' },
  new VElement('h1', {}, 'Hello'),
  new VElement('p', { style: { color: 'blue' } }, 'World')
);

app.mount(document.getElementById('root'));

// Example 2: DOM builder with fluent API
class DOMBuilder {
  constructor(tag) {
    this.element = document.createElement(tag);
  }

  text(content) {
    this.element.textContent = content;
    return this;
  }

  class(...names) {
    this.element.classList.add(...names);
    return this;
  }

  attr(name, value) {
    this.element.setAttribute(name, value);
    return this;
  }

  data(key, value) {
    this.element.dataset[key] = value;
    return this;
  }

  on(event, handler) {
    this.element.addEventListener(event, handler);
    return this;
  }

  style(prop, value) {
    this.element.style[prop] = value;
    return this;
  }

  append(child) {
    if (child instanceof DOMBuilder) {
      this.element.appendChild(child.element);
    } else if (child instanceof Node) {
      this.element.appendChild(child);
    } else if (typeof child === 'string') {
      this.element.appendChild(document.createTextNode(child));
    }
    return this;
  }

  mount(parent) {
    parent.appendChild(this.element);
    return this;
  }

  get() {
    return this.element;
  }
}

const $ = (tag) => new DOMBuilder(tag);

$('div')
  .class('container')
  .append(
    $('h1').text('Title').class('heading'),
    $('p')
      .text('Description')
      .style('color', '#666')
      .on('click', () => alert('Clicked'))
  )
  .mount(document.body);

// Example 3: Element recycling pool (for performance)
class ElementPool {
  constructor(tag, maxSize = 100) {
    this.tag = tag;
    this.maxSize = maxSize;
    this.pool = [];
  }

  acquire() {
    if (this.pool.length > 0) {
      const el = this.pool.pop();
      // Reset to default state
      el.className = '';
      el.textContent = '';
      el.style.cssText = '';
      return el;
    }
    return document.createElement(this.tag);
  }

  release(element) {
    if (this.pool.length < this.maxSize) {
      // Detach from parent
      if (element.parentNode) {
        element.parentNode.removeChild(element);
      }
      this.pool.push(element);
    }
  }
}

const divPool = new ElementPool('div');
const recycledDiv = divPool.acquire();
// Use it...
divPool.release(recycledDiv);
```

### Real-World Use Cases

- **Virtual Scrolling**: Creating DOM elements only for visible items in large lists
- **Chart Libraries**: D3.js uses createElement/svg element creation for data visualization
- **Dynamic Forms**: Generating form fields based on JSON schema
- **Component Systems**: Web component libraries build elements programmatically in constructor
- **Testing Utilities**: Creating DOM fixtures for unit tests
- **Animation Systems**: Spawning and removing animated elements on demand
- **Search Results**: Rendering real-time search suggestions as DOM elements

### Common Mistakes

```javascript
// Mistake 1: Creating elements inside loops without fragment
for (const item of items) {
  const li = document.createElement('li');
  li.textContent = item;
  list.appendChild(li); // Reflow triggered each iteration
}
// Better: Use DocumentFragment

// Mistake 2: Appending the same element multiple times
const el = document.createElement('div');
el.textContent = 'Item';
parent1.appendChild(el);
parent2.appendChild(el); // Moves from parent1 to parent2 (single element!)

// Mistake 3: Not removing from old parent before appending
// appendChild automatically handles this, so this is actually fine
// But being explicit can improve readability

// Mistake 4: Forgetting to set properties before appending
const btn = document.createElement('button');
parent.appendChild(btn);
btn.textContent = 'Click'; // Works but causes extra reflow if visible
// Better to configure before inserting
```

### Best Practices

- Always use `DocumentFragment` for batch insertions (single reflow)
- Configure elements fully before appending to the DOM
- Use `replaceChild` and `insertBefore` for precise positioning
- Cache references to frequently created element types
- Consider `insertAdjacentElement` for positioning relative to existing elements
- Use `isConnected` to check if an element is in the DOM
- Detach elements for intensive manipulation, then reattach

### Performance Considerations

`createElement` is faster than innerHTML parsing for complex structures because it bypasses the HTML parser. However, each `appendChild` triggers a potential reflow if the element is in the visible DOM. Batch operations with `DocumentFragment` minimize reflows.

```javascript
// Fast: createElement with fragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  fragment.appendChild(div);
}
container.appendChild(fragment); // 1 reflow

// Slow: innerHTML in loop
for (let i = 0; i < 1000; i++) {
  container.innerHTML += `<div>Item ${i}</div>`; // 1000 reflows + parsing
}
```

### Interview Questions

1. What is the difference between `appendChild` and `append`?
2. How does `DocumentFragment` improve DOM insertion performance?
3. What happens when you append a node that already has a parent?
4. How do you create elements with namespace URIs?
5. What is `insertAdjacentElement` and when would you use it?
6. How does `replaceChild` work?
7. What is the `cloneNode` method and its `deep` parameter?
8. How can you create an element and insert it at a specific position?

### Coding Challenges

1. **DOM Builder**: Create a fluent API for building DOM trees programmatically
2. **Virtual DOM**: Implement a minimal create/diff/patch system using createElement
3. **Element Pool**: Implement an element recycling system for performance optimization
4. **Component Renderer**: Build a function that renders a JavaScript object tree to DOM elements
5. **Batch Inserter**: Implement efficient batch insertion with progress tracking

### Related Topics

- [insertAdjacentElement](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement)
- [removeChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/removeChild)
- [replaceChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/replaceChild)
- [cloneNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)
- [DocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment)

## classList API

### What It Is

The `classList` property (available on `Element` nodes) returns a live `DOMTokenList` representing the element's `class` attribute. It provides methods to add, remove, toggle, replace, and check for CSS classes without manipulating the `className` string directly.

```javascript
const element = document.querySelector('.item');
element.classList.add('active');
element.classList.remove('hidden');
element.classList.toggle('expanded');
element.classList.contains('highlight');
element.classList.replace('old-class', 'new-class');
```

### Why It Is Important

The `classList` API provides a clean, readable, and performant way to manage CSS classes. It avoids string manipulation errors, handles multiple classes correctly, and provides atomic operations that are consistent across browsers. CSS class manipulation is the primary mechanism for controlling visual state in web applications.

### How It Works Internally

`classList` returns a `DOMTokenList` that is live-backed by the element's `className` property. Methods like `add`, `remove`, and `toggle` modify the internal space-separated class string and trigger attribute change mutation records that the browser can use for style recalculation.

```javascript
// Conceptual behavior:
// classList.add('active', 'visible')
// -> className += ' active visible' (with deduplication)
// -> Triggers style recalculation if active/visible CSS rules exist
```

### Syntax

```javascript
element.classList.add(className1, className2, ...);
element.classList.remove(className1, className2, ...);
element.classList.toggle(className, force);
element.classList.contains(className);
element.classList.replace(oldClass, newClass);
element.classList.item(index);
element.classList.length;
element.classList.value; // Space-separated class string
```

### Beginner Examples

```javascript
// Example 1: Basic class operations
const nav = document.querySelector('nav');
nav.classList.add('sticky');
nav.classList.remove('sticky');
nav.classList.toggle('open');
nav.classList.contains('open'); // true/false

// Example 2: Multiple classes
element.classList.add('foo', 'bar', 'baz');
element.classList.remove('foo', 'bar');

// Example 3: Conditional class based on state
function updateButtonState(isLoading) {
  const btn = document.querySelector('.submit-btn');
  btn.classList.toggle('loading', isLoading);
  btn.classList.toggle('disabled', isLoading);
  btn.textContent = isLoading ? 'Saving...' : 'Save';
}

// Example 4: Class checking
if (element.classList.contains('error')) {
  showErrorIndicator(element);
}
```

### Intermediate Examples

```javascript
// Example 1: Class-based state machine
class StateMachine {
  constructor(element, states) {
    this.element = element;
    this.states = states;
    this.currentState = null;
  }

  transition(newState) {
    if (!this.states.includes(newState)) {
      throw new Error(`Invalid state: ${newState}`);
    }

    if (this.currentState) {
      this.element.classList.remove(`state-${this.currentState}`);
    }

    this.element.classList.add(`state-${newState}`);
    this.currentState = newState;
  }

  get state() {
    return this.currentState;
  }
}

const statusIndicator = document.querySelector('.status');
const machine = new StateMachine(statusIndicator, ['idle', 'loading', 'success', 'error']);
machine.transition('loading');
// Uses classList.remove('state-idle'), add('state-loading')

// Example 2: Toggle with callback
function toggleWithCallback(element, className, callback) {
  const isNowActive = element.classList.toggle(className);
  callback(isNowActive);
  return isNowActive;
}

toggleWithCallback(menu, 'open', (isOpen) => {
  document.body.classList.toggle('menu-open', isOpen);
  if (isOpen) {
    trapFocus(menu);
  }
});

// Example 3: Class list diffing
function syncClasses(element, targetClasses) {
  const currentClasses = Array.from(element.classList);

  // Remove classes not in target
  for (const cls of currentClasses) {
    if (!targetClasses.includes(cls)) {
      element.classList.remove(cls);
    }
  }

  // Add target classes not present
  for (const cls of targetClasses) {
    if (!currentClasses.includes(cls)) {
      element.classList.add(cls);
    }
  }
}

syncClasses(element, ['active', 'highlighted', 'visible']);
```

### Advanced Examples

```javascript
// Example 1: CSS class-based animation system
class AnimationManager {
  constructor(element) {
    this.element = element;
    this.listeners = new Map();
  }

  play(animationName) {
    return new Promise((resolve) => {
      const el = this.element;

      // Start animation
      el.classList.add(`anim-${animationName}`);

      // Listen for animation end
      const handler = () => {
        el.classList.remove(`anim-${animationName}`);
        el.removeEventListener('animationend', handler);
        resolve();
      };

      el.addEventListener('animationend', handler);
    });
  }

  playSequential(animations) {
    return animations.reduce(
      (promise, anim) => promise.then(() => this.play(anim)),
      Promise.resolve()
    );
  }
}

const anim = new AnimationManager(document.querySelector('.card'));
await anim.playSequential(['fadeIn', 'slideUp', 'bounce']);

// Example 2: Class-based theme system
class ThemeManager {
  constructor(root = document.documentElement) {
    this.root = root;
    this.themes = ['light', 'dark', 'contrast'];
    this.currentTheme = this.detectTheme();
  }

  detectTheme() {
    for (const theme of this.themes) {
      if (this.root.classList.contains(`theme-${theme}`)) {
        return theme;
      }
    }
    return 'light';
  }

  setTheme(theme) {
    if (!this.themes.includes(theme)) return;

    this.root.classList.remove(...this.themes.map(t => `theme-${t}`));
    this.root.classList.add(`theme-${theme}`);
    this.currentTheme = theme;

    localStorage.setItem('theme', theme);
    this.dispatchThemeChange(theme);
  }

  toggleTheme() {
    const index = this.themes.indexOf(this.currentTheme);
    const next = this.themes[(index + 1) % this.themes.length];
    this.setTheme(next);
  }

  dispatchThemeChange(theme) {
    window.dispatchEvent(
      new CustomEvent('themechange', { detail: { theme } })
    );
  }
}

const themeManager = new ThemeManager();
themeManager.setTheme('dark');

// Example 3: classList-based responsive behavior
class ResponsiveClassManager {
  constructor(breakpoints) {
    this.breakpoints = breakpoints;
    this.mqls = new Map();

    this.init();
  }

  init() {
    for (const [name, width] of Object.entries(this.breakpoints)) {
      const mql = window.matchMedia(`(min-width: ${width}px)`);
      this.mqls.set(name, mql);

      const handler = (e) => {
        document.documentElement.classList.toggle(`bp-${name}`, e.matches);
      };

      mql.addEventListener('change', handler);
      handler(mql);
    }
  }

  destroy() {
    for (const mql of this.mqls.values()) {
      mql.removeEventListener('change', handler);
    }
  }
}

const manager = new ResponsiveClassManager({
  sm: 640,
  md: 768,
  lg: 1024,
  xl: 1280
});
// Adds/removes classes like 'bp-md' based on viewport width
```

### Real-World Use Cases

- **Form Validation**: Adding `is-valid`/`is-invalid` classes to form fields
- **Navigation Menus**: Toggling `open` class on mobile hamburger menus
- **Modal Dialogs**: Adding `is-open` class to show/hide modals with CSS transitions
- **Dark Mode**: Toggling `dark-theme` class on `document.documentElement`
- **Accordions**: Toggling `expanded` class on accordion panels
- **Tabs**: Managing `active` class on tab buttons and panels
- **Loading States**: Adding `loading` class to buttons during async operations

### Common Mistakes

```javascript
// Mistake 1: Using classList.add with a string containing spaces
element.classList.add('foo bar'); // Adds class "foo bar" (with space in name)
element.classList.add('foo', 'bar'); // Correct: adds two classes

// Mistake 2: Not checking before remove
element.classList.remove('non-existent'); // No error, just no-op (which is fine)

// Mistake 3: Using className when classList is more appropriate
element.className += ' active'; // Creates "class1 class2 active" (can duplicate)
element.classList.add('active'); // Handles duplicates automatically

// Mistake 4: Toggle without checking return
const result = element.classList.toggle('active');
console.log(result); // true if class was added, false if removed
```

### Best Practices

- Use `classList.toggle(className, force)` with a boolean condition
- Prefer `classList.add`/`remove` over direct `className` manipulation
- Batch class changes where possible to minimize style recalculations
- Use `contains()` for class existence checks
- Prefer `replace()` over `remove()` + `add()` for atomic class swaps
- Use meaningful class names that describe state (e.g., `is-active`, `has-error`)

### Performance Considerations

`classList` operations are highly optimized in modern browsers. They use internal string manipulation that is faster than manual `className` string operations. The `toggle` method with a boolean `force` parameter is the most efficient way to conditionally set a class.

```javascript
// Fast: classList
element.classList.toggle('active', condition);
element.classList.add('visible');

// Slower: className manipulation
element.className = element.className.replace('active', '').trim() + (condition ? ' active' : '');
```

### Interview Questions

1. What is the difference between `className` and `classList`?
2. Does `classList` support multiple classes in `add`/`remove`?
3. What does `classList.toggle` return?
4. How do you replace one class with another using `classList`?
5. Is `classList` live? What does that mean?
6. How does `classList.value` differ from `className`?
7. Can `classList.add` throw errors?
8. What is the browser support for `classList.replace`?

### Coding Challenges

1. **Class Observer**: Monitor class changes on an element and log them
2. **CSS State Machine**: Build a finite state machine using CSS classes
3. **Class Difference Calculator**: Given two states, compute add/remove operations

### Related Topics

- [CSS Classes and Selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Class_selectors)
- [DOMTokenList](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList)
- [className](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)
- [Style Property](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)

## insertBefore and removeChild

### What It Is

`insertBefore()` inserts a node as a child of a parent before a specified reference child. `removeChild()` removes a specified child node from its parent and returns it. These methods provide precise control over DOM structure beyond simple appending.

```javascript
// Insert before reference
parent.insertBefore(newChild, referenceChild);

// Remove specific child
const removed = parent.removeChild(child);
```

### Why It Is Important

These methods enable precise DOM manipulation—inserting elements at specific positions, reordering children, and removing specific elements. They are essential for building dynamic lists, sortable tables, priority queues, and any UI that requires ordered content management.

### How It Works Internally

`insertBefore` updates the parent's child node linked list by inserting the new node between the previous sibling and the reference child. If the node is already in the DOM, it is first removed from its current position (move operation). `removeChild` updates the linked list to bypass the removed node and returns a reference to it.

```javascript
// Conceptual internal behavior
// insertBefore(newChild, referenceChild):
// 1. If newChild has parent, removeChild(newChild) first
// 2. Update referenceChild.previousSibling.nextSibling = newChild
// 3. newChild.previousSibling = referenceChild.previousSibling
// 4. newChild.nextSibling = referenceChild
// 5. referenceChild.previousSibling = newChild
// 6. Update parent.childNodes
```

### Syntax

```javascript
// Insert before
const insertedNode = parent.insertBefore(newNode, referenceNode);
// referenceNode must be a child of parent. If null, newNode is appended.

// Remove child
const removedNode = parent.removeChild(child);
// child must be a direct child of parent
```

### Beginner Examples

```javascript
// Example 1: Inserting at the beginning
const list = document.getElementById('list');
const firstItem = document.createElement('li');
firstItem.textContent = 'First item';
list.insertBefore(firstItem, list.firstChild);

// Example 2: Removing an element
const itemToRemove = document.getElementById('item-3');
itemToRemove.parentNode.removeChild(itemToRemove);

// Example 3: Safe removal
function safeRemove(element) {
  if (element && element.parentNode) {
    element.parentNode.removeChild(element);
  }
}

// Example 4: Moving an element
const item1 = document.getElementById('item-1');
const item3 = document.getElementById('item-3');
const parent = document.getElementById('list');
parent.insertBefore(item3, item1); // Move item3 before item1

// Example 5: Append using insertBefore with null
parent.insertBefore(newItem, null); // Equivalent to appendChild
```

### Intermediate Examples

```javascript
// Example 1: Reordering list items
function moveItem(list, fromIndex, toIndex) {
  const children = list.children;
  if (fromIndex === toIndex) return;
  if (fromIndex < 0 || fromIndex >= children.length) return;
  if (toIndex < 0 || toIndex > children.length) return;

  const item = children[fromIndex];
  const reference = toIndex < children.length ? children[toIndex] : null;

  list.insertBefore(item, reference);
}

// Example 2: Sorting DOM elements
function sortChildren(parent, comparator) {
  const items = Array.from(parent.children);
  items.sort(comparator);

  const fragment = document.createDocumentFragment();
  for (const item of items) {
    fragment.appendChild(item);
  }
  parent.appendChild(fragment); // All reordered in one operation
}

// Sort list alphabetically
sortChildren(document.getElementById('list'), (a, b) =>
  a.textContent.localeCompare(b.textContent)
);

// Example 3: Batch removal with filter
function removeMatching(parent, predicate) {
  const toRemove = Array.from(parent.children).filter(predicate);
  for (const item of toRemove) {
    parent.removeChild(item);
  }
  return toRemove;
}

const removed = removeMatching(
  document.getElementById('list'),
  item => item.classList.contains('archived')
);

// Example 4: InsertAfter equivalent (not natively available)
function insertAfter(newNode, referenceNode) {
  if (referenceNode.nextSibling) {
    referenceNode.parentNode.insertBefore(newNode, referenceNode.nextSibling);
  } else {
    referenceNode.parentNode.appendChild(newNode);
  }
}

insertAfter(newElement, document.querySelector('.last-item'));
```

### Advanced Examples

```javascript
// Example 1: Range-based element manipulation
class DOMRange {
  constructor(parent, startIndex, endIndex) {
    this.parent = parent;
    this.startIndex = startIndex;
    this.endIndex = endIndex;
  }

  get elements() {
    const result = [];
    const children = this.parent.children;
    for (let i = this.startIndex; i <= this.endIndex && i < children.length; i++) {
      result.push(children[i]);
    }
    return result;
  }

  remove() {
    const elements = this.elements;
    for (const el of elements) {
      this.parent.removeChild(el);
    }
    return elements;
  }

  wrap(wrapperTag) {
    const wrapper = document.createElement(wrapperTag);
    const elements = this.elements;
    if (elements.length === 0) return;

    this.parent.insertBefore(wrapper, elements[0]);
    for (const el of elements) {
      wrapper.appendChild(el);
    }
    return wrapper;
  }

  moveBefore(reference) {
    const elements = this.elements;
    const fragment = document.createDocumentFragment();
    for (const el of elements) {
      fragment.appendChild(el);
    }
    this.parent.insertBefore(fragment, reference);
  }
}

const range = new DOMRange(
  document.getElementById('list'),
  0,
  2
);
range.wrap('div');

// Example 2: Virtual scrolling with element recycling
class VirtualScroller {
  constructor(container, options) {
    this.container = container;
    this.itemHeight = options.itemHeight;
    this.totalItems = options.totalItems;
    this.renderItem = options.renderItem;
    this.visibleItems = Math.ceil(container.clientHeight / this.itemHeight) + 2;
    this.startIndex = 0;
    this.elementCache = [];
  }

  init() {
    this.container.style.position = 'relative';
    this.container.style.height = `${this.totalItems * this.itemHeight}px`;
    this.container.addEventListener('scroll', () => this.update());
    this.update();
  }

  update() {
    const scrollTop = this.container.scrollTop;
    const newStart = Math.floor(scrollTop / this.itemHeight);

    if (newStart === this.startIndex) return;
    this.startIndex = newStart;

    // Remove items out of view
    while (this.container.children.length > this.visibleItems) {
      const last = this.container.lastChild;
      this.container.removeChild(last);
      this.elementCache.push(last);
    }

    // Insert new items
    const fragment = document.createDocumentFragment();
    for (let i = this.startIndex; i < this.startIndex + this.visibleItems; i++) {
      if (i >= this.totalItems) break;

      let el = this.elementCache.pop();
      if (!el) {
        el = this.renderItem(i);
      }
      el.style.position = 'absolute';
      el.style.top = `${i * this.itemHeight}px`;
      fragment.appendChild(el);
    }

    this.container.insertBefore(fragment, this.container.firstChild);
  }
}

// Example 3: Transactional DOM operations
class DOMTransaction {
  constructor() {
    this.operations = [];
  }

  insertBefore(parent, newNode, reference) {
    this.operations.push(() => {
      parent.insertBefore(newNode, reference);
    });
    return this;
  }

  removeChild(parent, child) {
    this.operations.push(() => {
      parent.removeChild(child);
    });
    return this;
  }

  replaceChild(parent, newChild, oldChild) {
    this.operations.push(() => {
      parent.replaceChild(newChild, oldChild);
    });
    return this;
  }

  commit() {
    // Batch operations minimize reflows
    for (const op of this.operations) {
      op();
    }
    this.operations = [];
    return this;
  }

  rollback() {
    this.operations = [];
    return this;
  }
}

const transaction = new DOMTransaction();
transaction
  .removeChild(list, item1)
  .insertBefore(list, item1, list.firstChild)
  .commit();
```

### Real-World Use Cases

- **Todo List Apps**: Adding/removing/reordering todo items
- **Kanban Boards**: Moving cards between columns
- **File Managers**: Drag-and-drop reordering of file lists
- **Priority Queues**: Reordering items by priority
- **Tab Management**: Reordering browser tabs in custom implementations
- **Comment Threads**: Inserting new comments at specific positions
- **Sortable Tables**: Client-side column sorting by moving rows

### Common Mistakes

```javascript
// Mistake 1: Reference node not a child of parent
const parent = document.getElementById('list');
const reference = document.getElementById('other-list-item');
parent.insertBefore(newItem, reference); // Error: reference not a child of parent

// Mistake 2: Removing a child while iterating children
const children = parent.children;
for (let i = 0; i < children.length; i++) {
  if (children[i].matches('.remove')) {
    parent.removeChild(children[i]); // Children re-indexed, skipping next element
  }
}
// Fix: iterate backwards or convert to array

// Mistake 3: Using removeChild on a node that's not a child
const element = document.querySelector('.element');
document.body.removeChild(element); // Error if element is not in body

// Mistake 4: Not using the return value (detached element can be reused)
const removed = parent.removeChild(child);
// removed is now detached but still has all its properties/events
someOtherParent.appendChild(removed); // Re-insert elsewhere
```

### Best Practices

- Always verify the reference node is a valid child before `insertBefore`
- Check `parentNode` before calling `removeChild`
- Use `DocumentFragment` for moving multiple elements at once
- Cache `children.length` when iterating if elements won't be removed
- Prefer `append()` and `prepend()` for simpler cases (newer API)
- Use `replaceChild()` as an atomic alternative to remove + insert
- Handle edge cases: empty lists, null references, first/last positions

### Performance Considerations

`insertBefore` and `removeChild` are O(1) operations for the actual DOM manipulation, but they can trigger reflow when applied to elements in the visible document. Batch operations minimize reflow costs. Moving elements is more expensive than inserting new ones because it involves detaching and reattaching.

```javascript
// Bad: Multiple individual removals
list1.removeChild(item);
list2.appendChild(item);
list1.removeChild(item2);
list2.appendChild(item2);

// Good: Batch all removals, then insert
const fragment = document.createDocumentFragment();
fragment.appendChild(item);
fragment.appendChild(item2);
list2.appendChild(fragment); // Single reflow for fragment
```

### Interview Questions

1. What happens to event listeners on a child removed with `removeChild`?
2. How do you insert an element after another element (not before)?
3. What does `insertBefore` do if the reference node is `null`?
4. Can `removeChild` be called on a DocumentFragment?
5. How do you move an element from one parent to another?
6. What is the difference between `removeChild` and `remove`?
7. How does `insertBefore` handle the case where the new node already has a parent?
8. What is the return value of `removeChild`?

### Coding Challenges

1. **Sortable List**: Implement drag-and-drop reordering using insertBefore
2. **Virtual Scroller**: Build a performant virtual scroller using element recycling
3. **DOM Transaction**: Implement a transactional DOM operation system
4. **Range Operations**: Build a utility for selecting and manipulating ranges of children

### Related Topics

- [prepend/append](https://developer.mozilla.org/en-US/docs/Web/API/Element/prepend)
- [before/after](https://developer.mozilla.org/en-US/docs/Web/API/Element/before)
- [replaceChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/replaceChild)
- [remove()](https://developer.mozilla.org/en-US/docs/Web/API/ChildNode/remove)
- [InsertAdjacentElement](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement)
