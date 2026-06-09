# Events - addEventListener, event object, event types, event handlers

## Introduction

Events are the primary mechanism through which JavaScript interacts with user actions and browser state changes. An event is a signal that something has occurred—a click, a key press, a form submission, a page load, or a timer firing. The browser's event system allows developers to define handlers that execute in response to these signals, forming the foundation of interactive web applications.

The DOM Events specification defines a standardized event system that works consistently across browsers. Understanding how events flow through the DOM, how to attach and remove handlers, and how the event object provides context about the occurrence is essential for every front-end developer.

## addEventListener()

### What It Is

`addEventListener()` is the standard method for registering an event handler on a DOM element. It accepts the event type (as a string), a callback function to execute when the event occurs, and an optional options object or boolean for configuration. It is the preferred method over legacy `on`-prefixed properties (like `onclick`) because it supports multiple handlers for the same event on the same element.

```javascript
const button = document.querySelector('button');
button.addEventListener('click', (event) => {
  console.log('Button clicked!', event);
});
```

### Why It Is Important

`addEventListener` provides a clean, flexible, and composable event registration system. Unlike inline event handlers or `onclick` properties, `addEventListener` allows multiple independent handlers to listen for the same event on the same element without overwriting each other. It also supports options for one-time execution, passive listeners (performance optimization), and capture-phase listening.

### How It Works Internally

Each DOM element maintains an internal event listener map. When `addEventListener` is called, the browser adds the handler function to this map along with its configuration. When an event of the matching type is dispatched on the element, the browser iterates through the registered handlers and calls them in the order they were added.

```javascript
// Conceptual internal event listener map
// element._listeners = {
//   'click': [
//     { handler: fn1, options: { capture: false, once: false, passive: false } },
//     { handler: fn2, options: { capture: true, once: false, passive: false } }
//   ],
//   'keydown': [
//     { handler: fn3, options: { capture: false, once: true, passive: false } }
//   ]
// }
```

### Syntax

```javascript
// Basic syntax
target.addEventListener(type, listener);
target.addEventListener(type, listener, options);
target.addEventListener(type, listener, useCapture);

// Options object
target.addEventListener(type, listener, {
  capture: false,    // Use capture phase (default: false)
  once: false,       // Auto-remove after first invocation (default: false)
  passive: false,    // Don't call preventDefault() (default: false)
  signal: abortSignal // AbortController signal for removal
});
```

### Beginner Examples

```javascript
// Example 1: Basic click handler
const btn = document.querySelector('#myButton');
btn.addEventListener('click', () => {
  alert('Button clicked!');
});

// Example 2: Multiple handlers on same element
const input = document.querySelector('#myInput');
input.addEventListener('focus', () => {
  input.style.borderColor = 'blue';
});
input.addEventListener('blur', () => {
  input.style.borderColor = '';
});
input.addEventListener('input', () => {
  console.log('Value:', input.value);
});

// Example 3: One-time event
const intro = document.querySelector('#intro');
intro.addEventListener('click', () => {
  intro.style.color = 'red';
}, { once: true });

// Example 4: Passive scroll listener (performance optimization)
document.addEventListener('scroll', () => {
  console.log('Scrolled');
}, { passive: true });

// Example 5: Using AbortController for cleanup
const controller = new AbortController();
window.addEventListener('resize', () => {
  console.log('Window resized');
}, { signal: controller.signal });
// Later: controller.abort(); // Removes the listener
```

### Intermediate Examples

```javascript
// Example 1: Event handler factory
function createDebouncedHandler(fn, delay = 300) {
  let timer = null;
  return function (event) {
    clearTimeout(timer);
    timer = setTimeout(() => fn(event), delay);
  };
}

const searchInput = document.querySelector('#search');
searchInput.addEventListener('input', createDebouncedHandler((e) => {
  performSearch(e.target.value);
}));

// Example 2: Event handler with cleanup
function setupResizeHandler() {
  const handler = () => {
    console.log('Width:', window.innerWidth);
  };

  window.addEventListener('resize', handler);

  // Return cleanup function
  return () => {
    window.removeEventListener('resize', handler);
  };
}

const cleanup = setupResizeHandler();
// Later, when component unmounts:
cleanup();

// Example 3: Conditional handler registration
function registerIf(condition, target, type, handler, options) {
  if (condition) {
    target.addEventListener(type, handler, options);
    return () => target.removeEventListener(type, handler, options);
  }
  return () => {};
}

const unregister = registerIf(
  'ontouchstart' in window,
  document.body,
  'touchstart',
  handleTouch
);

// Example 4: Monitoring all events on an element
function monitorEvents(element) {
  const eventTypes = ['click', 'mouseenter', 'mouseleave', 'focus', 'blur'];

  const unregister = eventTypes.map(type => {
    const handler = (e) => {
      console.log(`[${type}] on`, element, e);
    };
    element.addEventListener(type, handler);
    return () => element.removeEventListener(type, handler);
  });

  return () => unregister.forEach(fn => fn());
}

const stopMonitoring = monitorEvents(document.querySelector('.target'));
```

### Advanced Examples

```javascript
// Example 1: Event delegation with addEventListener
class EventDelegator {
  constructor(root) {
    this.root = root;
    this.handlers = new Map();
  }

  on(eventType, selector, handler) {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, new Map());
      this.root.addEventListener(eventType, (e) => {
        this.dispatch(eventType, e);
      });
    }

    this.handlers.get(eventType).set(selector, handler);
  }

  dispatch(eventType, event) {
    const handlers = this.handlers.get(eventType);
    if (!handlers) return;

    let target = event.target;
    while (target && target !== this.root) {
      for (const [selector, handler] of handlers) {
        if (target.matches(selector)) {
          handler.call(target, event, target);
        }
      }
      target = target.parentElement;
    }
  }

  off(eventType, selector) {
    const handlers = this.handlers.get(eventType);
    if (handlers) {
      handlers.delete(selector);
    }
  }

  destroy() {
    this.handlers.clear();
  }
}

const delegator = new EventDelegator(document.querySelector('.list'));
delegator.on('click', '.item', (e, el) => {
  console.log('Clicked:', el.textContent);
});

// Example 2: Composable event pipeline
function composeEventHandlers(...handlers) {
  return function (event) {
    for (const handler of handlers) {
      if (event.defaultPrevented) break;
      if (typeof handler === 'function') {
        handler(event);
      }
    }
  };
}

const composedHandler = composeEventHandlers(
  validateField,
  updateUI,
  sendAnalytics
);

input.addEventListener('blur', composedHandler);

// Example 3: Event listener with priority queue
class PriorityEventTarget {
  constructor(target) {
    this.target = target;
    this.priorityMap = new Map();
    this.listenerId = 0;
  }

  addEventListener(type, handler, priority = 0) {
    const id = ++this.listenerId;

    if (!this.priorityMap.has(type)) {
      this.priorityMap.set(type, []);
      this.target.addEventListener(type, (e) => {
        this.dispatch(type, e);
      });
    }

    this.priorityMap.get(type).push({ id, handler, priority });
    this.priorityMap.get(type).sort((a, b) => b.priority - a.priority);

    return id;
  }

  removeEventListener(id) {
    for (const [, handlers] of this.priorityMap) {
      const index = handlers.findIndex(h => h.id === id);
      if (index >= 0) {
        handlers.splice(index, 1);
        return true;
      }
    }
    return false;
  }

  dispatch(type, event) {
    const handlers = this.priorityMap.get(type);
    if (!handlers) return;

    for (const { handler } of handlers) {
      if (event.defaultPrevented) break;
      handler(event);
    }
  }
}

const priorityTarget = new PriorityEventTarget(button);
priorityTarget.addEventListener('click', () => console.log('Second'), 0);
priorityTarget.addEventListener('click', () => console.log('First'), 100);

// Example 4: Custom event system with middleware
function createEventSystem(target) {
  const middleware = [];

  function use(fn) {
    middleware.push(fn);
  }

  function on(type, handler, options) {
    const wrappedHandler = (event) => {
      let canProceed = true;
      for (const mw of middleware) {
        if (mw(event) === false) {
          canProceed = false;
          break;
        }
      }
      if (canProceed) {
        handler(event);
      }
    };
    target.addEventListener(type, wrappedHandler, options);
    return () => target.removeEventListener(type, wrappedHandler, options);
  }

  return { on, use, target };
}

const system = createEventSystem(document);
system.use((e) => {
  if (e.type === 'click' && e.target.matches('[data-disabled]')) {
    return false; // Block event
  }
});

system.on('click', '.link', (e) => {
  console.log('Navigating:', e.target.href);
});
```

### Real-World Use Cases

- **Form Handling**: `submit`, `input`, `change`, `focus`, `blur` events for form logic
- **Drag and Drop**: `mousedown`, `mousemove`, `mouseup` for custom drag implementations
- **Keyboard Shortcuts**: `keydown`/`keyup` for application hotkeys
- **Scroll-Based Animations**: `scroll` event with `passive: true` for parallax effects
- **Responsive Design**: `resize` event for layout adjustments
- **Touch Interfaces**: `touchstart`, `touchmove`, `touchend` for mobile interactions
- **Game Development**: `keydown` for game controls, `requestAnimationFrame` for game loop

### Common Mistakes

```javascript
// Mistake 1: Passing function result instead of function reference
button.addEventListener('click', handleClick()); // Calls handleClick immediately, not on click
button.addEventListener('click', handleClick); // Correct

// Mistake 2: Anonymous functions can't be removed
button.addEventListener('click', () => console.log('Click'));
button.removeEventListener('click', () => console.log('Click')); // Different reference, not removed!
// Fix: store the reference
const handler = () => console.log('Click');
button.addEventListener('click', handler);
button.removeEventListener('click', handler);

// Mistake 3: Forgetting passive option for scroll events
document.addEventListener('touchmove', (e) => {
  e.preventDefault(); // Will throw warning in Chrome if not passive
}, { passive: false });

// Mistake 4: Memory leaks from unremoved listeners
function setup() {
  const button = document.querySelector('#temp-btn');
  button.addEventListener('click', () => {
    console.log('This prevents garbage collection of the closure');
  });
} // button might be removed from DOM but listener keeps reference alive
```

### Best Practices

- Store handler references if you need to remove them later
- Use `AbortController` for managing multiple listener cleanup
- Prefer `{ passive: true }` for scroll/touch events when not calling `preventDefault`
- Use `{ once: true }` for one-shot events (like toasts, notifications)
- Keep handlers small and focused; delegate to named functions
- Remove event listeners when cleaning up (component unmount, element removal)
- Avoid closures that capture large objects in event handlers (memory leaks)

### Performance Considerations

Each event listener consumes memory. For hundreds of similar elements, event delegation is more efficient than individual listeners. Passive listeners improve scroll performance by telling the browser not to wait for the handler before scrolling.

```javascript
// Bad: Individual listener per element
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleItemClick);
});

// Good: Single delegated listener
document.querySelector('.list').addEventListener('click', (e) => {
  if (e.target.matches('.item')) {
    handleItemClick(e);
  }
});

// Performance: Passive vs non-passive scroll
// The browser can start scrolling before the handler runs
window.addEventListener('scroll', handler, { passive: true });
```

### Interview Questions

1. What is the difference between `addEventListener` and `onclick`?
2. What does the `once` option do?
3. What is a passive event listener and when should you use it?
4. How do you remove an event listener added with `addEventListener`?
5. Can you have multiple handlers for the same event on the same element?
6. What is `AbortController` and how does it work with events?
7. How does the `capture` option affect event handling?
8. What happens if you add the same listener twice?

### Coding Challenges

1. **Event Spy**: Create a utility that logs all events on a given element for debugging
2. **Event Bus**: Implement a publish/subscribe system using addEventListener
3. **Priority Handlers**: Create an addEventListener that supports priority ordering
4. **Debounced Events**: Implement a debounced event handler factory

### Related Topics

- [removeEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)
- [EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget)
- [CustomEvent](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)
- [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)

## Event Object

### What It Is

The `Event` object is created by the browser when an event occurs and is passed as the first argument to event handlers. It contains properties and methods that provide context about the event: the target element, event type, timestamp, coordinates (for mouse events), key codes (for keyboard events), and methods to control event flow like `stopPropagation()` and `preventDefault()`.

```javascript
element.addEventListener('click', (event) => {
  console.log(event.type); // "click"
  console.log(event.target); // The clicked element
  console.log(event.currentTarget); // The element with the listener
  console.log(event.clientX, event.clientY); // Mouse coordinates
});
```

### Why It Is Important

The Event object is the communication channel between the browser and your code. Without it, handlers would have no context about what happened, where it happened, or how to respond. It provides all the information needed to build responsive, context-aware interactions.

### How It Works Internally

When an event is dispatched, the browser creates an Event instance (or a subclass like `MouseEvent`, `KeyboardEvent`, `FocusEvent`) and populates it with information from the underlying system event. The event then travels through the DOM phases (capture, target, bubble) with the same object instance. Properties like `target` and `currentTarget` are updated as the event propagates.

```javascript
// Event class hierarchy
// Event
//   ├── UIEvent
//   │     ├── MouseEvent
//   │     │     ├── WheelEvent
//   │     │     └── DragEvent
//   │     ├── KeyboardEvent
//   │     ├── FocusEvent
//   │     └── TouchEvent
//   ├── PointerEvent
//   ├── CompositionEvent
//   ├── ClipboardEvent
//   ├── InputEvent
//   ├── SubmitEvent
//   └── CustomEvent
```

### Syntax

```javascript
// Event object properties
event.type;              // Event type string (e.g., "click")
event.target;            // Element that triggered the event
event.currentTarget;     // Element that has the listener attached
event.eventPhase;        // Current phase: 1 (capture), 2 (at target), 3 (bubble)
event.bubbles;           // Whether the event bubbles
event.cancelable;        // Whether preventDefault can be called
event.defaultPrevented;  // Whether preventDefault was called
event.timestamp;         // Time when event was created
event.isTrusted;         // true if dispatched by browser, false if by script

// Event object methods
event.preventDefault();   // Prevents default browser behavior
event.stopPropagation();  // Stops event propagation
event.stopImmediatePropagation(); // Stops propagation AND other handlers on same element
```

### Beginner Examples

```javascript
// Example 1: Mouse event properties
document.addEventListener('click', (e) => {
  console.log({
    type: e.type,
    target: e.target,
    screen: { x: e.screenX, y: e.screenY },
    client: { x: e.clientX, y: e.clientY },
    page: { x: e.pageX, y: e.pageY },
    button: e.button, // 0 = left, 1 = middle, 2 = right
    buttons: e.buttons,
    altKey: e.altKey,
    ctrlKey: e.ctrlKey,
    shiftKey: e.shiftKey,
    metaKey: e.metaKey
  });
});

// Example 2: Keyboard event properties
document.addEventListener('keydown', (e) => {
  console.log({
    key: e.key,        // "a", "Enter", "ArrowUp"
    code: e.code,      // "KeyA", "Enter", "ArrowUp"
    keyCode: e.keyCode, // Deprecated, use key instead
    repeat: e.repeat,
    altKey: e.altKey,
    ctrlKey: e.ctrlKey,
    shiftKey: e.shiftKey
  });

  // Prevent default for specific keys
  if (e.key === 'F5') {
    e.preventDefault();
  }
});

// Example 3: Preventing form submission
document.querySelector('form').addEventListener('submit', (e) => {
  if (!validateForm()) {
    e.preventDefault(); // Prevent submission
  }
});

// Example 4: Using event.isTrusted to distinguish real vs synthetic events
button.addEventListener('click', (e) => {
  if (e.isTrusted) {
    console.log('Real user click');
  } else {
    console.log('Programmatic click');
  }
});

// Dispatch a synthetic event
button.dispatchEvent(new Event('click'));
```

### Intermediate Examples

```javascript
// Example 1: Custom data in events
document.addEventListener('click', (e) => {
  // Getting custom data attributes from target
  const action = e.target.closest('[data-action]')?.dataset.action;
  const id = e.target.closest('[data-id]')?.dataset.id;

  if (action) {
    handleAction(action, id);
  }
});

// Example 2: Creating and dispatching custom events with data
function notify(message, type = 'info') {
  const event = new CustomEvent('app:notification', {
    detail: { message, type, timestamp: Date.now() },
    bubbles: true,
    cancelable: true
  });
  document.dispatchEvent(event);
}

document.addEventListener('app:notification', (e) => {
  showToast(e.detail.message, e.detail.type);
});

notify('User saved successfully', 'success');

// Example 3: Mouse event modifiers
function handleDrag(dragElement) {
  let isDragging = false;
  let startX, startY, origX, origY;

  dragElement.addEventListener('mousedown', (e) => {
    isDragging = true;
    startX = e.clientX;
    startY = e.clientY;
    origX = dragElement.offsetLeft;
    origY = dragElement.offsetTop;
    dragElement.style.cursor = 'grabbing';
  });

  document.addEventListener('mousemove', (e) => {
    if (!isDragging) return;
    const dx = e.clientX - startX;
    const dy = e.clientY - startY;
    dragElement.style.left = `${origX + dx}px`;
    dragElement.style.top = `${origY + dy}px`;
  });

  document.addEventListener('mouseup', () => {
    isDragging = false;
    dragElement.style.cursor = '';
  });
}

// Example 4: Focus event delegation
document.addEventListener('focusin', (e) => {
  // Validate on focus loss
  const field = e.target;
  if (field.matches('.validate')) {
    field.classList.remove('invalid');
    field.classList.add('touched');
  }
});

document.addEventListener('focusout', (e) => {
  const field = e.target;
  if (field.matches('.validate') && field.classList.contains('touched')) {
    validateField(field);
  }
});
```

### Advanced Examples

```javascript
// Example 1: Event object proxy for logging/monitoring
function createEventMonitor() {
  return new Proxy(window, {
    get(target, prop) {
      if (prop === 'addEventListener') {
        return function (type, handler, options) {
          const wrappedHandler = function (event) {
            const eventProxy = new Proxy(event, {
              get(eventTarget, eventProp) {
                if (eventProp === 'target' || eventProp === 'currentTarget') {
                  console.log(`[Event Monitor] Accessing ${eventProp} of ${event.type}`);
                }
                return Reflect.get(eventTarget, eventProp);
              }
            });
            return handler(eventProxy);
          };
          return target.addEventListener(type, wrappedHandler, options);
        };
      }
      return Reflect.get(target, prop);
    }
  });
}

// Example 2: Event serialization for analytics
function serializeEvent(event) {
  const base = {
    type: event.type,
    target: getElementPath(event.target),
    timestamp: event.timeStamp,
    isTrusted: event.isTrusted
  };

  if (event instanceof MouseEvent) {
    return {
      ...base,
      x: event.clientX,
      y: event.clientY,
      button: event.button
    };
  }

  if (event instanceof KeyboardEvent) {
    return {
      ...base,
      key: event.key,
      code: event.code
    };
  }

  if (event instanceof FocusEvent) {
    return {
      ...base,
      relatedTarget: event.relatedTarget?.tagName || null
    };
  }

  return base;
}

function getElementPath(element) {
  const path = [];
  let current = element;
  while (current && current !== document) {
    path.unshift({
      tag: current.tagName,
      id: current.id || undefined,
      class: current.className || undefined
    });
    current = current.parentElement;
  }
  return path;
}

document.addEventListener('click', (e) => {
  const eventData = serializeEvent(e);
  navigator.sendBeacon('/analytics', JSON.stringify(eventData));
});

// Example 3: Event state machine
class EventDrivenStateMachine {
  constructor(initialState, transitions) {
    this.state = initialState;
    this.transitions = transitions;
    this.eventTarget = new EventTarget();
  }

  handle(event) {
    const possibleTransitions = this.transitions
      .filter(t => t.from === this.state && t.event === event.type);

    for (const transition of possibleTransitions) {
      if (transition.condition && !transition.condition(event)) continue;

      const oldState = this.state;
      this.state = transition.to;

      this.eventTarget.dispatchEvent(new CustomEvent('statechange', {
        detail: { from: oldState, to: this.state, event }
      }));

      if (transition.action) {
        transition.action(event);
      }
      break;
    }
  }

  onStateChange(handler) {
    this.eventTarget.addEventListener('statechange', handler);
  }

  bindTo(element) {
    const events = new Set(this.transitions.map(t => t.event));
    events.forEach(eventType => {
      element.addEventListener(eventType, (e) => this.handle(e));
    });
  }
}

const machine = new EventDrivenStateMachine('idle', [
  { from: 'idle', event: 'click', to: 'active', condition: (e) => e.button === 0 },
  { from: 'active', event: 'dblclick', to: 'expanded', condition: (e) => e.button === 0 },
  { from: 'active', event: 'click', to: 'idle', condition: (e) => e.target.matches('.close') },
  { from: 'expanded', event: 'click', to: 'idle' }
]);

machine.bindTo(document.querySelector('.widget'));

// Example 4: Preventing default with async validation
form.addEventListener('submit', async (e) => {
  e.preventDefault(); // Always prevent initially

  const submitBtn = form.querySelector('[type="submit"]');
  submitBtn.disabled = true;
  submitBtn.textContent = 'Validating...';

  try {
    const isValid = await asyncValidate(form);
    if (isValid) {
      // Re-dispatch or use fetch directly
      const data = new FormData(form);
      await submitForm(data);
    } else {
      showValidationErrors(form);
      // Re-enable
    }
  } finally {
    submitBtn.disabled = false;
    submitBtn.textContent = 'Submit';
  }
});
```

### Real-World Use Cases

- **Analytics**: Serializing event objects to track user behavior
- **Form Handling**: Using event.target to identify changed fields
- **Drag and Drop**: Tracking mouse coordinates for element movement
- **Keyboard Shortcuts**: Using event.key and modifier keys for hotkeys
- **Context Menus**: Using event.button to detect right-click
- **Infinite Scroll**: Using scroll event properties to detect bottom of page
- **Game Input**: Using keyboard and mouse event properties for controls

### Common Mistakes

```javascript
// Mistake 1: Using event.target where event.currentTarget is needed
list.addEventListener('click', (e) => {
  e.target.style.color = 'red'; // Colors the clicked child, not the list item
  e.currentTarget.style.color = 'blue'; // Colors the list itself
});

// Mistake 2: Modifying event object properties incorrectly
e.clientX = 100; // Read-only, will be ignored in strict mode

// Mistake 3: Calling preventDefault on non-cancelable events
document.addEventListener('DOMContentLoaded', (e) => {
  e.preventDefault(); // Does nothing, event is not cancelable
});

// Mistake 4: Not preventing default when needed
link.addEventListener('click', (e) => {
  // e.preventDefault(); // Missing - page will navigate
  executeSPARouting(e.target.href);
});
```

### Best Practices

- Use `event.target` to identify the originating element
- Use `event.currentTarget` to access the element with the listener
- Always call `preventDefault()` before async operations (it's synchronous)
- Check `event.cancelable` before calling `preventDefault()`
- Use `event.stopPropagation()` sparingly (can break event delegation)
- Prefer `CustomEvent` with `detail` for custom data passing
- Use `isTrusted` to distinguish real vs synthetic events when needed

### Performance Considerations

Accessing event object properties has negligible cost. However, creating custom event objects and dispatching them synchronously triggers the full event propagation pipeline, which includes handler execution for all registered listeners. For high-frequency events (like `mousemove` or `scroll`), minimize the work done inside handlers rather than worrying about event object access.

### Interview Questions

1. What is the difference between `event.target` and `event.currentTarget`?
2. What does `event.preventDefault()` do?
3. What is the `event.eventPhase` property?
4. How do you pass custom data with events?
5. What does `event.isTrusted` indicate?
6. What is the difference between `stopPropagation` and `stopImmediatePropagation`?
7. How do you check if `preventDefault` was called?
8. What is the `relatedTarget` property used for?

### Coding Challenges

1. **Event Serializer**: Create a function that serializes any event to JSON for logging
2. **Event History**: Build a recorder that captures and replays events
3. **Custom Gesture Recognizer**: Use mouse/touch events to detect swipe, pinch, rotate
4. **Event Validator**: Add runtime validation to event handlers

### Related Topics

- [Event.target](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)
- [Event.currentTarget](https://developer.mozilla.org/en-US/docs/Web/API/Event/currentTarget)
- [CustomEvent](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)
- [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)
- [KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
- [TouchEvent](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent)

## Event Types (click, keydown, submit, etc.)

### What It Is

The DOM specification defines dozens of event types, categorized by the kind of interaction they represent. Each event type has a corresponding interface (like `MouseEvent`, `KeyboardEvent`, `FocusEvent`) that provides type-specific properties. Understanding the available event types and their specific characteristics is essential for building interactive web applications.

### Why It Is Important

Different event types carry different data and behave differently. Choosing the right event type determines what information is available in the handler and how the interaction is processed. Using `change` instead of `input`, or `keydown` instead of `keypress`, can make the difference between a working and broken feature.

### How It Works Internally

The browser's event system classifies events into categories. Mouse events are generated by the platform's pointer system, keyboard events by the input system, and so on. Each event type is created as an instance of the appropriate interface and dispatched through the event loop.

```javascript
// Major event categories
// Mouse Events: click, dblclick, mousedown, mouseup, mousemove, mouseenter, mouseleave, mouseover, mouseout, contextmenu
// Keyboard Events: keydown, keyup, keypress (deprecated)
// Focus Events: focus, blur, focusin, focusout
// Form Events: submit, reset, change, input, select, invalid
// Document Events: DOMContentLoaded, readystatechange, beforeunload, unload
// Window Events: resize, scroll, load, error, hashchange, popstate
// Touch Events: touchstart, touchmove, touchend, touchcancel
// Pointer Events: pointerdown, pointerup, pointermove, pointerenter, pointerleave, pointercancel
// Drag Events: dragstart, drag, dragenter, dragleave, dragover, drop, dragend
// Clipboard Events: copy, cut, paste
// Animation Events: animationstart, animationend, animationiteration
// Transition Events: transitionstart, transitionend, transitionrun, transitioncancel
// Media Events: play, pause, ended, timeupdate, volumechange
// Network Events: online, offline
// Progress Events: loadstart, progress, load, loadend, error, abort
// Storage Events: storage
```

### Syntax

```javascript
// Mouse events
element.addEventListener('click', handler);    // Single click
element.addEventListener('dblclick', handler); // Double click
element.addEventListener('mousedown', handler); // Mouse button pressed
element.addEventListener('mouseup', handler);  // Mouse button released
element.addEventListener('mousemove', handler); // Mouse moved
element.addEventListener('mouseenter', handler); // Mouse enters element (no bubble)
element.addEventListener('mouseleave', handler); // Mouse leaves element (no bubble)
element.addEventListener('mouseover', handler); // Mouse enters element or child
element.addEventListener('mouseout', handler);  // Mouse leaves element or child
element.addEventListener('contextmenu', handler); // Right-click

// Keyboard events
document.addEventListener('keydown', handler); // Key pressed down
document.addEventListener('keyup', handler);   // Key released

// Focus events
element.addEventListener('focus', handler);   // Element gains focus (no bubble)
element.addEventListener('blur', handler);    // Element loses focus (no bubble)
element.addEventListener('focusin', handler); // Element gains focus (bubbles)
element.addEventListener('focusout', handler); // Element loses focus (bubbles)

// Form events
form.addEventListener('submit', handler);     // Form submitted
form.addEventListener('reset', handler);      // Form reset
input.addEventListener('change', handler);    // Value committed (after blur)
input.addEventListener('input', handler);     // Value changed (immediate)
input.addEventListener('invalid', handler);   // Validation failed

// Document/Window events
document.addEventListener('DOMContentLoaded', handler);
window.addEventListener('load', handler);
window.addEventListener('beforeunload', handler);
window.addEventListener('resize', handler);
window.addEventListener('scroll', handler, { passive: true });
window.addEventListener('hashchange', handler);
window.addEventListener('popstate', handler);
```

### Beginner Examples

```javascript
// Example 1: Click event variations
button.addEventListener('click', () => { /* Single click */ });
button.addEventListener('dblclick', () => { /* Double click */ });
button.addEventListener('contextmenu', (e) => {
  e.preventDefault(); // Prevent browser context menu
  showCustomMenu(e.clientX, e.clientY);
});

// Example 2: Keyboard events
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    closeModal();
  }
  if (e.key === 'Enter' && e.target.matches('input')) {
    submitForm();
  }
  // Ctrl+S to save
  if ((e.ctrlKey || e.metaKey) && e.key === 's') {
    e.preventDefault();
    saveDocument();
  }
});

// Example 3: Focus events
input.addEventListener('focus', () => {
  input.parentElement.classList.add('focused');
});
input.addEventListener('blur', () => {
  input.parentElement.classList.remove('focused');
  validateInput();
});

// Example 4: Form events
form.addEventListener('submit', (e) => {
  e.preventDefault();
  const data = new FormData(form);
  submitData(data);
});

input.addEventListener('input', () => {
  characterCounter.textContent = `${input.value.length}/100`;
});

input.addEventListener('change', () => {
  console.log('Final value:', input.value);
});

// Example 5: Window events
window.addEventListener('resize', () => {
  console.log('Viewport:', window.innerWidth, 'x', window.innerHeight);
});

window.addEventListener('scroll', () => {
  const scrollPercent = window.scrollY / (document.body.scrollHeight - window.innerHeight);
  progressBar.style.width = `${scrollPercent * 100}%`;
}, { passive: true });
```

### Intermediate Examples

```javascript
// Example 1: Distinguishing single from double click
function setupClickHandler(element, { onClick, onDblClick, delay = 250 }) {
  let clickTimer = null;

  element.addEventListener('click', (e) => {
    if (clickTimer) {
      clearTimeout(clickTimer);
      clickTimer = null;
      onDblClick(e);
    } else {
      clickTimer = setTimeout(() => {
        clickTimer = null;
        onClick(e);
      }, delay);
    }
  });
}

setupClickHandler(item, {
  onClick: (e) => console.log('Single click:', e.target),
  onDblClick: (e) => console.log('Double click:', e.target)
});

// Example 2: Cross-browser clipboard handling
document.addEventListener('paste', (e) => {
  const clipboardData = e.clipboardData || window.clipboardData;
  const pastedText = clipboardData.getData('text/plain');

  if (pastedText.length > 1000) {
    e.preventDefault(); // Block very large pastes
    showToast('Content too large to paste');
    return;
  }

  // Sanitize pasted content
  const sanitized = sanitizeText(pastedText);
  document.execCommand('insertText', false, sanitized);
});

// Example 3: Drag and drop file handling
const dropZone = document.querySelector('.drop-zone');

dropZone.addEventListener('dragenter', (e) => {
  e.preventDefault();
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragover', (e) => {
  e.preventDefault(); // Required for drop to work
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragleave', (e) => {
  e.preventDefault();
  dropZone.classList.remove('drag-over');
});

dropZone.addEventListener('drop', (e) => {
  e.preventDefault();
  dropZone.classList.remove('drag-over');

  const files = Array.from(e.dataTransfer.files);
  handleFiles(files);
});

// Example 4: Touch vs mouse detection
let isTouchDevice = false;

element.addEventListener('touchstart', () => {
  isTouchDevice = true;
}, { once: true });

element.addEventListener('click', (e) => {
  if (isTouchDevice) {
    // Handle touch interaction differently
    handleTouchInteraction(e);
  } else {
    handleMouseInteraction(e);
  }
});
```

### Advanced Examples

```javascript
// Example 1: Unified pointer interface
class UnifiedPointer {
  constructor(element) {
    this.element = element;
    this.isDown = false;
    this.startPos = null;
    this.currentPos = null;
  }

  start() {
    if ('PointerEvent' in window) {
      this.setupPointerEvents();
    } else if ('ontouchstart' in window) {
      this.setupTouchEvents();
    } else {
      this.setupMouseEvents();
    }
  }

  setupPointerEvents() {
    this.element.addEventListener('pointerdown', (e) => this.onDown(e));
    document.addEventListener('pointermove', (e) => this.onMove(e));
    document.addEventListener('pointerup', (e) => this.onUp(e));
  }

  setupMouseEvents() {
    this.element.addEventListener('mousedown', (e) => this.onDown(e));
    document.addEventListener('mousemove', (e) => this.onMove(e));
    document.addEventListener('mouseup', (e) => this.onUp(e));
  }

  setupTouchEvents() {
    this.element.addEventListener('touchstart', (e) => {
      const touch = e.touches[0];
      this.onDown({ clientX: touch.clientX, clientY: touch.clientY, isTouch: true });
    });
    document.addEventListener('touchmove', (e) => {
      const touch = e.touches[0];
      this.onMove({ clientX: touch.clientX, clientY: touch.clientY, isTouch: true });
    });
    document.addEventListener('touchend', (e) => this.onUp({ isTouch: true }));
  }

  onDown(pos) {
    this.isDown = true;
    this.startPos = { x: pos.clientX, y: pos.clientY };
    this.element.dispatchEvent(new CustomEvent('pointer-start', { detail: pos }));
  }

  onMove(pos) {
    if (!this.isDown) return;
    this.currentPos = { x: pos.clientX, y: pos.clientY };
    this.element.dispatchEvent(new CustomEvent('pointer-move', {
      detail: { ...pos, dx: pos.clientX - this.startPos.x, dy: pos.clientY - this.startPos.y }
    }));
  }

  onUp(pos) {
    if (!this.isDown) return;
    this.isDown = false;
    this.element.dispatchEvent(new CustomEvent('pointer-end', {
      detail: { ...pos, dx: pos.clientX - this.startPos.x, dy: pos.clientY - this.startPos.y }
    }));
  }
}

const pointer = new UnifiedPointer(document.querySelector('.draggable'));
pointer.start();
pointer.element.addEventListener('pointer-move', (e) => {
  element.style.transform = `translate(${e.detail.dx}px, ${e.detail.dy}px)`;
});

// Example 2: Event-driven undo/redo system
class UndoManager {
  constructor() {
    this.undoStack = [];
    this.redoStack = [];
    this.maxSize = 50;

    document.addEventListener('keydown', (e) => {
      if ((e.ctrlKey || e.metaKey) && e.key === 'z') {
        e.preventDefault();
        if (e.shiftKey) {
          this.redo();
        } else {
          this.undo();
        }
      }
    });
  }

  record(action) {
    this.undoStack.push(action);
    this.redoStack = []; // Clear redo on new action
    if (this.undoStack.length > this.maxSize) {
      this.undoStack.shift();
    }
  }

  undo() {
    const action = this.undoStack.pop();
    if (action) {
      action.undo();
      this.redoStack.push(action);
    }
  }

  redo() {
    const action = this.redoStack.pop();
    if (action) {
      action.redo();
      this.undoStack.push(action);
    }
  }
}

// Example 3: Custom event dispatcher for component communication
class EventBus {
  constructor() {
    this.target = document.createElement('div');
  }

  on(type, handler, options) {
    this.target.addEventListener(type, handler, options);
    return () => this.target.removeEventListener(type, handler);
  }

  emit(type, detail = {}) {
    this.target.dispatchEvent(new CustomEvent(type, {
      detail,
      bubbles: false,
      cancelable: true
    }));
  }

  once(type, handler) {
    return this.on(type, handler, { once: true });
  }
}

const bus = new EventBus();
const unsub = bus.on('user:login', (e) => {
  console.log('User logged in:', e.detail);
});
bus.emit('user:login', { id: 123, name: 'John' });
```

### Real-World Use Cases

- **Click**: Button actions, link navigation, toggles
- **Keydown**: Keyboard shortcuts, game controls, form navigation
- **Submit**: Form submission handling and validation
- **Input**: Live search, character counting, form auto-save
- **Change**: Dropdown selection, file input handling
- **Scroll**: Lazy loading, parallax, sticky headers, scroll-spy
- **Resize**: Responsive layout adjustments, canvas resizing
- **Focus/Blur**: Form field validation, UI state management
- **Drag/Drop**: File upload, reorderable lists, kanban boards
- **Copy/Paste**: Enhanced clipboard handling, rich text editing
- **Online/Offline**: Network status detection, offline app mode
- **BeforeUnload**: Unsaved changes warning
- **PopState**: SPA routing with browser back/forward

### Common Mistakes

```javascript
// Mistake 1: Using 'click' for mobile (may have 300ms delay)
// Use 'touchend' or pointer events for mobile

// Mistake 2: Not checking keyboard event modifiers correctly
if (e.key === 'a' && e.ctrlKey) { /* Ctrl+A */ }
// But: CapsLock affects e.key, not just shift

// Mistake 3: Using 'load' instead of 'DOMContentLoaded'
window.addEventListener('load', () => {
  // Waits for all resources (images, styles)
});
document.addEventListener('DOMContentLoaded', () => {
  // Fires when HTML is parsed, before resources loaded
});

// Mistake 4: Not preventing default for form submit in validation
form.addEventListener('submit', (e) => {
  // If validation fails:
  e.preventDefault(); // Must be called synchronously
});
```

### Best Practices

- Use `pointer` events for unified mouse/touch handling
- Prefer `input` over `change` for real-time value tracking
- Use `submit` on form elements rather than click on submit buttons
- Always call `preventDefault()` synchronously in async handlers
- Use `focusin`/`focusout` instead of `focus`/`blur` when you need bubbling
- Use `passive: true` for scroll/touch events when not preventing default
- Prefer `DOMContentLoaded` over `load` for DOM initialization

### Performance Considerations

High-frequency events like `scroll`, `resize`, `mousemove`, and `input` can fire many times per second. Debouncing or throttling handlers is essential for performance. Passive listeners tell the browser it doesn't need to wait for the handler before performing the default action.

```javascript
// Debounced scroll handler
let scrollTimeout;
window.addEventListener('scroll', () => {
  clearTimeout(scrollTimeout);
  scrollTimeout = setTimeout(() => {
    // Expensive scroll handling
    updateLayout();
  }, 100);
}, { passive: true });

// Throttled resize handler
let lastResizeTime = 0;
window.addEventListener('resize', () => {
  const now = Date.now();
  if (now - lastResizeTime >= 200) {
    lastResizeTime = now;
    handleResize();
  }
});
```

### Interview Questions

1. What is the difference between `mouseenter`/`mouseleave` and `mouseover`/`mouseout`?
2. When should you use `input` vs `change` events?
3. What is the difference between `focus`/`blur` and `focusin`/`focusout`?
4. How do you handle keyboard shortcuts with modifiers?
5. What events are fired during a drag-and-drop operation?
6. What is the `submit` event and how is it different from clicking a submit button?
7. What events fire when a user copies text?
8. What is `beforeunload` and how do you use it?

### Coding Challenges

1. **Gesture Recognizer**: Implement swipe, pinch, and rotate using touch events
2. **Keyboard Shortcut Manager**: Build a declarative keyboard shortcut system
3. **Event Simulator**: Create a utility that fires all event types on an element for testing
4. **Form Auto-Saver**: Implement an auto-save feature using input events and debouncing

### Related Topics

- [Event Reference (MDN)](https://developer.mozilla.org/en-US/docs/Web/Events)
- [Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)
- [Pointer Events](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events)
- [Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)
- [Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API)

## Event Handler Removal

### What It Is

Event handler removal is the process of detaching a previously registered event listener using `removeEventListener()`. The method requires the same event type, handler function reference, and options (capture flag) that were used during registration. Proper removal prevents memory leaks and unintended behavior in long-running web applications.

```javascript
function handler() { console.log('Clicked'); }
button.addEventListener('click', handler);
// Later:
button.removeEventListener('click', handler);
```

### Why It Is Important

Removing event listeners is critical for preventing memory leaks in single-page applications where components are dynamically created and destroyed. Without proper cleanup, detached DOM elements cannot be garbage collected because the event listener holds a reference to them. This is a common source of performance degradation in long-lived applications.

### How It Works Internally

When `removeEventListener` is called, the browser searches the element's internal listener map for a matching entry (same type, same function reference, same capture flag) and removes it. If no matching entry is found, the call is silently ignored. The event loop must complete all currently executing handlers before the removal takes effect for future events.

```javascript
// Conceptual:
// element._listeners['click'] = [
//   { handler: fn1, capture: false },
//   { handler: fn2, capture: true }
// ];
//
// removeEventListener('click', fn1, false)
// -> element._listeners['click'] = [
//   { handler: fn2, capture: true }
// ];
```

### Syntax

```javascript
// Remove event listener
target.removeEventListener(type, listener);
target.removeEventListener(type, listener, options);
target.removeEventListener(type, listener, useCapture);

// The options/capture must match the ones used in addEventListener
element.addEventListener('click', handler, { capture: true });
element.removeEventListener('click', handler, { capture: true }); // Must match

element.addEventListener('click', handler); // capture defaults to false
element.removeEventListener('click', handler); // Must omit or pass false
```

### Beginner Examples

```javascript
// Example 1: Basic removal
function handleClick() {
  console.log('Clicked');
}

const btn = document.querySelector('button');
btn.addEventListener('click', handleClick);

// Remove after first click
btn.removeEventListener('click', handleClick);

// Example 2: One-time event (manual)
function handleOnce() {
  console.log('This runs only once');
  btn.removeEventListener('click', handleOnce);
}
btn.addEventListener('click', handleOnce);

// Example 3: Using once option (preferred)
btn.addEventListener('click', () => {
  console.log('Runs once');
}, { once: true });

// Example 4: Cleanup on component destroy
class Component {
  constructor(element) {
    this.element = element;
    this.handleClick = this.handleClick.bind(this);
    this.element.addEventListener('click', this.handleClick);
  }

  handleClick(event) {
    console.log('Component clicked', event);
  }

  destroy() {
    this.element.removeEventListener('click', this.handleClick);
  }
}
```

### Intermediate Examples

```javascript
// Example 1: Self-cleaning event handler
function listenOnce(target, type, handler, options = {}) {
  const wrappedHandler = (event) => {
    handler(event);
    target.removeEventListener(type, wrappedHandler, options);
  };
  target.addEventListener(type, wrappedHandler, options);
  return () => target.removeEventListener(type, wrappedHandler, options);
}

listenOnce(button, 'click', () => console.log('First click only'));

// Example 2: Event cleanup utility
class EventManager {
  constructor() {
    this.listeners = [];
  }

  add(target, type, handler, options) {
    target.addEventListener(type, handler, options);
    this.listeners.push({ target, type, handler, options });
    return this;
  }

  remove(target, type, handler, options) {
    target.removeEventListener(type, handler, options);
    this.listeners = this.listeners.filter(l =>
      !(l.target === target && l.type === type && l.handler === handler)
    );
  }

  removeAll() {
    for (const { target, type, handler, options } of this.listeners) {
      target.removeEventListener(type, handler, options);
    }
    this.listeners = [];
  }
}

const events = new EventManager();
events.add(button, 'click', handleClick);
events.add(window, 'resize', handleResize);
events.add(document, 'keydown', handleKeydown);

// Clean up all at once
events.removeAll();

// Example 3: Using AbortController for cleanup
function setupWithAbort(target, type, handler) {
  const controller = new AbortController();
  target.addEventListener(type, handler, { signal: controller.signal });
  return () => controller.abort();
}

const cleanup = setupWithAbort(button, 'click', handleClick);
// Later:
cleanup();

// Multiple listeners with same AbortController
const controller = new AbortController();
const signal = controller.signal;

window.addEventListener('resize', handleResize, { signal });
window.addEventListener('scroll', handleScroll, { signal });
document.addEventListener('keydown', handleKeydown, { signal });

// Abort all at once
controller.abort();

// Example 4: Conditional handler removal
function createToggleHandler(element, onHandler, offHandler) {
  let isActive = false;
  let currentHandler = null;

  function toggle() {
    if (isActive) {
      element.removeEventListener('click', currentHandler);
      currentHandler = offHandler;
    } else {
      element.removeEventListener('click', currentHandler);
      currentHandler = onHandler;
    }
    element.addEventListener('click', currentHandler);
    isActive = !isActive;
  }

  currentHandler = onHandler;
  element.addEventListener('click', currentHandler);

  return { toggle, isActive: () => isActive };
}

const toggleable = createToggleHandler(
  button,
  () => console.log('Mode A'),
  () => console.log('Mode B')
);
toggleable.toggle(); // Switch to Mode B
```

### Advanced Examples

```javascript
// Example 1: Automatic cleanup with WeakRef (for frameworks)
function autoCleanup(target, type, handler, options) {
  const ref = new WeakRef(target);
  target.addEventListener(type, handler, options);

  // Return cleanup that works even if target reference is lost
  return () => {
    const element = ref.deref();
    if (element) {
      element.removeEventListener(type, handler, options);
    }
  };
}

// Example 2: Event listener registry for component lifecycle
class ComponentLifecycle {
  constructor() {
    this.registry = new FinalizationRegistry(({ element, type, handler }) => {
      // Automatic cleanup if component is garbage collected
      if (element && element.removeEventListener) {
        try {
          element.removeEventListener(type, handler);
        } catch (e) {
          // Element already collected
        }
      }
    });
  }

  addEventListener(component, element, type, handler, options) {
    const boundHandler = handler.bind(component);
    element.addEventListener(type, boundHandler, options);

    // Register for cleanup when component is GC'd
    this.registry.register(component, { element, type, handler: boundHandler });

    return () => {
      element.removeEventListener(type, boundHandler, options);
      this.registry.unregister(component);
    };
  }
}

// Example 3: Event listener profiler
function createListenerProfiler() {
  const originalAdd = EventTarget.prototype.addEventListener;
  const originalRemove = EventTarget.prototype.removeEventListener;
  const listenerCount = new Map();

  EventTarget.prototype.addEventListener = function (type, handler, options) {
    const key = `${this.tagName || 'window'}#${type}`;
    listenerCount.set(key, (listenerCount.get(key) || 0) + 1);
    console.log(`[Listener: added] ${key} (total: ${listenerCount.get(key)})`);
    return originalAdd.call(this, type, handler, options);
  };

  EventTarget.prototype.removeEventListener = function (type, handler, options) {
    const key = `${this.tagName || 'window'}#${type}`;
    const count = listenerCount.get(key) || 0;
    if (count > 0) {
      listenerCount.set(key, count - 1);
    }
    console.log(`[Listener: removed] ${key} (total: ${listenerCount.get(key)})`);
    return originalRemove.call(this, type, handler, options);
  };

  return {
    getCounts: () => Object.fromEntries(listenerCount),
    restore: () => {
      EventTarget.prototype.addEventListener = originalAdd;
      EventTarget.prototype.removeEventListener = originalRemove;
    }
  };
}

// Usage during development
const profiler = createListenerProfiler();
// ... run app ...
// profiler.restore(); // Restore original methods
```

### Real-World Use Cases

- **Single Page Applications**: Cleaning up listeners when navigating between routes
- **Modal Dialogs**: Removing backdrop click listeners when closing
- **Infinite Scroll**: Detaching scroll listeners when all content is loaded
- **WebSocket Connections**: Removing event handlers on disconnect
- **Resize Observers**: Cleaning up resize handlers when component unmounts
- **Drag and Drop**: Removing temporary mousemove/mouseup listeners after drag ends
- **Animations**: Removing animationend listeners after the animation completes

### Common Mistakes

```javascript
// Mistake 1: Cannot remove anonymous functions
element.addEventListener('click', () => console.log('Click'));
element.removeEventListener('click', () => console.log('Click')); // Won't work!

// Mistake 2: Options mismatch
element.addEventListener('click', handler, true); // capture: true
element.removeEventListener('click', handler); // Default capture: false - won't remove!

// Mistake 3: Forgetting .bind() creates a new function each time
class Widget {
  setup() {
    // Wrong: bind creates new function each time
    element.addEventListener('click', this.handleClick.bind(this));
    element.removeEventListener('click', this.handleClick.bind(this)); // Different reference!
  }

  // Correct: bind once, store reference
  constructor() {
    this.handleClick = this.handleClick.bind(this);
  }

  setupCorrect() {
    element.addEventListener('click', this.handleClick);
    element.removeEventListener('click', this.handleClick); // Same reference
  }
}

// Mistake 4: Memory leak - not removing listeners
function createWidget() {
  const element = document.createElement('div');
  element.addEventListener('click', () => {
    console.log('Widget clicked');
  });
  document.body.appendChild(element);
  // When element is removed, the listener still exists in memory
  // until element is GC'd, but the closure keeps references
}
```

### Best Practices

- Store handler function references for later removal
- Use `AbortController` for managing groups of listeners
- Prefer `{ once: true }` option for one-shot handlers
- Always clean up listeners in component `destroy`/`unmount` methods
- Use `WeakRef` or `FinalizationRegistry` for framework-level cleanup
- Bind methods in constructor, not in `addEventListener` calls
- Use utility classes like `EventManager` for tracking active listeners

### Performance Considerations

Unremoved event listeners on detached DOM elements are a primary cause of memory leaks in web applications. Each listener holds a reference to the handler function and its closure scope. Modern browsers have improved garbage collection for circular references, but the listener-to-element reference chain keeps the handler's closure alive.

```javascript
// Bad: Memory leak
function setup() {
  const largeData = new Array(1000000).fill('data');
  element.addEventListener('click', () => {
    console.log(largeData.length); // Closure keeps largeData alive
  });
}

// Good: Clean up
function setup() {
  const largeData = new Array(1000000).fill('data');
  const handler = () => console.log(largeData.length);
  element.addEventListener('click', handler);
  return () => {
    element.removeEventListener('click', handler);
  };
}
const cleanup = setup();
// Later: cleanup();
```

### Interview Questions

1. Why can't you remove an anonymous event listener?
2. How does `removeEventListener` determine which handler to remove?
3. What happens if you call `removeEventListener` with a handler that wasn't added?
4. How does `AbortController` simplify listener cleanup?
5. Why is it important to remove event listeners in SPAs?
6. What is the memory leak pattern with closures in event handlers?
7. How do you properly remove listeners added with `{ capture: true }`?
8. What happens to event listeners when an element is removed from the DOM?

### Coding Challenges

1. **Event Leak Detector**: Build a tool that detects and reports unremoved event listeners
2. **Auto-Cleanup Component**: Create a base component class that automatically manages listener lifecycle
3. **Listener Tracker**: Implement an event listener registry with automatic cleanup via FinalizationRegistry
4. **Safe Event Binding**: Create a wrapper that prevents double-listener bugs

### Related Topics

- [removeEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)
- [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)
- [FinalizationRegistry](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry)
- [WeakRef](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef)
