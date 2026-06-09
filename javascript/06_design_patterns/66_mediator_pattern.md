# Mediator Pattern - Centralized control, component decoupling, mediator vs observer

## Introduction

The Mediator Pattern centralizes communication between multiple objects (colleagues) by introducing a mediator object that handles all interactions. Instead of components communicating directly, they communicate through the mediator, which reduces coupling and makes the system easier to maintain and extend.

This pattern is widely used in GUI systems, chat applications, air traffic control systems, and complex form validation. This file covers centralized control via a mediator, how mediators decouple components, and a comparison between mediator and observer patterns.

## Centralized Control

### What It Is

Centralized control in the Mediator Pattern means that a single mediator object manages all interactions between colleague objects. Colleagues don't communicate directly; they send messages to the mediator, which determines how to route and handle them.

### Why It Is Important

Without a mediator, components in a system become tightly coupled — each component must know about many others. This creates a many-to-many communication graph. The mediator reduces this to a star topology: all components connect only to the mediator, simplifying maintenance and making the system easier to understand.

### How It Works Internally

Each colleague holds a reference to the mediator. When a colleague needs to communicate, it calls a method on the mediator. The mediator then decides which colleagues need to be notified and how. The mediator encapsulates the interaction logic in one place, rather than distributing it across all components.

### Syntax

```javascript
// Mediator interface
class Mediator {
  notify(sender, event, data) {
    // To be overridden
  }
}

// Colleague base
class Colleague {
  constructor(mediator) {
    this.mediator = mediator;
  }
}
```

### Beginner Examples

```javascript
// Chat room mediator
class ChatRoom {
  constructor() {
    this.users = new Map();
  }

  register(user) {
    this.users.set(user.name, user);
    user.setMediator(this);
    this.notify(null, "join", { user: user.name });
  }

  notify(sender, event, data) {
    switch (event) {
      case "join":
        this.broadcast(sender, `${data.user} joined the chat`);
        break;
      case "leave":
        this.broadcast(sender, `${data.user} left the chat`);
        break;
      case "message":
        this.broadcast(sender, `${sender.name}: ${data.message}`);
        break;
      case "private":
        const recipient = this.users.get(data.to);
        if (recipient) {
          recipient.receive(private, sender.name, data.message);
        }
        break;
    }
  }

  broadcast(sender, message) {
    for (const user of this.users.values()) {
      if (user !== sender) {
        user.receive("broadcast", "System", message);
      }
    }
  }
}

class User {
  constructor(name) {
    this.name = name;
    this.mediator = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  send(message) {
    this.mediator.notify(this, "message", { message });
  }

  sendPrivate(to, message) {
    this.mediator.notify(this, "private", { to, message });
  }

  receive(type, from, message) {
    console.log(`[${type}] ${from}: ${message}`);
  }

  leave() {
    this.mediator.notify(this, "leave", { user: this.name });
  }
}

const chat = new ChatRoom();
const alice = new User("Alice");
const bob = new User("Bob");
const charlie = new User("Charlie");

chat.register(alice);
chat.register(bob);
chat.register(charlie);

alice.send("Hello everyone!");
bob.sendPrivate("Alice", "Hey Alice!");
```

### Intermediate Examples

```javascript
// Form validation mediator
class FormMediator {
  constructor() {
    this.fields = new Map();
    this.submitButton = null;
  }

  registerField(name, field) {
    this.fields.set(name, field);
    field.setMediator(this);
  }

  registerSubmitButton(button) {
    this.submitButton = button;
    button.setMediator(this);
  }

  notify(sender, event, data) {
    switch (event) {
      case "change":
        this.onFieldChange(sender);
        break;
      case "click":
        this.onSubmit();
        break;
    }
  }

  onFieldChange(changedField) {
    // Validate all fields on any change
    let allValid = true;
    for (const [name, field] of this.fields) {
      const error = field.validate();
      if (error) {
        allValid = false;
        field.showError(error);
      } else {
        field.clearError();
      }
    }
    this.submitButton.setEnabled(allValid);
  }

  onSubmit() {
    let allValid = true;
    const data = {};

    for (const [name, field] of this.fields) {
      const error = field.validate();
      if (error) {
        allValid = false;
        field.showError(error);
      } else {
        data[name] = field.getValue();
      }
    }

    if (allValid) {
      console.log("Form submitted:", data);
    }
  }
}

class FormField {
  constructor(mediator) {
    this.mediator = mediator;
    this.value = "";
    this.errorEl = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  setValue(value) {
    this.value = value;
    this.mediator.notify(this, "change");
  }

  getValue() { return this.value; }
  validate() { /* Override */ }
  showError(msg) { console.error(msg); }
  clearError() {}
}

class EmailField extends FormField {
  validate() {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.value)
      ? null
      : "Invalid email address";
  }
}

class PasswordField extends FormField {
  validate() {
    return this.value.length >= 8
      ? null
      : "Password must be at least 8 characters";
  }
}

class SubmitButton {
  constructor(mediator) {
    this.mediator = mediator;
    this.enabled = false;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  setEnabled(enabled) {
    this.enabled = enabled;
    console.log(`Submit button ${enabled ? "enabled" : "disabled"}`);
  }

  click() {
    if (this.enabled) {
      this.mediator.notify(this, "click");
    }
  }
}

// Usage
const form = new FormMediator();
const email = new EmailField();
const password = new PasswordField();
const submit = new SubmitButton();

form.registerField("email", email);
form.registerField("password", password);
form.registerSubmitButton(submit);

email.setValue("test@example.com");
password.setValue("12345678");
submit.click();
```

### Advanced Examples

```javascript
// Air traffic control mediator
class AirTrafficControl {
  constructor() {
    this.aircraft = new Map();
    this.runways = new Map();
    this.weather = { visibility: 10, wind: { direction: 270, speed: 15 } };
  }

  registerAircraft(aircraft) {
    this.aircraft.set(aircraft.callsign, aircraft);
    aircraft.setMediator(this);
  }

  registerRunway(runway) {
    this.runways.set(runway.id, runway);
    runway.setMediator(this);
  }

  notify(sender, event, data) {
    switch (event) {
      case "requestTakeoff":
        return this.handleTakeoffRequest(sender);
      case "requestLanding":
        return this.handleLandingRequest(sender, data);
      case "position":
        this.handlePositionUpdate(sender, data);
        break;
      case "emergency":
        this.handleEmergency(sender, data);
        break;
      case "runwayStatus":
        this.handleRunwayStatusChange(sender, data);
        break;
    }
  }

  handleTakeoffRequest(aircraft) {
    const availableRunway = this.findAvailableRunway();
    if (!availableRunway) {
      return { approved: false, reason: "No available runway" };
    }

    if (this.weather.visibility < 2 || this.weather.wind.speed > 40) {
      return { approved: false, reason: "Weather conditions not safe" };
    }

    const conflicting = this.findConflictingAircraft(aircraft, "takeoff");
    if (conflicting) {
      return { approved: false, reason: `Conflicting with ${conflicting.callsign}` };
    }

    availableRunway.allocate(aircraft);
    this.broadcast(aircraft, `${aircraft.callsign} cleared for takeoff on runway ${availableRunway.id}`);
    return { approved: true, runway: availableRunway.id };
  }

  handleLandingRequest(aircraft, data) {
    const availableRunway = this.findAvailableRunway();
    if (!availableRunway) {
      aircraft.hold();
      return { approved: false, reason: "No available runway, holding pattern" };
    }

    const spacing = this.calculateSpacing(aircraft, data.approachSpeed);
    availableRunway.allocate(aircraft);
    this.broadcast(aircraft, `${aircraft.callsign} cleared to land on runway ${availableRunway.id}`);
    return { approved: true, runway: availableRunway.id };
  }

  findAvailableRunway() {
    for (const runway of this.runways.values()) {
      if (!runway.isOccupied()) return runway;
    }
    return null;
  }

  findConflictingAircraft(aircraft, operation) {
    for (const other of this.aircraft.values()) {
      if (other === aircraft) continue;
      if (other.isInConflict(aircraft.getPosition(), operation)) {
        return other;
      }
    }
    return null;
  }

  broadcast(sender, message) {
    for (const aircraft of this.aircraft.values()) {
      aircraft.receiveMessage(message);
    }
  }

  handleEmergency(sender, data) {
    console.log(`EMERGENCY: ${sender.callsign} - ${data.type}`);
    // Clear runway, redirect traffic, notify emergency services
    for (const runway of this.runways.values()) {
      if (runway.currentAircraft === sender) {
        continue;
      }
      runway.clearForEmergency();
    }
    this.broadcast(sender, `EMERGENCY: All aircraft hold position. ${sender.callsign} has priority.`);
  }

  handlePositionUpdate(sender, data) {
    // Check for potential collisions
    for (const other of this.aircraft.values()) {
      if (other === sender) continue;
      const distance = this.calculateDistance(data, other.getPosition());
      if (distance < 3) { // less than 3 nautical miles
        this.warnCollision(sender, other);
      }
    }
  }

  calculateDistance(pos1, pos2) {
    return Math.sqrt(
      Math.pow(pos1.x - pos2.x, 2) + Math.pow(pos1.y - pos2.y, 2)
    );
  }

  warnCollision(a, b) {
    console.log(`COLLISION WARNING: ${a.callsign} and ${b.callsign}`);
    this.broadcast(a, `COLLISION ALERT: ${a.callsign} and ${b.callsign} on converging courses`);
  }
}

class Aircraft {
  constructor(callsign, type) {
    this.callsign = callsign;
    this.type = type;
    this.mediator = null;
    this.position = { x: 0, y: 0, altitude: 0 };
    this.heading = 0;
    this.speed = 0;
    this.holding = false;
  }

  setMediator(mediator) { this.mediator = mediator; }
  getPosition() { return { ...this.position }; }

  requestTakeoff() {
    return this.mediator.notify(this, "requestTakeoff");
  }

  requestLanding(approachSpeed) {
    return this.mediator.notify(this, "requestLanding", { approachSpeed });
  }

  updatePosition(x, y, altitude) {
    this.position = { x, y, altitude };
    this.mediator.notify(this, "position", this.position);
  }

  declareEmergency(type) {
    this.mediator.notify(this, "emergency", { type });
  }

  isInConflict(otherPos, operation) {
    const distance = Math.sqrt(
      Math.pow(this.position.x - otherPos.x, 2) +
      Math.pow(this.position.y - otherPos.y, 2)
    );
    return distance < 5;
  }

  hold() {
    this.holding = true;
    console.log(`${this.callsign} entering holding pattern`);
  }

  receiveMessage(message) {
    console.log(`[${this.callsign}] ${message}`);
  }
}

class Runway {
  constructor(id) {
    this.id = id;
    this.mediator = null;
    this.currentAircraft = null;
    this.active = true;
  }

  setMediator(mediator) { this.mediator = mediator; }
  isOccupied() { return this.currentAircraft !== null; }
  allocate(aircraft) { this.currentAircraft = aircraft; }

  release() {
    this.currentAircraft = null;
    this.mediator.notify(this, "runwayStatus", { id: this.id, status: "available" });
  }

  clearForEmergency() {
    if (this.currentAircraft) {
      console.log(`Runway ${this.id}: ${this.currentAircraft.callsign} instructed to clear`);
    }
  }
}

// Mediator with async operations and queuing
class AsyncMediator {
  constructor() {
    this.colleagues = new Map();
    this.messageQueue = [];
    this.isProcessing = false;
  }

  register(name, colleague) {
    this.colleagues.set(name, colleague);
    colleague.setMediator(this);
  }

  async send(from, to, type, data) {
    const promise = new Promise((resolve, reject) => {
      this.messageQueue.push({ from, to, type, data, resolve, reject });
    });

    if (!this.isProcessing) {
      this.processQueue();
    }

    return promise;
  }

  async processQueue() {
    this.isProcessing = true;

    while (this.messageQueue.length > 0) {
      const msg = this.messageQueue.shift();
      try {
        const result = await this.handleMessage(msg);
        msg.resolve(result);
      } catch (err) {
        msg.reject(err);
      }
    }

    this.isProcessing = false;
  }

  async handleMessage(msg) {
    const { from, to, type, data } = msg;

    if (to === "*") {
      // Broadcast
      const results = [];
      for (const [name, colleague] of this.colleagues) {
        if (name !== from) {
          results.push(await colleague.onMessage(from, type, data));
        }
      }
      return results;
    }

    const recipient = this.colleagues.get(to);
    if (!recipient) throw new Error(`Colleague "${to}" not found`);

    return recipient.onMessage(from, type, data);
  }
}
```

### Real-World Use Cases

- **Chat applications**: Chat server acts as mediator between clients.
- **UI frameworks**: Dialog/modals as mediators for form controls.
- **Air traffic control**: Central ATC manages all aircraft communication.
- **Microservices orchestration**: Orchestrator mediates between services.
- **Smart home systems**: Central hub mediates between devices.
- **Multiplayer games**: Game server mediates player interactions.

### Common Mistakes

```javascript
// Mistake 1: Mediator becomes a god object (does everything)
class GodMediator {
  notify(sender, event, data) {
    // 500 lines of conditional logic
    if (event === "click") { /* 50 lines */ }
    else if (event === "change") { /* 50 lines */ }
    // ...
  }
}
// Solution: Split into domain-specific mediators

// Mistake 2: Colleagues bypassing the mediator
class BadComponent {
  constructor() {
    this.mediator = null;
  }

  doSomething() {
    otherComponent.doSomethingElse(); // Direct coupling!
  }
}

// Mistake 3: Mediator holding too much state
// The mediator should coordinate, not store all application data

// Mistake 4: Colleagues that are tightly coupled to mediator details
class CoupledComponent {
  notify(mediator, event) {
    // Knowing mediator internals
    mediator.database.save(mediator.getState().currentUser);
  }
}
```

### Best Practices

```javascript
// Keep mediator focused on coordination, not business logic
class DialogMediator {
  notify(sender, event, data) {
    switch (event) {
      case "ok":
        this.close();
        break;
      case "cancel":
        this.close();
        break;
      case "change":
        this.validate();
        break;
    }
  }
  // Delegate validation and close logic to dedicated objects
}

// Use event constants for communication
const Events = Object.freeze({
  USER_LOGIN: "user:login",
  USER_LOGOUT: "user:logout",
  DATA_LOADED: "data:loaded"
});

// Avoid passing raw data; use typed message objects
class Message {
  constructor(type, sender, payload) {
    this.type = type;
    this.sender = sender;
    this.payload = payload;
    this.timestamp = Date.now();
  }
}

// Colleagues should only know about the mediator interface
// not about other colleagues
```

### Performance Considerations

- The mediator becomes a single point of routing — all communication passes through it.
- For high-frequency events, the mediator can become a bottleneck.
- Consider domains-specific mediators rather than one global mediator.
- Asynchronous mediator with queuing can handle backpressure.
- Mediator's notify method should be O(1) or O(log n) for event dispatch.

### Interview Questions

**Q: When should you use the Mediator pattern?**
A: Use the Mediator pattern when a set of objects communicate in complex ways, creating a tangled web of dependencies. The mediator centralizes this communication, making the system easier to understand, maintain, and extend. Chat rooms, form validators, and air traffic control are classic examples.

**Q: What is the "mediator knows all" anti-pattern?**
A: This occurs when the mediator accumulates too much responsibility — validating data, storing state, handling business logic, and coordinating communication. The mediator should only coordinate communication; other responsibilities should be delegated to dedicated objects.

**Q: How is mediator different from facade?**
A: Facade provides a simplified interface to a subsystem without adding functionality. Mediator actively coordinates communication between components and adds the coordination behavior. Facade hides complexity; Mediator manages interactions.

### Coding Challenges

```javascript
// Challenge 1: Implement a task board mediator (like Trello)
// that coordinates between Task, Column, Board, and User components.
// Moving a task between columns should trigger notifications,
// user assignments, and audit logging.

// Challenge 2: Build a smart home mediator that coordinates
// between Thermostat, Lights, DoorLock, and SecurityCamera.
// When the alarm is armed, doors should lock and lights should turn off.

// Challenge 3: Create a tournament match mediator for a game
// that coordinates between Player, Scorekeeper, Timer, and Spectator components.
// The mediator should manage game start, scoring, timeouts, and win conditions.
```

### Related Topics

- Observer pattern
- Facade pattern
- Command pattern
- Event bus architecture
- Enterprise Service Bus (ESB)

## Component Decoupling

### What It Is

Component decoupling through the Mediator Pattern means that components (colleagues) have no direct references to each other. They only know about the mediator interface, not about other components' existence. This eliminates many-to-many dependencies in favor of a star topology.

### Why It Is Important

Decoupled components are independently developable, testable, and replaceable. A change in one component doesn't ripple through the system. New components can be added without modifying existing ones. This directly supports the Open/Closed Principle and makes large systems manageable.

### How It Works Internally

Each colleague maintains a reference to the mediator (typically injected via constructor or setter). When a colleague needs to communicate, it calls a generic method on the mediator (e.g., `notify`, `send`). The mediator resolves the recipients and delivers the message. Colleagues never import or reference each other.

### Syntax

```javascript
// Fully decoupled component
class ComponentA {
  constructor(mediator) {
    this.mediator = mediator;
  }

  doAction() {
    this.mediator.notify(this, "action", { /* data */ });
  }
}

class ComponentB {
  constructor(mediator) {
    this.mediator = mediator;
  }

  handleAction(data) {
    // React without knowing about ComponentA
  }
}
```

### Beginner Examples

```javascript
// GUI dialog mediator - decoupled form elements
class DialogMediator {
  constructor() {
    this.title = null;
    this.message = null;
    this.okButton = null;
    this.cancelButton = null;
  }

  setTitle(title) { this.title = title; title.setMediator(this); }
  setMessage(msg) { this.message = msg; msg.setMediator(this); }
  setOkButton(btn) { this.okButton = btn; btn.setMediator(this); }
  setCancelButton(btn) { this.cancelButton = btn; btn.setMediator(this); }

  notify(sender, event) {
    if (event === "ok") {
      const titleText = this.title.getText();
      const msgText = this.message.getText();
      console.log(`Dialog confirmed: "${titleText}" - "${msgText}"`);
      this.close();
    } else if (event === "cancel") {
      console.log("Dialog cancelled");
      this.close();
    }
  }

  close() {
    this.title.hide();
    this.message.hide();
    this.okButton.hide();
    this.cancelButton.hide();
  }
}

class UIElement {
  constructor(mediator) { this.mediator = mediator; }
  setMediator(m) { this.mediator = m; }
  show() { console.log(`${this.constructor.name} shown`); }
  hide() { console.log(`${this.constructor.name} hidden`); }
}

class Title extends UIElement {
  constructor(text) { super(); this.text = text; }
  getText() { return this.text; }
}

class Message extends UIElement {
  constructor(text) { super(); this.text = text; }
  getText() { return this.text; }
}

class Button extends UIElement {
  constructor(label) { super(); this.label = label; }
  click() { this.mediator.notify(this, this.label.toLowerCase()); }
}

// Components have no knowledge of each other
const dialog = new DialogMediator();
dialog.setTitle(new Title("Confirm Delete"));
dialog.setMessage(new Message("Are you sure?"));
dialog.setOkButton(new Button("OK"));
dialog.setCancelButton(new Button("Cancel"));

dialog.okButton.click(); // OK button doesn't know about Title or Message
```

### Intermediate Examples

```javascript
// Microservice orchestrator mediator
class ServiceOrchestrator {
  constructor() {
    this.services = new Map();
  }

  register(name, service) {
    this.services.set(name, service);
    service.setMediator(this);
  }

  async notify(sender, event, data) {
    switch (event) {
      case "user:registered":
        await this.handleUserRegistration(data);
        break;
      case "order:placed":
        await this.handleOrderPlaced(data);
        break;
      case "payment:completed":
        await this.handlePaymentCompleted(data);
        break;
    }
  }

  async handleUserRegistration(data) {
    // Coordinate multiple services independently
    const results = await Promise.allSettled([
      this.callService("email", "sendWelcome", { email: data.email }),
      this.callService("analytics", "track", { event: "registration", data }),
      this.callService("crm", "createContact", { name: data.name, email: data.email })
    ]);

    const failures = results.filter(r => r.status === "rejected");
    if (failures.length > 0) {
      console.error("Some registration tasks failed:", failures);
    }
  }

  async handleOrderPlaced(data) {
    this.callService("inventory", "reserveItems", data.items);
    this.callService("payment", "processPayment", {
      orderId: data.orderId,
      amount: data.total,
      userId: data.userId
    });
    this.callService("email", "sendConfirmation", {
      email: data.email,
      orderId: data.orderId
    });
  }

  async handlePaymentCompleted(data) {
    this.callService("shipping", "createShipment", data);
    this.callService("email", "sendShippingNotification", data);
    this.callService("loyalty", "awardPoints", {
      userId: data.userId,
      points: Math.floor(data.amount)
    });
  }

  async callService(name, method, data) {
    const service = this.services.get(name);
    if (!service) throw new Error(`Service ${name} not found`);
    return service[method](data);
  }
}

// Services with no knowledge of each other
class EmailService {
  setMediator(m) { this.mediator = m; }
  async sendWelcome({ email }) {
    console.log(`Sending welcome email to ${email}`);
  }
  async sendConfirmation({ email, orderId }) {
    console.log(`Sending order ${orderId} confirmation to ${email}`);
  }
  async sendShippingNotification(data) {
    console.log(`Sending shipping notification`);
  }
}

class PaymentService {
  setMediator(m) { this.mediator = m; }
  async processPayment({ orderId, amount }) {
    console.log(`Processing payment of $${amount} for order ${orderId}`);
    return { success: true, transactionId: Date.now() };
  }
}

class InventoryService {
  setMediator(m) { this.mediator = m; }
  async reserveItems(items) {
    console.log(`Reserving ${items.length} items in inventory`);
  }
}

class ShippingService {
  setMediator(m) { this.mediator = m; }
  async createShipment({ orderId }) {
    console.log(`Creating shipment for order ${orderId}`);
  }
}

class AnalyticsService {
  setMediator(m) { this.mediator = m; }
  async track({ event, data }) {
    console.log(`Tracking event: ${event}`);
  }
}

class CrmService {
  setMediator(m) { this.mediator = m; }
  async createContact({ name, email }) {
    console.log(`Creating CRM contact: ${name} <${email}>`);
  }
}

class LoyaltyService {
  setMediator(m) { this.mediator = m; }
  async awardPoints({ userId, points }) {
    console.log(`Awarding ${points} points to user ${userId}`);
  }
}

// Usage
const orchestrator = new ServiceOrchestrator();
orchestrator.register("email", new EmailService());
orchestrator.register("payment", new PaymentService());
orchestrator.register("inventory", new InventoryService());
orchestrator.register("shipping", new ShippingService());
orchestrator.register("analytics", new AnalyticsService());
orchestrator.register("crm", new CrmService());
orchestrator.register("loyalty", new LoyaltyService());

// Trigger from user registration
await orchestrator.notify(null, "user:registered", {
  name: "Alice",
  email: "alice@example.com"
});
```

### Advanced Examples

```javascript
// Plugin system with mediator decoupling
class PluginMediator {
  constructor() {
    this.plugins = new Map();
    this.hooks = new Map();
    this.core = null;
  }

  registerPlugin(name, plugin) {
    this.plugins.set(name, plugin);
    plugin.setMediator(this);

    // Register hooks declared by the plugin
    if (plugin.hooks) {
      for (const [hookName, handler] of Object.entries(plugin.hooks)) {
        if (!this.hooks.has(hookName)) {
          this.hooks.set(hookName, []);
        }
        this.hooks.get(hookName).push({ plugin: name, handler });
      }
    }

    plugin.onRegistered();
  }

  setCore(core) {
    this.core = core;
    core.setMediator(this);
  }

  async runHook(hookName, context) {
    const hooks = this.hooks.get(hookName) || [];
    let result = context;

    for (const { plugin, handler } of hooks) {
      try {
        result = await handler(result);
      } catch (err) {
        console.error(`Plugin ${plugin} failed on hook ${hookName}:`, err);
      }
    }

    return result;
  }

  async notify(sender, event, data) {
    switch (event) {
      case "core:initialized":
        await this.runHook("afterInit", data);
        break;
      case "core:beforeRender":
        await this.runHook("beforeRender", data);
        break;
      case "core:afterRender":
        await this.runHook("afterRender", data);
        break;
      case "plugin:event":
        await this.runHook(data.hook, data.context);
        break;
    }
  }

  getPlugin(name) {
    return this.plugins.get(name);
  }
}

class PluginCore {
  constructor() {
    this.mediator = null;
  }
  setMediator(m) { this.mediator = m; }

  async initialize() {
    console.log("Core initialized");
    await this.mediator.notify(this, "core:initialized", { version: "1.0" });
  }

  async render(content) {
    let processed = await this.mediator.runHook("beforeRender", content);
    // Core rendering logic
    processed = `<main>${processed}</main>`;
    processed = await this.mediator.runHook("afterRender", processed);
    return processed;
  }
}

class SEOPlugin {
  constructor() {
    this.mediator = null;
    this.hooks = {
      afterInit: async (ctx) => {
        console.log("SEO: Initializing sitemap generator");
        return { ...ctx, seo: true };
      },
      beforeRender: async (content) => {
        return `<!-- SEO optimized -->\n${content}`;
      },
      afterRender: async (html) => {
        return html + `\n<!-- Generated by SEO Plugin -->`;
      }
    };
  }
  setMediator(m) { this.mediator = m; }
  onRegistered() {
    console.log("SEO Plugin registered");
  }
}

class AnalyticsPlugin {
  constructor() {
    this.mediator = null;
    this.hooks = {
      afterInit: async (ctx) => {
        console.log("Analytics: Setting up tracking");
        return { ...ctx, analytics: true };
      },
      afterRender: async (html) => {
        return html + `\n<script>console.log('Analytics tracking');</script>`;
      }
    };
  }
  setMediator(m) { this.mediator = m; }
  onRegistered() {
    console.log("Analytics Plugin registered");
  }
}

// Usage
const mediator = new PluginMediator();
const core = new PluginCore();

mediator.setCore(core);
mediator.registerPlugin("seo", new SEOPlugin());
mediator.registerPlugin("analytics", new AnalyticsPlugin());

await core.initialize();
const result = await core.render("<h1>Hello World</h1>");
console.log(result);

// Event-driven UI with pub/sub mediator
class UIEventMediator {
  constructor() {
    this.components = new Map();
    this.eventHistory = [];
    this.maxHistory = 100;
  }

  addComponent(name, component) {
    this.components.set(name, component);
    component.connect(this);
  }

  dispatch(event) {
    const entry = { ...event, timestamp: Date.now() };
    this.eventHistory.push(entry);
    if (this.eventHistory.length > this.maxHistory) {
      this.eventHistory.shift();
    }

    const component = this.components.get(event.target);
    if (component) {
      component.handleEvent(event);
    }
  }

  queryEvents(filter) {
    return this.eventHistory.filter(e => {
      return (!filter.type || e.type === filter.type) &&
             (!filter.target || e.target === filter.target) &&
             (!filter.since || e.timestamp > filter.since);
    });
  }
}

class UIComponent {
  constructor() {
    this.mediator = null;
  }
  connect(mediator) {
    this.mediator = mediator;
  }
  handleEvent(event) {
    // Override in subclasses
  }
  dispatch(event) {
    this.mediator.dispatch({
      ...event,
      source: this.constructor.name
    });
  }
}

class SearchBox extends UIComponent {
  constructor() { super(); this.query = ""; }
  onInput(text) {
    this.query = text;
    this.dispatch({
      type: "search:query",
      target: "searchResults",
      data: { query: text }
    });
  }
}

class SearchResults extends UIComponent {
  handleEvent(event) {
    if (event.type === "search:query") {
      this.search(event.data.query);
    }
  }
  search(query) {
    console.log(`Searching for: ${query}`);
  }
}

class FilterPanel extends UIComponent {
  constructor() { super(); this.filters = {}; }
  applyFilter(filterName, value) {
    this.filters[filterName] = value;
    this.dispatch({
      type: "filter:changed",
      target: "searchResults",
      data: this.filters
    });
  }
}
```

### Real-World Use Cases

- **UI frameworks**: WPF (MVVM), Angular services, React Context — all mediate component interactions.
- **Microservices choreography**: An orchestrator service mediates between domain services.
- **Plugin systems**: WordPress hooks, VS Code extensions — mediators manage plugin interactions.
- **Game engines**: Entity-Component-System (ECS) patterns use mediators for decoupled entity interactions.
- **Workflow engines**: Steps communicate through a central coordinator.

### Common Mistakes

```javascript
// Mistake 1: Components holding direct references to each other
class LeakyComponent {
  constructor() {
    this.mediator = null;
    this.otherComponent = null; // Direct reference!
  }
}

// Mistake 2: The mediator knowing component internals
class BadMediator {
  notify(component, event) {
    if (component instanceof SpecificComponent) {
      component.specificInternalMethod(); // Too coupled
    }
  }
}

// Mistake 3: Passing the mediator reference everywhere
// Only pass to components that actually need it

// Mistake 4: Using a single global mediator
// Better to use multiple focused mediators
```

### Best Practices

```javascript
// Inject mediator via constructor (clear dependency)
class Component {
  constructor(mediator) {
    if (!mediator) throw new Error("Mediator required");
    this.mediator = mediator;
  }
}

// Use interfaces for mediator to keep components generic
const MediatorInterface = {
  notify(sender, event, data) {}
};

// Keep mediator interface minimal
// Only expose the methods components actually need

// Test components independently by mocking the mediator
class MockMediator {
  notify(sender, event, data) {
    this.lastCall = { sender, event, data };
  }
}

// Consider event-based mediators for very loose coupling
// Components emit events, mediator subscribes
```

### Performance Considerations

- Decoupling adds indirection — every communication goes through the mediator.
- For extremely high-frequency communication (like real-time audio), direct coupling may be necessary.
- Batch updates through the mediator to avoid cascading notifications.
- Consider using a mediator tree (hierarchical mediators) to scope communication.

### Interview Questions

**Q: How does the Mediator pattern help with testing?**
A: Components can be tested in isolation by injecting a mock mediator that records interactions. You can verify that the component sends the correct events without needing to instantiate all other components. This dramatically simplifies unit testing in complex systems.

**Q: What is the difference between mediator and facade?**
A: Facade provides a simplified interface to a subsystem but doesn't add behavior. Mediator actively manages communication between components and adds coordination logic. Components don't know about each other in Mediator; in Facade, the subsystem components may still know about each other.

**Q: How would you refactor a tightly coupled system to use a mediator?**
A: (1) Identify the communication patterns between components. (2) Define a mediator interface. (3) Create the mediator class that handles all communications. (4) Remove direct references between components and replace with mediator calls. (5) Test each component independently with a mock mediator.

### Coding Challenges

```javascript
// Challenge 1: Refactor a tightly coupled e-commerce checkout
// (Cart, Payment, Shipping, Email, Inventory) to use a mediator.
// Components currently call each other directly.

// Challenge 2: Build an undo/redo mediator that tracks state changes
// from multiple components and provides centralized undo/redo.

// Challenge 3: Create a mediator for a multiplayer game that
// decouples Player, GameBoard, ScoreManager, Timer, and PowerUp components.
// The mediator should handle all game events.
```

### Related Topics

- Facade pattern
- Observer pattern
- Dependency injection
- Event-driven architecture
- Inversion of Control

## Mediator vs Observer

### What It Is

The Mediator and Observer patterns are both behavioral patterns that facilitate communication between objects. While they share the goal of reducing coupling, they differ in structure, intent, and use cases. Understanding the distinction is crucial for choosing the right pattern.

### Why It Is Important

Choosing between Mediator and Observer affects system architecture fundamentally. The wrong choice can lead to over-engineering (using Mediator for simple one-to-many communication) or under-engineering (using Observer for complex many-to-many interactions where a central coordinator is needed).

### How It Works Internally

**Observer**: A subject maintains a list of observers and notifies them directly. Communication is one-to-many from subject to observers.

**Mediator**: Colleagues communicate through a central mediator. Communication is many-to-many, but all through a central point.

The key difference is in the direction of communication and the level of indirection.

### Syntax

```javascript
// Observer: Subject knows observers
class Subject {
  constructor() {
    this.observers = [];
  }
  notify(data) {
    this.observers.forEach(o => o.update(data));
  }
}

// Mediator: Components only know the mediator
class Mediator {
  notify(sender, event, data) {
    // Route to appropriate colleagues
  }
}
```

### Beginner Examples

```javascript
// Observer example: Stock ticker
class Stock {
  constructor(symbol) {
    this.symbol = symbol;
    this.price = 0;
    this.investors = [];
  }

  addInvestor(investor) {
    this.investors.push(investor);
  }

  setPrice(price) {
    this.price = price;
    this.investors.forEach(i => i.notify(this.symbol, price));
  }
}

// Mediator example: Stock trading platform
class TradingPlatform {
  constructor() {
    this.traders = new Map();
    this.stocks = new Map();
  }

  addTrader(name, trader) {
    this.traders.set(name, trader);
    trader.connect(this);
  }

  placeOrder(trader, stockSymbol, quantity, type) {
    const stock = this.stocks.get(stockSymbol);
    if (type === "buy") {
      // Match with sellers
      const seller = this.findSeller(stockSymbol, quantity);
      if (seller) {
        this.executeTrade(trader, seller, stockSymbol, quantity);
      }
    }
  }

  executeTrade(buyer, seller, symbol, quantity) {
    this.traders.forEach(t => {
      if (t === buyer || t === seller) {
        t.onTradeExecuted(symbol, quantity);
      }
    });
  }
}

// When to use which:
// Use Observer when:
// - You have a one-to-many dependency
// - The subject creates the events
// - Simple notification is sufficient

// Use Mediator when:
// - Many objects interact in complex ways
// - You need central coordination
// - Components should not know about each other
```

### Intermediate Examples

```javascript
// Mixed pattern: Mediator using Observers internally
class SmartHomeMediator {
  constructor() {
    this.devices = new Map();
    this.eventBus = new Map(); // Internal observer pattern
  }

  registerDevice(name, device) {
    this.devices.set(name, device);
    device.setMediator(this);

    // Device-specific subscriptions
    if (device.listenTo) {
      for (const event of device.listenTo) {
        this.subscribe(event, device);
      }
    }
  }

  subscribe(event, device) {
    if (!this.eventBus.has(event)) {
      this.eventBus.set(event, new Set());
    }
    this.eventBus.get(event).add(device);
  }

  notify(sender, event, data) {
    // Mediator logic: decide what happens
    switch (event) {
      case "motion:detected":
        this.handleMotion(sender, data);
        break;
      case "temperature:high":
        this.handleHighTemp(sender, data);
        break;
      case "alarm:triggered":
        this.handleAlarm(sender, data);
        break;
      default:
        this.broadcast(event, data);
    }
  }

  handleMotion(sensor, data) {
    // Complex coordination logic
    const lights = this.devices.get("living_room_lights");
    if (data.time === "night") {
      lights.turnOn({ brightness: 30 });
    }
    const camera = this.devices.get("security_camera");
    camera.startRecording();
  }

  handleHighTemp(sensor, data) {
    const ac = this.devices.get("thermostat");
    ac.setTemperature(22);
    const phone = this.devices.get("phone");
    phone.sendNotification("Temperature is high, AC adjusted");
  }

  handleAlarm(sensor, data) {
    this.broadcast("alarm", data);
    const phone = this.devices.get("phone");
    phone.sendNotification("ALARM TRIGGERED!");
    phone.callEmergency();
  }

  broadcast(event, data) {
    const subscribers = this.eventBus.get(event) || new Set();
    for (const device of subscribers) {
      device.onEvent(event, data);
    }
  }
}

class SmartDevice {
  constructor(name) {
    this.name = name;
    this.mediator = null;
    this.listenTo = [];
  }

  setMediator(m) { this.mediator = m; }

  onEvent(event, data) {
    // Override in subclasses
  }
}

class MotionSensor extends SmartDevice {
  constructor() {
    super("motion_sensor");
  }

  detectMotion() {
    this.mediator.notify(this, "motion:detected", {
      time: new Date().getHours() < 6 ? "night" : "day"
    });
  }
}

class SmartLight extends SmartDevice {
  constructor() {
    super("light");
    this.listenTo = ["alarm"];
  }

  turnOn(config = {}) {
    console.log(`Light turned on (brightness: ${config.brightness || 100})`);
  }

  turnOff() {
    console.log("Light turned off");
  }

  onEvent(event, data) {
    if (event === "alarm") {
      this.turnOn({ brightness: 100 });
    }
  }
}

class Thermostat extends SmartDevice {
  constructor() {
    super("thermostat");
  }

  setTemperature(temp) {
    console.log(`Thermostat set to ${temp}°C`);
  }
}

// Comparison table in implementation
const patternComparison = {
  observer: {
    communication: "One-to-many",
    coupling: "Subject knows observers",
    whenToUse: "Simple notification, single source of events",
    example: "DOM event listeners, reactive state"
  },
  mediator: {
    communication: "Many-to-many",
    coupling: "All through mediator",
    whenToUse: "Complex interactions, coordination needed",
    example: "Chat room, form validation, workflow"
  }
};
```

### Advanced Examples

```javascript
// Hybrid Mediator-Observer: A mediator that uses observable state
class ObservableState {
  constructor(initialState) {
    this.state = { ...initialState };
    this.observers = new Set();
  }

  get(key) { return this.state[key]; }

  set(key, value) {
    const old = this.state[key];
    this.state[key] = value;
    this.notify({ key, value, oldValue: old });
  }

  observe(fn) {
    this.observers.add(fn);
    return () => this.observers.delete(fn);
  }

  notify(change) {
    for (const fn of this.observers) {
      fn(change);
    }
  }
}

// Mediator using observable state to coordinate
class AppMediator {
  constructor() {
    this.state = new ObservableState({
      user: null,
      cart: [],
      theme: "light",
      notifications: []
    });

    this.components = new Map();
  }

  register(name, component) {
    this.components.set(name, component);
    component.connect(this);

    // Component can observe state changes
    if (component.onStateChange) {
      this.state.observe((change) => {
        component.onStateChange(change);
      });
    }
  }

  dispatch(action) {
    console.log(`Mediator dispatching: ${action.type}`);

    switch (action.type) {
      case "USER_LOGIN":
        this.state.set("user", action.payload);
        this.state.set("notifications", [
          ...this.state.get("notifications"),
          { text: `Welcome ${action.payload.name}!`, type: "success" }
        ]);
        break;

      case "USER_LOGOUT":
        this.state.set("user", null);
        this.state.set("cart", []);
        break;

      case "CART_ADD":
        const cart = this.state.get("cart");
        this.state.set("cart", [...cart, action.payload]);
        break;

      case "CART_REMOVE":
        this.state.set("cart",
          this.state.get("cart").filter(item => item.id !== action.payload.id)
        );
        break;

      case "TOGGLE_THEME":
        this.state.set("theme",
          this.state.get("theme") === "light" ? "dark" : "light"
        );
        break;
    }
  }

  getState(key) {
    return this.state.get(key);
  }
}

// Usage
const app = new AppMediator();

class HeaderComponent {
  connect(mediator) {
    this.mediator = mediator;
  }

  onStateChange(change) {
    if (change.key === "user") {
      this.renderUser(change.value);
    }
    if (change.key === "theme") {
      this.applyTheme(change.value);
    }
  }

  renderUser(user) {
    console.log(`Header: ${user ? `Welcome, ${user.name}` : "Not logged in"}`);
  }

  applyTheme(theme) {
    console.log(`Header: Theme changed to ${theme}`);
  }
}

class CartComponent {
  connect(mediator) {
    this.mediator = mediator;
  }

  onStateChange(change) {
    if (change.key === "cart") {
      this.renderCart(change.value);
    }
  }

  renderCart(items) {
    console.log(`Cart: ${items.length} items ($${items.reduce((s, i) => s + i.price, 0)})`);
  }
}

class ThemeToggle {
  connect(mediator) {
    this.mediator = mediator;
  }
  toggle() {
    this.mediator.dispatch({ type: "TOGGLE_THEME" });
  }
}

app.register("header", new HeaderComponent());
app.register("cart", new CartComponent());
app.register("theme", new ThemeToggle());

app.dispatch({ type: "USER_LOGIN", payload: { name: "Alice", id: 1 } });
app.dispatch({ type: "CART_ADD", payload: { id: "p1", name: "Laptop", price: 999 } });
app.dispatch({ type: "TOGGLE_THEME" });
```

### Real-World Use Cases

| Scenario | Choose Mediator | Choose Observer |
|----------|----------------|-----------------|
| Chat application | Yes (chat server) | No |
| Stock ticker alerts | No | Yes (observers watch stock) |
| Form validation | Yes (mediator coordinates fields) | No |
| DOM event listeners | No | Yes (addEventListener) |
| Complex GUI dialog | Yes (dialog mediates controls) | No |
| State management | No (Flux/Redux uses observer-like pattern) | Yes (store notifies views) |
| Multiplayer game | Yes (game server mediates) | No |
| Plugin system | Yes (mediator coordinates plugins) | No |

### Common Mistakes

```javascript
// Mistake 1: Using Observer when Mediator is needed
// Many-to-many communication with just Observer creates spaghetti
users.forEach(user => {
  user.observe(order, () => { /* complex logic */ });
  user.observe(payment, () => { /* complex logic */ });
  order.observe(user, () => { /* complex logic */ });
});

// Mistake 2: Using Mediator when Observer is sufficient
// Simple one-to-many with a mediator is over-engineering
class SimpleMediator {
  notify(sender, event, data) {
    if (event === "price_changed") {
      display.update(data); // Just use Observer!
    }
  }
}

// Mistake 3: Mixing the patterns inconsistently
// Pick one approach and be consistent
```

### Best Practices

```javascript
// Choose Observer for:
// - UI event handlers
// - Reactive state subscriptions
// - Stream processing
// - Simple notification

// Choose Mediator for:
// - Complex forms/wizards
// - Workflow orchestration
// - Multi-step processes
// - Systems with many interacting components

// You can combine them:
// Mediator internally uses Observer to manage its subscriptions
// Observer components can be coordinated by a Mediator

// When in doubt, start with Observer and refactor to Mediator
// when coordination complexity warrants it
```

### Performance Considerations

- Observer: Direct notification, minimal overhead. O(n) per subject.
- Mediator: All communication goes through one point — can become a bottleneck.
- Mediator with many components can have higher latency due to routing logic.
- For real-time systems with thousands of events/second, Observer is generally faster.

### Interview Questions

**Q: Can Mediator and Observer be used together?**
A: Yes, they complement each other well. A Mediator can internally use Observer to manage subscriptions and notifications. The Mediator pattern provides the high-level coordination while Observer provides the low-level notification mechanism. Many real-world systems use this hybrid approach.

**Q: How would you refactor Observer-based code to use Mediator?**
A: (1) Identify complex interaction patterns in your observers. (2) Create a Mediator class that encapsulates the coordination logic. (3) Replace direct subject-observer relationships with mediated communication. (4) Keep simple observer relationships (one-to-many) as is.

**Q: What pattern does Redux use — Mediator or Observer?**
A: Redux is closer to Observer. The store is a subject, and React components are observers that subscribe to store changes via `useSelector`/`connect`. However, Redux's middleware system has Mediator-like qualities, with middleware mediating between dispatched actions and reducers.

### Coding Challenges

```javascript
// Challenge 1: Analyze the following system and refactor it from
// Observer to Mediator (or vice versa) as appropriate:
// - A dashboard with StockChart, PriceAlert, NewsFeed, and PortfolioSummary
// - All currently observe StockPrice directly
// - But StockPrice needs to coordinate with TradingBot and RiskAnalyzer

// Challenge 2: Implement a smart traffic light system where:
// - Traffic lights observe vehicle sensors (Observer)
// - But traffic lights need to coordinate with each other (Mediator)
// - Emergency vehicles need to override all lights (Mediator + Observer)

// Challenge 3: Build a hybrid system where a FormMediator uses
// internal observers to notify components of changes, while
// also providing external mediator methods for complex operations.
```

### Related Topics

- Observer pattern
- Pub/Sub pattern
- Event-driven architecture
- CQRS
- Flux/Redux architecture
