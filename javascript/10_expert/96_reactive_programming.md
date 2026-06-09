# Reactive Programming - Observables, RxJS, operators, streams, backpressure

## Introduction

Reactive programming is a programming paradigm oriented around data streams and the propagation of change. Instead of imperative sequences, you express logic as operations on streams of data that flow over time. RxJS (Reactive Extensions for JavaScript) is the most popular implementation, providing a powerful set of operators for composing asynchronous and event-based programs. Understanding reactive programming is essential for handling complex async flows, real-time data, and user interactions.

## Observable Pattern

### What It Is

An Observable represents a stream of values over time. It is a lazy push-based collection that emits items to observers when subscribed. Observables are like Promises but for multiple values over time. They can emit any number of values, then either complete or error.

### Why It Is Important

Observables solve the problem of handling multiple async values elegantly. Unlike callbacks (callback hell), Promises (single value), or EventEmitters (no built-in composition), Observables provide a unified model for any data source—user clicks, HTTP responses, WebSocket messages, array iterations—with powerful composition operators.

### How It Works Internally

An Observable is created with a subscriber function that receives an Observer (object with `next`, `error`, `complete` methods). The subscriber function defines how to produce values and returns a teardown function for cleanup.

When `.subscribe(observer)` is called:
1. The subscriber function runs
2. Values are pushed to `observer.next()`
3. On error, `observer.error()` is called
4. On completion, `observer.complete()` is called
5. On unsubscribe, the teardown function runs (cleanup)

```javascript
// Observable internals (simplified)
class Observable {
  constructor(subscribe) {
    this._subscribe = subscribe;
  }
  
  subscribe(observer) {
    const safeObserver = {
      next: (v) => { try { observer.next?.(v); } catch (e) { observer.error?.(e); } },
      error: (e) => observer.error?.(e),
      complete: () => observer.complete?.()
    };
    const teardown = this._subscribe(safeObserver);
    return { unsubscribe: () => teardown?.() };
  }
  
  pipe(...operators) {
    return operators.reduce((prev, operator) => operator(prev), this);
  }
}
```

### Syntax

```javascript
const { Observable } = require('rxjs');

// Creating an Observable
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});

// Subscribing
observable.subscribe({
  next: value => console.log(value),    // 1, 2, 3
  error: err => console.error(err),
  complete: () => console.log('Done')
});

// Observable that emits over time
const interval$ = new Observable(subscriber => {
  let count = 0;
  const id = setInterval(() => {
    subscriber.next(count++);
    if (count > 5) {
      subscriber.complete();
      clearInterval(id);
    }
  }, 1000);
  
  return () => clearInterval(id); // Teardown
});

const subscription = interval$.subscribe(console.log);
// Later: subscription.unsubscribe();
```

### Beginner Examples

```javascript
import { from, of, fromEvent } from 'rxjs';

// from: Convert array/iterable/promise to observable
const array$ = from([1, 2, 3]);
array$.subscribe(console.log); // 1, 2, 3

// of: Emit arguments as values
const values$ = of('a', 'b', 'c');
values$.subscribe(console.log); // a, b, c

// fromEvent: DOM events as observable
const clicks$ = fromEvent(document, 'click');
clicks$.subscribe(event => console.log('Clicked:', event.clientX));

// Promise as observable
const data$ = from(fetch('/api/data').then(r => r.json()));
data$.subscribe(data => console.log(data));

// Observable lifecycle
const test$ = new Observable(sub => {
  console.log('Observable created');
  sub.next(1);
  sub.next(2);
  return () => console.log('Cleanup');
});

const sub = test$.subscribe(v => console.log('Received:', v));
// Output:
// Observable created
// Received: 1
// Received: 2

sub.unsubscribe();
// Cleanup
```

### Intermediate Examples

```javascript
import { Observable, interval, Subject } from 'rxjs';

// Cold vs Hot Observables
// Cold: producer is created per subscription (new interval each time)
const cold$ = interval(1000);
cold$.subscribe(v => console.log('A:', v)); // Starts new interval
setTimeout(() => cold$.subscribe(v => console.log('B:', v)), 2000); // Starts another interval

// Hot: producer is shared (like Subject)
const subject = new Subject();
subject.subscribe(v => console.log('A:', v));
setTimeout(() => subject.subscribe(v => console.log('B:', v)), 2000);
subject.next(1); // A: 1
subject.next(2); // A: 2
// B subscribes here after 2 seconds
subject.next(3); // A: 3, B: 3

// Multicasting with share
import { share } from 'rxjs/operators';

const shared$ = interval(1000).pipe(share());
shared$.subscribe(v => console.log('A:', v));
setTimeout(() => shared$.subscribe(v => console.log('B:', v)), 2000);
// Now both A and B share the same interval

// Custom observable for WebSocket
function createWebSocket$(url) {
  return new Observable(subscriber => {
    const ws = new WebSocket(url);
    
    ws.onopen = () => console.log('WS connected');
    ws.onmessage = (event) => subscriber.next(JSON.parse(event.data));
    ws.onerror = (error) => subscriber.error(error);
    ws.onclose = () => subscriber.complete();
    
    return () => ws.close();
  });
}

const ws$ = createWebSocket$('wss://echo.websocket.org');
const sub = ws$.subscribe(msg => console.log('WS message:', msg));
```

## RxJS Library

### What It Is

RxJS is the Reactive Extensions library for JavaScript. It provides Observable creation functions, transformation/combination operators, and Subjects for multicasting. RxJS is used in Angular (HttpClient, Forms, Router), but is framework-agnostic. The library follows the ReactiveX API pattern.

### Why It Is Important

RxJS provides battle-tested, well-optimized primitives for reactive programming. It's the most popular reactive library in the JavaScript ecosystem with a rich operator set (100+ operators). RxJS powers Angular's async pipe, HTTP client, and reactive forms.

### How It Works Internally

The library consists of:
- **Observable**: Core type for push-based collections
- **Operators**: Pure functions that transform Observables (`map`, `filter`, `mergeMap`, etc.)
- **Subject**: Multicast observable (pushes to multiple subscribers)
- **Schedulers**: Control when execution happens (async, asap, animationFrame)
- **Subscriptions**: Represent the execution of an Observable

Operators are composable via `.pipe()` and return new Observables without mutating the source (purity).

### Syntax

```javascript
import { of, interval, throwError } from 'rxjs';
import { map, filter, catchError, retry, take } from 'rxjs/operators';

// Chain operators with pipe
const source$ = of(1, 2, 3, 4, 5);

source$.pipe(
  filter(x => x % 2 === 0),
  map(x => x * 10)
).subscribe(console.log); // 20, 40

// Error handling
interval(1000).pipe(
  map(x => {
    if (x > 3) throw new Error('Too high');
    return x;
  }),
  retry(2),
  catchError(err => of('Fallback value'))
);

// Creation functions
import { ajax } from 'rxjs/ajax';
const api$ = ajax.getJSON('https://api.example.com/data');

// Subjects
import { Subject, BehaviorSubject, ReplaySubject, AsyncSubject } from 'rxjs';
```

### Beginner Examples

```javascript
import { interval, fromEvent, merge, empty } from 'rxjs';
import { take, throttleTime, map, filter } from 'rxjs/operators';

// Timer that takes 5 values
const countdown$ = interval(1000).pipe(
  map(i => 5 - i),
  take(6)
);
countdown$.subscribe(console.log); // 5, 4, 3, 2, 1, 0

// Typeahead search
const searchInput = document.getElementById('search');
const typeahead$ = fromEvent(searchInput, 'input').pipe(
  map(e => e.target.value),
  filter(text => text.length >= 3),
  throttleTime(300)
);

typeahead$.subscribe(query => {
  console.log('Search for:', query);
  // fetch results
});

// Combine multiple streams
const mouseMoves$ = fromEvent(document, 'mousemove').pipe(
  map(e => ({ x: e.clientX, y: e.clientY }))
);

const clicks$ = fromEvent(document, 'click').pipe(
  map(e => ({ clickX: e.clientX, clickY: e.clientY }))
);

merge(mouseMoves$, clicks$).subscribe(console.log);

// Simple state management with BehaviorSubject
import { BehaviorSubject } from 'rxjs';

const state$ = new BehaviorSubject({ count: 0 });
state$.subscribe(console.log); // { count: 0 } (immediate current value)

state$.next({ count: 1 }); // { count: 1 }
state$.next({ count: 2 }); // { count: 2 }
```

### Intermediate Examples

```javascript
import { Subject, merge, combineLatest, zip, forkJoin } from 'rxjs';
import { map, scan, withLatestFrom, startWith } from 'rxjs/operators';

// State management with scan (like Redux)
type Action = { type: 'INCREMENT' | 'DECREMENT' | 'RESET'; payload?: number };

const actions$ = new Subject<Action>();

const state$ = actions$.pipe(
  startWith({ count: 0 }),
  scan((state, action) => {
    switch (action.type) {
      case 'INCREMENT': return { count: state.count + 1 };
      case 'DECREMENT': return { count: state.count - 1 };
      case 'RESET': return { count: 0 };
      default: return state;
    }
  })
);

state$.subscribe(console.log);
actions$.next({ type: 'INCREMENT' }); // { count: 1 }
actions$.next({ type: 'INCREMENT' }); // { count: 2 }
actions$.next({ type: 'DECREMENT' }); // { count: 1 }

// combineLatest: latest values from multiple streams
const name$ = new Subject();
const age$ = new Subject();

combineLatest([name$, age$]).subscribe(([name, age]) => {
  console.log(`${name} is ${age} years old`);
});

name$.next('Alice');
age$.next(30);    // 'Alice is 30'
name$.next('Bob'); // 'Bob is 30'

// Current value with BehaviorSubject
import { BehaviorSubject, combineLatest } from 'rxjs';

const search$ = new BehaviorSubject('');
const results$ = new BehaviorSubject([]);

combineLatest([search$, results$]).subscribe(([search, results]) => {
  // Both always have a value
  console.log(`Search: "${search}", ${results.length} results`);
});

// forkJoin: wait for all to complete (like Promise.all)
import { forkJoin, of } from 'rxjs';
import { delay } from 'rxjs/operators';

const req1$ = of('User data').pipe(delay(1000));
const req2$ = of('Posts data').pipe(delay(2000));

forkJoin([req1$, req2$]).subscribe(([user, posts]) => {
  console.log('All data loaded:', user, posts);
});

// Higher-order mapping
import { fromEvent, interval } from 'rxjs';
import { switchMap, debounceTime, distinctUntilChanged } from 'rxjs/operators';

// switchMap: cancel previous inner observable
const search$ = fromEvent(searchInput, 'input').pipe(
  map(e => e.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => fetch(`/api/search?q=${query}`).then(r => r.json()))
);

search$.subscribe(results => displayResults(results));
```

### Advanced Examples

```javascript
import { Observable, Subject, merge, timer, EMPTY } from 'rxjs';
import { 
  mergeMap, concatMap, exhaustMap, switchMap,
  groupBy, buffer, windowTime, throttle, audit,
  shareReplay, publish, refCount, multicast
} from 'rxjs/operators';

// Concurrency control operators
// mergeMap: all inner observables run concurrently
// concatMap: one at a time (queue)
// switchMap: cancel previous, start new (latest only)
// exhaustMap: ignore new until current completes

function simulateRequest(id: number): Observable<string> {
  const delay = Math.random() * 2000 + 500;
  return of(`Result ${id}`).pipe(delay(delay));
}

// mergeMap (parallel)
from([1, 2, 3, 4]).pipe(
  mergeMap(id => simulateRequest(id), 2) // concurrency: 2
).subscribe(console.log);

// concatMap (sequential)
from([1, 2, 3]).pipe(
  concatMap(id => simulateRequest(id))
).subscribe(console.log);

// buffer: collect values over time
fromEvent(document, 'click').pipe(
  buffer(interval(1000))
).subscribe(clicks => {
  console.log(`${clicks.length} clicks in the last second`);
});

// groupBy: partition stream by key
from([
  { type: 'A', value: 1 },
  { type: 'B', value: 2 },
  { type: 'A', value: 3 }
]).pipe(
  groupBy(item => item.type),
  mergeMap(group => group.pipe(
    map(items => ({ type: group.key, sum: items.reduce((a, b) => a + b.value, 0) }))
  ))
).subscribe(console.log);

// Share replay for caching
const apiData$ = ajax.getJSON('/api/data').pipe(
  shareReplay(1, 5000) // Cache for 5 seconds
);

// Multiple subscribers share the same HTTP request
apiData$.subscribe(data => console.log('A:', data));
apiData$.subscribe(data => console.log('B:', data)); // Gets cached result

// Custom operator
function retryWithBackoff(maxRetries = 3, delayMs = 1000) {
  return (source$) => source$.pipe(
    retryWhen(errors => errors.pipe(
      concatMap((error, index) => {
        if (index >= maxRetries) throw error;
        return timer(delayMs * Math.pow(2, index));
      })
    ))
  );
}

// Usage
ajax.getJSON('/api/unreliable').pipe(
  retryWithBackoff(3, 500)
).subscribe(console.log);
```

## Operators (map, filter, mergeMap, debounceTime)

### What It Is

RxJS operators are pure functions that create new Observables based on source Observables. They transform, filter, combine, or schedule data flow. Operators are the building blocks of reactive programming, enabling complex data flow with simple, composable functions.

### Why It Is Important

Operators provide a declarative vocabulary for describing async data flow. Instead of writing nested callbacks or complex Promise chains, operators express transformations concisely. Understanding the key operators (especially the higher-order mapping operators) is essential for effective reactive programming.

### How It Works Internally

Each operator returns a function that takes a source Observable and returns a new Observable. The new Observable subscribes to the source and transforms the values. Because operators are pure, they can be composed freely.

```javascript
// How operators work internally
function map(project) {
  return (source$) => new Observable(subscriber => {
    return source$.subscribe({
      next: value => subscriber.next(project(value)),
      error: err => subscriber.error(err),
      complete: () => subscriber.complete()
    });
  });
}

function filter(predicate) {
  return (source$) => new Observable(subscriber => {
    return source$.subscribe({
      next: value => { if (predicate(value)) subscriber.next(value); },
      error: err => subscriber.error(err),
      complete: () => subscriber.complete()
    });
  });
}
```

### Syntax

```javascript
import { of, fromEvent, interval } from 'rxjs';
import { 
  map, filter, reduce, scan,
  mergeMap, switchMap, concatMap, exhaustMap,
  debounceTime, throttleTime, delay, timeout,
  take, takeUntil, distinctUntilChanged,
  catchError, retry, finalize,
  combineLatest, withLatestFrom, startWith
} from 'rxjs/operators';

// Category: Transformation
map(x => x * 2)
scan((acc, x) => acc + x, 0)
reduce((acc, x) => acc + x, 0)

// Category: Filtering
filter(x => x > 5)
take(3)
debounceTime(300)
distinctUntilChanged()
skip(2)

// Category: Combination
mergeMap(x => inner$(x))
switchMap(x => inner$(x))    // Cancel previous
concatMap(x => inner$(x))   // Queue
exhaustMap(x => inner$(x))  // Ignore during

// Category: Error Handling
catchError(err => of(fallback))
retry(3)
finalize(() => cleanup())
```

## Streams and Backpressure

### What It Is

Streams represent sequences of data over time. Backpressure is the mechanism for handling situations where data is produced faster than it can be consumed. Without backpressure, unbounded buffering can cause memory exhaustion. RxJS handles backpressure through controlled push (operators like `buffer`, `throttle`, `sample`, `audit`).

### Why It Is Important

In real-world applications, data sources (sensors, user input, network) often produce data faster than it can be processed. Without backpressure handling, applications crash from memory exhaustion or become unresponsive. Reactive programming provides declarative backpressure strategies.

### How It Works Internally

RxJS operators handle backpressure in several ways:
- **Lossy**: Drop values (`throttle`, `sample`, `audit`, `debounce`)
- **Buffering**: Collect values (`buffer`, `bufferTime`, `window`)
- **Controlled pull**: `Observable` to `Iterable` conversion (backpressure-aware)
- **Concurrency limiting**: `mergeMap` with concurrency parameter

### Syntax

```javascript
import { fromEvent, interval } from 'rxjs';
import { 
  throttle, throttleTime, debounce, debounceTime,
  sample, sampleTime, audit, auditTime,
  buffer, bufferTime, bufferCount, bufferToggle,
  window, windowTime,
  mergeMap, concatMap, exhaustMap
} from 'rxjs/operators';

// Lossy backpressure
fromEvent(document, 'mousemove').pipe(
  throttleTime(100) // Emit at most once per 100ms
);

// Buffering backpressure
fromEvent(document, 'click').pipe(
  bufferTime(1000) // Collect clicks in 1-second buffers
);

// Concurrency control
from(someArray).pipe(
  mergeMap(item => handleItem(item), 2) // Max 2 concurrent
);
```

### Beginner Examples

```javascript
import { fromEvent, interval } from 'rxjs';
import { throttleTime, buffer, auditTime, sampleTime } from 'rxjs/operators';

// throttleTime: emit first, ignore for duration
const throttledClicks$ = fromEvent(document, 'click').pipe(
  throttleTime(1000)
);
throttledClicks$.subscribe(e => console.log('Click (throttled)'));

// debounceTime: emit after quiet period
const debounced$ = fromEvent(searchInput, 'input').pipe(
  debounceTime(300)
);
debounced$.subscribe(e => console.log('User stopped typing'));

// auditTime: emit last value in window
fromEvent(document, 'mousemove').pipe(
  auditTime(5000) // Emit last mouse position every 5 seconds
).subscribe(e => console.log('Position:', e.clientX, e.clientY));

// buffer: collect values and emit as array
fromEvent(document, 'click').pipe(
  buffer(interval(2000))
).subscribe(clicks => {
  console.log(`2-second window: ${clicks.length} clicks`);
});

// sampleTime: emit latest at interval
fromEvent(document, 'mousemove').pipe(
  sampleTime(1000)
).subscribe(e => console.log('Sample at 1Hz:', e.clientX, e.clientY));
```

### Real-World Use Cases

- Autocomplete search (debounce + distinctUntilChanged + switchMap)
- Infinite scroll (throttle scroll events)
- Live dashboards (WebSocket streams with sample/frequency control)
- Drag-and-drop (mouse event streams)
- Real-time collaboration (Operational Transformation with CRDTs)
- Game input handling (throttle input frequency)
- Form validation (debounce + combineLatest)
- Request deduplication (switchMap cancels inflight requests)

### Common Mistakes

- Subscribing to Observables inside subscribe() (nested subscriptions)
- Not unsubscribing (memory leaks)
- Using `mergeMap` when `switchMap` is appropriate (API requests)
- Forgetting error handling in Observable chains
- Confusing hot vs cold Observables
- Not using `shareReplay` for shared HTTP requests
- Overusing RxJS for simple operations (over-engineering)

### Best Practices

- Always unsubscribe (use `takeUntil`, `async` pipe, or managed subscriptions)
- Use `switchMap` for "latest" scenarios (search, navigation)
- Use `mergeMap` for parallel independent operations
- Use `concatMap` for sequential order-dependent operations
- Use `exhaustMap` for operations that shouldn't overlap (save, submit)
- Use `debounceTime` + `distinctUntilChanged` for search inputs
- Use `shareReplay(1)` for caching observable results
- Keep operators in `pipe()` for readability
- Handle errors with `catchError` at the end of pipes

### Performance Considerations

- Observable creation is cheap (just defines behavior)
- Operators create new Observables (chain of wrappers)
- Unsubscribing prevents memory leaks (cleanup teardowns)
- `share`/`shareReplay` prevents duplicate subscriptions
- Higher-order mapping operators manage inner subscriptions
- Without backpressure handling, unbounded buffers cause OOM
- Performance of RxJS is well-optimized for most use cases

### Interview Questions

**Q: What is the difference between `mergeMap` and `switchMap`?** A: `mergeMap` subscribes to all inner Observables concurrently and emits their values as they arrive. `switchMap` cancels the previous inner Observable when a new value arrives, subscribing only to the latest. Use `mergeMap` for parallel independent operations (multiple concurrent requests), `switchMap` for "latest only" scenarios (search autocomplete, route navigation).

**Q: What is backpressure and how does RxJS handle it?** A: Backpressure is the buildup of data when producers emit faster than consumers can process. RxJS handles it through operators that control the flow: lossy operators (throttle, debounce, sample) drop values; buffering operators (buffer, window) batch values; and concurrency control (mergeMap with concurrency limit) restricts parallel processing.


### Coding Challenges
```javascript
// Challenge 1: Simple Observable implementation
class Observable {
  constructor(subscribe) { this._subscribe = subscribe; }
  subscribe(observer) { return this._subscribe(observer); }
  map(fn) {
    return new Observable(observer => 
      this.subscribe({ next: x => observer.next(fn(x)), error: e => observer.error(e), complete: () => observer.complete() })
    );
  }
}

const obs = new Observable(observer => {
  observer.next(1); observer.next(2); observer.next(3); observer.complete();
});
obs.map(x => x * 10).subscribe({ next: console.log });

// Challenge 2: RxJS style debounce
const { fromEvent } = require('rxjs');
const { debounceTime, map } = require('rxjs/operators');
// fromEvent(document.getElementById('search'), 'input').pipe(
//   debounceTime(300),
//   map(e => e.target.value)
// ).subscribe(console.log);

// Challenge 3: Merge multiple streams
// const { merge } = require('rxjs');
// const { scan } = require('rxjs/operators');
// const clicks1 = fromEvent(btn1, 'click');
// const clicks2 = fromEvent(btn2, 'click');
// merge(clicks1, clicks2).pipe(
//   scan(count => count + 1, 0)
// ).subscribe(count => console.log(`Clicked ${count} times`));
```

### Related Topics

- Observer pattern
- Functional reactive programming
- Stream processing
- Promises vs Observables
- Angular's use of RxJS
- State management with observables
- Event sourcing and CQRS
- Node.js streams (push vs pull)
- Async Iterators and Generators
- WebSocket and real-time protocols
