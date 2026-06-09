# Event Bubbling - Bubbling phase, capturing phase, stopPropagation

## Introduction

Event propagation is the mechanism by which events travel through the DOM tree. When an event occurs on an element, it doesn't just fire on that single element—it goes through three phases: capturing (from the document root down to the target), target (on the element itself), and bubbling (from the target back up to the document root). Understanding this propagation model is essential for building complex event handling systems, debugging unexpected behavior, and implementing patterns like event delegation.

The concept of event bubbling is often surprising to developers new to the DOM, but it is a powerful feature that enables efficient event management. Instead of attaching listeners to every element, you can attach a single listener to a parent and handle events from all children through bubbling.

## Bubbling Phase

### What It Is

The bubbling phase is the third phase of event propagation, where the event travels from the target element up through its ancestors to the document root. Almost all events bubble (with notable exceptions like `focus`, `blur`, `scroll`, `mouseenter`, `mouseleave`, and `load`). During bubbling, event listeners attached with `capture: false` (the default) are triggered on each ancestor in order from the target up to the document.

```javascript
// HTML: <div id="grandparent"><div id="parent"><div id="child">Click me</div></div></div>

document.getElementById('grandparent').addEventListener('click', () => {
  console.log('Grandparent (bubbling)');
});
document.getElementById('parent').addEventListener('click', () => {
  console.log('Parent (bubbling)');
});
document.getElementById('child').addEventListener('click', () => {
  console.log('Child (bubbling)');
});

// Click on child outputs:
// Child (bubbling)
// Parent (bubbling)
// Grandparent (bubbling)
```

### Why It Is Important

Bubbling enables event delegation—attaching a single listener to a parent element to handle events from many children. It also allows parent elements to respond to events triggered by their descendants, which is crucial for patterns like clicking outside a modal to close it, or tracking all clicks within a section for analytics.

### How It Works Internally

After the event fires on the target element, the browser walks up the DOM tree from `target.parentNode` to `document`, invoking matching event listeners at each level. The browser maintains the event's propagation state and stops traversal if `stopPropagation()` is called.

```javascript
// Conceptual bubbling algorithm
function bubbleEvent(event) {
  let current = event.target.parentNode;

  while (current && !event.propagationStopped) {
    // Invoke all capture: false listeners on current node
    const listeners = getListeners(current, event.type, { capture: false });
    for (const listener of listeners) {
      listener(event);
      if (event.immediatePropagationStopped) break;
    }
    current = current.parentNode;
  }
}
```

### Syntax

```javascript
// Bubbling is the default behavior (capture: false is default)
element.addEventListener('click', handler); // capture defaults to false = bubbling phase
element.addEventListener('click', handler, false); // Explicit bubbling phase

// Events that bubble:
// click, dblclick, mousedown, mouseup, keydown, keyup, submit, change, input
// focusin, focusout, dragstart, drag, dragenter, dragleave, dragover, drop

// Events that do NOT bubble:
// focus, blur, scroll (on non-window), mouseenter, mouseleave, load, unload
// resize (on elements), toggle (details element)
```

### Beginner Examples

```javascript
// Example 1: Basic bubbling demonstration
const outer = document.querySelector('.outer');
const inner = document.querySelector('.inner');
const button = document.querySelector('.inner button');

outer.addEventListener('click', () => console.log('Outer clicked'));
inner.addEventListener('click', () => console.log('Inner clicked'));
button.addEventListener('click', () => console.log('Button clicked'));

// Clicking the button logs:
// Button clicked
// Inner clicked
// Outer clicked

// Example 2: Checking if an event bubbles
const testEvent = new Event('test', { bubbles: false });
console.log(testEvent.bubbles); // false

const bubblingEvent = new Event('test', { bubbles: true });
console.log(bubblingEvent.bubbles); // true

// Example 3: Non-bubbling events
const div = document.querySelector('div');
const input = document.querySelector('input');

input.addEventListener('focus', () => console.log('Input focused'));
div.addEventListener('focus', () => console.log('Div focus (will not fire)')); // Won't fire

// To handle non-bubbling focus, use focusin:
div.addEventListener('focusin', () => console.log('Div focusin (will fire)'));

// Example 4: Event phase detection
element.addEventListener('click', (e) => {
  if (e.eventPhase === Event.BUBBLING_PHASE) {
    console.log('Handler called during bubbling phase');
  }
  console.log('Phase:', e.eventPhase);
  // 1 = CAPTURING_PHASE, 2 = AT_TARGET, 3 = BUBBLING_PHASE
});
```

### Intermediate Examples

```javascript
// Example 1: Click outside to close
document.addEventListener('click', (e) => {
  const modal = document.getElementById('modal');
  if (modal && !modal.contains(e.target)) {
    modal.classList.remove('open');
  }
});

// Example 2: Nested event handling with state
let clickCount = 0;

document.querySelector('.card').addEventListener('click', (e) => {
  clickCount++;

  // Track all clicks in the card area
  console.log(`Click ${clickCount} at`, e.clientX, e.clientY);

  if (e.target.classList.contains('delete-btn')) {
    const item = e.target.closest('.item');
    item.remove();
    console.log('Item deleted');
  } else if (e.target.classList.contains('edit-btn')) {
    const item = e.target.closest('.item');
    editItem(item.dataset.id);
  }
});

// Example 3: Event path inspection
document.addEventListener('click', (e) => {
  const path = e.composedPath();
  console.log('Event path:', path.map(el =>
    el.tagName ? `<${el.tagName.toLowerCase()}${el.id ? '#' + el.id : ''}>` : el.nodeName
  ));

  // Find specific element in path
  const form = path.find(el => el.tagName === 'FORM');
  if (form) {
    console.log('Clicked inside form:', form.id || 'unnamed');
  }
});

// Example 4: Custom event with bubbling
function dispatchCustomEvent(name, detail, element = document) {
  const event = new CustomEvent(name, {
    detail,
    bubbles: true,
    cancelable: true
  });
  element.dispatchEvent(event);
}

document.addEventListener('app:action', (e) => {
  console.log(`Action: ${e.detail.type}`, e.detail);
});

// Widget dispatches the event
dispatchCustomEvent('app:action', {
  type: 'user-login',
  userId: 123,
  timestamp: Date.now()
});
```

### Advanced Examples

```javascript
// Example 1: Bubbling control for nested forms
class FormManager {
  constructor(form) {
    this.form = form;
    this.setup();
  }

  setup() {
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.handleSubmit(e);
    }, true); // Capture phase to intercept before child forms

    // Handle nested form submissions
    this.form.addEventListener('submit', (e) => {
      // This runs in bubbling phase
      if (e.target !== this.form) {
        // A child form was submitted, prevent outer form submission
        console.log('Child form submitted, not bubbling up');
        e.stopPropagation();
      }
    });
  }

  handleSubmit(e) {
    console.log('Main form submitted');
    const data = new FormData(this.form);
    // Submit data...
  }
}

// Example 2: Event propagation visualizer
class PropagationVisualizer {
  constructor() {
    this.colors = ['#ff6b6b', '#feca57', '#48dbfb', '#ff9ff3', '#54a0ff'];
    this.highlights = new Map();
  }

  start() {
    document.addEventListener('click', (e) => this.visualize(e), true); // Capture
    document.addEventListener('click', (e) => this.visualize(e), false); // Bubble
  }

  visualize(event) {
    const el = event.currentTarget;
    const phase = event.eventPhase === Event.CAPTURING_PHASE ? 'CAPTURE' : 'BUBBLE';

    // Highlight the element
    const originalBg = el.style.backgroundColor;
    const color = this.colors[Math.floor(Math.random() * this.colors.length)];

    el.style.backgroundColor = color;
    el.style.transition = 'background-color 0.5s';

    console.log(`[${phase}] <${el.tagName.toLowerCase()}${el.id ? '#' + el.id : ''}>`);

    setTimeout(() => {
      el.style.backgroundColor = originalBg;
    }, 500);
  }
}

// new PropagationVisualizer().start();

// Example 3: Bubbling-aware event bus
class BubblingEventBus {
  constructor(root = document) {
    this.root = root;
    this.eventTypes = new Map();
  }

  register(type, options = {}) {
    if (this.eventTypes.has(type)) return;

    this.eventTypes.set(type, options);
    this.root.addEventListener(type, (e) => {
      // Handle bubbling events
      this.dispatch(type, e);
    });
  }

  dispatch(type, originalEvent) {
    const options = this.eventTypes.get(type);
    const handlers = this.getHandlers(type, originalEvent);

    for (const handler of handlers) {
      if (originalEvent.cancelBubble) break;

      // Call handler with context of the element it was registered on
      handler.call(handler.element, originalEvent);
    }
  }

  getHandlers(type, event) {
    const handlers = [];
    let current = event.target;

    while (current && current !== this.root.parentNode) {
      // Find handlers for this element
      for (const handler of (current._eventHandlers?.[type] || [])) {
        handlers.push(handler);
      }
      current = current.parentElement;
    }

    return handlers;
  }
}

// Example 4: Bubbling with Web Components and Shadow DOM
class BubblingWidget extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <div>
        <slot></slot>
        <button>Shadow Button</button>
      </div>
    `;

    this.shadowRoot.querySelector('button').addEventListener('click', (e) => {
      console.log('Shadow button clicked');
      // Event retargeting: e.target will be the shadow host for outside listeners
    });
  }
}

customElements.define('bubbling-widget', BubblingWidget);

// Outside listener
document.addEventListener('click', (e) => {
  // e.target might be the custom element (shadow host) due to retargeting
  console.log('Document click target:', e.target);
});

// To get actual clicked element inside shadow DOM:
document.addEventListener('click', (e) => {
  const composedPath = e.composedPath();
  console.log('Composed path:', composedPath);
  // First element is the actual clicked element inside shadow DOM
});
```

### Real-World Use Cases

- **Modal Backdrop**: Click outside modal content to close (check if click target is the backdrop)
- **Accordion Panels**: Click on the panel header to toggle, with bubbling delegating to the container
- **Table Row Actions**: Click on any cell or button within a row to trigger row-level action
- **Dropdown Menus**: Click anywhere in the document to close an open dropdown
- **Form Validation**: Show validation errors at the form level when any field fails
- **Analytics Tracking**: Track all clicks within a section using a single listener
- **Nested UI Components**: Handle events from child components through bubbling

### Common Mistakes

```javascript
// Mistake 1: Assuming all events bubble
div.addEventListener('scroll', handler); // Won't bubble if on a div
div.addEventListener('focus', handler); // Won't bubble

// Mistake 2: Stopping propagation unnecessarily (breaks event delegation)
child.addEventListener('click', (e) => {
  e.stopPropagation(); // Parent's delegated listener won't see this click
  doSomething();
});

// Mistake 3: Forgetting that event.target can be a descendant
parent.addEventListener('click', (e) => {
  // e.target might not be the element you think
  if (e.target === parent) {
    console.log('Clicked directly on parent');
  }
});

// Mistake 4: Not considering composedPath with shadow DOM
document.addEventListener('click', (e) => {
  // e.target inside shadow DOM is retargeted to the host
  if (e.target.matches('.shadow-element')) {
    // This won't work if .shadow-element is inside shadow DOM
  }
  // Use e.composedPath() instead
});
```

### Best Practices

- Use bubbling for event delegation to reduce memory usage
- Only call `stopPropagation()` when you have a specific reason
- Consider using `event.composedPath()` when working with shadow DOM
- Use `focusin`/`focusout` instead of `focus`/`blur` when you need bubbling
- Check `event.eventPhase` when a listener is used in both capture and bubble
- Prefer `e.target.closest()` over `e.target` for delegation matching
- Document when and why you stop propagation (it can surprise other developers)

### Performance Considerations

Bubbling itself has negligible performance cost—the browser must traverse ancestor nodes anyway for CSS pseudo-classes like `:hover`. The performance concern is the number of handlers invoked during bubbling. A single delegated listener is much more efficient than many individual listeners.

```javascript
// Bad: 1000 individual listeners
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleItem);
});

// Good: 1 delegated listener
document.querySelector('.list').addEventListener('click', (e) => {
  const item = e.target.closest('.item');
  if (item) handleItem(e, item);
});
```

### Interview Questions

1. What is the difference between bubbling and capturing?
2. Which events do not bubble?
3. How does `event.target` differ from `event.currentTarget` during bubbling?
4. What is `composedPath()` and when is it useful?
5. How does shadow DOM affect event bubbling?
6. What happens to event listeners on removed elements during bubbling?
7. Can you have an event that only fires on the target and doesn't bubble?
8. How does event retargeting work with shadow DOM?

### Coding Challenges

1. **Bubbling Visualizer**: Create a tool that shows event propagation visually on a webpage
2. **Nested Form Handler**: Build a form system that handles nested form submissions correctly
3. **Event Path Logger**: Implement a utility that logs the full propagation path for any event
4. **Bubbling Event Bus**: Create a communication system that uses event bubbling for component interaction

### Related Topics

- [Event.composedPath()](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath)
- [Event.eventPhase](https://developer.mozilla.org/en-US/docs/Web/API/Event/eventPhase)
- [Shadow DOM and Events](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)

## Capturing Phase

### What It Is

The capturing phase is the first phase of event propagation. The event travels from the document root down through the DOM tree to the target element. Listeners registered with `capture: true` are invoked during this phase. Capturing is less commonly used directly but is essential for intercepting events before they reach their target.

```javascript
// HTML: <div id="grandparent"><div id="parent"><div id="child"></div></div></div>

document.getElementById('grandparent').addEventListener('click', () => {
  console.log('Grandparent (capturing)');
}, true);
document.getElementById('parent').addEventListener('click', () => {
  console.log('Parent (capturing)');
}, true);
document.getElementById('child').addEventListener('click', () => {
  console.log('Child (capturing)');
}, true);

// Click on child outputs:
// Grandparent (capturing)
// Parent (capturing)
// Child (capturing)
```

### Why It Is Important

The capturing phase allows parent elements to intercept events before child elements see them. This is useful for implementing event guards, global event preprocessing, context menu overrides, and ensuring certain behaviors happen before any child-specific logic runs.

### How It Works Internally

The browser walks from `document` down to `event.target.parentNode`, invoking capture-phase listeners. This is the reverse direction of bubbling. The capture phase completes before the target phase begins, which means capture listeners fire before any bubble listeners.

```javascript
// Full propagation order:
// 1. Capture phase: document -> html -> body -> ... -> target.parentNode
// 2. Target phase: target (both capture and bubble listeners on target)
// 3. Bubble phase: target.parentNode -> ... -> body -> html -> document

element.addEventListener('click', handler, true); // Capture phase
element.addEventListener('click', handler, false); // Bubble phase (default)
```

### Syntax

```javascript
// Register capture phase listener
element.addEventListener('click', handler, true);
element.addEventListener('click', handler, { capture: true });
element.addEventListener('click', handler); // false (bubbling) by default

// Check if listener is capture
// Note: there is no direct API to check, but you can infer from behavior
```

### Beginner Examples

```javascript
// Example 1: Basic capture demonstration
const container = document.querySelector('.container');
const button = document.querySelector('button');

container.addEventListener('click', () => {
  console.log('Container capture');
}, true);

button.addEventListener('click', () => {
  console.log('Button bubble (default)');
});

// Click on button:
// Container capture
// Button bubble

// Example 2: Intercepting events before children
form.addEventListener('submit', (e) => {
  // This runs in capture phase, before any child submit listeners
  console.log('Form validation (capture)');
  if (!validateAll()) {
    e.preventDefault();
    e.stopPropagation();
  }
}, true);

// Example 3: Mixed phase handlers on same element
element.addEventListener('click', () => {
  console.log('Capture handler');
}, true);

element.addEventListener('click', () => {
  console.log('Bubble handler');
}, false);

// Click on element:
// Capture handler (fires first since capture phase is before bubble)
// Bubble handler

// Note: when target element is the one clicked, handlers fire in registration order
// for the target phase, regardless of capture flag
```

### Intermediate Examples

```javascript
// Example 1: Event interception for logging
function createEventInterceptor() {
  document.addEventListener('click', (e) => {
    console.log('[INTERCEPT] Click on', e.target.tagName);
    // Can modify or prevent the event before any other handler
  }, true); // Capture phase

  document.addEventListener('submit', (e) => {
    console.log('[INTERCEPT] Form submit:', e.target.action);
  }, true);
}

createEventInterceptor();

// Example 2: Global right-click prevention
document.addEventListener('contextmenu', (e) => {
  if (e.target.matches('[data-prevent-context]')) {
    e.preventDefault();
    showCustomMenu(e.clientX, e.clientY);
  }
}, true); // Capture to handle before any element-specific handler

// Example 3: Capture-based event guard
function createEventGuard(condition) {
  return {
    start() {
      document.addEventListener('click', (e) => {
        if (!condition()) {
          e.stopPropagation();
          console.log('Event guard blocked propagation');
        }
      }, true);
    }
  };
}

const guard = createEventGuard(() => {
  // Only allow clicks when app is in editable mode
  return document.body.classList.contains('editing');
});

guard.start();

// Example 4: Capture for keyboard shortcuts
document.addEventListener('keydown', (e) => {
  // Global shortcuts should be handled in capture phase
  // to prevent page-specific handlers from stealing them
  if (e.key === 'F2') {
    e.preventDefault();
    startRenaming();
  }
  if (e.key === 'Escape') {
    closeAllModals();
  }
}, true);
```

### Advanced Examples

```javascript
// Example 1: Capture-based drag prevention
class DragGuard {
  constructor() {
    this.setup();
  }

  setup() {
    // Prevent text selection during drag operations
    document.addEventListener('selectstart', (e) => {
      if (this.isDragging) {
        e.preventDefault();
      }
    }, true);

    // Prevent default drag behavior on images
    document.addEventListener('dragstart', (e) => {
      if (e.target.matches('[data-no-drag]')) {
        e.preventDefault();
      }
    }, true);
  }

  enableDragging() {
    this.isDragging = true;
  }

  disableDragging() {
    this.isDragging = false;
  }
}

const dragGuard = new DragGuard();

// Example 2: Capture-based event routing system
class CaptureRouter {
  constructor() {
    this.routes = [];
    this.setup();
  }

  setup() {
    document.addEventListener('click', (e) => {
      for (const route of this.routes) {
        if (route.condition(e)) {
          route.handler(e);
          if (route.intercept) {
            e.stopPropagation();
          }
          break;
        }
      }
    }, true);
  }

  addRoute(condition, handler, options = {}) {
    this.routes.push({ condition, handler, ...options });
  }
}

const router = new CaptureRouter();

router.addRoute(
  (e) => e.target.matches('[data-action="delete"]'),
  (e) => {
    if (!confirm('Delete?')) {
      e.preventDefault();
      e.stopPropagation();
    }
  },
  { intercept: true }
);

router.addRoute(
  (e) => e.target.matches('.external-link'),
  (e) => {
    e.preventDefault();
    window.open(e.target.href, '_blank');
  }
);

// Example 3: Custom event priority with capture/bubble ordering
class PriorityEvents {
  constructor() {
    this.highPriority = new Map();
    this.lowPriority = new Map();
  }

  addHighPriority(type, handler) {
    if (!this.highPriority.has(type)) {
      this.highPriority.set(type, []);
      document.addEventListener(type, (e) => {
        const handlers = this.highPriority.get(e.type);
        if (handlers) {
          for (const h of handlers) h(e);
        }
      }, true); // Capture for high priority
    }
    this.highPriority.get(type).push(handler);
  }

  addLowPriority(type, handler) {
    if (!this.lowPriority.has(type)) {
      this.lowPriority.set(type, []);
      document.addEventListener(type, (e) => {
        const handlers = this.lowPriority.get(e.type);
        if (handlers) {
          for (const h of handlers) h(e);
        }
      }, false); // Bubble for low priority
    }
    this.lowPriority.get(type).push(handler);
  }
}

const priority = new PriorityEvents();
priority.addHighPriority('click', () => console.log('Runs first'));
priority.addLowPriority('click', () => console.log('Runs last'));

// Example 4: Capture-phase form validation framework
class CaptureValidator {
  constructor() {
    document.addEventListener('submit', (e) => {
      const form = e.target;
      const validators = form._validators || [];

      for (const validator of validators) {
        const result = validator(form);
        if (result !== true) {
          e.preventDefault();
          showError(form, result);
          return;
        }
      }
    }, true);
  }

  addValidator(form, validator) {
    if (!form._validators) form._validators = [];
    form._validators.push(validator);
  }
}

const validator = new CaptureValidator();
const form = document.querySelector('form');

validator.addValidator(form, (form) => {
  const email = form.querySelector('[type="email"]');
  return email.value.includes('@') || 'Invalid email';
});
```

### Real-World Use Cases

- **Global Keyboard Shortcuts**: Intercept key events before page-specific handlers
- **Form Validation**: Validate entire form before individual field handlers run
- **Drag Protection**: Prevent image dragging or text selection during drag operations
- **Security**: Block right-click context menus on sensitive content
- **Analytics**: Log all events of a certain type before any processing
- **Accessibility**: Ensure focus management happens before other handlers
- **Testing**: Simulate events that bypass application-level guards

### Common Mistakes

```javascript
// Mistake 1: Confusing capture flag with event.target
element.addEventListener('click', handler, true); // capture: true
// The handler fires during capture phase for this element
// It does NOT mean only clicks directly on this element are caught

// Mistake 2: Capture listeners on window vs document
window.addEventListener('click', handler, true);
document.addEventListener('click', handler, true);
// These are different: window capture fires before document capture

// Mistake 3: Forgetting that target-phase listeners fire between capture and bubble
// Target phase listeners fire in registration order, capture flag is ignored

// Mistake 4: Overusing capture when bubbling would suffice
// Most use cases should use bubbling (default) for event delegation
```

### Best Practices

- Use capture phase for events that must be handled before anything else
- Reserve capture for global concerns (keyboard shortcuts, guards, logging)
- Document why capture is used (it's less common and can be surprising)
- Use capture sparingly—most applications only need bubbling
- Be aware that capture listeners fire on ancestors, not just the target
- Combine capture and bubble for different priority levels of the same event

### Performance Considerations

Capture phase listeners have the same performance characteristics as bubble listeners. However, since capture fires first, heavy capture handlers can make the page feel unresponsive if they block. Keep capture handlers lightweight, especially for events like `mousemove` or `scroll`.

### Interview Questions

1. What is the order of event phases? (capture -> target -> bubble)
2. How do you register a capture-phase listener?
3. When would you use capturing instead of bubbling?
4. What happens when a capture listener and bubble listener are on the same element?
5. Does `stopPropagation()` work in the capture phase?
6. What is the `eventPhase` value for capture phase?
7. How does capture interact with shadow DOM?
8. What is the difference between `useCapture` and `options.capture`?

### Coding Challenges

1. **Capture Router**: Build a routing system that intercepts events in the capture phase
2. **Priority Event System**: Create a system where events are handled in priority order using capture/bubble
3. **Event Guard**: Implement a capture-based guard that prevents specified events from bubbling
4. **Global Shortcut Manager**: Build a keyboard shortcut system using capture-phase listeners

### Related Topics

- [Event.eventPhase](https://developer.mozilla.org/en-US/docs/Web/API/Event/eventPhase)
- [addEventListener options](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [Event dispatch and DOM event flow](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)

## stopPropagation() and stopImmediatePropagation()

### What It Is

`stopPropagation()` is a method on the Event object that prevents the event from continuing its journey through the DOM. In the capture phase, it prevents further capture-phase listeners from being called. In the bubble phase, it prevents bubbling. `stopImmediatePropagation()` does the same but also prevents any other listeners on the current element from being called.

```javascript
element.addEventListener('click', (e) => {
  e.stopPropagation();
  // Parent elements won't receive this click event (bubbling)
  // But other handlers on this same element still fire
});

element.addEventListener('click', (e) => {
  e.stopImmediatePropagation();
  // Parent elements AND other handlers on this element won't fire
});
```

### Why It Is Important

These methods give developers fine-grained control over event flow. `stopPropagation` is essential when you need to prevent parent handlers from responding to a child event (e.g., clicking a button inside a panel should not trigger the panel's click handler). `stopImmediatePropagation` is useful when a handler must be the absolute last word for an event.

### How It Works Internally

The Event object maintains internal flags (`propagationStopped` and `immediatePropagationStopped`). During event dispatch, the browser checks these flags before invoking each subsequent listener. Once set to `true`, the propagation loop terminates.

```javascript
// Event dispatch loop (conceptual)
function dispatchEvent(event) {
  // Capture phase
  for (let el = document; el !== event.target; el = el.firstChild) {
    invokeListeners(el, event, CAPTURE);
    if (event.propagationStopped) return;
  }

  // Target phase
  invokeListeners(event.target, event, ALL);
  if (event.propagationStopped) return;

  // Bubble phase
  for (let el = event.target.parentNode; el; el = el.parentNode) {
    invokeListeners(el, event, BUBBLE);
    if (event.propagationStopped) return;
  }
}
```

### Syntax

```javascript
event.stopPropagation();
// Stops further propagation in current phase (capture or bubble)
// Other listeners on current element still execute

event.stopImmediatePropagation();
// Stops propagation AND prevents other listeners on current element
// No more handlers execute for this event on any element

// Check if propagation was stopped
event.cancelBubble = true; // Legacy alias for stopPropagation
event.returnValue = false; // Legacy for preventDefault
```

### Beginner Examples

```javascript
// Example 1: Basic stopPropagation
const parent = document.querySelector('.parent');
const child = document.querySelector('.child');

parent.addEventListener('click', () => {
  console.log('Parent clicked');
});

child.addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('Child clicked - propagation stopped');
});

// Clicking child only logs: "Child clicked - propagation stopped"
// Parent does NOT receive the click event

// Example 2: stopImmediatePropagation
const button = document.querySelector('button');

button.addEventListener('click', () => {
  console.log('First handler');
});

button.addEventListener('click', (e) => {
  e.stopImmediatePropagation();
  console.log('Second handler - stops all');
});

button.addEventListener('click', () => {
  console.log('Third handler'); // Never fires
});

// Click logs:
// First handler
// Second handler - stops all

// Example 3: Practical example - accordion
document.querySelectorAll('.accordion-header').forEach(header => {
  header.addEventListener('click', (e) => {
    const panel = header.nextElementSibling;
    panel.classList.toggle('open');

    // Prevent parent accordion groups from toggling
    e.stopPropagation();
  });
});

// Example 4: Modal inside another modal
document.querySelector('.modal-backdrop').addEventListener('click', () => {
  closeModal();
});

document.querySelector('.modal-content').addEventListener('click', (e) => {
  // Prevent click on modal content from closing the modal
  e.stopPropagation();
});
```

### Intermediate Examples

```javascript
// Example 1: Selective propagation
document.querySelector('.dropdown').addEventListener('click', (e) => {
  const dropdown = e.currentTarget;
  const menu = dropdown.querySelector('.dropdown-menu');

  if (e.target.closest('.dropdown-toggle')) {
    menu.classList.toggle('open');
    // Don't stop propagation: allow document-level handler
  } else if (!e.target.closest('.dropdown-menu')) {
    // Clicked outside menu, close it
    menu.classList.remove('open');
    e.stopPropagation();
  }
});

// Document-level handler to close dropdown when clicking outside
document.addEventListener('click', () => {
  document.querySelectorAll('.dropdown-menu.open').forEach(menu => {
    menu.classList.remove('open');
  });
});

// Example 2: Propagation context
function createPropagationContext() {
  let isStopped = false;

  return {
    stop() {
      isStopped = true;
    },
    shouldContinue() {
      return !isStopped;
    },
    wrap(handler) {
      return (e) => {
        if (isStopped) {
          e.stopPropagation();
          return;
        }
        handler(e, this);
      };
    }
  };
}

// Example 3: Nested drag-and-drop containment
class DragContainer {
  constructor(element) {
    this.element = element;
    this.isDragging = false;
    this.setup();
  }

  setup() {
    this.element.addEventListener('mousedown', (e) => {
      const draggable = e.target.closest('.draggable');
      if (!draggable) return;

      // Only start drag if clicking on the draggable header
      if (!e.target.closest('.drag-handle')) return;

      this.isDragging = true;

      // Stop propagation to prevent parent drag containers from activating
      e.stopPropagation();
      this.startDrag(e, draggable);
    });
  }

  startDrag(e, element) {
    // Drag logic...
  }
}

// Example 4: StopImmediatePropagation for event middleware
function createEventMiddleware() {
  const middleware = [];

  function use(fn) {
    middleware.push(fn);
  }

  function wrapHandler(handler) {
    return (e) => {
      for (const mw of middleware) {
        const result = mw(e);
        if (result === false) {
          e.stopImmediatePropagation();
          return;
        }
      }
      handler(e);
    };
  }

  return { use, wrapHandler };
}

const middleware = createEventMiddleware();
middleware.use((e) => {
  if (e.target.matches('[data-disabled]')) {
    return false; // Block the event entirely
  }
});

button.addEventListener('click', middleware.wrapHandler(() => {
  console.log('This runs only if middleware allows');
}));
```

### Advanced Examples

```javascript
// Example 1: Propagation-aware event delegation
class SmartDelegator {
  constructor(root) {
    this.root = root;
    this.handlers = new Map();
    this.setup();
  }

  setup() {
    this.root.addEventListener('click', (e) => {
      let target = e.target;
      while (target && target !== this.root) {
        const handlers = this.handlers.get(target);
        if (handlers) {
          for (const handler of handlers) {
            if (e.cancelBubble) return;
            handler(e, target);
          }
        }
        if (e.cancelBubble) return;
        target = target.parentElement;
      }
    });
  }

  on(selector, handler) {
    const elements = this.root.querySelectorAll(selector);
    elements.forEach(el => {
      if (!this.handlers.has(el)) {
        this.handlers.set(el, []);
      }
      this.handlers.get(el).push(handler);
    });
  }

  // Respect stopPropagation from children
  off(selector, handler) {
    const elements = this.root.querySelectorAll(selector);
    elements.forEach(el => {
      const handlers = this.handlers.get(el);
      if (handlers) {
        const idx = handlers.indexOf(handler);
        if (idx >= 0) handlers.splice(idx, 1);
      }
    });
  }
}

// Example 2: Event propagation recorder for debugging
class PropagationRecorder {
  constructor() {
    this.recordings = [];
    this.enabled = false;
  }

  start() {
    this.enabled = true;
    this.setupCapture();
    this.setupBubble();
  }

  setupCapture() {
    document.addEventListener('click', (e) => this.record(e), true);
    document.addEventListener('keydown', (e) => this.record(e), true);
  }

  setupBubble() {
    document.addEventListener('click', (e) => this.record(e), false);
    document.addEventListener('keydown', (e) => this.record(e), false);
  }

  record(event) {
    if (!this.enabled) return;

    this.recordings.push({
      type: event.type,
      phase: event.eventPhase === Event.CAPTURING_PHASE ? 'capture' : 'bubble',
      target: event.target.tagName,
      currentTarget: event.currentTarget.tagName,
      timestamp: event.timeStamp,
      stopped: event.cancelBubble
    });
  }

  stop() {
    this.enabled = false;
    const summary = this.analyze();
    this.recordings = [];
    return summary;
  }

  analyze() {
    const captureCount = this.recordings.filter(r => r.phase === 'capture').length;
    const bubbleCount = this.recordings.filter(r => r.phase === 'bubble').length;
    const stoppedCount = this.recordings.filter(r => r.stopped).length;

    return {
      total: this.recordings.length,
      capture: captureCount,
      bubble: bubbleCount,
      propagationStopped: stoppedCount
    };
  }
}

// Example 3: Controlled propagation with AbortSignal
class ControlledEvent {
  constructor(target, type, options = {}) {
    this.target = target;
    this.type = type;
    this.options = options;
    this.propagationController = new AbortController();
  }

  dispatch(detail = {}) {
    const event = new CustomEvent(this.type, {
      detail,
      bubbles: this.options.bubbles !== false,
      cancelable: this.options.cancelable !== false
    });

    // Store the abort signal on the event
    event._propagationSignal = this.propagationController.signal;

    this.target.dispatchEvent(event);
  }

  stopPropagation() {
    this.propagationController.abort();
  }
}

// Usage
const event = new ControlledEvent(document, 'app:action', { bubbles: true });

document.addEventListener('app:action', (e) => {
  console.log('First listener');
  if (e.detail.shouldStop) {
    event.stopPropagation(); // Uses controller to signal stop
  }
});

// Second listener checks the controller
document.addEventListener('app:action', (e) => {
  if (e._propagationSignal?.aborted) {
    console.log('Propagation was stopped');
    return;
  }
  console.log('Second listener');
});

// Example 4: Event propagation tree builder
function buildPropagationTree(event) {
  const tree = {
    type: event.type,
    target: {
      tag: event.target.tagName,
      id: event.target.id || null,
      classes: Array.from(event.target.classList)
    },
    capturePath: [],
    bubblePath: []
  };

  // Build capture path (document -> target)
  let current = document;
  const capturePath = [];
  while (current && current !== event.target) {
    capturePath.unshift({
      tag: current.tagName || current.nodeName,
      id: current.id || null
    });
    current = current.firstElementChild;
  }
  tree.capturePath = capturePath;

  // Build bubble path (target -> document)
  current = event.target.parentElement;
  const bubblePath = [];
  while (current) {
    bubblePath.push({
      tag: current.tagName,
      id: current.id || null
    });
    current = current.parentElement;
  }
  tree.bubblePath = bubblePath;

  return tree;
}

document.addEventListener('click', (e) => {
  console.log('Propagation tree:', buildPropagationTree(e));
});
```

### Real-World Use Cases

- **Stop Button Clicks from Triggering Parent**: A button inside a card should not trigger the card's click handler
- **Modal Inside Modal**: Clicking the inner modal should not close the outer modal
- **Context Menu Items**: Right-click menu items should not propagate to the document context menu
- **Nested Sortable Lists**: Clicking a nested item should not trigger the parent sortable
- **Accordion Groups**: Expanding one panel should not affect sibling panels
- **Form Submit in Nested Forms**: Submitting a child form should not submit the parent form
- **Tab Stop for Event Handlers**: A handler that handles an event should prevent other unrelated handlers

### Common Mistakes

```javascript
// Mistake 1: Overusing stopPropagation (breaks legitimate delegation)
// Don't stop propagation unless you have a specific reason

// Mistake 2: Forgetting that stopPropagation doesn't prevent default actions
link.addEventListener('click', (e) => {
  e.stopPropagation(); // Still navigates!
  e.preventDefault(); // Must also call this to prevent navigation
});

// Mistake 3: Relying on stopPropagation for security
child.addEventListener('click', (e) => {
  e.stopPropagation();
  // Parent can still see the event if they use capture phase!
});

// Mistake 4: stopPropagation doesn't stop other handlers on the same element
element.addEventListener('click', handler1);
element.addEventListener('click', (e) => {
  e.stopPropagation();
  // handler2 still fires! Use stopImmediatePropagation to prevent this
});
element.addEventListener('click', handler2);
element.addEventListener('click', handler3); // WON'T fire because handler2 stopped propagation
// Actually wait - stopPropagation() stops propagation to parent elements, NOT other handlers on same element
// handler3 WILL fire because stopPropagation only affects ancestors
```

### Best Practices

- Use `stopPropagation` sparingly and document why
- Prefer `stopPropagation` over `stopImmediatePropagation` unless you have a clear need
- Consider if `event delegation` is a better pattern than stopping propagation
- Don't stop propagation for delegation-based analytics or logging
- Be aware that capture-phase listeners still fire even if you stop bubbling
- Use `stopImmediatePropagation` only when a handler must be the last one to execute
- Test with capture listeners to ensure propagation stopping works as expected

### Performance Considerations

`stopPropagation` and `stopImmediatePropagation` are fast operations—they merely set a boolean flag on the event object. However, using `stopPropagation` excessively can break application architecture that relies on delegation, leading to bugs and workarounds that are harder to maintain.

### Interview Questions

1. What is the difference between `stopPropagation` and `stopImmediatePropagation`?
2. Does `stopPropagation` work in the capture phase?
3. Can a capture-phase listener stop bubbling?
4. What is the difference between `stopPropagation` and `preventDefault`?
5. How do you stop all listeners on the same element?
6. What is `cancelBubble` and how does it relate to `stopPropagation`?
7. Does `stopPropagation` affect the event's default action?
8. How can you test if propagation was stopped?

### Coding Challenges

1. **Propagation Guard**: Build a utility that selectively allows/disallows event propagation based on conditions
2. **Event Filter**: Create a system where certain events are blocked based on a filter function
3. **Propagation Tree Builder**: Write a function that visualizes the full propagation path of an event
4. **Controlled Event System**: Implement an event system where propagation can be controlled from any handler

### Related Topics

- [Event.preventDefault()](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault)
- [Event.cancelBubble](https://developer.mozilla.org/en-US/docs/Web/API/Event/cancelBubble)
- [Event.stopImmediatePropagation()](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopImmediatePropagation)
- [Event Flow (W3C)](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)
