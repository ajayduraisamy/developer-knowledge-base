# Forms - Form events, validation, FormData, controlled inputs

## Introduction

Forms are the primary mechanism for collecting user input on the web. The DOM provides a rich set of APIs for handling form interactions, validating input, serializing data, and managing form state. Modern web applications rely on forms for everything from login and registration to complex multi-step data entry workflows.

Understanding the form event system, the Constraint Validation API, the FormData interface, and input control patterns is essential for building robust, user-friendly forms. This knowledge applies equally to plain JavaScript applications and form handling within frameworks like React, Vue, or Angular.

## Form Events (submit, change, input)

### What It Is

Form-related events fire at various stages of user interaction. The primary events are `submit` (form submission), `change` (value committed), and `input` (value changed in real-time). Additional events include `focus`, `blur`, `focusin`, `focusout`, `reset`, `invalid`, and `select`. Each event provides specific timing and context for handling user input.

```javascript
form.addEventListener('submit', (e) => {
  e.preventDefault();
  handleSubmit(new FormData(form));
});

input.addEventListener('input', (e) => {
  console.log('Real-time value:', e.target.value);
});

input.addEventListener('change', (e) => {
  console.log('Final value (committed):', e.target.value);
});
```

### Why It Is Important

The choice of event determines your application's responsiveness and user experience. Using `input` for real-time validation provides immediate feedback, while `change` is better for non-critical updates. The `submit` event is the only reliable way to intercept form submissions across all scenarios (button click, Enter key, `form.requestSubmit()`).

### How It Works Internally

When a user submits a form (by pressing Enter in a field or clicking a submit button), the browser fires a `submit` event on the `<form>` element. Before submission, the browser runs constraint validation on all form elements. The `input` event fires on every value change (including paste, undo, drag-and-drop text insertion). The `change` event fires when the value is committed (blur for text inputs, immediately for checkboxes/radio/select).

```javascript
// Input event sequence for text input:
// 1. focus
// 2. keydown (each key)
// 3. input (each character)
// 4. keyup (each key)
// 5. change (on blur, if value changed)
// 6. blur
```

### Syntax

```javascript
// Submit event
form.addEventListener('submit', (event) => {
  event.preventDefault(); // Prevent page reload
  // Handle submission
});

// Input event (fires on every value change)
input.addEventListener('input', (event) => {
  console.log(event.target.value);
  console.log(event.data); // The character(s) inserted
  console.log(event.inputType); // 'insertText', 'deleteContentBackward', etc.
});

// Change event (fires when value is committed)
input.addEventListener('change', (event) => {
  if (event.target.value !== event.target.defaultValue) {
    console.log('Value changed');
  }
});

// Reset event
form.addEventListener('reset', (event) => {
  if (!confirm('Reset form?')) {
    event.preventDefault();
  }
});

// Invalid event (fires on each invalid element during validation)
element.addEventListener('invalid', (event) => {
  event.preventDefault(); // Prevent browser validation bubble
  showCustomError(event.target);
});
```

### Beginner Examples

```javascript
// Example 1: Basic form submission handling
const loginForm = document.querySelector('#login-form');
loginForm.addEventListener('submit', (event) => {
  event.preventDefault();

  const formData = new FormData(loginForm);
  const data = Object.fromEntries(formData.entries());

  fetch('/api/login', {
    method: 'POST',
    body: JSON.stringify(data),
    headers: { 'Content-Type': 'application/json' }
  })
    .then(res => res.json())
    .then(result => {
      if (result.success) {
        window.location.href = '/dashboard';
      } else {
        showError(result.message);
      }
    });
});

// Example 2: Real-time character counter
const textarea = document.querySelector('#bio');
const counter = document.querySelector('#char-count');
const maxChars = 160;

textarea.addEventListener('input', () => {
  const remaining = maxChars - textarea.value.length;
  counter.textContent = `${remaining} characters remaining`;
  counter.style.color = remaining < 20 ? 'red' : 'inherit';

  if (remaining < 0) {
    textarea.value = textarea.value.slice(0, maxChars);
  }
});

// Example 3: Live search with debounced input
const searchInput = document.querySelector('#search');
const results = document.querySelector('#results');
let debounceTimer;

searchInput.addEventListener('input', () => {
  clearTimeout(debounceTimer);

  const query = searchInput.value.trim();
  if (query.length < 2) {
    results.innerHTML = '';
    return;
  }

  debounceTimer = setTimeout(() => {
    performSearch(query);
  }, 300);
});

async function performSearch(query) {
  const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
  const data = await response.json();
  renderResults(data);
}

// Example 4: Checkbox/radio change events
document.querySelector('#newsletter').addEventListener('change', (event) => {
  const emailField = document.querySelector('#email');
  emailField.disabled = !event.target.checked;

  if (!event.target.checked) {
    emailField.value = '';
  }
});

document.querySelectorAll('input[name="theme"]').forEach(radio => {
  radio.addEventListener('change', (event) => {
    if (event.target.checked) {
      document.documentElement.className = `theme-${event.target.value}`;
    }
  });
});
```

### Intermediate Examples

```javascript
// Example 1: Form state management
class FormState {
  constructor(form) {
    this.form = form;
    this.initialData = this.serialize();
    this.setup();
  }

  setup() {
    this.form.addEventListener('input', () => this.checkDirty());
    this.form.addEventListener('change', () => this.checkDirty());

    // Warn before leaving with unsaved changes
    window.addEventListener('beforeunload', (event) => {
      if (this.isDirty()) {
        event.preventDefault();
        event.returnValue = ''; // Required for legacy browsers
      }
    });
  }

  serialize() {
    const data = new FormData(this.form);
    return JSON.stringify([...data.entries()]);
  }

  isDirty() {
    return this.serialize() !== this.initialData;
  }

  checkDirty() {
    const submitBtn = this.form.querySelector('[type="submit"]');
    if (submitBtn) {
      submitBtn.disabled = !this.isDirty();
    }
  }

  reset() {
    this.form.reset();
    this.initialData = this.serialize();
    this.checkDirty();
  }
}

const formState = new FormState(document.querySelector('#settings-form'));

// Example 2: Multi-step form wizard
class FormWizard {
  constructor(form, steps) {
    this.form = form;
    this.steps = steps;
    this.currentStep = 0;
    this.data = {};
    this.setup();
  }

  setup() {
    this.form.addEventListener('submit', (event) => {
      event.preventDefault();

      if (this.currentStep < this.steps.length - 1) {
        if (this.validateStep()) {
          this.saveStep();
          this.goToStep(this.currentStep + 1);
        }
      } else {
        this.submitAll();
      }
    });

    // Handle Enter key for navigation
    this.form.addEventListener('keydown', (event) => {
      if (event.key === 'Enter' && !event.target.matches('textarea')) {
        event.preventDefault();
        this.form.querySelector('[type="submit"]').click();
      }
    });

    this.goToStep(0);
  }

  goToStep(index) {
    this.steps.forEach((step, i) => {
      step.classList.toggle('active', i === index);
      step.querySelectorAll('input, select, textarea').forEach(el => {
        el.required = i === index; // Only current step is required
      });
    });
    this.currentStep = index;
    this.updateButtons();
  }

  saveStep() {
    const step = this.steps[this.currentStep];
    const formData = new FormData(step);
    formData.forEach((value, key) => {
      this.data[key] = value;
    });
  }

  validateStep() {
    const step = this.steps[this.currentStep];
    return step.checkValidity();
  }

  submitAll() {
    this.saveStep();
    console.log('Final data:', this.data);
    // Submit all data
  }

  updateButtons() {
    const submitBtn = this.form.querySelector('[type="submit"]');
    if (this.currentStep === this.steps.length - 1) {
      submitBtn.textContent = 'Submit';
    } else {
      submitBtn.textContent = 'Next';
    }
  }
}

// new FormWizard(document.querySelector('#wizard'), [
//   document.querySelector('#step-1'),
//   document.querySelector('#step-2'),
//   document.querySelector('#step-3')
// ]);

// Example 3: Auto-save with input events
class AutoSave {
  constructor(form, options = {}) {
    this.form = form;
    this.delay = options.delay || 2000;
    this.saveKey = options.saveKey || 'form-auto-save';
    this.timer = null;
    this.setup();
  }

  setup() {
    this.form.addEventListener('input', () => {
      clearTimeout(this.timer);
      this.timer = setTimeout(() => this.save(), this.delay);
    });

    this.form.addEventListener('submit', () => {
      this.clearSaved();
    });

    // Restore saved data
    this.restore();
  }

  save() {
    const formData = new FormData(this.form);
    const data = {};

    formData.forEach((value, key) => {
      data[key] = value;
    });

    localStorage.setItem(this.saveKey, JSON.stringify(data));
    console.log('Auto-saved');
  }

  restore() {
    const saved = localStorage.getItem(this.saveKey);
    if (!saved) return;

    const data = JSON.parse(saved);
    for (const [name, value] of Object.entries(data)) {
      const field = this.form.querySelector(`[name="${name}"]`);
      if (field) field.value = value;
    }
  }

  clearSaved() {
    localStorage.removeItem(this.saveKey);
  }
}
```

### Advanced Examples

```javascript
// Example 1: Undo/redo for form inputs
class FormUndoRedo {
  constructor(form) {
    this.form = form;
    this.undoStack = [];
    this.redoStack = [];
    this.maxSize = 50;
    this.isUndoing = false;
    this.setup();
  }

  setup() {
    this.form.addEventListener('input', (event) => {
      if (this.isUndoing) return;

      const field = event.target;
      if (!field.name) return;

      this.undoStack.push({
        field: field.name,
        oldValue: field.dataset.previousValue || field.defaultValue,
        newValue: field.value
      });

      if (this.undoStack.length > this.maxSize) {
        this.undoStack.shift();
      }
      this.redoStack = [];

      field.dataset.previousValue = field.value;
    });

    this.form.addEventListener('focusin', (event) => {
      const field = event.target;
      if (field.name) {
        field.dataset.previousValue = field.value;
      }
    });

    // Keyboard shortcuts
    this.form.addEventListener('keydown', (event) => {
      if ((event.ctrlKey || event.metaKey) && event.key === 'z') {
        event.preventDefault();
        if (event.shiftKey) {
          this.redo();
        } else {
          this.undo();
        }
      }
    });
  }

  undo() {
    const action = this.undoStack.pop();
    if (!action) return;

    this.isUndoing = true;
    const field = this.form.querySelector(`[name="${action.field}"]`);
    if (field) {
      field.value = action.oldValue;
      field.dispatchEvent(new Event('input', { bubbles: true }));
    }
    this.isUndoing = false;
    this.redoStack.push(action);
  }

  redo() {
    const action = this.redoStack.pop();
    if (!action) return;

    this.isUndoing = true;
    const field = this.form.querySelector(`[name="${action.field}"]`);
    if (field) {
      field.value = action.newValue;
      field.dispatchEvent(new Event('input', { bubbles: true }));
    }
    this.isUndoing = false;
    this.undoStack.push(action);
  }
}

// Example 2: Custom input type detection and handling
class SmartFormHandler {
  constructor(form) {
    this.form = form;
    this.setup();
  }

  setup() {
    this.form.addEventListener('input', (event) => {
      const field = event.target;
      const handler = this.getHandler(field);
      if (handler) {
        handler(field, event);
      }
    });
  }

  getHandler(field) {
    const handlers = {
      // URL input: auto-prefix protocol
      url: (f) => {
        if (f.value && !f.value.startsWith('http://') && !f.value.startsWith('https://')) {
          f.value = 'https://' + f.value;
        }
      },
      // Phone: auto-format
      tel: (f) => {
        f.value = f.value.replace(/[^\d+()-]/g, '');
      },
      // Number: clamp to range
      number: (f) => {
        const min = parseFloat(f.min);
        const max = parseFloat(f.max);
        if (!isNaN(min) && parseFloat(f.value) < min) f.value = min;
        if (!isNaN(max) && parseFloat(f.value) > max) f.value = max;
      }
    };

    return handlers[field.type] || null;
  }
}

// Example 3: Cross-field validation with events
class CrossFieldValidator {
  constructor(form) {
    this.form = form;
    this.setup();
  }

  setup() {
    // Validate on input with debounce
    this.form.addEventListener('input', () => {
      this.validateAll();
    });

    // Validate on blur for individual fields
    this.form.addEventListener('focusout', (event) => {
      this.validateField(event.target);
    });
  }

  validateAll() {
    const fields = this.form.querySelectorAll('input, select, textarea');
    for (const field of fields) {
      this.validateField(field);
    }
  }

  validateField(field) {
    const errors = [];

    // Built-in validation
    if (!field.checkValidity()) {
      errors.push(field.validationMessage);
    }

    // Cross-field rules
    if (field.name === 'password') {
      this.validatePassword(field, errors);
    } else if (field.name === 'confirm_password') {
      this.validateConfirmPassword(field, errors);
    } else if (field.name === 'email') {
      this.validateEmail(field, errors);
    }

    this.showErrors(field, errors);
    return errors.length === 0;
  }

  validatePassword(field, errors) {
    const value = field.value;

    if (value.length < 8) {
      errors.push('Password must be at least 8 characters');
    }
    if (!/[A-Z]/.test(value)) {
      errors.push('Password must contain an uppercase letter');
    }
    if (!/[0-9]/.test(value)) {
      errors.push('Password must contain a number');
    }

    // Check confirm password if it exists
    const confirm = this.form.querySelector('[name="confirm_password"]');
    if (confirm && confirm.value) {
      this.validateConfirmPassword(confirm, []);
    }
  }

  validateConfirmPassword(field, errors) {
    const password = this.form.querySelector('[name="password"]')?.value;
    if (field.value !== password) {
      errors.push('Passwords do not match');
    }
  }

  validateEmail(field, errors) {
    // Check if email is already taken (async)
    if (field.value && field.validity.valid) {
      this.checkEmailAvailability(field.value).then(taken => {
        if (taken) {
          this.showErrors(field, ['Email is already registered']);
        }
      });
    }
  }

  async checkEmailAvailability(email) {
    const response = await fetch(`/api/check-email?email=${encodeURIComponent(email)}`);
    const data = await response.json();
    return data.taken;
  }

  showErrors(field, errors) {
    const errorEl = field.parentElement.querySelector('.error-message')
      || this.createErrorElement(field);

    if (errors.length > 0) {
      errorEl.textContent = errors[0];
      errorEl.style.display = 'block';
      field.classList.add('invalid');
      field.classList.remove('valid');
    } else {
      errorEl.style.display = 'none';
      field.classList.remove('invalid');
      field.classList.add('valid');
    }
  }

  createErrorElement(field) {
    const errorEl = document.createElement('div');
    errorEl.className = 'error-message';
    errorEl.style.color = 'red';
    errorEl.style.fontSize = '0.85em';
    field.parentElement.appendChild(errorEl);
    return errorEl;
  }
}
```

### Real-World Use Cases

- **Login/Registration**: Submit handling, real-time password strength, email validation
- **Checkout Forms**: Multi-step validation, credit card formatting, address autocomplete
- **Search**: Debounced input for live search with suggestions
- **Settings Pages**: Auto-save on change, dirty state tracking for unsaved changes
- **Content Creation**: Auto-save drafts with debounce, character/word counts
- **Data Tables**: Inline editing with blur-triggered saves
- **Survey/Quiz Tools**: Step validation, progress tracking, conditional questions

### Common Mistakes

```javascript
// Mistake 1: Not preventing default form submission
form.addEventListener('submit', (e) => {
  // e.preventDefault(); // Missing! Page will reload
  handleAjaxSubmit(form);
});

// Mistake 2: Using click on submit button instead of submit on form
submitBtn.addEventListener('click', (e) => {
  // Won't catch Enter key submission
  handleSubmit();
});
// Better: Listen on form submit event

// Mistake 3: Input event for non-text inputs
checkbox.addEventListener('input', handler); // Works but change is more semantic
checkbox.addEventListener('change', handler); // Better

// Mistake 4: Forgetting that input fires on programmatic changes
input.value = 'new value';
input.dispatchEvent(new Event('input', { bubbles: true })); // Must fire manually

// Mistake 5: Not cleaning up debounce timers
input.addEventListener('input', () => {
  setTimeout(() => { // Timer not cleaned up
    // This might run after component unmount
  }, 1000);
});
```

### Best Practices

- Always listen on `submit` event, not `click` on submit buttons
- Use `input` for real-time feedback and `change` for committed values
- Debounce `input` handlers for search/API calls (300ms recommended)
- Use `preventDefault()` in submit handler for AJAX forms
- Store timers for cleanup to avoid memory leaks
- Use `event.data` and `event.inputType` for granular input control
- Handle `beforeunload` for forms with unsaved changes
- Programmatically dispatch `input` event when changing values in code

### Performance Considerations

The `input` event fires on every keystroke (including fast typing). Heavy handlers can cause jank. Debounce expensive operations like API calls. The `change` event fires less frequently (once per field completion) and is safer for non-critical operations.

```javascript
// Bad: API call on every keystroke
input.addEventListener('input', () => fetchSearch(input.value));

// Good: Debounced API call
let timer;
input.addEventListener('input', () => {
  clearTimeout(timer);
  timer = setTimeout(() => fetchSearch(input.value), 300);
});

// Best: Debounce with trailing edge and max wait
function debounce(fn, delay, maxWait) {
  let timer, lastCall = 0;
  return (...args) => {
    const now = Date.now();
    clearTimeout(timer);
    if (now - lastCall >= maxWait) {
      fn(...args);
      lastCall = now;
    } else {
      timer = setTimeout(() => fn(...args), delay);
    }
  };
}
```

### Interview Questions

1. What is the difference between `input` and `change` events?
2. How do you prevent a form from submitting?
3. What is `event.inputType` and what values can it have?
4. Why should you use `submit` event instead of `click` on submit button?
5. How do you handle form submission with Enter key?
6. What is `form.requestSubmit()` and how is it different from `form.submit()`?
7. How do you detect unsaved changes in a form?
8. What events fire when a user pastes text into an input?

### Coding Challenges

1. **Form Auto-Save**: Implement auto-save to localStorage with debounced input
2. **Multi-Step Wizard**: Build a form wizard with step validation and data accumulation
3. **Undo/Redo for Forms**: Implement undo/redo for all form fields
4. **Smart Input**: Create inputs that auto-format (phone, credit card, date)
5. **Conditional Form**: Show/hide fields based on other field values using events

### Related Topics

- [Form submit event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)
- [Input event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)
- [Change event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/change_event)
- [Focus and blur events](https://developer.mozilla.org/en-US/docs/Web/API/Element/focus_event)

## Form Validation API

### What It Is

The Constraint Validation API is a set of properties and methods on form elements that enable declarative and programmatic form validation. It includes HTML attributes for declaring constraints (`required`, `minlength`, `maxlength`, `pattern`, `min`, `max`, `type`) and JavaScript APIs for checking validity (`checkValidity()`, `reportValidity()`, `setCustomValidity()`).

```javascript
const input = document.querySelector('#email');
input.required = true;
input.type = 'email';
input.minLength = 5;
input.pattern = '[^@]+@[^@]+\\.[^@]+';

if (!input.checkValidity()) {
  console.log(input.validationMessage);
  console.log(input.validity);
}
```

### Why It Is Important

The Constraint Validation API provides browser-native validation that works consistently across platforms. It reduces the amount of custom validation code needed, provides accessible error messages (read by screen readers), and integrates with form submission to prevent invalid data from being sent to the server.

### How It Works Internally

The browser tracks validity state through a `ValidityState` object on each form element. This object has boolean properties for each validation constraint (`valueMissing`, `typeMismatch`, `patternMismatch`, `tooLong`, `tooShort`, `rangeUnderflow`, `rangeOverflow`, `stepMismatch`, `badInput`, `customError`, `valid`). When a form is submitted or `checkValidity()` is called, the browser evaluates all constraints and updates the `ValidityState`.

```javascript
// ValidityState properties
input.validity.valueMissing;   // Required field is empty
input.validity.typeMismatch;   // Wrong type (e.g., email without @)
input.validity.patternMismatch; // Doesn't match pattern attribute
input.validity.tooLong;        // Exceeds maxlength
input.validity.tooShort;       // Below minlength
input.validity.rangeUnderflow; // Below min
input.validity.rangeOverflow;  // Above max
input.validity.stepMismatch;   // Doesn't match step
input.validity.badInput;       // Browser can't parse the value
input.validity.customError;    // Custom error set via setCustomValidity()
input.validity.valid;          // true if all constraints pass
```

### Syntax

```javascript
// HTML validation attributes
// <input required minlength="3" maxlength="20" pattern="[A-Za-z]+" type="email">

// JavaScript validation API
element.checkValidity();       // Returns boolean, fires invalid event if false
element.reportValidity();      // Like checkValidity() but shows browser validation UI
element.setCustomValidity(msg);// Set custom validation message (empty string = valid)
element.validationMessage;     // Current validation message string
element.validity;              // ValidityState object
element.willValidate;          // Whether the element will be validated

// Form level validation
form.checkValidity();          // Validates all elements, returns boolean
form.reportValidity();         // Validates all and shows browser UI for first invalid
form.novalidate;               // HTML attribute to disable validation
```

### Beginner Examples

```javascript
// Example 1: Using HTML validation with custom styling
const form = document.querySelector('#register-form');

form.addEventListener('submit', (event) => {
  if (!form.checkValidity()) {
    event.preventDefault();
    showErrors();
  }
});

function showErrors() {
  const fields = form.querySelectorAll('input:invalid');
  fields.forEach(field => {
    field.classList.add('error');
    const errorEl = field.parentElement.querySelector('.error-msg') || createErrorMsg(field);
    errorEl.textContent = field.validationMessage;
  });
}

function createErrorMsg(field) {
  const el = document.createElement('span');
  el.className = 'error-msg';
  field.parentElement.appendChild(el);
  return el;
}

// Example 2: Real-time validation with input event
const emailInput = document.querySelector('#email');
emailInput.addEventListener('input', () => {
  emailInput.classList.remove('valid', 'invalid');

  if (emailInput.validity.valid) {
    emailInput.classList.add('valid');
  } else if (emailInput.value.length > 0) {
    emailInput.classList.add('invalid');
    showTooltip(emailInput, emailInput.validationMessage);
  }
});

// Example 3: Custom validation message
const password = document.querySelector('#password');
password.addEventListener('input', () => {
  if (password.value.length < 8) {
    password.setCustomValidity('Password must be at least 8 characters');
  } else if (!/[A-Z]/.test(password.value)) {
    password.setCustomValidity('Password must include an uppercase letter');
  } else if (!/[0-9]/.test(password.value)) {
    password.setCustomValidity('Password must include a number');
  } else {
    password.setCustomValidity(''); // Mark as valid
  }
});
```

### Intermediate Examples

```javascript
// Example 1: Validation summary component
class ValidationSummary {
  constructor(form) {
    this.form = form;
    this.summary = this.createSummary();
    this.setup();
  }

  createSummary() {
    const div = document.createElement('div');
    div.className = 'validation-summary';
    div.style.cssText = 'display:none; background:#fee; border:1px solid #fcc; padding:1em; margin:1em 0;';
    this.form.prepend(div);
    return div;
  }

  setup() {
    this.form.addEventListener('submit', (event) => {
      if (!this.form.checkValidity()) {
        event.preventDefault();
        this.showSummary();
      }
    });

    // Clear summary when user starts correcting
    this.form.addEventListener('input', () => {
      this.hideSummary();
    });
  }

  showSummary() {
    const errors = [];
    const fields = this.form.querySelectorAll('input, select, textarea');

    fields.forEach(field => {
      if (!field.validity.valid) {
        const label = this.form.querySelector(`[for="${field.id}"]`)?.textContent || field.name || field.id;
        errors.push({ field: label, message: field.validationMessage });
      }
    });

    if (errors.length > 0) {
      this.summary.innerHTML = `
        <strong>Please fix the following errors (${errors.length}):</strong>
        <ul>
          ${errors.map(e => `<li><a href="#${e.field}">${e.field}</a>: ${e.message}</li>`).join('')}
        </ul>
      `;
      this.summary.style.display = 'block';
      this.summary.scrollIntoView({ behavior: 'smooth' });
    }
  }

  hideSummary() {
    this.summary.style.display = 'none';
  }
}

// Example 2: Async validation
class AsyncValidator {
  constructor(form) {
    this.form = form;
    this.setup();
  }

  setup() {
    this.form.addEventListener('submit', async (event) => {
      event.preventDefault();

      // Basic validation first
      if (!this.form.checkValidity()) {
        this.form.reportValidity();
        return;
      }

      // Async validation
      const valid = await this.asyncValidate();
      if (valid) {
        this.form.submit(); // Now submit for real
      }
    });
  }

  async asyncValidate() {
    const fields = this.form.querySelectorAll('[data-async-validate]');
    const promises = [];

    fields.forEach(field => {
      const validator = field.dataset.asyncValidate;
      promises.push(
        this.runValidator(validator, field)
          .then(isValid => {
            if (!isValid) {
              field.setCustomValidity('Async validation failed');
              field.classList.add('invalid');
            }
          })
      );
    });

    const results = await Promise.all(promises);
    return results.every(Boolean);
  }

  async runValidator(name, field) {
    switch (name) {
      case 'username':
        return this.checkUsername(field.value);
      case 'email':
        return this.checkEmail(field.value);
      default:
        return true;
    }
  }

  async checkUsername(username) {
    const res = await fetch(`/api/check-username?q=${encodeURIComponent(username)}`);
    const data = await res.json();
    return data.available;
  }

  async checkEmail(email) {
    const res = await fetch(`/api/check-email?q=${encodeURIComponent(email)}`);
    const data = await res.json();
    return data.available;
  }
}

// Example 3: Progressive enhancement of validation
function enhanceValidation(form) {
  // Add real-time validation to all fields
  form.querySelectorAll('input, select, textarea').forEach(field => {
    // Show validation on blur
    field.addEventListener('blur', () => {
      if (field.value) {
        const isValid = field.checkValidity();
        field.classList.toggle('invalid', !isValid);
        field.classList.toggle('valid', isValid);
      }
    });

    // Clear invalid state on input
    field.addEventListener('input', () => {
      if (field.classList.contains('invalid')) {
        if (field.validity.valid) {
          field.classList.remove('invalid');
          field.classList.add('valid');
        }
      }
    });

    // Show/hide password toggle
    if (field.type === 'password') {
      const toggle = document.createElement('button');
      toggle.type = 'button';
      toggle.className = 'password-toggle';
      toggle.textContent = 'Show';
      toggle.addEventListener('click', () => {
        field.type = field.type === 'password' ? 'text' : 'password';
        toggle.textContent = field.type === 'password' ? 'Show' : 'Hide';
      });
      field.parentElement.appendChild(toggle);
    }
  });
}
```

### Advanced Examples

```javascript
// Example 1: Custom validation rules engine
class ValidationEngine {
  constructor(form) {
    this.form = form;
    this.rules = new Map();
    this.defaultMessages = {
      required: 'This field is required',
      email: 'Please enter a valid email address',
      min: 'Value must be at least {min}',
      max: 'Value must be at most {max}',
      minlength: 'Must be at least {minlength} characters',
      maxlength: 'Must be at most {maxlength} characters',
      pattern: 'Please match the requested format',
      match: 'Must match {field}'
    };
  }

  addRule(name, validator, message) {
    this.rules.set(name, { validator, message });
  }

  setup() {
    this.form.addEventListener('submit', (event) => {
      if (!this.validate()) {
        event.preventDefault();
      }
    });

    this.form.addEventListener('input', (event) => {
      this.validateField(event.target);
    });
  }

  validate() {
    const fields = this.form.querySelectorAll('[data-rules]');
    let isValid = true;

    fields.forEach(field => {
      if (!this.validateField(field)) {
        isValid = false;
      }
    });

    return isValid;
  }

  validateField(field) {
    const rulesStr = field.dataset.rules;
    if (!rulesStr) return true;

    const rules = rulesStr.split('|');
    let errorMessage = '';

    for (const ruleDef of rules) {
      const [ruleName, ...params] = ruleDef.split(':');
      const rule = this.rules.get(ruleName);

      if (rule && !rule.validator(field, params)) {
        errorMessage = rule.message || this.getDefaultMessage(ruleName, params);
        break;
      }
    }

    if (errorMessage) {
      field.setCustomValidity(errorMessage);
      field.classList.add('invalid');
      field.classList.remove('valid');
      this.showFieldError(field, errorMessage);
    } else {
      field.setCustomValidity('');
      field.classList.remove('invalid');
      field.classList.add('valid');
      this.hideFieldError(field);
    }

    return !errorMessage;
  }

  getDefaultMessage(ruleName, params) {
    let msg = this.defaultMessages[ruleName] || `Invalid`;
    params.forEach((param, i) => {
      msg = msg.replace(`{${i}}`, param).replace(`{${ruleName}}`, param);
    });
    return msg;
  }

  showFieldError(field, message) {
    let errorEl = field.parentElement.querySelector('.field-error');
    if (!errorEl) {
      errorEl = document.createElement('div');
      errorEl.className = 'field-error';
      field.parentElement.appendChild(errorEl);
    }
    errorEl.textContent = message;
  }

  hideFieldError(field) {
    const errorEl = field.parentElement.querySelector('.field-error');
    if (errorEl) errorEl.remove();
  }
}

// Usage
const engine = new ValidationEngine(document.querySelector('#my-form'));

engine.addRule('required', (f) => f.value.trim() !== '');
engine.addRule('email', (f) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(f.value));
engine.addRule('minlength', (f, [len]) => f.value.length >= parseInt(len));
engine.addRule('maxlength', (f, [len]) => f.value.length <= parseInt(len));
engine.addRule('match', (f, [name]) => {
  const other = f.form.querySelector(`[name="${name}"]`);
  return f.value === other?.value;
});

// Example 2: File validation with the API
class FileValidator {
  constructor(input) {
    this.input = input;
    this.setup();
  }

  setup() {
    this.input.addEventListener('change', () => this.validate());
  }

  validate() {
    const files = this.input.files;
    if (!files || files.length === 0) return;

    const maxSize = parseInt(this.input.dataset.maxSize) || 5 * 1024 * 1024; // 5MB
    const allowedTypes = (this.input.accept || '').split(',').map(t => t.trim());

    for (const file of files) {
      if (file.size > maxSize) {
        this.input.setCustomValidity(`File "${file.name}" exceeds ${maxSize / 1024 / 1024}MB`);
        return;
      }

      if (allowedTypes.length > 0 && allowedTypes[0] !== '') {
        const fileExt = '.' + file.name.split('.').pop();
        const typeMatch = allowedTypes.some(type =>
          type === file.type || type === fileExt || type.endsWith('/*')
        );
        if (!typeMatch) {
          this.input.setCustomValidity(`File type "${file.type}" is not accepted`);
          return;
        }
      }
    }

    this.input.setCustomValidity('');
  }
}

// Example 3: Custom validity with complex rules
class CreditCardValidator {
  constructor(input) {
    this.input = input;
    this.setup();
  }

  setup() {
    this.input.addEventListener('input', () => {
      this.formatAndValidate();
    });
  }

  formatAndValidate() {
    let value = this.input.value.replace(/\D/g, '');

    // Format with spaces every 4 digits
    const formatted = value.replace(/(\d{4})(?=\d)/g, '$1 ');
    this.input.value = formatted;

    // Detect card type from first digits
    const cardType = this.detectCardType(value);
    const typeIndicator = this.input.parentElement.querySelector('.card-type');
    if (typeIndicator) {
      typeIndicator.textContent = cardType ? cardType.name : '';
    }

    // Validate
    if (value.length > 0) {
      if (!this.isValidLength(value, cardType)) {
        this.input.setCustomValidity(`Invalid length for ${cardType?.name || 'card'}`);
      } else if (!this.luhnCheck(value)) {
        this.input.setCustomValidity('Invalid card number');
      } else {
        this.input.setCustomValidity('');
      }
    } else {
      this.input.setCustomValidity('');
    }
  }

  detectCardType(number) {
    const patterns = [
      { name: 'Visa', pattern: /^4/ },
      { name: 'Mastercard', pattern: /^5[1-5]/ },
      { name: 'Amex', pattern: /^3[47]/ },
      { name: 'Discover', pattern: /^6(?:011|5)/ }
    ];
    return patterns.find(p => p.pattern.test(number)) || null;
  }

  isValidLength(number, type) {
    if (!type) return number.length >= 13 && number.length <= 19;
    const lengths = {
      'Visa': [13, 16, 19],
      'Mastercard': [16],
      'Amex': [15],
      'Discover': [16, 19]
    };
    return lengths[type.name]?.includes(number.length) || false;
  }

  luhnCheck(number) {
    let sum = 0;
    let alternate = false;
    for (let i = number.length - 1; i >= 0; i--) {
      let digit = parseInt(number[i], 10);
      if (alternate) {
        digit *= 2;
        if (digit > 9) digit -= 9;
      }
      sum += digit;
      alternate = !alternate;
    }
    return sum % 10 === 0;
  }
}
```

### Real-World Use Cases

- **Registration Forms**: Password strength, email format, username availability
- **Payment Forms**: Credit card number (Luhn check), CVV, expiry date
- **Checkout**: ZIP code format, phone format, address validation
- **File Uploads**: File type, size, and dimension validation
- **Date Pickers**: Date range validation, past/future date constraints
- **Multi-Step Wizards**: Step-by-step validation with conditional rules
- **Admin Panels**: Complex business rule validation on form fields

### Common Mistakes

```javascript
// Mistake 1: Not calling setCustomValidity('') to clear errors
input.setCustomValidity('Error message');
// ... later, when valid:
input.setCustomValidity(''); // Must clear explicitly

// Mistake 2: Relying only on HTML validation attributes
// HTML validates on submit only, not on input
// Always add JavaScript validation for real-time feedback

// Mistake 3: Not handling novalidate for custom validation
form.noValidate = true; // Disable browser validation
// Must manually validate in JavaScript

// Mistake 4: Forgetting to revalidate after dynamic changes
field.disabled = true; // Disabled fields are not validated
field.readOnly = true; // Read-only fields are validated
```

### Best Practices

- Use HTML validation attributes as a baseline, enhance with JavaScript
- Call `setCustomValidity('')` to clear custom errors
- Use `checkValidity()` for silent validation, `reportValidity()` for visible
- Provide accessible error messages (associate with `aria-describedby`)
- Validate on blur for individual fields, on submit for the whole form
- Clear validation state when the user starts correcting
- Disable HTML validation with `noValidate` when using custom validation

### Performance Considerations

The Constraint Validation API is browser-native and highly optimized. Calling `checkValidity()` on individual fields is O(1). Calling it on a form validates all fields, which is O(n). For very large forms, validate fields individually on blur rather than all at once.

### Interview Questions

1. What is the Constraint Validation API?
2. What properties does `ValidityState` have?
3. How do you set a custom validation message?
4. What is the difference between `checkValidity` and `reportValidity`?
5. How does `setCustomValidity` work?
6. What is the `willValidate` property?
7. How do you disable HTML validation for a form?
8. What happens when validation fails during form submission?

### Coding Challenges

1. **Custom Validation Engine**: Build a validation engine with configurable rules
2. **Async Validator**: Implement async validation (username/email availability)
3. **Formatted Inputs**: Create validated inputs for phone, credit card, date
4. **Validation Summary**: Build a component that aggregates and displays all form errors
5. **Password Strength Meter**: Create a visual password strength indicator with validation rules

### Related Topics

- [Constraint Validation API](https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation)
- [ValidityState](https://developer.mozilla.org/en-US/docs/Web/API/ValidityState)
- [HTML Forms Guide](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Forms)
- [Accessible Forms](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/forms)

## FormData Object

### What It Is

The `FormData` interface provides a way to construct key-value pairs representing form fields and their values. It can be populated from an HTML form element or created empty and populated programmatically. `FormData` is primarily used with `fetch()` or `XMLHttpRequest` to send form data via AJAX, automatically handling the `multipart/form-data` encoding for file uploads.

```javascript
// From existing form
const formData = new FormData(document.querySelector('form'));

// Empty, populate manually
const data = new FormData();
data.append('username', 'john');
data.append('avatar', fileInput.files[0]);

// Send via fetch
fetch('/api/users', {
  method: 'POST',
  body: formData
});
```

### Why It Is Important

`FormData` simplifies AJAX form submission by automatically serializing all form fields, including file inputs. It handles the correct `Content-Type` header (`multipart/form-data` for files, `application/x-www-form-urlencoded` otherwise) and encoding, which would otherwise require significant manual work.

### How It Works Internally

When created from a form element, `FormData` iterates over all form controls and extracts their names and values using the same algorithm the browser uses for form submission. It respects disabled state, checkbox/radio selection, and multiple-select elements. Each entry is stored as a `[name, value]` pair, with support for multiple values under the same name.

```javascript
// Internal behavior (conceptual):
// For each form element:
// - If disabled: skip
// - If checkbox/radio and not checked: skip
// - If select-multiple: append each selected option
// - Otherwise: append name and value
```

### Syntax

```javascript
// Create FormData
const fd = new FormData(); // Empty
const fd = new FormData(formElement); // From form
const fd = new FormData(formElement, submitter); // From form with submit button

// Methods
fd.append(name, value);                  // Add a value (allows duplicates)
fd.append(name, blob, filename);         // Add a file with custom filename
fd.set(name, value);                     // Set a value (replaces all entries with this name)
fd.set(name, blob, filename);            // Set a file
fd.delete(name);                         // Remove all entries with this name
fd.get(name);                            // Get first value with this name
fd.getAll(name);                         // Get all values with this name (array)
fd.has(name);                            // Check if entry exists
fd.entries();                            // Iterator of [key, value] pairs
fd.keys();                               // Iterator of keys
fd.values();                             // Iterator of values
fd.forEach(callback);                    // Iterate over entries
```

### Beginner Examples

```javascript
// Example 1: Basic form submission with FormData
const form = document.querySelector('#profile-form');

form.addEventListener('submit', async (event) => {
  event.preventDefault();

  const formData = new FormData(form);

  try {
    const response = await fetch('/api/profile', {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      showToast('Profile updated');
    } else {
      const error = await response.json();
      showToast(error.message, 'error');
    }
  } catch (error) {
    showToast('Network error', 'error');
  }
});

// Example 2: Reading FormData
const data = new FormData();
data.append('name', 'John');
data.append('age', '30');

// Iterate over entries
for (const [key, value] of data.entries()) {
  console.log(`${key}: ${value}`);
}

// Get specific values
console.log(data.get('name')); // 'John'
console.log(data.get('age')); // '30'

// Example 3: Append multiple files
const fileInput = document.querySelector('#photos');
fileInput.addEventListener('change', () => {
  const formData = new FormData();

  for (const file of fileInput.files) {
    formData.append('photos[]', file);
  }

  // Preview
  for (const [name, file] of formData.entries()) {
    console.log(`Uploading ${file.name} (${file.size} bytes)`);
  }
});

// Example 4: Convert FormData to plain object
function formDataToObject(formData) {
  const object = {};
  formData.forEach((value, key) => {
    object[key] = value;
  });
  return object;
}

const obj = formDataToObject(new FormData(form));
console.log(obj); // { username: 'john', email: 'john@example.com' }
```

### Intermediate Examples

```javascript
// Example 1: Handling nested form data
function serializeNested(formData) {
  const result = {};

  for (const [key, value] of formData.entries()) {
    // Handle nested keys like user[name], user[email]
    const match = key.match(/^(\w+)\[(\w+)]$/);
    if (match) {
      const [, parent, child] = match;
      if (!result[parent]) result[parent] = {};
      result[parent][child] = value;
    } else {
      // Handle array keys like items[]
      const arrayMatch = key.match(/^(\w+)\[\]$/);
      if (arrayMatch) {
        const [, name] = arrayMatch;
        if (!result[name]) result[name] = [];
        result[name].push(value);
      } else {
        result[key] = value;
      }
    }
  }

  return result;
}

const formData = new FormData();
formData.append('user[name]', 'John');
formData.append('user[email]', 'john@example.com');
formData.append('items[]', 'item1');
formData.append('items[]', 'item2');

console.log(serializeNested(formData));
// { user: { name: 'John', email: 'john@example.com' }, items: ['item1', 'item2'] }

// Example 2: FormData diff (for dirty tracking)
class FormDataDiff {
  constructor(form) {
    this.form = form;
    this.initial = new FormData(form);
  }

  getChanges() {
    const current = new FormData(this.form);
    const changes = [];

    // Check for added or changed fields
    for (const [key, value] of current.entries()) {
      const initialValues = this.initial.getAll(key);
      if (!initialValues.includes(value)) {
        changes.push({ type: 'modified', key, oldValue: this.initial.get(key), newValue: value });
      }
    }

    // Check for removed fields
    for (const [key] of this.initial.entries()) {
      if (!current.has(key)) {
        changes.push({ type: 'removed', key, oldValue: this.initial.get(key) });
      }
    }

    return changes;
  }

  hasChanges() {
    return this.getChanges().length > 0;
  }

  reset() {
    this.initial = new FormData(this.form);
  }
}

// Example 3: Progressive file upload with FormData
async function uploadWithProgress(formData) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', (event) => {
      if (event.lengthComputable) {
        const percent = (event.loaded / event.total) * 100;
        updateProgressBar(percent);
      }
    });

    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });

    xhr.addEventListener('error', () => reject(new Error('Network error')));
    xhr.open('POST', '/api/upload');
    xhr.send(formData);
  });
}
```

### Advanced Examples

```javascript
// Example 1: FormData with JSON and file hybrid
class HybridFormData {
  constructor() {
    this.formData = new FormData();
  }

  append(key, value) {
    if (value instanceof File || value instanceof Blob) {
      this.formData.append(key, value);
    } else if (typeof value === 'object') {
      this.formData.append(key, new Blob([JSON.stringify(value)], { type: 'application/json' }));
    } else {
      this.formData.append(key, String(value));
    }
  }

  getFormData() {
    return this.formData;
  }
}

const hybrid = new HybridFormData();
hybrid.append('avatar', fileInput.files[0]);
hybrid.append('metadata', { width: 800, height: 600 });
hybrid.append('name', 'Profile Picture');

// Server receives multipart/form-data with:
// - avatar: binary file
// - metadata: JSON blob
// - name: string

// Example 2: FormData streaming (for large payloads)
class StreamingFormData {
  constructor(form) {
    this.form = form;
    this.chunks = [];
  }

  async *[Symbol.asyncIterator]() {
    const formData = new FormData(this.form);

    for (const [key, value] of formData.entries()) {
      if (value instanceof File) {
        // Stream file in chunks
        const stream = value.stream();
        const reader = stream.getReader();
        let chunkIndex = 0;

        while (true) {
          const { done, value: chunk } = await reader.read();
          if (done) break;
          this.chunks.push({
            type: 'file-chunk',
            key,
            fileName: value.name,
            chunkIndex: chunkIndex++,
            data: chunk,
            isLast: done
          });
        }
      } else {
        this.chunks.push({
          type: 'field',
          key,
          value
        });
      }
    }
  }

  getSize() {
    let total = 0;
    for (const chunk of this.chunks) {
      if (chunk.type === 'file-chunk' && chunk.data) {
        total += chunk.data.byteLength;
      } else if (chunk.type === 'field') {
        total += new TextEncoder().encode(chunk.value).length;
      }
    }
    return total;
  }
}

// Example 3: FormData promise-based wrapper
class FormDataPromise {
  constructor(form) {
    this.form = form;
  }

  async submit(url, options = {}) {
    const formData = new FormData(this.form);
    const method = options.method || this.form.method || 'POST';

    // Optional: compress large payloads
    if (options.compress && this.shouldCompress(formData)) {
      return this.submitCompressed(url, formData, method);
    }

    // Read file contents for progress tracking
    const totalSize = this.getTotalSize(formData);

    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();

      if (options.onProgress && totalSize > 0) {
        xhr.upload.addEventListener('progress', (event) => {
          if (event.lengthComputable) {
            options.onProgress(event.loaded / event.total);
          }
        });
      }

      xhr.addEventListener('load', () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve(JSON.parse(xhr.responseText));
        } else {
          reject(new Error(`HTTP ${xhr.status}: ${xhr.responseText}`));
        }
      });

      xhr.addEventListener('error', () => reject(new Error('Network error')));
      xhr.open(method, url);
      xhr.send(formData);
    });
  }

  shouldCompress(formData) {
    let size = 0;
    for (const [, value] of formData.entries()) {
      if (value instanceof File) {
        size += value.size;
      } else {
        size += value.length;
      }
    }
    return size > 10 * 1024 * 1024; // > 10MB
  }

  getTotalSize(formData) {
    let total = 0;
    for (const [, value] of formData.entries()) {
      if (value instanceof File) {
        total += value.size;
      } else {
        total += new Blob([value]).size;
      }
    }
    return total;
  }

  async submitCompressed(url, formData, method) {
    // Compress text fields
    const compressed = new FormData();
    for (const [key, value] of formData.entries()) {
      if (value instanceof File) {
        compressed.append(key, value);
      } else {
        const blob = new Blob([value]);
        const stream = blob.stream().pipeThrough(new CompressionStream('gzip'));
        compressed.append(key, stream);
      }
    }

    const response = await fetch(url, {
      method,
      body: compressed,
      headers: {
        'X-Compressed': 'true'
      }
    });

    return response.json();
  }
}

// Example 4: FormData field interaction observer
function observeFormData(form, callback) {
  const handler = {
    get(target, prop) {
      if (prop === 'append' || prop === 'set' || prop === 'delete') {
        return function (...args) {
          const result = FormData.prototype[prop].apply(target, args);
          callback({
            action: prop,
            args,
            formData: target,
            timestamp: Date.now()
          });
          return result;
        };
      }
      return Reflect.get(target, prop);
    }
  };

  return new Proxy(new FormData(form), handler);
}

const observed = observeFormData(form, (change) => {
  if (change.action === 'append') {
    console.log(`Field added: ${change.args[0]} = ${change.args[1]}`);
  }
});
```

### Real-World Use Cases

- **AJAX Form Submission**: Submit forms without page reload, including file uploads
- **API Clients**: Send multipart/form-data to REST APIs
- **File Uploads**: Upload single or multiple files with metadata
- **Form Synchronization**: Sync form state with server via periodic FormData snapshots
- **Dynamic Forms**: Build form data from dynamically generated fields
- **Progressive Enhancement**: Enhance standard HTML forms with AJAX submission
- **Batch Processing**: Collect data from multiple forms into a single request

### Common Mistakes

```javascript
// Mistake 1: Forgetting to set Content-Type header with fetch
fetch('/api/data', {
  method: 'POST',
  body: formData,
  headers: { 'Content-Type': 'application/json' } // Wrong! Remove this
});
// fetch automatically sets multipart/form-data with boundary when body is FormData

// Mistake 2: Trying to JSON.stringify FormData
const data = new FormData(form);
fetch('/api/data', {
  method: 'POST',
  body: JSON.stringify(data) // [object FormData]
});
// Use Object.fromEntries or formDataToObject first

// Mistake 3: Not handling multiple values correctly
select.multiple = true;
// FormData handles multiple selected options automatically
// But data.get('colors') only returns the first selected value
// Use data.getAll('colors') to get all

// Mistake 4: Modifying input values after FormData creation
const fd = new FormData(form);
input.value = 'changed'; // FormData already captured the old value
// Create FormData immediately before sending
```

### Best Practices

- Create `FormData` immediately before sending, not at setup time
- Use `getAll()` for fields that may have multiple values
- Use `set()` to replace all values for a field name (vs `append()` to add)
- Don't set `Content-Type` header manually when sending FormData with fetch
- Use `formData.entries()` or `forEach()` for iteration
- Handle file inputs by checking `value instanceof File` or `value instanceof Blob`
- Use `files` property for `<input type="file">` to access File objects

### Performance Considerations

`FormData` creation from a form is O(n) where n is the number of form elements. For very large forms, avoid recreating `FormData` on every keystroke. When uploading large files, `FormData` streams the file content without loading the entire file into memory.

### Interview Questions

1. What is `FormData` and when would you use it?
2. How does `FormData` handle file inputs?
3. What is the difference between `append` and `set` methods?
4. How do you convert `FormData` to a plain object?
5. Why shouldn't you set `Content-Type` header when sending FormData with fetch?
6. How does `FormData` handle disabled fields?
7. What is the difference between `get` and `getAll`?
8. How does `FormData` handle `select multiple` elements?

### Coding Challenges

1. **FormData Diff**: Create a utility that detects changes between two FormData snapshots
2. **Nested Serializer**: Convert FormData to nested objects (user[name] -> { user: { name } })
3. **File Upload with Progress**: Build a file upload component with FormData and XHR progress
4. **FormData Replay**: Record FormData submissions and replay them

### Related Topics

- [FormData (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [Using FormData Objects](https://developer.mozilla.org/en-US/docs/Web/API/FormData/Using_FormData_Objects)
- [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

## Controlled Inputs

### What It Is

Controlled inputs are a pattern where the form field's value is managed by JavaScript state rather than the DOM. Instead of reading the value from the DOM when needed, the value is stored in JavaScript and explicitly set on the input element. Every change to the input updates the JavaScript state, which then updates the input's displayed value. This pattern is central to React, but can be implemented in plain JavaScript.

```javascript
// Controlled input pattern in plain JS
let state = { name: '' };

const input = document.querySelector('#name');
input.value = state.name;

input.addEventListener('input', () => {
  state.name = input.value;
  // Optional: transform or validate
  if (state.name.length > 50) {
    state.name = state.name.slice(0, 50);
  }
  input.value = state.name;
});
```

### Why It Is Important

Controlled inputs give JavaScript complete control over form field values. This enables value transformation, conditional formatting, real-time validation, and integration with state management systems. The single source of truth prevents synchronization issues between DOM state and JavaScript state.

### How It Works Internally

In a controlled input, the flow is:
1. User types in input
2. `input` event fires
3. JavaScript handler reads the event
4. JavaScript updates state
5. JavaScript sets `input.value` to the (possibly transformed) state value
6. DOM reflects the new value

This creates a unidirectional data flow: User Action -> Event -> State -> DOM Update.

```javascript
// Uncontrolled: DOM is source of truth
// User types -> DOM updates -> Read from DOM when needed

// Controlled: JavaScript is source of truth
// User types -> Event -> Update JS state -> Set DOM value
```

### Syntax

```javascript
// Basic controlled input
const state = { value: '' };

function updateState(newValue) {
  state.value = newValue;
  input.value = state.value; // Sync DOM to state
}

input.addEventListener('input', (event) => {
  updateState(event.target.value);
});

// Initial value
input.value = state.value;
```

### Beginner Examples

```javascript
// Example 1: Uppercase input
const nameInput = document.querySelector('#name');
nameInput.value = '';

nameInput.addEventListener('input', () => {
  // Force uppercase
  nameInput.value = nameInput.value.toUpperCase();
});

// Example 2: Character-limited input
const bioInput = document.querySelector('#bio');
const counter = document.querySelector('#bio-counter');
const MAX_CHARS = 160;

bioInput.value = '';

bioInput.addEventListener('input', () => {
  let value = bioInput.value;

  // Truncate to max length
  if (value.length > MAX_CHARS) {
    value = value.slice(0, MAX_CHARS);
  }

  // Remove newlines
  value = value.replace(/\n/g, ' ');

  bioInput.value = value;
  counter.textContent = `${value.length}/${MAX_CHARS}`;
});

// Example 3: Number-only input
const ageInput = document.querySelector('#age');
ageInput.value = '';

ageInput.addEventListener('input', () => {
  // Remove non-digits
  ageInput.value = ageInput.value.replace(/\D/g, '');

  // Clamp to range
  let value = parseInt(ageInput.value, 10);
  if (isNaN(value)) value = '';
  if (value > 150) value = 150;
  ageInput.value = value;
});

// Example 4: Currency input
const priceInput = document.querySelector('#price');
priceInput.value = '';

priceInput.addEventListener('input', () => {
  let value = priceInput.value;

  // Remove non-numeric except decimal point
  value = value.replace(/[^\d.]/g, '');

  // Ensure only one decimal point
  const parts = value.split('.');
  if (parts.length > 2) {
    value = parts[0] + '.' + parts.slice(1).join('');
  }

  // Limit decimal places to 2
  if (parts.length === 2 && parts[1].length > 2) {
    value = parts[0] + '.' + parts[1].slice(0, 2);
  }

  priceInput.value = value;
});
```

### Intermediate Examples

```javascript
// Example 1: Controlled input state manager
class ControlledInput {
  constructor(element, options = {}) {
    this.element = element;
    this.options = options;
    this._value = element.value || options.defaultValue || '';
    this.history = [];
    this.historyIndex = -1;
    this.setup();
  }

  get value() {
    return this._value;
  }

  set value(newValue) {
    if (this._value !== newValue) {
      const oldValue = this._value;
      this._value = newValue;
      this.element.value = newValue;
      this.options.onChange?.(newValue, oldValue);
    }
  }

  setup() {
    this.element.addEventListener('input', (event) => {
      let rawValue = event.target.value;

      // Apply transformers
      if (this.options.transform) {
        rawValue = this.options.transform(rawValue);
      }

      // Apply validators
      if (this.options.validate && !this.options.validate(rawValue)) {
        event.preventDefault();
        this.element.value = this._value; // Revert
        return;
      }

      this.pushHistory(this._value);
      this.value = rawValue;
    });

    // Support undo
    if (this.options.undoable) {
      this.element.addEventListener('keydown', (event) => {
        if ((event.ctrlKey || event.metaKey) && event.key === 'z') {
          event.preventDefault();
          this.undo();
        }
      });
    }
  }

  pushHistory(value) {
    this.history.push(value);
    if (this.history.length > 50) this.history.shift();
    this.historyIndex = this.history.length - 1;
  }

  undo() {
    if (this.historyIndex > 0) {
      this.historyIndex--;
      this.value = this.history[this.historyIndex];
    }
  }

  redo() {
    if (this.historyIndex < this.history.length - 1) {
      this.historyIndex++;
      this.value = this.history[this.historyIndex];
    }
  }

  reset() {
    this.value = this.options.defaultValue || '';
    this.history = [];
    this.historyIndex = -1;
  }
}

const nameControl = new ControlledInput(document.querySelector('#name'), {
  defaultValue: '',
  transform: (value) => value.replace(/[^a-zA-Z\s]/g, ''),
  onChange: (value) => console.log('Name changed:', value),
  undoable: true
});

// Example 2: Controlled form state management
class ControlledForm {
  constructor(form) {
    this.form = form;
    this.state = {};
    this.setup();
  }

  setup() {
    const fields = this.form.querySelectorAll('input, select, textarea');

    fields.forEach(field => {
      const name = field.name || field.id;
      if (!name) return;

      // Initialize state
      this.state[name] = field.value;

      // Add change listener
      field.addEventListener('input', () => {
        this.state[name] = field.value;
        this.onStateChange(name, field.value);
      });

      field.addEventListener('change', () => {
        if (field.type === 'checkbox') {
          this.state[name] = field.checked;
        } else if (field.type === 'radio') {
          if (field.checked) {
            this.state[name] = field.value;
          }
        }
      });
    });
  }

  onStateChange(name, value) {
    console.log(`State update: ${name} = ${value}`);
    this.updateDerivedFields(name, value);
  }

  updateDerivedFields(changedField, value) {
    // Example: if country changes, update available regions
    if (changedField === 'country') {
      this.setField('region', '');
      this.setFieldOptions('region', this.getRegionsForCountry(value));
    }

    // Example: if use-shipping is checked, copy address
    if (changedField === 'use_shipping') {
      if (value) {
        this.setField('shipping_address', this.state.billing_address);
      }
    }
  }

  setField(name, value) {
    const field = this.form.querySelector(`[name="${name}"]`);
    if (field) {
      this.state[name] = value;
      field.value = value;
      field.dispatchEvent(new Event('input', { bubbles: true }));
    }
  }

  setFieldOptions(name, options) {
    const select = this.form.querySelector(`[name="${name}"]`);
    if (select) {
      select.innerHTML = options.map(opt =>
        `<option value="${opt.value}">${opt.label}</option>`
      ).join('');
    }
  }

  getState() {
    return { ...this.state };
  }

  reset() {
    this.form.reset();
    this.state = {};
    this.setup(); // Re-initialize
  }
}

// Example 3: Controlled input with suggestions
class ControlledAutocomplete {
  constructor(input, options = {}) {
    this.input = input;
    this.options = options;
    this._value = input.value;
    this.suggestions = [];
    this.selectedIndex = -1;
    this.setup();
  }

  get value() {
    return this._value;
  }

  set value(newValue) {
    this._value = newValue;
    this.input.value = newValue;
  }

  setup() {
    this.input.addEventListener('input', async () => {
      this.value = this.input.value;

      if (this.value.length >= (this.options.minLength || 2)) {
        const results = await this.options.fetch(this.value);
        this.showSuggestions(results);
      } else {
        this.hideSuggestions();
      }
    });

    this.input.addEventListener('keydown', (event) => {
      if (event.key === 'ArrowDown') {
        event.preventDefault();
        this.selectedIndex = Math.min(this.selectedIndex + 1, this.suggestions.length - 1);
        this.highlightSuggestion();
      } else if (event.key === 'ArrowUp') {
        event.preventDefault();
        this.selectedIndex = Math.max(this.selectedIndex - 1, -1);
        this.highlightSuggestion();
      } else if (event.key === 'Escape') {
        this.hideSuggestions();
      } else if (event.key === 'Enter' && this.selectedIndex >= 0) {
        event.preventDefault();
        this.selectSuggestion(this.suggestions[this.selectedIndex]);
      }
    });

    this.input.addEventListener('blur', () => {
      setTimeout(() => this.hideSuggestions(), 200);
    });
  }

  showSuggestions(results) {
    this.suggestions = results;
    this.selectedIndex = -1;
    // Render dropdown
    this.options.onShow?.(results);
  }

  hideSuggestions() {
    this.suggestions = [];
    this.selectedIndex = -1;
    this.options.onHide?.();
  }

  highlightSuggestion() {
    this.options.onHighlight?.(this.selectedIndex);
  }

  selectSuggestion(suggestion) {
    this.value = suggestion[this.options.valueKey || 'value'];
    this.hideSuggestions();
    this.options.onSelect?.(suggestion);
  }
}
```

### Advanced Examples

```javascript
// Example 1: Controlled input with reactivity system
class ReactiveForm {
  constructor(form) {
    this.form = form;
    this.state = new Proxy({}, {
      set: (target, key, value) => {
        target[key] = value;
        this.syncField(key, value);
        this.notify(key, value);
        return true;
      },
      get: (target, key) => target[key]
    });

    this.watchers = new Map();
    this.setup();
  }

  setup() {
    this.form.querySelectorAll('[name]').forEach(field => {
      const name = field.name;
      this.state[name] = field.value;

      field.addEventListener('input', () => {
        let value = field.value;
        if (field.type === 'checkbox') value = field.checked;
        this.state[name] = value;
      });

      field.addEventListener('change', () => {
        if (field.type === 'radio' && field.checked) {
          this.state[name] = field.value;
        }
      });
    });
  }

  syncField(name, value) {
    const field = this.form.querySelector(`[name="${name}"]`);
    if (!field) return;

    if (field.type === 'checkbox') {
      field.checked = value;
    } else {
      field.value = value;
    }
  }

  watch(expression, handler) {
    this.watchers.set(expression, handler);
  }

  notify(key, value) {
    for (const [expr, handler] of this.watchers) {
      if (expr.includes(key)) {
        handler(this.state);
      }
    }
  }

  getState() {
    return { ...this.state };
  }

  setState(partial) {
    Object.assign(this.state, partial);
  }

  reset() {
    this.form.reset();
    this.form.querySelectorAll('[name]').forEach(field => {
      this.state[field.name] = field.value;
    });
  }
}

const form = new ReactiveForm(document.querySelector('#registration-form'));

form.watch('password', (state) => {
  // Password strength indicator updates reactively
  updatePasswordStrength(state.password);
});

form.watch('country', (state) => {
  // Country changes trigger region options update
  loadRegions(state.country);
});

// Example 2: Input masking with controlled inputs
class InputMask {
  constructor(input, mask) {
    this.input = input;
    this.mask = mask;
    this._value = '';
    this.setup();
  }

  setup() {
    this.input.addEventListener('input', () => {
      const raw = this.input.value.replace(/\D/g, '');
      const formatted = this.applyMask(raw);
      this.input.value = formatted;
      this._value = raw;
    });

    // Initial value
    this.input.value = this.applyMask(this.input.value.replace(/\D/g, ''));
  }

  applyMask(raw) {
    let result = '';
    let rawIndex = 0;

    for (const char of this.mask) {
      if (rawIndex >= raw.length) break;

      if (char === '9') {
        result += raw[rawIndex++];
      } else if (char === 'A') {
        const charCode = raw.charCodeAt(rawIndex);
        if (charCode >= 65 && charCode <= 90 || charCode >= 97 && charCode <= 122) {
          result += raw[rawIndex++];
        } else {
          result += char;
          rawIndex++;
        }
      } else if (char === '*') {
        result += raw[rawIndex++];
      } else {
        result += char;
      }
    }

    return result;
  }

  getRawValue() {
    return this._value;
  }

  setValue(raw) {
    this._value = raw;
    this.input.value = this.applyMask(raw);
  }
}

const phoneMask = new InputMask(
  document.querySelector('#phone'),
  '(999) 999-9999'
);

const ssnMask = new InputMask(
  document.querySelector('#ssn'),
  '999-99-9999'
);

// Example 3: Debounced controlled input with validation pipeline
class PipelinesControlledInput {
  constructor(input, pipelines = []) {
    this.input = input;
    this.pipelines = pipelines;
    this._value = input.value;
    this.timer = null;
    this.setup();
  }

  setup() {
    this.input.addEventListener('input', (event) => {
      clearTimeout(this.timer);

      const pipeline = [...this.pipelines];
      let currentValue = event.target.value;

      // Run synchronous transformations immediately
      for (const step of pipeline) {
        if (!step.async) {
          currentValue = step.transform(currentValue, this._value);
          if (currentValue === false) {
            this.input.value = this._value;
            return;
          }
        }
      }

      // Run async steps with debounce
      const asyncSteps = pipeline.filter(s => s.async);
      if (asyncSteps.length > 0) {
        this.timer = setTimeout(async () => {
          let result = currentValue;
          for (const step of asyncSteps) {
            result = await step.transform(result, this._value);
            if (result === false) {
              this.input.value = this._value;
              return;
            }
          }
          this.commitValue(result);
        }, 300);
      }

      this.commitValue(currentValue);
    });
  }

  commitValue(value) {
    this._value = value;
    this.input.value = value;
  }

  get value() {
    return this._value;
  }

  set value(val) {
    this._value = val;
    this.input.value = val;
  }
}

// Usage
const emailInput = new PipelinesControlledInput(
  document.querySelector('#email'),
  [
    { transform: (v) => v.replace(/\s/g, '') }, // No spaces
    { transform: (v) => v.toLowerCase() },       // Lowercase
    { transform: async (v) => {                   // Check availability
      const res = await fetch(`/api/check-email?q=${v}`);
      const data = await res.json();
      return data.available ? v : false;
    }, async: true }
  ]
);

// Example 4: Controlled input with undo/redo and cursor preservation
class ControlledInputWithHistory {
  constructor(input) {
    this.input = input;
    this.history = [];
    this.historyIndex = -1;
    this.maxHistory = 100;
    this.setup();
  }

  setup() {
    this.input.addEventListener('input', () => {
      this.pushState();
    });

    this.input.addEventListener('keydown', (event) => {
      if ((event.ctrlKey || event.metaKey) && event.key === 'z') {
        event.preventDefault();
        if (event.shiftKey) {
          this.redo();
        } else {
          this.undo();
        }
      }
    });
  }

  pushState() {
    // Remove any future states beyond current index
    this.history = this.history.slice(0, this.historyIndex + 1);

    this.history.push({
      value: this.input.value,
      selectionStart: this.input.selectionStart,
      selectionEnd: this.input.selectionEnd
    });

    if (this.history.length > this.maxHistory) {
      this.history.shift();
    }

    this.historyIndex = this.history.length - 1;
  }

  undo() {
    if (this.historyIndex > 0) {
      this.historyIndex--;
      this.restoreState(this.history[this.historyIndex]);
    }
  }

  redo() {
    if (this.historyIndex < this.history.length - 1) {
      this.historyIndex++;
      this.restoreState(this.history[this.historyIndex]);
    }
  }

  restoreState(state) {
    const isFocused = document.activeElement === this.input;
    this.input.value = state.value;

    // Restore cursor position if input is focused
    if (isFocused) {
      this.input.setSelectionRange(state.selectionStart, state.selectionEnd);
    }
  }
}
```

### Real-World Use Cases

- **Input Formatting**: Phone, credit card, SSN, currency formatting as user types
- **Real-time Validation**: Field validation with immediate visual feedback
- **Character Limits**: Enforcing max lengths with smooth truncation
- **Input Transformation**: Auto-capitalization, character filtering, whitespace handling
- **Form State Management**: Centralized form state for complex multi-field forms
- **Undo/Redo**: User history for form inputs
- **Autocomplete**: Controlled suggestions with keyboard navigation
- **Reactive Forms**: Framework-like form reactivity in plain JavaScript

### Common Mistakes

```javascript
// Mistake 1: Creating infinite update loops
input.addEventListener('input', () => {
  input.value = input.value.toUpperCase(); // Triggers another input event!
});
// Fix: Check if value actually changed before setting
input.addEventListener('input', () => {
  const upper = input.value.toUpperCase();
  if (input.value !== upper) {
    input.value = upper;
  }
});

// Mistake 2: Not preserving cursor position
input.addEventListener('input', () => {
  const cursor = input.selectionStart;
  input.value = transform(input.value);
  input.setSelectionRange(cursor, cursor); // Restore cursor
});

// Mistake 3: Modifying value during composition (IME input)
input.addEventListener('input', (event) => {
  if (event.isComposing) return; // Skip during IME composition
  input.value = input.value.toUpperCase();
});

// Mistake 4: Using input event for programmatic changes
input.value = 'new value';
// Doesn't fire input event automatically
// Must dispatch: input.dispatchEvent(new Event('input', { bubbles: true }))
```

### Best Practices

- Preserve cursor position when transforming input values
- Check `event.isComposing` to avoid interfering with IME input
- Avoid causing infinite update loops by checking if value actually changed
- Use `input` event for controlled inputs (not `keyup` or `change`)
- Keep transformation logic pure and predictable
- Debounce expensive operations (API calls, complex validation)
- Consider using `inputType` to distinguish between different types of input changes

### Performance Considerations

Controlled inputs add overhead per keystroke (event handler + state update + DOM write). For simple inputs, this overhead is negligible. For complex transformations or large forms, debounce or use `requestAnimationFrame` to batch updates. Avoid heavy computation in synchronous input handlers.

```javascript
// Batch DOM writes with requestAnimationFrame
let pendingUpdate = false;
input.addEventListener('input', () => {
  if (pendingUpdate) return;
  pendingUpdate = true;

  requestAnimationFrame(() => {
    input.value = transform(input.value);
    pendingUpdate = false;
  });
});
```

### Interview Questions

1. What is a controlled input?
2. How is a controlled input different from an uncontrolled input?
3. Why would you use controlled inputs in vanilla JavaScript?
4. How do you preserve cursor position in a controlled input?
5. What is the `isComposing` property and when should you check it?
6. How do you avoid infinite update loops with controlled inputs?
7. How would you implement undo/redo for a controlled input?
8. What are the performance implications of controlled inputs?

### Coding Challenges

1. **Input Mask**: Implement phone, date, and SSN input masks using controlled input pattern
2. **Rich Text Input**: Build a controlled contenteditable div with formatting
3. **Undo/Redo Input**: Create an input with full undo/redo history
4. **Reactive Form**: Build a form where fields react to changes in other fields
5. **Input Pipeline**: Implement a configurable transformation/validation pipeline for inputs

### Related Topics

- [React Controlled Components](https://reactjs.org/docs/forms.html#controlled-components)
- [InputEvent](https://developer.mozilla.org/en-US/docs/Web/API/InputEvent)
- [InputEvent.inputType](https://developer.mozilla.org/en-US/docs/Web/API/InputEvent/inputType)
- [ContentEditable](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/contenteditable)
- [Selection API](https://developer.mozilla.org/en-US/docs/Web/API/Selection)
