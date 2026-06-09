# Event Delegation - Delegation pattern, dynamic elements, performance

## Introduction

Event delegation is a powerful JavaScript pattern that leverages event bubbling to handle events efficiently. Instead of attaching event listeners to individual elements, a single listener is attached to a common ancestor. When an event bubbles up from a child element, the ancestor's listener evaluates the event target and responds accordingly. This pattern is essential for managing events on dynamically created elements and optimizing performance in applications with many interactive elements.

Event delegation is used extensively in modern web frameworks and libraries. React's synthetic event system, jQuery's event methods, and Vue's event handling all leverage delegation internally. Understanding delegation is fundamental to writing efficient, scalable DOM event code.

## Event Delegation Pattern

### What It Is

Event delegation is a technique where a single event listener is attached to a parent element to handle events from multiple children. The listener uses `event.target` (or `event.target.closest()`) to determine which child triggered the event and applies the appropriate logic. This works because most events bubble from the target element up through the DOM tree.

```javascript
// Instead of this (individual listeners):
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleItemClick);
});

// Do this (delegation):
document.querySelector('.list').addEventListener('click', (event) => {
  const item = event.target.closest('.item');
  if (item) {
    handleItemClick(event, item);
  }
});
```

### Why It Is Important

Event delegation reduces memory usage by replacing N listeners with 1. It simplifies code maintenance since there is a single registration point. Most importantly, it automatically handles dynamically added elements—new children inherit the event handling without any additional setup.

### How It Works Internally

The delegation pattern relies on event bubbling. When a child element fires an event, the event travels up through parent elements. The delegated listener on a common ancestor receives the event and uses `event.target` (the actual element that triggered the event) to determine if it should respond.

```javascript
// Conceptual delegation flow:
// 1. User clicks on <li class="item"> inside <ul class="list">
// 2. Click event fires on <li>
// 3. Event bubbles to <ul>
// 4. <ul>'s click listener fires
// 5. Listener checks: event.target.matches('.item')? (or closest)
// 6. If match, execute handler with the matched element
```

### Syntax

```javascript
// Basic delegation pattern
parent.addEventListener('click', (event) => {
  const target = event.target.closest('.child-selector');
  if (target) {
    // Handle the event
    handler(event, target);
  }
});

// For elements without closest (older browsers):
parent.addEventListener('click', (event) => {
  let target = event.target;
  while (target && target !== parent) {
    if (target.matches('.child-selector')) {
      handler(event, target);
      break;
    }
    target = target.parentElement;
  }
});
```

### Beginner Examples

```javascript
// Example 1: Delegated click handler for list items
const list = document.querySelector('.todo-list');
list.addEventListener('click', (event) => {
  const item = event.target.closest('.todo-item');
  if (!item) return;

  if (event.target.matches('.delete-btn')) {
    item.remove();
  } else if (event.target.matches('.complete-btn')) {
    item.classList.toggle('completed');
  } else {
    item.classList.toggle('selected');
  }
});

// Example 2: Delegated form input validation
const form = document.querySelector('.form');
form.addEventListener('input', (event) => {
  const field = event.target;
  if (!field.matches('.validate')) return;

  if (field.validity.valid) {
    field.classList.remove('invalid');
    field.classList.add('valid');
  } else {
    field.classList.remove('valid');
    field.classList.add('invalid');
  }
});

// Example 3: Delegated mouseover for tooltips
const container = document.querySelector('.container');
container.addEventListener('mouseover', (event) => {
  const tooltipTrigger = event.target.closest('[data-tooltip]');
  if (!tooltipTrigger) return;

  showTooltip(tooltipTrigger, tooltipTrigger.dataset.tooltip);
});

container.addEventListener('mouseout', (event) => {
  const tooltipTrigger = event.target.closest('[data-tooltip]');
  if (!tooltipTrigger) return;

  hideTooltip(tooltipTrigger);
});
```

### Intermediate Examples

```javascript
// Example 1: Multi-event delegation
class DelegatedController {
  constructor(root) {
    this.root = root;
    this.handlers = {};
    this.setup();
  }

  setup() {
    ['click', 'dblclick', 'contextmenu'].forEach(eventType => {
      this.root.addEventListener(eventType, (event) => {
        this.handleEvent(event);
      });
    });
  }

  on(selector, eventType, handler) {
    const key = `${selector}:${eventType}`;
    this.handlers[key] = handler;
  }

  handleEvent(event) {
    let target = event.target;
    while (target && target !== this.root) {
      for (const [key, handler] of Object.entries(this.handlers)) {
        const [selector, eventType] = key.split(':');
        if (event.type === eventType && target.matches(selector)) {
          handler(event, target);
        }
      }
      target = target.parentElement;
    }
  }
}

const controller = new DelegatedController(document.querySelector('.app'));
controller.on('.delete-btn', 'click', (e, el) => {
  el.closest('.item').remove();
});
controller.on('.edit-btn', 'dblclick', (e, el) => {
  startEditing(el.closest('.item'));
});

// Example 2: Data-attribute driven delegation
document.body.addEventListener('click', (event) => {
  const actionEl = event.target.closest('[data-action]');
  if (!actionEl) return;

  const action = actionEl.dataset.action;
  const payload = actionEl.dataset.payload;

  switch (action) {
    case 'delete':
      deleteItem(payload);
      break;
    case 'edit':
      editItem(payload);
      break;
    case 'toggle':
      toggleItem(payload);
      break;
    case 'navigate':
      navigateTo(payload);
      break;
    default:
      console.warn(`Unknown action: ${action}`);
  }
});

// Example 3: Delegated form submission
document.addEventListener('submit', (event) => {
  const form = event.target;

  if (form.matches('.ajax-form')) {
    event.preventDefault();
    handleAjaxSubmit(form);
  } else if (form.matches('.confirm-form')) {
    if (!confirm('Are you sure?')) {
      event.preventDefault();
    }
  }
});

async function handleAjaxSubmit(form) {
  const data = new FormData(form);
  const url = form.action;
  const method = form.method;

  try {
    const response = await fetch(url, { method, body: data });
    if (response.ok) {
      showToast('Form submitted successfully');
    }
  } catch (error) {
    showToast('Submission failed', 'error');
  }
}
```

### Advanced Examples

```javascript
// Example 1: Full-featured delegation system
class EventDelegationSystem {
  constructor(root = document) {
    this.root = root;
    this.rules = new Map();
    this.setup();
  }

  setup() {
    // Capture all bubbled events and dispatch to rules
    const eventTypes = ['click', 'dblclick', 'mouseenter', 'mouseleave',
                       'focusin', 'focusout', 'change', 'input', 'submit'];

    eventTypes.forEach(type => {
      this.root.addEventListener(type, (event) => {
        this.dispatch(event);
      });
    });
  }

  addRule(selector, eventType, handler, options = {}) {
    const key = `${eventType}:${selector}`;
    if (!this.rules.has(key)) {
      this.rules.set(key, []);
    }
    this.rules.get(key).push({ handler, options });
  }

  removeRule(selector, eventType, handler) {
    const key = `${eventType}:${selector}`;
    const rules = this.rules.get(key);
    if (rules) {
      const idx = rules.findIndex(r => r.handler === handler);
      if (idx >= 0) rules.splice(idx, 1);
    }
  }

  dispatch(event) {
    // Walk from target to root, matching rules
    let current = event.target;
    while (current && current !== this.root) {
      for (const [key, handlers] of this.rules) {
        const [eventType, selector] = key.split(':');
        if (event.type !== eventType) continue;
        if (event.cancelBubble) return;

        if (current.matches(selector)) {
          for (const { handler, options } of handlers) {
            if (options.once) {
              this.removeRule(selector, eventType, handler);
            }
            handler(event, current);
            if (event.cancelBubble) break;
          }
        }
      }
      current = current.parentElement;
    }
  }
}

const delegation = new EventDelegationSystem();
delegation.addRule('.item', 'click', (e, el) => {
  console.log('Item clicked:', el.textContent);
}, { once: true });

// Example 2: Delegation with middleware pipeline
class DelegationPipeline {
  constructor(root) {
    this.root = root;
    this.middleware = [];
    this.handlers = new Map();
  }

  use(fn) {
    this.middleware.push(fn);
  }

  on(selector, eventType, handler) {
    const key = `${eventType}:${selector}`;
    if (!this.handlers.has(key)) {
      this.handlers.set(key, []);
      this.root.addEventListener(eventType, (event) => {
        this.execute(event, key);
      });
    }
    this.handlers.get(key).push(handler);
  }

  execute(event, key) {
    const [eventType, selector] = key.split(':');
    let target = event.target;

    while (target && target !== this.root) {
      if (target.matches(selector)) {
        // Run middleware
        let canProceed = true;
        for (const mw of this.middleware) {
          if (mw(event, target) === false) {
            canProceed = false;
            break;
          }
        }

        if (canProceed) {
          const handlers = this.handlers.get(key);
          for (const handler of handlers) {
            if (event.cancelBubble) break;
            handler(event, target);
          }
        }
      }
      target = target.parentElement;
    }
  }
}

const pipeline = new DelegationPipeline(document.body);

// Middleware: analytics
pipeline.use((event, target) => {
  console.log(`[Analytics] ${event.type} on ${target.tagName}`);
});

// Middleware: guard
pipeline.use((event, target) => {
  if (target.matches('[data-disabled]')) {
    event.stopPropagation();
    return false;
  }
});

pipeline.on('.btn', 'click', (e, el) => {
  console.log('Button clicked:', el.textContent);
});

// Example 3: Delegated drag-and-drop
class DelegatedDragDrop {
  constructor(container) {
    this.container = container;
    this.dragItem = null;
    this.setup();
  }

  setup() {
    this.container.addEventListener('mousedown', (event) => {
      const handle = event.target.closest('.drag-handle');
      if (!handle) return;

      this.dragItem = handle.closest('.draggable');
      if (!this.dragItem) return;

      this.offsetX = event.clientX - this.dragItem.offsetLeft;
      this.offsetY = event.clientY - this.dragItem.offsetTop;

      this.dragItem.classList.add('dragging');
      this.moveListener = (e) => this.onMouseMove(e);
      this.upListener = () => this.onMouseUp();

      document.addEventListener('mousemove', this.moveListener);
      document.addEventListener('mouseup', this.upListener);
    });

    // Delegated drop zone detection
    this.container.addEventListener('mouseup', (event) => {
      if (!this.dragItem) return;

      const dropZone = event.target.closest('.drop-zone');
      if (dropZone && dropZone !== this.dragItem.parentElement) {
        dropZone.appendChild(this.dragItem);
      }
    });
  }

  onMouseMove(event) {
    if (!this.dragItem) return;
    this.dragItem.style.left = `${event.clientX - this.offsetX}px`;
    this.dragItem.style.top = `${event.clientY - this.offsetY}px`;
  }

  onMouseUp() {
    if (this.dragItem) {
      this.dragItem.classList.remove('dragging');
      this.dragItem.style.left = '';
      this.dragItem.style.top = '';
    }
    this.dragItem = null;
    document.removeEventListener('mousemove', this.moveListener);
    document.removeEventListener('mouseup', this.upListener);
  }
}

// Example 4: Delegation with event history
class DelegationWithHistory {
  constructor(root) {
    this.root = root;
    this.history = [];
    this.maxHistory = 100;
    this.setup();
  }

  setup() {
    this.root.addEventListener('click', (event) => {
      this.record(event);
      this.process(event);
    });
  }

  record(event) {
    const entry = {
      type: event.type,
      target: event.target.matches('[data-track]') ? event.target.dataset.track : event.target.tagName,
      timestamp: Date.now(),
      x: event.clientX,
      y: event.clientY
    };

    this.history.push(entry);
    if (this.history.length > this.maxHistory) {
      this.history.shift();
    }
  }

  process(event) {
    const action = event.target.closest('[data-action]');
    if (action) {
      const handler = this.getHandler(action.dataset.action);
      if (handler) {
        handler(event, action);
      }
    }
  }

  getHandler(action) {
    const handlers = {
      delete: (e, el) => el.closest('.item')?.remove(),
      edit: (e, el) => this.editItem(el.closest('.item')?.dataset.id),
      save: (e, el) => this.saveItem(el.closest('.form')),
      cancel: (e, el) => this.cancelEdit(el.closest('.item')?.dataset.id)
    };
    return handlers[action];
  }

  undo() {
    // Not implemented - would require reverse operations
  }

  getHistory() {
    return [...this.history];
  }
}
```

### Real-World Use Cases

- **Todo Lists**: Add, delete, complete items dynamically without binding each new item
- **Data Tables**: Sort, filter, select rows with thousands of entries using a single listener
- **Dropdown Menus**: Handle clicks on any menu item through a single delegated listener
- **Dynamic Forms**: Form fields added by the user are automatically validated
- **Kanban Boards**: Drag-and-drop cards between columns using delegation
- **Tree Views**: Expand/collapse tree nodes dynamically created from data
- **Navigation Menus**: Handle clicks on menu items, even when menu structure changes
- **Infinite Scroll**: Items loaded on scroll are immediately interactive

### Common Mistakes

```javascript
// Mistake 1: Using event.target without closest() for nested elements
list.addEventListener('click', (event) => {
  // If .item contains child elements, click on child won't match
  if (event.target.matches('.item')) { // May miss clicks on inner elements
    handleItem(event.target);
  }
});
// Better:
list.addEventListener('click', (event) => {
  const item = event.target.closest('.item');
  if (item) handleItem(item);
});

// Mistake 2: Selecting too broad a parent
document.body.addEventListener('click', handleAll); // Too broad
document.querySelector('.specific-section').addEventListener('click', handleSection); // Better

// Mistake 3: Forgetting to check if target exists
parent.addEventListener('click', (event) => {
  const item = event.target.closest('.item');
  item.doSomething(); // Error if item is null (click outside .item)
});

// Mistake 4: Using delegation for events that don't bubble
parent.addEventListener('focus', handleFocus); // Won't bubble!
parent.addEventListener('focusin', handleFocus); // Bubbles! Correct
```

### Best Practices

- Use `event.target.closest()` for matching instead of `event.target.matches()`
- Scope delegation to the nearest common ancestor, not the document
- Document which events are delegated and the selectors used
- Combine multiple event types in a single delegation system
- Use data attributes (`data-action`) for declarative action mapping
- Avoid delegation for events that don't bubble (use `focusin` instead of `focus`)
- Consider performance implications for very high-frequency events (mousemove)

### Performance Considerations

Event delegation uses O(1) memory instead of O(n) for individual listeners. For n elements, delegation uses 1 listener vs n listeners. Matching time is O(d) where d is the depth of the DOM tree (usually negligible). The `closest()` method is highly optimized in modern browsers.

```javascript
// Memory comparison for 1000 elements:
// Individual: 1000 function objects + 1000 event registrations
// Delegation: 1 function object + 1 event registration

// Performance: check if selector matches
// closest() is fast - O(depth) where depth is usually < 20
// For 1000 items in a flat list, depth is typically 2-3 levels
```

### Interview Questions

1. What is event delegation and how does it work?
2. Why does event delegation reduce memory usage?
3. How do you handle dynamically added elements with event delegation?
4. What is the difference between `event.target` and `event.currentTarget` in delegation?
5. Why is `closest()` preferred over `matches()` in delegation?
6. Which events cannot be delegated and why?
7. How would you implement a delegation system?
8. What are the performance trade-offs of delegation vs individual listeners?

### Coding Challenges

1. **Delegation Framework**: Build a reusable event delegation utility library
2. **Action Dispatcher**: Create a data-attribute driven delegation system
3. **Delegation Debugger**: Build a tool that visualizes which delegated handler responds to clicks
4. **Multi-Event Delegator**: Build a system that handles click, dblclick, contextmenu through one delegation

### Related Topics

- [Event Bubbling](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Event_bubbling)
- [Element.closest()](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest)
- [Element.matches()](https://developer.mozilla.org/en-US/docs/Web/API/Element/matches)

## Dynamic Element Handling

### What It Is

Dynamic element handling refers to the ability to manage events for elements that are added to the DOM after the initial page load. These elements are created programmatically through JavaScript (after AJAX calls, user interactions, or data updates). Event delegation is the primary technique for handling events on dynamic elements without manually attaching listeners to each new element.

```javascript
// Dynamic element creation
const newItem = document.createElement('li');
newItem.className = 'item';
newItem.textContent = 'New item';
list.appendChild(newItem);

// With delegation, the new item automatically has click handling
// without any additional code
```

### Why It Is Important

Modern web applications are highly dynamic—content is loaded asynchronously, components mount and unmount, and users interact with constantly changing interfaces. Manual event binding for every new element is impractical and error-prone. Event delegation solves this by handling events for all current and future elements through a common ancestor.

### How It Works Internally

When a new element is added to the DOM, it becomes part of the event propagation chain for its ancestors. The delegated listener on an ancestor already exists and will receive events from the new element when they bubble up. No additional registration is needed because the listener watches for events on the ancestor, and the new element is now a descendant.

```javascript
// 1. Delegated listener attached to parent
parent.addEventListener('click', handler);

// 2. New element added
const child = document.createElement('div');
parent.appendChild(child);

// 3. Click on child fires event
// 4. Event bubbles: child -> ... -> parent
// 5. Parent's listener receives event with event.target === child
// 6. handler checks if child matches selector -> YES
// 7. handler executes
```

### Syntax

```javascript
// Dynamic element handling requires no special syntax
// Just use event delegation as usual:

// This works for BOTH existing AND future .item elements
document.querySelector('.list').addEventListener('click', (event) => {
  const item = event.target.closest('.item');
  if (item) {
    console.log('Item clicked:', item.textContent);
  }
});

// Add elements dynamically
function addItem(text) {
  const li = document.createElement('li');
  li.className = 'item';
  li.textContent = text;
  document.querySelector('.list').appendChild(li);
  // No need to attach event listener!
}
```

### Beginner Examples

```javascript
// Example 1: Dynamic todo items
const todoList = document.querySelector('.todo-list');
const addForm = document.querySelector('.add-form');

// Delegated handler for all todo items (existing and future)
todoList.addEventListener('click', (event) => {
  const item = event.target.closest('.todo-item');
  if (!item) return;

  if (event.target.matches('.delete-btn')) {
    item.remove();
    updateCounter();
  } else if (event.target.matches('.checkbox')) {
    item.classList.toggle('done');
    updateCounter();
  }
});

// Add new todo
addForm.addEventListener('submit', (event) => {
  event.preventDefault();
  const input = addForm.querySelector('input');
  if (!input.value.trim()) return;

  const li = document.createElement('li');
  li.className = 'todo-item';
  li.innerHTML = `
    <input type="checkbox" class="checkbox">
    <span class="text">${escapeHtml(input.value)}</span>
    <button class="delete-btn">×</button>
  `;
  todoList.appendChild(li);
  input.value = '';
});

function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

// Example 2: Dynamic content loaded from API
const feed = document.querySelector('.feed');

feed.addEventListener('click', (event) => {
  const post = event.target.closest('.post');
  if (!post) return;

  if (event.target.matches('.like-btn')) {
    toggleLike(post.dataset.postId);
  } else if (event.target.matches('.comment-btn')) {
    showComments(post.dataset.postId);
  } else if (event.target.matches('.share-btn')) {
    sharePost(post.dataset.postId);
  }
});

async function loadMorePosts() {
  const response = await fetch('/api/posts?page=2');
  const posts = await response.json();

  const fragment = document.createDocumentFragment();
  for (const post of posts) {
    const div = document.createElement('div');
    div.className = 'post';
    div.dataset.postId = post.id;
    div.innerHTML = `
      <h3>${post.title}</h3>
      <p>${post.body}</p>
      <button class="like-btn">Like (${post.likes})</button>
      <button class="comment-btn">Comment</button>
      <button class="share-btn">Share</button>
    `;
    fragment.appendChild(div);
  }
  feed.appendChild(fragment);
}

// Example 3: Dynamic form fields
const formBuilder = document.querySelector('.form-builder');
formBuilder.addEventListener('click', (event) => {
  if (event.target.matches('.remove-field')) {
    event.target.closest('.form-field').remove();
  }
});

function addField(type) {
  const field = document.createElement('div');
  field.className = 'form-field';
  field.innerHTML = `
    <label>${type.charAt(0).toUpperCase() + type.slice(1)}:</label>
    <input type="${type}" class="dynamic-input">
    <button class="remove-field">Remove</button>
  `;
  formBuilder.appendChild(field);
}
```

### Intermediate Examples

```javascript
// Example 1: Dynamic tab system
class DynamicTabs {
  constructor(container) {
    this.container = container;
    this.tabCount = 0;
    this.setupDelegation();
  }

  setupDelegation() {
    this.container.addEventListener('click', (event) => {
      if (event.target.matches('.tab-header')) {
        this.switchTab(event.target.dataset.tabId);
      } else if (event.target.matches('.close-tab')) {
        this.closeTab(event.target.closest('.tab-header').dataset.tabId);
      }
    });
  }

  addTab(title, content) {
    const tabId = `tab-${++this.tabCount}`;

    const header = document.createElement('div');
    header.className = 'tab-header';
    header.dataset.tabId = tabId;
    header.innerHTML = `${title} <span class="close-tab">×</span>`;

    const panel = document.createElement('div');
    panel.className = 'tab-panel';
    panel.id = tabId;
    panel.innerHTML = content;

    this.container.querySelector('.tab-headers').appendChild(header);
    this.container.querySelector('.tab-panels').appendChild(panel);

    this.switchTab(tabId);
  }

  switchTab(tabId) {
    this.container.querySelectorAll('.tab-header').forEach(h => h.classList.remove('active'));
    this.container.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
    this.container.querySelector(`[data-tab-id="${tabId}"]`).classList.add('active');
    document.getElementById(tabId).classList.add('active');
  }

  closeTab(tabId) {
    document.getElementById(tabId).remove();
    this.container.querySelector(`[data-tab-id="${tabId}"]`).remove();

    // Activate first remaining tab
    const firstTab = this.container.querySelector('.tab-header');
    if (firstTab) this.switchTab(firstTab.dataset.tabId);
  }
}

// Example 2: Dynamic data table with sorting
class DynamicDataTable {
  constructor(container) {
    this.container = container;
    this.data = [];
    this.setup();
  }

  setup() {
    this.container.addEventListener('click', (event) => {
      if (event.target.matches('.sort-header')) {
        const key = event.target.dataset.sortKey;
        this.sortBy(key);
      } else if (event.target.matches('.delete-row')) {
        const row = event.target.closest('tr');
        const id = row.dataset.rowId;
        this.deleteRow(id);
      }
    });
  }

  setData(data) {
    this.data = data;
    this.render();
  }

  render() {
    const table = this.container.querySelector('table') || document.createElement('table');
    const headers = Object.keys(this.data[0] || {});

    table.innerHTML = `
      <thead>
        <tr>
          ${headers.map(key => `<th class="sort-header" data-sort-key="${key}">${key} ↕</th>`).join('')}
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        ${this.data.map((row, i) => `
          <tr data-row-id="${i}">
            ${headers.map(key => `<td>${row[key]}</td>`).join('')}
            <td><button class="delete-row">Delete</button></td>
          </tr>
        `).join('')}
      </tbody>
    `;

    this.container.appendChild(table);
  }

  sortBy(key) {
    this.data.sort((a, b) => String(a[key]).localeCompare(String(b[key])));
    this.render();
  }

  deleteRow(id) {
    this.data.splice(id, 1);
    this.render();
  }

  addRow(row) {
    this.data.push(row);
    this.render();
  }
}

// Example 3: Dynamic infinite scroll with delegation
class InfiniteScrollList {
  constructor(container, options) {
    this.container = container;
    this.loadMore = options.loadMore;
    this.page = 0;
    this.loading = false;

    this.setupListener();
  }

  setupListener() {
    // Delegated click for items
    this.container.addEventListener('click', (event) => {
      const item = event.target.closest('.scroll-item');
      if (item) {
        this.onItemClick(item.dataset.itemId);
      }
    });

    // Scroll detection (not delegated, but using a single listener)
    window.addEventListener('scroll', () => {
      if (this.loading) return;

      const rect = this.container.getBoundingClientRect();
      if (rect.bottom <= window.innerHeight + 100) {
        this.loadNextPage();
      }
    }, { passive: true });
  }

  async loadNextPage() {
    this.loading = true;
    this.page++;
    const items = await this.loadMore(this.page);
    this.appendItems(items);
    this.loading = false;
  }

  appendItems(items) {
    const fragment = document.createDocumentFragment();
    for (const item of items) {
      const div = document.createElement('div');
      div.className = 'scroll-item';
      div.dataset.itemId = item.id;
      div.textContent = item.title;
      fragment.appendChild(div);
    }
    this.container.appendChild(fragment);
  }

  onItemClick(id) {
    console.log('Item clicked:', id);
  }
}
```

### Advanced Examples

```javascript
// Example 1: Dynamic component system with delegation
class DynamicComponentSystem {
  constructor(root) {
    this.root = root;
    this.components = new Map();
    this.setup();
  }

  setup() {
    // Single delegation for all component types
    this.root.addEventListener('click', (event) => {
      const componentEl = event.target.closest('[data-component]');
      if (!componentEl) return;

      const componentName = componentEl.dataset.component;
      const component = this.components.get(componentName);
      if (component) {
        component.handleEvent(event, componentEl);
      }
    });
  }

  register(name, component) {
    this.components.set(name, component);
  }

  createComponent(name, data) {
    const component = this.components.get(name);
    if (!component) throw new Error(`Unknown component: ${name}`);

    const el = component.render(data);
    el.dataset.component = name;
    this.root.appendChild(el);
    return el;
  }
}

// Define components
const components = new DynamicComponentSystem(document.querySelector('.app-container'));

components.register('card', {
  render({ title, body }) {
    const div = document.createElement('div');
    div.className = 'card';
    div.innerHTML = `
      <h3 class="card-title">${title}</h3>
      <p class="card-body">${body}</p>
      <button class="card-btn" data-action="expand">Expand</button>
      <button class="card-btn" data-action="delete">Delete</button>
    `;
    return div;
  },
  handleEvent(event, element) {
    const action = event.target.closest('[data-action]');
    if (!action) return;

    switch (action.dataset.action) {
      case 'expand':
        element.querySelector('.card-body').classList.toggle('expanded');
        break;
      case 'delete':
        element.remove();
        break;
    }
  }
});

// Dynamic creation
components.createComponent('card', {
  title: 'Dynamic Card',
  body: 'This card was created dynamically and handles events via delegation'
});

// Example 2: Dynamic context menu on dynamic elements
class DynamicContextMenu {
  constructor(root) {
    this.root = root;
    this.menu = this.createMenu();
    this.setup();
  }

  createMenu() {
    const menu = document.createElement('div');
    menu.className = 'context-menu';
    menu.style.display = 'none';
    menu.innerHTML = `
      <ul>
        <li data-action="edit">Edit</li>
        <li data-action="copy">Copy</li>
        <li data-action="delete">Delete</li>
        <li data-action="duplicate">Duplicate</li>
      </ul>
    `;
    document.body.appendChild(menu);
    return menu;
  }

  setup() {
    // Delegated context menu trigger
    this.root.addEventListener('contextmenu', (event) => {
      const target = event.target.closest('[data-context]');
      if (!target) return;

      event.preventDefault();
      this.show(target, event.clientX, event.clientY);
    });

    // Menu item clicks (also delegated to document)
    document.addEventListener('click', (event) => {
      const action = event.target.closest('[data-action]');
      if (action && this.currentTarget) {
        this.execute(action.dataset.action, this.currentTarget);
        this.hide();
      } else {
        this.hide();
      }
    });
  }

  show(target, x, y) {
    this.currentTarget = target;
    this.menu.style.display = 'block';
    this.menu.style.left = `${x}px`;
    this.menu.style.top = `${y}px`;
  }

  hide() {
    this.menu.style.display = 'none';
    this.currentTarget = null;
  }

  execute(action, target) {
    switch (action) {
      case 'edit':
        this.edit(target);
        break;
      case 'copy':
        this.copy(target);
        break;
      case 'delete':
        target.remove();
        break;
      case 'duplicate':
        const clone = target.cloneNode(true);
        target.parentNode.appendChild(clone);
        break;
    }
  }

  edit(target) {
    const originalText = target.textContent;
    const input = document.createElement('input');
    input.value = originalText;
    target.textContent = '';
    target.appendChild(input);
    input.focus();
    input.addEventListener('blur', () => {
      target.textContent = input.value;
    });
  }

  copy(target) {
    navigator.clipboard.writeText(target.textContent);
  }
}

const ctxMenu = new DynamicContextMenu(document.querySelector('.content-area'));

// Example 3: Dynamic list with undo support
class DynamicUndoList {
  constructor(container) {
    this.container = container;
    this.history = [];
    this.setup();
  }

  setup() {
    this.container.addEventListener('click', (event) => {
      const item = event.target.closest('[data-item-id]');
      if (!item) return;

      if (event.target.matches('.remove-item')) {
        this.removeItem(item);
      } else if (event.target.matches('.edit-item')) {
        this.editItem(item);
      }
    });
  }

  addItem(text) {
    const id = Date.now();
    const div = document.createElement('div');
    div.className = 'list-item';
    div.dataset.itemId = id;
    div.innerHTML = `
      <span class="item-text">${text}</span>
      <button class="edit-item">Edit</button>
      <button class="remove-item">×</button>
    `;
    this.container.appendChild(div);

    this.history.push({
      type: 'add',
      element: div,
      timestamp: id
    });
  }

  removeItem(item) {
    const text = item.querySelector('.item-text').textContent;
    const nextSibling = item.nextElementSibling;

    this.history.push({
      type: 'remove',
      text,
      parent: item.parentNode,
      nextSibling,
      element: item
    });

    item.remove();
  }

  editItem(item) {
    const textEl = item.querySelector('.item-text');
    const currentText = textEl.textContent;

    const input = document.createElement('input');
    input.type = 'text';
    input.value = currentText;
    input.className = 'edit-input';

    textEl.replaceWith(input);
    input.focus();

    const saveEdit = () => {
      const newText = input.value;
      const newSpan = document.createElement('span');
      newSpan.className = 'item-text';
      newSpan.textContent = newText;
      input.replaceWith(newSpan);

      this.history.push({
        type: 'edit',
        element: item,
        oldText: currentText,
        newText
      });
    };

    input.addEventListener('blur', saveEdit);
    input.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') {
        input.blur();
      } else if (e.key === 'Escape') {
        input.value = currentText;
        input.blur();
      }
    });
  }

  undo() {
    const action = this.history.pop();
    if (!action) return;

    switch (action.type) {
      case 'add':
        action.element.remove();
        break;
      case 'remove':
        if (action.nextSibling) {
          action.parent.insertBefore(action.element, action.nextSibling);
        } else {
          action.parent.appendChild(action.element);
        }
        break;
      case 'edit':
        const span = document.createElement('span');
        span.className = 'item-text';
        span.textContent = action.oldText;
        action.element.querySelector('.item-text').replaceWith(span);
        break;
    }
  }
}
```

### Real-World Use Cases

- **Chat Applications**: Messages appear dynamically; all need click, hover, reply handling
- **Social Media Feeds**: New posts loaded from API need interaction buttons
- **E-commerce Product Grid**: Products loaded on scroll or filter change need add-to-cart buttons
- **Dashboard Widgets**: Users add/remove widgets; all need close, resize, config handlers
- **Comment Threads**: Nested comments loaded on demand need reply, vote, report buttons
- **Email Clients**: Emails loaded in batches need read, archive, delete, flag actions
- **Project Management Boards**: Cards moved between columns, new cards added dynamically

### Common Mistakes

```javascript
// Mistake 1: Trying to bind events to elements that don't exist yet
dynamicContainer.addEventListener('load', () => {
  document.querySelectorAll('.dynamic-item').forEach(item => {
    // These items might not exist yet if loaded async
    item.addEventListener('click', handler); // May miss some
  });
});
// Fix: Use delegation instead

// Mistake 2: Rebinding events after every dynamic addition
function addItem() {
  const item = createItem();
  list.appendChild(item);
  bindEvents(); // Don't do this - use delegation!
}

// Mistake 3: Using innerHTML which destroys existing event listeners
container.innerHTML = newContent; // If you used direct binding, listeners are lost
// Delegation survives innerHTML changes because listener is on parent

// Mistake 4: Assuming delegation works for non-bubbling events
dynamicContainer.addEventListener('focus', handler); // Won't work for new elements!
// Use focusin instead
```

### Best Practices

- Always use delegation for elements that are dynamically created
- Avoid mixing direct binding and delegation for the same events
- Use `closest()` to handle nested dynamic elements correctly
- Scope delegation to the nearest dynamic container (not document.body)
- Combine delegation with `MutationObserver` for complex dynamic scenarios
- Clean up data structures when dynamic elements are removed
- Use `WeakMap` to store data associated with dynamic elements (avoids memory leaks)

### Performance Considerations

Delegation for dynamic elements has the same performance profile as for static elements. The key advantage is that no additional work is needed when elements are added or removed. There is zero per-element setup cost, making it ideal for large lists and frequently changing DOM structures.

```javascript
// O(1) per element added - simply append to DOM
// No listener registration, no closure creation

// For 10,000 dynamic items:
// Direct binding: 10,000 closures + 10,000 registration calls
// Delegation: 0 additional work
```

### Interview Questions

1. Why can't you attach events directly to elements that will be created in the future?
2. How does event delegation handle dynamically created elements?
3. What happens if you mix delegation with direct event binding?
4. How would you handle events on elements loaded via AJAX?
5. What is the `closest()` method and why is it important for dynamic elements?
6. How do you handle non-bubbling events on dynamic elements?
7. How would you implement a dynamic form where fields can be added and removed?
8. What are the memory implications of delegation vs individual binding for dynamic elements?

### Coding Challenges

1. **Dynamic Form Builder**: Create a form system where fields are added dynamically with validation
2. **Live Search Results**: Build a search that displays results with delegated click handlers
3. **Dynamic Dashboard**: Create a dashboard widget system with add/remove/refresh capabilities
4. **Reusable List Component**: Build a generic list component that handles items added at any time

### Related Topics

- [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)
- [Element.closest()](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest)
- [Event Bubbling](https://developer.mozilla.org/en-US/docs/Web/API/Event/bubbles)

## Performance Benefits

### What It Is

Event delegation provides significant performance advantages over individual event binding, particularly in applications with many interactive elements. The benefits include reduced memory consumption, faster setup time, lower CPU usage for event registration, and automatic scaling with dynamic content.

```javascript
// Performance comparison:
// Individual binding: O(n) memory and O(n) setup time
// Event delegation: O(1) memory and O(1) setup time (regardless of element count)
```

### Why It Is Important

Web applications increasingly handle large amounts of interactive content. A data table with 10,000 rows, each with multiple buttons, would require 30,000+ individual event listeners. This consumes significant memory and slows down initial rendering. Event delegation reduces this to a single listener, making applications more responsive and memory-efficient.

### How It Works Internally

Each event listener registered with `addEventListener` creates an internal data structure in the browser's C++ layer. These structures consume memory for function references, configuration, and closure scope references. When events fire, the browser must iterate through all registered listeners. With delegation, one listener replaces potentially thousands, reducing both memory and dispatch time.

```javascript
// Internal browser data per listener (conceptual):
// {
//   type: 'click',
//   handler: Function,
//   capture: false,
//   once: false,
//   passive: false,
//   element: Element,     // Reference keeps element alive
//   closureEnvironment: { ... } // Closure scope
// }
```

### Syntax

```javascript
// Performance comparison setup
function measureSetup(elements, useDelegation) {
  const start = performance.now();

  if (useDelegation) {
    elements[0].parentElement.addEventListener('click', handler);
  } else {
    elements.forEach(el => el.addEventListener('click', handler));
  }

  return performance.now() - start;
}

function measureDispatch(elements, useDelegation) {
  const start = performance.now();

  if (useDelegation) {
    elements[0].parentElement.dispatchEvent(new Event('click'));
  } else {
    elements.forEach(el => el.dispatchEvent(new Event('click')));
  }

  return performance.now() - start;
}
```

### Beginner Examples

```javascript
// Example 1: Memory comparison
// Bad: 10,000 listeners for 10,000 items
document.querySelectorAll('.item').forEach((item, i) => {
  item.addEventListener('click', () => {
    console.log('Item', i, 'clicked');
  });
});

// Good: 1 listener for 10,000 items
document.querySelector('.list').addEventListener('click', (event) => {
  const item = event.target.closest('.item');
  if (item) {
    console.log('Item', item.dataset.index, 'clicked');
  }
});

// Example 2: Setup time comparison
function createLargeList(count) {
  const container = document.getElementById('perf-test');
  container.innerHTML = '';

  const list = document.createElement('div');
  list.className = 'list';

  for (let i = 0; i < count; i++) {
    const item = document.createElement('div');
    item.className = 'item';
    item.dataset.index = i;
    item.textContent = `Item ${i}`;
    list.appendChild(item);
  }

  const start = performance.now();

  // Test: comment out one approach
  // Approach A: Individual listeners
  list.querySelectorAll('.item').forEach(el => {
    el.addEventListener('click', () => {});
  });

  // Approach B: Delegated listener
  // list.addEventListener('click', (e) => {
  //   const item = e.target.closest('.item');
  // });

  return performance.now() - start;
}

// console.log('Setup time (ms):', createLargeList(10000));
```

### Intermediate Examples

```javascript
// Example 1: Performance measurement utility
class EventPerformanceBenchmark {
  constructor() {
    this.results = [];
  }

  benchmark({
    elementCount = 1000,
    eventType = 'click',
    delegation = false
  } = {}) {
    // Create test elements
    const container = document.createElement('div');
    container.id = 'bench-container';
    document.body.appendChild(container);

    for (let i = 0; i < elementCount; i++) {
      const el = document.createElement('div');
      el.className = 'bench-item';
      el.dataset.id = i;
      container.appendChild(el);
    }

    const items = container.querySelectorAll('.bench-item');

    // Measure setup time
    const setupStart = performance.now();

    if (delegation) {
      container.addEventListener(eventType, (e) => {
        const item = e.target.closest('.bench-item');
        if (item) {
          window.__benchResult = item.dataset.id;
        }
      });
    } else {
      items.forEach(el => {
        el.addEventListener(eventType, () => {
          window.__benchResult = el.dataset.id;
        });
      });
    }

    const setupTime = performance.now() - setupStart;

    // Measure dispatch time
    const dispatchStart = performance.now();

    if (delegation) {
      container.dispatchEvent(new Event(eventType));
    } else {
      items.forEach(el => el.dispatchEvent(new Event(eventType)));
    }

    const dispatchTime = performance.now() - dispatchStart;

    // Measure memory (approximate)
    const memoryEstimate = delegation
      ? 1 // one listener
      : elementCount; // one per element

    // Cleanup
    container.remove();
    this.results.push({
      elementCount,
      delegation,
      setupTime: setupTime.toFixed(2),
      dispatchTime: dispatchTime.toFixed(2),
      listeners: delegation ? 1 : elementCount,
      memoryEstimate: `${memoryEstimate}x units`
    });

    return this.results[this.results.length - 1];
  }

  compare(count = 10000) {
    const direct = this.benchmark({ elementCount: count, delegation: false });
    const delegated = this.benchmark({ elementCount: count, delegation: true });

    console.table({
      'Direct Binding': direct,
      'Event Delegation': delegated
    });
  }
}

// const bench = new EventPerformanceBenchmark();
// bench.compare(10000);

// Example 2: Scroll event delegation optimization
class OptimizedScrollHandler {
  constructor() {
    this.handlers = new Map();
    this.setupDelegation();
  }

  setupDelegation() {
    let ticking = false;

    window.addEventListener('scroll', (event) => {
      if (!ticking) {
        requestAnimationFrame(() => {
          // Run all registered handlers
          for (const [, handler] of this.handlers) {
            handler(event);
          }
          ticking = false;
        });
        ticking = true;
      }
    }, { passive: true });
  }

  register(id, handler) {
    this.handlers.set(id, handler);
  }

  unregister(id) {
    this.handlers.delete(id);
  }
}

const scrollHandler = new OptimizedScrollHandler();
scrollHandler.register('parallax', () => updateParallax());
scrollHandler.register('sticky-header', () => updateStickyHeader());

// Example 3: Input event delegation for many fields
function setupDelegatedValidation() {
  const form = document.querySelector('form');

  // Single input handler for all fields
  form.addEventListener('input', (event) => {
    const field = event.target;
    if (!field.matches('[data-validate]')) return;

    const rules = field.dataset.validate.split('|');
    for (const rule of rules) {
      const [name, param] = rule.split(':');
      const error = validateField(name, field.value, param);
      showFieldError(field, error);
    }
  }, { passive: true }); // passive: true since we don't preventDefault on input

  // Single blur handler for all fields
  form.addEventListener('focusout', (event) => {
    const field = event.target;
    if (!field.matches('[data-validate]')) return;

    // Touch the field to show validation state
    field.classList.add('touched');
  });
}

function validateField(rule, value, param) {
  switch (rule) {
    case 'required': return value.trim() ? '' : 'Required';
    case 'min': return value.length >= parseInt(param) ? '' : `Min ${param} chars`;
    case 'email': return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? '' : 'Invalid email';
    default: return '';
  }
}
```

### Advanced Examples

```javascript
// Example 1: Event delegation cost analyzer
class DelegationCostAnalyzer {
  constructor() {
    this.originalAdd = EventTarget.prototype.addEventListener;
    this.listenerCount = 0;
  }

  start() {
    this.listenerCount = 0;
    EventTarget.prototype.addEventListener = (...args) => {
      this.listenerCount++;
      return this.originalAdd.apply(this, args);
    };
  }

  stop() {
    EventTarget.prototype.addEventListener = this.originalAdd;
    return this.analyze();
  }

  analyze(referenceElement) {
    const directCost = this.calculateDirectCost();
    const delegationCost = this.calculateDelegationCost(referenceElement);

    return {
      currentListeners: this.listenerCount,
      estimatedDirectCost: directCost,
      estimatedDelegationCost: delegationCost,
      savings: ((directCost - delegationCost) / directCost * 100).toFixed(1) + '%'
    };
  }

  calculateDirectCost() {
    // Assume each listener consumes ~1KB for closure + internal structures
    return this.listenerCount * 1024; // bytes
  }

  calculateDelegationCost(referenceElement) {
    if (!referenceElement) return 0;

    // Delegation uses 1 listener per event type per container
    const containerCount = document.querySelectorAll('[data-delegation-container]').length || 1;
    const eventTypes = new Set();

    document.querySelectorAll('*').forEach(el => {
      const attrs = el.getAttributeNames().filter(a => a.startsWith('data-event-'));
      attrs.forEach(a => eventTypes.add(a.replace('data-event-', '')));
    });

    return (eventTypes.size || 1) * containerCount * 1024;
  }
}

// Example 2: Adaptive delegation switching
class AdaptiveDelegation {
  constructor(container, selector, eventType, handler) {
    this.container = container;
    this.selector = selector;
    this.eventType = eventType;
    this.handler = handler;
    this.mode = 'delegation'; // or 'direct'
    this.threshold = 50; // elements
    this.observer = null;
  }

  start() {
    this.container.addEventListener(this.eventType, this.delegatedHandler.bind(this));
    this.setupObserver();
  }

  delegatedHandler(event) {
    const target = event.target.closest(this.selector);
    if (target) {
      this.handler(event, target);
    }
  }

  setupObserver() {
    this.observer = new MutationObserver(() => {
      this.evaluateStrategy();
    });

    this.observer.observe(this.container, {
      childList: true,
      subtree: true
    });
  }

  evaluateStrategy() {
    const count = this.container.querySelectorAll(this.selector).length;

    if (count > this.threshold && this.mode === 'direct') {
      this.switchToDelegation();
    } else if (count <= this.threshold && this.mode === 'delegation') {
      this.switchToDirect();
    }
  }

  switchToDelegation() {
    this.container.removeEventListener(this.eventType, this.directHandler);
    this.container.addEventListener(this.eventType, this.delegatedHandler.bind(this), this.eventType);
    this.mode = 'delegation';
    console.log(`Switched to delegation (${this.container.querySelectorAll(this.selector).length} elements)`);
  }

  switchToDirect() {
    this.container.removeEventListener(this.eventType, this.delegatedHandler.bind(this));
    this.container.addEventListener(this.eventType, this.directHandler);
    this.mode = 'direct';
    console.log(`Switched to direct binding (${this.container.querySelectorAll(this.selector).length} elements)`);
  }

  stop() {
    if (this.observer) this.observer.disconnect();
  }
}

// Example 3: Real-time delegation performance dashboard
class DelegationDashboard {
  constructor() {
    this.stats = {
      totalListeners: 0,
      delegationListeners: 0,
      directListeners: 0,
      estimatedMemoryKB: 0
    };
  }

  scan() {
    this.stats.totalListeners = this.countListeners();
    this.estimateSavings();
    this.render();
  }

  countListeners() {
    // Cannot directly count event listeners from JS
    // This is a heuristic based on common patterns
    const allElements = document.querySelectorAll('*');

    // Count elements with inline handlers (onclick, etc.)
    let inlineCount = 0;
    allElements.forEach(el => {
      for (const attr of el.attributes) {
        if (attr.name.startsWith('on')) inlineCount++;
      }
    });

    return {
      estimated: inlineCount,
      note: 'Listeners added via addEventListener cannot be counted directly'
    };
  }

  estimateSavings() {
    const elements = document.querySelectorAll('.item, .row, .cell, li');
    const delegationParents = document.querySelectorAll('[data-delegation]');

    const directEstimate = elements.length * 1; // 1 listener per element
    const delegationEstimate = delegationParents.length || 1;

    this.stats.estimatedMemoryKB = {
      direct: (directEstimate * 1024) / 1024,
      delegation: (delegationEstimate * 1024) / 1024,
      savings: `${(((directEstimate - delegationEstimate) / directEstimate) * 100).toFixed(1)}%`
    };
  }

  render() {
    console.table({
      'Total DOM elements': document.querySelectorAll('*').length,
      'Interactive elements': document.querySelectorAll('.item, .row, .cell, button, a').length,
      'Direct binding estimate': `${this.stats.estimatedMemoryKB.direct} KB`,
      'Delegation estimate': `${this.stats.estimatedMemoryKB.delegation} KB`,
      'Memory savings': this.stats.estimatedMemoryKB.savings,
      'Recommendation': 'Use event delegation for lists with >10 items'
    });
  }
}

// Usage: new DelegationDashboard().scan();
```

### Real-World Use Cases

- **Large Data Tables**: A table with 1,000 rows and 5 action columns needs 5,000+ listeners; delegation uses 5
- **Chat Applications**: 10,000+ messages with reply/react/delete buttons per message
- **Social Feeds**: Infinite scroll feeds with thousands of post cards
- **Dashboard Grids**: Widgets with toolbars, each with 10+ actions
- **Email Clients**: Thousands of email entries with subject, sender, date, attachment, star, checkbox interactions
- **Online Code Editors**: Syntax highlighted lines (thousands) with clickable line numbers

### Common Mistakes

```javascript
// Mistake 1: Thinking delegation is always faster for dispatch
// Individual dispatch: O(1) - listener is called directly
// Delegation dispatch: O(depth) event bubbles up + O(1) matching
// For very small lists (< 5 items), individual may be slightly faster

// Mistake 2: Delegating every event type unnecessarily
// Only delegate events that are used by many elements
// Keyboard events on individual inputs are fine without delegation

// Mistake 3: Over-delegating to document.body
document.body.addEventListener('click', handler); // Every click on page runs through this
// Better: Delegate to the nearest container

// Mistake 4: Not removing delegation when container is removed
// If the delegation container is removed, the listener is GC'd automatically
// But if you keep a reference, it persists
```

### Best Practices

- Use delegation for elements that are numerous (10+) or dynamic
- Delegate to the closest common ancestor, not the document
- Use `passive: true` for scroll/touch delegated events
- Be selective about what you delegate—not every event needs delegation
- Measure performance before optimizing; delegation is not always faster for tiny lists
- Combine delegation with debouncing for high-frequency events (mousemove, scroll)

### Performance Considerations

Metrics for 10,000 interactive elements:
- **Memory**: Individual = ~10MB (10,000 closures + 10,000 listener objects). Delegation = ~1KB
- **Setup Time**: Individual = 50-100ms. Delegation = <1ms
- **Dispatch Time**: Individual = <0.1ms per listener. Delegation = <0.5ms total

```javascript
// Measured performance (approximate, varies by browser/system)
// Setup: 10,000 elements
const setupTimes = {
  individual: '50-100ms',
  delegation: '< 1ms',
  improvement: '50x-100x'
};

// Memory: 10,000 listeners
const memoryUsage = {
  individual: '~10 MB',
  delegation: '~1 KB',
  improvement: '10,000x'
};
```

### Interview Questions

1. How much memory does a single event listener consume?
2. What is the Big O complexity of event delegation vs individual?
3. When would individual listeners be better than delegation?
4. How does delegation affect event dispatch time?
5. What is the memory impact of closures in event handlers?
6. How would you benchmark event delegation performance?
7. What happens to listener memory when an element is removed from the DOM?
8. How does passive: true affect delegated scroll performance?

### Coding Challenges

1. **Performance Benchmark**: Create a script that benchmarks delegation vs direct binding for various list sizes
2. **Adaptive Strategy**: Build a system that switches between delegation and direct binding based on element count
3. **Memory Profiler**: Create a tool that estimates memory usage of event listeners on a page
4. **Optimized Form Handler**: Build a form validation system with optimal event strategy

### Related Topics

- [Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)
- [Browser Rendering Performance](https://developers.google.com/web/fundamentals/performance/rendering)
- [Passive Event Listeners](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
